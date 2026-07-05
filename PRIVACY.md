# Privacy Policy — MantisBT Integration (Claude Code Plugin)

This plugin has no backend or infrastructure of its own. It configures Claude Code to launch the [MantisBT MCP Server](https://codeberg.org/dpesch/mantisbt-mcp-server) (`@dpesch/mantisbt-mcp-server`) locally, as a subprocess on your own machine, and connects it directly to the MantisBT instance you specify.

## What data this plugin handles

- **MantisBT URL and API key** — collected once via Claude Code's built-in `userConfig` prompt at enable time, and stored by Claude Code itself (not by this plugin or its author). See [Claude Code's own documentation](https://code.claude.com) for how it stores configuration values.
- **Issue data** (tickets, notes, attachments, project metadata) — read from and written to *your own* MantisBT instance, directly between your machine and that instance's REST API. It never passes through any server operated by this plugin's author.
- **Optional local semantic search** — if enabled (`MANTIS_SEARCH_ENABLED=true`), issue text is embedded using a locally downloaded, locally run model. This runs entirely offline; no issue content or query is sent to any external API or third party.

## What this plugin does not do

- No analytics, telemetry, or usage tracking.
- No data is collected, transmitted to, or stored by the plugin author.
- No third-party services are involved beyond the MantisBT instance you configure and, for semantic search, the one-time download of an open embedding model file.

## Claude Code itself

This document covers only the plugin's own behavior. Data handling by Claude Code as a product (conversation content, tool-call handling, etc.) is governed by Anthropic's own privacy policy, not this one.

## Contact

Questions about this plugin's data handling: [d.pesch@11com7.de](mailto:d.pesch@11com7.de).
