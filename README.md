<img src="https://avatars.githubusercontent.com/u/1785912?s=96&v=4" alt="Cru logo" title="Cru" align="right" height="96" width="96"/>

# mise-ccrotate
[ccrotate](https://github.com/CruGlobal/ccrotate) plugin for [mise](https://mise.jdx.dev).

ccrotate rotates between Claude Enterprise seats before any one of them
exhausts its five-hour usage window, so running Claude Code sessions and their
sub-agents never hit a rate limit.

**macOS only.** ccrotate uses the macOS Keychain and launchd. The install
script refuses to run anywhere else.

## Requirements
- [mise](https://mise.jdx.dev/)
- [GitHub CLI](https://cli.github.com/) `v2.68.0` or later, authenticated

## Install
Ensure you have [mise](https://mise.jdx.dev/getting-started.html) installed and the [GitHub CLI](https://github.com/cli/cli#installation) installed.
Since the `ccrotate` repository is private, you will need to have the GitHub CLI authenticated with your GitHub account.
You can check GitHub CLI authentication status with `gh auth status` or re-authenticate with `gh auth login`.

```shell
brew install gh && gh auth login
```

This plugin shells out to `gh release download`, which reuses the credentials
you already have. There is no personal access token to create, and no Go
toolchain to install, since the binaries are prebuilt.

This plugin repository is public so that `mise plugin add` works without
credentials. The private part is the binaries, not the plugin.

#### Install plugin
```shell
mise plugin add ccrotate https://github.com/CruGlobal/mise-ccrotate
```

#### Install `ccrotate` version
```shell
# Show all installable versions
mise ls-remote ccrotate

# Install latest version
mise install ccrotate@latest

# Install specific version
mise install ccrotate@<version>

# Set a version globally
mise use -g ccrotate@latest
```

#### Configuration with mise.toml
You can also configure ccrotate in your project's `mise.toml` or global `~/.config/mise/config.toml`:

```toml
[tools]
ccrotate = "latest"
```

Then run:
```shell
mise install
```

## Setup

```shell
ccrotate init
ccrotate account add <name> --email <you@cru.org>   # once per seat, opens a browser
ccrotate doctor
ccrotate install
```

`ccrotate install` loads a LaunchAgent and points Claude Code at the local proxy
by setting `ANTHROPIC_BASE_URL` in `~/.claude/settings.json` (backed up first).
Restart any running `claude` sessions afterwards. Remote Control and `/schedule`
are disabled while traffic goes through the proxy. `ccrotate uninstall` reverses
both changes.

## Upgrading

```shell
mise install ccrotate@<version>
mise use -g ccrotate@<version>
ccrotate install      # re-point the LaunchAgent
```

That last step is not optional. The LaunchAgent records an absolute path to the
binary, and mise installs each version to its own directory under
`~/.local/share/mise/installs/ccrotate/`, so the daemon keeps running the
previous release, which still exists, so nothing errors.
`ccrotate doctor` warns when the agent and your `PATH` disagree.

## Migrating from asdf-ccrotate

If you previously used [asdf-ccrotate](https://github.com/CruGlobal/asdf-ccrotate):

```shell
mise plugin add ccrotate https://github.com/CruGlobal/mise-ccrotate
mise install ccrotate@latest
mise use -g ccrotate@latest
ccrotate install      # re-point the LaunchAgent at the mise-managed binary
```

Then remove the asdf version with `asdf plugin remove ccrotate` once
`ccrotate doctor` is happy.
