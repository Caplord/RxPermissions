# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RxPermissions is an Android library that wraps Android 6.0+ runtime permissions with RxJava 3. It allows developers to request permissions using reactive streams instead of managing callbacks through `Activity.onRequestPermissionsResult()`.

**Multi-module structure:**
- `lib/` - Main library module
- `sample/` - Example Android application demonstrating library usage

## Build & Development Commands

### Build Commands

```bash
# Build all modules
./gradlew build

# Build library only
./gradlew lib:build

# Build with no tests
./gradlew build -x test

# Clean build artifacts
./gradlew clean
```

### Testing

```bash
# Run all unit tests
./gradlew test

# Run tests for library module only
./gradlew lib:test

# Run a single test class
./gradlew lib:test --tests com.tbruyelle.rxpermissions3.RxPermissionsTest

# Run tests with detailed output
./gradlew lib:test --info

# Run tests without cache (fresh run)
./gradlew lib:test --rerun-tasks
```

### Compilation & Linting

```bash
# Assemble library AAR
./gradlew lib:assemble

# Compile Java sources (without packaging)
./gradlew lib:compileDebugJavaWithJavac lib:compileReleaseJavaWithJavac

# Check Android Lint
./gradlew lib:lint
```

### Publishing/Installation

```bash
# Publish to local Maven repository
./gradlew lib:publishToMavenLocal

# Build AAR for release
./gradlew lib:assembleRelease
```

## Architecture

### Core Components

**RxPermissions (lib/src/main/java/com/tbruyelle/rxpermissions3/RxPermissions.java)**
- Main public API entry point
- Accepts either a `FragmentActivity` or `Fragment` in constructor
- Uses lazy-loading pattern with a headless `RxPermissionsFragment` to handle permission requests
- Provides methods: `request()`, `requestEach()`, `requestEachCombined()`, `ensure()`, `ensureEach()`, `ensureEachCombined()`
- Returns observables that emit `Boolean` (combined results) or `Permission` objects (detailed per-permission results)

**RxPermissionsFragment (lib/src/main/java/com/tbruyelle/rxpermissions3/RxPermissionsFragment.java)**
- Headless fragment added to the activity's fragment manager
- Located using a tag in `FragmentManager`
- Intercepts `onRequestPermissionsResult()` callbacks from the OS
- Emits results via `PublishSubject` observables
- Handles pre-M devices (automatically grants permissions) and post-M devices

**Permission (lib/src/main/java/com/tbruyelle/rxpermissions3/Permission.java)**
- Data class representing a single permission result
- Fields: `name`, `granted`, `shouldShowRequestPermissionRationale`
- Can combine multiple `Permission` objects via constructor overload
- Uses RxJava blocking operations (`blockingGet()`) for combining results

### Key Design Patterns

1. **Fragment-based approach**: Uses a headless fragment to survive configuration changes and handle lifecycle properly
2. **Lazy singleton pattern**: `RxPermissionsFragment` is created on first use and reused
3. **Publisher pattern**: Uses RxJava `PublishSubject` to emit permission request results
4. **Observable composition**: Request methods return operators that can be composed with other RxJava streams
5. **Backward compatibility**: Pre-M devices automatically receive granted results

### Permission Request Flow

1. User calls `rxPermissions.request(...)` or similar method
2. RxPermissions obtains (or creates) the `RxPermissionsFragment`
3. Fragment calls `ActivityCompat.requestPermissions()` via `requestPermissions()`
4. OS shows permission dialog (or auto-grants on pre-M)
5. Fragment receives callback in `onRequestPermissionsResult()`
6. Results are emitted through the subscription chain
7. Subscriber receives Boolean or Permission object

### Important Constraints

- **Must be initialized in onCreate or similar**: Permission requests must be set up during initialization phases, not in `onResume` or other pausing methods, to avoid infinite loops if the app is restarted during the request
- **Fragment vs FragmentActivity**: When using with a Fragment, pass the fragment instance (not the parent activity) to ensure proper fragment manager use
- **Java 21**: Project compiles with Java 21 bytecode (source/target compatibility set to VERSION_21). Java 21 toolchain configured via Foojay auto-provisioning (AGP 8.10.0+ supports Java 21)
- **RxJava 3**: Library uses RxJava 3 (reactive-streams via `io.reactivex.rxjava3`)

## Testing

