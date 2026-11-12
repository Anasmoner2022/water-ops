# Water Ops

A portfolio delivery project for a fictional water utility: receive water-leak incidents, locate them within service zones, dispatch maintenance teams, execute a BPMN workflow, track response SLAs, and hand over an operational solution.

## Project status

Pre-implementation. This repository contains planning documents and Git configuration. Application code, database migrations, runnable infrastructure, and executable workflows have not been created.

## Planning documents

| Document | Purpose | Status |
| --- | --- | --- |
| [Product specification](docs/product-spec.md) | Scope, operating rules, US-01 to US-12, acceptance criteria, and UAT scenarios | Written; review before implementation |
| [Implementation backlog](docs/backlog.md) | Ordered tasks B01 to B46 and requirements traceability | Written; tasks not started |
| [Pre-coding checklist](docs/pre-coding-checklist.md) | Remaining design files, decisions, and the first implementation gate | Written |
| [Backend language proposal](docs/decisions/0001-backend-language.md) | Go recommendation, tradeoffs, and Camunda integration validation | Proposed |

## Proposed technology boundaries

- Go for the application API, integration adapters, and durable background workers.
- PostgreSQL and PostGIS for operational data and spatial queries.
- Camunda for executable process orchestration and response timers.
- GeoServer for published map layers.
- Apache Superset for operational analytics.
- React and Leaflet for the operator and technician interfaces.

The backend language is a proposal pending review. No framework, dependency versions, or Go module path has been locked in.

## First delivery milestone

Complete backlog tasks B01 to B24 to demonstrate one incident moving from creation through assignment, acceptance, repair, review, and closure. Later tasks complete map services, escalation, CRM delivery, analytics, UAT, training, and handover.

All customer names, assets, locations, incidents, and integrations used for the demo must be explicitly synthetic. Any workshop or acceptance exercise without a real customer must be described as a simulation.
