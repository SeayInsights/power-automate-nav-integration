# Scope Addendum — Multi-Supplier Delivery Management

## Context

The base engagement scope (as defined in the Master Services Agreement) covers a single-supplier delivery workflow: one primary vendor submitting delivery dockets via the Power Apps portal, with Power Automate processing each submission and writing back to Microsoft Dynamics NAV.

This addendum extends the engagement to support multi-supplier scenarios, where multiple vendors submit delivery dockets with differing formats, approval chains, and NAV posting logic. The client has identified four additional suppliers who will be onboarded to the portal within 90 days of go-live.

## Additional Scope Items

- **Supplier portal row-level security (RLS)** — Extend the Power Apps portal with Azure AD group-based RLS so each supplier account can only view and manage their own submissions. Supplier accounts are provisioned as Azure AD B2B guest users.
- **Supplier profile management application** — Build a canvas app allowing the operations team to onboard new suppliers, assign document layout templates, set approval thresholds, and manage contact information. Data stored in a SharePoint Supplier List.
- **Multi-template AI Builder model training** — Train a separate AI Builder document extraction model for each supplier's delivery docket layout. This addendum covers up to five additional supplier templates beyond the base scope model.
- **Parallel approval flow routing** — Extend the Power Automate hub flow with supplier-specific approval chains. Supplier A requires one-level approval for all submissions; Supplier B requires two-level approval for submissions above $5,000. Thresholds and approver assignments are configurable via the Supplier Profile SharePoint list.
- **NAV vendor matching and PO validation** — Add a lookup step to match inbound supplier portal logins to NAV Vendor records and validate that the submitted Purchase Order reference exists and is open in NAV before write-back proceeds.
- **Discrepancy reporting dashboard** — Build a SharePoint-based discrepancy dashboard showing delivery dockets where the received quantity differs from the PO quantity by more than a configurable tolerance (default: ±5%). Dashboard visible to Operations Manager; email alert sent daily for unresolved discrepancies older than 48 hours.

## Impact on Flow Architecture

The single-flow architecture is replaced by a hub-and-spoke model:

**Hub flow (`FieldOps-DeliveryHub`)**
- Receives all supplier submissions from the Power Apps portal
- Identifies the submitting supplier from the Azure AD B2B user context
- Routes the submission to the appropriate spoke flow based on the Supplier ID

**Spoke flows (one per supplier template)**
- `FieldOps-Spoke-SupplierA`
- `FieldOps-Spoke-SupplierB`
- `FieldOps-Spoke-SupplierC` (and additional spokes per onboarded supplier)
- Each spoke applies supplier-specific validation rules, runs the corresponding AI Builder model, and posts to NAV with the supplier-specific vendor posting group

**Error flow (unchanged)**
- Centralized error handler with supplier context added to all error log entries
- Error log SharePoint list updated to include `Supplier_ID` and `Supplier_Name` columns

## Additional Effort Estimate

| Item | Hours | Notes |
|---|---|---|
| Supplier portal RLS configuration | 8 | Azure AD B2B guest provisioning + SharePoint audience filtering |
| Supplier profile management app | 12 | Canvas app with CRUD against Supplier SharePoint list |
| AI Builder — 5 additional templates | 20 | 4 hrs per template: sample collection, tagging, training, evaluation |
| Hub-and-spoke flow architecture | 16 | Hub flow + 3 initial spoke flows; tested across all supplier scenarios |
| Multi-level approval flow configuration | 10 | Configurable thresholds via SharePoint; Teams adaptive card approvals |
| NAV vendor matching and PO validation | 6 | Lookup expression + fallback to manual assignment queue |
| Discrepancy reporting dashboard | 8 | Canvas app dashboard + SharePoint data source + daily email alert |
| Documentation and handoff | 4 | Updated flow docs, supplier onboarding guide, admin training |
| **Total** | **84** | **At $125/hr = $10,500** |

## Assumptions and Dependencies

- Client provides representative sample delivery dockets for each of the five additional supplier templates (minimum 10 documents per supplier)
- NAV Vendor records are current and include a consistent vendor ID format matching the Azure AD B2B user UPN prefix
- Azure AD B2B guest accounts are created and licensed before portal RLS configuration begins
- Client approval hierarchy is documented and formally signed off before approval flow build begins
- Supplier onboarding after the initial five is out of scope and will be quoted separately per supplier

## Sign-Off

This addendum is subject to written approval before work begins. Approved scope and hours will be added to the active Statement of Work.

| Party | Signature | Date |
|---|---|---|
| SeayInsights LLC | _________________________________ | ___________ |
| Client — Authorized Signatory | _________________________________ | ___________ |
