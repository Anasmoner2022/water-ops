# Water Ops Product Specification

Status: planning baseline, not an implementation report.

This project simulates delivery of a configurable incident-management solution to a fictional water utility. Its deliverables cover requirements, data preparation, configuration, integrations, GIS, testing, training, and handover.

## Scope

- One incident type: water leak.
- Three service zones, five maintenance teams, thirty synthetic assets, and one hundred demonstration incidents.
- Operator interface and a browser-based technician interface usable on a phone.
- Executable Camunda process, GeoServer map layers, CRM simulator, and Superset dashboard.
- Requirements traceability, UAT evidence, operating instructions, user guides, and a recorded English demonstration.

Deferred: native mobile applications, road-network routing, live GPS, inventory, billing, and reopening a closed incident.

## Roles

| Role | Permissions |
| --- | --- |
| Dispatcher | Create incidents, view the operational list and map, assign teams, and reassign before work starts |
| Technician | View the assigned team's work, accept tasks, start work, and submit repair results |
| Supervisor | Dispatcher permissions, repair review, closure approval, and escalation monitoring |
| Administrator | Manage users, teams, zones, assets and SLA policies; inspect integration failures |

Each team has one representative technician account in the first release. API authorization must enforce roles and data scope independently of the interface. Operational permissions are granted by the operational role; configuration administration does not itself grant closure approval.

## Incident lifecycle

| State | Meaning | Allowed next state |
| --- | --- | --- |
| NEW | Saved and not assigned | ASSIGNED |
| ASSIGNED | Team reserved | ACCEPTED, or reassignment |
| ACCEPTED | Team accepted | IN_PROGRESS, or reassignment |
| IN_PROGRESS | Repair started | RESOLVED |
| RESOLVED | Repair submitted for review | CLOSED or IN_PROGRESS |
| CLOSED | Supervisor approved | Terminal in this release |

Operating rules:

1. A team has at most one open assignment and remains reserved while repair review is pending.
2. Reassignment is available only before work starts. It releases the old team, reserves the new team, and preserves assignment history.
3. Rejected repair returns to the same team and requires a reason.
4. Asset association is optional. A selected asset must be active and in the incident's service zone.
5. Priority is immutable after creation in this release.
6. Camunda orchestrates process tasks and escalation. The backend validates commands, authorization, and operational persistence. Their consistency and recovery protocol must be designed before implementation.
7. Team coverage or activation changes must not invalidate an open assignment.

## Response SLA

| Priority | Time from successful creation to first team acceptance |
| --- | --- |
| HIGH | 30 minutes |
| MEDIUM | 60 minutes |
| LOW | 120 minutes |

An incident stores its SLA policy version and calculated response deadline at creation. Reassignment does not reset the deadline or erase first acceptance. Policy edits apply only to new incidents. Late acceptance is allowed and retains the breach history.

A separate demonstration policy may use one, two and three minutes. It must be identified as a demo policy. The exact equality rule at the deadline remains a B01 decision and must be used consistently in commands, timers, reports and tests.

## Data requirements

| Entity | Required information |
| --- | --- |
| Incident | ID, description, priority, source, point, zone, optional asset, state, current team, creation, first acceptance and closure times, policy version and deadline |
| Service zone | Unique code, name and valid polygon |
| Asset | Unique code, name, type, point, zone and active flag |
| Team | Unique code, name, coverage zones, specialty, recorded point and active flag |
| Assignment | Incident, team, assigning user, timestamps and reassignment history |
| SLA policy | Version, priority durations and activation time |
| Audit event | Incident, action, actor, timestamp, relevant changes and reason |
| Integration delivery | Stable event ID, direction, payload reference, attempts, response and last error |

Store coordinates using EPSG:4326 and timestamps in UTC. Display time using Africa/Cairo. Team distance is straight-line spatial distance in meters, not an estimated driving time.

## User stories and acceptance criteria

### US-01 Login and authorization

As a user, I want to access functions and data allowed by my role so that I can perform my responsibilities.

- AC-01.1: Valid credentials allow access to role-appropriate functions.
- AC-01.2: Disabled accounts cannot sign in or continue making authenticated API requests.
- AC-01.3: A technician cannot read or modify another team's incidents, including through direct API requests.
- AC-01.4: Missing or invalid authentication returns 401; an authenticated but forbidden action returns 403.

### US-02 Asset import and data quality

As an administrator, I want to import assets and inspect rejected data so that operational data is reliable.

- AC-02.1: CSV import accepts asset code, name, type, longitude, latitude and zone code.
- AC-02.2: Reject rows with missing required values, invalid coordinates, unknown zones or points outside their stated zone.
- AC-02.3: Existing asset codes are rejected without overwriting existing data. All rows sharing a duplicate code within the same input file are rejected.
- AC-02.4: Report accepted and rejected counts. The downloadable error report includes the input row number and rejection reason.
- AC-02.5: Service zones are loaded from valid GeoJSON; invalid polygons and overlapping polygon interiors are rejected.

