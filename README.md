# ado-ai-agents

Production-grade AI agent system prompts for Azure DevOps automation.

Built and battle-tested on a real enterprise .NET platform processing insurance
and financial services workflows across multiple business lines. Every design
decision in these prompts exists because something broke without it — agents
hallucinating paths, overwriting existing pages, inventing privilege codes,
producing documentation that silently diverged from the actual codebase.

This is not a tutorial. These prompts ran in production.

---

## Agents

### Wiki Agent

Generates structured documentation for any section of your codebase by synthesizing
three sources of truth: source code, ADO work items, and existing wiki content.

The agent adapts to any project architecture through a declarative configuration
model: you define the page structure, the section boundaries, and which reasoning
capabilities to activate. The agent does the rest.

**Architecture:**

- **Preset system**: ready-made page schemas for common project types
  (`dotnet_enterprise`, `microservice`, `feature_module`). Copy a preset into
  the configuration block and the agent knows what pages to generate and what
  to put in each one. Contributors can add new presets via pull request.

- **Custom mode**: define your own page schema when no preset fits. Each page
  is described by a title pattern, a one-sentence description, and a list of
  content hints. The agent maps each hint to a search strategy and generates
  the corresponding section from code, work items, and wiki.

- **Autodiscovery**: the agent infers project sections from the repository folder
  structure at runtime. No static registry required. For ambiguous structures,
  the agent reports candidates and asks for confirmation before proceeding.

- **Configurable ownership**: each page has an owner field. Pages owned by `dev`
  are fully populated by the agent. Pages owned by any other value (`qa`, `ops`,
  `platform`) are generated as structured placeholders with an ownership header.
  Teams with no ownership separation set all pages to `dev`.

**Non-obvious design decisions:**

- **Source priority is explicit and enforced.** Source code always overrides work
  items, which always override existing wiki content. The agent never uses wiki
  content as a factual source — only as a structural reference. This prevents
  documentation from drifting from reality across iterations.

- **Style guide is read at runtime, not hardcoded.** The agent fetches the team's
  style guide from the wiki on every run and treats it as the absolute formatting
  authority. Three rules are immutable regardless of what the guide says: destination
  path, page prefix, and the prohibition on modifying existing pages. Everything else
  defers to the live guide.

- **Path resolution uses API response values, never string reconstruction.**
  `create_wiki_page` does not support `parentId`. When any ancestor in the path
  does not exist as a wiki page, the API returns `WikiAncestorPageNotFoundException`.
  The agent resolves this by reading the exact `path` field from `get_wiki_structure`
  for existing parents, and chaining child paths from the `path` field in each
  `create_wiki_page` response.
  → [path-chaining-strategy.md](./docs/path-chaining-strategy.md)

- **Traceability index with confidence levels.** Work item references in
  documentation (`introduced in #ID`) are written only when a cross-reference
  reaches HIGH or MEDIUM confidence. Three signal types in order of reliability:
  inline code comments with work item ID, acceptance criteria match against
  method signatures and routes, sprint/date correlation (LOW only, never sole
  justification).

- **Privilege detection uses four independent signals.** The Privileges page is
  generated only if at least one signal is found: EF entity classes matching
  privilege naming patterns, seeding data inserting into privilege tables, string
  constants or enums matching authorization identifier patterns, or `[Authorize]`
  attribute references. All privilege codes come from seeding data or code
  constants — never invented.

- **Pages are created exactly once, fully populated.** The agent drafts all page
  content in memory before any write operation. No page is created empty and then
  updated. This prevents partial page states from appearing in the wiki if the
  agent is interrupted mid-run.

- **Progressive versioning instead of overwriting.** If pages for a section
  already exist, the agent appends a version suffix (V2, V3...) rather than
  overwriting. The check runs per-page independently to handle partial previous
  runs correctly.

- **Bulk documentation is blocked by design.** The agent accepts one section per
  run. Documenting all sections simultaneously prevents granular review, gap
  correction, and clean error recovery.

