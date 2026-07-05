# DEVELOPMENT.md – mantisbt-claude-plugin

## What is this project?

A **Claude Code plugin** that integrates the [MantisBT MCP Server](https://github.com/dpesch/mantisbt-mcp-server) into Claude Code. It bundles:

- An MCP server configuration that launches `@dpesch/mantisbt-mcp-server` via `npx`
- Four user-invokable skills: `/mantisbt:research`, `/mantisbt:create`, `/mantisbt:sync`, `/mantisbt:setup`
- A read-only sub-agent: `mantis-researcher`

The plugin prompts users for their MantisBT URL and API key at enable time (`userConfig`). No manual `settings.json` editing required.

---

## Tech stack

- Pure **Markdown + JSON** — no build step, no dependencies
- Plugin manifest: `.claude-plugin/plugin.json`
- MCP server config: `.mcp.json` (references npm package `@dpesch/mantisbt-mcp-server`)
- Skills: `skills/<name>/SKILL.md`
- Agents: `agents/<name>.md`

---

## Key commands

```bash
claude plugin validate .          # Validate plugin manifest and structure
claude plugin validate . --strict # Treat warnings as errors (use before release)
claude --plugin-dir .             # Test plugin locally in the current directory
```

---

## Versioning

The plugin version in `.claude-plugin/plugin.json` is independent of the MCP server version. Bump it when skills, agents, or the MCP configuration change meaningfully.

Follow [Semantic Versioning](https://semver.org/):
- **Patch** — wording improvements, typo fixes, description tweaks
- **Minor** — new skill or agent added
- **Major** — skill renamed/removed, `userConfig` keys renamed, breaking behavior change

---

## Publishing workflow

Use the local `/release` skill (`.claude/skills/release/SKILL.md`) to prepare a release — it validates the manifest, proposes a version bump, updates `CHANGELOG.md`, commits, and creates an annotated `vX.Y.Z` tag. It never pushes without explicit confirmation.

1. Run the `/release` skill and confirm the proposed version bump
2. Confirm pushing `main` + the tag to Codeberg (`origin`) when asked
3. CI (`.gitea/workflows/ci.yml`) runs a single `validate-and-release` job: on pull requests it only validates the manifest; on a `v*` tag push it validates and then creates a Codeberg release automatically (requires the `CODEBERG_RELEASE_TOKEN` repository secret to be configured in Codeberg's repo settings). Plain pushes to `main` no longer trigger CI — the `/release` skill's local `claude plugin validate . --strict` pre-flight check already covers that, so a second CI run on every commit would just double the runner queue wait without adding safety.
4. Codeberg push-mirrors `main` and tags to `github.com/dpesch/mantisbt-claude-plugin` automatically (configured under Codeberg repo Settings → Mirror Settings, syncs on push)
5. Submit or update the listing at [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit) — manual step, not automated; relies on the GitHub mirror from step 4 so future releases are picked up without re-submitting

---

## Relationship to mantisbt-mcp-server

This plugin **consumes** the MCP server as an npm package — it does not contain its source code. If the MCP server adds or renames tools, update the skill descriptions here to reflect the new capabilities.
