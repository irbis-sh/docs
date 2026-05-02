---
title: Uninstall the agent from an endpoint
weight: 20
description: Instructions for uninstalling the Zen Enterprise agent from Windows, macOS, and Linux endpoints.
---

Learn how to uninstall the agent.

{{< tabs items="Windows,macOS,Linux" >}}
  {{< tab >}}
    1. Open __PowerShell as Administrator__.
    2. Run:
        ```powershell
        Set-ExecutionPolicy Bypass -Scope Process -Force;
        ([System.Text.Encoding]::UTF8.GetString((Invoke-WebRequest -Uri 'https://dl.irbis.sh/uninstall.ps1' -UseBasicParsing).Content)) |
          Set-Content "$env:TEMP\Uninstall-ZenEnterpriseAgent.ps1";
        Invoke-Expression "& '$env:TEMP\Uninstall-ZenEnterpriseAgent.ps1'"
        ```
    3. Restart the computer.

    The uninstaller stops and removes the service, unregisters the endpoint (if the executable is still present), and deletes all program files.
  {{< /tab >}}
  {{< tab >}}
    The macOS agent is a work in progress.
  {{< /tab >}}
  {{< tab >}}
    The Linux agent is a work in progress.
  {{< /tab >}}
{{< /tabs >}}
