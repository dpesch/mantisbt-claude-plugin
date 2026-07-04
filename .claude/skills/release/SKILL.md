---
description: Prepare and cut a release of this plugin — validates the manifest, decides the version bump, updates plugin.json/CHANGELOG.md, commits, and tags. Use when the user asks to "release", "cut a release", "prepare a release", or "publish a new version" of this plugin.
---

Prepare a release of the `mantisbt` Claude Code plugin. Follow every step in order. **Never push to `origin` or push a tag without an explicit final confirmation from the user in this conversation** — a prior approval to run this skill is not approval to push.

## 1. Pre-flight checks

- Run `git status --porcelain`. If there are uncommitted changes unrelated to this release, stop and ask the user how to proceed (commit, stash, or abort) — do not silently include unrelated work in the release commit.
- Run `claude plugin validate . --strict`. If it fails, stop and fix the reported issues (or ask the user) before continuing — do not release a plugin that fails strict validation.

## 2. Decide the plugin version bump

The plugin version (`.claude-plugin/plugin.json`) is **independent** of the pinned MantisBT MCP server version in `.mcp.json` — bumping one does not require bumping the other. Only the actual content changes to skills, agents, or plugin configuration since the last tag determine the plugin's version bump.

- Run `git log <last-tag>..HEAD --oneline -- skills agents .claude-plugin .mcp.json` (find the last tag with `git describe --tags --abbrev=0`) to see what actually changed.
- Classify the change per the existing convention:
  - **Patch** — wording/description tweaks, typo fixes, no behavior change
  - **Minor** — a new skill or agent added, or existing ones gained meaningful new capability
  - **Major** — a skill was renamed/removed, a `userConfig` key was renamed, or behavior changed in a breaking way
- Present your classification and the resulting version number to the user via **AskUserQuestion** (options: accept the proposed bump, or pick a different bump level) — do not decide silently, this is a judgment call the user should confirm.

## 3. Decide whether to update the pinned MCP server version

Ask the user (AskUserQuestion) whether `.mcp.json`'s pinned `@dpesch/mantisbt-mcp-server@<version>` should be updated as part of this release:
- Check the current published version with `npm view @dpesch/mantisbt-mcp-server version`.
- If the user wants to update it, this is a separate fact from the plugin version bump decided in step 2 — it does not by itself justify a higher bump level unless the skills/agent content also needed changes to match new MCP capabilities.

## 4. Update files

- `.claude-plugin/plugin.json`: set `version` to the number agreed in step 2.
- `.mcp.json`: update the pinned MCP version if agreed in step 3.
- `CHANGELOG.md`: prepend a new `## [<version>] - <today's date>` section (Keep a Changelog style: `### Added` / `### Changed` / `### Fixed` / `### Removed` as applicable), summarizing the actual changes found in step 2's git log. Use today's date from the environment, not a guessed one.

## 5. Commit

Stage exactly the files touched in step 4 (plus any related changes the user approved in step 1) and create a commit following this repo's convention (see the global commit conventions — imperative verb, blank line, body explains *why*). Example subject: `Bereite Release v<version> vor` or an English equivalent if the repo's commit history is in English — check `git log --oneline -5` first and match the existing language.

## 6. Tag

Create an **annotated** git tag `v<version>` on the release commit: `git tag -a v<version> -m "v<version>"`. Do not create a lightweight tag — the CI release workflow and Codeberg release UI both work better with annotated tags.

## 7. Confirm before pushing

Show the user a summary: the new version, the commit, and the tag that was created locally. Then explicitly ask (plain question, not just proceeding) whether to push `main` and the tag to `origin` now. Only run `git push origin main` and `git push origin v<version>` after the user confirms in this turn. If they don't confirm, stop here — the commit and tag remain local and can be pushed later.

## 8. After pushing

If the tag was pushed, remind the user that the `.gitea/workflows/ci.yml` release job will create a Codeberg release automatically (provided the `GITEA_TOKEN` repository secret is configured) — no further manual action needed there. Mention that submitting/updating the listing at platform.claude.com/plugins/submit remains a manual step.
