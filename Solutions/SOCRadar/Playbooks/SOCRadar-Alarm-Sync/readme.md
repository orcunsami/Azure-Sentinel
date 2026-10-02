# SOCRadar Alarm Sync

Syncs closed Microsoft Sentinel incidents back to SOCRadar with classification mapping.

## Features

- Looks at SOCRadar-tagged incidents closed in the last 24 hours that have no Synced tag yet, so a closure whose write failed is retried on every run
- If a page of the closed-incidents query fails, the run ends Failed (`SyncIncomplete`) and the next run starts over
- Maps Microsoft Sentinel classification to SOCRadar status
- Synced tag prevents duplicate sync operations
- Configurable polling interval (the lookback is fixed at 24 hours)

## Classification Mapping

| Microsoft Sentinel Classification | SOCRadar Status |
|------------------------|-----------------|
| FalsePositive | FALSE_POSITIVE |
| BenignPositive | MITIGATED |
| TruePositive | RESOLVED |
| Undetermined | RESOLVED |

## Deployment

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FSOCRadar%2FPlaybooks%2FSOCRadar-Alarm-Sync%2Fazuredeploy.json)

You can also install this playbook via **Microsoft Sentinel Content Hub**.

## Prerequisites

- SOCRadar XTI Platform API Key
- Microsoft Sentinel workspace with SOCRadar incidents
- Managed Identity with Microsoft Sentinel Responder role (default, Contributor optional)
