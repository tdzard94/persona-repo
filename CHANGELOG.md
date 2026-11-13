# Changelog

Published Sileo builds (`com.0x17.persona.rootless` / `.roothide`). Newest first.

## [2.8.1] - 2026-11-13

### Fixed
- Crash on opening App Targeting on iOS 18.4.1 (Dopamine): app icons were rendered through IconServices/CoreImage, which took its OpenGL ES path in Persona's process and called a NULL function. Icons are now read straight from each app's bundle; the private icon API is only used on iOS 26+

## [2.8.0] - 2026-11-13

### Added
- `persona://system-enable` / `persona://system-disable`: standalone on/off for Override System (fake serial / UDID / MAC), the same as `persona://identity?overrideSystem=true|false` without touching any other setting
- Sysctl spoofing now covers the numeric-MIB `sysctl()` path too, not just `sysctlbyname()`: fingerprint code that queries `sysctl({CTL_KERN, …})` / `{CTL_HW, HW_PRODUCT}` directly used to read the real kernel values. Added the full `CTL_KERN` set (`KERN_OSTYPE`/`OSRELEASE`/`OSVERSION`/`VERSION`/`HOSTNAME`/`MAXVNODES`/`BOOTTIME`) and `HW_PRODUCT`, plus the string keys `kern.osproductversion`, `kern.version`, `kern.bootsessionuuid`, `kern.maxvnodes`
- Generate Device Name switch (Identity → Device): off reports a generic "iPhone" for the device name, `uname` nodename and `kern.hostname` instead of the fake name. Also settable via `persona://identity?generateDeviceName=true|false`. `kern.osproductversion` now follows the Override OS Version switch

## [2.7.0] - 2026-11-07

### Added
- `persona://identity?overrideSystem=true|false` turns Override System (fake serial / UDID / MAC) on or off, as the Identity screen's switch does but without its confirmation; turning it on with no system identity saved makes one
- `persona://device-selection?hardware=…&firmware=…` (each repeatable) replaces the devices and iOS versions that reset picks from: hardware by model name (`iPhone 14 Pro`, every variant of it) or identifier (`iPhone15,2`), firmware by version (`17.4.1`), looked up in the server's device catalog (needs a login and a network). A key left out keeps its picks, a name the catalog does not know changes nothing, and an iOS version that no picked device runs is left out (as Quick Select does)

## [2.6.1] - 2026-11-05

### Fixed
- IDFA never changed on Reset / Backup & Reset / Switch Identity: the app sent the current IDFA to the device-generate API, which echoed it back unchanged. The old IDFA is no longer sent, so every device change now gets a fresh IDFA (idfv/uuid already rotated)

## [2.6.0] - 2026-11-05

### Added
- RRS backups can be renamed (tap a backup → Rename): updates the zip file name, the root folder inside the zip (via `zipnote`, no recompression), `metadata.json`, the database row and the Restore Queue — works on a temp copy and only swaps it in once verified

### Fixed
- IDFA spoofing had no effect in most apps: ad SDKs check App Tracking Transparency first and never read the IDFA when tracking is not authorized. With Fake IDFA on, `ATTrackingManager.trackingAuthorizationStatus` / `requestTrackingAuthorization` now report "authorized" (no system prompt) and `isAdvertisingTrackingEnabled` returns YES
- Crash when tapping Rotate Proxy in More (also on timed / `persona://rotate-proxy` rotation): the "proxy rotated" notification was posted from a background thread and rebuilt the UI off-main
- Global Config: the keyboard covered the tracking URL field — the list now scrolls the field above the keyboard; Done or a swipe dismisses it

## [2.5.0] - 2026-10-28

### Changed
- Backup names: no spaces, app name + compact timestamp (`Tinder_20260924_211532`, multi-app joined with `+`); manual names are sanitized too; same-name collisions get a `-2`/`-3` suffix
- Production writes nothing to `/var/tmp`: injection markers, the MobileGestalt self-verify (`dlopen` + `MGCopyAnswer` in the daemon ctor), the storekitd AMS class probe, the SpringBoard markers/kill log and the LaunchServices API dumps are gated on a sentinel — `touch /var/jb/etc/persona-diag` turns them back on for debugging, no rebuild needed

### Fixed
- Quick back-to-back backups colliding on the same timestamp name and failing to create a new file
- App Store downloads silently failing with Override-System UDID on: the purchase window (real UDID for `appstored`/`storekitd` during a purchase, real `guid=` on `buyProduct`) had been disabled, so the store backend answered HTTP 500 again
- Upgrading from a 2.3.0 install silently killed Override System: the old `system_device.json` (encrypted `info` blob, no udid/ecid/MACs) failed to decode, so toggles answered "No system data to update", the reset flow dropped its system step and the tweak served nothing. Fields are now optional and a legacy file is completed locally on first launch.
- Toggling a setting by tapping the ROW (not the switch) flipped the switch visually but never saved: 8 rows in Global Config and the App Store "Fake Device ID" row now persist on row taps too.

### Security
- Helper daemon socket is root-only (`0600` + `getpeereid()` gate): before, any app on the device could connect to `/var/run/persona-helper.sock` and drive the root daemon — wipe every app's keychain, clear SQL rows, sign the store account out
- Bundle ids sent to the Helper (and in its local SQL fallback) are validated against `[A-Za-z0-9._-]` before reaching the `DELETE FROM genp/inet/cert/keys` statements, closing an SQL injection that ran as root against the system keychain

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
