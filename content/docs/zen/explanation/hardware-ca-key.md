---
title: About the hardware-backed CA key
linkTitle: Hardware-backed CA key
description: About the hardware-backed CA key setting in Zen - what it protects against, and what to know before turning it on.
---

> [!NOTE]
> **TL;DR:** The setting keeps Zen's CA key in your computer's security chip, where it can't be copied. We recommend most people give it a go. If you run into issues, please [report them here](https://github.com/irbis-sh/zen-desktop/discussions/new?category=issue-triage).

With the **Hardware-backed CA key** setting, Zen creates the private key of its certificate authority (CA) inside security hardware built into your computer, instead of keeping it in a file on disk. The hardware is the [Secure Enclave](https://support.apple.com/guide/security/sec59b0b31ff/web) on a Mac, or the [TPM](https://learn.microsoft.com/en-us/windows/security/hardware-security/tpm/trusted-platform-module-overview) on a Windows PC.

The setting is experimental and not available on Linux.

## Why Zen has a CA key

To filter HTTPS traffic, Zen creates a certificate for each site you visit. Your browser accepts these certificates because Zen adds its root CA to your system's list of trusted authorities when you first start the proxy.

Whoever holds the root CA's private key can create a certificate for any site, which your computer will accept. The key never leaves your machine in normal use, but by default it is an ordinary file in Zen's data folder, `rootCA-key.pem`.

Sometimes, files get leaked. Say a backup tool uploads your user folder, including that key, to cloud storage. Anyone who later gets into that storage can pose as any website to your computer, as long as they can also get between you and the internet, for example on the same Wi-Fi. For practical purposes the key never expires, so the risk lasts until you uninstall Zen's CA.

## What changes when the key is in hardware

With the setting on, Zen asks the Secure Enclave or the TPM to create the root key. The key is stored encrypted by the chip, so it's useless on any other computer. Zen asks the chip to sign with the key, but neither Zen nor any other program can read it out. There is no `rootCA-key.pem` to steal.

Signing in the chip takes significant time, up to hundreds of milliseconds on some TPMs. The chip also doesn't support parallel operations, so if Zen used it for every site, a page that loads content from twenty third-party hosts would wait for twenty signatures in a row. So, each time you start the proxy, and every 24 hours after that, the chip signs an *intermediate* certificate, and Zen uses the intermediate to sign the certificates for individual sites. The intermediate's key lives only in Zen's memory, and its certificate expires after 24 hours.

## What it protects against

In general, the setting protects against the key leaking through:
- Backups
- Cloud sync
- Disk images
- A drive lost without disk encryption
- Generic non-persistent malware that targets the filesystem, such as a malicious installer that scans your files once

## What it doesn't protect against

On **Windows**, any program running under your user account can ask the TPM to sign with Zen's key. Malware can't copy the key, but it doesn't need to: it can use the TPM to create its own intermediate certificate that lasts for years and send that away instead. In practice, this setting on Windows guards against leaked files, not against long-running or purpose-made malware.

On **macOS**, only Zen can use the key. Other programs can't sign with it, so it would take a serious vulnerability in Zen to sign an unwanted certificate. However, as on Windows, a single signature is enough to create a long-lived intermediate and send it away. Someone with administrator access doesn't need Zen's key at all: they can install a CA of their own.

## Things to know before turning it on

We recommend turning the setting on for most people, as it significantly improves the security of Zen's CA. Before you do, please familiarise yourself with the following.

**Switching creates a new CA.** A key can't be moved into the hardware or out of it, so Zen uninstalls the current CA and creates a new one the next time you start the proxy.

**The key belongs to this device.** It is tied to the chip it was created in, so it's lost when you move to a new Mac with Migration Assistant, restore a backup onto a different computer, erase the disk, or clear the TPM.

**TPMs can break in unexpected ways.** Firmware updates on some motherboards reset the TPM, and so can a motherboard replacement or reset. Some TPMs are also unreliable in everyday use. Any of these can erase the key or make it unusable, and Zen then can't start the proxy until you uninstall the CA and it creates a new one.

**It's experimental.** The setting has been tested on a limited range of machines. If something goes wrong, you can turn it off to return to a key on disk. Please also [report the problem](https://github.com/irbis-sh/zen-desktop/discussions/new?category=issue-triage), so we can potentially fix it.
