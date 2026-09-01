# Repository Guidelines

## Project Overview

This is the `Caplord/RxPermissions` fork: a Java Android library that exposes runtime-permission requests as RxJava 3 streams. The current checked-out release is `1.0.4-java21`, modernized for Java 21, SDK 36, Gradle 8.14.4, and AGP 8.10.0. The consuming OpenFleet Android app uses the JitPack coordinate `com.github.caplord:rxpermissions:1.0.4-java21`.

## Project Structure

- `lib/` is the published `rxpermissions` Android library. Core classes are `RxPermissions`, the headless `RxPermissionsFragment`, and the `Permission` value object under `lib/src/main/java/com/tbruyelle/rxpermissions3`.
- `sample/` demonstrates library usage and should remain buildable after public API or dependency changes.
- Root `build.gradle` owns shared SDK/dependency/publication metadata; `lib/build.gradle` owns Android library, Java toolchain, Robolectric, and Maven publication configuration.
- `settings.gradle` configures Foojay JDK provisioning and includes both modules.

## Build, Test, and Publish Commands

- `./gradlew build` builds all modules; `./gradlew :rxpermissions:build` limits work to the library (`settings.gradle` renames the `lib/` directory project to `rxpermissions`).
- `./gradlew test` runs unit tests; `./gradlew :rxpermissions:test --tests com.tbruyelle.rxpermissions3.RxPermissionsTest` runs the main test class.
- `./gradlew :rxpermissions:lint` runs Android lint and `./gradlew :rxpermissions:assembleRelease` builds the release AAR.
- `./gradlew :rxpermissions:publishToMavenLocal` validates Maven publication locally. Do not create tags or publish externally without explicit authorization.

## Required Consumer Validation

- First validate this repository with `./gradlew :rxpermissions:build :rxpermissions:test :rxpermissions:lint`; when public behavior changes, also run `./gradlew :sample:assembleDebug :sample:testDebugUnitTest`.
- `openfleet_android` consumes a released JitPack tag through `android_dependencies/dependencies.gradle`; local changes are not tested by the app until a tag/version is intentionally selected there.
- When integrating a new version, update `rxPermissions_version`, then from the `openfleet_android` root run `./gradlew :openfleet_installed:assembleOpenfleetStagingDebug`, `./gradlew allTests`, and `./gradlew allLint`.
- Report library and consumer results separately. Do not claim main-project validation until the consumer resolves the tested release.

## Architecture and Compatibility Rules

- Permission requests are mediated by a retained/headless AndroidX fragment and emitted through RxJava 3. Preserve lifecycle behavior and configuration-change handling.
- Requests must be initialized from a stable lifecycle phase such as `onCreate`, not repeatedly from `onResume`. Fragment callers must pass the fragment, not its activity, so the correct FragmentManager is used.
- Keep pre-runtime-permission behavior, per-permission results, combined results, rationale flags, and cancellation/error semantics backward compatible.
- The library uses Java 21 source/target and toolchain settings, `minSdk 26`, and `compileSdk`/`targetSdk 36. Coordinate build-tool changes across the wrapper, AGP, Foojay resolver, Robolectric, and CI/JitPack expectations.
- The main package is `com.tbruyelle.rxpermissions3`; do not rename it or change public method signatures in a patch/minor fork release.

## Testing and Code Style

- Unit tests use JUnit 4, Mockito, and Robolectric. Keep tests deterministic and avoid requiring a device or emulator for library logic.
- Preserve the Robolectric SDK configuration and Java 21 `--add-opens` arguments unless a verified toolchain upgrade removes the need.
- Use four-space Java indentation, clear null/lifecycle handling, and thread-safe collections for concurrent permission results. Add regression tests for batches, duplicate requests, cancellation, and configuration changes.

## Commits and Releases

- Follow the existing Conventional Commit style and keep dependency/toolchain upgrades separate from behavior changes.
- Update `README.md`, version metadata, and this file when the supported SDK/JDK or published coordinate changes.
- Verify `build`, `:rxpermissions:test`, `:rxpermissions:lint`, and local publication before tagging. Never overwrite or move a published tag.
