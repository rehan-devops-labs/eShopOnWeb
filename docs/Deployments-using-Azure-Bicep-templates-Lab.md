# Deployments using Azure Bicep templates - Lab

**Duration:** 45 minutes

This lab guides you through creating an Azure Bicep template, modularizing it using Bicep modules, and deploying resources to Azure using Azure DevOps YAML pipelines.

## Lab Requirements

- Microsoft Edge or an [Azure DevOps supported browser](https://docs.microsoft.com/azure/devops/server/compatibility)
- Azure DevOps organization ([create one if needed](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization))
- Azure subscription
- Microsoft account or Microsoft Entra account with:
  - Owner role in the Azure subscription
  - Global Administrator role in the Microsoft Entra tenant

## Lab Overview

In this lab, you will:

1. Understand an Azure Bicep template's structure
2. Create a reusable Bicep module
3. Modify the main template to use the module
4. Deploy all resources to Azure using Azure YAML pipelines

## Objectives

After completing this lab, you will be able to:

- Understand Azure Bicep template structure
- Create reusable Bicep modules
- Modify main templates to reference modules
- Deploy templates using YAML pipelines in Azure DevOps

## Exercise 0: Configure Lab Prerequisites

### Task 1: Create and Configure the Team Project

1. In your Azure DevOps organization, click **New Project**
2. Name the project **eShopOnWeb** and use default settings
3. Click **Create**

### Task 2: Import eShopOnWeb Git Repository

1. Navigate to **Repos > Files**
2. Click **Import a Repository**
3. Paste the repository URL: `https://github.com/MicrosoftLearning/eShopOnWeb.git`
4. Click **Import**

The repository structure includes:
- `.ado` folder: Azure DevOps YAML pipelines
- `.devcontainer` folder: Container development setup
- `infra` folder: Bicep and ARM templates
- `.github` folder: GitHub workflow definitions
- `src` folder: .NET website

### Task 3: Set Main Branch as Default

1. Go to **Repos > Branches**
2. Hover over the `main` branch
3. Click the ellipsis and select **Set as default branch**

## Exercise 1: Understand Azure Bicep Template and Create a Module

### Task 1: Review the Azure Bicep Template

1. Navigate to **Repos > Files** and open the `infra` folder
2. Review the `simple-windows-vm.bicep` file

Key components to understand:
- Parameters with types, default values, and validation rules
- Variables for naming and configuration
- Resource types:
  - Microsoft.Storage/storageAccounts
  - Microsoft.Network/publicIPAddresses
  - Microsoft.Network/virtualNetworks
  - Microsoft.Network/networkInterfaces
  - Microsoft.Compute/virtualMachines

Note the simplified resource definitions and implicit symbolic name references instead of explicit `dependsOn` declarations.

### Task 2: Create a Bicep Module for Storage Resources

1. Open `simple-windows-vm.bicep` for editing
2. Delete the storage resource definition from the main template:

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
}
```

3. Change `publicIPAllocationMethod` default from `Dynamic` to `Static`
4. Change `publicIpSku` default from `Basic` to `Standard`
5. Commit your changes

6. Create a new file named `storage.bicep` in the `infra` folder
7. Add the following code:

```bicep
@description('Location for all resources.')
param location string = resourceGroup().location

@description('Name for the storage account.')
param storageAccountName string

resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageURI string = storageAccount.properties.primaryEndpoints.blob
```

8. Commit the file

### Task 3: Modify Main Template to Use the Module

1. Open `simple-windows-vm.bicep` for editing
2. Add the module reference after the variables section:

```bicep
module storageModule './storage.bicep' = {
  name: 'linkedTemplate'
  params: {
    location: location
    storageAccountName: storageAccountName
  }
}
```

3. In the virtual machine resource, update the `diagnosticsProfile` section to use the module output:

```bicep
diagnosticsProfile: {
  bootDiagnostics: {
    enabled: true
    storageUri: storageModule.outputs.storageURI
  }
}
```

4. Review the key concepts:
   - Module symbolic name (`storageModule`) is used for configuring dependencies
   - Only Incremental deployment mode is supported with modules
   - Relative paths are used for template modules
   - Parameters pass values from main template to modules

5. Commit the template

## Exercise 2: Deploy Templates to Azure Using YAML Pipelines

### Task 1: Deploy Resources to Azure by YAML Pipelines

1. Navigate to **Pipelines** and click **Create pipeline**
2. Select **Azure Repos Git (YAML)** as your code source
3. Select the **eShopOnWeb** repository
4. On the Configure pipeline pane, select **Existing Azure Pipelines YAML File**
5. Specify the following:
   - Branch: `main`
   - Path: `.ado/eshoponweb-cd-windows-cm.yml`
6. Click **Continue**
7. In the variables section:
   - Set your resource group name
   - Set your desired location (e.g., `eastus` or `centralus`)
   - Replace the service connection with an existing one
8. Click **Save and run** from the top right corner
9. In the commit dialog, click **Save and run** again
10. Wait for the deployment to complete and review the results

### Important Notes

- Grant the pipeline permission to use your Service Connection
- **Remember to delete the resources created in Azure after completing the lab to avoid unnecessary charges**

## Exercise 3: Clean Up Resources

### Task 1: Delete Resource Group via Azure Portal

1. Navigate to the [Azure portal](https://portal.azure.com)
2. Go to **Resource groups**
3. Search for and select your resource group (e.g., `AZ400-EWebShop-Rehan`)
4. Click **Delete resource group**
5. Enter the resource group name to confirm
6. Click **Delete**
7. Wait for the deletion to complete

### Task 2: Delete Resource Group via Azure CLI

Alternatively, you can delete the resource group using Azure CLI:

```bash
az group delete --name AZ400-EWebShop-Rehan --yes --no-wait
```

Parameters:
- `--name`: Name of the resource group to delete
- `--yes`: Skip confirmation prompt
- `--no-wait`: Don't wait for the operation to complete

### Task 3: Clean Up DevOps Pipeline (Optional)

If you no longer need the pipeline:

1. In Azure DevOps, navigate to **Pipelines**
2. Select the pipeline you created (e.g., `eshoponweb-cd-windows-cm`)
3. Click the ellipsis menu and select **Delete**
4. Confirm the deletion

### What Gets Deleted

When you delete the resource group, the following resources are removed:

- Virtual Machine (simple-vm)
- Storage Account (for boot diagnostics)
- Virtual Network (MyVNET)
- Network Interface (myVMNic)
- Public IP Address (myPublicIP)
- Network Security Group (default-NSG)

This ensures no ongoing charges for unused Azure resources.

## Lab Summary

In this lab, you:

1. Created and configured an Azure DevOps project
2. Imported the eShopOnWeb repository
3. Reviewed the structure of an Azure Bicep template
4. Extracted storage resources into a reusable module
5. Modified the main template to reference the module
6. Deployed the complete infrastructure using a YAML pipeline

## Key Takeaways

- Bicep modules promote code reusability and maintainability
- Template modules enable implicit dependency management
- Azure DevOps YAML pipelines automate infrastructure deployment
- Modular templates scale better for complex infrastructure scenarios

## VM Size Considerations

When deploying Windows VMs, ensure the selected VM size is available in your target region. The lab uses `Standard_D2s_v3` by default, which is widely available. If you experience SKU availability errors:

1. Check available SKUs: `az vm list-skus --location <region> --size Standard_D --all`
2. Update the `vmSize` parameter in the Bicep template if needed
3. Verify the deployment region in the pipeline variables

**Reference:** [Microsoft Learning Lab - Deployments using Azure Bicep templates](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M05_L12_Deployments_using_Azure_Bicep_templates.html)