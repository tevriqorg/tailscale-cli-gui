# Tailscale CLI GUI

<img width="257" height="311" alt="tailscale-cli-gui-ss1" src="https://github.com/user-attachments/assets/b6f73437-ee47-483c-8bd9-8c792bf73418" />


A macOS menu bar app for the open-source/Homebrew Tailscale installation.

The Mac App Store version of Tailscale includes a GUI, but the open-source CLI version (which supports advanced features like SSH hosting) does not. This app bridges that gap.

## PLEASE NOTE

This project is 100% vibe-coded. Let Claude write this utility while attending to other, more important work. Please audit the code before running.

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
