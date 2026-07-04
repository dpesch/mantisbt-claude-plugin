---
description: Research MantisBT issues, bugs, and project history. Use this skill when the user asks about issues, bug reports, tickets, or needs context from the MantisBT bug tracker — for example "what's the status of issue 1234", "find open P1 bugs in project X", or "summarize recent activity on this ticket".
---

Use the MantisBT MCP tools to research the requested information:

- `get_issue` — fetch a single issue by ID
- `get_issues` — fetch multiple issues by ID in one batch call instead of looping single `get_issue` calls
- `list_issues` — list issues with filters (project, status, priority, assignee, dates, etc.)
- `search_issues` — full-text or semantic search across issues
- `list_notes` — read comments and notes on an issue
- `list_issue_files` — list attachments
- `get_project_versions` / `get_project_categories` — project metadata
- `get_current_user` — confirm authentication

## Prefer MCP resources over tool calls where available

Some data is exposed as MCP resources and can be read without a tool call: `mantis://me` (your profile), `mantis://projects` (cached project list), `mantis://projects/{id}` (a single project), `mantis://enums` (valid severity/priority/status/resolution/reproducibility values). Use these first if the server exposes them; fall back to the equivalent tool (`get_current_user`, `list_projects`, `get_issue_enums`) otherwise.

## Reduce response size with field projection

`get_issue`, `list_issues`, and `search_issues` accept a `select` parameter — a comma-separated list of fields to return (e.g. `"id,summary,status,priority,handler"`). Use it whenever you don't need the full issue payload; this matters most for large tickets with long histories. Discover valid field names with `get_issue_fields`. `get_issues` (batch-by-ID) does not support `select` and always returns full issues.

## Semantic search vs. a complete listing

`search_issues` returns the top-N most relevant matches — it is a relevance ranking, not a full scan, and depends on a search index that not every MantisBT instance has built. Keep two things in mind:

- Never treat `search_issues` results as a total count or frequency statistic — they are examples, not a census. For "how many", "most common", or "all issues matching X" questions, use `list_issues` with filters and pagination instead (increase `page` while the result count equals `page_size`).
- If `search_issues` fails because the index is empty or unavailable, fall back transparently to `list_issues` with a `select` projection and filter the results yourself — don't ask the user to configure anything.

## Links

Issue and note responses include a `view_url` field with the ready-to-use link — use it directly instead of constructing a URL from a base path and ID.

## Delegate large research to the `mantis-researcher` agent

For token-intensive work — fetching more than a handful of issues by ID, cross-project searches, full paginated listings, or `get_metadata_full()` — spawn the `mantis-researcher` agent instead of doing it inline. It keeps large API responses out of the main conversation and returns a structured summary.

Do not create, update, or delete any data unless the user explicitly asks. This skill is read-only by default.
