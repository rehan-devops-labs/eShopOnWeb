# AZ-400 Lab: Integrate Azure Key Vault with Azure DevOps

## Lab Overview

Azure Key Vault provides secure storage and management of sensitive data such as keys, passwords, and certificates. Integrating Azure Key Vault with Azure DevOps pipelines helps avoid exposing secrets in source code and enables controlled, auditable access at runtime.

In this lab, you will:

- Create an Azure Key Vault and store an Azure Container Registry (ACR) password as a secret.
- Grant the Azure DevOps service connection access to Key Vault secrets.
- Configure an Azure DevOps Variable Group linked to Key Vault.
- Configure CI/CD pipelines to build container images and deploy to Azure Container Instance (ACI) using the secret.

## Objectives

After completing this lab, you will be able to:

- Create an Azure Key Vault.
- Retrieve a secret from Azure Key Vault in an Azure DevOps pipeline.
- Use the secret in a subsequent task in the pipeline.
- Deploy a container image to Azure Container Instance (ACI) using the secret.
- Create a Variable Group connected to Azure Key Vault.

## Estimated Time

40 minutes

## Lab Requirements

- Microsoft Edge or another Azure DevOps-supported browser.
- Completed **Validate lab environment** setup, including:
  - Azure DevOps organization and project.
  - Azure service connection used by pipelines.
- Azure subscription available for resource deployment.

If you do not already have an Azure DevOps organization, create one by following Microsoft guidance for creating an organization or project collection.

## Exercise 0: Configure Lab Prerequisites (Skip if already done)

### Task 1: Create and configure the team project

1. Open your Azure DevOps organization in a browser.
2. Select **New Project**.
3. Name the project `eShopOnWeb`.
4. Keep default settings and select **Create**.

### Task 2: Import eShopOnWeb Git repository

1. In the `eShopOnWeb` project, go to **Repos > Files > Import**.
2. Import from this URL:

```text
https://github.com/MicrosoftLearning/eShopOnWeb.git
```

Repository structure used in this lab:

- `.ado` contains Azure DevOps YAML pipelines.
- `.devcontainer` contains container development setup.
- `infra` contains Bicep/ARM IaC templates.
- `.github` contains GitHub workflow definitions.
- `src` contains the .NET 8 sample app.

### Task 3: Set main as default branch

1. Go to **Repos > Branches**.
2. On branch `main`, open the ellipsis menu.
3. Select **Set as default branch**.

## Exercise 1: Set up CI pipeline to build eShopOnWeb container

### Task 1: Set up and run the CI pipeline

1. Go to **Pipelines > Pipelines** and select **Create Pipeline** (or **New pipeline**).
2. Select **Azure Repos Git (YAML)** and choose the `eShopOnWeb` repository.
3. Select **Existing Azure Pipelines YAML file**.
4. Set:
   - Branch: `main`
   - Path: `/.ado/eshoponweb-ci-dockercompose.yml`
5. Select **Continue**.
6. In the YAML, customize:
   - Replace `NAME` in `AZ400-EWebShop-NAME` with a unique suffix.
   - Replace `YOUR-SUBSCRIPTION-ID` with your Azure subscription ID.
7. Select **Save and Run**.
8. If needed, select **Save and Run** again to complete creation and execution.

Important:

- If prompted with permission message for resource access, open the build job, select **View**, then **Permit**, and **Permit** again.
- Build can take a few minutes.

CI pipeline tasks include:

- `AzureResourceManagerTemplateDeployment` to deploy ACR with Bicep.
- `PowerShell` task to capture Bicep output (ACR login server) into pipeline variables.
- `DockerCompose` task to build and push container images to ACR.

Rename the pipeline after first successful run:

1. Go to **Pipelines > Pipelines**.
2. Open the newly created pipeline.
3. Open ellipsis menu > **Rename/Remove**.
4. Rename to `eshoponweb-ci-dockercompose` and save.

Validation:

1. In Azure portal, open resource group `AZ400-EWebShop-NAME`.
2. Verify ACR exists and includes images:
   - `eshoppublicapi`
   - `eshopwebmvc`
