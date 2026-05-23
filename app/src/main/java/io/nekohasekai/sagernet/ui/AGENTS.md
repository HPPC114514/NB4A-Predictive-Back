# AGENTS.md

This file applies to UI code under `io.nekohasekai.sagernet.ui`.

Continue following the repository-level and app-level `AGENTS.md` files. This file adds rules for Activity, Fragment, Dialog, navigation, and screen-level back behavior.

## UI back behavior

Back handling in this area must remain user-first. A predictive back animation is only correct if the underlying back behavior is also correct.

Before changing a back path, identify what the current screen should do when back is requested. Common cases include closing an open drawer, closing search UI, dismissing a dialog, popping a Fragment stack, confirming unsaved changes, cancelling a temporary flow, or finishing the current Activity.

Do not make back handling skip an intermediate UI state. If the old behavior closed a drawer before leaving the screen, the new behavior should still close the drawer before leaving the screen.

## AndroidX-first rule

Use AndroidX back APIs as the normal implementation path.

Do not add direct Android platform predictive back callbacks in individual screens unless AndroidX cannot express the required behavior. If such code is required, keep it localized and document why.

Do not add one-off version branches to each Activity or Fragment. Shared behavior should be centralized or expressed through AndroidX lifecycle-aware callbacks.

## State-sensitive callbacks

Back callbacks should be enabled only when the relevant UI state is active.

Examples:

- A search callback should be enabled while search UI is expanded.
- A drawer callback should be enabled while the drawer is open.
- An unsaved-edit callback should be enabled while there are unsaved changes.
- A nested-navigation callback should reflect whether the nested stack can actually go back.

Avoid always-enabled callbacks that intercept every back event without checking the current state.

## Navigation safety

Do not bypass existing FragmentManager, NavController, toolbar navigation, or AppCompat behavior just to force a result.

Do not replace structured navigation with scattered `finish()` calls.

Do not clear back stacks, reset screens, or recreate Activities unless the existing feature already required that behavior.

## Dialogs and temporary UI

Dialogs, confirmation prompts, bottom sheets, temporary panels, and permission-related screens need careful handling.

A back gesture that is cancelled should not commit a destructive action. Destructive or state-changing behavior should happen only when the existing back action is actually confirmed.

Do not remove or weaken confirmation prompts for profile editing, configuration changes, import flows, or app exit behavior.

## Testing awareness

When modifying a screen in this package, note which UI state was considered. At minimum, distinguish between ordinary back, nested back, temporary UI back, and unsaved-state back.

If predictive back behavior is changed in this package, update `docs/predictive-back-handoff.md` and `docs/predictive-back-progress.md`.

If either file does not exist, create it with a minimal useful structure instead of leaving the state only in the final response.
