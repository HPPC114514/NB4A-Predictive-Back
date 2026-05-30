# Predictive Back Handoff

Last updated: 2026-05-30

## Current Phase

Post phase 4 animation prototype: the AndroidX-first predictive back code baseline is in place through main navigation, guarded editors, dialogs/import-adjacent transient Activities, VPN request handoff, tools pages, and settings-adjacent pages. Android 16 real-device verification confirmed the main behavior paths before this phase. The main shell now has a first AndroidX progress-driven animation prototype for drawer close, non-configuration fallback to configuration, and configuration-to-home approximation. This still needs Android 16 device verification.

## Task Goal

- Continue from `docs/predictive-back-progress.md`.
- Do not repeat phase 1 `MainActivity` work, phase 2 editor migration, or phase 3 transient Activity cleanup unless testing exposes a regression.
- Prefer verification, CI fixes, and targeted follow-up changes over more broad scanning.
- Current animation work is limited to the main shell only: open drawer back animation, non-configuration page back to configuration, and configuration page back to launcher/home.
- Keep configuration search behavior unchanged for now: first back may hide the IME, and the next back collapses search/focus.
- Keep using AndroidX `OnBackPressedDispatcher` / `OnBackPressedCallback` where code changes are needed.
- Do not add Android 16-only page-level callbacks.
- Do not change protocol parsing, VPN/service startup, database schema, plugin behavior, subscription update logic, or proxy runtime behavior.

## AGENTS.md Read

- `AGENTS.md`
- `app/AGENTS.md`
- `app/src/main/res/AGENTS.md`
- `app/src/main/java/moe/matsuri/nb4a/AGENTS.md`
- `app/src/main/java/io/nekohasekai/sagernet/ui/AGENTS.md`
- `app/src/main/java/io/nekohasekai/sagernet/ui/profile/AGENTS.md`

## Files Modified

### Phase 1 Files Preserved

- `app/src/main/java/io/nekohasekai/sagernet/ui/MainActivity.kt`
  - Keeps AndroidX callbacks for drawer-first and main fallback navigation.
  - Phase 4 adds AndroidX back progress/cancel handling for the main-shell animation prototype.
- `app/src/main/java/io/nekohasekai/sagernet/ui/ConfigurationFragment.kt`
  - Keeps search collapse through `ToolbarFragment.onBackPressed()`.
  - Phase 4 adds a non-mutating search-active helper so main-shell animation stays suppressed while search should consume back.

### Phase 2 Files Preserved

- `app/src/main/java/io/nekohasekai/sagernet/ui/profile/ProfileSettingsActivity.kt`
- `app/src/main/java/io/nekohasekai/sagernet/ui/profile/ConfigEditActivity.kt`
- `app/src/main/java/io/nekohasekai/sagernet/ui/profile/ChainSettingsActivity.kt`
- `app/src/main/java/moe/matsuri/nb4a/proxy/config/ConfigSettingActivity.kt`
- `app/src/main/java/io/nekohasekai/sagernet/ui/GroupSettingsActivity.kt`
- `app/src/main/java/io/nekohasekai/sagernet/ui/RouteSettingsActivity.kt`

### Phase 3 Files Modified

- `app/src/main/java/io/nekohasekai/sagernet/ui/AssetsActivity.kt`
  - Replaced the remaining legacy Activity `onBackPressed()` override with an AndroidX dispatcher callback that preserves the existing finish behavior.
- `app/src/main/java/io/nekohasekai/sagernet/ui/VpnRequestActivity.kt`
  - Added an AndroidX dispatcher callback that finishes the transient VPN request Activity when back is pressed.
  - The `StartService` contract, `VpnService.prepare()` path, permission-denied result, and service start behavior were not changed.
- `app/src/main/java/io/nekohasekai/sagernet/ui/SwitchActivity.kt`
  - Added an AndroidX dispatcher callback that preserves the dialog-style Activity finish behavior.
- `app/src/main/java/io/nekohasekai/sagernet/ui/ProfileSelectActivity.kt`
  - Added an AndroidX dispatcher callback that preserves the selector Activity finish behavior without setting a result.
