# PRD: `device` CLI

## Overview

A unified command-line tool for Android device management that wraps `adb` and `scrcpy`. It provides screen mirroring, an auto-connect daemon, and logcat streaming through a single `device` command with per-device configuration.

## Dependencies

- `adb` — Android SDK platform-tools
- `scrcpy` — screen mirroring
- `jq` — JSON parsing for device config

The CLI checks for all three at startup and prints actionable install guidance for any that are missing.

## Device Configuration

Devices are configured in a `.devices.json` file in the current working directory.

### Format

```json
{
    "defaults": {
        "scrcpyArgs": "-m 800"
    },
    "<nickname>": {
        "serial": "<adb serial>",
        "logfileSuffix": "<suffix for log filenames>",
        "scrcpyArgs": "<scrcpy flags>"
    }
}
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `serial` | Yes | Device serial as shown by `adb devices` |
| `logfileSuffix` | No | Human-readable suffix for log filenames (e.g. `redmi-note8pro`) |
| `scrcpyArgs` | No | Per-device scrcpy arguments (overrides defaults) |
| `defaults.scrcpyArgs` | No | Fallback scrcpy arguments for all devices |

### `device init`

Creates a `.devices.json` template in the current directory. Refuses if the file already exists.

## Device Resolution

All subcommands accept an optional `DEVICE` argument. Resolution follows this order:

1. If `DEVICE` is provided, look it up as a nickname in `.devices.json`
2. If not found in config (or no config exists), treat `DEVICE` as a raw adb serial
3. If `DEVICE` is omitted, use the first connected device reported by `adb devices`

When a device is auto-detected by serial (step 3), the CLI also checks `.devices.json` for a matching `serial` field to pick up associated config (e.g. `logfileSuffix`).

## Subcommands

### `device screen [DEVICE]` (alias: `scr`)

Launches scrcpy once for the resolved device, then exits.

**scrcpy arguments precedence:**
1. Per-device `scrcpyArgs` from `.devices.json`
2. `defaults.scrcpyArgs` from `.devices.json`
3. Hardcoded `-m 800`

### `device watch [DEVICE]` (alias: `w`)

Daemon mode — polls for a device connection and auto-launches scrcpy when the device is available.

**Behavior:**
- Polls device state every 2 seconds
- When a device connects, launches scrcpy with the resolved arguments
- When scrcpy exits with code 0 (user closed the window), the daemon exits
- When scrcpy exits with any non-zero code (device disconnected, error, etc.), the daemon resumes polling
- If the device is already reconnected after a non-zero exit, polling resumes immediately; otherwise waits 5 seconds before polling
- Logs unauthorized and offline device states as warnings so the user knows why nothing is happening
- Ctrl+C (SIGINT) or SIGTERM cleanly kills scrcpy (if running) and exits

**Device state checking:**
- With a known serial: uses `adb -s <serial> get-state` for a targeted check
- Without a serial (any-device mode): parses `adb devices` output and picks the first connected device

### `device logs [DEVICE] [--file]` (alias: `log`)

Streams logcat output from the resolved device.

**Terminal mode (default):**
- Streams `adb logcat` directly to stdout

**File mode (`--file`):**
- Creates a `logs/` directory if it doesn't exist
- Writes logcat output to `logs/<timestamp>-<suffix>.txt`
- On Ctrl+C, zips the log file and prints the paths
- The `<suffix>` is resolved as: `logfileSuffix` from config > device serial number

## CLI Reference

```
device screen [DEVICE]          Launch scrcpy once (alias: scr)
device watch  [DEVICE]          Auto-launch scrcpy on device connect (alias: w)
device logs   [DEVICE] [--file] Stream logcat to terminal or file (alias: log)
device init                     Create .devices.json template in current directory
device --help                   Show usage information
```