Unit tests use **Robolectric** for Android component simulation (no emulator required). Key test file:
- `lib/src/test/java/com/tbruyelle/rxpermissions3/RxPermissionsTest.java`

**Robolectric Configuration:**
- Version: 4.12.2 (supports Java 21 compilation)
- Special JVM args in `lib/build.gradle` to handle Java 21 compatibility (--add-opens flags)
- **Critical**: Test SDK level must match or be compatible with compileSdkVersion (36). Tests are configured with `@Config(sdk = Build.VERSION_CODES.TIRAMISU)` to ensure Robolectric's manifest parser works correctly
- If test SDK is significantly lower than compileSdk, Robolectric's PackageParser may fail due to API incompatibilities

## Gradle Configuration

**Modernized Build (caplord fork - version 1.0.1-java21+):**
- **Gradle**: 8.14.4
- **Android Gradle Plugin**: 8.10.0
- **Min SDK**: 26
- **Compile SDK**: 36
- **Target SDK**: 36
- **Build Tools**: 36.x.x (auto-selected by AGP)
- **Java Version**: 21 source/target (full Java 21 bytecode compilation enabled)
- **Kotlin stdlib exclusions**: Added to all dependencies to prevent duplicate class warnings
- **Foojay auto-provisioning**: Enables automatic JDK 21 download via SDK manager
- **AndroidX enabled**: `android.useAndroidX = true` in gradle.properties

**Original Repository (Official - version 0.9.5):**
Still uses older configuration (Java 8, SDK 29, AGP 7.0.4, Gradle 7.6.4). The caplord fork provides modernized build configuration for projects requiring Java 21, SDK 36, and AGP 8.10.0 compatibility.

## Publishing

**Original Repository (tbruyelle/RxPermissions):**
- Published to JitPack as `com.github.tbruyelle:RxPermissions`
- Latest stable version: `0.9.5`

**Modernized Fork (caplord/RxPermissions):**
- Published to JitPack as `com.github.caplord:rxpermissions`
- Latest version: `1.0.1-java21` (Java 21 bytecode compilation enabled)
- Use this fork in projects requiring Java 21 compilation, SDK 36, and AGP 8.10.0+ support

Maven publication is configured in `lib/build.gradle` via maven-publish plugin for JitPack distribution.

## Custom Fork (caplord)

A custom fork is maintained at `github.com/caplord/RxPermissions` with modernized build configuration aligned with consuming projects.

**Latest Version 1.0.1-java21:**
- **Gradle**: 8.14.4 (from 8.10.2)
- **Android Gradle Plugin**: 8.10.0 (from 8.1.0)
- **compileSdkVersion**: 36
- **minSdkVersion**: 26
- **targetSdkVersion**: 36
- **Build Tools**: 36.x.x (auto-selected by AGP)
- **Java Compilation**: 21 source/target (full Java 21 bytecode enabled)
- **Java 21 Toolchain**: Foojay auto-provisioning enabled
  - JDK 21 automatically downloaded and used for compilation
  - Full Java 21 support with AGP 8.10.0
- **Robolectric**: 4.12.2 (supports Java 21 compilation)
- **AndroidX**: fragment 1.6.2, annotation 1.7.1, appcompat 1.7.1
- **Fixes**: Kotlin stdlib duplicate class errors, manifest configuration for Android 12+

**Previous Versions:**
- `1.0.0-java21-ready`: Java 21 toolchain configured but source/target set to Java 17 (AGP 8.1.0 limitation)
- `1.0.0-java17-aligned`: SDK-aligned version without Java 21 toolchain
- `1.0.0-java17-modern`: Initial modernized build with minSdk 21, compileSdk 35
- `1.0.0-java17-compat`: Initial semantic version of testBuildJitPack
- `testBuildJitPack`: Original test build (deprecated)

**Reason:** Official 0.9.5 stable release is not compatible with:
- Java 17 compilation (uses Java 8)
- Android SDK 36 compilation
- Robolectric 4.10.3+ (Java 17+ support)
- Mockito 5.x (Java 17+ support)
- AGP 8.x (requires SDK 30+)
- Java 21 compilation and toolchain

**Used by:** openfleet_android project (and others)

This fork provides a modern build solution with full Java 21 compilation support and alignment with Android 15+ SDK requirements. Should be retired once the official repository releases a stable version with contemporary Android tooling support, Java 21, SDK 36, and AGP 8.10.0+ compatibility.
