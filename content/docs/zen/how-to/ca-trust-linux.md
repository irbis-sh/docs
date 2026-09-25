---
title: Trust Zen's CA certificate on Linux
linkTitle: CA certificate on Linux
description: Make apps other than browsers trust Zen's CA certificate on NixOS and other Linux systems.
---

This guide is on how to make apps other than browsers trust Zen's CA certificate on systems where Zen can't add it to the system trust store, such as NixOS.

Until you complete this guide, apps that follow your system proxy settings but don't use your browsers' certificate databases will show certificate errors when Zen is active.

## NixOS

1. Copy `~/.local/share/zen/certs/rootCA.pem` into your system configuration folder, e.g. as `zen-ca.pem`.

   > [!WARNING]
   > Copy only `rootCA.pem`, **not** `rootCA-key.pem`.

2. Add the certificate to your configuration:
   ```nix
   security.pki.certificateFiles = [ ./zen-ca.pem ];
   ```
3. Rebuild:
   ```
   sudo nixos-rebuild switch
   ```

If you use flakes, run `git add zen-ca.pem` before rebuilding, because flakes ignore files that git doesn't track. Alternatively, paste the contents of `rootCA.pem` into your configuration directly:

```nix
security.pki.certificates = [
  ''
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----
  ''
];
```

If you uninstall the CA in Zen's settings, Zen creates a new certificate the next time you start the proxy. Repeat the steps above with the new `rootCA.pem`.

To remove Zen's certificate completely, uninstall the CA in Zen, then delete the entry from your configuration and rebuild.

## Other distributions

Add `rootCA.pem` to the system trust store as your distribution's documentation describes.

## Check that it works

1. In Zen, click **Stop**. Go to **Settings** and set **Proxy port** to a fixed value, e.g. `28733`.
2. Quit Zen from its tray menu and open it again.
3. Click **Start**. The warning about the missing trust store should no longer appear.
4. Run:
   ```
   curl -v -x http://127.0.0.1:28733 https://example.com
   ```
   The request should succeed, and the `issuer:` line should show `Zen Personal CA`.

> [!NOTE]
> Apps that ship their own list of trusted certificates, such as Python programs that use certifi, Java apps and some Flatpak apps, may still not trust Zen's certificate.