- `app/src/main/java/io/nekohasekai/sagernet/ui/ScannerActivity.kt`
  - Added an AndroidX dispatcher callback that preserves scanner finish behavior.
  - Added toolbar up handling to finish, matching the close icon shown in the toolbar.
  - QR parsing, file import, camera permission handling, and profile import logic were not changed.
- `app/src/main/java/io/nekohasekai/sagernet/ui/AppManagerActivity.kt`
  - Added toolbar up handling to finish when AppCompat does not handle parent navigation.
  - Per-app proxy selection, import/export clipboard, and auto-select logic were not changed.
- `app/src/main/java/io/nekohasekai/sagernet/ui/StunActivity.kt`
  - Added toolbar up handling to finish, matching the back icon shown in the toolbar.
  - STUN test execution and error dialog behavior were not changed.
- `docs/predictive-back-handoff.md`
  - Updated this handoff for phase 3.
- `docs/predictive-back-progress.md`
  - Updated page/module progress statuses.

### Post Phase 3 Infrastructure Files Modified

- `buildSrc/src/main/kotlin/Helpers.kt`
  - `compileSdk` and `targetSdk` were raised from 35 to 36.
  - `minSdk` remains 21.
  - `buildToolsVersion` remains `35.0.1`.
- `.github/workflows/android-ci.yml`
  - Added a dedicated Android CI workflow for this fork.
  - Runs static predictive-back guardrails.
  - Builds and caches `app/libs/libcore.aar`.
  - Installs Android platform 36, build tools 35.0.1, and NDK 25.0.8775105.
  - Builds `app:assembleOssDebug`.
  - Uploads the `arm64-v8a` debug APK as `arm64-v8a-debug-apk`.
  - Fixed a guardrail false positive by using fixed-string grep for `OnBackInvokedCallback`, `OnBackAnimationCallback`, and `android.window`.
  - Replaced GitHub Actions `hashFiles()` cache expressions with shell-computed SHA-256 keys after the workflow template rejected the directory glob expression.

### Phase 4 Animation Prototype Files Modified

- `app/src/main/java/io/nekohasekai/sagernet/ui/MainActivity.kt`
  - Replaced lambda-only AndroidX back callbacks with explicit `OnBackPressedCallback` objects.
  - Uses `BackEventCompat.progress` through AndroidX `handleOnBackStarted`, `handleOnBackProgressed`, and `handleOnBackCancelled`.
  - Drawer back progress translates the active navigation drawer toward the start edge, restores it on cancel, and closes it without a second close animation on commit.
  - Main fallback and task-to-home paths apply a shared subtle fade plus scale to the main coordinator during gesture progress.
  - Configuration search active state suppresses the main-shell animation for that gesture so search and IME behavior remains unchanged.
- `app/src/main/java/io/nekohasekai/sagernet/ui/ConfigurationFragment.kt`
  - Added `hasActiveSearchBackState()` as a non-mutating state query used by `MainActivity`.
  - `onBackPressed()` now reuses the helper but preserves the existing search collapse behavior.
- `docs/predictive-back-handoff.md`
  - Updated for phase 4 implementation status, risks, and verification needs.
- `docs/predictive-back-progress.md`
  - Updated main-shell rows from planned to implemented/pending verification.

## Files Checked But Not Modified

- `app/src/main/AndroidManifest.xml`
  - Contains `android:enableOnBackInvokedCallback="true"`. This remains only a manifest opt-in and is not treated as a complete implementation.
- Dialog helpers:
  - `app/src/main/java/com/github/shadowsocks/plugin/fragment/AlertDialogFragment.kt`
  - `app/src/main/java/io/nekohasekai/sagernet/widget/QRCodeDialog.kt`
  - `app/src/main/java/moe/matsuri/nb4a/ui/Dialogs.kt`
  - Existing Dialog/AppCompat behavior remains the back path for dialogs. No direct platform callback was needed.
