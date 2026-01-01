# Tailscale CLI GUI

A macOS menu bar app for the open-source/Homebrew Tailscale installation.

The Mac App Store version of Tailscale includes a GUI, but the open-source CLI version (which supports advanced features like SSH hosting) does not. This app bridges that gap.

## Features

- Connection status with menu bar icon
- View/copy Tailscale IP address
- View/copy current tailnet
- Connect/disconnect
- Switch between multiple tailnets
- Auto-refresh (30s) + network change detection

## Requirements

- macOS 13.0+
- Tailscale CLI installed (`brew install tailscale`)

## Build

```bash
xcodebuild -scheme "Tailscale CLI GUI" -configuration Release build
```

Or open `Tailscale CLI GUI.xcodeproj` in Xcode and build.

## Install

Copy `Tailscale CLI GUI.app` to `/Applications`.

The app runs as a menu bar agent (no Dock icon).

## TODO

- Settings screen with version and register for open on login
- Exit node list and connection

## License

MIT
