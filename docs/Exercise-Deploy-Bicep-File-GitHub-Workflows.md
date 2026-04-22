# Exercise - Deploy a Bicep file from GitHub workflows

**Status:** Completed  
**XP:** 100 XP  
**Duration:** 5 minutes

GitHub Actions are similar to Azure Pipelines in nature. They provide a way to automate software development and deployments. In this exercise, you'll learn how to deploy a Bicep file using a GitHub Action.

## Prerequisites

To follow along, you'll need:

- A **GitHub account** ([create for free here](https://github.com/join))
- A **GitHub repository** to store your Bicep file and workflows created in the [Exercise - Create Bicep templates](https://learn.microsoft.com/en-us/training/modules/implement-bicep/4-exercise-create-bicep-templates)
- Push the Bicep file into your GitHub repository
- An **Azure subscription** ([create for free here](https://azure.microsoft.com/pricing/purchase-options/azure-account))

## Creating a service principal in Azure

To deploy your resources to Azure, you'll need to create a service principal which GitHub can use. Open a terminal or use Cloud Shell in the Azure portal and run the following commands:

```bash
az login
az ad sp create-for-rbac --name myApp --role contributor --scopes /subscriptions/{subscription-id}/resourceGroups/Bicep --sdk-auth
```

> **Important:** Don't forget to replace `{subscription-id}` with your actual subscription ID.

### Understanding the command

- **az ad sp create-for-rbac:** Creates a service principal for role-based access control
- **--name:** Name of the service principal
- **--role:** Assigns the `contributor` role, allowing full management of resources
- **--scopes:** Limits access to a specific resource group
- **--sdk-auth:** Outputs credentials in a format suitable for SDK authentication

When the operation succeeds, it will output a JSON object containing your `tenantId`, `subscriptionId`, `clientId`, `clientSecret`, and other properties:

```json
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
}
```

Save this JSON object as you'll need to add it to your GitHub secrets.

## Creating a GitHub secret

In your GitHub repository, navigate to **Settings > Secrets > Actions**. For this lab, the repository is located at: https://github.com/rehan-devops-labs/eShopOnWeb/settings/secrets/actions

Create the following secrets:

1. **AZURE_CREDENTIALS:** Paste the entire JSON object from the previous step
2. **AZURE_RG:** The name of your resource group (e.g., `Bicep`)
3. **AZURE_SUBSCRIPTION:** Your Azure subscription ID

This is where we have added the secrets for this project.

### Why use secrets?

GitHub secrets store sensitive information securely:

- **Encrypted:** Stored securely in GitHub
- **Not visible in logs:** Masked in workflow run logs
- **Accessible in workflows:** Available as environment variables during workflow execution

## Creating a GitHub action

### Create a new workflow

1. Navigate to your GitHub repository and select the **Actions** menu
2. Set up a new workflow to create an empty workflow file
3. You can rename the file to a different name if you prefer

### Define the workflow

Replace the content of the file with the following snippet:

```yaml
on: [push]
name: Azure Resource Manager
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      # Checkout code
      - uses: actions/checkout@main

        # Log into Azure
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

        # Deploy Bicep file
      - name: deploy
        uses: azure/arm-deploy@v1
        with:
          subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
          resourceGroupName: ${{ secrets.AZURE_RG }}
          template: ./main.bicep
          parameters: storagePrefix=stg
          failOnStdErr: false
```

> **Tip:** Feel free to replace the `storagePrefix` parameter value with your own.

### Understanding the workflow

This GitHub Action workflow performs the following:

- **Trigger:** Runs automatically when code is pushed to the repository (`on: [push]`)
- **Job:** Defines a job named `build-and-deploy` that runs on an Ubuntu runner
- **Steps:**
  - **Checkout code:** Uses `actions/checkout@main` to check out the repository code
  - **Log into Azure:** Uses `azure/login@v1` to authenticate with Azure using the `AZURE_CREDENTIALS` secret
  - **Deploy Bicep file:** Uses `azure/arm-deploy@v1` to deploy the Bicep template:
    - `subscriptionId`: Azure subscription ID from secrets
    - `resourceGroupName`: Resource group name from secrets
    - `template`: Path to the Bicep file
    - `parameters`: Parameters passed to the Bicep template
    - `failOnStdErr`: Set to `false` to not fail on standard error output

> **Note:** The first part of the workflow defines the trigger and its name. The rest defines a job and uses tasks to check out the code, sign in to Azure, and deploy the Bicep file.

### Commit the workflow

1. Select **Start commit** and enter a title and description
2. Select **Commit directly to the main branch**
3. Select **Commit new file**

### Monitor the workflow

1. Navigate to the **Actions** tab
2. Select the newly created action that should be running

### Verify deployment

1. Monitor the workflow status
2. When the job is finished, check the Azure portal to verify that the storage account has been created

## Next steps

Now that you've learned how to deploy Bicep files from GitHub workflows, explore how to use Azure Bicep templates for more advanced deployment scenarios.

**Reference:** [Microsoft Learn - Deploy a Bicep file from GitHub workflows](https://learn.microsoft.com/en-us/training/modules/implement-bicep/7-exercise-deploy-bicep-file-github-workflows)