- Import flows:
  - `MainActivity.importSubscription()` / `MainActivity.importProfile()`
  - `BackupFragment` backup import/export dialogs and document pickers
  - `ScannerActivity` QR/file import
  - `AssetsActivity` asset file import
  - Existing confirmation dialogs, file pickers, and import processing were preserved.
- Tools/settings host pages:
  - `ToolsFragment.kt`
  - `SettingsFragment.kt`
  - `SettingsPreferenceFragment.kt`
  - `BackupFragment.kt`
  - `NetworkFragment.kt`
- Background shortcut Activities:
  - `QuickEnableShortcut.kt`
  - `QuickDisableShortcut.kt`
  - `QuickToggleShortcut.kt`
  - These extend plain `android.app.Activity` in the background process and immediately perform shortcut work, so they were documented rather than converted.
- `BlankActivity.kt`
  - Crash-log forwarding Activity immediately finishes and was left unchanged.

## Technical Direction

- Default path remains AndroidX `OnBackPressedDispatcher` / `OnBackPressedCallback`.
- No direct platform `OnBackInvokedCallback`, `OnBackAnimationCallback`, or Android 16-only page-level implementation was added.
- Dialogs and external system pickers are left to AppCompat/Dialog/activity-result behavior so a cancelled predictive gesture does not commit import, delete, reset, VPN, or service-start work early.
- Transient Activities that already finished on back now express that behavior through AndroidX where they are AppCompat/ComponentActivity based.
- Build and artifact verification should prefer GitHub Actions because this repository needs `libcore.aar`, Android SDK/NDK, Go/gomobile, and sing-box assets before Android Studio can compile cleanly.
- The app already depends on `androidx.activity:activity-ktx:1.10.1`, so the next phase can evaluate AndroidX predictive back progress callbacks before considering any platform API. Keep any progress-animation code centralized in `MainActivity` or a small main-shell helper rather than duplicating page logic.
- Prefer one shared in-app transition style for main-shell fallback paths. A subtle fade plus scale is safer than a right-exit slide because these destinations are not true stack pops and the current main fragments are replaced centrally.
- Phase 4 uses only AndroidX `BackEventCompat` progress callbacks. It does not add direct `OnBackInvokedCallback`, `OnBackAnimationCallback`, or `android.window` usage.
- The configuration-to-home animation remains an in-app approximation before `moveTaskToBack(true)`, not a true system launcher/home preview.

## Completed Back Behavior

- Drawer visible: back closes the drawer first. Phase 4 adds a progress-driven drawer translation prototype; Android 16 verification is pending.
- Configuration search active or expanded: Android 16 real-device verification showed first back hides the IME when typing, and the next back collapses search/focus. This is accepted behavior and should not be changed by default.
- Current main fragment is not `ConfigurationFragment`: back switches to `ConfigurationFragment`. Phase 4 adds a shared fade/scale progress animation; Android 16 verification is pending.
- Current main fragment is `ConfigurationFragment` with no active search: back calls `moveTaskToBack(true)`. Phase 4 adds a shared fade/scale progress approximation before moving the task to background; Android 16 verification is pending.
- Profile protocol editors: unsaved system back shows the existing save/discard/cancel dialog; clean system back falls through to the default Activity behavior.
- Chain profile editor: reorder/remove/add marks profile dirty through the shared helper so unsaved system back remains guarded.
- Custom JSON/config editor: unsaved system back shows the existing save/discard/cancel dialog; save still runs JSON formatting/validation first.
- Custom sing-box config profile: preference edits enable the inherited profile unsaved-change callback.
- Group settings: unsaved system back shows the existing save/discard/cancel dialog; clean system back falls through to default Activity behavior.
- Route settings: unsaved system back shows the existing save/discard/cancel dialog; apply/save still preserves the existing empty-route validation.
- Assets Activity: system back now uses AndroidX and preserves direct finish behavior.
- VPN request Activity: system back finishes the transient request Activity without altering VPN permission or service start result handling.
- Switch/profile-select/scanner transient Activities: system back now uses AndroidX and preserves finish behavior.
- Scanner, AppManager, and Stun toolbar up actions now finish consistently with their visible back/close icons.

## Android 16 Device Verification

