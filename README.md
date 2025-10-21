# Nix Action

A custom GitHub Action that installs [Determinate Nix](https://github.com/DeterminateSystems/determinate-nix-action), [FlakeHub Cache](https://github.com/DeterminateSystems/flakehub-cache-action), and [Devenv](https://devenv.sh/) with support for additional Nix profiles.

## Features

- Installs Determinate Nix with optimized configuration
- Sets up FlakeHub cache for faster builds
- Installs Devenv by default
- Supports installing additional Nix profiles
- Customizable Nix configuration

## Usage

### Basic Usage

```yaml
- uses: onsails/nix-action@v1
```

### With Additional Profiles

```yaml
- uses: onsails/nix-action@v1
  with:
    additionalProfiles: 'nixpkgs#sccache nixpkgs#cachix'
```

### With Custom Nix Configuration

```yaml
- uses: onsails/nix-action@v1
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

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `additionalProfiles` | Space-separated list of additional Nix profiles to install (e.g., `nixpkgs#sccache nixpkgs#cachix`) | No | `''` |
| `nix-extra-conf` | Additional Nix configuration options | No | See default config below |

### Default Nix Configuration

```
lazy-trees = true
http-connections = 64
max-substitution-jobs = 64
fsync-metadata = false
show-trace = true
max-silent-time = 900
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
        uses: your-username/nix-action@v1
        with:
          additionalProfiles: 'nixpkgs#sccache nixpkgs#cachix'

      - name: Build project
        run: devenv shell -- make build
```

## What This Action Does

1. **Installs Determinate Nix** - Sets up Nix with optimized configuration for CI/CD
2. **Configures FlakeHub Cache** - Enables faster builds with FlakeHub's binary cache
3. **Installs Devenv** - Automatically installs `nixpkgs#devenv`
4. **Installs Additional Profiles** - Optionally installs any additional Nix profiles you specify

## License

MIT
