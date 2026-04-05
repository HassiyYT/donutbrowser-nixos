# Repository Guidelines

## Project Structure & Module Organization
This repository is a small Nix flake for packaging Donut Browser on `x86_64-linux`. `flake.nix` exposes the package, app entrypoint, overlay, and development shell. `package.nix` contains the derivation, AppImage extraction, and runtime wrapper logic. `scripts/update-version.sh` updates the upstream version, asset name, and hash. CI and release automation live under `.github/workflows/`, with required GitHub settings documented in `.github/REPOSITORY_SETTINGS.md`.

## Build, Test, and Development Commands
Use Nix-native commands from the repo root:

- `nix develop` enters the dev shell with `gh`, `jq`, `cachix`, `nix`, and `nixpkgs-fmt`.
- `nix build .#donutbrowser --print-build-logs` builds the package exactly as CI does.
- `nix run .#donutbrowser` launches the packaged app locally.
- `./scripts/update-version.sh --check` checks whether a newer upstream release exists.
- `./scripts/update-version.sh --version 0.19.0` pins to a specific release; omit `--version` to update to latest.

## Coding Style & Naming Conventions
Match the existing style in touched files. Nix code uses two-space indentation, simple attrsets, and explicit semicolons; format Nix edits with `nixpkgs-fmt flake.nix package.nix`. Bash scripts should keep `#!/usr/bin/env bash`, `set -euo pipefail`, `snake_case` function names, and uppercase `readonly` constants for shared values. Keep changes narrow: packaging, wrapper behavior, and automation should stay easy to review.

## Testing Guidelines
There is no separate unit-test suite; validation is build plus smoke test. Before opening a PR, run `nix build .#donutbrowser --print-build-logs`, then verify `./result/bin/donutbrowser --version >/dev/null 2>&1 || true`. If you modify the updater, also run `./scripts/update-version.sh --check` from the repository root.

## Commit & Pull Request Guidelines
Follow the existing commit pattern from history: `chore: update donutbrowser to version 0.19.0` for version bumps, and `fix:` or `chore:` with an imperative summary for manual changes. PRs should state what changed, why it changed, and which commands you used to validate it. If you touch `package.nix`, `flake.nix`, `scripts/**`, or `.github/workflows/**`, expect CI to rebuild the package.

## Security & Configuration Tips
Do not commit secrets. `CACHIX_AUTH_TOKEN` belongs in GitHub Actions secrets only. Keep Cachix settings and workflow permissions aligned with `.github/REPOSITORY_SETTINGS.md` when changing CI or release automation.
