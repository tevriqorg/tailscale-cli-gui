# Tailscale CLI GUI - Project Context for Claude

## Overview

This is a macOS menu bar (status bar) app that provides a GUI for the open-source/Homebrew installation of Tailscale. The official Tailscale GUI only comes with the Mac App Store version, but the open-source install supports advanced features like SSH hosting. This app bridges that gap.

## Project Structure

```
Tailscale CLI GUI/
├── Tailscale CLI GUI.xcodeproj/
├── Tailscale CLI GUI/
│   ├── AppDelegate.swift          # App entry point, initializes StatusBarController
│   ├── TailscaleStatus.swift      # Data models (ConnectionState, TailnetAccount, Machine, TailscaleUser, TailscaleInfo, errors)
│   ├── TailscaleService.swift     # Actor handling all Tailscale CLI interactions
│   ├── StatusBarController.swift  # Menu bar UI and user interactions
│   ├── SettingsWindowController.swift  # Settings window (version, launch at login)
│   ├── Info.plist                 # LSUIElement=YES (agent app, no dock icon)
│   ├── Base.lproj/
│   │   └── MainMenu.xib           # Minimal XIB, just wires up AppDelegate
│   └── Assets.xcassets/           # App icon and menu bar icons
├── icon-app.svg                   # App icon source
├── icon-*-template.svg            # Menu bar icon sources (7 files)
├── CLAUDE.md                      # This file
├── README.md
└── LICENSE                        # MIT
```

## Architecture

### Data Flow
1. `StatusBarController` owns the UI and refresh logic
2. `TailscaleService` (Swift Actor) handles all CLI commands with mutex protection
3. `TailscaleStatus.swift` defines shared data types
4. Refresh triggers: Timer (5 min), NWPathMonitor (network changes), menu open, manual

### Key Design Decisions

