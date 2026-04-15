# android-util

Unified CLI for Android device management — screen mirroring, logcat streaming, and auto-connect daemon.

## Prerequisites

- [adb](https://developer.android.com/tools/adb) (Android SDK platform-tools)
- [scrcpy](https://github.com/Genymobile/scrcpy)
- [jq](https://jqlang.github.io/jq/)

## Install

On macOS, symlink the script into your PATH:

```bash
ln -s "$(pwd)/device" ~/.local/bin/device
```

Then you can run `device` from any directory.

## Setup

```bash
# Create a .devices.json config in your working directory
device init

# Edit with your device serials (from `adb devices`)
vim .devices.json
```

### .devices.json format

```json
{
    "defaults": {
        "scrcpyArgs": "-m 800",
        "cleanPackages": [
            "com.example.app"
        ]
    },
    "redmi": {
        "serial": "jfpn4pw4aenrqod6",
        "logfileSuffix": "redmi-note8pro",
        "scrcpyArgs": "-m 1200 --stay-awake",
        "cleanPackages": [
            "com.example.app.dev",
            "com.example.app.staging"
        ]
    },
    "moto": {
        "serial": "NRMR410136",
        "logfileSuffix": "motog7"
    }
}
```

`scrcpyArgs` precedence: per-device > `defaults.scrcpyArgs` > hardcoded `-m 800`.

`cleanPackages` precedence: per-device > `defaults.cleanPackages`.

## Usage

```bash
# Mirror screen (one-shot)
device screen redmi
device scr redmi          # alias

# Auto-launch scrcpy when device connects (daemon mode)
device watch redmi
device w redmi            # alias
device watch              # any device

# Stream logcat to terminal
device logs redmi
device log redmi          # alias

# Stream logcat to file, zip on Ctrl+C
device logs redmi --file

# Uninstall configured packages
device clean redmi
device clean              # first connected device

# Show help
device --help
```

`DEVICE` can be a nickname from `.devices.json` or a raw adb serial. Optional for `screen`/`watch`/`logs`/`clean` (defaults to first connected device).
