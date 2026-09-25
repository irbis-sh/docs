---
title: Install Zen via Nix
linkTitle: Install via Nix
weight: 30
description: Install Zen on NixOS and other Linux distributions using Nix.
---

Zen is available for Linux as a __Nix flake__. You need flakes enabled (`experimental-features = nix-command flakes`).

## Install

If you manage packages with `nix profile`, run:

```bash
nix profile install github:irbis-sh/zen-desktop
```

If you manage your NixOS or Home Manager configuration with a flake:

1. Add the input:
   ```nix
   inputs.zen-desktop.url = "github:irbis-sh/zen-desktop";
   ```
2. Add the package to `environment.systemPackages` (NixOS) or `home.packages` (Home Manager):
   ```nix
   inputs.zen-desktop.packages.${pkgs.stdenv.hostPlatform.system}.default
   ```
   This assumes your flake passes `inputs` to your modules through `specialArgs` (NixOS) or `extraSpecialArgs` (Home Manager).

   If you use overlays, add `inputs.zen-desktop.overlays.default` to `nixpkgs.overlays` and use `pkgs.zen-adblocker` instead.
3. Rebuild.

Start Zen from your app menu, or run `zen`.

If you use NixOS, click **Start** in Zen once, then follow [Trust Zen's CA certificate on Linux](/docs/zen/how-to/ca-trust-linux/) so that apps other than browsers work with Zen.

## Update

If you installed with `nix profile`, run:

```bash
nix profile upgrade zen-desktop
```

If you installed with a flake, run `nix flake update zen-desktop` and rebuild.

## Uninstall

1. In Zen, click **Stop**, go to **Settings** and click **Uninstall CA**.
2. Run `nix profile remove zen-desktop`, or remove Zen from your configuration and rebuild. If you added Zen's certificate to `security.pki`, remove that too before rebuilding.

## See also

- [Zen's `flake.nix`](https://github.com/irbis-sh/zen-desktop/blob/master/flake.nix)
