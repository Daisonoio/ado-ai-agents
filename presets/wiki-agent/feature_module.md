# Preset: feature_module

**Target**: a self-contained feature area inside a monolith or modular application.
Covers functional overview, API surface, changelog, and known limitations.
Lightweight — no EF Core or privilege assumptions.

**Required modules**: `ef_core: false`, `privileges: false`, `traceability: true`, `api_docs: true`

**How to use**: copy the two tables below into C3.1 and C3.2 of your AGENTS.md.
Set C3.0 Mode to `preset`.

---

## C3.1 Page Definitions

| Page ID | Title pattern | Description | Content hints | Owner | Conditional | Condition | Modules re# Preset: feature_module

**Target**: a self-contained feature area inside a monolith or modular application.
Covers functional overview, API surface, changelog, and known limitations.
Lightweight — no EF Core or privilege assumptions.

**Required modules**: `ef_core: false`, `privileges: false`, `traceability: true`, `api_docs: true`

**How to use**: copy the two tables below into C3.1 and C3.2 of your AGENTS.md.
Set C3.0 Mode to `preset`.

---

## C3.1 Page Definitions

| Page ID | Title pattern | Description | Content hints | Owner | Conditional | Condition | Modules required |
|---|---|---|---|---|---|---|---|
| `root` | `<PAGE_PREFIX> {name} <UNIT_LABEL_SINGULAR>` | Root page. Describes the purpose, scope, and boundaries of this feature area. | overview, business purpose, technical responsibilities, entry points, known constraints, open issues | `dev` | No | — | — |
| `api` | `<PAGE_PREFIX> {name} API` | Public API surface exposed by this feature, including internal APIs consumed by other features. | API endpoints, HTTP methods, routes, request models, response models, internal interfaces, work item references | `dev` | No | — | `api_docs` |
| `changelog` | `<PAGE_PREFIX> {name} Changelog` | Chronological history of significant changes to this feature, grouped by sprint or release. | work items history, features added, behaviors changed, breaking changes, sprint grouping | `dev` | No | — | `traceability` |

---

## C3.2 Page Hierarchy

| Page ID | Parent page ID | Creation order |
|---|---|---|
| `root` | — | 1 |
| `api` | `root` | 2 |
| `changelog` | `root` | 3 |

---

## Notes

- This is the lightest preset. It produces three pages and has no module
  hard dependencies beyond `traceability` and `api_docs`.
- The `changelog` page requires `traceability: true`. If your project does not
  use ADO work items, disable `traceability` and remove the `changelog` row —
  the agent cannot build a reliable changelog without work item data.
- If your feature owns database tables, enable `ef_core: true` and add a
  `config` page row from the `dotnet_enterprise` preset.
- If your feature has privilege/authorization logic, enable both `ef_core: true`
  and `privileges: true` and add the `privileges` row from `dotnet_enterprise`.quired |
|---|---|---|---|---|---|---|---|
| `root` | `<PAGE_PREFIX> {name} <UNIT_LABEL_SINGULAR>` | Root page. Describes the purpose, scope, and boundaries of this feature area. | overview, business purpose, technical responsibilities, entry points, known constraints, open issues | `dev` | No | — | — |
| `api` | `<PAGE_PREFIX> {name} API` | Public API surface exposed by this feature, including internal APIs consumed by other features. | API endpoints, HTTP methods, routes, request models, response models, internal interfaces, work item references | `dev` | No | — | `api_docs` |
| `changelog` | `<PAGE_PREFIX> {name} Changelog` | Chronological history of significant changes to this feature, grouped by sprint or release. | work items history, features added, behaviors changed, breaking changes, sprint grouping | `dev` | No | — | `traceability` |

---
