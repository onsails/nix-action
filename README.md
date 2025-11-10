# Nix Action

A custom GitHub Action that installs [Determinate Nix](https://github.com/DeterminateSystems/determinate-nix-action), [FlakeHub Cache](https://github.com/DeterminateSystems/flakehub-cache-action), and [Devenv](https://devenv.sh/) with support for additional Nix profiles.

## Features

- Installs Determinate Nix with optimized configuration
- Sets up FlakeHub cache for faster builds
- Installs Devenv by default
- Supports installing additional Nix profiles
- Customizable Nix configuration
- Optional WarpBuild snapshot support for even faster CI runs

## Usage

### Basic Usage

```yaml
- uses: onsails/nix-action@v2
```

### With Additional Profiles

```yaml
- uses: onsails/nix-action@v2
  with:
    additionalProfiles: 'nixpkgs#sccache nixpkgs#cachix'
```

### With Unfree Packages

```yaml
- uses: onsails/nix-action@v2
  with:
    additionalUnfreeProfiles: 'nixpkgs#terraform nixpkgs#claude-code'
```

### With Custom Nix Configuration

```yaml
- uses: onsails/nix-action@v2
  with:
    additionalProfiles: 'nixpkgs#sccache'
    nix-extra-conf: |
      lazy-trees = true
      http-connections = 128
      max-substitution-jobs = 128
      fsync-metadata = false
      show-trace = true
      max-silent-time = 1800
```

### With WarpBuild Snapshots (Fastest CI Runs)

[WarpBuild](https://warpbuild.com) snapshot runners allow you to save the VM state after Nix and all profiles are installed, dramatically speeding up subsequent CI runs.

```yaml
jobs:
  build:
    runs-on: warp-ubuntu-latest-x64-2x
    steps:
      - uses: actions/checkout@v4

      - uses: onsails/nix-action@v2
        with:
          wb-snapshot: true
          additionalProfiles: 'nixpkgs#sccache'

      - name: Build project
        run: devenv shell -- make build
```

**How it works:**
1. First run: Installs Nix + profiles, then creates a snapshot
2. Subsequent runs on same branch: Restore from snapshot (seconds instead of minutes)
3. Snapshot is automatically unique per repository, workflow, branch, and input configuration

**Requirements:**
- WarpBuild runner (e.g., `warp-ubuntu-latest-x64-2x`)
- Set `wb-snapshot: true` in action inputs

The snapshot is created **after** all Nix profiles are installed but **before** FlakeHub authentication, ensuring no credentials are stored in the snapshot.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `additionalProfiles` | Space-separated list of additional Nix profiles to install (e.g., `nixpkgs#sccache nixpkgs#cachix`) | No | `''` |
| `additionalUnfreeProfiles` | Space-separated list of unfree Nix profiles to install (e.g., `nixpkgs#terraform nixpkgs#claude-code`) | No | `''` |
| `nix-extra-conf` | Additional Nix configuration options | No | See default config below |
| `wb-snapshot` | Enable WarpBuild snapshot creation (opt-in, requires WarpBuild runner) | No | `'false'` |
| `impatient-networking` | Enable aggressive timeout and retry settings for faster CI (opt-out, enabled by default) | No | `'true'` |

### Default Nix Configuration

The action automatically applies these optimized settings for fast CI performance:

**Impatient Networking** (enabled by default, opt-out with `impatient-networking: 'false'`):
```
connect-timeout = 5
stalled-download-timeout = 10
download-attempts = 3
narinfo-cache-negative-ttl = 5
fallback = false
```

**Additional Defaults** (can be overridden with `nix-extra-conf`):
```
lazy-trees = true
http-connections = 64
max-substitution-jobs = 64
fsync-metadata = false
show-trace = true
max-silent-time = 900
```

The impatient networking configuration ensures CI jobs fail fast rather than waiting on slow or unavailable substituters, improving overall build reliability and speed. You can disable it if you need more patient retry behavior:

```yaml
- uses: onsails/nix-action@v2
  with:
    impatient-networking: 'false'
```

## Complete Example

```yaml
name: Build with Nix

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Nix and Devenv
        uses: onsails/nix-action@v2
        with:
          additionalProfiles: 'nixpkgs#sccache nixpkgs#cachix'

      - name: Build project
        run: devenv shell -- make build
```

## What This Action Does

1. **Installs Determinate Nix** - Sets up Nix with optimized configuration for CI/CD
2. **Installs Devenv** - Automatically installs `nixpkgs#devenv`
3. **Installs Additional Profiles** - Optionally installs any additional Nix profiles you specify
4. **Creates WarpBuild Snapshot** (Optional) - Saves VM state after Nix installation for faster subsequent runs
5. **Configures FlakeHub Cache** - Enables faster builds with FlakeHub's binary cache

## License

MIT