Verified on a real Android 16 device with predictive back animations enabled:

- Drawer open then back: closes the drawer and stays on the current page; no predictive animation yet.
- Configuration search with input: first back hides the keyboard, second back collapses search/focus; keep this behavior.
- Non-configuration main page back: returns to the configuration page successfully; animation adaptation remains.
- Configuration page ordinary back: moves the task to launcher/home successfully; animation adaptation remains.
- Profile, group, route, and config editors with unsaved changes: existing save/discard/cancel dialog appears successfully.

Phase 4 implementation has not yet been verified on device. Re-test the same paths and confirm whether the new animations track the gesture, cancel cleanly, and leave no stuck alpha/scale/translation after returning to the app.

## Unfinished Pages And Risks

- Background shortcut Activities still extend plain `android.app.Activity`; converting them to AndroidX would be a broader lifecycle/runtime change in the background process and was left for a focused pass only if needed.
- `BlankActivity` still immediately finishes after optional crash-log forwarding and does not use AndroidX.
- Dialogs, import flows, VPN permission handoff, scanner permission flow, and toolbar up flows still need deeper gesture verification beyond the main paths listed above.
- No Android emulator predictive-back gesture verification has been performed yet; current verification evidence is from a real Android 16 device.
- Drawer predictive animation is medium risk: the prototype translates only the active `NavigationView`, while `DrawerLayout` still owns drawer state and scrim. Verify cancel/commit carefully for visual desync.
- Non-configuration-to-configuration animation is moderate risk: the prototype animates the outgoing main coordinator and swaps to configuration on commit. It does not pre-render the configuration destination during progress.
- Configuration-to-home animation is the hardest item: the prototype preserves `moveTaskToBack(true)`, so it is an in-app fade/scale approximation rather than true system home preview.
- Local Windows build attempts failed or were blocked by missing Android SDK/Go/libcore setup. This is an environment issue, not evidence of a Kotlin source regression.
- The new `Android CI` workflow should be used as the primary build gate. If it fails, inspect the failing job log first and keep fixes scoped to the workflow or build environment unless the log clearly identifies a source issue.

## Next Round Starting Point

Start with Android 16 verification of the phase 4 animation prototype:

- Run GitHub Actions `Android CI` on `main`.
  - Expected static checks: SDK level checks pass, predictive-back guardrails pass, docs exist.
  - Expected build output: artifact named `arm64-v8a-debug-apk`.
  - If `LibCore cache` or `Gradle cache` fails again, fix the workflow cache key or cache path only; do not change app behavior to work around CI.
- Re-test Android 16 real-device paths:
  - drawer open: gesture should visually move drawer toward closed, cancel should restore it, commit should close it without leaving translation;
  - configuration search with keyboard: keep the accepted first-back IME behavior and ensure main-shell fade/scale does not run for search collapse;
  - non-configuration page: gesture should fade/scale the main shell and commit to configuration;
  - configuration ordinary back: gesture should fade/scale the main shell and commit to launcher/home through `moveTaskToBack(true)`;
  - after returning from launcher/home, app alpha/scale should be reset.
- If true system home preview becomes mandatory, evaluate replacing the terminal `moveTaskToBack(true)` path separately and document the behavior tradeoff before changing it.

Only after the main-shell animation phase, consider whether background shortcut Activities or `BlankActivity` need an AndroidX wrapper.

## Do Not Repeat

- Do not redo the `MainActivity` drawer/configuration baseline unless a test exposes a regression.
- Do not redo the phase 2 editor dirty-state migration unless a test exposes a regression.
- Do not redo the phase 3 transient Activity conversion unless a test exposes a regression.
- Do not treat `android:enableOnBackInvokedCallback="true"` as completed predictive back support.
- Do not introduce Android 16-only callbacks as the default implementation.
- Do not add direct `OnBackInvokedCallback`, `OnBackAnimationCallback`, or `android.window` imports for the main-shell animation prototype unless AndroidX progress callbacks are proven insufficient and the reason is documented.
- Do not copy page-level back logic separately for Android 13, 14, 15, and 16.
- Do not remove unsaved-change, delete, import, VPN permission, plugin, or subscription confirmation flows.
- Do not modify protocol parsing, VPN/service startup, database schema, plugin communication, subscription update, or proxy runtime logic for predictive back cleanup.
- Do not try to make local Windows Android Studio builds work by committing generated `app/libs/libcore.aar`, generated assets, Gradle caches, or local SDK paths.