### US-03 Create an incident

As a dispatcher, I want to record a leak with a location and priority so that response can begin.

- AC-03.1: Description length is 10 to 2000 characters; priority and location are mandatory.
- AC-03.2: Resolve the zone from the point. Reject points outside all service zones without creating an incident.
- AC-03.3: If the point matches multiple zone boundaries, require a choice from the matching zones.
- AC-03.4: An optional selected asset must be active and belong to the selected zone.
- AC-03.5: Successful persistence creates a unique ID, NEW state, response deadline and creation event.

### US-04 Operational list and map

As a dispatcher, I want to find and locate incidents so that I can prioritize work.

- AC-04.1: Display ID, priority, state, zone, team, and remaining or overdue response time.
- AC-04.2: Support ID search and filters for state, priority, zone and creation date, with 20-row pages.
- AC-04.3: Render GeoServer zones and assets, plus incident points matching the current filters.
- AC-04.4: Selecting an incident in either view opens the same incident detail.
- AC-04.5: Map-service failure shows a useful message while list and detail remain usable.

### US-05 Team suggestion and assignment

As a dispatcher, I want suitable available teams to be ranked so that I can allocate a resource.

- AC-05.1: Eligible teams are active, cover the zone, have the water-maintenance specialty and have no open assignment.
- AC-05.2: Rank eligible teams by straight-line distance from their recorded points; show meters or kilometers.
- AC-05.3: With no eligible team, retain NEW state and explain why assignment is unavailable.
- AC-05.4: Successful assignment reserves the team and changes the incident to ASSIGNED.
- AC-05.5: Two simultaneous attempts to reserve one team for different incidents result in one successful reservation only.
- AC-05.6: Reassignment before work starts releases the former team, reserves the new one, preserves history and retains the original response deadline.

### US-06 Technician execution

As a technician, I want to accept and update my team's task so that operations can follow progress.

- AC-06.1: The assigned incident appears in the team's task view.
- AC-06.2: Acceptance changes ASSIGNED to ACCEPTED and records acceptance time.
- AC-06.3: Starting changes ACCEPTED to IN_PROGRESS. Acceptance cannot be skipped.
- AC-06.4: Repair submission requires a work summary and changes IN_PROGRESS to RESOLVED.
- AC-06.5: Technicians cannot change priority or team, or approve closure.

### US-07 Supervisor repair review

As a supervisor, I want to review repair results so that closure reflects accepted work.

- AC-07.1: List pending reviews with repair summaries and incident history.
- AC-07.2: Approval changes RESOLVED to CLOSED, records reviewer and time, and releases the team.
- AC-07.3: Rejection requires a reason and returns the incident to IN_PROGRESS with the same reserved team.
- AC-07.4: Preserve prior repair submissions and review decisions.
- AC-07.5: Closed incidents reject further operational changes in this release.

### US-08 SLA configuration and escalation

As a supervisor, I want overdue response to be escalated so that I can intervene.

- AC-08.1: Administrators can set a positive integer duration in minutes for each priority without code changes.
- AC-08.2: Existing incidents retain their policy version and deadline; new incidents use the new policy.
- AC-08.3: Expiry before first acceptance creates a supervisor escalation and an overdue indicator.
- AC-08.4: Retried escalation processing does not create duplicate notifications for the same breach.
- AC-08.5: Late acceptance permits work to continue while retaining breach history.

### US-09 CRM incident intake

As an integration administrator, I want to receive CRM incidents so that the customer can retain their intake channel.

- AC-09.1: Require valid authentication, an external reference, description, location and urgency.
- AC-09.2: Map P1, P2 and P3 to HIGH, MEDIUM and LOW respectively.
- AC-09.3: Apply the manual intake validation rules; business-validation failures return 422 with field details.
- AC-09.4: First successful creation returns 201 and the internal incident ID.
- AC-09.5: Repeating source and external reference with the same normalized input returns 200 and the existing incident without creating another incident or workflow.
- AC-09.6: Reusing the reference with different business input returns 409 without modifying the existing incident. Normalization rules must be defined in the integration contract.

### US-10 CRM closure notification

As an integration administrator, I want to deliver closure outcomes and investigate failed delivery.

- AC-10.1: Prepare an incident.closed event with a stable event ID, internal and external references, closure time and repair summary. Define handling of incidents without an external reference before implementation.
- AC-10.2: A 2xx response marks delivery successful.
- AC-10.3: Timeout or 5xx triggers at most three total attempts, with 10 seconds before attempt two and 30 seconds before attempt three.
- AC-10.4: A 4xx response is terminal for automatic retry in this release and is visible for review.
- AC-10.5: Manual resend retains the same event ID. The CRM simulator deduplicates received event IDs.
- AC-10.6: Delivery failure does not reopen or roll back a closed incident; integration status is separate.

