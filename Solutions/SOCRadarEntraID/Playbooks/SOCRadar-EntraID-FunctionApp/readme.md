# SOCRadar-EntraID-FunctionApp

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Sentinel%2Fmaster%2FSolutions%2FSOCRadarEntraID%2FPlaybooks%2FSOCRadar-EntraID-FunctionApp%2Fazuredeploy.json)

Pulls leaked employee credentials from SOCRadar (Botnet Data, PII Exposure, VIP Protection), looks the users up in Microsoft Entra ID and takes the remediation actions you enable. Findings go to four custom tables in the Microsoft Sentinel workspace.

## Before you deploy

The App Registration must already exist and hold admin consent for the Microsoft Graph application permissions of the actions you enable (see the solution readme). This template does not create it.

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| WorkspaceName | - | Microsoft Sentinel workspace name |
| CreateWorkspace | true | Create the workspace when it does not exist; an existing one is never modified. Set false so a misspelled name fails before anything is created |
| WorkspaceResourceGroup | deployment resource group | Set when the workspace is in another resource group (Microsoft Sentinel must already be enabled there) |
| SocradarApiKey | - | SOCRadar Platform API key |
| SocradarCompanyId | - | SOCRadar Company ID |
| EntraIdClientId | - | Client ID of the existing, consented App Registration |
| SkipFicCreation | true | true: add the federated credential yourself with the command in the `ficCommandToRun` output. false: a deployment script adds it, which needs the Application.ReadWrite.OwnedBy permission on top of ownership |
| EntraIdTenantId / EntraIdTenantIds | current tenant | One tenant, or a comma-separated list to monitor several |
| EnableBotnetSource / EnablePiiSource / EnableVipSource | true / true / false | Which SOCRadar sources to read |
| EnableUserLookup, EnableRevokeSession | true | Look the user up and revoke sessions. The other actions are off by default |
| PollingIntervalHours | 6 | Polling interval, 1-24 |
| HostingPlanSku | Y1 | Y1 is best-effort for timers; B1 with Always On is the production choice |
| PackageUri | published release | Function App package, read once at install time |

The full list is in the standalone repository.

## What it creates

Function App, storage account (checkpoints and action ledger), Application Insights, user-assigned managed identity with its role assignments, four custom tables with a Direct Data Collection Rule, and a deployment script that stages the package and starts the first import. The workspace and the Microsoft Sentinel onboarding are created only when CreateWorkspace is true.

## After the deployment

Unless you set SkipFicCreation to false, run the command from the `ficCommandToRun` output as an owner of the App Registration. Without the federated credential the Function App cannot get a Microsoft Graph token.
