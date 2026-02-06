# Azure Coding Agent Setup Guide

This guide provides step-by-step instructions for enabling AI agents (such as GitHub Copilot coding agent) to provision, deploy, and test against Azure autonomously—removing the need for manual human testing of `azd` workflows.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Install the azd Coding Agent Extension](#step-1-install-the-azd-coding-agent-extension)
- [Step 2: Configure Azure Access for the Coding Agent](#step-2-configure-azure-access-for-the-coding-agent)
- [Step 3: Add MCP Server Configuration](#step-3-add-mcp-server-configuration)
- [Step 4: Merge the Generated Pull Request](#step-4-merge-the-generated-pull-request)
- [Step 5: Verify the Setup](#step-5-verify-the-setup)
- [What the Extension Configures](#what-the-extension-configures)
- [Customizing the Copilot Setup Steps Workflow](#customizing-the-copilot-setup-steps-workflow)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Overview

By default, AI agents can write and locally test code but cannot provision or deploy to Azure. This creates a workflow bottleneck: a human must manually run `azd provision`, deploy, and verify functionality against Azure.

The [Azure Developer CLI Copilot Coding Agent Extension](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/extensions/copilot-coding-agent-extension) (`azure.coding-agent`) solves this by:

- Creating a managed identity with federated credentials (OIDC) for secure, secretless authentication
- Configuring a GitHub Actions environment (`copilot`) with the necessary Azure variables
- Generating a `copilot-setup-steps.yml` workflow that prepares the agent's runtime environment
- Providing MCP server configuration so the agent can interact with Azure resources

Once configured, AI agents can autonomously provision Azure resources, deploy applications, and run tests against deployed services.

## Prerequisites

Before you begin, ensure you have:

- **Azure OIDC configured**: Complete the [Azure OIDC setup](azure-oidc-setup.md) first (or at minimum, have an active Azure subscription with permissions to create resource groups and managed identities)
- **Azure Developer CLI (`azd`)**: Installed and authenticated. Verify with `azd version`.
  - Install: <https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd>
- **GitHub Copilot**: Access to GitHub Copilot coding agent (requires Copilot Pro, Business, or Enterprise)
- **Repository admin access**: Permissions to push workflow changes and configure GitHub environments
- **Local clone**: The repository must be cloned locally for `azd` CLI operations

## Step 1: Install the azd Coding Agent Extension

Install (or upgrade) the `azure.coding-agent` extension:

```bash
azd extension install azure.coding-agent
```

To upgrade an existing installation:

```bash
azd extension upgrade azure.coding-agent
```

Verify the extension is installed:

```bash
azd extension list
```

## Step 2: Configure Azure Access for the Coding Agent

From your local repository root, run:

```bash
azd coding-agent config
```

The command will interactively guide you through:

1. **Selecting your Azure subscription** — Choose the subscription for the managed identity
2. **Selecting or creating a resource group** — The managed identity will be placed here
3. **Creating a user-assigned managed identity** — Used for OIDC-based authentication
4. **Assigning RBAC roles** — Default is `Reader`; you can assign additional roles as needed for your scenario (e.g., `Contributor` for resource provisioning)
5. **Creating a federated credential** — Links the GitHub repository to the managed identity
6. **Configuring the GitHub environment** — Sets up a `copilot` environment in your repository with the required variables (`AZURE_CLIENT_ID`, `AZURE_SUBSCRIPTION_ID`, `AZURE_TENANT_ID`)
7. **Generating a workflow file** — Creates `.github/workflows/copilot-setup-steps.yml` on a new branch and opens a pull request

> **Note**: The `azd coding-agent config` command requires interactive authentication. Run it locally—it cannot be run by an AI agent.

## Step 3: Add MCP Server Configuration

After running `azd coding-agent config`, the CLI outputs an MCP server configuration JSON block. Copy this configuration and add it to your repository's GitHub Copilot MCP settings.

The output will look similar to:

```json
{
  "mcpServers": {
    "Azure": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@azure/mcp@latest", "server", "start"],
      "tools": ["*"]
    }
  }
}
```

To add this configuration:

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Copilot** → **Coding agent**
3. Under **MCP configuration**, paste the JSON block
4. Save the configuration

## Step 4: Merge the Generated Pull Request

The `azd coding-agent config` command creates a branch (typically `azd-enable-copilot-coding-agent-with-azure`) and opens a pull request that adds the `copilot-setup-steps.yml` workflow.

1. Review the pull request on GitHub
2. Verify the workflow file contents are appropriate for your project
3. Merge the pull request

## Step 5: Verify the Setup

After merging:

1. **Check the GitHub environment**: Navigate to **Settings** → **Environments** and confirm the `copilot` environment exists with the correct variables
2. **Verify the managed identity**: In the Azure Portal, confirm the managed identity has the expected role assignments
3. **Test with an AI agent**: Assign a task to the Copilot coding agent that requires Azure interaction (e.g., reading a resource group) and verify it succeeds

## What the Extension Configures

| Component | Description |
|-----------|-------------|
| **Managed identity** | A user-assigned managed identity in your Azure subscription |
| **Federated credential** | OIDC trust between the GitHub repository and the managed identity |
| **GitHub environment** | A `copilot` environment with `AZURE_CLIENT_ID`, `AZURE_SUBSCRIPTION_ID`, and `AZURE_TENANT_ID` |
| **Workflow file** | `.github/workflows/copilot-setup-steps.yml` — prepares the agent's runtime environment |
| **MCP configuration** | JSON config enabling the Azure MCP server for the Copilot coding agent |

## Customizing the Copilot Setup Steps Workflow

The generated `copilot-setup-steps.yml` workflow runs before each Copilot agent session. You can customize it to install additional tools or dependencies your project requires.

For example, to add Azure Developer CLI to the agent's environment:

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Install azd
    uses: Azure/setup-azd@v2
```

> **Important**: The job name in the workflow must be exactly `copilot-setup-steps`.

## Troubleshooting

### The Copilot agent cannot authenticate with Azure

**Possible causes**:
- The managed identity's federated credential does not match the repository or environment
- The `copilot` GitHub environment is missing or has incorrect variables
- The managed identity lacks sufficient RBAC permissions

**Solutions**:
- Re-run `azd coding-agent config` to reconfigure
- Verify the federated credential subject matches `repo:OWNER/REPO:environment:copilot`
- Check role assignments on the managed identity in the Azure Portal

### The `copilot-setup-steps.yml` workflow fails

**Possible causes**:
- Missing or misconfigured tools in the workflow
- Network issues during dependency installation

**Solutions**:
- Check the workflow run logs in the **Actions** tab
- Ensure all tools referenced in the workflow are available on the runner

### The MCP server is not available to the agent

**Possible causes**:
- MCP configuration was not saved in repository settings
- The `npx` command cannot find the `@azure/mcp` package

**Solutions**:
- Verify MCP configuration in **Settings** → **Copilot** → **Coding agent**
- Ensure the workflow installs Node.js if it is not available by default on the runner

## Additional Resources

- **Azure Developer CLI Extension**: [Connect GitHub Copilot coding agent with Azure MCP Server using azd](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/extensions/copilot-coding-agent-extension)
- **Azure SDK Blog**: [Introducing the azd extension to configure GitHub Copilot coding agent](https://devblogs.microsoft.com/azure-sdk/azure-developer-cli-copilot-coding-agent-config/)
- **GitHub Documentation**: [Extending Copilot coding agent with MCP](https://docs.github.com/en/copilot/using-github-copilot/coding-agent/extending-copilot-coding-agent-with-mcp)
- **Azure OIDC Setup**: See [docs/azure-oidc-setup.md](azure-oidc-setup.md) for foundational Azure OIDC configuration
