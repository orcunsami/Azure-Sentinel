# SOCRadar-TAXII-FunctionApp

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FSOCRadarTAXII%2FPlaybooks%2FSOCRadar-TAXII-FunctionApp%2Fazuredeploy.json)

Imports STIX 2.1 indicators from the SOCRadar TAXII 2.1 server into Microsoft Sentinel threat intelligence with an Azure Function App.

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| WorkspaceName | - | Microsoft Sentinel workspace name |
| DeployNewWorkspace | true | Create the workspace when it does not exist; an existing one is never modified. Set false so a misspelled name fails before anything is created |
| WorkspaceResourceGroup | deployment resource group | Set when the workspace is in another resource group (Microsoft Sentinel must already be enabled there) |
| ApiRoots | - | Comma-separated TAXII API root names |
| CollectionIds | - | Comma-separated collection UUIDs, same order as ApiRoots |
| TAXIIUsername | - | TAXII username issued by SOCRadar, for example radar_330 |
| TAXIIPassword | - | SOCRadar Platform API Key |
| PollingIntervalMinutes | 60 | Polling interval, 5-1440 |
| InitialLookbackHours | 48 | Hours of history on the first run, 0 for all |
| EnableAuditLogging | true | Log to SOCRadar_TAXII_Audit_CL and install the workbook |
| PackageUri | published release | Function App package, read once at install time |

## What It Creates

Function App, storage account (checkpoints), Application Insights, user-assigned managed identity with its role assignments, and when audit logging is on the audit table, DCE, DCR and the workbook. A deployment script triggers the first import.
