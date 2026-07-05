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

The plugin's base SemVer version **always mirrors the pinned MantisBT MCP server version** in `.mcp.json` — there is no independent Patch/Minor/Major decision for the plugin itself. The only thing that determines the version is: *does this release change the MCP pin?*

- **The MCP pin changes to `X.Y.Z`** → the release is the plain tag `vX.Y.Z`. This holds regardless of what else shipped in the same release — new skills, doc fixes, even a breaking change to the plugin itself. There's no separate Major/Minor path; the MCP version *is* the plugin version.
- **The MCP pin does not change** → *any* other change (new/removed/renamed skill or agent, `userConfig` change, doc/CI housekeeping, cosmetic wording, anything) is a **build-metadata release**: `<current-pin>+ci.<N>`, e.g. `1.11.0+ci.3`. `<N>` increments per such release since the last time the MCP pin changed, and resets to `1` the next time the pin changes.

Per the SemVer spec, build metadata is ignored for version precedence, so `1.11.0+ci.1` and `1.11.0+ci.3` are precedence-equal to `1.11.0` — a `+ci.N` release never registers as a plugin update to marketplace/update tooling that compares versions correctly. `claude plugin validate --strict` accepts this format.

The git tag is `v<version>` (`+` is a valid character in git tag names). The `CHANGELOG.md` heading must match the tag exactly, minus the leading `v` (e.g. `## [1.11.0+ci.3]`) — both CI workflows extract release notes by matching that heading literally, with `.`/`+` escaped for the `awk` regex (see `.gitea/workflows/ci.yml` / `.github/workflows/release.yml`).

The `/release` skill (`.claude/skills/release/SKILL.md`) automates this — it asks whether the MCP pin should be updated as part of the release and derives the version from that single decision.

---

## Publishing workflow

Use the local `/release` skill (`.claude/skills/release/SKILL.md`) to prepare a release — it validates the manifest, derives the version from the MCP-pin decision (see Versioning above), updates `CHANGELOG.md`, commits, and creates an annotated `vX.Y.Z[+ci.N]` tag. It never pushes without explicit confirmation.

1. Run the `/release` skill and confirm the proposed version
2. Confirm pushing `main` + the tag to Codeberg (`origin`) when asked
3. CI (`.gitea/workflows/ci.yml`) runs a single `validate-and-release` job on `codeberg-tiny`: on pull requests it only validates the manifest; on a `v*` tag push it validates and then creates a Codeberg release automatically (requires the `CODEBERG_RELEASE_TOKEN` repository secret to be configured in Codeberg's repo settings, and the repo's **Releases** unit to be enabled — Settings → Units). Plain pushes to `main` no longer trigger CI — the `/release` skill's local `claude plugin validate . --strict` pre-flight check already covers that.
4. Codeberg push-mirrors `main` and tags to `github.com/dpesch/mantisbt-claude-plugin` automatically (configured under Codeberg repo Settings → Mirror Settings → Push Mirror, syncs on push; the GitHub-side classic PAT used for this needs the `public_repo` scope **and** `workflow` scope, since `.github/workflows/release.yml` is itself part of what gets pushed)
5. Once the tag lands on the GitHub mirror, `.github/workflows/release.yml` triggers there independently and creates the matching GitHub release from the same `CHANGELOG.md` section — no extra secret needed, it uses the built-in `GITHUB_TOKEN`
6. Submitting the plugin listing is a **one-time manual step**, already done for this plugin — the exact URL depends on account type: Team/Enterprise accounts use `claude.ai/admin-settings/directory/submissions/plugins/new` (needs Organization Owner or delegated directory-management access), while individual Console-only authors use `platform.claude.com/plugins/submit`. Either way it relies on the GitHub mirror from step 4, so future releases are picked up automatically without re-submitting.

Note: a git tag only triggers the GitHub workflow if the workflow file already existed in the repo *before* that tag was created (tags are immutable snapshots) — this bit `v1.11.0` and `v1.11.0+ci.1` the first time the mirror was set up (both tags arrived on GitHub in the same initial sync that introduced the workflow file, so GitHub never fired it for them retroactively; both releases were backfilled once via `gh release create`). Any tag created after the workflow file already existed on GitHub triggers it normally.

---

## Relationship to mantisbt-mcp-server

This plugin **consumes** the MCP server as an npm package — it does not contain its source code. If the MCP server adds or renames tools, update the skill descriptions here to reflect the new capabilities.
