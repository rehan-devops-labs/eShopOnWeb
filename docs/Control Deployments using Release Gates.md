# AZ-400 Lab: Control Deployments Using Release Gates

Source lab:
https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M03_L08_Control_Deployments_using_Release_Gates.html

## Lab Overview

This lab demonstrates how to control classic release deployments in Azure DevOps by combining approvals and gates.

You will:

- Build and publish the eShopOnWeb app from a YAML CI pipeline.
- Deploy through a classic release pipeline with DevTest and Production stages.
- Add pre-deployment approvals.
- Add post-deployment release gates backed by Azure Monitor alerts.
- Test gate behavior by intentionally creating failed requests.

## Objectives

After completing this lab, you should be able to:

- Configure a classic release pipeline.
- Configure release approvals and gates.
- Validate and troubleshoot gate behavior during staged deployments.

## Estimated Time

75 minutes

## Prerequisites

- Azure DevOps organization and project admin permissions.
- Azure subscription where you have Owner permissions.
- Microsoft Entra tenant where you have Global Administrator permissions.
- Existing or new Azure DevOps project named `eShopOnWeb`.
- Browser supported by Azure DevOps.
- Local Azure DevOps agent installed at `D:\Learning\AZ-400\Azure Agent`.
- Self-hosted agent pool registered in Azure DevOps.

## Agent Setup

Before running the CI or release pipeline, start the local self-hosted agent.

1. Open PowerShell or Command Prompt.
2. Change to the agent directory:

```cmd
cd D:\Learning\AZ-400\Azure Agent
```

3. Start the agent:

```cmd
run.cmd
```

Leave this terminal window open while the pipeline and release stages are running.

Note: The agent must already be configured and registered to an Azure DevOps pool such as `Default`. Use that same pool name when selecting the agent pool in Azure DevOps.

## Exercise 0: Lab Prerequisites

### Task 1: Create the project (skip if already done)

1. In Azure DevOps, create a project named `eShopOnWeb`.
2. Keep default settings unless your environment requires changes.

### Task 2: Import repository (skip if already done)

1. Go to Repos > Files > Import repository.
2. Import:

```text
https://github.com/MicrosoftLearning/eShopOnWeb.git
```

3. Go to Repos > Branches and set `main` as default if needed.

### Task 3: Configure CI pipeline (YAML)

1. Go to Pipelines > Create pipeline.
2. Select Azure Repos Git (YAML).
3. Select repository `eShopOnWeb`.
4. Choose Existing Azure Pipelines YAML file.
5. Select:
   - Branch: `main`
   - Path: `.ado/eshoponweb-ci.yml`
6. Confirm the YAML uses your self-hosted pool, for example:

```yaml
pool:
   name: 'Default'
```

7. Run the pipeline and confirm success.
8. Rename pipeline to `eshoponweb-ci`.

## Exercise 1: Create Azure Resources for Release

### Task 1: Create two Azure Web Apps

Use Azure Cloud Shell (Bash):

```bash
REGION='<region>'
RESOURCEGROUPNAME='az400m03l08-RG'
az group create -n $RESOURCEGROUPNAME -l $REGION

SERVICEPLANNAME='az400m03l08-sp1'
az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1

SUFFIX=$RANDOM$RANDOM
az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
```

Notes:

- Save the generated DevTest app name for later use.
- If you see provider registration errors, run:

```bash
az provider register --namespace Microsoft.Web
```

### Task 2: Configure Application Insights + alert rule

1. Create an Application Insights resource in `az400m03l08-RG`.
2. Use the DevTest web app name as the App Insights resource name.
3. In the DevTest web app, enable Application Insights and link the created resource.
4. Create an alert rule based on `Failed Requests` with threshold `> 0`.
5. Use alert settings similar to:
   - Severity: `2 - Warning`
   - Alert rule name: `RGATESDevTest_FailedRequests`
   - Auto-resolve: disabled

## Exercise 2: Configure Classic Release Pipeline

### Task 1: Create release stages and deployment tasks

