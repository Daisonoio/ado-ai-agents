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

Generates structured module documentation by synthesizing three sources of truth:
source code, ADO work items, and existing wiki content. Produces up to five wiki
pages per module with consistent structure, progressive versioning, and a
human-in-the-loop confirmation step for structural extensions.

**Non-obvious design decisions:**

- **Source priority is explicit and enforced, not implicit.** Source code always
  overrides work items, which always override existing wiki content. The agent
  never uses wiki content as a factual source — only as a structural reference.
  This prevents documentation from drifting from reality across iterations.

- **Style guide is read at runtime, not hardcoded.** The agent fetches the team's
  style guide from the wiki on every run and treats it as the absolute formatting
  authority. Three rules are immutable regardless of what the guide says: destination
  path, page prefix, and the prohibition on modifying existing pages. Everything else
  defers to the live guide. This makes the agent portable across teams with different
  conventions without prompt changes.

- **Path resolution uses API response values, never string reconstruction.**
  `create_wiki_page` accepts only a `pagePath` string and does not support `parentId`.
  When any ancestor in the path does not exist as a wiki page, the API returns
  `WikiAncestorPageNotFoundException`. The agent resolves this by reading the exact
  `path` field from `get_wiki_structure` for existing parents, and chaining subsequent
  child paths from the `path` field in each `create_wiki_page` response — never from
  titles reconstructed in memory.
  → [Deep dive: path-chaining-strategy.md](./docs/path-chaining-strategy.md)

- **DB table inventory is exhaustive by definition.** The agent identifies tables
  belonging to a module using four independent signals: EF entity namespace, migration
  file name, repository/handler namespace, and shared schema prefix. It re-scans until
  no new tables are found. Completeness is a hard requirement, not a best-effort target.

- **Traceability index with confidence levels.** Work item references in documentation
  (`introduced in #ID`) are only written when a cross-reference reaches HIGH or MEDIUM
  confidence. Three signal types are used in order of reliability: inline code comments
  containing the work item ID (strongest), acceptance criteria match against method
  signatures and routes (HIGH/MEDIUM/LOW depending on directness), and sprint/date
  correlation (LOW, never used as sole justification).

- **Privilege detection uses four independent signals.** The Privileges subpage is
  created only if at least one of the following is found in module code: EF entity
  classes matching privilege naming patterns, seeding data inserting into privilege
  tables, string constants or enums matching authorization identifier patterns, or
  `[Authorize]` attribute references to privilege codes. All privilege codes in
  documentation come from seeding data or code constants — never invented.

- **Pages are created exactly once, fully populated.** The agent drafts complete
  content for all pages in memory before any write operation. No page is created
  empty and then updated. This prevents partial page states from appearing in the
  wiki if the agent is interrupted mid-run.

- **Progressive versioning instead of overwriting.** If pages for a module already
  exist, the agent appends a version suffix (V2, V3...) to all page titles rather
  than overwriting. The check runs per-page independently to handle partial previous
  runs correctly.

→ [View AGENTS.md](./agents/wiki-agent/AGENTS.md)

---

## Why This Exists

These prompts were built while solving real problems in production:

**Agents hallucinating wiki paths.** An LLM constructing a page path from a title
string produces paths that don't match what Azure DevOps actually stored. The API
returns `WikiAncestorPageNotFoundException` with no useful diagnostic. The fix
requires reading paths from API responses, not constructing them from memory.

**LLMs overwriting existing pages.** Without an explicit prohibition and a
versioning protocol, agents on second runs silently replace V1 pages. Progressive
versioning with a per-page existence check solves this without requiring the user
to remember which modules were already documented.

**Documentation drifting from source code.** When agents use existing wiki content
as a factual source, inaccuracies compound across iterations. Enforcing source code
as the primary source of truth, with wiki content treated as structural reference
only, keeps documentation accurate across refactors.

**Inconsistent structure across modules.** Hardcoding style rules in the prompt
means that when the team updates its conventions, all prompts must be updated
manually. Reading the style guide at runtime from a canonical wiki page makes
the agent self-updating.

**Privilege codes invented by the model.** Without explicit detection signals and
a hard rule linking all privilege codes to seeding data or code constants, LLMs
fill gaps with plausible-looking but fictional authorization codes. The four-signal
detection approach with a strict "no invention" constraint eliminates this.

---

## Stack

- **LLM runtime**: Claude (Anthropic) or Azure OpenAI GPT-4o via Semantic Kernel
- **Tool integration**: Azure DevOps MCP Server
- **Target environment**: Azure DevOps (cloud or on-premise TFS)
- **Language ecosystem**: .NET / C# (agents are LLM-runtime-agnostic)

---

## Structure

```
ado-ai-agents/
├── agents/
│   └── wiki-agent/
│       └── AGENTS.md
├── docs/
│   ├── path-chaining-strategy.md
│   └── defensive-mcp-patterns.md
└── README.md
```

---

## Usage

1. Clone the repo.
2. Open the relevant `AGENTS.md` in your agent runtime (Cursor, Claude, Copilot Studio, etc.).
3. Replace all `<PLACEHOLDER>` values in the **Configuration** section with your
   project-specific settings.
4. Ensure your MCP server exposes the tools listed in the **Tools Available** table
   inside the agent file.
5. Run the agent with a target module name as input.

---

## Author

Luca — Senior .NET Backend Developer, AI-integrated systems  
Insurance and financial services domain · Azure · Semantic Kernel · MCP  
[GitHub](https://github.com/Daisonoio) · [LinkedIn](https://www.linkedin.com/in/lucameli/)
