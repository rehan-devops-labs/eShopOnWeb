# AZ-400 Lab: Enable Dynamic Configuration and Feature Flags

## Lab Overview

Azure App Configuration provides centralized management of application settings and feature flags. Modern cloud applications often have distributed components spread across services, making configuration management difficult. By using App Configuration, you store all application settings in one secure location and enable dynamic updates without redeploying code.

In this lab, you will:

- Set up CI/CD pipelines for the eShopOnWeb application.
- Create an Azure App Configuration resource.
- Enable managed identity on the Web App to securely access App Configuration.
- Configure dynamic settings and feature flags.
- Test real-time configuration updates without code changes or redeployment.

## Objectives

After completing this lab, you will be able to:

- Enable dynamic configuration in an Azure DevOps CI/CD pipeline.
- Manage feature flags through Azure App Configuration.
- Use managed identity for secure service-to-service authentication.
- Update application configuration at runtime.

## Estimated Time

45 minutes

## Lab Requirements

- Microsoft Edge or another Azure DevOps-supported browser.
- Azure DevOps organization (create one if needed).
- Azure subscription with Contributor or Owner role.
- Ability to create Azure resources and role assignments.

## Exercise 0: Configure Lab Prerequisites (Skip if already done)

### Task 1: Create and configure the team project

1. Open your Azure DevOps organization in a browser.
2. Select **New Project**.
3. Name the project `eShopOnWeb`.
4. Under **Work item process**, select **Scrum**.
5. Select **Create**.

### Task 2: Import eShopOnWeb Git repository

1. In the `eShopOnWeb` project, go to **Repos > Files > Import**.
2. Import from this URL:

```text
https://github.com/MicrosoftLearning/eShopOnWeb.git
```

Repository structure:

- `.ado` contains Azure DevOps YAML pipelines.
- `.devcontainer` contains container development setup.
- `infra` contains Bicep/ARM IaC templates.
- `.github` contains GitHub workflow definitions.
- `src` contains the .NET 8 sample app.

### Task 3: Set main as default branch

1. Go to **Repos > Branches**.
2. On branch `main`, open the ellipsis menu.
3. Select **Set as default branch**.

## Exercise 1: Import and Run CI/CD Pipelines

### Task 1: Import and run the CI pipeline

1. Go to **Pipelines > Pipelines**.
2. Select **Create Pipeline** (if empty) or **New pipeline**.
3. Select **Azure Repos Git (YAML)**.
4. Select repository `eShopOnWeb`.
5. Select **Existing Azure Pipelines YAML File**.
6. Set:
   - Branch: `main`
   - Path: `/.ado/eshoponweb-ci.yml`
7. Select **Continue**.
8. Select **Run** to execute the pipeline.
9. After the first successful run, rename the pipeline:
   - Go to **Pipelines > Pipelines**.
   - Open the newly created pipeline.
   - Ellipsis menu > **Rename/Remove**.
   - Name it `eshoponweb-ci` and save.

### Task 2: Import and run the CD pipeline

1. Go to **Pipelines > Pipelines**.
2. Select **New pipeline**.
3. Select **Azure Repos Git (YAML)**.
4. Select repository `eShopOnWeb`.
5. Select **Existing Azure Pipelines YAML File**.
6. Set:
   - Branch: `main`
   - Path: `/.ado/eshoponweb-cd-webapp-code.yml`
7. Select **Continue**.
8. In the YAML, customize:
   - Replace `YOUR-SUBSCRIPTION-ID` with your Azure subscription ID.
   - Replace `az400eshop-NAME` with a globally unique name for the web app.
   - Replace `AZ400-EWebShop-NAME` with your resource group name.
9. Select **Save and Run**.
10. If prompted with a permission dialog:
    - Click **View**.
    - Click **Permit**.
    - Click **Permit** again to complete.
11. Wait for deployment to finish (may take a few minutes).
12. Rename the pipeline after first run:
    - Go to **Pipelines > Pipelines**.
    - Open the newly created pipeline.
    - Ellipsis menu > **Rename/Remove**.
    - Name it `eshoponweb-cd-webapp-code` and save.

## Exercise 2: Manage Azure App Configuration

### Task 1: Create the App Configuration resource

1. In Azure portal, search for **App Configuration**.
2. Select **Create app configuration**.
3. Configure:

| Setting | Value |
|---|---|
| Subscription | Your Azure subscription |
| Resource group | `AZ400-EWebShop-NAME` (from CD pipeline) |
| Resource name | Unique name, e.g. `appcs-NAME-REGION` |
| Location | Same region as your web app |
| Pricing tier | Free |