→ [View AGENTS.md](./agents/wiki-agent/AGENTS.md)

---

## Why This Exists

**Agents hallucinating wiki paths.** An LLM constructing a page path from a title
string produces paths that don't match what Azure DevOps actually stored. The API
returns `WikiAncestorPageNotFoundException` with no useful diagnostic. The fix
requires reading paths from API responses, not constructing them from memory.

**LLMs overwriting existing pages.** Without an explicit prohibition and a
versioning protocol, agents on second runs silently replace existing pages.
Progressive versioning with a per-page existence check solves this without
requiring the user to track which sections were already documented.

**Documentation drifting from source code.** When agents use existing wiki content
as a factual source, inaccuracies compound across iterations. Enforcing source code
as the primary source of truth keeps documentation accurate across refactors.

**Inconsistent structure across sections.** Hardcoding style rules in the prompt
means that when the team updates its conventions, all prompts require manual updates.
Reading the style guide at runtime from a canonical wiki page makes the agent
self-updating without prompt changes.

**Privilege codes invented by the model.** Without explicit detection signals and
a hard rule linking all privilege codes to seeding data or code constants, LLMs
fill gaps with plausible-looking but fictional authorization codes.

**No portable documentation framework for ADO projects.** Most documentation
agents are either too generic (no ADO-specific knowledge) or too rigid (hardcoded
page structures). The preset system addresses this: opinionated defaults for common
project types, fully replaceable for teams with different conventions.

---

## Getting Started

### Prerequisites

- An Azure DevOps organization (cloud or on-premise TFS)
- An MCP server exposing the tools listed in the `tools:` section of `AGENTS.md`
- An LLM runtime that accepts system prompt files (Claude, Cursor, Copilot Studio,
  Semantic Kernel, or equivalent)
- A wiki style guide page in your ADO project (can be an empty page — the agent
  falls back to built-in rules if the guide is empty)

### Step 1 — Clone the repo

```bash
git clone https://github.com/Daisonoio/ado-ai-agents
cd ado-ai-agents
```

### Step 2 — Open the agent file

```
agents/wiki-agent/AGENTS.md
```

Everything is contained in this single file. The top half is the CONFIGURATION
block — the only part you edit. The bottom half is the AGENT SPECIFICATION —
do not modify it.

### Step 3 — Fill in C1: Technical Configuration

Replace the seven placeholders in the C1 table with your project values:

| Placeholder | Where to find it |
|---|---|
| `<ADO_SERVER>` | Your ADO organization URL or on-premise hostname |
| `<ADO_PROJECT>` | Project name as it appears in ADO URLs |
| `<WIKI_NAME>` | Wiki name from ADO Wiki settings |
| `<WIKI_DOCS_PATH>` | The wiki path where generated pages should be created |
| `<STYLE_GUIDE_PAGE_ID>` | Integer ID of your style guide page (visible in the ADO URL when viewing the page) |
| `<PAGE_PREFIX>` | A short prefix for all agent-created page titles, e.g. `[Draft]` |
| `<SOURCE_BRANCH>` | The branch to read source code from, e.g. `main` |

If you do not have a style guide, create an empty wiki page, note its ID, and
use that. The agent reads the page at runtime — if it is empty, built-in
formatting rules apply.

### Step 4 — Fill in C2: Architecture Profile

Set the architecture type and unit label that matches your project:

```
Architecture type:    modular_monolith
Unit label singular:  module
Unit label plural:    modules
Discovery signal:     folder_structure
```

Leave C2.2 empty. The agent will discover sections automatically from your
repository folder structure when you run it.

If autodiscovery produces incorrect results for your project (e.g. non-standard
folder layout), populate C2.2 manually with one row per documentable section.

### Step 5 — Choose and configure C3: Page Schema

**Option A — Use a preset (recommended for most teams)**

Browse `presets/wiki-agent/` and choose the preset that best matches your project:

