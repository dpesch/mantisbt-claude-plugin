# MantisBT Plugin for Claude Code

*[Deutsche Version](README.de.md)*

A [Claude Code](https://claude.com/product/claude-code) plugin that integrates the [MantisBT MCP Server](https://github.com/dpesch/mantisbt-mcp-server) into Claude Code, so you can research, create, and manage MantisBT bug tracker issues directly from a Claude Code session.

## What's included

- **MCP server** — launches `@dpesch/mantisbt-mcp-server` via `npx`, connecting to your MantisBT instance's REST API
- **Skills**
  - `/mantisbt:research` — research issues, bugs, and project history (read-only)
  - `/mantisbt:create` — guided creation of a new issue (bug, feature, task)
  - `/mantisbt:sync` — check or refresh the MantisBT metadata cache
  - `/mantisbt:setup` — guided setup, status check, and troubleshooting for the optional semantic search
- **Agent**
  - `mantis-researcher` — a read-only sub-agent for token-intensive research (bulk lookups, cross-project searches, large result sets), isolated from the main conversation

## Requirements

- A MantisBT instance with the REST API enabled
- A MantisBT API token (**MantisBT → Profile → API Tokens**)
- Node.js / `npx` available on the machine running Claude Code (used to launch the MCP server)

## Installation

Install the plugin in Claude Code, then enable it:

```
/plugin install <marketplace-or-repo-reference>
```

On first use, Claude Code will prompt you for two settings (`userConfig`):

| Setting | Description |
|---|---|
| **MantisBT API URL** | Base URL of your MantisBT REST API, e.g. `https://bugs.example.com/api/rest` |
| **API Key** | Your MantisBT API token (stored securely, never shown in plain text) |

## Optional: local semantic search

The underlying MCP server can optionally build a local, offline semantic search index over your MantisBT issues, so `/mantisbt:research` can answer natural-language queries (e.g. *"login fails after password reset"*) instead of relying on exact keyword matches. The embedding model (~80 MB) is downloaded once on first use and runs entirely offline — no external API calls.

It's disabled by default. To enable it, set these environment variables for the process launching Claude Code (they are not part of this plugin's `userConfig`, since they configure the MCP server itself):

| Variable | Default | Purpose |
|---|---|---|
| `MANTIS_SEARCH_ENABLED` | `false` | Set to `true` to enable semantic search |
| `MANTIS_SEARCH_BACKEND` | `vectra` | `vectra` (pure JS, no extra install) or `sqlite-vec` (faster for 10,000+ issues, requires `npm install sqlite-vec better-sqlite3`) |
| `MANTIS_SEARCH_DIR` | `{MANTIS_CACHE_DIR}/search` | Directory for the search index |
| `MANTIS_SEARCH_MODEL` | `Xenova/paraphrase-multilingual-MiniLM-L12-v2` | Embedding model name |
| `MANTIS_SEARCH_THREADS` | `1` | ONNX threads used when indexing |

See the [MantisBT MCP Server README](https://codeberg.org/dpesch/mantisbt-mcp-server) for details, or just run `/mantisbt:setup` — it checks whether semantic search is active and walks you through enabling it if not.

## Usage

```
/mantisbt:research what's the status of issue 1234?
/mantisbt:create file a bug: login fails after password reset
/mantisbt:sync --info
/mantisbt:setup
```

You can also just describe what you need in natural language ("find open P1 bugs in project X", "create a feature request for dark mode") — the relevant skill is triggered automatically based on context.

## Versioning

The plugin version (`.claude-plugin/plugin.json`) is independent of the MantisBT MCP server version pinned in `.mcp.json`. See [DEVELOPMENT.md](DEVELOPMENT.md) for versioning rules and the release process.

## License

[MIT](LICENSE) © Dominik Pesch