3. Open **ACR > Access keys**.
4. Enable **Admin user** (if disabled).
5. Copy the password value for use as Key Vault secret.

### Task 2: Create Azure Key Vault

1. In Azure portal search for **Key vault** and select **Create > Key Vault**.
2. On **Basics**, set:

| Setting | Value |
|---|---|
| Subscription | Your lab subscription |
| Resource group | `AZ400-EWebShop-NAME` |
| Key vault name | Unique name, e.g. `ewebshop-kv-NAME` |
| Region | Region near your lab environment |
| Pricing tier | Standard |
| Days to retain deleted vaults | 7 |
| Purge protection | Disabled |

3. On **Access configuration**:
   - Select **Vault access policy**.
   - In **Access policies**, select **+ Create**.
4. In **Permissions**:
   - Under **Secret permissions**, select **Get** and **List**.
5. In **Principal**:
   - Select your Azure subscription service connection principal (commonly `azure subs`).
   - If prompted, use **Authorize** to create the policy automatically.
6. Complete policy creation (**Next > Next > Create**).
7. Select **Review + create > Create** for the key vault.
8. After deployment, go to the Key Vault resource.
9. Go to **Objects > Secrets > Generate/Import**.
10. Create secret with:

| Setting | Value |
|---|---|
| Upload options | Manual |
| Name | `acr-secret` |
| Secret value | ACR password copied earlier |

### Task 3: Create a Variable Group linked to Azure Key Vault

1. In Azure DevOps, go to **Pipelines > Library**.
2. Select **+ Variable Group**.
3. Configure:

| Setting | Value |
|---|---|
| Variable group name | `eshopweb-vg` |
| Link secrets from an Azure Key Vault | Enabled |
| Azure subscription | Service connection (e.g. `Azure subs`) |
| Key vault name | Your created key vault |

4. Under **Variables**, select **+ Add**.
5. Select secret `acr-secret` and confirm.
6. Save the variable group.

### Task 4: Set up CD pipeline to deploy to Azure Container Instance

1. Go to **Pipelines > Pipelines > New Pipeline**.
2. Select **Azure Repos Git (YAML)** and repository `eShopOnWeb`.
3. Select **Existing Azure Pipelines YAML file**.
4. Set:
   - Branch: `main`
   - Path: `/.ado/eshoponweb-cd-aci.yml`
5. Select **Continue**.
6. Update YAML values:
   - `YOUR-SUBSCRIPTION-ID` with your subscription ID.
   - `az400eshop-NAME` with a globally unique name.
   - `YOUR-ACR.azurecr.io` and `ACR-USERNAME` with your ACR details.
   - `AZ400-EWebShop-NAME` with your resource group.
7. Select **Save and Run** (repeat once if required for first creation/run).
8. Open the build job to review and approve permission prompts.

Important:

- If prompted that the pipeline needs permission to access resources, select **View > Permit > Permit**.
- Deployment may take a few minutes.

CD pipeline behavior includes:

- `resources` configured to support CI completion trigger and repository download for Bicep.
- `variables` for Deploy stage connected to `eshopweb-vg` variable group for `acr-secret`.
- `AzureResourceManagerTemplateDeployment` deploying ACI and passing ACR login parameters.

Validation:

1. In Azure portal, open resource group `AZ400-EWebShop-NAME`.
2. Verify the container instance `az400eshop` exists.

Rename the CD pipeline:

1. Open the newly created pipeline.
2. Ellipsis menu > **Rename/Remove**.
3. Rename to `eshoponweb-cd-aci` and save.

## Review

In this lab, you integrated Azure Key Vault with Azure DevOps by:

- Creating an Azure Key Vault and storing ACR password as a secret.
- Granting service connection access to Key Vault secrets.
- Linking secrets to Azure DevOps through a Variable Group.
- Retrieving the secret in a deployment pipeline.
- Deploying containerized workload to Azure Container Instance using that secret.

## Cleanup

To avoid unnecessary charges, delete lab resources when done:

- Resource group `AZ400-EWebShop-NAME`.
- Any associated ACI and ACR resources created for the lab.
- Optional: remove pipelines/variable groups if they were created only for practice.
