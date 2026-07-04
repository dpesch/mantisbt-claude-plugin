# MantisBT Plugin for Claude Code

*[Deutsche Version](README.de.md)*

A [Claude Code](https://claude.com/product/claude-code) plugin that integrates the [MantisBT MCP Server](https://github.com/dpesch/mantisbt-mcp-server) into Claude Code, so you can research, create, and manage MantisBT bug tracker issues directly from a Claude Code session.

## What's included

- **MCP server** — launches `@dpesch/mantisbt-mcp-server` via `npx`, connecting to your MantisBT instance's REST API
- **Skills**
  - `/mantisbt:research` — research issues, bugs, and project history (read-only)
  - `/mantisbt:create` — guided creation of a new issue (bug, feature, task)
  - `/mantisbt:sync` — check or refresh the MantisBT metadata cache
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

## Usage

```
/mantisbt:research what's the status of issue 1234?
/mantisbt:create file a bug: login fails after password reset
/mantisbt:sync --info
```

You can also just describe what you need in natural language ("find open P1 bugs in project X", "create a feature request for dark mode") — the relevant skill is triggered automatically based on context.

## Versioning

The plugin version (`.claude-plugin/plugin.json`) is independent of the MantisBT MCP server version pinned in `.mcp.json`. See [DEVELOPMENT.md](DEVELOPMENT.md) for versioning rules and the release process.

## License

[MIT](LICENSE) © Dominik Pesch
