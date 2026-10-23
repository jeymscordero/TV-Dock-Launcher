# TV Dock

**A clean, aurora-themed home screen (launcher) for Android TV and Google TV.**
Big focus-friendly tiles, a live clock header, a built-in Inputs switcher, and a tile list you fully control with the remote. No account, no ads, no internet access.

> Design inspired by the [CleanYourTV](https://github.com/Gathin23/cleanyourtv) home screen (aurora look). TV Dock is an independent project and contains none of its code.

<p align="center"><img width="1280" height="720" alt="Snapshot" src="https://github.com/user-attachments/assets/b150b1b8-06dd-42a8-a2da-7e56ae3e416e" /></p>


---

## Features

- **Aurora home screen:** a deep-blue and teal wallpaper drawn in code, so there are no heavy image files.
- **Live header:** greeting, large clock, date, and quick pills for **ScreenShare**, **Inputs**, **Wi-Fi** and **Settings**.
- **Inputs switcher:** lists the TV's own inputs (HDMI, AV and others) by the names set in the TV, and marks the ones with a connected device.
- **Dock and grid:** the first four tiles sit in a rounded dock band, the rest in a grid that scrolls up when you press Down.
- **Floating highlight:** one glass highlight glides between tiles, and the focused app name appears on a large shelf above.
- **Real app art:** tiles use each app's own TV banner, or its icon when there is no banner.
- **Fully customizable tiles:** add any installed app, reorder, rename, recolour or remove, all from the remote. Changes are saved on the TV.
- **First-run defaults:** on the first launch it shows popular apps that are actually installed on your TV, and you decide the rest.
- **Set as Home without a computer:** the welcome screen asks Android to make TV Dock your Home screen.
- **Scales to any screen:** layout units are based on screen width, so 720p, 1080p and 4K TVs look the same.
- **No permissions requested, no network access, no tracking.**

## Compatibility

| | |
|---|---|
| Android version | Android 5.0 (API 21) and newer. Developed on Android 14 |
| Devices | Android TV and Google TV devices, TV boxes and sticks |
| Input | Remote with D-pad (arrow keys + OK/Select). No touchscreen needed |
| Developed on | Xiaomi Google TV (Android 14). **Other brands are untested, and reports are welcome** |

## Install

### Option A: copy the APK to the TV
1. Download `app-release.apk` (or the latest debug APK) from the [Releases](../../releases) page.
2. Put it on a USB drive, or send it with an app such as *Send Files to TV*.
3. Open it with a file manager on the TV and choose **Install**. If asked, allow that file manager to install unknown apps.

Installing a newer APK over an older one (same signing key) updates TV Dock and keeps your tiles.

### Option B: ADB
```bash
adb connect <TV-IP>:<PORT>
adb install -r app-release.apk
```
Enable **Developer options > Wireless debugging** (or USB debugging) on the TV first. The port changes whenever Wireless debugging is toggled or the TV restarts.

## Make it your Home screen

- **Easiest:** on first launch choose **Set TV Dock as my Home screen**. You can do it any time by holding OK on a tile and choosing **Set TV Dock as Home screen**.
- **Manual:** Settings > Apps > Default apps > Home app (the name varies by TV), or press Home and choose TV Dock when asked.
- **ADB (if your TV ignores the prompt):**
  ```bash
  adb shell cmd package set-home-activity com.james.dock/.MainActivity
  ```

The original launcher stays installed, and nothing is disabled or removed. To go back, choose the original launcher in the same settings screen, or uninstall TV Dock.

## Using it

| Action | How |
|---|---|
| Move around | D-pad arrows |
| Open an app | OK / Select |
| Open tile menu (move, edit, remove, add, reset) | Hold OK on a tile |
| Add an app | Select the dashed **+** tile, or **Add app** in the tile menu |
| Add an app that isn't listed | **Add app > Enter a package name…** |
| Switch HDMI/AV | **Inputs** pill at the top |
| Network / system settings | **Wi-Fi** and **Settings** pills |
| Screen sharing / cast | **ScreenShare** pill (shown only when the TV supports it) |

## Default tiles on first launch

TV Dock checks which of these are installed on your TV and shows only those. Nothing is added for apps you don't have:

Live TV · YouTube · Netflix · Prime Video · Disney+ · HBO Max · Hulu · Apple TV · Spotify · YouTube Music · Plex · Kodi · Bilibili · VLC · Browser · TV Manager (Xiaomi) · Play Store · Settings

Some package names differ by region or firmware, so an app can be missed on first launch. You can add it yourself with the **+** tile. To change the defaults for everyone, edit the `featured` list in `MainActivity.kt`:

```kotlin
Candidate("netflix", "Netflix", listOf("com.netflix.ninja")),
```

Each entry lists several package names. The first one installed on the TV is used.

## Build from source

Requirements: Android Studio, JDK 17, Android SDK 34.

```bash
git clone https://github.com/<your-user>/TvDock.git
cd TvDock
./gradlew assembleDebug        # app/build/outputs/apk/debug/app-debug.apk
./gradlew assembleRelease      # needs your own signing config
```

Before publishing a release build:
- Set your own permanent `applicationId` and `versionCode` / `versionName` in `app/build.gradle`.
- Create and back up your own signing key. Updates only install over each other when signed with the same key.
- Provide the banner image (`tv_dock_banner`, 320×180 px) under `res/drawable`.

### Project layout

```
app/src/main/
├── AndroidManifest.xml      # HOME + LEANBACK_LAUNCHER, package-visibility queries
├── java/com/james/dock/
│   └── MainActivity.kt      # the whole launcher UI and logic
└── res/drawable/
    ├── tv_dock_banner.*     # app banner / icon
    ├── ic_screenshare.xml
    ├── ic_input.xml
    ├── ic_wifi.xml
    └── ic_settings.xml
```

## How it works

- **No layout XML:** the UI is built in Kotlin with all sizes in reference units (1280 units = screen width).
- **Tiles:** stored as JSON in `SharedPreferences` (`dock_prefs`), so they survive restarts.
- **Inputs:** read from Android's `TvInputManager`. Selecting one asks the TV's live-TV app to open that input.
- **App list:** queried with `PackageManager`, using `<queries>` entries for Android 11+ package visibility. TV Dock does not request `QUERY_ALL_PACKAGES`.
- **Wallpaper:** a small gradient bitmap generated at start-up and stretched to the screen.

## Limitations and known issues

- **Inputs depends on the TV's firmware.** Some TVs list fewer inputs to third-party apps, and opening an input needs the TV's own Live TV app to handle it.
- **ScreenShare** opens the TV's cast or screen-sharing screen when one exists. If a TV has none, the pill is hidden.
- **Home prompt:** some TVs hard-wire their own launcher and ignore the Home request. Use the manual or ADB option above.
- Scrolling between dock and grid is instant, not animated, and the font is the system sans-serif.
- Mostly tested on a single TV model so far.

## Troubleshooting

| Problem | Try |
|---|---|
| `adb: device offline` / port changed | Toggle Wireless debugging off and on, then `adb kill-server` and reconnect with the new port |
| Install fails with a signature error | The APK was signed with a different key. Don't uninstall unless you accept losing your tile layout |
| An app tile shows only a name | The app isn't installed, or its package name differs. Edit the tile (hold OK > Edit) |
| Inputs list is empty | Your TV doesn't expose inputs to apps. Use the TV's own input button |
| Tile layout looks wrong after an update | Hold OK on a tile > **Reset to default tiles** |

## Privacy

TV Dock requests **no Android permissions**, has **no internet access**, and sends nothing anywhere. Your tile list is stored only on the device.

## Roadmap

- Multi-select app picker on first launch
- Animated dock-to-grid scrolling
- Optional alternative wallpapers and themes
- Optional custom wallpaper
- More tested devices and regional app lists

## Contributing

Issues and pull requests are welcome, especially:
- Your TV model and Android version, and whether it worked
- Package names for apps in your region, so the defaults cover more TVs
- Screenshots for the README

## License

Add a `LICENSE` file before publishing (for example MIT) and state it here.

## Credits and disclaimer

- Visual style inspired by [CleanYourTV](https://github.com/Gathin23/cleanyourtv).
- Android and Google TV are trademarks of Google LLC. Netflix, YouTube, Prime Video, Disney+, HBO Max, Hulu, Apple TV, Spotify, Plex, Kodi, VLC and other app names are trademarks of their owners and are used only to identify apps already installed on your TV. TV Dock is not affiliated with or endorsed by any of them.
