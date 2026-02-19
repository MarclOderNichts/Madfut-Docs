# MADFUT APK Documentation

---

## Overview

This document reconstructs runtime behavior of the decompiled `com.trivela.madfut` APK at Smali level.

Primary targets:

```
smali/com/trivela/madfut/MainActivity.smali
smali/C5/j.smali
smali/f5.1/k.smali
smali/D5/r.smali
smali/X5/b.smali
smali/com/trivela/madfut/realm/Player.smali
```

---

# 1. Architectural Model

The application follows an Activity-centered monolithic orchestration model with:

- Global mutable runtime state
- Static service locator registry
- Obfuscated feature modules
- View-driven rendering
- Realm-backed persistence
- Extensive SDK integration

No Unity / Unreal / libGDX runtime.

Rendering pipeline is Android View-based.

---

# 2. Runtime Control Plane

## 2.1 MainActivity

File:

```
smali/com/trivela/madfut/MainActivity.smali
```

### Responsibilities

- UI root inflation (`activity_main`)
- Global module graph construction
- Service registry binding
- Lifecycle state synchronization
- Mode routing
- Network listener attachment
- Permission management
- Foreground/background flag management

### Startup Behavior (`onCreate`)

Observed sequence:

```
1. Window/theme setup
2. Layout inflation
3. Static registry binding
4. Module instantiation
5. Persistent data load
6. Async listener registration
```

Method body size indicates heavy graph assembly during cold start.

---

## 2.2 Global Runtime State — `C5/j`

File:

```
smali/C5/j.smali
```

### Structural Characteristics

- High number of static fields
- Boolean mode flags
- Static object references
- Lazy providers (`LH6/o`)
- Firestore accessor (`l()`)

### Functional Role

Acts as:

```
Process-wide state bus
```

Encodes feature state transitions via flag combinations.

### Risk Profile

- High coupling
- Implicit transitions
- Lifecycle-sensitive behavior
- Difficult deterministic reset

---

## 2.3 Service Locator — `f5.1/k`

File:

```
smali/f5.1/k.smali
```

### Pattern

Static registry exposing accessors:

```
A()
B()
i0()
k0()
z0()
...
```

Backed by static module instances from packages:

```
D5/*
I5/*
R5/*
L5/*
Q5/*
```

### Architectural Effect

- Low friction access
- Tight global coupling
- Shared lifetime across features

---

# 3. UI & Scene Composition

## 3.1 Root Layout

```
res/layout/activity_main.xml
```

Contains:

- fragmentContainer
- toolbar
- title/search area
- dialogs container
- reward/message overlays
- ad banner container
- splash overlay include

UI shell remains constant; feature swaps are fragment-driven.

---

## 3.2 Main Menu Layer

```
res/layout/fragment_main_menu.xml
```

Architecture:

- ViewPager (3 pages)
- Worm dots indicator

Indicates page-based navigation instead of multiple Activities.

---

## 3.3 Custom View Surface

Directory:

```
smali/com/trivela/madfut/customViews/
```

Contains:

- Card views
- Chemistry bars
- Draft widgets
- Fatal mode screens
- Pick interfaces
- Knockout widgets

Rendering logic resides inside custom View subclasses.

---

# 4. Gameplay Logic Reconstruction

## 4.1 Implicit State Machine

State encoded via boolean flags in:

```
C5/j
```

Interpreted in:

```
MainActivity.dispatchTouchEvent
MainActivity.onResume
```

Branch clusters correlate with:

- Draft mode
- Squad builder
- Fatal modes
- Trading flows
- Selection interfaces

Transitions are flag-driven, not enum-driven.

---

## 4.2 Chemistry Engine — `D5/r`

File:

```
smali/D5/r.smali
```

Constructor builds dense adjacency maps.

Functional purpose:

- Position relation graph
- Formation constraints
- Deterministic chemistry scoring

Data structures:

- Position index maps
- Neighbor matrices
- Formation adjacency lists

Indicates deterministic graph-based evaluation model.

---

## 4.3 Reward Query Model — `X5/b`

File:

```
smali/X5/b.smali
```

Acts as parameterized query container.

Contains:

- Rating bounds
- Position constraints
- Toggle flags
- JSON/Map constructor path
- Query key materialization

Likely sits between:

```
Firestore payload → Domain object → UI projection
```

---

## 4.4 Core Domain Entity — `realm/Player`

File:

```
smali/com/trivela/madfut/realm/Player.smali
```

Characteristics:

- Realm-backed model
- Numerous scalar/stat fields
- Companion/schema metadata

Represents canonical card entity used across features.

Most feature projections derive from this entity.

---

# 5. Data Flow Model

## 5.1 Origins

- Touch input
- Realm persistence
- SharedPreferences
- Firebase (Firestore / RTDB)
- In-memory global state

## 5.2 Transformations

- Gson JSON deserialization
- Map → Domain object constructors
- Computed projections (chemistry/scoring)
- Query filtering

## 5.3 Persistence

- Realm database (asset-seeded `default.realm`)
- SharedPreferences
- Firebase cloud state

---

# 6. Lifecycle & Control Flow

## 6.1 onCreate

Graph construction and persistent hydration.

Heavy static field writes.

---

## 6.2 onPause

- Foreground flag reset
- Helper suspension
- Mode persistence

---

## 6.3 onResume

- Foreground flag restore
- Mode reactivation
- Reflection-based animation-scale correction path

---

## 6.4 Input Dispatch

```
dispatchTouchEvent
```

Performs cleanup of transient selection widgets on gesture cancellation.

Indicates manual state normalization.

---

# 7. Smali Characteristics

## 7.1 Invocation Patterns

- Frequent `invoke-virtual`
- Static accessors
- Lazy evaluation via `LH6/o.getValue()`

## 7.2 Control Flow

- High branch density
- `packed-switch` / `sparse-switch`
- Large obfuscated methods merging feature logic

## 7.3 Lambda Footprint

Obfuscated namespaces (e.g., `f5.1/*`) contain Kotlin-lowered lambdas and callbacks.

---

# 8. Security & Integrity Surface

## 8.1 Pairip License Protection

Directory:

```
smali/com/pairip/licensecheck/
```

Observed mechanisms:

- ContentProvider early trigger
- Binder call to:
  com.android.vending.licensing.ILicensingService
- Installer verification via `getInstallSourceInfo`
- JWS parsing
- RSA signature verification (`SHA256withRSA`)
- Fallback paywall/exit logic

This is a functional integrity layer.

---

## 8.2 Crypto Surface

- RSA public key embedded in license client
- No custom cipher implementation detected in core packages

---

## 8.3 Reflection / Dynamic Loading

- Reflection present in app
- Dex loading mainly in SDK internals
- No app-owned dynamic module loader observed

---

## 8.4 Integrity Anomaly

`MainActivity.onCreate` contains injected Base64 toast path referencing external branding.

This is inconsistent with normal startup logic.

Conclusion:

```
Sample likely repackaged or modified.
```

Signature/hash comparison recommended before drawing security conclusions.

---

# 9. Dependency Surface

SDK presence includes:

- Firebase (Auth, Firestore, Messaging, Crashlytics, App Check)
- Google Play Services
- Play Integrity
- AppLovin mediation
- WorkManager
- AndroidX startup components

Internal coupling is high due to static registry model.

---

# 10. Further Reverse Engineering Targets

High-value next steps:

```
1. CFG extraction for MainActivity lifecycle
2. Full flag matrix derivation for C5/j
3. Firestore collection path tracing
4. Signature/hash verification
5. Removal of injected startup branch for clean baseline
```

---

