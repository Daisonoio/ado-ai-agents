# Path Chaining Strategy for Azure DevOps Wiki Agents

## The problem

The Azure DevOps Wiki REST API creates pages via:

```
PUT /pages?path={encodedPath}&api-version=7.0
```

`pagePath` must be the full path of the page to create. If any ancestor in that
path does not already exist as a wiki page, the API returns:

```
WikiAncestorPageNotFoundException: The ancestor page at path '/Docs/SectionName' does not exist.
```

When an LLM agent constructs this path from a page title, two things break:

**1. Write order is not enforced by the API.**
The API does not queue or defer writes. If an agent attempts to write a child page
before its parent exists — whether due to parallelism, incorrect sequencing, or
optimistic assumptions — the call fails immediately with no retry mechanism.

**2. The path value returned by the API is not always what you sent.**
Azure DevOps encodes, normalizes, and in some cases restructures paths on write.
A page titled "Payments Module" may be stored at `/TECHNICAL-Documents/Payments-Module`
(hyphenated, not space-separated). Constructing the child path from the original
title string produces a path that does not match the actual stored path, causing
`WikiAncestorPageNotFoundException` on the child write even when the parent exists.

---

## Why parentId does not solve it

The natural response to a path resolution error is to use a `parentId` parameter
instead of constructing a path string. This does not work.

`create_wiki_page` accepts only two parameters: `pagePath` and `content`.
It does not accept `parentId`. Passing it is silently ignored. The path-based
error persists regardless.

---

## The solution: path chaining from API responses

The correct approach uses path values returned by the API, never strings
constructed in memory.

**For existing parent pages:**

Call `get_wiki_structure` before any write operation. Locate the parent page
and extract its exact `path` field from the response:

```json
{
  "id": 672,
  "path": "/TECHNICAL-Documents",
  "name": "TECHNICAL-Documents"
}
```

Use `/TECHNICAL-Documents` as the base. Do not construct this from the page name.

**For newly created pages:**

After each successful `create_wiki_page` call, the response includes the
canonical path of the created page:

```json
{
  "path": "/TECHNICAL-Documents/Payments-Module",
  "id": 1043
}
```

Use `response.path` as the base for all child page paths. Do not use the
`pagePath` value you sent in the request — use what came back.

```
childPath = response.path + "/Configuration-Details"
```

**Write order:**

Pages must be created strictly parent-first. A child page write must never be
attempted before the parent write has completed successfully and its `path`
value has been stored.

---

## How this is encoded in AGENTS.md

The path chaining strategy is enforced in two places in the agent specification.

In the tools section, as a hard constraint on `create_wiki_page`:

> Never reconstruct wiki paths from string concatenation. Always use the `path`
> field from API responses. After each successful `create_wiki_page` call, log
> the returned `path` value and use it as the base for subsequent child paths.

In Step 2 of the workflow, as the path resolution procedure:

> Call `get_wiki_structure` and locate the entry for `<WIKI_DOCS_PATH>`.
> Extract its exact `path` value as returned by the API and store it as
> `PARENT_PATH`. Do not guess or reconstruct this path manually.

And in the page creation protocol (Step 8):

> Always use the `path` field from the API response of the preceding call to
> construct child paths. Never reconstruct paths manually from titles in memory.
> Never create a page and then update it in a second call.

This is classified as a hard constraint, not a formatting preference. Violation
produces incorrect wiki structure silently — the agent completes without errors
but child pages are created at wrong paths or not created at all.

---

## Failure modes without this strategy

| Scenario | Symptom | Root cause |
|---|---|---|
| Child written before parent | `WikiAncestorPageNotFoundException` | Write order not enforced |
| Path constructed from title string | `WikiAncestorPageNotFoundException` | Title encoding differs from stored path |
| Path chained from sent `pagePath` instead of response `path` | Pages created at wrong location | API normalizes path on write |
| `parentId` passed instead of constructing path | Error persists, `parentId` silently ignored | Parameter not supported |

---

## General principle

This pattern applies to any API that manages hierarchical resources and returns
canonical identifiers on write. The stored representation of a resource identifier
is not guaranteed to match the value you submitted. Always use identifiers returned
by the API to reference resources in subsequent calls — never reconstruct them from
display names, titles, or assumptions about encoding.
