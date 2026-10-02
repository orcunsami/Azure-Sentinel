# SOCRadar Feeds Function App

Deploys an Azure Function App that imports threat intelligence indicators from SOCRadar feed collections into Microsoft Sentinel TI.

## Features

- 1 default recommended feed (APT Block Hash) + custom collections
- Batch upload to Microsoft Sentinel TI (uploadIndicators API)
- Checkpoint-based deduplication via Azure Table Storage
- User-Assigned Managed Identity (stable principal, survives restarts)
- Application Insights monitoring with step-by-step logging
- Automatic first run after deployment (deployment script restarts function)
- `InitialLookbackDays` (default 30): the first run of a collection sends only indicators last seen within that many days, `0` sends the whole feed
- `PackageUri`: the function code is downloaded once at install time, stored in the storage account and loaded from there; a redeploy picks up a newer package
- Optional feeds and audit table logging (requires Infrastructure playbook)

## Prerequisites

- Existing Microsoft Sentinel workspace in the resource group you deploy to
- SOCRadar Platform API Key
- Infrastructure playbook deployed first (for custom table logging)

## Deployment

Deploy the **SOCRadar-Feeds-Infrastructure** playbook first, then deploy this playbook.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FSOCRadar%20Threat%20Feeds%2FPlaybooks%2FSOCRadar-Feeds-FunctionApp%2Fazuredeploy.json)

You can also install this playbook via **Microsoft Sentinel Content Hub**.

## Post-Deployment

The function automatically runs after deployment. Check Application Insights for execution logs and the Invocations tab for status.

If an upload fails after some batches reached Microsoft Sentinel, the checkpoint is held, the run is recorded as `PartialSuccess` and the next run sends the indicators again. Nothing is lost and Microsoft Sentinel TI is unaffected (same ids), but the rows of the batches that already landed can appear twice in `SOCRadar_Feeds_CL`.

The dashboard tiles use `dcount`, which is approximate, so a tile can differ slightly from the Microsoft Sentinel TI count. For an exact count use `SOCRadar_Feeds_CL | distinct IndicatorValue | count`, or per collection `SOCRadar_Feeds_CL | distinct CollectionName, IndicatorValue | summarize count() by CollectionName`.

## Resources Created

- Azure Function App (Python 3.11, Consumption Plan)
- Application Insights (workspace-based, 30 day retention)
- Storage Account + FeedState table (checkpoint tracking)
- User-Assigned Managed Identity
- Role assignments: Microsoft Sentinel Contributor (on the workspace), Storage Table Data Contributor, Monitoring Metrics Publisher on each DCR, Website Contributor
- Deployment script: downloads the package, stages it, restarts the app and waits until a function is indexed
