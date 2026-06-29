---
description: Research MantisBT issues, bugs, and project history. Use this skill when the user asks about issues, bug reports, tickets, or needs context from the MantisBT bug tracker — for example "what's the status of issue 1234", "find open P1 bugs in project X", or "summarize recent activity on this ticket".
---

Use the MantisBT MCP tools to research the requested information:

- `get_issue` — fetch a single issue by ID
- `get_issues` / `list_issues` — list issues with filters (project, status, priority, etc.)
- `search_issues` — full-text or semantic search across issues
- `list_notes` — read comments and notes on an issue
- `list_issue_files` — list attachments
- `get_project_versions` / `get_project_categories` — project metadata
- `get_current_user` — confirm authentication

Prefer `search_issues` for keyword queries. Use `get_issue` to fetch the full detail of specific tickets. Return a clear, structured summary of findings.

Do not create, update, or delete any data unless the user explicitly asks. This skill is read-only by default.