## Build And Check Results

- `git diff --check`: passed after phase 3 edits. Git reported line-ending normalization warnings (`LF will be replaced by CRLF the next time Git touches it`) for edited Kotlin files, but no whitespace errors.
- Legacy Activity back scan: `rg -n override.*onBackPressed app/src/main/java` now reports only `ConfigurationFragment.kt`, which is the existing fragment hook from phase 1, not an Activity `onBackPressed()` override.
- Platform back API scan:
  - `OnBackInvoked`: only the existing Manifest `android:enableOnBackInvokedCallback="true"` opt-in remains.
  - `OnBackAnimationCallback`: no matches.
  - `android.window`: no matches.
- `gradlew.bat :app:assembleDebug`: failed before Gradle started because `JAVA_HOME` was not set and `java` was not on `PATH`.
- `set JAVA_HOME=C:/Progra~1/Android/ANDROI~1/jbr && gradlew.bat :app:assembleDebug`: Gradle wrapper download required network access; after approval, Gradle started but failed before app compilation with `Could not move temporary workspace ... C:/Users/hppc_/.gradle/caches/8.10.2/transforms/... to immutable location`.
- `set GRADLE_USER_HOME=%CD%/.gradle/home && gradlew.bat :app:assembleDebug`: Gradle 8.10.2 downloaded into the workspace cache and started a daemon, then stayed silent in configuration/dependency resolution for several minutes. The stuck build command tree was terminated. This did not produce Kotlin or Android compile diagnostics.
- `gradlew.bat --stop` with the workspace Gradle home also stopped at `Stopping Daemon(s)` and did not exit promptly.
- No feature or source behavior was removed to work around build environment failures.
- SDK 36 update:
  - `compileSdk = 36`.
  - `targetSdk = 36`.
  - `minSdk = 21`.
- `Android CI` workflow status history:
  - Initial static guardrail had a false positive on `android:windowNoTitle` because `android.window` was treated as a regex. Fixed by using `grep -F`.
  - Initial cache key used `hashFiles('.github/workflows/*', 'buildScript/**', 'libcore/**')` and failed workflow template validation. Fixed by computing cache keys in shell and passing them through step outputs.
  - After the cache-key fix, the workflow needs to be run again on GitHub to confirm `libcore` and APK artifact generation.
- Android 16 real-device verification on 2026-05-30:
  - Drawer close, configuration search, non-configuration fallback, configuration-to-home, and dirty editor guard behavior were tested.
  - Behavior passed for the tested paths.
  - Missing animation remains for drawer close, non-configuration fallback, and configuration-to-home.
- 2026-05-30 docs-only update:
  - `git diff --check`: passed.
  - Static docs inspection found no trailing whitespace.
  - Predictive-back keyword scan found only documented references; no app source files were changed.
- 2026-05-30 phase 4 implementation checks:
  - `git diff --check -- app/src/main/java/io/nekohasekai/sagernet/ui/MainActivity.kt app/src/main/java/io/nekohasekai/sagernet/ui/ConfigurationFragment.kt docs/predictive-back-handoff.md docs/predictive-back-progress.md`: passed.
  - `git grep -n -F -e OnBackInvokedCallback -e OnBackAnimationCallback -e android.window -- app/src/main/java app/src/main/res`: no matches.
  - `git grep -n 'override fun onBackPressed' -- app/src/main/java`: only `ConfigurationFragment.kt`, the existing fragment hook.
  - Trailing-whitespace scan on edited code/docs files found no matches.
  - `gradlew.bat --version`: not run successfully because `JAVA_HOME` was not set and `java` was not on `PATH`.
  - Android 16 device verification for the new animations is still pending.
