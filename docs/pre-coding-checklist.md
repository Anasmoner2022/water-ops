# Water Ops Pre Coding Checklist

This checklist records what is already written and which design artifacts must be prepared before their related implementation tasks. Planned file paths below do not imply those files already exist.

## Existing documents

| File | Status | Review needed |
| --- | --- | --- |
| product-spec.md | Written | Confirm operating assumptions and open decisions |
| backlog.md | Written | Confirm first delivery scope and task dependencies |
| decisions/0001-backend-language.md | Proposed | Review Go selection and Camunda validation plan |

The stories, backlog and UAT baseline are planning material. No acceptance test has run and no product feature is implemented.

## Remaining design files

| Planned file | Required content | Needed before |
| --- | --- | --- |
| docs/business-rules.md | State/role/action matrix; SLA equality and race handling; reservation rules; boundary-zone choice; terminal behavior; manual-incident CRM policy | B01 completion |
| docs/architecture.md | Module boundaries, PostgreSQL/Camunda authority, durable intents/outbox, command idempotency, uncertain outcomes, reconciliation, auth boundary, map/dashboard access | B02-B03 and B12-B15 |
| docs/database-design.md | ERD, field types, foreign keys, uniqueness, one-open-assignment constraints, spatial indexes, audit/policy history, timestamp rules and retention assumptions | B03 |
| api/openapi.yaml | Real OpenAPI contract: auth, input/output schemas, error envelope, roles, pagination, filter rules, idempotency and operational commands | Related endpoint implementation |
| workflows/incident-lifecycle.bpmn | Valid Camunda-targeted BPMN with human/service tasks, review gateway, rework, stored-deadline timer, correlation and error/retry boundaries | B14; draft during B01 |
| docs/integration-contracts.md | CRM samples, mapping, normalized-payload comparison, uniqueness key, event schema/ID, auth, retry matrix, resend and deduplication | Intake identifiers in B03; implementation in B30-B34 |
| docs/test-plan.md | Story/criterion/test traceability, UAT steps, known-answer metrics, authorization/concurrency/recovery checks and evidence template | Design early; expand with each feature |
| data/samples/assets.csv | Valid rows and explicit invalid/duplicate examples with expected outcomes | B09 |
| data/samples/service-zones.geojson | Three synthetic valid zones, plus separate invalid/boundary test fixtures | B04/B08 |
| compose.yaml and .env.example | Pinned compatible services, development-only settings, ports, health checks, persistent volumes and secret placeholders | B02; expand as services are connected |

Do not create empty OpenAPI, BPMN or Compose placeholders that appear executable. Add each artifact when it has useful, internally consistent content.

## First coding gate

- [ ] Review scope and resolve the listed operating decisions.
- [ ] Review the backend language proposal; choose a module path and compatible toolchain when creating application code.
- [ ] Sketch the lifecycle and define the first vertical slice: create, assign, accept, repair, review and close.
- [ ] Specify core tables and hard invariants, including atomic team reservation and intake deduplication.
- [ ] Define the first API contracts and the durable PostgreSQL/Camunda coordination protocol.
- [ ] Define initial test data and acceptance checks for that slice.

Detailed dashboards, training materials and final handover evidence can be developed with their backlog stages. The first gate requires enough design to implement the core slice consistently, not completion of every release document.

## Go and Camunda feasibility check

At the beginning of B14/B15, use the selected Camunda version to deploy a minimal process, start it from Go, complete a human task, process one service job, and exercise a timeout with an uncertain start response. Record the API surface, auth, compatibility and recovery behavior before extending the adapter.

Use a database-backed durable queue/outbox for restart-sensitive work. In-memory goroutines are execution mechanisms, not persisted delivery guarantees.
