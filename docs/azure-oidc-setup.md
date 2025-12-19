# Azure OIDC Configuration Guide

This guide provides step-by-step instructions for configuring Azure OpenID Connect (OIDC) authentication for GitHub Actions workflows in this repository.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Create Azure AD App Registration](#step-1-create-azure-ad-app-registration)
- [Step 2: Configure Federated Credentials](#step-2-configure-federated-credentials)
- [Step 3: Assign Azure Permissions](#step-3-assign-azure-permissions)
- [Step 4: Configure GitHub Repository Secrets](#step-4-configure-github-repository-secrets)
- [Step 5: Verify Configuration](#step-5-verify-configuration)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Overview

Azure OIDC allows GitHub Actions workflows to authenticate with Azure without storing long-lived credentials as secrets. Instead, GitHub Actions exchanges a short-lived token with Azure using OpenID Connect, providing enhanced security.

This repository includes a validation workflow (`azure-oidc-check.yml`) that you can use to verify your OIDC configuration is working correctly.

## Prerequisites

Before you begin, ensure you have:

- **Azure Subscription**: An active Azure subscription where you have appropriate permissions
- **Azure CLI**: Installed locally for running commands ([Install Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli))
- **Azure Permissions**: Ability to create App Registrations and assign roles (typically requires Contributor or Owner role)
- **GitHub Repository**: Admin access to the GitHub repository where you want to configure OIDC
- **GitHub Repository Settings**: Ensure your repository has Actions enabled

## Step 1: Create Azure AD App Registration

1. **Sign in to Azure Portal**:
   ```bash
   az login
   ```

2. **Create a new App Registration**:
   ```bash
   az ad app create --display-name "GitHub-OIDC-YourRepoName"
   ```

   Take note of the `appId` (also called Client ID) from the output, then set it as an environment variable:
   ```bash
   export APP_ID="<your-app-id-from-output>"
   ```

3. **Create a Service Principal** for the app:
   ```bash
   az ad sp create --id $APP_ID
   ```

4. **Retrieve your Tenant ID and Subscription ID**:
   ```bash
   # Get Tenant ID
   az account show --query tenantId -o tsv
   
   # Get Subscription ID
   az account show --query id -o tsv
   ```

   Save these values - you'll need them for GitHub secrets configuration.

## Step 2: Configure Federated Credentials

Federated credentials establish the trust relationship between GitHub and Azure.

1. **Set environment variables** for easier configuration:
   ```bash
   export APP_ID="<your-app-id>"
   export GITHUB_ORG="<your-github-org>"
   export GITHUB_REPO="<your-repo-name>"
   ```

2. **Create a federated credential** for the main branch:
   ```bash
   az ad app federated-credential create \
     --id $APP_ID \
     --parameters '{
       "name": "github-federated-credential",
       "issuer": "https://token.actions.githubusercontent.com",
       "subject": "repo:'"$GITHUB_ORG"'/'"$GITHUB_REPO"':ref:refs/heads/main",
       "audiences": ["api://AzureADTokenExchange"]
     }'
   ```

3. **(Optional) Add federated credentials for other branches or environments**:
   
   For pull requests:
   ```bash
   az ad app federated-credential create \
     --id $APP_ID \
     --parameters '{
       "name": "github-pr-credential",
       "issuer": "https://token.actions.githubusercontent.com",
       "subject": "repo:'"$GITHUB_ORG"'/'"$GITHUB_REPO"':pull_request",
       "audiences": ["api://AzureADTokenExchange"]
     }'
   ```

   For specific environments:
   ```bash
   az ad app federated-credential create \
     --id $APP_ID \
     --parameters '{
       "name": "github-env-credential",
       "issuer": "https://token.actions.githubusercontent.com",
       "subject": "repo:'"$GITHUB_ORG"'/'"$GITHUB_REPO"':environment:production",
       "audiences": ["api://AzureADTokenExchange"]
     }'
   ```

## Step 3: Assign Azure Permissions

The service principal needs appropriate permissions to access Azure resources.

1. **Assign a role to the service principal**:
   
   First, set your subscription ID as a variable (if not already set):
   ```bash
   export SUBSCRIPTION_ID=$(az account show --query id -o tsv)
   ```
   
   For read-only access:
   ```bash
   az role assignment create \
     --assignee $APP_ID \
     --role Reader \
     --scope /subscriptions/$SUBSCRIPTION_ID
   ```

   For contributor access (allows resource creation/modification):
   ```bash
   az role assignment create \
     --assignee $APP_ID \
     --role Contributor \
     --scope /subscriptions/$SUBSCRIPTION_ID
   ```

2. **Verify role assignment**:
   ```bash
   az role assignment list --assignee $APP_ID --output table
   ```

## Step 4: Configure GitHub Repository Secrets

Add the following secrets to your GitHub repository:

1. **Navigate to GitHub Repository Settings**:
   - Go to your repository on GitHub
   - Click **Settings** → **Secrets and variables** → **Actions**

2. **Add the following repository secrets**:

   | Secret Name | Description | How to Get |
   |------------|-------------|------------|
   | `AZURE_CLIENT_ID` | Application (client) ID | From Step 1, or run `echo $APP_ID` |
   | `AZURE_TENANT_ID` | Azure AD Tenant ID | From Step 1, or run `az account show --query tenantId -o tsv` |
   | `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID | From Step 1, or run `az account show --query id -o tsv` |

3. **Click "New repository secret"** for each value and enter:
   - **Name**: The secret name (e.g., `AZURE_CLIENT_ID`)
   - **Value**: The corresponding value from your Azure configuration
   - Click **Add secret**

> ⚠️ **Important**: Never commit these values to your repository or share them publicly.

## Step 5: Verify Configuration

This repository includes a workflow to validate your Azure OIDC configuration.

1. **Navigate to Actions tab** in your GitHub repository

2. **Select "Azure OIDC Connectivity Check"** from the workflows list

3. **Click "Run workflow"** → **Run workflow** (on your desired branch)

4. **Monitor the workflow run**:
   - ✅ If successful, your Azure OIDC is configured correctly
   - ❌ If it fails, check the [Troubleshooting](#troubleshooting) section below

The workflow performs the following checks:
- Verifies all required secrets are configured
- Attempts to authenticate with Azure using OIDC
- Runs `az account show` to confirm connectivity

## Troubleshooting

### Error: "AADSTS70021: No matching federated identity record found"

**Cause**: The federated credential subject doesn't match the GitHub context.

**Solution**:
- Verify the subject in your federated credential matches your repository structure:
  ```
  repo:OWNER/REPO:ref:refs/heads/BRANCH
  ```
- Ensure you're running the workflow from the branch specified in the credential
- Check for typos in organization or repository name

### Error: "AZURE_CLIENT_ID secret is not configured"

**Cause**: Required secrets are missing from GitHub repository settings.

**Solution**:
- Go to Repository Settings → Secrets and variables → Actions
- Verify all three secrets exist: `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`
- Ensure secret names match exactly (case-sensitive)

### Error: "Authorization failed"

**Cause**: The service principal doesn't have sufficient permissions.

**Solution**:
- Verify role assignments: `az role assignment list --assignee $APP_ID`
- Ensure the service principal has at least Reader role on the subscription
- Wait a few minutes after creating role assignments (propagation delay)

### Error: "Subscription not found"

**Cause**: Incorrect subscription ID or service principal doesn't have access.

**Solution**:
- Verify subscription ID: `az account show --query id -o tsv`
- Check if the subscription is active: `az account list -o table`
- Ensure the service principal has been assigned a role in the subscription

### Workflow succeeds but Azure CLI commands fail

**Cause**: Service principal lacks specific permissions for resources or operations.

**Solution**:
- Review the specific permission required for the operation
- Assign appropriate role (e.g., Contributor instead of Reader)
- Consider using custom roles for least-privilege access

## Additional Resources

- **Azure Documentation**: [Configure OpenID Connect in Azure](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure)
- **GitHub Documentation**: [Security hardening with OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- **azure/login Action**: [GitHub Marketplace](https://github.com/marketplace/actions/azure-login)
- **README**: See the [Post-Creation Checklist](../README.md#post-creation-checklist) for context on when to configure Azure OIDC

---

**Need Help?** If you encounter issues not covered in this guide, consider:
- Reviewing Azure AD App Registration and Federated Credentials in Azure Portal
- Checking GitHub Actions workflow logs for detailed error messages
- Consulting the Azure CLI documentation for specific commands
