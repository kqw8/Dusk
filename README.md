English | [简体中文](README.zh-CN.md)

# Dusk

A macOS CLI tool that dims the MacBook's built-in display when an external display connects, and restores it when disconnected.

**Dusk mirrors the built-in display and sets brightness to zero; it does not power off the panel or intentionally dim your only screen.**

## Install

Requires macOS 13+ and Xcode or Command Line Tools; Homebrew builds from source.

```sh
brew install kqw8/tools/dusk
brew services start kqw8/tools/dusk
```

Installation alone does not start Dusk. Start the user service without `sudo`; this also enables start-at-login.
**Use only `brew services` for service management; do not mix it with native `dusk install/start/stop/uninstall/login`.** For migration, remove the old standalone agent first; see the guide below.

## Common commands

```sh
dusk status
brew services list
brew services stop kqw8/tools/dusk
```

`dusk status` shows display capabilities and saved state; its agent registration is not Homebrew service status. Stopping the service restores the built-in display and disables start-at-login.

## Limitations

No Accessibility or Screen Recording permission is required. Private `DisplayServices` APIs may break after macOS updates.
Tested on an Apple Silicon MacBook Pro with macOS 26 and Apple Studio Display; other setups are untested. Opening the lid or switching mirroring may briefly light or flash the panel.

## Uninstall

Stop the service first and wait for the built-in display to recover:

```sh
brew services stop kqw8/tools/dusk
```

Then remove the binary:

```sh
brew uninstall kqw8/tools/dusk
```

See the [guide](docs/guide.md) for standalone/source installation, migration, manual override, behavior, limitations, debugging, and leftover cleanup. The standalone binary is not Apple-notarized; its installer removes quarantine after checksum verification.

[MIT License](LICENSE)