### US-11 Operational dashboard

As a supervisor, I want SQL-based operational metrics so that I can understand workload and service performance.

- AC-11.1: Show open incidents, overdue incidents not yet accepted, average response time, average closure time and response compliance.
- AC-11.2: Response time is first acceptance minus creation. Average response includes accepted incidents only.
- AC-11.3: Response compliance is accepted incidents meeting the deadline divided by all accepted incidents; label this denominator explicitly.
- AC-11.4: Average closure time includes closed incidents only and uses approved closure minus creation.
- AC-11.5: Creation-date, zone and priority filters match reference SQL results.
- AC-11.6: Undefined averages or ratios display no data rather than a misleading zero or calculation error.

### US-12 Audit and troubleshooting

As a support administrator, I want correlated history so that I can investigate and escalate a problem with evidence.

- AC-12.1: Audit state transitions, assignments, reviews and escalations with actor and timestamp.
- AC-12.2: Capture reasons and relevant prior/new values for reassignment and repair rejection.
- AC-12.3: Every API request has a correlation_id returned to the caller and attached to related logs.
- AC-12.4: Show integration attempt times, response status, outcome and last error.
- AC-12.5: Application interfaces cannot edit audit history. Logs exclude passwords and authentication tokens.

## Cross-cutting operational acceptance

- If Camunda is unavailable at creation, preserve the incident and display that workflow preparation is pending. Assignment remains unavailable until preparation succeeds.
- Recover preparation after service restoration without creating duplicate incidents or multiple active processes. Handle uncertain start outcomes through reconciliation, not blind retries.
- Workflow delay does not reset the response deadline; an overdue unaccepted incident must be escalated after recovery.
- Failed state commands must not be shown as successful or persist contradictory final states.
- User-facing failures explain what happened and the available next action; technical detail goes to correlated logs.
- Document local service startup, configuration and synthetic data loading.
- Technician acceptance, start and repair submission work at 360 CSS pixels without horizontal scrolling.

## UAT scenarios

| ID | Scenario | Expected result | Coverage |
| --- | --- | --- | --- |
| UAT-01 | Create, assign, accept, repair and close | Valid lifecycle, complete history and team release | US-03, US-05, US-06, US-07 |
| UAT-02 | Technician accesses another team's incident | Forbidden, with no mutation | US-01 |
| UAT-03 | Create outside service zones | Rejected, no incident created | US-03 |
| UAT-04 | Import valid, duplicate and invalid-coordinate rows | Correct accepted set and detailed rejection report | US-02 |
| UAT-05 | Concurrent reservation of one team | Only one reservation succeeds | US-05 |
| UAT-06 | Unaccepted incident expires | One escalation and overdue indicator | US-08 |
| UAT-07 | Change SLA after creation | Existing deadline retained; new policy used for new incidents | US-08 |
| UAT-08 | Repeat CRM intake, then change its business payload | Existing result returned, then conflict | US-09 |
| UAT-09 | CRM fails during closure and recovers | Closure retained and event delivery deduplicated | US-10 |
| UAT-10 | Reject repair, correct and approve | Return to work and both submissions retained | US-06, US-07 |
| UAT-11 | Compare dashboard against known data | Counts, durations and ratios match reference SQL | US-11 |
| UAT-12 | Camunda unavailable at creation, then restored | Incident preserved and process recovered without duplication | Operational acceptance |
| UAT-13 | GeoServer unavailable | List/detail usable with explanatory map failure | US-04 |
| UAT-14 | Investigate a failed API delivery | Request, incident and attempts located by correlation | US-12 |

Record steps, test data, expected and actual result, pass/fail, evidence and linked defect for each executed test. These scenarios are a baseline and do not replace all individual acceptance criteria.

## Release completion

- Complete executable lifecycle, authorization, data checks, maps, metrics and CRM integration.
- Passing UAT evidence with no unresolved lifecycle-blocking or authorization defects.
- Reproducible synthetic data, Postman collection, user/admin instructions, recovery guidance and known limitations.
- Simulated training exercise using the guides; label simulated stakeholder work honestly.
- One reviewed AI-assisted SQL or transformation example, including the validation evidence.
- An English demonstration lasting five to eight minutes and a trace from stories to acceptance criteria, tests and results.

## Decisions still required before code

- Deadline equality and timer/acceptance race semantics.
- API command idempotency and normalized CRM intake comparison.
- Camunda/PostgreSQL authority boundaries, durable intents and reconciliation.
- Closure delivery policy for manually entered incidents without a CRM reference.
- Session implementation, role/data-scope matrix and account-disable behavior.
- Specific service versions, compatibility and local resource requirements.
