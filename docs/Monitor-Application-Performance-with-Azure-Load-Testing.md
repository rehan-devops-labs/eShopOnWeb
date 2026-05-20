# Monitor Application Performance with Azure Load Testing

**Source:** [Microsoft Learning - AZ-400 Lab: Monitor Application Performance with Azure Load Testing](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M08_L14_Monitor_Application_Performance_with_Azure_Load_Testing.html)

## Lab Overview

Azure Load Testing is a fully managed load-testing service that enables you to generate high-scale load and simulate traffic for your applications, regardless of where they're hosted. Developers, testers, and QA engineers can use it to optimize application performance, scalability, and capacity.

You can:
- Quickly create a load test for your web application using a URL without prior knowledge of testing tools
- Abstract the complexity and infrastructure to run load tests at scale
- Create load tests by reusing existing Apache JMeter test scripts for advanced scenarios
- Integrate load testing into your CI/CD pipelines

## Lab Objectives

After completing this lab, you will be able to:

- Deploy Azure App Service web apps
- Compose and run a YAML-based CI/CD pipeline
- Deploy Azure Load Testing resources
- Investigate Azure web app performance using Azure Load Testing
- Integrate Azure Load Testing into your CI/CD pipelines

**Estimated timing:** 60 minutes

## Lab Requirements

### Prerequisites

- **Browser:** Microsoft Edge or Azure DevOps supported browser
- **Azure DevOps Organization:** Create one if needed
- **Azure Subscription:** Active subscription with Owner role
- **Azure Entra:** Global Administrator role in the Microsoft Entra tenant

### Create Azure Resources

You'll need to create:
1. A resource group (e.g., "az400m08l14-RG")
2. An App Service plan (e.g., "az400l14-sp" with SKU B3)
3. A web app with a unique name (e.g., "az400eshoponweb{random}{random}")

**Cloud Shell Commands:**
```bash
RESOURCEGROUPNAME='az400m08l14-RG'
LOCATION='<region>'
az group create --name $RESOURCEGROUPNAME --location $LOCATION

SERVICEPLANNAME='az400l14-sp'
az appservice plan create --resource-group $RESOURCEGROUPNAME \
    --name $SERVICEPLANNAME --sku B3

WEBAPPNAME=az400eshoponweb$RANDOM$RANDOM
az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
```

## Exercise 1: Configure CI/CD Pipelines as Code with YAML

### Setup Project

1. Create an Azure DevOps project named "eShopOnWeb" (Scrum process)
2. Import the Git repository: https://github.com/MicrosoftLearning/eShopOnWeb.git
3. Set the main branch as the default branch

### Create YAML Pipeline

1. Navigate to Pipelines → New Pipeline
2. Select "Azure Repos Git (YAML)"
3. Select the eShopOnWeb repository
4. Create a starter pipeline

**Pipeline Template Structure:**

The pipeline includes two stages:
- **Build Stage:** Restore, Build, Publish the .NET Core solution
- **Deploy Stage:** Download artifacts and deploy to Azure Web App

**Key Pipeline Tasks:**
- DotNetCoreCLI tasks for restore, build, and publish
- PublishBuildArtifacts to store build output
- AzureRmWebAppDeployment to deploy to Azure App Service

**Configuration Parameters:**
- Set Package path: `$(Build.ArtifactStagingDirectory)/**/Web.zip`
- App Settings: `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`
- Select your Azure subscription service connection

### Pipeline Execution

1. Save the pipeline file as `m08l14-pipeline.yml`
2. Commit changes to main branch
3. Run the pipeline manually
4. Approve permissions when prompted during the Deploy stage
5. Verify the web app deploys successfully

## Exercise 2: Deploy and Setup Azure Load Testing

### Deploy Azure Load Testing Resource

1. Navigate to Azure Portal and search for "Azure Load Testing"
2. Create a new Azure Load Testing resource with:
   - **Name:** eShopOnWebLoadTesting
   - **Resource Group:** Same as your App Service
   - **Region:** Close to your location (note: not all regions support this service)

3. Wait for deployment to complete
4. Go to the resource when deployment finishes

### Create Load Test Scenarios

**Test 1: Virtual Users Load Test**

1. Navigate to Tests → Create → Create a URL-based test
2. Uncheck "Enable advanced settings" to display all options
3. Configure:
   - Test URL: `https://az400eshoponweb{yourname}.azurewebsites.net`
   - Load Specification: Virtual Users
   - Number of Virtual Users: 50
   - Test Duration: 5 minutes
   - Ramp-up time: 1 minute
4. Review and create the test
5. Wait for the 5-minute test to complete

**Test 2: Requests Per Second (RPS) Load Test**

1. Create another URL-based test
2. Configure:
   - Test URL: Same as Test 1
   - Load Specification: Requests per second (RPS)
   - Requests per second: 100
   - Response time threshold: 500 milliseconds
   - Test Duration: 5 minutes
   - Ramp-up time: 1 minute
