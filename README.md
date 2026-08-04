# PlayBot — Geode Mod for Geometry Dash

A practice & showcase toolkit built on the [Geode](https://geode-sdk.org) modding
framework. Includes an in-level menu, several practice-assist "hacks", a
showcase render overlay, and automatic checkpoint saving.

**Targets:** Geode `v5.7.1` · Geometry Dash `2.2081`

> This is pinned to an exact Geode version (`5.7.1`) rather than a floating
> minimum, per how this build was requested. Geode enforces major.minor
> compatibility between a mod and the installed loader, so if your Geode
> install is on a different `5.x` minor version (e.g. `5.8.x`), either
> update your loader to match, or edit the `"geode"` field in `mod.json` to
> your installed version and rebuild — the source code itself doesn't need
> to change for point-release bumps within the same major version.

> ⚠️ **Fair use note:** NoClip and Speedhack are intended as *practice tools*
> for dissecting level geometry and route-planning — not for faking legit
> completions or leaderboard submissions. Use responsibly and follow the
> rules of any server/community you play in.

## Features

- **NoClip** — survive hazard hits while in practice mode, to study geometry.
- **Speedhack** — adjustable playback speed multiplier via a keybind/setting.
- **Click Between Frames (CBF)** — sub-frame input timestamping for
  frame-perfect precision inputs. (Note: GD 2.208 added its own native
  "Click on Steps" setting; this is a separate, independent implementation
  kept for parity with classic CBF timing.)
- **Auto Save** — automatically drops a practice checkpoint every N seconds
  (configurable) so you never lose more than a few seconds of progress.
- **Showcase Render Layer**
  - Fading motion trajectory trail (great for showcase/verification videos).
  - Live hitbox outlines for the player + nearby hazards.
  - Both trail color and hitbox color are configurable.
- **In-game PlayBot menu** — a "PB" button added to the pause menu opens a
  popup with quick on/off toggles for every feature above, no need to dig
  through the mod settings page mid-level.
- **Android armeabi-v7a support** — builds for 32-bit ARM Android devices in
  addition to 64-bit and PC/Mac. See build steps below.

## Project layout

```
PlayBot/
├── mod.json              # Geode manifest + settings schema
├── CMakeLists.txt         # Build script (incl. Android ABI handling)
├── README.md
├── LICENSE
├── resources/
│   └── logo.png           # Mod icon
└── src/
    ├── main.cpp            # Entry point, setting-change listeners
    ├── Hacks.hpp/.cpp       # NoClip, Speedhack, Click Between Frames
    ├── Render.hpp/.cpp      # Showcase trajectory + hitbox render layer
    ├── AutoSave.hpp/.cpp    # Practice checkpoint auto-save
    └── ModMenu.cpp          # In-game PlayBot popup + pause menu button
```

## Prerequisites

1. **Geode CLI** — install from https://geode-sdk.org/docs/getting_started/installation
   ```
   pip install geode-cli
   # or via the installer on the Geode site
   ```
2. **CMake ≥ 3.21** and a C++20-capable compiler (MSVC on Windows, Clang on
   Mac, or the Android NDK for Android builds — the Geode CLI can fetch/manage
   the NDK for you).
3. A working **Geometry Dash** install for testing (`2.2081`, matching
   `mod.json`).

## Building — Windows / Mac (desktop)

```bash
cd PlayBot
geode build -p win       # Windows
# or
geode build -p mac       # macOS
```

The resulting `.geode` file will be in `build/`. Install it with:

```bash
geode install build/PlayBot.geode
```

## Building for Android — including armeabi-v7a (32-bit)

Geode supports two Android ABIs: `arm64-v8a` (64-bit, most modern devices)
and `armeabi-v7a` (32-bit, older/lower-end devices). This project is set up
to build for **both**.

1. Make sure the Geode CLI has an Android/NDK profile configured:
   ```bash
   geode config setup-android
   ```
2. Build the 32-bit ABI explicitly:
   ```bash
   geode build -p android32
   ```
   This is the `armeabi-v7a` target — `CMakeLists.txt` pins
   `ANDROID_ABI=armeabi-v7a` and defines `PLAYBOT_ANDROID32` for this build.
3. (Optional) Also build 64-bit for newer devices:
   ```bash
   geode build -p android64
   ```
4. Grab the generated `.geode` package(s) from `build-android32/` /
   `build-android64/` and copy them to your device's
   `Geode/mods` (or `Geode/unzipped`) folder in the GD install directory —
   or side-load with:
   ```bash
   geode install build-android32/PlayBot.geode --platform android32
   ```

If you're building manually outside the CLI, pass the ABI directly to CMake:

```bash
cmake -B build-android32 -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
  -DANDROID_ABI=armeabi-v7a \
  -DANDROID_PLATFORM=android-23
cmake --build build-android32
```

## Configuring

All settings are editable two ways:
- **In-game:** open a level, pause, tap the **PB** button → toggle any
  feature directly.
- **Geode mod settings:** Geode menu → PlayBot → gear icon, for the full
  settings list (colors, intervals, multipliers, etc).

## Notes on this source drop

This zip contains the complete, ready-to-build Geode project source. It has
**not** been compiled here because compiling requires the proprietary GD/
Cocos2d headers pulled in by the Geode SDK/CLI (and, for Android, the NDK
toolchain) — none of which are available outside a real Geode dev
environment. Running `geode build` per the steps above will produce the
installable `.geode` file(s).

## License

MIT — see `LICENSE`.
