# VELTRIX — Android Dynamic Island

VELTRIX is a native Kotlin + Jetpack Compose Android project. It is designed around an optional floating Island, individually switchable modules, a dark premium control center, and GitHub-based release checking.

## Build configuration

| Component | Version |
|---|---:|
| Android Gradle Plugin | 8.11.1 |
| Gradle | 8.13 |
| Kotlin | 2.2.20 |
| Compose BOM | 2026.09.00 |
| Compile SDK | 36 |
| Target SDK | 35 |
| Minimum SDK | 26 (Android 8.0) |
| Java / JVM target | 17 |

The Kotlin/Gradle/AGP versions are pinned to a compatible combination. The Android build workflow installs `platforms;android-36` and `build-tools;36.0.0`; it deliberately does **not** request the obsolete SDK package named `tools`.

## Included in this source package

- Custom VELTRIX adaptive launcher icon (vector resources).
- Dark control-center UI with animated custom ON/OFF switches.
- Global Island master switch and independent module switches saved in local preferences.
- Native `TYPE_APPLICATION_OVERLAY` Island service with screen-width clamping, status-bar/cutout-aware top offset, collapsed/expanded states, and long-press to stop.
- Foreground-service notification with a stop action.
- Notification Listener integration for Android media sessions, basic media controls, and temporary notification previews.
- Battery / charging state shown by the overlay.
- LRC import (including multiple timestamps on one LRC line), persisted lyric source, and line-synchronized lyrics using the active Android media-session position.
- Prayer-time lookup using AlAdhan calculation method 20 (KEMENAG), cached timetable display, exact-alarm permission flow, daily refresh scheduling, notification Accept action, and optional user-selected adhan audio file.
- GitHub Releases checker showing tag, release notes, APK file name/size, publish date and publish time converted to WIB.
- GitHub Actions workflows for debug APK artifacts and signed release APK publication.

## Important capability boundaries

Some modules are shown in the Module Manager as independent switches, with their integration state clearly marked. Not every entry is a complete Android integration in this initial code drop. Weather and live score need data-provider integration; navigation, screen activity, system volume and thermal details are restricted by the signals Android exposes; timer, clipboard, notes, QR and extra utility surfaces need their dedicated screens/handlers completed before they should be treated as production-ready.

**Lyrics:** Spotify's public Web API does not provide a lyrics endpoint. VELTRIX therefore does not scrape Spotify's private endpoints or invent lyric data. This build supports timestamped `.lrc` import and synchronizes the active line to the Android media session. Automatic catalogue lyrics require a properly licensed provider and its credentials.

**Prayer times:** the app requests prayer timings from AlAdhan using calculation method `20` (KEMENAG). Treat the result as a calculated schedule and compare it with local mosque / official regional schedules if local adjustments are used. Location is only used after the user grants permission. Exact reminder delivery depends on Android alarm access and device power-management behavior; Android may delay inexact alarms.

**Overlay placement:** Android controls application-overlay visibility and may move/hide overlays to protect system UI. VELTRIX constrains its width to the display and offsets the pill below the status-bar/display-cutout safe area, but an OEM-specific notch or camera cutout may still require a device-specific offset adjustment.

## Build APK from GitHub on a phone

1. Create a new GitHub repository named `VELTRIX` (or any name you prefer).
2. Upload the contents of this folder to the repository root. Do not upload any `.jks` signing key.
3. Open **Actions** and enable GitHub Actions for the repository.
4. Run **VELTRIX Android CI** (or push a commit). Download the `VELTRIX-debug-apk` artifact from the completed workflow run.
5. Debug APK uses the application ID suffix `.debug`, so it can be installed separately from a signed release build.

## Signed releases and in-app update checks

The release workflow publishes a signed APK to GitHub Releases when you push a tag such as `v1.0.1`.

Create these repository Actions secrets:

- `VELTRIX_KEYSTORE_BASE64`
- `VELTRIX_KEYSTORE_PASSWORD`
- `VELTRIX_KEY_ALIAS`
- `VELTRIX_KEY_PASSWORD`

The signing keystore must remain private and must be the same for every update. Never regenerate it after publishing the first release, or Android will reject updates as a signature mismatch. See `docs/SIGNING_AND_RELEASES.md`.

In VELTRIX **Settings → Update Center**, set the repository as `owner/repository`. The About screen reads the public GitHub latest-release API and `update/update.json`. Public repository access is anonymous and subject to GitHub API rate limits. `published_at` is converted from UTC to `Asia/Jakarta` (WIB).

The app can download the APK through Android Download Manager. Android still requires the user to open/confirm an APK installation; a third-party app cannot silently replace itself under normal Android permissions.

## First-run permissions

- Display over other apps: required for the floating Island.
- Notification access: required for media sessions and notification preview.
- Post notifications: required for prayer reminders on Android 13+ (grant it in Android App settings if alerts do not appear).
- Location: optional; required only to fetch prayer times from device coordinates.
- Alarms & reminders: grant exact-alarm access if you expect tight prayer reminder timing. Without it, the app uses an inexact alarm fallback.

## Updating the development status

Edit `update/update.json` on the default branch:

- `development` → yellow status.
- `none` → red status when there is no newer release.

A newer published GitHub Release than the installed version overrides the status file and is shown in green.
