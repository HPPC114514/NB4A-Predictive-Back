# AGENTS.md

This file applies to Android resources under `app/src/main/res`.

Continue following the repository-level and app-level `AGENTS.md` files.

## Resource safety

Resource changes can affect the entire app. Avoid broad theme, style, animation, transition, color, dimension, or string changes unless the task explicitly requires them.

Do not change global themes to solve a single screen's back behavior issue.

Do not remove existing resource qualifiers, localized strings, launcher resources, backup resources, shortcuts, network security configuration, or permission-related XML unless directly required.

## Predictive back related resources

If a resource change is related to predictive back, keep it narrowly scoped and explain which screen or transition it affects.

Do not use resource changes to hide incorrect navigation behavior. The underlying Activity, Fragment, Dialog, or UI state handling must still be correct.

## Compatibility

Resource changes should not break older Android versions, TV / Leanback behavior, right-to-left layout behavior, or existing configuration-change handling.

Avoid adding resources that only work on a single Android version unless a compatible fallback exists.
