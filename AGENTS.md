# AGENTS.md

This repository is a fork of NB4A / nekoboxforandroid. The purpose of this fork is to improve predictive back behavior while keeping the upstream application stable and easy to merge.

The project should not be treated as an Android 16-only experiment. Predictive back work should prefer AndroidX compatibility APIs and lifecycle-aware behavior over platform-version-specific implementations.

## Scope

These instructions apply to the entire repository unless a deeper `AGENTS.md` provides more specific guidance.

Deeper `AGENTS.md` files inherit these rules. A deeper file may add stricter rules for its area, but it should not weaken the repository-level safety rules.

## Project direction

- Use AndroidX back handling as the default approach for predictive back work.
- Prefer `OnBackPressedDispatcher` and `OnBackPressedCallback` when changing Activity or Fragment back behavior.
- Avoid direct Android platform back APIs unless there is a clearly documented reason.
- Do not add Android 16-only back handling unless an AndroidX-compatible approach is not sufficient.
- Keep Android 13-15 behavior in mind, but do not duplicate page-level logic for each Android version.
- Keep traditional back behavior working on Android versions that do not support platform predictive back, using AndroidX compatibility paths where possible.

## Upstream compatibility

This is a fork of an upstream Android application. Keep changes small, understandable, and easy to compare with upstream.

Avoid unrelated formatting, package renaming, file moves, resource renaming, dependency churn, or large rewrites. Do not change protocol, VPN, database, plugin, or networking behavior unless the task explicitly requires it.

When a change is related to predictive back, limit the change to the relevant navigation, Activity, Fragment, Dialog, or UI state handling code.

## Back behavior principles

Back handling must preserve user-visible behavior before improving animation behavior.

A page should not exit, discard input, close a flow, or finish an Activity earlier than the previous behavior allowed. Predictive back gestures may be started, cancelled, or completed, so state changes must happen at the correct point in the existing back flow.

Give priority to existing UI state before leaving the current screen. Examples include open drawers, active search UI, dialogs, bottom sheets, unsaved forms, nested navigation stacks, and permission flows.

Do not bypass FragmentManager, NavController, AppCompat, or existing lifecycle-aware components just to trigger a system animation.

## Documentation expectations

When changing predictive back behavior, update the project progress documentation if it exists. If a relevant progress file does not exist yet, keep the code change self-contained and mention what still needs to be recorded in the final summary or commit description.

## Long-term project memory

Predictive back work must use repository-tracked handoff and progress documents.

Before changing predictive back behavior, read these files if they exist:

- `docs/predictive-back-handoff.md`
- `docs/predictive-back-progress.md`

After changing predictive back behavior, create or update both files:

- `docs/predictive-back-handoff.md`
- `docs/predictive-back-progress.md`

The handoff document records what the next agent needs to know before continuing. It should include the current phase, files changed, files inspected but not changed, technical direction, completed behavior, unresolved risks, build or verification result, and the next recommended task.

The progress document records page-level or module-level status. It should include the area, file or screen, AndroidX back handling status, risk, next step, and last update.

Do not rely only on the final chat summary for project memory. Important predictive back state must be written into repository files.

## Safety rules

- Do not remove confirmation dialogs that protect unsaved user data.
- Do not remove existing permission, VPN, import, export, or plugin flows to simplify back handling.
- Do not make broad dependency upgrades only to solve a local back behavior issue.
- Do not use Android 16-specific APIs as the default solution.
- Do not introduce behavior that only works on one Android version unless guarded and documented.
- Do not assume a back animation is correct just because the app builds.
