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
- **Java 8+**: Project uses Java 8 syntax (lambdas) with target compatibility set to Java 1.8
- **RxJava 3**: Library uses RxJava 3 (reactive-streams via `io.reactivex.rxjava3`)

## Testing

Unit tests use **Robolectric** for Android component simulation (no emulator required). Key test file:
- `lib/src/test/java/com/tbruyelle/rxpermissions3/RxPermissionsTest.java`

Robolectric version: 4.10.3 (configured with special JVM args in `lib/build.gradle` to handle Java 17 compatibility)

## Gradle Configuration

- **Android Gradle Plugin**: 7.0.4
- **Min SDK**: 14
- **Compile SDK**: 29
- **Target SDK**: 29
- **Java Version**: 1.8 source/target
- **AndroidX enabled**: `android.useAndroidX = true` in gradle.properties

## Publishing

Library is published to JitPack. Current version structure in `build.gradle`:
- Published group ID: `com.github.tbruyelle`
- Artifact: `RxPermissions`
- Version: `0.9.5` (in `libraryVersion` property)

Maven publication is configured in `lib/build.gradle` for distribution.

## Custom Fork (caplord)

A custom fork is maintained at `github.com/caplord/RxPermissions` with additional compatibility fixes:
- **Version**: `1.0.0-java17-compat` tag
- **Fixes included**: Java 17, Robolectric 4.10.3, and Mockito 5.11.0 compatibility
- **Reason**: Official 0.12 stable release is not compatible with these versions
- **Used by**: openfleet_android project

This fork is temporary and should be retired once the official repository releases a stable version with these compatibility fixes.
