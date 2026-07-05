---
description: Guide the user through enabling, checking, or troubleshooting the optional local semantic search (embedding-based natural-language search over issues). Use when the user wants to set up, enable, configure, or check semantic search, or asks things like "how do I turn on semantic search", "is semantic search active", or "rebuild the search index".
argument-hint:
  - "Optional: --status (check current state only, no setup guidance)"
---

Semantic search is a feature of the underlying `mantisbt-mcp-server`, not of this plugin — it is off by default and configured entirely through environment variables passed to the MCP server process, not through this plugin's `userConfig`. This skill cannot set those variables itself; it checks the current state and walks the user through the manual steps.

## 1. Check the current state first

Try `mcp__mantisbt__get_search_index_status()`.

- **Succeeds** → semantic search is already enabled. Report the indexed/total issue count and last sync time. Skip straight to section 3 (status & rebuild) unless the user asked for setup help specifically.
- **Fails / tool not available** → semantic search is disabled (or the server predates this feature). Continue with section 2.

If `--status` was passed, stop here after reporting — don't offer setup guidance unless the user asks.

## 2. Guide activation (only if disabled)

Explain that this requires setting environment variables for the process that launches Claude Code (shell profile, `.env` loaded before launch, or the launcher's environment — not this plugin's settings), then restarting Claude Code so the MCP server relaunches with the new environment:

```bash
export MANTIS_SEARCH_ENABLED=true
```

Ask (or infer from context, e.g. project size mentioned earlier in the conversation) which backend fits, using **AskUserQuestion** if genuinely unclear:

- **`vectra`** (default) — pure JS, no extra install, fine for up to roughly 10,000 issues
- **`sqlite-vec`** — faster for larger instances, requires `npm install sqlite-vec better-sqlite3` in addition to setting `MANTIS_SEARCH_BACKEND=sqlite-vec`

Mention the other optional variables only if the user needs to change a default: `MANTIS_SEARCH_DIR` (index location), `MANTIS_SEARCH_MODEL` (embedding model), `MANTIS_SEARCH_THREADS` (indexing threads, default `1`).

Note that the embedding model (~80 MB) downloads once on first use and everything then runs fully offline — no external API calls, no API key.

After the user confirms they've set the variable(s) and restarted, re-run `get_search_index_status()` to confirm it's now active, then continue with section 3.

## 3. Status & rebuild

Report what `get_search_index_status()` returned (indexed count vs. total, last sync). Issues are indexed incrementally on every server start, so the index is normally current without any action.

Only offer a full rebuild (`rebuild_search_index(full: true)`) if the user asks for one or the status looks clearly stale/inconsistent — confirm with **AskUserQuestion** first, since a full rebuild re-embeds every issue and can take a while on larger instances. A plain `rebuild_search_index()` (incremental) is safe to run without confirmation.

## Notes

- Never suggest editing this plugin's `userConfig` (MantisBT URL/API key) for this — semantic search config is unrelated and lives in the MCP server's own environment.
- If `sqlite-vec` was selected but `rebuild_search_index` or `search_issues` errors on a missing native module, remind the user to run `npm install sqlite-vec better-sqlite3` and restart.
