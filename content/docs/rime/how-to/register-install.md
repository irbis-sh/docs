---
title: Register and install your first endpoint
weight: 10
description: Learn how to register your organization, create policies, generate deployment tokens, and install the Zen Enterprise Agent on your first endpoint.
---

Get started by creating your organization, configuring a policy, and deploying your first endpoint.

Before proceeding, we recommend familiarizing yourself with key concepts in our [Glossary](/docs/enterprise/reference/glossary).

{{% steps %}}

### Create your organization account

1. Go to the [Zen Enterprise Dashboard](https://dash.irbis.sh).
2. Register a new administrator account or sign in if you already have one.

After registration, you will have access to the admin dashboard.

### Create your first policy

1. Open [Policies](https://dash.irbis.sh/policies).
2. Click __Create policy__.
3. Give it a name (e.g., "Default").
4. Navigate to the newly created policy by clicking its card in the list.
5. Add one or more filter lists:
    - Click __"Add filter list"__.
    - Choose from the built-in presets or add a custom URL.

> [!TIP]
> See our [Recommended filter lists](/docs/enterprise/reference/recommended-filter-lists) for suggestions on effective filter lists to use.

You now have a baseline configuration your endpoints will use.

### Generate a deployment token

1. Navigate to [Deployment tokens](https://dash.irbis.sh/deployment-tokens).
2. Click __Create token__.
3. Name the token (e.g., "IT Department").
4. Select one or more policies to assign to endpoints that will use this token.
5. Save the token.

### Install the agent on an endpoint

{{< tabs items="Windows,macOS,Linux" >}}
  {{< tab >}}
    1. Open __PowerShell as Administrator__.
    2. Run (replace `<YOUR-TOKEN-HERE>` with the token from the previous step):
        ```powershell
        Set-ExecutionPolicy Bypass -Scope Process -Force;
        ([System.Text.Encoding]::UTF8.GetString((Invoke-WebRequest -Uri 'https://dl.irbis.sh/install.ps1' -UseBasicParsing).Content)) | 
          Set-Content "$env:TEMP\Install-ZenEnterpriseAgent.ps1";
        Invoke-Expression "& '$env:TEMP\Install-ZenEnterpriseAgent.ps1' -DeploymentToken '<YOUR-TOKEN-HERE>'"
        ```
    3. Restart the computer.

    The script downloads and installs the agent, register the endpoint with your organization, and begins enforcing the assigned policies.

    Note the Endpoint ID shown at the end of the installation process – you can use it to identify the endpoint in the dashboard.
  {{< /tab >}}
  {{< tab >}}
    The macOS agent is a work in progress.
  {{< /tab >}}
  {{< tab >}}
    The Linux agent is a work in progress.
  {{< /tab >}}
{{< /tabs >}}

### Verify the endpoint in the dashboard

1. Open [Endpoints](https://dash.irbis.sh/endpoints).
2. Confirm the newly registered endpoint shows:
  - Policies assigned from the deployment token.
  - Hostname.
  - Endpoint ID.

### Test policy enforcement

On the deployed endpoint:

1. Visit a website known to serve ads or trackers.
2. Confirm that unwanted content is blocked.

{{% /steps %}}

## Next steps

- Add more policies for different departments.
- Create additional deployment tokens for segmented rollouts.
- Roll out the agent fleet-wide using RMM/MDM tools.
