# Predictive Back Handoff

Last updated: 2026-05-24

## Current Phase

Phase 3: cover dialogs, import flows, VPN permission handoff, temporary Activities, tools pages, and settings-related pages after the phase 1 main-shell baseline and phase 2 guarded editor migration.

## Task Goal

- Continue from `docs/predictive-back-progress.md`.
- Do not repeat phase 1 `MainActivity` work or phase 2 profile/config/group/route editor migration.
- Review Dialog, import, VPN permission, temporary Activity, tools, and settings return behavior.
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
- `app/src/main/java/io/nekohasekai/sagernet/ui/ConfigurationFragment.kt`
  - Keeps search collapse through `ToolbarFragment.onBackPressed()`.

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

## Completed Back Behavior

- Drawer visible: back closes the drawer first.
- Configuration search active or expanded: back collapses search and clears focus before leaving the configuration page.
- Current main fragment is not `ConfigurationFragment`: back switches to `ConfigurationFragment`.
- Current main fragment is `ConfigurationFragment` with no active search: back calls `moveTaskToBack(true)`.
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

## Unfinished Pages And Risks

- Background shortcut Activities still extend plain `android.app.Activity`; converting them to AndroidX would be a broader lifecycle/runtime change in the background process and was left for a focused pass only if needed.
- `BlankActivity` still immediately finishes after optional crash-log forwarding and does not use AndroidX.
- Dialogs, import flows, VPN permission handoff, scanner permission flow, and toolbar up flows were not gesture-tested on a device/emulator.
- No Android emulator predictive-back gesture verification has been performed yet.
- Debug build verification did not reach Kotlin/Android compilation because Gradle setup/cache handling failed before app tasks started.

## Next Round Starting Point

Start with verification rather than more migration:

- Retry a debug build from a clean terminal/Gradle state. This workspace can use Android Studio's JBR at `C:\Progra~1\Android\ANDROI~1\jbr`; the failed phase 3 attempts are listed below.
- Device/emulator test phase 1, 2, and 3 paths:
  - main drawer/search/navigation;
  - dirty and clean profile/config/group/route editors;
  - backup import confirm/cancel/progress dialog;
  - asset import and direct asset back;
  - scanner camera permission, file import, toolbar up, and system back;
  - VPN permission accept/deny/back;
  - switch/profile-select back without result;
  - AppManager and Stun toolbar up/system back.

Only after verification, consider whether background shortcut Activities or `BlankActivity` need an AndroidX wrapper.

## Do Not Repeat

- Do not redo the `MainActivity` drawer/configuration baseline unless a test exposes a regression.
- Do not redo the phase 2 editor dirty-state migration unless a test exposes a regression.
- Do not treat `android:enableOnBackInvokedCallback="true"` as completed predictive back support.
- Do not introduce Android 16-only callbacks as the default implementation.
- Do not copy page-level back logic separately for Android 13, 14, 15, and 16.
- Do not remove unsaved-change, delete, import, VPN permission, plugin, or subscription confirmation flows.
- Do not modify protocol parsing, VPN/service startup, database schema, plugin communication, subscription update, or proxy runtime logic for predictive back cleanup.

## Build And Check Results

- `git diff --check`: passed after phase 3 edits. Git reported line-ending normalization warnings (`LF will be replaced by CRLF the next time Git touches it`) for edited Kotlin files, but no whitespace errors.
- Legacy Activity back scan: `rg -n override.*onBackPressed app\src\main\java` now reports only `ConfigurationFragment.kt`, which is the existing fragment hook from phase 1, not an Activity `onBackPressed()` override.
- Platform back API scan:
  - `OnBackInvoked`: only the existing Manifest `android:enableOnBackInvokedCallback="true"` opt-in remains.
  - `OnBackAnimationCallback`: no matches.
  - `android.window`: no matches.
- `gradlew.bat :app:assembleDebug`: failed before Gradle started because `JAVA_HOME` was not set and `java` was not on `PATH`.
- `set JAVA_HOME=C:\Progra~1\Android\ANDROI~1\jbr && gradlew.bat :app:assembleDebug`: Gradle wrapper download required network access; after approval, Gradle started but failed before app compilation with `Could not move temporary workspace ... C:\Users\hppc_\.gradle\caches\8.10.2\transforms\... to immutable location`.
- `set GRADLE_USER_HOME=%CD%\.gradle\home && gradlew.bat :app:assembleDebug`: Gradle 8.10.2 downloaded into the workspace cache and started a daemon, then stayed silent in configuration/dependency resolution for several minutes. The stuck build command tree was terminated. This did not produce Kotlin or Android compile diagnostics.
- `gradlew.bat --stop` with the workspace Gradle home also stopped at `Stopping Daemon(s)` and did not exit promptly.
- No feature or source behavior was removed to work around build environment failures.
