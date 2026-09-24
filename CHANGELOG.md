# Changelog

Published Sileo builds (`com.0x17.persona.rootless` / `.roothide`). Newest first.

## [Unreleased]

## [2.3.0] - 2026-09-24

### Added
- "Uninstall Apps after Reset" toggle in Global Config (default off) — when on, reset ends with a final step that uninstalls the targeted apps
- App Targeting: "Not Installed" section listing selected apps that were removed from the device — tap to unselect them (they were invisible before and stayed ticked forever)
- Live step progress in loading alerts ("Step X/Y — label") for reset / backup-and-reset / backup; failure alerts now name the exact step that failed
- RRS backup/restore run as an explicit overall step plan with progress (backup: 7 steps incl. ZIP integrity verification; restore: 8 steps incl. service restart)

### Changed
- RRS list filters "Restored" / "Not restored" now mean restored today / not restored today; selection toolbar simplified (per-row restore & re-backup removed from toolbar)
- Device quick-select defaults are iPhone-only now (iPads excluded)

### Fixed
- Full-dump keychain restore was broken — a SQL dump (CREATE TABLE + INSERTs) can never be applied to the live DB. Restore now builds a fresh DB from the dump and swaps it in after restarting `securityd` (live DB kept as `.bak-persona`)
- Wipe of uninstalled targeted apps: orphaned App Group containers + app bundles are now found via container metadata / Info.plist scans and deleted (SpringBoard usually removes data containers itself, but groups and bundles could survive)
- Stacked alerts on back-to-back operations — result alerts no longer pile up 2–3 deep; the previous alert is dismissed before the next one presents
- RRS selection state going stale after filtering (filtered list is now computed live instead of a stored copy)

## [2.2.0] - 2026-09-23

### Added
- App Store account management via URL scheme: `persona://login-itunes` (Apple ID, 2FA-aware), `persona://itunes-status`, `persona://logout-itunes`; new `Itunes/` subproject (`PersonaItunes` binary) + AMS sign-in hooks + "Login App Store" row in Settings
- `logout-itunes` now also wipes the system account store: SIGKILLs `accountsd`/`amsaccountsd`, then clears `/var/mobile/Library/Accounts` after a successful store sign-out
- Helper daemon subproject (`com.0x17.persona.helper`): keychain backup/restore CLI + socket daemon; SQL-based selective or full keychain backup (works around iOS 26 root keychain-scope limitation)
- Timezone override without hooks: re-points the `/private/var/db/timezone/localtime` symlink, with `PersonaTZGuard` in the Helper daemon re-applying it every 30s so it survives chronod/cfprefsd restarts
- Safari browser spoofing: custom page UA + `navigator.platform`, WebRTC leak block/override (main frame)
- Quick actions on the More screen: switch identity, rotate/toggle proxy, backup, restore
- Device quick-select defaults (iPhone X+ / iOS 16+) + Restore Default button

### Changed
- Single app-scope semantics: ticked app = targeted app; drives injection and backup/reset/wipe/clear alike. The "all apps"/exclusion mode is removed
- Settings consolidated into Global Config (General/Config/ServiceStatus screens removed)
- No apps ticked → data ops stop with an explicit error instead of silently doing nothing

### Fixed
- App Store downloads failing silently under `override_udid` — purchase window where MobileGestalt serves the real UDID to `appstored`/`storekitd` only during the buy flow
- Sysctl hook ↔ `SFDeviceData` bootstrap recursion (Safari killed on every page load)
- Reset scope: explicit app targeting now wins over the "Wipe All Apps" toggle