- **Agent App**: `LSUIElement=YES` in Info.plist hides from Dock
- **No Sandbox**: Disabled to allow shell command execution
- **Actor for Service**: Prevents concurrent CLI calls via `isRefreshing` flag
- **NWPathMonitor**: Provides near-realtime updates when network interfaces change (including Tailscale's utun)

## Features Implemented

### Status Display
- Connection state (Connected/Disconnected/Unknown)
- Tailscale IP address (100.x.x.x)
- Current tailnet name
- Custom menu bar icons (see Icons section below)

### Actions
- **Connect**: `tailscale up`
- **Disconnect**: `tailscale down`
- **Switch Tailnet**: `tailscale switch <id>` (with auto disconnect/reconnect if was connected)
- **Enable/Disable SSH**: `tailscale set --ssh=true/false`
- **Copy IP**: Click IP line to copy
- **Copy Tailnet**: Click tailnet line to copy
- **Copy Machine IP/Domain**: Click machine to copy IP, Option-click for Tailscale domain
- **Refresh**: Manual refresh option

### Tailnet Switching
- Submenu only appears when 2+ tailnets available
- Current tailnet marked with checkmark
- Switch logic:
  - If connected: down → switch → up
  - If disconnected: just switch
- Errors shown in alert dialog

### Machines List
- Shows online machines on the tailnet (excludes Mullvad exit nodes)
- Organized into "Tagged" (tagged-devices) and user submenus
- Online status shown with filled circle icon
- Click copies IPv4 address, Option-click copies Tailscale domain name

## CLI Commands Used

```bash
# Find binary
which tailscale

# Check status (JSON, active only to limit output size)
tailscale status --json --active

# List machines (peers, excluding self)
tailscale status --json --self=false

# List tailnets (accounts)
tailscale switch --list

# Connect/Disconnect
tailscale up
tailscale down

# Switch tailnet
tailscale switch <id>

# Get preferences (SSH status, etc.)
tailscale debug prefs

# Enable/disable SSH
tailscale set --ssh=true
tailscale set --ssh=false
```

## Binary Discovery

Searches in order:
1. `$PATH` via `which tailscale`
2. `/usr/local/bin/tailscale`
3. `/opt/homebrew/bin/tailscale`
4. `/usr/bin/tailscale`
5. `/bin/tailscale`

Symlinks are resolved to find actual binary location.

## Parsing Logic

### `tailscale status --json`
Returns JSON with key fields:
- `BackendState`: "Running" = active, "Stopped" = disconnected
- `Self.Online`: true/false for online status
- `Self.TailscaleIPs`: array with IPv4 (100.x.x.x) and IPv6 addresses
- `CurrentTailnet.Name`: current tailnet name (e.g., "flyclops.com")

### `tailscale switch --list`
```
ID    Tailnet                 Account
3fd9  example-corp.com        user@example-corp.com*
835d  example.com             user@example.com
```
- Asterisk (*) at end of Account indicates current selection
- Parse all rows to populate Tailnets submenu

### `tailscale debug prefs`
Returns JSON with preferences:
- `RunSSH`: true/false for SSH server status

### `tailscale status --json --self=false`
Returns JSON with `Peer` and `User` dictionaries:
- `Peer`: Dictionary keyed by nodekey, each peer has:
  - `HostName`: device name
  - `DNSName`: full DNS name (e.g., "device.tailnet.ts.net.")
  - `OS`: operating system (macOS, iOS, android, linux, etc.)
  - `UserID`: numeric user ID (matches keys in `User` dict)
  - `Tags`: array of tags (Mullvad nodes have "tag:mullvad-exit-node")
  - `Online`: true/false
- `User`: Dictionary keyed by UserID, each user has:
  - `LoginName`: email or "tagged-devices"
  - `DisplayName`: human-readable name
- Mullvad nodes are filtered out (check Tags or DNSName contains ".mullvad.ts.net")
- Tagged devices belong to user with `LoginName: "tagged-devices"`

## Menu Structure

```
Connected                    (or Disconnected/Status Unknown)
IP: 100.x.x.x               (click to copy, dimmed if no IP)
Tailnet: example-corp.com   (click to copy)
Enable SSH                   (or Disable SSH, toggles SSH server)
Machines                  ▶  (only if online machines exist)
  Tagged                  ▶  (only if tagged devices exist)
    ● server-1               (click to copy IP, Option-click for domain)
  User Name               ▶  (one submenu per user)
    ● laptop
Disconnect                   (or Connect, based on state)
────────────────────────────
Tailnets                  ▶  (only if 2+ tailnets)
  ✓ example-corp.com
    other-tailnet.com
────────────────────────────
Refresh                      (Option key to show)
Settings...
Quit
```

## Important Implementation Details

### Refresh Mutex
`TailscaleService.refresh()` returns `TailscaleInfo?` - returns `nil` if refresh already in progress to prevent race conditions. StatusBarController skips UI update when nil.

### Network Monitor Timing
`NWPathMonitor` fires immediately on start. To avoid race conditions, it's started AFTER the initial refresh completes, not in init.

### Menu Layout Warning Fix
Had to use `DispatchQueue.main.async` in `menuWillOpen` to avoid "layoutSubtreeIfNeeded" recursion warning.

### Working Animation
Operations (connect/disconnect/switch) cycle through working icon frames (top→middle→bottom row sweep) at 0.2s intervals. Refresh afterward restores correct status icon.

## Logging

Uses `os.Logger` with subsystem `com.flyclops.tailscale-cli-gui`:
- Category `TailscaleService`: Binary discovery, CLI command results
- Category `StatusBarController`: Refresh triggers with reasons, state changes

View in Console.app filtered by subsystem.

## Future Enhancements (Not Yet Implemented)

- Exit nodes (blocked: CLI version on macOS cannot activate exit nodes)

## Icons

Custom menu bar icons using a 3x3 dot grid design inspired by Tailscale's mesh network logo, with a `<` caret shape representing CLI. Icons are macOS template images (monochrome with transparency) that automatically adapt to light/dark mode.

### Grid Layout
```
1  2  3
4  5  6
7  8  9
```

### Icon Patterns

| Icon | State | Bright Dots | Notes |
|------|-------|-------------|-------|
| `icon-connected` | Connected | 1,4,5,6,7,8 + caret | Caret (1→5→7) connects the bright dots |
| `icon-disconnected` | Disconnected | None | All dots + caret at 40% opacity |
| `icon-unknown` | Unknown | 1,2,3,5,6,8 | Question mark pattern, no caret |
| `icon-error` | Binary not found | 1,3,5,7,9 | X pattern, no caret |
| `icon-working-1` | Working frame 1 | 1,2,3 | Top row bright |
| `icon-working-2` | Working frame 2 | 4,5,6 | Middle row bright |
| `icon-working-3` | Working frame 3 | 7,8,9 | Bottom row bright |

### Technical Details (Menu Bar Icons)
- SVG source files: `icon-*-template.svg` (64x64 viewBox)
- PNG exports: @1x (18px) and @2x (36px) in `Assets.xcassets`
- Dot radius: 4.56 (in 64px space)
- Caret stroke width: 9.12 (matches dot diameter)
- Muted elements: 40% opacity
- Asset catalog: `template-rendering-intent: template`
- Conversion tool: `rsvg-convert` (from librsvg)

### App Icon
- SVG source: `icon-app.svg` (1024x1024 viewBox)
- Design: Connected state pattern with dark background
- Background: rgb(27, 24, 22) with rounded corners (rx/ry=180)
- Foreground: Bright dots rgb(254, 253, 250), muted dots rgb(104, 101, 101)
- Sizes generated: 16, 32, 64, 128, 256, 512, 1024px
- Location: `Assets.xcassets/AppIcon.appiconset/`

## Build Notes

- Xcode project uses XIB (not SwiftUI)
- App Sandbox: OFF (required for shell command execution)
- Hardened Runtime: OFF
- Deployment target: macOS 13.0+
- Bundle ID: `com.flyclops.tailscale-cli-gui`
- Requires: Tailscale CLI (`brew install tailscale`)
