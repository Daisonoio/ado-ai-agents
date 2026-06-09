# Preset: microservice

**Target**: single microservice, any stack. Covers API documentation, runtime
configuration, dependencies, and operational runbook. No EF Core assumptions.

**Required modules**: `ef_core: false`, `privileges: false`, `traceability: true`, `api_docs: true`

**How to use**: copy the two tables below into C3.1 and C3.2 of your AGENTS.md.
Set C3.0 Mode to `preset`.

---

## C3.1 Page Definitions

| Page ID | Title pattern | Description | Content hints | Owner | Conditional | Condition | Modules required |
|---|---|---|---|---|---|---|---|
| `root` | `<PAGE_PREFIX> {name} <UNIT_LABEL_SINGULAR>` | Root page. Describes the purpose, responsibilities, and boundaries of this service. | overview, business purpose, technical responsibilities, upstream and downstream dependencies, known constraints | `dev` | No | — | — |
| `api` | `<PAGE_PREFIX> {name} API` | Full documentation of all public endpoints exposed by this service. | API endpoints, HTTP methods, routes, request models, response models, query parameters, error codes, work item references | `dev` | No | — | `api_docs` |
| `config` | `<PAGE_PREFIX> Configuration` | All runtime configuration required by this service. | required environment variables, appsettings keys, feature flags, external service URLs, timeout and retry settings | `dev` | No | — | — |
| `dependencies` | `<PAGE_PREFIX> Dependencies` | Inventory of all external dependencies consumed by this service. | external services, HTTP clients, message queues, databases, third-party SDKs, dependency purpose and failure behavior | `dev` | No | — | — |
| `runbook` | `<PAGE_PREFIX> Runbook` | Operational guide for on-call engineers and deployment teams. | deployment prerequisites, startup sequence, health check endpoints, known failure modes, rollback procedures, alerting thresholds | `ops` | No | — | — |

---

## C3.2 Page Hierarchy

| Page ID | Parent page ID | Creation order |
|---|---|---|
| `root` | — | 1 |
| `api` | `root` | 2 |
| `config` | `root` | 3 |
| `dependencies` | `root` | 4 |
| `runbook` | `root` | 5 |

---

## Notes

- `ef_core` and `privileges` are disabled. This preset makes no assumptions about
  the persistence layer. If your service uses EF Core, enable `ef_core: true` and
  add a `config` page row referencing database tables.
- The `runbook` page is owned by `ops` by default. If your team has no operational
  separation, change the owner to `dev`.
- The `dependencies` page requires no specific module. The agent infers dependencies
  from injected services, HTTP clients, and external references in source code.
- Work item references in the API page require `traceability: true`. If your project
  does not use ADO work items, set `traceability: false` — the agent will omit
  `(introduced in #ID)` references silently.
