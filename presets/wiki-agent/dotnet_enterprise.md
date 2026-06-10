# Preset: dotnet_enterprise

**Target**: .NET enterprise platforms with EF Core, modular architecture,
ADO work items, and separate Dev/QA team ownership.

**Required modules**: `ef_core: true`, `privileges: true`, `traceability: true`, `api_docs: true`

**How to use**: copy the two tables below into C3.1 and C3.2 of your AGENTS.md.
Set C3.0 Mode to `preset`.

---

## C3.1 Page Definitions

| Page ID | Title pattern | Description | Content hints | Owner | Conditional | Condition | Modules required |
|---|---|---|---|---|---|---|---|
| `root` | `<PAGE_PREFIX> {name} <UNIT_LABEL_SINGULAR>` | Root page. Describes the purpose, responsibilities, and boundaries of this section. | overview, business purpose, technical responsibilities, key integrations, known constraints | `dev` | No | — | — |
| `config` | `<PAGE_PREFIX> Configuration Details` | Full inventory of all database setup tables owned by this section, with column-level documentation. | database tables, EF Core entity mappings, column definitions, standard audit columns, configuration areas | `dev` | No | — | `ef_core` |
| `privileges` | `<PAGE_PREFIX> Privileges` | Inventory of all privilege codes, authorization constants, and role-permission tables seeded by this section. | privileges, permissions, authorization constants, seeding data, role configuration tables | `dev` | Yes | `privilege_evidence_found` | `ef_core`, `privileges` |
| `api` | `<PAGE_PREFIX> {name} API` | Full documentation of all public REST endpoints exposed by this section. | API endpoints, HTTP methods, routes, request models, response models, query parameters, work item references, breaking changes | `dev` | No | — | `api_docs` |
| `setup` | `<PAGE_PREFIX> Setup` | Environment setup and deployment guide for this section. | deployment prerequisites, required environment variables, configuration keys, known setup issues, rollback procedures | `qa` | No | — | — |

---

## C3.2 Page Hierarchy

| Page ID | Parent page ID | Creation order |
|---|---|---|
| `root` | — | 1 |
| `config` | `root` | 2 |
| `privileges` | `config` | 3 |
| `api` | `root` | 4 |
| `setup` | `root` | 5 |

---

## Notes

- The `privileges` page is conditional: generated only if privilege evidence is
  found in source code (see S3 privilege evidence detection rules).
- The `setup` page is owned by `qa` by default. If your team has no QA separation,
  change the owner to `dev` — the agent will fully populate it from source.
- The `config` page uses EF Core scanning exclusively. If your project does not
  use EF Core, disable the `ef_core` module and remove or replace the `config` page.
- DB table names are always written without schema prefix (schema stripping rule):
  the schema qualifier (e.g. `dbo.`, `app.`) is removed from all table references
  in generated documentation. Only the bare table name is written.