4. Select **Review + create > Create**.
5. After creation, go to **Overview**.
6. Copy and save the **Endpoint** value (format: `https://appcs-NAME-REGION.azconfig.io`).

### Task 2: Enable Managed Identity

1. Go to the Web App deployed by the CD pipeline (name: `az400-webapp-NAME`).
2. In the left menu under **Settings**, select **Identity**.
3. Under **System assigned**, toggle status to **On**.
4. Select **Save > Yes** and wait a few seconds.
5. Return to the **App Configuration** resource.
6. In the left menu, select **Access control (IAM)**.
7. Select **+ Add > Add role assignment**.
8. Configure role assignment:
   - Role: Select **App Configuration Data Reader**.
   - Members: Select **Manage identity**.
   - Click **+ Select members**.
   - In the **Managed identity** dropdown, select **App Service**.
   - Select your app service (the one you just enabled identity on).
   - Click **Select**.
9. Review the settings and select **Review + assign** twice to complete.

### Task 3: Configure the Web App environment variables

1. Go back to your **Web App** (e.g., `az400-webapp-NAME`).
2. In the left menu under **Settings**, select **Environment variables**.
3. Add two new application settings:

**First setting:**
| Name | Value |
|---|---|
| Name | `UseAppConfig` |
| Value | `true` |

**Second setting:**
| Name | Value |
|---|---|
| Name | `AppConfigEndpoint` |
| Value | The Endpoint you saved earlier (e.g., `https://appcs-NAME-REGION.azconfig.io`) |

4. Select **Apply**.
5. Select **Confirm** and wait for the settings to update.
6. Go to **Overview** and click **Browse** to open the website.

**Note:** At this point, the website appears unchanged because App Configuration contains no data yet. You will add configuration in the next tasks.

### Task 4: Test dynamic configuration

1. On the website, locate the **Brand** dropdown (top of the page).
2. Select **Visual Studio** from the dropdown.
3. Click the arrow button (>) to search.
4. Observe the message: **"THERE ARE NO RESULTS THAT MATCH YOUR SEARCH"**.
5. Now, return to the **App Configuration** resource in Azure portal.
6. In the left menu under **Operations**, select **Configuration Explorer**.
7. Select **Create > Key-value**.
8. Add a new configuration:

| Setting | Value |
|---|---|
| Key | `eShopWeb:Settings:NoResultsMessage` |
| Value | Enter a custom message, e.g., `"No Visual Studio products available right now"` |

9. Select **Apply**.
10. Return to the website in your browser and refresh the page.
11. You should now see your custom message instead of the default message.

**Key insight:** The configuration updated dynamically without restarting the website or deploying new code.

### Task 5: Test the feature flag

1. Return to the **App Configuration** resource.
2. In the left menu under **Operations**, select **Feature manager**.
3. Select **Create**.
4. Configure the feature flag:

| Setting | Value |
|---|---|
| Enable feature flag | Checked |
| Feature flag name | `SalesWeekend` |

5. Select **Apply**.
6. Go back to the website and refresh the page.
7. You should now see an image with text: **"ALL T-SHIRTS ON SALE THIS WEEKEND"**.

**Testing:** To verify the feature flag works, go back to **Feature manager** in App Configuration and uncheck the feature flag. Refresh the website—the image and sale message should disappear.

## Review

In this lab, you learned how to:

- Set up CI/CD pipelines to deploy eShopOnWeb to Azure.
- Create and configure Azure App Configuration for centralized settings management.
- Enable managed identity on a Web App for secure access to App Configuration.
- Update configuration settings dynamically without code changes or redeployment.
- Manage feature flags to enable/disable features at runtime.

## Cleanup

To avoid unnecessary charges, delete lab resources when complete:

**Important:** Before deleting resources, disable the `eshoponweb-cd-webapp-code` pipeline or it will recreate deleted resources on the next run of `eshoponweb-ci`.

1. In Azure DevOps, go to **Pipelines > Pipelines**.
2. Open **eshoponweb-cd-webapp-code** and select **More options (...)**.
3. Select **Disable** to prevent automatic triggers.
4. In Azure portal, delete:
   - Resource group `AZ400-EWebShop-NAME`.
   - App Configuration resource (if in a separate resource group).
5. Optional: Remove the CI/CD pipelines from Azure DevOps if they were created only for this lab.
