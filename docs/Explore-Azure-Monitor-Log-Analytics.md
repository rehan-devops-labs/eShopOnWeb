# Explore Azure Monitor and Log Analytics

**Source:** [Microsoft Learn - Explore Azure Monitor and Log Analytics](https://learn.microsoft.com/en-us/training/modules/implement-tools-track-usage-flow/4-explore-azure-monitor-log-analytics)

## Overview

Azure Monitor is Microsoft's native cloud monitoring solution that provides comprehensive observability across your entire estate. It collects monitoring telemetry from different kinds of on-premises and Azure sources, creating a unified view of your infrastructure and applications regardless of where they run.

## Understanding Azure Monitor and Log Analytics

### Azure Monitor Architecture and Capabilities

**Data Collection Sources:**
- Azure resources: Platform metrics, resource logs, activity logs
- Applications: Application Insights SDK telemetry
- Virtual machines: Performance counters, event logs, syslog
- Containers: Container logs, Kubernetes metrics
- Custom sources: REST API, Data Collector API, custom agents

**Management Tool Integrations:**
- Azure Security Center: Security alerts and recommendations
- Azure Automation: Runbook execution logs and automation telemetry
- Azure Backup: Backup job status and recovery point information
- Azure Site Recovery: Replication health and failover events
- Third-party tools: SIEM integrations, ITSM connectors

### The Log Analytics Data Store

Log Analytics aggregates and stores telemetry in a log data store optimized for cost and performance.

**Performance Characteristics:**
- Fast ingestion: Handle billions of events per day across thousands of sources
- Efficient storage: Columnar compression reduces storage costs by 90% or more
- Query speed: Execute complex analytics across terabytes in seconds
- Scalability: Automatically scale to handle workload variations
- Retention flexibility: Configure different retention periods by table (30 days to 12 years)

**Cost Optimization Features:**
- Data tiers: Basic Logs (low-cost, limited queries) vs Analytics Logs (full query capability)
- Commitment tiers: Discounts for predictable ingestion volumes
- Archiving: Move old data to low-cost archive storage
- Data collection rules: Filter data before ingestion to reduce costs

### Capabilities of Azure Monitor

**Analysis Capabilities:**
- Log Analytics queries: Use Kusto Query Language (KQL) to analyze telemetry
- Workbooks: Create interactive reports combining queries and visualizations
- Dashboards: Pin important metrics and queries to shared dashboards
- Power BI integration: Export data to Power BI for advanced reporting

**Alerting Capabilities:**
- Metric alerts: Trigger on threshold violations or anomalies
- Log alerts: Fire when query results meet conditions
- Activity log alerts: Notify on Azure resource operations
- Smart detection: Automatically identify abnormal patterns

**Insights Capabilities:**
- Application Insights: APM with distributed tracing and profiling
- Container Insights: Kubernetes cluster health and performance
- VM Insights: Virtual machine dependencies and process monitoring
- Network Insights: Network topology and connectivity analysis

**Machine Learning Features:**
- Anomaly detection: Automatically identify unusual metric patterns
- Root cause analysis: Suggest likely causes for performance issues
- Smart groups: Cluster related alerts to reduce noise
- Predictive analytics: Forecast resource utilization trends

## Learning Objectives

By completing this hands-on tutorial, you'll be able to:

1. **Set up Log Analytics workspace:**
   - Create a workspace using PowerShell automation
   - Configure intelligence packs (solutions) for specialized monitoring
   - Enable IIS log collection for web server monitoring
   - Configure Windows event log collection
   - Understand workspace security keys and access management

2. **Connect virtual machines to a Log Analytics workspace:**
   - Install the Microsoft Monitoring Agent extension on VMs
   - Configure the workspace connection with secure keys
   - Troubleshoot extension installation issues
   - Understand agent communication and data flow
   - Manage multi-workspace scenarios

3. **Configure Log Analytics workspace to collect custom performance counters:**
   - Define performance counter collection rules
   - Set sampling intervals for different counter types
   - Configure instance-specific vs. aggregate collection
   - Create SQL Server-specific monitoring configurations
   - Optimize counter collection to balance detail and cost

4. **Analyze telemetry using Kusto Query Language (KQL):**
   - Query collected performance data
   - Correlate events across different sources
   - Create visualizations from query results
   - Build alerts based on query conditions

5. **Generate and observe test data:**
   - Use load testing tools to simulate realistic workloads
   - Observe performance counter behavior under load
   - Correlate load patterns with collected telemetry
   - Understand latency between event occurrence and availability in logs

## Getting Started

### Prerequisites

**Azure Resources:**
- Active Azure subscription with permissions to create resources
- Resource group (existing or ability to create one)
- One or more Windows VMs with RDP access
- PowerShell access (Azure Cloud Shell or local Azure PowerShell module)

**Access Requirements:**
- Contributor role on the resource group
- VM Contributor role to install extensions on virtual machines
- RDP access to the VM for validation and load generation

### Understanding the Setup Process

This tutorial uses PowerShell automation to configure monitoring, demonstrating Infrastructure as Code (IaC) principles:

**Benefits of PowerShell Automation:**
- Repeatability: Same script works across multiple environments
- Documentation: Script serves as documentation of configuration
- Version control: Store scripts in source control for tracking changes
- Consistency: Eliminates manual configuration errors
- Scale: Apply configuration to dozens of VMs with minimal effort

## Hands-On Setup Steps

### Step 1: Create Log Analytics Workspace

Log into Azure Cloud Shell and execute the following script. This creates a new resource group and Log Analytics workspace, then configures multiple monitoring solutions.

**Before running:** Replace the variable values with your environment-specific information:
- `$ResourceGroup`: Your resource group name (e.g., "azwe-rg-devtest-logs-001")
- `$WorkspaceName`: Unique workspace name (e.g., "azwe-devtest-logs-01")
- `$Location`: Azure region (e.g., "westeurope", "eastus", "westus2")

**What this script does:**
1. Creates or verifies resource group
2. Creates Log Analytics workspace
3. Installs intelligence packs (solutions)
4. Enables IIS log collection
5. Configures Windows event logs

```powershell
$ResourceGroup = "azwe-rg-devtest-logs-001"
$WorkspaceName = "azwe-devtest-logs-01"
$Location = "westeurope"

# List of solutions to enable
$Solutions = "CapacityPerformance", "LogManagement", "ChangeTracking", "ProcessInvestigator"

# Create the resource group if needed
try {
    Get-AzResourceGroup -Name $ResourceGroup -ErrorAction Stop
} catch {
    New-AzResourceGroup -Name $ResourceGroup -Location $Location
}

# Create the workspace
New-AzOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -ResourceGroupName $ResourceGroup

# List all solutions and their installation status
Get-AzOperationalInsightsIntelligencePacks -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Add solutions
foreach ($solution in $Solutions) {
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -IntelligencePackName $solution -Enabled $true
}

# List enabled solutions
(Get-AzOperationalInsightsIntelligencePacks -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName).Where({($_.enabled -eq $true)})

# Enable IIS Log Collection using the agent
Enable-AzOperationalInsightsIISLogCollection -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Windows Event
New-AzOperationalInsightsWindowsEventDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -EventLogName "Application" -CollectErrors -CollectWarnings -Name "Example Application Event Log"
```

**Important:** Record the WorkspaceId from the output—you'll use it in subsequent steps.

### Step 2: Retrieve Workspace Keys

Retrieve the Log Analytics workspace secure key. This key authenticates agents connecting to the workspace.

```powershell
Get-AzOperationalInsightsWorkspaceSharedKey `
    -ResourceGroupName azwe-rg-devtest-logs-001 `
    -Name azwe-devtest-logs-01
```

**What is the workspace key?**
- Authentication credential: Proves agent is authorized to send data to the workspace
- Shared secret: Used by all agents connecting to this workspace
- Security consideration: Should be rotated periodically and stored securely
- Regeneration: Can be regenerated if compromised

### Step 3: Connect VMs to Log Analytics Workspace

Map existing virtual machines with the Log Analytics workspace by installing the Microsoft Monitoring Agent (MMA) extension.

**Before running:** Replace the variable values:
- `$ResourceGroupName`: Resource group containing the VM
- `$VMName`: Name of the virtual machine to monitor
- `$Location`: Azure region where the VM is located
- `$PublicSettings["workspaceId"]`: WorkspaceId from Step 1
- `$ProtectedSettings["workspaceKey"]`: Workspace key from Step 2

```powershell
$PublicSettings = @{"workspaceId" = "<myWorkspaceId>"}
$ProtectedSettings = @{"workspaceKey" = "<myWorkspaceKey>"}
$ResourceGroupName = "azwe-rg-devtest-logs-001"
$VMName = "azsu-d-sql01-01"
$Location = "westeurope"

Set-AzVMExtension -ExtensionName "Microsoft.EnterpriseCloud.Monitoring" `
    -ResourceGroupName $ResourceGroupName `
    -VMName $VMName `
    -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
    -ExtensionType "MicrosoftMonitoringAgent" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location $Location
```

**If you have multiple subscriptions:**
```powershell
Set-AzContext -SubscriptionId "<subscription-id>"
```

**If the extension fails:**
```powershell
Remove-AzVMExtension -ResourceGroupName $ResourceGroupName -VMName $VMName -Name "Microsoft.EnterpriseCloud.Monitoring"
```

Then retry the extension installation command.

### Step 4: Configure Performance Counter Collection

Configure performance counters to be collected from connected virtual machines. This adds dozens of performance counters covering system, network, and SQL Server metrics.

**Performance Counter Categories:**
- System: Processor Queue Length (overall system health)
- Processor: % Processor Time (CPU utilization per core)
- Memory: Available MBytes, Page Faults/sec (memory pressure detection)
- LogicalDisk: Disk Transfers/sec, Avg. Disk sec/Read (disk I/O performance)
- Network: Bytes Received/sec, Bytes Sent/sec (network throughput)
- Process: % Processor Time (per-process resource usage)
- SQL Agent: Activated alerts, Failed jobs (SQL Server agent health)
- SQL Server: Lock escalations, Deadlocks/sec (database performance)

**Collection Interval Considerations:**
- 10 seconds: High resolution for troubleshooting (higher cost)
- 60 seconds: Standard interval for operational monitoring (balanced)
- 300 seconds (5 minutes): Low resolution for capacity planning (lower cost)

See the original Microsoft Learn module for the complete performance counter configuration script.

### Step 5: Generate Test Data

Download the HeavyLoad utility (a free load testing utility) and run it on the virtual machine to simulate high CPU, memory, and IOPS consumption.

**Why Generate Test Load?**
- Validate monitoring: Confirm that performance counters are being collected
- Observe correlation: See how different metrics correlate under load
- Test alerting: Verify that alerts fire when thresholds are exceeded
- Understand baseline: Compare loaded vs. unloaded performance
- Practice investigation: Use realistic data for learning KQL queries

**Recommended Test Procedure:**
1. RDP into the VM
2. Download HeavyLoad (jam-software.com/heavyload)
3. Run baseline collection (5-10 minutes with no load)
4. Start moderate load (CPU 50%, Memory 50% for 5 minutes)
5. Increase load (CPU 80%, Memory 70% for 5 minutes)
6. Run heavy load (CPU 95%, Memory 85% for 5 minutes)
7. Stop load (allow 5-10 minutes return to baseline)
8. Query data (after 10-15 minutes for data latency)

**Data Ingestion Latency:**
- Typical latency: 5-10 minutes from event occurrence to queryability
- Can be longer: During high volume periods, up to 15 minutes
- Use timestamp: Query by TimeGenerated field (event occurrence time), not ingestion time

## How It Works

### Microsoft Monitoring Agent Architecture

The Microsoft Monitoring Agent service runs on machines and includes several components:

**1. Data Collection:**
- Performance counter collector: Samples metrics at configured intervals
- Event log reader: Monitors Windows event logs for new entries
- IIS log parser: Processes web server logs
- Custom data sources: Collects user-defined logs and metrics

**2. Local Processing:**
- Filtering: Apply collection rules to determine what data to send
- Parsing: Extract structured data from unstructured logs
- Enrichment: Add context (computer name, timestamp, resource ID)
- Buffering: Store events locally in case of network issues

**3. Secure Transmission:**
- TLS encryption: All data encrypted in transit using TLS 1.2+
- Authentication: Workspace ID and key authenticate the agent
- Compression: Reduce bandwidth by compressing data
- Batching: Send multiple events together for efficiency

**4. Reliability Features:**
- Local buffer: 10 GB disk buffer for storing events during outages
- Automatic retry: Retry failed transmissions with exponential backoff
- Health monitoring: Agent reports its own health status
- Automatic recovery: Restart service if it crashes

### Validating Agent Configuration

To verify agent installation and configuration:

1. Log in to the virtual machine via RDP
2. Navigate to `C:\Program Files\Microsoft Monitoring Agent\MMA`
3. Open the agent control panel (`control panel.exe Microsoft Monitoring Agent`)
4. Review the Azure Log Analytics tab showing:
   - Connected workspaces: Workspace ID and connection status
   - Computer name: How the VM is identified in logs
   - Agent version: Current MMA version installed
   - Last heartbeat: When agent last communicated with the workspace

### Multi-Workspace Capability

You can add multiple Log Analytics workspaces to publish log data into various workspaces. This is useful for:

**Centralized and Delegated Monitoring:**
- Central IT workspace: All VMs report to a central workspace
- Application team workspace: Same VMs report to application-specific workspaces
- Security workspace: Security logs sent to a separate SIEM-integrated workspace

**Multi-Tenant Scenarios:**
- Managed service provider: Customer VMs report to both MSP and customer workspaces
- Cost allocation: Separate workspaces for accurate usage tracking per business unit

**Compliance Requirements:**
- Regional data residency: Different workspaces in different regions
- Retention policies: Send security logs to long-retention workspace

**Considerations:**
- Increased cost: Data ingestion charged in each workspace
- Agent overhead: Additional CPU and network for sending to multiple destinations
- Configuration complexity: Maintain consistent collection rules across workspaces

## Summary

### What You've Accomplished

By completing this tutorial, you've built a complete Log Analytics monitoring solution:

**Infrastructure Setup:**
- Created a Log Analytics workspace with appropriate naming conventions
- Configured intelligence packs for specialized monitoring scenarios
- Enabled multiple data sources (IIS logs, Windows event logs)
- Retrieved workspace keys for secure agent authentication

**Agent Deployment:**
- Installed Microsoft Monitoring Agent extension on a virtual machine
- Configured secure workspace connection using workspace ID and key
- Validated agent installation through the agent control panel
- Understood multi-workspace capabilities for complex scenarios

**Data Collection Configuration:**
- Configured comprehensive performance counter collection (system, application, process-level)
- Set sampling intervals appropriate for monitoring needs
- Automated configuration using PowerShell scripts

**Testing and Validation:**
- Generated realistic test data using load testing utilities
- Stressed multiple system components to observe monitoring behavior
- Understood data ingestion latency
- Prepared performance data for analysis

### Next Steps

The next unit covers Kusto Query Language (KQL)—the powerful query language used to analyze telemetry and extract actionable insights:

- Write KQL queries to filter and analyze performance data
- Create visualizations from query results
- Identify performance bottlenecks using collected metrics
- Build alerts based on query conditions
- Create dashboards for real-time monitoring

### Key Takeaways

**Infrastructure as Code Approach:**
- All configuration performed via PowerShell scripts
- Scripts can be version controlled and reused
- Automation eliminates manual configuration errors
- Same scripts work across dev, test, and production

**Agent-Based Monitoring:**
- Microsoft Monitoring Agent runs as a Windows service
- Local buffering ensures data isn't lost during outages
- Secure encrypted communication protects telemetry
- Multi-workspace capability supports complex organizational structures

**Performance Counter Collection:**
- Hundreds of counters available for monitoring
- Sampling interval balances detail vs. cost
- Instance selection allows granular or aggregate collection
- Application-specific counters (SQL Server, IIS) provide deep insights

**Data Latency Considerations:**
- Typical 5-10 minute delay from event to queryability
- Use TimeGenerated field for event occurrence time
- Local buffering handles temporary connectivity issues
- Plan alerting and analysis workflows accounting for this latency
