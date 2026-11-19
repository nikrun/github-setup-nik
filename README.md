<p align="left">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://nikrun.dev/brand/nikrun-lockup-dark-mono.svg">
    <img alt="Nikrun" src="https://nikrun.dev/brand/nikrun-lockup-light.svg" height="52">
  </picture>
</p>

# Nikrun GitHub Action

[![Test Action](https://github.com/nikrun/github-setup-nik/actions/workflows/test.yml/badge.svg)](https://github.com/nikrun/github-setup-nik/actions/workflows/test.yml)
[![License: GPL-3.0-or-later](https://img.shields.io/badge/license-GPL--3.0--or--later-blue.svg)](./LICENSE)
[![Website: nikrun.dev](https://img.shields.io/badge/website-nikrun.dev-0b0e14)](https://nikrun.dev)

> **Official GitHub Action (`nikrun/github-setup-nik`) to install the standalone `nik` CLI for Free Pascal & Object Pascal in GitHub Actions workflows.**

[nikrun.dev](https://nikrun.dev) · [`nik` CLI](https://github.com/nikrun/cli) · [Free Pascal SDKs](https://github.com/nikrun/sdk) · [VS Code Extension](https://github.com/nikrun/vscode) · **[GitHub Action](https://github.com/nikrun/github-setup-nik)** · [Website & Docs](https://github.com/nikrun/web) · [Brand Assets](https://github.com/nikrun/brand) · [Build Library](https://github.com/nikrun/build-lib)

---

**Nikrun GitHub Action** (`nikrun/github-setup-nik`) installs the standalone **`nik` CLI** ([`nikrun/cli`](https://github.com/nikrun/cli)) into user scope (`~/.nikrun`) across Linux, macOS, and Windows (`x86_64` & `aarch64`) and adds `~/.nikrun/bin` to `GITHUB_PATH`.

The action intentionally stays minimal: it invokes the official SHA-256-verified installer endpoints (`https://nikrun.dev/install.sh` and `https://nikrun.dev/install.ps1`) and delegates all Free Pascal SDK and package provisioning directly to `nik` itself (for example on `nik build`).

---

## Quick Start

```yaml
name: Build

on: [push, pull_request]

jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}

    steps:
      - uses: actions/checkout@v4

      - uses: nikrun/github-setup-nik@v1

      - run: nik build -p release --locked
```

Because `nik build` automatically downloads and SHA-256-verifies both the Free Pascal SDK declared in `nik.yaml` (`toolchain.fpc`) and any locked packages (`nik.lock`), no separate `nik sdk install` step is needed.

### Caching SDKs & Packages Across Runs

Combine `nikrun/github-setup-nik` with [`actions/cache`](https://github.com/actions/cache) to persist `~/.nikrun/sdks` and `~/.nikrun/cache/pkg` across workflow runs:

```yaml
      - uses: actions/checkout@v4

      - uses: nikrun/github-setup-nik@v1

      - name: Cache Nikrun SDKs & Packages
        uses: actions/cache@v4
        with:
          path: |
            ~/.nikrun/sdks
            ~/.nikrun/cache/pkg
          key: nikrun-${{ runner.os }}-${{ runner.arch }}-${{ hashFiles('nik.yaml', 'nik.lock') }}
          restore-keys: |
            nikrun-${{ runner.os }}-${{ runner.arch }}-

      - run: nik build -p release --locked
```

---

## Inputs & Outputs

### Inputs

| Input | Default | Description |
| :--- | :--- | :--- |
| `channel` | `stable` | Release channel to install (`stable` or `insider`). |
| `version` | `""` | Specific `nik` CLI version or release tag (e.g. `0.1.0` or `v0.1.0`). When omitted, installs the latest release in `channel`. |
| `nikrun-home` | `""` | Custom `NIKRUN_HOME` directory (defaults to `~/.nikrun`). When provided, `NIKRUN_HOME` is also exported to `GITHUB_ENV` for subsequent steps. |
| `github-token` | `${{ github.token }}` | GitHub token passed to the installer to avoid unauthenticated GitHub Releases API rate limits. |

### Outputs

| Output | Description |
| :--- | :--- |
| `version` | Resolved `nik` CLI version tag reported by `nik --version`. |
| `nik-path` | Absolute path to the installed `nik` (`nik.exe` on Windows) binary. |
| `nikrun-home` | Resolved `NIKRUN_HOME` root directory. |

---

## Supported Runners

| OS | `x86_64` | `aarch64` / `arm64` |
| :--- | :--- | :--- |
| **Linux** | `ubuntu-latest`, `ubuntu-24.04` | `ubuntu-24.04-arm` |
| **macOS** | `macos-15-intel` | `macos-latest` |
| **Windows** | `windows-latest` | `windows-11-arm` |

---

## License

Copyright (c) 2026 The Nikrun Contributors.

Licensed under the **GNU General Public License v3.0 or later** (`GPL-3.0-or-later`) — see [LICENSE](./LICENSE).
