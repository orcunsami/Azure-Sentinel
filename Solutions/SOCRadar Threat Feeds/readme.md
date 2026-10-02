<img src="logo/socradar.svg" alt="SOCRadar" width="75px" height="75px">

# SOCRadar Threat Feeds for Microsoft Sentinel

## Overview

The SOCRadar Threat Feeds solution imports threat intelligence indicators from SOCRadar feed collections into Microsoft Sentinel TI using an Azure Function App. Supports recommended and custom feed collections with batch upload and checkpoint-based deduplication.

## Key Features

- Azure Function App with User-Assigned Managed Identity
- Batch upload to Microsoft Sentinel TI (uploadIndicators API)
- Checkpoint-based deduplication via Azure Table Storage
- Application Insights monitoring with step-by-step logging
- Automatic first run after deployment; `InitialLookbackDays` (default 30) limits the first run of a collection
- SOCRadar Threat Feeds Dashboard with indicator statistics
- 4 hunting queries for feed analysis
- Optional custom table logging and audit trail (`PartialSuccess` runs and failed-indicator counts are recorded)

## Prerequisites

- An existing Microsoft Sentinel workspace; the playbooks do not create one
- SOCRadar Platform API Key ([socradar.io](https://socradar.io))

## Installation

Install the SOCRadar Threat Feeds solution from the Microsoft Sentinel **Content Hub**.

1. In Microsoft Sentinel, go to **Content Hub**
2. Search for **SOCRadar Threat Feeds**
3. Select the solution and click **Install**
4. Deploy the Infrastructure playbook first
5. Deploy the Function App playbook with your API Key and the Infrastructure values (DCE endpoint and DCR immutable IDs)

The Infrastructure deployment outputs are shown when you use the **Deploy to Azure** button. When you install from Content Hub there are no outputs: take the DCE **Logs Ingestion** URI from the data collection endpoint `SOCRadar-Feeds-DCE`, and `immutableId` (JSON View) from the data collection rules `SOCRadar-Feeds-DCR` and `SOCRadar-Feeds-Audit-DCR`. Details are in the Infrastructure playbook readme.

The function automatically runs after deployment. Check Application Insights for execution logs.

## Playbooks

| Playbook | Description |
|----------|-------------|
| [SOCRadar-Feeds-FunctionApp](Playbooks/SOCRadar-Feeds-FunctionApp) | Azure Function App that imports SOCRadar feed indicators into Microsoft Sentinel TI |
| [SOCRadar-Feeds-Infrastructure](Playbooks/SOCRadar-Feeds-Infrastructure) | Creates custom tables and data collection infrastructure |

## About SOCRadar

SOCRadar is an Extended Threat Intelligence (XTI) platform that provides actionable threat intelligence, digital risk protection, and external attack surface management.

Learn more at [socradar.io](https://socradar.io)

## Support

- **Documentation:** [docs.socradar.io](https://docs.socradar.io)
- **Support:** integration@socradar.io
