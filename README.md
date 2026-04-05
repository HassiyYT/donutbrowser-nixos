# donutbrowser-nixos

Nix flake packaging [Donut Browser](https://github.com/zhom/donutbrowser) for `x86_64-linux`.

This repository currently exposes:

- `packages.x86_64-linux.{default,donutbrowser}`
- `apps.x86_64-linux.{default,donutbrowser}`
- `overlays.default`

It does not provide a NixOS module or a Home Manager module. For system-wide or
Home Manager installs, use the package output directly.

## Requirements

- Linux on `x86_64`
- Nix with flakes enabled

Canonical flake reference:

```bash
github:HassiyYT/donutbrowser-nixos
```

If you already cloned this repository, replace that reference with `.` in the
examples below.

## Try Without Installing

Run the packaged app directly:

```bash
nix run github:HassiyYT/donutbrowser-nixos#donutbrowser
```

From a local checkout:

```bash
nix run .#donutbrowser
```

## Install For One User

Install Donut Browser into your current Nix profile:

```bash
nix profile install github:HassiyYT/donutbrowser-nixos#donutbrowser
```

From a local checkout:

```bash
nix profile install .#donutbrowser
```

The installed executable is:

```bash
donutbrowser
```

## Install System-Wide On NixOS

Add the flake as an input in your system flake:

```nix
{
  inputs.donutbrowser.url = "github:HassiyYT/donutbrowser-nixos";
}
```

Then add the package to `environment.systemPackages`:

```nix
{ inputs, pkgs, ... }:
{
  environment.systemPackages = [
    inputs.donutbrowser.packages.${pkgs.system}.donutbrowser
  ];
}
```

### Optional Overlay Style

If you prefer using `pkgs.donutbrowser`, add the overlay first:

```nix
{ inputs, ... }:
{
  nixpkgs.overlays = [ inputs.donutbrowser.overlays.default ];
}
```

Then install it as a normal package:

```nix
{ pkgs, ... }:
{
  environment.systemPackages = [ pkgs.donutbrowser ];
}
```

## Install With Home Manager

Add the same flake input:

```nix
{
  inputs.donutbrowser.url = "github:HassiyYT/donutbrowser-nixos";
}
```

Then add the package to `home.packages`:

```nix
{ inputs, pkgs, ... }:
{
  home.packages = [
    inputs.donutbrowser.packages.${pkgs.system}.donutbrowser
  ];
}
```

### Optional Overlay Style

If you already use overlays in Home Manager:

```nix
{ inputs, ... }:
{
  nixpkgs.overlays = [ inputs.donutbrowser.overlays.default ];
}
```

Then:

```nix
{ pkgs, ... }:
{
  home.packages = [ pkgs.donutbrowser ];
}
```

## Wayland And X11

The package wrapper is tuned for Wayland by default.

On startup it sets:

- `MOZ_ENABLE_WAYLAND=1` if unset
- `GDK_BACKEND=wayland,x11` if unset
- `XDG_SESSION_TYPE=wayland` if unset

If `WAYLAND_DISPLAY` is unset but `XDG_RUNTIME_DIR` contains a Wayland socket,
the wrapper will pick one automatically.

### Wayland Users

For Wayland sessions, the default `donutbrowser` launch path is the recommended
configuration. The wrapper also removes stale bundled Wayland libraries from the
AppImage payload and preloads Nixpkgs Wayland libraries to avoid known crashes
with some Hyprland and Firefox combinations.

### X11 Users

X11 is supported too. In a normal X11 desktop session, your session usually
already exports `XDG_SESSION_TYPE=x11`, so `donutbrowser` should just start.

If you want to force X11 explicitly, launch it like this:

```bash
env \
  GDK_BACKEND=x11 \
  MOZ_ENABLE_WAYLAND=0 \
  XDG_SESSION_TYPE=x11 \
  DONUTBROWSER_DISABLE_WAYLAND_PRELOAD=1 \
  donutbrowser
```

## Troubleshooting And Runtime Knobs

### Startup Binary Cleanup Workaround

The wrapper protects downloaded browser binaries during startup to avoid an
upstream cleanup bug that can remove them too early.

- Default protection window: `8` seconds
- Change the window: `DONUTBROWSER_STARTUP_PROTECT_SECS=<seconds>`
- Disable the workaround entirely: `DONUTBROWSER_ALLOW_BINARY_CLEANUP=1`

### Wayland Library Preload

By default, the wrapper preloads Nixpkgs `libwayland-client` and
`libwayland-cursor` to avoid crashes caused by stale bundled libraries.

Disable that preload only if you need to debug or work around a local graphics
stack issue:

```bash
DONUTBROWSER_DISABLE_WAYLAND_PRELOAD=1 donutbrowser
```

## Binary Cache

The flake already declares the project's Cachix cache:

- `https://hassiyyt.cachix.org`
- `hassiyyt.cachix.org-1:GPb2J+eS5AyHtVF9zQ+cchuQJl65WrxpcrdYsSiDjno=`

If you want to trust it globally instead of relying on per-flake `nixConfig`,
add this to your NixOS configuration or equivalent Nix settings:

```nix
{
  nix.settings = {
    extra-substituters = [ "https://hassiyyt.cachix.org" ];
    extra-trusted-public-keys = [
      "hassiyyt.cachix.org-1:GPb2J+eS5AyHtVF9zQ+cchuQJl65WrxpcrdYsSiDjno="
    ];
  };
}
```

## Maintainer Notes

Build exactly what CI builds:

```bash
nix build .#donutbrowser --print-build-logs
```

Smoke-test the packaged binary after a build:

```bash
./result/bin/donutbrowser --version >/dev/null 2>&1 || true
```

Check for a newer upstream release:

```bash
./scripts/update-version.sh --check
```

Pin to a specific upstream release:

```bash
./scripts/update-version.sh --version 0.19.0
```

Repository automation setup and required GitHub settings are documented in
[`./.github/REPOSITORY_SETTINGS.md`](./.github/REPOSITORY_SETTINGS.md).
