# SOCRadar TAXII Solution for Microsoft Sentinel

Ingests threat intelligence indicators (IOCs) from SOCRadar via TAXII 2.1 protocol into Microsoft Sentinel.

## Components

- **SOCRadar-TAXII-FunctionApp** - Azure Function App for TAXII 2.1 ingestion. Handles cursor-based pagination, STIX parsing, and Microsoft Sentinel TI upload. Also creates the audit table, data collection rule and workbook when `EnableAuditLogging` is true.
- **SOCRadar-TAXII-Infrastructure** - Creates the audit table and data collection infrastructure on its own, for a workspace you manage yourself.
- **Workbook** - Dashboard showing indicator types, collection statistics, ingestion trends, and audit monitoring.
- **Hunting Queries** - IOC overview and ingestion health monitoring.
- **Analytic Rule** - Reports a TAXII import run that left indicators behind.

## Architecture

- Azure Function App handles TAXII 2.1 collection polling, STIX pattern parsing, and Microsoft Sentinel TI batch upload
- Table Storage for cursor-based state persistence and checkpoint/resume across runs
- A run that cannot deliver indicators keeps its checkpoint and reports `PartialSuccess` in `SOCRadar_TAXII_Audit_CL`
- A run has a time budget of nine minutes, split across the collections. A first run with the default 48 hour lookback can pause, keep its place in the checkpoint and continue on the next run. The audit row stays `Success` and `ErrorMessage` reads `time budget reached after N pages, more pages pending, continues next run`; an empty `ErrorMessage` means the collection is caught up
- Optional audit logging via Data Collection Rules to SOCRadar_TAXII_Audit_CL

## Counting indicators

The TAXII server can send the same indicator on several pages of one fetch, and every copy is uploaded. `IndicatorsCreated` counts uploads Microsoft Sentinel accepted, and `ThreatIntelIndicators` is an append log with one row per upload, so `count()` over it overstates. For the number of different indicators count the ids:

```kql
ThreatIntelIndicators
| where SourceSystem == "SOCRadar TAXII"
| summarize Rows = count() by Id
| summarize Rows = sum(Rows), DistinctIndicators = count()
```

To list the collections that are still catching up:

```kql
SOCRadar_TAXII_Audit_CL
| where ErrorMessage has "time budget reached"
| order by TimeGenerated desc
```

## Prerequisites

- SOCRadar TAXII 2.1 credentials: the TAXII username issued by SOCRadar (for example `radar_330`, not the bare company ID), the API key, API root names and collection IDs
- Microsoft Sentinel workspace
- Owner or User Access Administrator on the resource group (the template creates role assignments)

## Deployment

Deploy the **SOCRadar-TAXII-FunctionApp** playbook. `WorkspaceName` is the Microsoft Sentinel workspace. By default (`DeployNewWorkspace=true`) the workspace is created when it does not exist and left untouched when it does; set it to `false` to make a misspelled name fail before anything is created. If the workspace is in another resource group, set `WorkspaceResourceGroup`; Microsoft Sentinel must already be enabled there.

The package is downloaded once, at install time, from `PackageUri`. Redeploy to pick up a later release.
