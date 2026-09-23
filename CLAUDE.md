# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a mise plugin for installing and managing the `ccrotate` tool. The plugin integrates with GitHub CLI to download and install prebuilt releases from the private CruGlobal/ccrotate repository.

ccrotate runs on macOS and Linux. On macOS it uses the Keychain and launchd; on Linux it uses the libsecret Secret Service (with a file fallback) and a systemd user service. `bin/install` accepts `darwin` and `linux` and exits with a clear message on anything else. `bin/list-all` works on any platform.

Mise is a polyglot runtime manager that is compatible with asdf plugins, so this plugin's scripts follow asdf/mise plugin conventions.

## Requirements

- **mise**: Latest version recommended
- **GitHub CLI (gh)**: v2.68.0 or later, authenticated with access to CruGlobal repositories
  - Check auth status: `gh auth status`
  - Re-authenticate if needed: `gh auth login`

## Architecture

### Plugin Structure

Two core scripts in `bin/`:

- **bin/list-all**: Fetches available versions from GitHub releases using `gh release list`. Returns versions sorted in ascending order with the `v` prefix removed.
- **bin/install**: Downloads and installs the specified version. Key behaviors:
  - Refuses to run unless `uname` reports `Darwin` or `Linux`
  - Refuses to run unless `gh` is installed and authenticated
  - Downloads the platform/architecture-specific tarball with `gh release download`
  - Extracts the `ccrotate` binary to `${install_path}/bin`
  - Supports architectures: amd64, arm64
  - Prints post-install setup steps and the service upgrade caveat

### Environment Variables

The plugin uses these mise environment variables:
- `MISE_INSTALL_TYPE`: Installation type (version, ref, path)
- `MISE_INSTALL_VERSION`: The version to install
- `MISE_INSTALL_PATH`: Where to install the tool

### Release Naming Convention

Tarballs follow the pattern: `ccrotate-v{version}-{platform}-{arch}.tar.gz`, containing a single `ccrotate` binary at the top level.

### Upgrade Caveat

`ccrotate install` writes a service definition holding the absolute path of the binary: a LaunchAgent plist on macOS, a systemd user unit on Linux. Because mise installs each version to its own directory, upgrading the tool does not upgrade the running daemon until the user re-runs `ccrotate install`. Keep this note in both the README and the install script's trailing message.

## Development

### Testing Local Changes

```bash
# Link this checkout as the plugin (for testing)
mise plugin link --force ccrotate /path/to/mise-ccrotate

# List all versions to test list-all script (works on Linux too)
mise ls-remote ccrotate

# Install a specific version to test the install script (macOS or Linux)
mise install ccrotate@<version>

# Verify installation
mise which ccrotate

# Remove plugin
mise plugin remove ccrotate
```

## Relationship to asdf-ccrotate

This is a port of [asdf-ccrotate](https://github.com/CruGlobal/asdf-ccrotate). The install logic is identical; the differences are:
- `MISE_INSTALL_*` environment variables instead of `ASDF_INSTALL_*`
- A guard that exits early when the scripts are run outside mise/asdf
- Documentation updated for mise commands (`mise install` vs `asdf install`)

When changing install behavior, mirror the change in asdf-ccrotate.
