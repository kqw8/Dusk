English | [简体中文](README.zh-CN.md)

# Dusk

A small macOS command-line tool that dims your MacBook's built-in display when an external display is connected, and restores it when disconnected. Keep the lid open for the camera, airflow, or personal preference while using the external screen.

Dusk **mirrors the built-in display to the external and sets its brightness to zero**. It does not physically power off the panel. It never intentionally dims your only screen.

## Install

Requires macOS 13 or later. Tested on an Apple Silicon MacBook Pro running macOS 26 with an Apple Studio Display.

### Homebrew (recommended)

Install from [kqw8/homebrew-tools](https://github.com/kqw8/homebrew-tools):

```bash
brew install kqw8/tools/dusk
```

The formula builds from source, so Xcode or Command Line Tools (`xcode-select --install`) is required. **Installation does not start Dusk.** Start it separately, without `sudo`:

```bash
brew services start kqw8/tools/dusk
brew services list
```

This starts the user service and enables start-at-login. To stop it, restore the built-in display, and disable start-at-login:

```bash
brew services stop kqw8/tools/dusk
```

Use only `brew services` for this installation's service lifecycle. Do not combine it with `dusk install`, `start`, `stop`, `uninstall`, or `login`. If migrating from the standalone agent, run `dusk uninstall` using the old binary before starting the Homebrew service.

### Standalone binary

No Homebrew or Xcode required. The installer downloads the latest universal binary (Apple Silicon + Intel), verifies its SHA-256 checksum, and installs it to `/usr/local/bin`:

```bash
curl -fsSL https://raw.githubusercontent.com/kqw8/Dusk/main/scripts/install.sh | bash
```

The binary is not Apple-notarized. The installer removes its quarantine attribute after checksum verification; browser downloads may trigger Gatekeeper.

Installation alone does not start Dusk. To install and start its own background agent:

```bash
dusk install
```

This starts the agent now, with start-at-login disabled by default.

### Build from source

Requires the Swift toolchain from Xcode or Command Line Tools:

```bash
git clone https://github.com/kqw8/Dusk.git
cd Dusk
swift build -c release
.build/release/dusk install
```

The last command starts the standalone agent, pointing to the built executable. Keep that executable at this path while the agent is installed.

## Usage

For either installation:

```bash
dusk status     # Display capabilities, displays, and saved state
dusk --version # Print the version
dusk --help    # Show help
```

`dusk status` reports Dusk's own agent registration, not Homebrew's service status. For Homebrew residency and start-at-login, use `brew services list`.

For the **standalone agent only**:

| Command | Action |
| --- | --- |
| `dusk install` | Install and start the agent; start-at-login off |
| `dusk start` | Start the agent |
| `dusk stop` | Stop the agent and restore the built-in display |
| `dusk login on` | Enable start-at-login |
| `dusk login off` | Disable start-at-login without stopping the running agent |
| `dusk uninstall` | Restore the display, stop the agent, and remove its registration, state, and logs; keep the binary |

For foreground debugging, stop any background service first, then run `dusk` without arguments. Ctrl-C exits and restores the display.

## How it works

- On connection, mirrors the built-in display to the external, disables built-in auto-brightness, and sets brightness to zero. The built-in is no longer a separate desktop for windows or the pointer.
- On disconnection or normal exit, restores the built-in display. When only the built-in remains, recovery logic ensures it is visible.
- Manual override: raise the built-in brightness or switch it to extended mode to reclaim it. Drag brightness back to zero to re-engage; reconnecting the external also re-engages.
- Event-driven, with no polling or third-party dependencies. Reacts to display changes, wake, and brightness notifications; runs as a `launchd` user agent.

No Accessibility or Screen Recording permission is required. Failures are reported through notifications, stderr, and saved state shown by `dusk status`; healthy background operation is silent. Standalone logs are `~/Library/Logs/dusk.out.log` and `~/Library/Logs/dusk.err.log`. Homebrew logs are under `$(brew --prefix)/var/log/` as `dusk.log` and `dusk.err.log`.

## Limitations

- macOS has no public API to physically power off one display. Dusk uses mirroring and dimming, not panel power control.
- Brightness and auto-brightness use Apple's private `DisplayServices` framework, dynamically loaded at runtime. macOS updates can break it; missing controls are reported rather than silently ignored. Instant re-engagement after dragging brightness to zero also depends on private brightness notifications.
- Opening the lid can briefly light the panel before Dusk dims it again. Switching from extended to mirrored mode can take a second or two and cause a flash.
- Previously enabled auto-brightness is restored, so ambient light can affect the restored brightness.
- Intel Macs, HDMI/DisplayPort monitors, multiple external displays, and macOS versions other than the tested setup above have not been tested.

## Uninstall

**Stop the service and let the built-in display recover before removing the binary.**

Homebrew:

```bash
brew services stop kqw8/tools/dusk
brew uninstall kqw8/tools/dusk
```

Standalone:

```bash
dusk stop
dusk uninstall
sudo rm /usr/local/bin/dusk
```

For a custom install or source build, remove the actual executable instead of `/usr/local/bin/dusk`. The standalone uninstaller removes its agent, state, and logs; Homebrew may leave runtime state and logs behind.

## License

[MIT](LICENSE) — Copyright (c) 2026 kqw8.
