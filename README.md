# Power Automate + NAV Integration

Documentation and technical reference for integrating Microsoft Power Automate with Microsoft Dynamics NAV (Navision) for field operations digitization.

## Project Context

This repository supports the **Field Operations Digitization** engagement — replacing paper-based workflows with a Power Apps mobile form → Power Automate → NAV write-back pipeline.

## Repository Structure

```
docs/
├── 01-flow-diagram.md          # Power Automate flow architecture
├── 02-ai-builder-walkthrough.md # AI Builder document processing guide
├── 03-json-to-csv-transform.md  # NAV OData → Power Automate data transform
├── 04-addendum-multi-supplier.md # Multi-supplier scope addendum
└── 05-deployment-guide.md       # Dev/Test/Prod deployment steps

architecture/
└── nav-integration-architecture.md  # NAV integration methods comparison
```

## Key Design Decisions

- **Integration Method:** NAV OData v4 API (over SOAP/Direct SQL) for maintainability and security
- **Auth:** OAuth 2.0 via Azure AD app registration
- **Platform:** Power Automate Premium + AI Builder credits

## Related Tools

- [seayinsights-proposal-toolkit](https://github.com/SeayInsights/seayinsights-proposal-toolkit) — Word/PDF proposal generator for this engagement

---

Built by [SeayInsights](https://github.com/SeayInsights)
