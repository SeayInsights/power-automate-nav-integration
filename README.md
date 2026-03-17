# Power Automate + NAV Integration

> **Portfolio Reference** — Architecture and technical documentation for a Power Automate + Microsoft Dynamics NAV field operations integration. This repo demonstrates enterprise-level integration design for ERP write-back pipelines using the Power Platform.

## Overview

This repository documents the architecture and design decisions for digitizing field operations workflows — replacing paper-based processes with a **Power Apps mobile form → Power Automate → NAV write-back** pipeline.

It covers the full integration stack: flow design, AI Builder document processing, NAV OData API integration, and multi-environment deployment.

## Architecture Highlights

- **Integration Method:** NAV OData v4 API — chosen over SOAP and Direct SQL for maintainability, security, and long-term supportability
- **Auth:** OAuth 2.0 via Azure AD app registration
- **Platform:** Power Automate Premium + AI Builder credits
- **Pattern:** Event-driven, low-code pipeline with structured error handling and dev/test/prod promotion path

## Repository Structure

```
docs/
├── 01-flow-diagram.md           # Power Automate flow architecture
├── 02-ai-builder-walkthrough.md # AI Builder document processing guide
├── 03-json-to-csv-transform.md  # NAV OData → Power Automate data transform
├── 04-addendum-multi-supplier.md # Multi-supplier scope addendum
└── 05-deployment-guide.md       # Dev/Test/Prod deployment steps

architecture/
└── nav-integration-architecture.md  # NAV integration methods comparison
```

## Key Design Decisions

**Why OData v4 over SOAP or Direct SQL?**
OData v4 exposes NAV/Business Central data as REST endpoints, keeps the integration maintainable across NAV upgrades, and fits cleanly into Power Automate's HTTP connector model. SOAP adds brittle WSDL coupling; Direct SQL bypasses NAV's business logic layer entirely.

**Why AI Builder for document processing?**
Field operations workflows typically involve mixed paper/digital inputs. AI Builder's form processing model handles unstructured document extraction without custom ML infrastructure.

**Multi-supplier extensibility**
The architecture includes a multi-supplier addendum — the pipeline was designed to scale beyond a single supplier context with parameterized flow templates.

## Related

- [seayinsights-proposal-toolkit](https://github.com/SeayInsights/seayinsights-proposal-toolkit) — Proposal generation tooling used in this engagement

---

Built by [SeayInsights](https://github.com/SeayInsights) · Power Platform · Microsoft Dynamics NAV · Azure AD
