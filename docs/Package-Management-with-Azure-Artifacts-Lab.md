# Package Management with Azure Artifacts

## Lab Overview

Azure Artifacts facilitate discovery, installation, and publishing NuGet, npm, and Maven packages in Azure DevOps. It's deeply integrated with other Azure DevOps features such as Build, making package management a seamless part of your existing workflows.

## Objectives

After completing this lab, you will be able to:

- Create and connect to a feed
- Create and publish a NuGet package
- Import a NuGet package
- Update a NuGet package

**Estimated timing: 35 minutes**

---

## Lab Requirements

- Microsoft Edge or an [Azure DevOps supported browser](https://docs.microsoft.com/azure/devops/server/compatibility)
- Azure DevOps organization (create one if needed following [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html))
- Sample eShopOnWeb Project
- Visual Studio 2022 Community Edition with:
  - ASP.NET and web development workload
  - Azure development workload
  - .NET Core cross-platform development workload
- .NET Core SDK (2.1.400+)
- [Azure Artifacts credential provider](https://go.microsoft.com/fwlink/?linkid=2099625)

---

## Instructions

### Exercise 0: Configure Lab Prerequisites

#### Task 1: Create and Configure the Team Project

1. In your Azure DevOps organization, click **New Project**
2. Name the project **eShopOnWeb** and use default settings
3. Click **Create**

#### Task 2: Import eShopOnWeb Git Repository

1. In the eShopOnWeb project, go to **Repos > Files**
2. Click **Import a Repository > Import**
3. Paste the URL: `https://github.com/MicrosoftLearning/eShopOnWeb.git`
4. Click **Import**

The repository is organized as follows:
- `.ado` folder - Contains Azure DevOps YAML pipelines
- `.devcontainer` folder - Container setup for VS Code or GitHub Codespaces
- `infra` folder - Bicep & ARM infrastructure as code templates
- `.github` folder - YAML GitHub workflow definitions
- `src` folder - .NET 8 website for lab scenarios

#### Task 3: Set Main Branch as Default Branch

1. Go to **Repos > Branches**
2. Hover over the **main** branch and click the ellipsis
3. Click **Set as default branch**

#### Task 4: Configure the eShopOnWeb Solution in Visual Studio

1. In the eShopOnWeb project, click **Repos**
2. Click **Clone** and select **Clone in VS Code** dropdown, then select **Visual Studio**
3. Click **Open** if prompted
4. Sign in with your Azure DevOps credentials if prompted
5. In the Azure DevOps pop-up, accept the default local path (C:\eShopOnWeb) and click **Clone**
6. Leave Visual Studio open for the lab

---

### Exercise 1: Working with Azure Artifacts

#### Task 1: Create and Connect to a Feed

1. In your Azure DevOps project settings, select **Artifacts** from the vertical navigation pane
2. Click **+ Create feed** at the top
3. In the **Create new feed** pane:
   - **Name**: `eShopOnWebShared`
   - **Visibility**: Select **Specific people**
   - **Scope**: Select **Project:eShopOnWeb**
   - Leave other settings with defaults
   - Click **Create**

4. Back on the Artifacts hub, click **Connect to feed**
5. Select **Visual Studio** in the NuGet section and copy the Source URL:
   ```
   https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json
   ```

6. In Visual Studio:
   - Click **Tools > NuGet Package Manager > Package Manager Settings**
   - Click **Package Sources** and click the plus sign to add a new source
   - Replace "Package source" with **eShopOnWebShared**
   - Paste the URL in the Source field
   - Click **Update** then **OK**

#### Task 2: Create and Publish an In-House Developed NuGet Package

1. In Visual Studio, create a new project:
   - Click **File > New > Project**
   - Search for and select **Class Library** template (C# targeting .NET or .NET Standard)
   - Click **Next**

2. Configure the project:
   - **Project name**: `eShopOnWeb.Shared`
   - Accept default location
   - **Solution name**: `eShopOnWeb.Shared`
   - Check **Place solution and project in the same directory**
   - Click **Next**
   - Accept **.NET 8** as the framework
   - Click **Create**

3. Delete **Class1.cs** from Solution Explorer

4. Build the project (Ctrl+Shift+B or right-click project > Build)

5. Open **Windows PowerShell as Administrator** and navigate to the project folder:
   ```powershell
   cd c:\eShopOnWeb\eShopOnWeb.Shared
   ```

6. Create a NuGet package (replace XXXXXX with a unique string):
   ```powershell
   dotnet pack .\eShopOnWeb.Shared.csproj -p:PackageId=eShopOnWeb-XXXXXX.Shared
   ```

7. Navigate to the release folder:
   ```powershell
   cd .\bin\Release
   ```

8. Publish the package to the eShopOnWebShared feed (replace the URL with your actual source URL):
   ```powershell
   dotnet nuget push --source "https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json" --api-key az "eShopOnWeb-XXXXXX.Shared.1.0.0.nupkg"
   ```

**Notes:**
- If you receive a 401 Unauthorized error, install the credential provider:
  ```powershell
  iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"
  ```
- You can use any non-empty API key (example uses "az")
- Sign in to your Azure DevOps organization when prompted
- If needed, add the `–interactive` parameter to the command

9. After successful push, verify in Azure DevOps:
   - Go to **Artifacts**
   - Select **eShopOnWebShared** feed from the dropdown
   - The newly published package should appear
   - Click the package to view its details

#### Task 3: Import an Open-Source NuGet Package to the Feed

This demonstrates using NuGet packages from the public NuGet gallery (nuget.org).

1. In PowerShell, navigate back to the eShopOnWeb.Shared folder:
   ```powershell
   cd ../..
   ```

2. Install the Newtonsoft.Json package:
   ```powershell
   dotnet add package Newtonsoft.Json
   ```

3. The output shows the feeds being queried:
   ```
   Feeds used:
     https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
     https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
   ```

4. In Visual Studio Solution Explorer, navigate to eShopOnWeb.Shared Project > Dependencies > Packages and locate Newtonsoft.Json

5. The Azure Artifacts feed enables upstream sources (like nuget.org), allowing automatic caching of public packages while maintaining a single source of truth

6. Verify in Azure DevOps Artifacts:
   - Refresh the **Artifacts** page
   - The feed now shows both the custom eShopOnWeb.Shared and the Newtonsoft.Json packages

7. In Visual Studio, you can also view packages in **NuGet Package Manager**:
   - Right-click the eShopOnWeb.Shared project > **Manage NuGet Packages**
   - Verify the Package Source is set to **eShopOnWebShared**
   - Click **Browse** to see both packages

---

## Review

In this lab, you learned how to work with Azure Artifacts by:

- Creating and connecting to a feed
- Creating and publishing a NuGet package
- Importing and integrating public NuGet packages
- Managing package sources in Visual Studio

---

## Additional Resources

- [Azure DevOps Documentation](https://docs.microsoft.com/azure/devops/)
- [NuGet Package Management](https://www.nuget.org/)
- [Azure Artifacts Credential Provider](https://go.microsoft.com/fwlink/?linkid=2099625)
- [Lab Source](https://github.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions)