| Preset | Best for |
|---|---|
| `dotnet_enterprise` | .NET with EF Core, modular architecture, separate Dev/QA teams |
| `microservice` | Any single microservice regardless of stack |
| `feature_module` | Feature areas inside a monolith |

Open the preset file, copy the two tables (C3.1 and C3.2), paste them into the
corresponding sections of your AGENTS.md, and set `Mode: preset` in C3.0.

Read the Notes section of the preset — it documents which modules to enable and
how to extend the schema for your specific case.

**Option B — Define a custom schema**

Set `Mode: custom` in C3.0 and fill in C3.1 and C3.2 directly.

Each row in C3.1 defines one wiki page: its title pattern, a one-sentence
description of its purpose, and a comma-separated list of content hints that
tell the agent what to search for and include.

See `presets/wiki-agent/creating-presets.md` for guidance on writing effective
content hints.

### Step 6 — Set C4: Active Modules

Enable or disable the four reasoning modules based on your stack:

| Module | Enable when... |
|---|---|
| `ef_core` | Your project uses EF Core and you want DB table documentation |
| `privileges` | Your project has a seeded privilege/permission system |
| `traceability` | Your project uses ADO work items and you want code-to-workitem cross-references |
| `api_docs` | Your project exposes REST endpoints you want documented |

If you are using a preset, its Notes section lists which modules are required.

### Step 7 — Load the agent into your runtime

Copy the full contents of `AGENTS.md` as the system prompt for your LLM runtime,
or load it as an agent definition file if your runtime supports it.

Ensure the MCP server is connected and the tools listed in the `tools:` section
of the file are available. The agent will call these tools during execution —
missing tools cause step failures reported in the final run report.

Supported runtimes: Claude (claude.ai or API), Cursor, GitHub Copilot Studio,
Semantic Kernel agent definitions, or any runtime that accepts a system prompt file.

### Step 8 — Run the agent

Type the name of the section you want to document:

```
document Payments
```

If C2.2 is empty (autodiscovery mode), the agent will first read the repository
structure, present the sections it found, and ask you to confirm which one to
document before proceeding.

The agent runs nine steps and produces a Documentation Report at the end listing
pages created, gaps found, and any errors with recovery actions.

### Step 9 — Review the output

Check the Documentation Report for:
- **Gaps**: content hints that could not be satisfied from any source. These
  require manual documentation.
- **Errors**: tool call failures with the step where they occurred and what
  the agent did to recover.
- **Outdated content**: wiki content that does not match the current source code.

Pages are created with a version suffix (V2, V3...) if pages for that section
already exist, so previous versions are never overwritten.

---

## Repository Structure

```
ado-ai-agents/
├── agents/
│   └── wiki-agent/
│       └── AGENTS.md                  ← agent engine, do not modify
├── presets/
│   └── wiki-agent/
│       ├── dotnet_enterprise.md       ← .NET enterprise preset
│       ├── microservice.md            ← microservice preset
│       ├── feature_module.md          ← feature module preset
│       └── creating-presets.md        ← guide for contributors
├── docs/
│   ├── path-chaining-strategy.md      ← technical deep dive on path resolution
└── README.md
```

---

## Contributing

Presets are the primary contribution surface. If you have a project type not
covered by the existing presets, adding one benefits everyone using the repo.

See `presets/wiki-agent/creating-presets.md` for the preset format, guidelines
on writing effective content hints, and instructions for submitting a pull request.

---

## Stack

- **LLM runtime**: Claude (Anthropic) or Azure OpenAI GPT-4o via Semantic Kernel
- **Tool integration**: Azure DevOps MCP Server
- **Target environment**: Azure DevOps (cloud or on-premise TFS)
- **Language ecosystem**: .NET / C# (agents are LLM-runtime-agnostic)

---

## Author

Luca — Senior .NET Backend Developer, AI-integrated systems  
Insurance and financial services domain · Azure · Semantic Kernel · MCP  
[GitHub](https://github.com/Daisonoio) · [LinkedIn](https://linkedin.com/in/yourprofile)
