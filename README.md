# microsoft-sentinel-foundations

# Microsoft Sentinel Foundations

## Project Overview

This project demonstrates the basic setup and use of Microsoft Sentinel as a cloud-native SIEM solution in Azure.

The goal was to create a Log Analytics workspace, enable Microsoft Sentinel, connect Azure Activity Logs, and use KQL queries to investigate Azure administrative activity.

## Objectives

- Create a resource group for the Sentinel lab
- Create a Log Analytics workspace
- Enable Microsoft Sentinel
- Configure Azure Activity Log ingestion
- Generate test Azure activity
- Query security data using KQL
- Understand the relationship between alerts, incidents, and investigations

## Architecture

```text
Azure Subscription
        |
        v
Azure Activity Logs
        |
        v
Diagnostic Setting
        |
        v
Log Analytics Workspace
        |
        v
Microsoft Sentinel
        |
        v
KQL Queries / Analytics Rules / Incidents
```

## Resources Created

- Resource Group: `rg-sentinel-foundations`
- Log Analytics Workspace: `law-sentinel-foundations-10223`
- Microsoft Sentinel workspace
- Azure Activity diagnostic setting
- Temporary test resources used to generate activity logs

## Main Azure CLI Commands

### Create the resource group

```bash
az group create \
  --name rg-sentinel-foundations \
  --location canadacentral
```

### Create the Log Analytics workspace

```bash
az monitor log-analytics workspace create \
  --resource-group rg-sentinel-foundations \
  --workspace-name law-sentinel-foundations-10223 \
  --location canadacentral
```

### Get the workspace resource ID

```bash
LAW_ID=$(az monitor log-analytics workspace show \
  --resource-group rg-sentinel-foundations \
  --workspace-name law-sentinel-foundations-10223 \
  --query id \
  --output tsv)
```

### Verify the workspace ID

```bash
echo "$LAW_ID"
```

### Configure Azure Activity Log ingestion

```bash
MSYS_NO_PATHCONV=1 az monitor diagnostic-settings subscription create \
  --name "send-activity-to-sentinel" \
  --workspace "$LAW_ID" \
  --logs '[{"category":"Administrative","enabled":true}]'
```

### Verify the diagnostic setting

```bash
az monitor diagnostic-settings subscription show \
  --name "send-activity-to-sentinel" \
  --output jsonc
```

## Generating Test Activity

A test resource group was created and modified to generate Azure administrative events.

```bash
az group create \
  --name rg-sentinel-ingestion-test \
  --location canadacentral
```

```bash
az group update \
  --name rg-sentinel-ingestion-test \
  --set tags.Test=Sentinel
```

Azure Activity Logs can be checked using:

```bash
az monitor activity-log list \
  --resource-group rg-sentinel-ingestion-test \
  --offset 1h \
  --output table
```

## KQL Queries

### Display recent Azure Activity events

```kusto
AzureActivity
| order by TimeGenerated desc
| take 20
```

### Display useful activity information

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| project
    TimeGenerated,
    Caller,
    CallerIpAddress,
    OperationNameValue,
    ActivityStatusValue,
    ResourceGroup,
    _ResourceId
| order by TimeGenerated desc
```

### Count operations by type

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| summarize EventCount=count() by OperationNameValue
| order by EventCount desc
```

### Count activity by user or identity

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| summarize Operations=count() by Caller
| order by Operations desc
```

## Troubleshooting

### AzureActivity table is not found

If the following error appears:

```text
The name 'AzureActivity' does not refer to any known table
```

verify that Azure Activity Logs are being exported to the correct Log Analytics workspace.

Check the diagnostic setting:

```bash
az monitor diagnostic-settings subscription list \
  --output table
```

### AzureActivity exists but returns no results

Verify that Azure itself has recorded activity:

```bash
az monitor activity-log list \
  --offset 1h \
  --output table
```

Then confirm that the diagnostic setting points to the correct Log Analytics workspace and that the `Administrative` category is enabled.

New diagnostic settings may also require some time before logs begin appearing.

### Git Bash changes Azure resource IDs

Git Bash may modify paths beginning with `/subscriptions/...`.

Use:

```bash
MSYS_NO_PATHCONV=1
```

before Azure CLI commands that use full Azure resource IDs.

## Skills Practiced

This project provided hands-on experience with:

- Microsoft Sentinel
- Azure Monitor
- Log Analytics
- Azure Activity Logs
- Diagnostic Settings
- Kusto Query Language (KQL)
- Security monitoring
- SIEM concepts
- Azure CLI
- Basic Sentinel troubleshooting

## Key Learning

Microsoft Sentinel works by collecting security and operational telemetry from Azure and other systems into Log Analytics.

KQL can then be used to investigate this data, while Sentinel analytics rules can automatically detect suspicious activity and generate security incidents.

The basic workflow is:

```text
Collect
  ↓
Store
  ↓
Query
  ↓
Detect
  ↓
Alert
  ↓
Investigate
  ↓
Respond
```

## Cleanup

When the project is complete, the resources can be removed with:

```bash
az group delete \
  --name rg-sentinel-foundations \
  --yes \
  --no-wait
```

Any separate test resource groups should also be removed.

## Conclusion

This project established the foundations of working with Microsoft Sentinel by connecting Azure Activity Logs to a Log Analytics workspace and analyzing Azure administrative events with KQL.

It provides a starting point for more advanced Sentinel projects involving analytics rules, incidents, threat hunting, automation, playbooks, and SOC monitoring.