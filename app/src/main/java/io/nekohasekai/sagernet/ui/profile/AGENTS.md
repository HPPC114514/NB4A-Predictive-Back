# AGENTS.md

This file applies to profile and proxy configuration editing UI under `io.nekohasekai.sagernet.ui.profile`.

Continue following all parent `AGENTS.md` files. This area has stricter rules because back behavior can affect user configuration data.

## Data safety comes first

Predictive back work in this directory must not cause profile edits, proxy settings, imported configuration, or unsaved form input to be lost silently.

If a screen previously asked the user to save, discard, confirm, or cancel, that behavior must remain intact.

Do not convert a guarded exit into a direct Activity finish. Do not dismiss a configuration editor before its existing validation, save, or discard logic has run.

## AndroidX-first rule

Use AndroidX back handling as the default approach.

Do not add Android 16-only back handling to profile editors. If platform-specific behavior is unavoidable, isolate it outside the editor-specific business logic and document why AndroidX was not enough.

## Edit screen behavior

Before changing back behavior in this directory, identify whether the screen has any of the following:

- Unsaved text input.
- Generated or imported configuration content.
- Validation errors.
- Save/apply actions.
- Discard confirmation.
- Nested editors.
- Dialogs or pickers.
- Protocol-specific fields.
- Shared base classes or reusable editor components.

Back handling must respect those states.

## Protocol-specific pages

Profile pages may represent different proxy protocols or configuration types. Do not assume all editors can share identical save, discard, or validation behavior.

Avoid broad refactors across all protocol editors unless the shared behavior is already clearly centralized.

When changing a shared base editor, consider every protocol editor that inherits or depends on it.

## Gesture cancellation safety

Predictive back gestures can be started and cancelled. Do not perform destructive actions merely because a gesture begins.

State changes such as clearing fields, saving profiles, discarding edits, or finishing the editor should happen only when the existing confirmed back action actually completes.

## Out-of-scope changes

Do not modify protocol parsing, serialization, database schema, subscription behavior, or proxy runtime behavior as part of profile editor back handling unless the task explicitly requires it.

Keep changes focused on UI state, navigation, and lifecycle-aware back handling.
