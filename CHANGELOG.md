# Changelog

Published Sileo builds (`com.0x17.persona.rootless` / `.roothide`). Newest first.

## [Unreleased]

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
