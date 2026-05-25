# Azure GitHub Actions OIDC Login Setup — Complete Steps

This guide shows how to configure GitHub Actions to securely log in to Azure using OIDC (`azure/login@v2`) without storing passwords or client secrets.

## 1. Create an Azure App Registration

Open:
- Microsoft Entra Admin Center

Navigate to:
- Microsoft Entra ID
- Applications
- App registrations

Click **New registration**.

Example:
- **Name:** `github-actions-app`
- **Supported account types:** `Default`

Click **Register**.

## 2. Copy Required IDs

After creating the app, copy:

| Azure Field | GitHub Secret |
| --- | --- |
| Application (client) ID | `AZURE_CLIENT_ID` |
| Directory (tenant) ID | `AZURE_TENANT_ID` |

## 3. Get Azure Subscription ID

Open:
- Azure Portal

Go to:
- Subscriptions
- Select your subscription

Copy:

| Azure Field | GitHub Secret |
| --- | --- |
| Subscription ID | `AZURE_SUBSCRIPTION_ID` |

Example used:
- `692b1673-5e14-400e-a0af-9fd64a687149`

## 4. Configure Federated Credential (OIDC)

In the App Registration:

- Go to **Certificates & secrets**
- Click **Federated credentials**
- Click **Add credential**
- Choose **GitHub Actions deploying Azure resources**

Fill:

| Field | Value |
| --- | --- |
| Organization | `Ajithkumar8006` |
| Repository | `azure-integration` |
| Branch | `main` |

Azure automatically creates the subject:

```text
repo:Ajithkumar8006/azure-integration:ref:refs/heads/main
```

This enables GitHub Actions OIDC authentication.

## 5. Assign Azure RBAC Permission

Without this step, login succeeds but Azure shows:
- `No subscriptions found`

Run this command locally:

```bash
az role assignment create \
  --assignee 37c412d1-1970-474b-998f-4bcb763be224 \
  --role Contributor \
  --scope /subscriptions/692b1673-5e14-400e-a0af-9fd64a687149
```

This grants the Service Principal access to the Azure subscription.

Successful output includes:
- `"principalType": "ServicePrincipal"`
- `"roleDefinitionId"`

## 6. Add GitHub Repository Secrets

In GitHub repository:

- Settings
- Secrets and variables
- Actions
- New repository secret

Create these secrets:

| Secret Name | Value |
| --- | --- |
| `AZURE_CLIENT_ID` | App Registration Client ID |
| `AZURE_TENANT_ID` | Tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Subscription ID |

## 7. Create GitHub Actions Workflow

Create:
- `.github/workflows/azure-login.yml`

Workflow:

```yaml
name: Azure Login and Account Info

on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  azure-login:
    runs-on: ubuntu-latest
    steps:
      - name: Azure Login
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Get Azure account info
        shell: bash
        run: |
          az account show --output table
          echo "Successfully logged in to Azure"
```

## 8. Commit and Push Workflow

Run:

```bash
git add .
git commit -m "Add Azure login workflow"
git push origin main
```

## 9. Run GitHub Action

In GitHub:

- Open repository
- Go to **Actions**
- Select workflow **Azure Login and Account Info**
- Click **Run workflow**

## 10. Expected Successful Output

GitHub Actions logs should show:

```text
Attempting Azure CLI login by using OIDC...
```

Then:

```text
Successfully logged in to Azure
```

And output from:

```bash
az account show
```

Example:

```json
{
  "environmentName": "AzureCloud",
  "id": "692b1673-5e14-400e-a0af-9fd64a687149",
  "name": "Azure subscription 1",
  "tenantId": "84167a5d-823d-4a6a-a632-b2241e62e273",
  "user": {
    "type": "servicePrincipal"
  }
}
```

## Official Documentation

- Azure Login GitHub Action
- Azure OIDC Documentation
