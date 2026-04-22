# Exercise - Deploy a Bicep file from Azure Pipelines

**Status:** Completed  
**XP:** 100 XP  
**Duration:** 5 minutes

Now that you know how to validate, compile, and deploy your resources from your local environment, it's time to extend that and see how to bring that into an Azure Pipeline to streamline your deployment process even further.

## Prerequisites

You'll need the following before you begin:

- An **Azure Subscription** (if you don't have one, [create a free account](https://azure.microsoft.com/pricing/purchase-options/azure-account))
- An **Azure DevOps organization** (if you don't have one, [create one for free](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/pipelines-sign-up/))
- A configured [service connection](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure/) in your project linked to your Azure subscription
- The Bicep file you created earlier pushed to your Azure Repository

## Creating the pipeline

### Create a new pipeline

1. From within your Azure DevOps project, select **Pipelines** > **New pipeline**
2. Select **Azure Repos Git (YAML)** and specify your Azure Repo as a source
3. Select the **Starter pipeline** template from the list

### Configure the pipeline YAML

Replace everything in the starter pipeline file with the following:

```yaml
trigger:
  - main

name: Deploy Bicep files

variables:
  vmImageName: "ubuntu-latest"

  azureServiceConnection: "myServiceConnection"
  resourceGroupName: "Bicep"
  location: "eastus"
  templateFile: "main.bicep"

pool:
  vmImage: $(vmImageName)

steps:
  - task: AzureCLI@2
    inputs:
      azureSubscription: $(azureServiceConnection)
      scriptType: bash
      scriptLocation: inlineScript
      inlineScript: |
        az --version
        az group create --name $(resourceGroupName) --location $(location)
        az deployment group create --resource-group $(resourceGroupName) --template-file $(templateFile)
```

### Understanding the pipeline

This YAML pipeline performs the following actions:

- **Trigger:** Automatically runs when code is pushed to the `main` branch
- **Variables:** Defines reusable values for the pipeline:
  - `vmImageName`: Specifies the build agent image (Ubuntu)
  - `azureServiceConnection`: Name of your Azure service connection
  - `resourceGroupName`: Target resource group for deployment
  - `location`: Azure region for resources
  - `templateFile`: Path to the Bicep file
- **Pool:** Uses the specified Ubuntu virtual machine image
- **AzureCLI task:** Executes Azure CLI commands:
  - `az --version`: Verifies Azure CLI version
  - `az group create`: Creates the resource group
  - `az deployment group create`: Deploys the Bicep template

> **Important:** Don't forget to replace the `azureServiceConnection` value with your service connection name.

### Save and run

1. Select **Save and run** to create a new commit in your repository containing the pipeline YAML file
2. The pipeline will then run automatically
3. Wait for the pipeline to finish running and check the status

### Verify deployment

Once the pipeline runs successfully, you should be able to see the resource group and the storage account in the Azure portal.

## Next steps

Head over to the next unit to learn about deploying Bicep files from GitHub workflows.

**Reference:** [Microsoft Learn - Deploy a Bicep file from Azure Pipelines](https://learn.microsoft.com/en-us/training/modules/implement-bicep/6-exercise-deploy-bicep-file-azure-pipelines)
