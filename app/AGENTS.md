# AGENTS.md

This file applies to the Android application module.

Continue following the repository-level `AGENTS.md`. This file adds application-module rules for Android UI, resources, Manifest, Gradle module configuration, and app lifecycle behavior.

## Module direction

Predictive back changes in this module should use AndroidX-first behavior.

Prefer lifecycle-aware AndroidX back handling over direct platform callbacks. If a direct platform API is introduced, keep it isolated and document the reason. Do not spread Android-version checks across unrelated Activity or Fragment code.

The goal is not to make the app depend on Android 16-only behavior. The goal is to make back behavior correct, compatible, and ready for predictive back across supported Android versions.

## Android app boundaries

Changes in this module can affect VPN startup, permission requests, import flows, proxy profile editing, database access, plugins, and foreground services. Treat back behavior changes as UI/navigation changes unless the task explicitly requires deeper logic changes.

Do not change service startup, VPN permission behavior, profile storage, database schema, proxy protocol parsing, subscription update logic, or plugin communication as part of predictive back work unless there is a direct and documented reason.

## Activity and Fragment behavior

When changing back behavior in an Activity or Fragment:

- Preserve the previous user-visible result.
- Keep confirmation flows for unsaved data.
- Let temporary UI state consume back before the screen exits.
- Avoid calling `finish()` earlier than before.
- Avoid directly replacing navigation stack behavior with manual Activity finishing.
- Prefer lifecycle-aware callback registration.
- Keep callback enablement tied to the real UI state.

Back callbacks should express whether the current screen can handle back at this moment. They should not become a global replacement for every navigation path.

## Compatibility expectations

Do not implement separate page-level back logic for Android 13, Android 14, Android 15, and Android 16 unless there is no cleaner shared approach.

Use AndroidX compatibility APIs as the shared path. Add guarded platform-specific code only when required and only in a small, localized area.

## Manifest expectations

The Manifest may enable predictive back support, but the Manifest flag is not considered a full implementation.

Do not rely on Manifest-only changes to claim predictive back support. The actual Activity, Fragment, Dialog, and navigation behavior must still be reviewed.

Do not remove existing Activity attributes, exported flags, launch modes, permissions, process declarations, task affinities, or configuration-change settings unless the task explicitly requires it.

## Dependency expectations

Do not upgrade AndroidX, Kotlin, Gradle, AGP, Material, Fragment, Navigation, or AppCompat dependencies merely because a newer version exists.

Dependency changes must be justified by the current task and kept separate from unrelated UI behavior changes where practical.
