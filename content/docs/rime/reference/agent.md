---
title: Agent
weight: 20
description: How the Zen Enterprise Agent installs, proxies traffic, and enforces policies with secure local HTTPS inspection.
---

Learn what the agent does on your endpoints.

## Installation

{{< tabs items="Windows,macOS,Linux" >}}
  {{< tab >}}
    The installation script (`install.ps1`) performs the following steps:

    1. Creates application folders under `C:\Program Files\Irbis\Zen Enterprise Agent` and `C:\ProgramData\Irbis\Zen Enterprise Agent`.
    2. Assigns the appropriate ACLs to the `ProgramData` folder.
    3. Downloads the `ZenEnterpriseAgent.exe` binary into `C:\Program Files\Irbis\Zen Enterprise Agent`.
    4. Registers the endpoint using the specified deployment token and stores the configuration in `ProgramData`.
    5. Creates and starts a [Windows service](https://en.wikipedia.org/wiki/Windows_service) named `ZenEnterpriseAgent`.
  {{< /tab >}}
  {{< tab >}}
    The macOS agent is a work in progress.
  {{< /tab >}}
  {{< tab >}}
    The Linux agent is a work in progress.
  {{< /tab >}}
{{< /tabs >}}

## Request proxying

> [!NOTE]
> All HTTP/HTTPS traffic inspection and modification happens __locally within the agent__, in real time. No data is routed or sent to a remote server for processing.

### PAC (Proxy auto-config)

For HTTP/HTTPS request proxying, the agent hosts a [PAC file](https://en.wikipedia.org/wiki/Proxy_auto-config) on an available local port.

Some hostnames are automatically excluded from proxying due to security or compatibility reasons. For more details, see [`irbis-sh/zen-https-exclusions`](https://github.com/irbis-sh/zen-https-exclusions) on GitHub.

### System configuration

{{< tabs items="Windows,macOS,Linux" >}}
  {{< tab >}}
    To instruct applications to use the proxy, the agent modifies the following Registry values:
    
    `HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\CurrentVersion\Internet Settings`:
    - `ProxySettingsPerUser` set to `0`.

    `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Internet Settings`:
    - `ProxyEnable` set to `0`.
    - `ProxyOverride` deleted.
    - `ProxyServer` deleted.
    - `AutoConfigURL` set to the local PAC URL.

    When the service shuts down or is uninstalled, all of these values are deleted.
  {{< /tab >}}
  {{< tab >}}
    The macOS agent is a work in progress.
  {{< /tab >}}
  {{< tab >}}
    The Linux agent is a work in progress.
  {{< /tab >}}
{{< /tabs >}}

### CA (Certificate Authority)

Zen's content blocking and filtering features require HTTPS inspection, so the agent generates and installs a local CA (Certificate Authority) certificate that the operating system and browsers trust. The following measures protect the certificate from misuse:
- The certificate key pair is generated entirely on the endpoint.
- The private key is never sent to any remote server.
- The private key is stored with restrictive permissions (`0600`) to reduce the risk of compromise by malicious processes on the same machine.
