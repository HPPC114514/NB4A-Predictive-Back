# AGENTS.md

This file applies to NB4A-specific code under `moe.matsuri.nb4a`.

Continue following the repository-level and app-level `AGENTS.md` files.

## Area focus

This area may include NB4A-specific proxy, configuration, or UI extensions. Treat predictive back work here as UI and navigation work unless the task explicitly requires changes to protocol or runtime behavior.

## Back behavior

Use AndroidX back handling as the default approach.

Do not add Android 16-only back behavior to NB4A-specific pages. Prefer shared lifecycle-aware back behavior that remains compatible across Android versions.

When changing a settings or configuration screen, preserve existing validation, save, discard, and import behavior.

## Protocol and runtime safety

Do not modify proxy protocol behavior, generated configuration, runtime service behavior, plugin behavior, or storage format as part of predictive back work unless the task explicitly requires it.

Keep UI navigation changes separate from protocol logic changes.

## Upstream and fork maintainability

Avoid unnecessary file moves, package renaming, broad formatting, or large rewrites. Small targeted changes are preferred because this fork may need to keep tracking upstream changes.
