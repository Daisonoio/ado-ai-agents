# Creating a Wiki Agent Preset

A preset is a ready-made page schema for a specific project type. It defines
which wiki pages the agent generates, what each page contains, and how pages
relate to each other — without requiring the user to design the schema from scratch.

---

## When to create a preset

Create a preset when you have a recurring project structure that others would benefit
from. Good candidates: microservices, Python Django apps, Java Spring Boot services,
React frontends, data pipelines, infrastructure-as-code repos.

A preset should be opinionated. It reflects real decisions about what documentation
matters for that project type — not a generic placeholder.

---

## Preset file structure

A preset file is a markdown file placed in `presets/wiki-agent/`.
Filename: `<preset_id>.md` (lowercase, underscores, no spaces).

Required sections:

```markdown
# Preset: <preset_id>

**Target**: one sentence describing which project type this preset is for.
**Required modules**: list which C4 modules must be enabled for this preset to work.
**How to use**: copy the two tables below into C3.1 and C3.2 of your AGENTS.md.

---

## C3.1 Page Definitions

| Page ID | Title pattern | Description | Content hints | Owner | Conditional | Condition | Modules required |
|---|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... | ... |

---

## C3.2 Page Hierarchy

| Page ID | Parent page ID | Creation order |
|---|---|---|
| ... | ... | ... |

---

## Notes

Any decisions, trade-offs, or customization guidance specific to this preset.
```

---

## Writing good content hints

Content hints are the most important part of a preset. They tell the agent what
to search for and include in each page.

**Be specific.** Vague hints produce vague output.

| Too vague | Better |
|---|---|
| `technical details` | `public API endpoints, HTTP methods, request/response schemas` |
| `database stuff` | `database tables, column definitions, EF Core entity mappings` |
| `config` | `required environment variables, appsettings keys, IOptions bindings` |

**Use comma separation** for multiple hints in one page. Each hint becomes a
section in the output.

**Match hints to available modules.** If a hint requires EF Core scanning
(`database tables`, `entity mappings`) but `ef_core` is disabled, the agent
will write a gap notice instead of content. Document this in your preset's
Notes section.

---

## Owner values

Use `dev` for pages the agent should fully populate from source code and work items.
Use any other value (`qa`, `ops`, `platform`, etc.) for pages where another team
is responsible for the content. The agent will generate the structure with placeholders.

---

## Testing your preset

Before submitting a pull request:
1. Copy your preset into C3.1 and C3.2 of a test AGENTS.md.
2. Run the agent against a real or representative codebase.
3. Verify that every content hint produces a populated section or an explicit gap notice.
4. Verify that the page hierarchy is correct and all pages are created in the right order.

---

## Submitting a preset

Open a pull request to this repository with:
- Your preset file in `presets/wiki-agent/<preset_id>.md`
- A one-line addition to the preset index below

## Preset index

| Preset ID | Target | Required modules |
|---|---|---|
| `dotnet_enterprise` | .NET enterprise platform with EF Core, modular architecture, separate Dev/QA ownership | `ef_core`, `privileges`, `traceability`, `api_docs` |
| `microservice` | Single microservice, any stack. API docs, configuration, dependencies, operational runbook | `traceability`, `api_docs` |
| `feature_module` | Feature area inside a monolith. Overview, API surface, changelog | `traceability`, `api_docs` |
| _(your preset here)_ | | |