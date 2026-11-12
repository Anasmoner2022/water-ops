# Backend Language Proposal

Status: Proposed. This document does not lock the implementation language or claim a measured performance result.

## Context

The user is interested in Go for performance. The application requires HTTP APIs, PostgreSQL/PostGIS access, CSV validation, workflow orchestration, outbound integration delivery, authorization and troubleshooting. The role prioritizes reliable delivery and customer communication in addition to technical implementation.

## Recommendation

Use Go for the application backend if the developer is comfortable with its fundamentals or can learn them without delaying the delivery milestone. Start with a modular monolith and narrowly scoped adapters for PostgreSQL, Camunda and CRM. The API and background workers can share one codebase and domain rules.

Go offers useful concurrency mechanisms for HTTP requests and bounded workers. The official language guidance describes goroutines as lightweight execution units multiplexed across operating-system threads. This supports the implementation approach; it does not prove that this application will outperform a Python implementation.

## Performance reasoning

No Go-versus-Python application benchmark has been run. Performance depends on the workload, implementation and libraries. For this proposed architecture, database queries, spatial indexing, connection pools and network/service latency are likely to be important constraints. This is an architectural inference, to be checked with measurements.

Keep spatial selection and distance work in PostGIS, use suitable spatial indexes, paginate lists, and measure API latency plus query/service time. Use bounded worker concurrency and explicit timeouts/cancellation. Do not use concurrency to conceal slow queries or as a substitute for durable delivery state.

The primary reason to choose Go here is a maintainable typed API and worker implementation with useful concurrency support, together with the developer's interest in Go. Expected raw speed alone is insufficient evidence of business value.

## Integration boundary

Camunda exposes the Orchestration Cluster REST API for process instances, user tasks, variables and other orchestration operations. A Go HTTP adapter can target those endpoints. No assumption is made that a particular Go SDK is officially supported or feature-complete.

Before committing to an adapter, pin the Camunda version and verify deployment, start, task completion, job processing, authentication, timeout handling and reconciliation against that version. A transport/client choice must not change domain behavior or bypass Camunda orchestration.

## Tradeoffs

- Go requires explicit request validation, database mappings and HTTP error handling; these must be budgeted in the backlog.
- Python may reduce delivery time when the developer is already more proficient with its tooling or needs extensive exploratory data processing.
- The current scope needs deterministic rules and AI-assisted development; it does not require hosting a machine-learning runtime in the backend.
- A single application codebase is sufficient for the initial scope. Extra services should follow demonstrated operational needs.

## Acceptance of this proposal

Record an accepted decision after reviewing developer familiarity and the required integration path. Record dependency versions and compatibility results when the feasibility check is performed. Until then, do not create a Go module or install application dependencies merely to imply the choice is final.

## Primary references

- [Go concurrency and goroutines](https://go.dev/doc/effective_go#concurrency)
- [Go performance FAQ](https://go.dev/doc/faq#Why_does_Go_perform_badly_on_benchmark_X)
- [Camunda Orchestration Cluster REST API](https://docs.camunda.io/docs/apis-tools/orchestration-cluster-api-rest/orchestration-cluster-api-rest-overview/)
- [PostGIS spatial indexing](https://postgis.net/workshops/postgis-intro/indexing.html)
