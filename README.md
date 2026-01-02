# Tailscale CLI GUI

<img width="334" height="299" alt="Screen Shot 2026-01-01 at 8 43 41 PM" src="https://github.com/user-attachments/assets/d318ac97-c5e4-42e9-b8a1-7fcf49372d5e" />

A macOS menu bar app for the open-source/Homebrew Tailscale installation.

The Mac App Store version of Tailscale includes a GUI, but the open-source CLI version (which supports advanced features like SSH hosting) does not. This app bridges that gap.

## Features

- Connection status with menu bar icon
- Connect/disconnect
- View/copy Tailscale IP address
- View/copy current tailnet
- Toggle SSH server
- Switch between multiple tailnets
- Auto-refresh + network change detection
- Machines list with ip/hostname copy
- Run at login

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

- Exit node list and connection

## License

See [LICENSE](LICENSE)
