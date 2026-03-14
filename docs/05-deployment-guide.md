# Deployment Guide — Dev/Test/Prod Promotion

## Environment Overview

| Environment | Purpose | URL Pattern | Access |
|---|---|---|---|
| Dev | Active development and unit testing by SeayInsights team | `org.crm.dynamics.com/orgDEV` | Developers only |
| Test (UAT) | Client acceptance testing and stakeholder sign-off | `org.crm.dynamics.com/orgTEST` | Developers + Key Users + Operations Manager |
| Production | Live operations — all licensed users | `org.crm.dynamics.com/orgPROD` | All licensed users |

## Solution Export Steps

Always export as a **managed** solution for Test and Production environments. Use unmanaged only in Dev.

1. Navigate to [make.powerapps.com](https://make.powerapps.com) and select the **Dev** environment
2. Go to **Solutions** → select `FieldOpsDigitization` solution
3. Click **Export** → select **Managed** → confirm "Export as a managed solution"
4. Verify the version number is incremented (e.g., `1.0.0.1` → `1.0.0.2`)
5. Click **Export** and download the `.zip` file
6. Label the file with version and date: `FieldOpsDigitization_1_0_0_2_managed_2025-06-15.zip`
7. Store in the project SharePoint folder: `/FieldOps/Deployments/`

## Solution Import Steps

1. Navigate to [make.powerapps.com](https://make.powerapps.com) and select the **Target** environment (Test or Prod)
2. Go to **Solutions** → **Import solution** → upload the managed `.zip` file
3. Review the import summary — check for any missing dependency warnings
4. On the **Connection References** step, map each connection reference to an existing connection in the target environment (see Connection References section below)
5. On the **Environment Variables** step, confirm all values are correct for the target environment (see Environment Variables section below)
6. Click **Import** and wait for completion (typically 3–8 minutes)
7. Review the import log — confirm no errors or skipped components before proceeding to validation

## Environment Variables

| Variable Name | Dev Value | Test Value | Prod Value | Notes |
|---|---|---|---|---|
| `NAV_BASE_URL` | `http://navdev:7048/BC` | `https://navtest.acme.com:7048/BC` | `https://nav.acme.com:7048/BC` | NAV OData endpoint base URL |
| `NAV_COMPANY` | `CRONUS` | `ACME_TEST` | `ACME` | NAV company name as it appears in OData URL |
| `SHAREPOINT_SITE_URL` | `https://acme.sharepoint.com/sites/FieldOpsDev` | `https://acme.sharepoint.com/sites/FieldOpsTest` | `https://acme.sharepoint.com/sites/FieldOps` | SharePoint site URL for all list operations |
| `APPROVAL_THRESHOLD` | `9999999` | `500` | `500` | Dollar amount above which 2-level approval is required |
| `AI_MODEL_ID` | `dev-model-guid-xxxx` | `test-model-guid-xxxx` | `prod-model-guid-xxxx` | Published AI Builder model ID — retrieve from AI Builder model details page |
| `ERROR_NOTIFY_EMAIL` | `dev@seayinsights.com` | `test-ops@acme.com` | `ops@acme.com` | Email address for error notification recipients |
| `NAV_TENANT_ID` | `dev-tenant-guid` | `test-tenant-guid` | `prod-tenant-guid` | Azure AD tenant ID for OAuth token requests |
| `NAV_CLIENT_ID` | `dev-app-client-id` | `test-app-client-id` | `prod-app-client-id` | Azure AD app registration client ID |

> **Important:** Never store client secrets in environment variables. Use Azure Key Vault references for production secrets, or the Power Platform connection reference credential store.

## Connection References

After import, update connection references in the target environment by navigating to **Solutions → FieldOpsDigitization → Connection References**:

| Connection Reference | Connector | Service Account | Notes |
|---|---|---|---|
| `FieldOps_SharePoint` | SharePoint | `svc-powerautomate@acme.com` | Must have Contribute access to the FieldOps SharePoint site |
| `FieldOps_Teams` | Microsoft Teams | `svc-powerautomate@acme.com` | Must be a member of the Operations Manager Teams channel |
| `FieldOps_NAV_HTTP` | HTTP (Premium) | Azure AD app registration | OAuth bearer token — credentials in environment variables |
| `FieldOps_AIBuilder` | AI Builder | `svc-powerautomate@acme.com` | Must have AI Builder license assigned in target environment |
| `FieldOps_Outlook` | Office 365 Outlook | `svc-powerautomate@acme.com` | For error notification emails |

After updating each connection reference, turn all flows in the solution **On** and run the post-deployment validation checklist.

## Post-Deployment Validation Checklist

- [ ] Submit a test work order via the Power Apps form — verify it appears in the SharePoint FieldOps list with Status = Pending
- [ ] Confirm the AI Builder extraction action runs — check the flow run history for confidence scores
- [ ] Verify the NAV write-back creates the correct Work Order record in NAV
- [ ] Confirm the SharePoint entry status updates to Completed after successful NAV write-back
- [ ] Test the validation fail path: submit a record with an empty Work Order Number — confirm Teams notification reaches the test technician account
- [ ] Test the error path: temporarily break the NAV URL in the environment variable — submit a record and confirm the error notification reaches the Operations Manager Teams channel and the SharePoint Error Log is updated
- [ ] Restore the NAV URL environment variable after error path test
- [ ] Confirm the scheduled retry flow activates and re-processes the Failed entry created during error path test
- [ ] Run the SharePoint permissions check — confirm that test supplier accounts cannot see records from other suppliers

## Rollback Procedure

If a deployment to Production causes issues:

1. **Immediately disable all flows** in the FieldOpsDigitization solution (do not delete — disabling preserves run history)
2. Communicate downtime to affected users and Operations Manager
3. Import the previous managed solution version (from the project SharePoint Deployments folder) to overwrite the failed deployment
4. Restore any environment variable values that were changed during the failed deployment
5. Re-enable all flows
6. Run the Post-Deployment Validation Checklist in full
7. Document the incident in the project change log: date, version deployed, issue description, resolution, and sign-off from Operations Manager