1. In Azure DevOps, go to Pipelines > Releases.
2. Create a new release pipeline from **Azure App Service Deployment** template.
3. Rename Stage 1 to `DevTest`.
4. Rename pipeline to `eshoponweb-cd`.
5. Clone `DevTest` stage and name clone `Production`.
6. Add build artifact from `eshoponweb-ci`.
7. Enable continuous deployment trigger on the artifact.

Configure DevTest deployment task:

- Agent pool: select your self-hosted pool such as `Default` in the stage Agent job.
- Azure subscription: authorize your service connection.
- App type: `Web App on Windows`.
- App service: select DevTest app.
- Package/folder:

```text
$(System.DefaultWorkingDirectory)/**/Web.zip
```

- App settings:

```text
-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development
```

Configure Production stage similarly, but target Prod app and use the same self-hosted agent pool.

8. Save the release pipeline.
9. Run `eshoponweb-ci` and verify release auto-triggers.
10. Confirm deployment succeeds in DevTest and Production.

Note: In a classic release pipeline, set the agent pool inside each stage's Agent job settings rather than in YAML.

## Exercise 3: Configure Release Approvals and Gates

### Task 1: Add pre-deployment approval (DevTest)

1. Open `eshoponweb-cd` in edit mode.
2. Open DevTest **Pre-deployment conditions**.
3. Enable pre-deployment approvals.
4. Add yourself (or approved team alias) as approver.
5. Save.
6. Create a release and approve DevTest when prompted.

### Task 2: Add post-deployment gate (Azure Monitor)

1. Open DevTest **Post-deployment conditions**.
2. Enable Gates.
3. Add **Query Azure Monitor Alerts** gate.
4. Set:
   - Subscription/service connection: your Azure connection
   - Resource group: `az400m03l08-RG`
5. In Advanced settings use:
   - Filter type: `None`
   - Severity: `Sev0, Sev1, Sev2, Sev3, Sev4`
   - Time range: `Past Hour`
   - Alert state: `Acknowledged, New`
   - Monitor condition: `Fired`
6. In Evaluation options set:
   - Re-evaluation interval: `5 minutes`
   - Gate timeout: `8 minutes`
   - On successful gates: ask for approvals
7. Save pipeline.

## Exercise 4: Test Release Gates

### Task 1: Trigger alerts and observe gate behavior

1. Open DevTest app in browser.
2. Append `/discount` to URL to generate 404/failed requests.
3. Refresh several times.
4. Check Application Insights / Azure Monitor alerts and verify alert creation.
5. In Azure DevOps, create a new release in `eshoponweb-cd`.
6. Approve DevTest pre-deployment when prompted.
7. Observe post-deployment gate evaluation.
8. Confirm gate initially fails while alert is active.
9. Wait for next re-evaluation cycle and confirm behavior changes when alert state clears.
10. Validate promotion to Production after successful gate checks.

## Validation Checklist

- CI pipeline `eshoponweb-ci` builds successfully.
- Release pipeline `eshoponweb-cd` deploys to DevTest and Production.
- DevTest stage requires manual approval before deployment.
- Post-deployment gate evaluates Azure Monitor alerts.
- Gate can block promotion while active alerts exist.

## Cleanup

### Stop the local agent

1. Return to the terminal window running the agent from `D:\Learning\AZ-400\Azure Agent`.
2. Press `Ctrl+C`.
3. Wait for the agent process to stop cleanly.

### Delete Azure resources

Delete Azure resources to avoid charges:

```bash
az group delete --name az400m03l08-RG --yes --no-wait
```

### Optional Azure DevOps cleanup

1. Disable CI/CD triggers on test pipelines.
2. Remove test release definitions if no longer needed.
3. Remove service connections created only for this lab.
4. Remove the self-hosted agent from the pool if you will not reuse it.

### Optional agent removal

If you want to unregister the agent after the lab:

1. In Azure DevOps, go to Organization Settings > Agent pools.
2. Open the pool used for this lab.
3. Remove the agent entry.
4. If needed, delete the local agent folder after confirming it is no longer in use.
