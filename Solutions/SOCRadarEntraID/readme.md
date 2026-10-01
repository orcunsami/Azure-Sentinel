# SOCRadar Entra ID Integration for Microsoft Sentinel

Responds to leaked employee credentials found by SOCRadar Dark Web Monitoring. It pulls Botnet Data, PII Exposure and VIP Protection records from the SOCRadar XTI Platform, looks the matching identities up in Microsoft Entra ID and takes the remediation actions you enable.

```
SOCRadar Platform (3 sources)
       |
       v
 Azure Function App (Python 3.11)
       |
   +---+--------------+
   |                  |
Microsoft Graph   Microsoft Sentinel (Log Analytics)
 look up           SOCRadar_Botnet_CL
 revoke            SOCRadar_PII_CL
 disable           SOCRadar_VIP_CL
 MFA wipe          SOCRadar_EntraID_Audit_CL
 group add         4 workbooks
 password change   incidents (optional)
```

## Data sources

| Source | Description |
|--------|-------------|
| Botnet Data v2 | Employee credentials harvested from botnet logs |
| PII Exposure v2 | Employee credentials from third-party data breaches |
| VIP Protection v2 | VIP users found across dark web sources |

## Remediation actions

Every action is a switch. Defaults: user lookup and session revocation on, everything else off.

- Revoke active sign-in sessions
- Add to or remove from a quarantine security group
- Disable or re-enable the account
- Force password change at next sign-in
- Force MFA re-registration (deletes all non-password authentication methods)
- Mark the user as compromised in Microsoft Entra ID Protection (needs P1/P2)
- Create a Microsoft Sentinel incident (give the managed identity the Microsoft Sentinel Responder role on the workspace)
- Resolve the SOCRadar alarm after remediation

## Authentication

Secretless. The Function App uses a user-assigned managed identity with a federated credential on an App Registration. No client secret to rotate.

## What gets installed

- 4 workbooks: Botnet Data, PII Exposure, VIP Protection, Combined
- 1 Function App playbook: timer trigger (default every 6 hours), writes to four custom tables through a Direct Data Collection Rule

## Before you install

- A Microsoft Sentinel workspace, or let the playbook create one
- SOCRadar Platform API key and Company ID
- An App Registration that already holds admin consent for the Microsoft Graph application permissions of the actions you enable. The playbook does not create it. If your tenant already has a consented SOCRadar App Registration, use it instead of creating a second one.
- After the playbook is deployed, add the federated credential for the new managed identity with the command in the `ficCommandToRun` output, as an owner of the App Registration. Without it the Function App cannot get a Graph token.

## Microsoft Graph permissions

| Permission | Needed for |
|------------|------------|
| `User.Read.All` | User lookup |
| `User.RevokeSessions.All` | Revoke sessions |
| `GroupMember.ReadWrite.All` | Group add and remove |
| `User-PasswordProfile.ReadWrite.All` | Force password change |
| `User.EnableDisableAccount.All` | Disable and enable account |
| `IdentityRiskyUser.ReadWrite.All` | Confirm compromised (Entra ID P1/P2) |
| `UserAuthenticationMethod.ReadWrite.All` | Force MFA re-registration (also needs the Privileged Authentication Administrator role) |

Grant only what the actions you enable need.

## Parameters

Source switches, action switches, polling interval, lookback window, tenant list and hosting plan are parameters of the playbook with conservative defaults. See the playbook readme and the standalone repository: https://github.com/orcunsami/SOCRadar-Azure-Entra-ID

## Support

- Email: integration@socradar.io
- SOCRadar Platform docs: https://docs.socradar.io
