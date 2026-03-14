# NAV Integration Methods — Architecture Decision Record

## Context

The Field Operations Digitization engagement requires Power Automate to create and update records in Microsoft Dynamics NAV. Several integration methods are available. This document evaluates each method and records the final architectural decision.

**Decision:** Integrate Power Automate with NAV using the **OData v4 API** exposed through NAV Web Services.

## Comparison Table

| Method | Protocol | Auth | Performance | Maintainability | Verdict |
|---|---|---|---|---|---|
| OData v4 API | REST/HTTPS | OAuth 2.0 / Basic | Good | Excellent | **Recommended** |
| SOAP Web Services | SOAP/HTTPS | Basic / NTLM | Moderate | Good | Fallback option |
| NAV Codeunits (direct) | SOAP or REST | Basic | Excellent | Moderate | Use for complex write-back only |
| Direct SQL | TCP/SQL | SQL auth | Excellent | Poor | Avoid |
| RPA (UI Automation) | Screen/UI | User credentials | Poor | Very Poor | Last resort only |

## Detailed Evaluation

### OData v4 API

**How it works:** NAV exposes pages (tables/views) as OData v4 endpoints via NAV Web Services configuration. Power Automate calls these endpoints using the HTTP connector or the Dynamics 365 Business Central connector.

**Pros:**
- Native Power Platform support — no custom connectors required
- Supports standard CRUD operations (GET, POST, PATCH, DELETE)
- Respects all NAV business logic — validation, posting routines, and triggers execute normally
- OAuth 2.0 support via Azure AD app registration — no service account password rotation issues
- Well-documented NAV API reference available for most standard pages
- Version-stable — OData URLs persist across NAV minor version upgrades

**Cons:**
- Premium connector license required for HTTP connector in Power Automate
- Some complex NAV operations (e.g., posting a sales order) are not available as OData endpoints and require Codeunit calls
- On-premises NAV requires an on-premises data gateway, which adds infrastructure overhead
- Nested entity relationships (e.g., order headers + lines) require multiple API calls or careful $expand usage

**Verdict: Recommended.** Best balance of maintainability, security, and Power Platform native support.

---

### SOAP Web Services

**How it works:** NAV exposes objects (pages, codeunits, queries) as SOAP endpoints. Power Automate consumes these via the HTTP connector using SOAP XML envelopes.

**Pros:**
- Available on older NAV versions that do not support OData v4
- Codeunit exposure enables complex server-side logic
- Well-established in the NAV ecosystem

**Cons:**
- SOAP XML construction in Power Automate is verbose and error-prone
- No native connector — requires manual XML envelope building and parsing
- Basic/NTLM authentication — requires service account with password; no OAuth support on older NAV versions
- Harder to maintain as NAV upgrades change WSDL schemas

**Verdict: Fallback option.** Use only if the client's NAV version does not support OData v4 endpoints.

---

### NAV Codeunits (Direct)

**How it works:** Custom C/AL or AL codeunits are published as SOAP or OData endpoints. Power Automate calls these custom endpoints to execute server-side business logic.

**Pros:**
- Maximum flexibility — any NAV business logic can be encapsulated
- Ideal for complex posting routines (posting sales orders, creating journal entries)
- Can combine multiple NAV operations into a single API call, reducing round trips

**Cons:**
- Requires NAV developer resource to write and maintain AL code
- Custom code increases upgrade risk — must be tested on every NAV version upgrade
- Increases total solution complexity and vendor lock-in

**Verdict: Use for complex write-back only.** Standard CRUD operations should use OData v4. Codeunits are appropriate for posting operations (e.g., posting a sales order to a ledger) where a simple PATCH to an OData endpoint is insufficient.

---

### Direct SQL

**How it works:** Power Automate (via Power Automate Desktop or a custom connector) connects directly to the NAV SQL Server database using a SQL Server connection.

**Pros:**
- Maximum read performance for reporting queries
- Can access any table without requiring a published NAV web service

**Cons:**
- **Completely bypasses NAV business logic** — inserts and updates will not trigger NAV validation, posting routines, or triggers. Data integrity risk is severe.
- NAV database schema is undocumented and changes between versions — high maintenance burden
- Requires direct network access to the SQL Server — security concern
- Microsoft does not support direct SQL access to Business Central Online (cloud) at all

**Verdict: Avoid for write operations.** Direct SQL may be acceptable for read-only reporting extracts in on-premises scenarios, but must never be used for inserts or updates.

---

### RPA (UI Automation / Power Automate Desktop)

**How it works:** Power Automate Desktop records and replays UI interactions with the NAV Windows client or web client, driving the application via screen scraping and simulated keystrokes.

**Pros:**
- Can automate any NAV operation without API access
- No NAV configuration required

**Cons:**
- Extremely fragile — any NAV UI change (button rename, screen layout update) breaks the automation
- Very poor performance compared to API-based approaches
- Cannot run unattended without a dedicated machine with an active Windows session
- No error handling capability for NAV business rule violations
- Maintenance cost is very high

**Verdict: Last resort only.** Acceptable only when no API or web service access is available and the operation cannot be deferred. Should be replaced with an API-based approach as soon as access is provisioned.

---

## Final Recommendation

**Use OData v4 API as the primary integration method** for all standard create and update operations (work orders, delivery records, customer record lookups).

**Supplement with NAV Codeunits** for posting operations that cannot be completed via a standard OData write (e.g., posting a delivery journal, creating a finalized sales invoice).

## Implementation Notes for OData v4

1. **Publish NAV pages as web services:** In NAV, go to Web Services → New → select Page type, assign a Service Name (e.g., `WorkOrders`), and check Published.
2. **Azure AD app registration:** Create an app registration with `Dynamics NAV` API permission — `Financials.ReadWrite.All`. Generate a client secret and store in Power Platform environment variables.
3. **On-premises gateway:** If NAV is not cloud-hosted, install and configure the On-Premises Data Gateway on a server with network access to NAV. Register the gateway in the Power Platform Admin Center.
4. **OData URL format:** `https://{server}:{port}/{instance}/ODataV4/Company('{company_name}')/{ServiceName}`
5. **Content-Type header:** Always set `Content-Type: application/json` and `Accept: application/json` on POST/PATCH requests.
6. **ETag for updates:** NAV OData v4 requires an `If-Match` header with the current ETag value for PATCH requests. Retrieve the ETag with a prior GET request or use `If-Match: *` to skip the concurrency check (use with caution in production).
