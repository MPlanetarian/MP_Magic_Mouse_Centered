# MP Magic Mouse Centered

[![Python 3](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey.svg)](#platform-support)
[![Display Servers](https://img.shields.io/badge/Display-X11%20%7C%20Wayland%20%7C%20Quartz%20%7C%20GDI-brightgreen.svg)](#platform-support)
[![Dependencies](https://img.shields.io/badge/Dependencies-Zero%20External-green.svg)](#architecture)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A lightweight, zero-dependency cross-platform utility that automatically centers your mouse cursor on your display whenever you log in to your desktop, and provides an instant global hotkey (**ALT+C**) to snap the cursor back to the center of your screen anytime.

> *"You will never need to find your Mouse Cursor ever again because it will always be in the Center of your Primary Screen when you login to your Desktop."*

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Platform Support](#platform-support)
- [Quick Start](#quick-start)
- [First-Time Run Experience](#first-time-run-experience)
- [Interactive CLI Menu](#interactive-cli-menu)
- [Command-Line Flags](#command-line-flags)
- [Architecture & Implementation](#architecture--implementation)
- [Multi-Monitor Configuration](#multi-monitor-configuration)
- [Remote Desktop & Virtual Displays](#remote-desktop--virtual-displays)
- [Repository Structure](#repository-structure)
- [Uninstallation](#uninstallation)
- [License](#license)

---

## Overview

On modern ultrawide screens, high-DPI 4K/5K displays, and multi-monitor setups, locating the mouse cursor after unlocking or logging into your desktop is a recurring annoyance. **MP Magic Mouse Centered** solves this permanently:

1. **At Desktop Login**: The background service automatically warps the mouse cursor directly to the center of your primary screen (or designated monitor).
2. **On Demand Anytime**: Pressing **ALT+C** instantly snaps your cursor to the center of the screen, saving time and eliminating cursor hunting.

---

## Key Features

- **Cross-Platform**: Full native support for **macOS**, **Windows**, and **Linux**.
- **Comprehensive Linux Support**: Seamless operation on both **X11** and modern **Wayland** compositors (GNOME, KDE Plasma, Sway, Hyprland, and wlroots-based compositors).
- **Zero Heavy External Dependencies**: Built entirely with Python standard library and native OS bindings via `ctypes` (`CoreGraphics` on macOS, Win32 `user32` on Windows, `libX11` / Wayland tools on Linux).
- **Permanent Background Service**:
  - **macOS**: Native user LaunchAgent (`~/Library/LaunchAgents/com.mp.magicmousecentered.plist`) managed with `launchctl`.
  - **Windows**: Startup Registry Run Key (`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`).
  - **Linux**: Systemd user service (`~/.config/systemd/user/mp-magic-mouse-centered.service`) + XDG Autostart desktop entry (`~/.config/autostart/`).
- **Instant Global Hotkey (ALT+C)**: Centers your mouse cursor instantly from any window or application.
- **Multi-Monitor Awareness**: Configure cursor centering on your primary screen (default) or any connected secondary display.
- **Synthesized Cheerful Chime**: Self-synthesizes and plays a friendly chime upon first-run centering and testing without requiring third-party audio packages.
- **Resource Efficient**: 0% CPU idle usage, zero disk clutter, and no unwanted log file accumulation.
- **Dual Interface**: Full interactive terminal menu for configuration, plus headless CLI flags for automation and shell scripting.

---

## Platform Support

| Operating System | Cursor Centering Engine | Hotkey Engine | Background Service |
|---|---|---|---|
| **macOS** (10.14+) | `cliclick` / `CoreGraphics` (`CGWarpMouseCursorPosition`) | `CGEventSourceKeyState` polling | `LaunchAgent` (`launchctl`) |
| **Windows** (10/11) | Win32 `user32.dll` (`SetCursorPos`) | Win32 `RegisterHotKey` (WM_HOTKEY) | Windows Registry Run key |
| **Linux (X11)** | `libX11` (`XWarpPointer`) / `xdotool` | `XGrabKey` / `libX11` event loop | `systemd --user` + XDG Autostart |
| **Linux (Wayland)** | `ydotool` / `hyprctl` / `wlrctl` / `kdotool` | Native compositor binds / XDG fallback | `systemd --user` + XDG Autostart |

---

## Quick Start

### 1. Clone the Repository
```bash
git clone git@github.com:MPlanetarian/MP_Magic_Mouse_Centered.git
cd MP_Magic_Mouse_Centered
```

### 2. Make Executable & Run
```bash
chmod +x MP_Magic_Mouse_Centered
./MP_Magic_Mouse_Centered
```
*(Alternatively: `python3 MP_Magic_Mouse_Centered.py`)*

---

## First-Time Run Experience

When run for the very first time, **MP Magic Mouse Centered** automatically performs the initial setup:
1. Warps your mouse cursor directly to the center of your primary monitor.
2. Plays the synthesized cheerful alert chime.
3. Prints the welcome notice:
   ```text
   ================================================================================
   You will never need to find your Mouse Cursor ever again because it will always be in the Center of your Primary Screen when you login to your Desktop.
   ================================================================================
   ```
4. Automatically installs and activates the permanent background login service for your operating system.
5. Saves your initial configuration to `config.json`.

---

## Interactive CLI Menu

Running `./MP_Magic_Mouse_Centered` without flags opens the interactive management console:

```text
=================================================================
                 MP MAGIC MOUSE CENTERED
=================================================================
 [1] Center Mouse Cursor Now
 [2] Configure Target Monitor (Multi-Display Setup)
 [3] Install / Enable Background Login Service & Hotkey (Alt+C)
 [4] Remove & Disable Mouse Centering Service
 [5] Test Happy Alert Sound
 [6] Toggle Alert Sound on Shortcut (Alt+C)
 [7] View Current Configuration & Service Status
 [0] Exit
=================================================================
```

### Options Breakdown
- **[1] Center Mouse Cursor Now**: Instantly warps the mouse cursor to the target monitor center.
- **[2] Configure Target Monitor**: Detects connected displays with geometry information and allows selecting the primary display or a specific display index.
- **[3] Install / Enable Background Login Service**: Installs and starts the background daemon and enables the **ALT+C** global shortcut.
- **[4] Remove & Disable Mouse Centering Service**: Safely terminates the background daemon, unloads the service, and cleans up autostart entries.
- **[5] Test Happy Alert Sound**: Tests playback of the synthesized alert chime.
- **[6] Toggle Alert Sound on Shortcut**: Toggles whether the chime sounds whenever **ALT+C** is pressed.
- **[7] View Current Configuration & Service Status**: Shows OS details, display server, monitor list, target screen, and daemon status.

---

## Command-Line Flags

Ideal for scripting, automation, dotfiles, or remote management:

| Flag | Description |
|---|---|
| `--center` | Immediately center mouse cursor on target display and exit |
| `--daemon` | Start background daemon (used by login service to center on login & listen for ALT+C) |
| `--install` | Install and activate the background login service |
| `--uninstall` | Stop and remove the background service |
| `--status` | Print current system configuration and service health |
| `--list-monitors` | List all connected displays with indices, IDs, and resolutions |
| `--set-monitor <idx\|primary>` | Set target monitor (e.g. `--set-monitor 1` or `--set-monitor primary`) |
| `--play-sound` | Test play the synthesized chime |
| `--menu` | Explicitly launch the interactive menu |

### Example CLI Commands
```bash
# Center cursor immediately from a terminal or custom shortcut
./MP_Magic_Mouse_Centered --center

# List all connected displays
./MP_Magic_Mouse_Centered --list-monitors

# Set cursor to target display index 1
./MP_Magic_Mouse_Centered --set-monitor 1

# Check current status
./MP_Magic_Mouse_Centered --status
```

---

## Architecture & Implementation

### macOS
- **Warp & Position**: Uses `cliclick` if installed, with native fallback to `CoreGraphics` (`CGWarpMouseCursorPosition`, `CGDisplayMoveCursorToPoint`, `CGDisplayBounds`).
- **Cursor Unhide**: Calls `CGDisplayShowCursor` and posts a zero-delta mouse event to ensure the pointer is unhidden if keyboard typing made it disappear.
- **Hotkey**: Uses `CGEventSourceKeyState` polling with adaptive sleep cycles for 0% idle CPU overhead, bypassing macOS Accessibility permission barriers.
- **Service**: Managed via `launchd` using a user LaunchAgent in `~/Library/LaunchAgents/com.mp.magicmousecentered.plist`.

### Windows
- **Warp & Position**: Direct invocation of Win32 `user32.dll` (`SetCursorPos` and `EnumDisplayMonitors`).
- **Hotkey**: Uses Win32 `RegisterHotKey` with message loop (`GetMessageW`), with fallback to `GetAsyncKeyState`.
- **Service**: Registered in Windows User Registry Run key (`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`).

### Linux (Wayland & X11)
- **Session Auto-Detection**: Checks `WAYLAND_DISPLAY` and `XDG_SESSION_TYPE`.
- **Wayland Support**: Integrates with `ydotool`, `hyprctl` (Hyprland), `wlrctl` (Sway/wlroots), and `kdotool` (KDE Plasma).
- **X11 Support**: Uses `libX11` via `ctypes` (`XWarpPointer`, `XGrabKey`), with seamless fallback to `xdotool` and `xrandr`.
- **Service**: Dual registration with `systemd --user` unit and standard XDG autostart desktop entry (`~/.config/autostart/`).

---

## Multi-Monitor Configuration

By default, the cursor is placed in the center of your **primary monitor**. For multi-monitor workstations:
1. Run `./MP_Magic_Mouse_Centered --list-monitors` to see detected displays.
2. Run `./MP_Magic_Mouse_Centered --set-monitor <index>` (e.g., `0`, `1`, etc.) or choose Option `[2]` in the interactive menu.
3. The selected monitor is persisted in `config.json`.

---

## Remote Desktop & Virtual Displays

If you manage your machine via **NoMachine**, **VNC**, **TeamViewer**, or **RDP**:
- Most remote desktop clients draw a decoupled *local client pointer* over the video stream, which can mask programmatic cursor repositioning on the server.
- **NoMachine Recommendation**: In the NoMachine client viewer menu (`Ctrl+Alt+0` or page-peel -> *Input*), check **"Show remote cursor pointer"**. This synchronizes the visible pointer directly with host operating system coordinates.
- On macOS hosts, installing `brew install cliclick` ensures maximum compatibility across remote desktop software.

---

## Repository Structure

```
MP_Magic_Mouse_Centered/
├── MP_Magic_Mouse_Centered     # Main executable script (Python 3)
├── MP_Magic_Mouse_Centered.py  # Entrypoint symlink
├── config.json                 # Persistent configuration settings
├── assets/
│   └── happy_alert.wav         # Synthesized cheerful alert chime
├── LICENSE                     # MIT License
├── .gitignore                  # Git ignore rules
└── README.md                   # Documentation
```

---

## Uninstallation

To completely remove the background service and autostart files:
```bash
./MP_Magic_Mouse_Centered --uninstall
```
Or launch the menu (`./MP_Magic_Mouse_Centered`) and select Option `[4]`.

---

## License

This project is licensed under the [MIT License](LICENSE) - see the LICENSE file for details.

Copyright (c) 2026 MPlanetarian.
