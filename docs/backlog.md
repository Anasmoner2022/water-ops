# Water Ops Implementation Backlog

Status: planning baseline. All tasks are unstarted.

Execute in phase and row order for one developer. B01 to B24 form the first complete delivery milestone. Map, escalation, integration, analytics and handover tasks remain required for the full first release.

## Phase 1 Foundation

| ID | Task | Completion evidence |
| --- | --- | --- |
| B01 | Finalize contracts and operating decisions | Request/response fields, state permissions, error codes, SLA boundary and unresolved product decisions documented |
| B02 | Create application structure and development environment | Frontend, backend, workflow, test and documentation structure; Docker configuration and .env.example |
| B03 | Build database schema and migrations | Domain, audit, policy and integration tables with appropriate constraints and indexes |
| B04 | Create reference seeds | Repeatable zones, teams, representative accounts, team points and initial SLA policy |
| B05 | Implement login and sessions | Login/logout and account-disable behavior verified |
| B06 | Enforce API authorization | Role and team-data scope tested, including direct forbidden requests |
| B07 | Implement errors, logs and service checks | Consistent error envelope, correlation IDs, secret redaction and health checks |

## Phase 2 Data and intake

| ID | Task | Completion evidence |
| --- | --- | --- |
| B08 | Manage zones and teams | Valid GeoJSON, no invalid polygons or interior overlaps; team coverage/location/activation controls preserve open assignments |
| B09 | Implement asset CSV processing | Validation, all duplicate-code rows rejected, correct accepted rows and no silent overwrite |
| B10 | Build import UI and quality report | Upload, result counts and downloadable row-level error report |
| B11 | Configure versioned response SLAs | Positive integer durations, policy snapshots and unchanged existing deadlines |
| B12 | Implement incident creation | Spatial and field validation, incident/deadline/audit persistence and durable workflow-start intent |
| B13 | Build intake UI | Location selection, optional asset, boundary-zone choice and field errors |

## Phase 3 Camunda and domain commands

Depends on schema, authorization, reference data and incident intake.

| ID | Task | Completion evidence |
| --- | --- | --- |
| B14 | Build and deploy lifecycle BPMN | Assignment, acceptance, work, review, rework and closure linked to backend behavior |
| B15 | Implement process start and reconciliation | Pending preparation, failure recovery and uncertain-response handling without duplicate active processes |
| B16 | Implement eligible-team suggestions | Coverage, activity, specialty and reservation checks; spatial distance ranking in correct units |
| B17 | Implement assignment/reassignment | Atomic reservation and concurrency checks, historical records and original deadline retained |
| B18 | Implement technician commands | Authorized acceptance/start/repair transitions and required repair summary |
| B19 | Implement supervisor review | Approval/return, required reason, preserved submissions and correct reservation release |

Commands and workers must tolerate retries without repeating their business effects. The detailed protocol is a B01/B14/B15 design responsibility, not an assumed distributed transaction.

## Phase 4 First usable delivery

| ID | Task | Completion evidence |
| --- | --- | --- |
| B20 | Build operational list, details and dispatch UI | Search, filters, 20-row pages and valid role/state actions |
| B21 | Build technician task UI | Accept/start/resolve usable at 360 CSS pixels |
| B22 | Build supervisor review UI | Review queue, results and approve/reject controls |
| B23 | Build incident timeline | Actor/time/change/reason visible for all relevant events |
| B24 | Verify first complete lifecycle | Happy path, rejected repair and pre-work reassignment demonstrated |

## Phase 5 GIS and escalation

| ID | Task | Completion evidence |
| --- | --- | --- |
| B25 | Configure GeoServer layers | Zones/assets published from a limited read account with appropriate styling |
| B26 | Connect the operational map | Filters and selection synchronized; map outage preserves list/detail |
| B27 | Add response timer orchestration | Stored deadline, first acceptance, reassignment and recovery semantics verified |
| B28 | Add supervisor escalation | Persisted breach and one in-app notification per breach |
| B29 | Verify spatial/time rules | Outside/boundary/distance, changed policy, late acceptance and overdue recovery cases |

## Phase 6 CRM integration

Depends on working intake, closure and audit history. Resolve manual-incident notification behavior in B01.

| ID | Task | Completion evidence |
| --- | --- | --- |
| B30 | Build CRM simulator | Intake sender, closure receiver, configurable faults and stable-event deduplication |
| B31 | Implement inbound integration | Authentication, urgency mapping, validation and 201/200/409/422 behavior; concurrent duplicate protection |
| B32 | Implement durable closure delivery | Stable event ID and persisted delivery intent; closure independent of external availability |
| B33 | Implement retry and delivery operations | Three total automatic attempts with 10/30-second intervals, terminal failures, history and manual resend |
| B34 | Build Postman/integration verification | Auth, validation, mappings, repeated/conflicting intake, recovery and deduplication scenarios |

## Phase 7 Demonstration data and analytics

| ID | Task | Completion evidence |
| --- | --- | --- |
| B35 | Prepare complete synthetic dataset | 30 assets and 100 varied incidents with consistent state/history/reservations and documented reload |
| B36 | Write reference metric SQL | Five metrics, explicit denominators and creation-date/zone/priority filters |
| B37 | Configure Superset dashboard | Five metrics and filters backed by read-only datasets |
| B38 | Reconcile dashboard results | Known-answer fixtures match SQL; empty/undefined results handled correctly |

## Phase 8 Acceptance and handover

| ID | Task | Completion evidence |
| --- | --- | --- |
| B39 | Exercise service recovery | Camunda/GeoServer/CRM interruption and restart; data, states and retries verified |
| B40 | Prepare UAT and traceability | Fourteen baseline scenarios expanded into steps/data/expected results and linked criteria |
| B41 | Execute UAT | Actual results, evidence and defects recorded with role-specific accounts |
| B42 | Fix acceptance defects | Lifecycle-blocking and authorization defects resolved; affected checks rerun |
| B43 | Write operations/admin guide | Setup, ports, seeds, health, logs, recovery and limitations |
| B44 | Write guides and exercise training | Role-specific guidance tested by another person; unclear instructions corrected |
| B45 | Document validated AI assistance | Actual prompt/output/review/correction and verification evidence |
| B46 | Package handover and demonstration | README, screenshots, Postman, BPMN, UAT, guides and five-to-eight-minute English video |

## Story traceability

| Story | Primary tasks |
| --- | --- |
| US-01 | B05, B06, B20-B22 |
| US-02 | B08-B10 |
| US-03 | B11-B15 |
| US-04 | B20, B25, B26 |
| US-05 | B16, B17, B20 |
| US-06 | B18, B21 |
| US-07 | B19, B22 |
| US-08 | B11, B27-B29 |
| US-09 | B30, B31, B34 |
| US-10 | B30, B32-B34 |
| US-11 | B35-B38 |
| US-12 | B03, B07, B23, B33 |
| Operations and handover | B02, B15, B39-B46 |

## Definition of done

- The task's outcome can be demonstrated or reproduced.
- Relevant acceptance criteria are verified. Authorization, consistency, concurrency, timing and integration rules have meaningful tests.
- Affected configuration and documentation are updated.
- Changes are reviewed, including AI-generated suggestions.

Workflow: Backlog -> Ready -> In Progress -> Review -> Done. Record blocking reasons without treating a blocked task as finished.