3. Review and create

### Validate Results

After tests complete, review the results:
- **Load Metrics:** Total requests, response times
- **Performance:** 90th percentile response time (90% of requests meet this threshold)
- **Throughput:** Requests per second
- **Dashboard:** Graph visualizations of performance data

Compare results between both tests to understand the impact of different load patterns.

## Exercise 3: Automate Load Testing in CI/CD

### Install Azure Load Testing Task Extension

1. Open Azure DevOps Marketplace
2. Search for and install "Azure Load Testing" task extension
3. Install to your Azure DevOps organization

### Grant Permissions

1. Navigate to Project Settings → Service Connections
2. Select your Azure subscription service connection
3. Manage service connection roles → Add role assignment
4. Assign "Load Test Contributor" role to the service connection user

### Export and Import Load Test Files

1. In Azure Load Testing portal, select your test and download input files
2. Extract the zip file containing:
   - `config.yaml` - Load test configuration
   - `quick_test.jmx` - JMeter test script

3. In Azure Repos:
   - Create a `/tests/jmeter/` folder
   - Upload both `config.yaml` and `quick_test.jmx`
   - Edit `config.yaml` to set displayName and testId to `ado_load_test`

### Update Pipeline YAML

Add Azure Load Testing task to your pipeline before the Deploy stage:

```yaml
- task: AzureLoadTest@1
  inputs:
    azureSubscription: 'YOUR_SUBSCRIPTION_NAME'
    loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
    resourceGroup: 'az400m08l14-RG'
    loadTestResource: 'eShopOnWebLoadTesting'
    loadTestRunName: 'ado_run'
    loadTestRunDescription: 'load testing from ADO'

- publish: $(System.DefaultWorkingDirectory)/loadTest
  artifact: loadTestResults
```

### Add Failure Criteria

Edit the `config.yaml` file to add failure criteria:

```yaml
failureCriteria:
  - avg(response_time_ms) > 300
  - percentage(error) > 50
```

These criteria will:
- Fail the test if average response time exceeds 300ms
- Fail the test if error percentage exceeds 50%
- Cause the pipeline task to fail if thresholds are exceeded

### Test Pipeline Integration

1. Run the pipeline
2. Monitor the AzureLoadTest task execution
3. View test results in Azure portal under the Load Testing resource
4. Check pipeline logs for test metrics and pass/fail status

**Expected Output:**
- Virtual Users count
- Test duration and timing
- Client-side metrics (response time, requests/sec, errors)
- Test criteria pass/fail results

## Load Testing Best Practices

### Performance Metrics to Monitor

- **Response Time:** Average, min, max, and percentile values (90th, 95th, 99th)
- **Throughput:** Requests per second
- **Error Rate:** Percentage of failed requests
- **Resource Utilization:** CPU, memory on target application

### Test Configuration Considerations

**Sampling Intervals:**
- Test Duration: 5-10 minutes for quick tests, longer for comprehensive testing
- Ramp-up Time: Gradually increase load (1-5 minutes) to simulate real user behavior
- Number of Virtual Users: Start conservative, increase based on capacity testing needs

**Load Types:**
- Virtual Users: Simulate concurrent users
- Requests Per Second (RPS): Specify exact request rate

### CI/CD Integration Benefits

- Automated performance regression testing
- Detect performance issues before production deployment
- Fail builds when thresholds are exceeded
- Track performance trends over time
- Combine with infrastructure scaling policies

## Troubleshooting

### Common Issues

1. **Extension Installation:** Ensure you have organization-level permissions
2. **Service Connection Permissions:** Verify the service connection has "Load Test Contributor" role
3. **YAML Syntax:** Check indentation alignment in pipeline YAML
4. **Azure Region Availability:** Azure Load Testing not available in all regions
5. **Test File Paths:** Use correct path format: `$(Build.SourcesDirectory)/tests/jmeter/config.yaml`

## Important Notes

⚠️ **Cost Awareness:** Azure Load Testing is a paid service. Each Load Testing Resource active during any part of a month incurs a monthly fee plus charges for Virtual User Hours (VUH) beyond included 50 VUH. Delete resources after completing the lab to avoid unnecessary charges.

See [Azure Load Testing Pricing](https://azure.microsoft.com/pricing/details/load-testing) for current rates.

## Summary

This lab demonstrates a complete workflow for implementing performance testing in your DevOps pipeline:

1. **Deploy Infrastructure:** Create App Service and Load Testing resources
2. **Build CI/CD Pipeline:** Automate application deployment
3. **Create Load Tests:** Define test scenarios with different load patterns
4. **Analyze Results:** Review performance metrics and identify bottlenecks
5. **Integrate Testing:** Automate load testing as part of your deployment pipeline
6. **Set Quality Gates:** Define failure criteria to catch performance regressions

By integrating Azure Load Testing into your CI/CD pipeline, you can catch performance issues early in your development cycle and ensure your application meets performance requirements before reaching production.
