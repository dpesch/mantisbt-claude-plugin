---
description: Check or refresh the MantisBT metadata cache (projects, categories, versions, users) and the search index status. Use when the user asks about cache freshness, wants to force a refresh after changes in MantisBT, or asks "is the mantis data up to date".
argument-hint:
  - "Optional: --info (show cache status without refreshing)"
  - "Optional: --force (force a refresh)"
---

Manage the MantisBT metadata cache maintained by the MCP server (projects, project members, categories, versions, and config values).

## No argument — show status

Call `mcp__mantisbt__get_metadata()` and present the summary it returns (project count, users per project, etc. — this call has returned a compact summary rather than a full dump since MCP server v1.8.0). If the response indicates the cache is empty or stale, ask the user whether to refresh it now.

## `--force` — refresh

1. If `rebuild_search_index` is available, call it to refresh the search index too.
2. Call `mcp__mantisbt__sync_metadata()` to force a cache refresh.
3. Call `get_metadata()` again and show the updated summary.

## `--info` — status only, no refresh

1. Call `get_metadata()` and show the summary (no refresh).
2. Call `mcp__mantisbt__get_search_index_status()` if available and report index size and last rebuild time.
3. If the server exposes MCP resources, mention them as no-tool-call shortcuts for common lookups: `mantis://me`, `mantis://projects`, `mantis://projects/{id}`, `mantis://enums`.

## Notes

- The MCP server manages its own TTL-based cache automatically; this skill is for on-demand visibility and manual refresh after MantisBT-side changes (new projects, categories, versions, or users).
- For the full raw metadata dump (all projects/users/categories at once, a large response), use `mcp__mantisbt__get_metadata_full()` via the `mantis-researcher` agent rather than inline.
- To keep the cache fresh automatically on a schedule, this skill can be invoked periodically (e.g. via a recurring task) with `--force`.
