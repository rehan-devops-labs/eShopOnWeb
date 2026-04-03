# AZ-400 Lab: Configure Pipelines as Code with YAML

Source lab:
https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M03_L07_Configure_Pipelines_as_Code_with_YAML.html

## Lab Overview

This lab walks through creating and extending an Azure DevOps YAML pipeline for the eShopOnWeb sample. You will:

- Set up prerequisites in Azure DevOps and Azure.
- Create a CI pipeline from an existing YAML file.
- Add a deploy stage for Azure App Service.
- Add environment approvals for controlled deployments.

## Objective

Configure CI/CD pipelines as code with YAML in Azure DevOps.

## Estimated Time

45 minutes

## Prerequisites

- Azure DevOps organization and a project you can manage.
- Azure subscription where you have Owner permissions.
- Microsoft Entra tenant where you have Global Administrator permissions.
- Browser supported by Azure DevOps.

## Exercise 0: Configure Prerequisites

### Task 1: Create and configure the project (skip if already done)

1. In Azure DevOps, create a new project named `eShopOnWeb_MultiStageYAML`.
2. Keep default settings unless your environment requires otherwise.

### Task 2: Import the eShopOnWeb repository (skip if already done)

1. Go to Repos > Files > Import repository.
2. Import from:

```text
https://github.com/MicrosoftLearning/eShopOnWeb.git
```

3. Go to Repos > Branches.
4. Set `main` as default branch (if not already default).

### Task 3: Create Azure resources

Run these commands in Azure Cloud Shell (Bash), replacing `<region>` with your preferred Azure region.

```bash
LOCATION='<region>'
RESOURCEGROUPNAME='az400m03l07-RG'
az group create --name $RESOURCEGROUPNAME --location $LOCATION

SERVICEPLANNAME='az400m03l07-sp1'
az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3

WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
```

Notes:

- If provider registration fails for `Microsoft.Web`, run:

```bash
az provider register --namespace Microsoft.Web
```

- Record the generated web app name; it is needed later in deployment settings.

## Exercise 1: Configure CI/CD Pipeline with YAML

### Task 1: Add a YAML build definition

1. In Azure DevOps, go to Pipelines > Create pipeline.
2. Select Azure Repos Git (YAML).
3. Select repository `eShopOnWeb_MultiStageYAML`.
4. Choose Existing Azure Pipelines YAML file.
5. Select branch `main` and path `.ado/eshoponweb-ci.yml`.
6. Run the pipeline and verify the build succeeds.

### Task 2: Add continuous delivery to YAML

Edit `.ado/eshoponweb-ci.yml` and append a deploy stage:

```yaml
- stage: Deploy
	displayName: Deploy to an Azure Web App
	jobs:
		- job: Deploy
			pool:
				vmImage: "windows-latest"
			steps:
```

Then add tasks under `steps` in this order.

1. Download build artifact:

```yaml
- task: DownloadBuildArtifacts@1
	inputs:
		buildType: "current"
		downloadType: "single"
		artifactName: "Website"
		downloadPath: "$(Build.ArtifactStagingDirectory)"
```

2. Deploy to Azure App Service (use your subscription and app name):

```yaml
- task: AzureRmWebAppDeployment@4
	inputs:
		ConnectionType: "AzureRM"
		azureSubscription: "<your-service-connection>"
		appType: "webApp"
		WebAppName: "<your-webapp-name>"
		packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
		AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
```

Important checks:

- Ensure indentation places both tasks inside `steps`.
- Save/commit YAML changes.
- Run pipeline manually.
- When prompted for protected resource access, review and permit access.

### Task 3: Review deployed site

1. Open the Azure portal and navigate to the web app.
2. Use Browse from the web app Overview blade.
3. Confirm eShopOnWeb loads.

## Exercise 2: Add Environment Approvals

### Task 1: Configure environment and checks

1. In Azure DevOps, go to Pipelines > Environments.
2. Create environment named `approvals` with no resource.
3. In Approvals and checks, add an Approval check.
4. Add your user as approver.

Update deploy job in `.ado/eshoponweb-ci.yml` to use deployment job + environment.

Before:

```yaml
jobs:
	- job: Deploy
		pool:
			vmImage: "windows-latest"
		steps:
```

After:

```yaml
jobs:
	- deployment: Deploy
		environment: approvals
		pool:
			vmImage: "windows-latest"
		strategy:
			runOnce:
				deploy:
					steps:
```

Then place existing deploy steps under `strategy.runOnce.deploy.steps`.

Run the pipeline again and validate behavior:

- Build stage runs.
- Deploy stage pauses for approval.
- Approver reviews and approves.
- Deploy stage completes successfully.

## Validation Checklist

- CI stage compiles/tests successfully.
- Artifact `Website` is published and downloaded in deploy stage.
- App is deployed to Azure Web App.
- Environment approval gate is enforced before deployment.

## Cleanup

Delete the lab Azure resources when finished to avoid ongoing costs.

Suggested cleanup commands:

```bash
az group delete --name az400m03l07-RG --yes --no-wait
```