---
description: Prepare and cut a release of this plugin — validates the manifest, decides the version bump, updates plugin.json/CHANGELOG.md, commits, and tags. Use when the user asks to "release", "cut a release", "prepare a release", or "publish a new version" of this plugin.
---

Prepare a release of the `mantisbt` Claude Code plugin. Follow every step in order. **Never push to `origin` or push a tag without an explicit final confirmation from the user in this conversation** — a prior approval to run this skill is not approval to push.

## 1. Pre-flight checks

- Run `git status --porcelain`. If there are uncommitted changes unrelated to this release, stop and ask the user how to proceed (commit, stash, or abort) — do not silently include unrelated work in the release commit.
- Run `claude plugin validate . --strict`. If it fails, stop and fix the reported issues (or ask the user) before continuing — do not release a plugin that fails strict validation.

## 2. Decide the plugin version bump

The plugin version (`.claude-plugin/plugin.json`) is **independent** of the pinned MantisBT MCP server version in `.mcp.json` — bumping one does not require bumping the other. Only the actual content changes to skills, agents, or plugin configuration since the last **plain** tag (a tag without a `+ci.N` suffix — see below) determine the plugin's version bump.

- Find the last plain tag: `git tag -l 'v*' | grep -vE '\+ci\.[0-9]+$' | sort -V | tail -1`. Do **not** use `git describe --tags --abbrev=0` for this — it returns the most recent tag by commit order, which could be a `+ci.N` tag.
- Run `git log <last-plain-tag>..HEAD --oneline -- skills agents .claude-plugin .mcp.json` to see what actually changed in the shipped plugin content.

### Case A: plugin content changed

Classify the change per the existing convention:
  - **Patch** — wording/description tweaks, typo fixes, no behavior change
  - **Minor** — a new skill or agent added, or existing ones gained meaningful new capability
  - **Major** — a skill was renamed/removed, a `userConfig` key was renamed, or behavior changed in a breaking way

Present your classification and the resulting version number to the user via **AskUserQuestion** (options: accept the proposed bump, or pick a different bump level) — do not decide silently, this is a judgment call the user should confirm. The resulting tag is a **plain** `v<version>` with no suffix, and the `+ci.N` counter resets: the next infra-only release after this one starts again at `+ci.1`.

### Case B: only repo/CI/doc-tooling changed, or a purely cosmetic `.claude-plugin/plugin.json` field

This is not a plugin content release — it's infra/tooling housekeeping (CI workflow changes, `DEVELOPMENT.md`, etc.) that ships nothing new to plugin users, and also covers cosmetic-only edits to `plugin.json` that change no behavior — e.g. `displayName` wording — even though that file is normally on the content-path list. The dividing line is *does this change what the plugin does*, not *which file was touched*: renaming a skill, changing `userConfig`, or bumping the `.mcp.json` pin is always Case A; rewording `displayName`/`description`/`keywords` with no functional difference can be Case B at the user's discretion. Confirm with the user which it is — don't decide silently either way. Use this repo's versioning scheme instead of a normal SemVer bump:

- **Base version** stays exactly the last plain tag's version (e.g. `1.11.0`) — do not bump patch/minor/major.
- **Suffix**: SemVer build metadata, `+ci.<N>`, where `<N>` is one more than the highest existing `+ci.N` tag for that base version (count with `git tag -l 'v<base>+ci.*'`; if none exist yet, `N=1`).
- The resulting version string is e.g. `1.11.0+ci.1`, tagged as `v1.11.0+ci.1`.
- Per SemVer, build metadata is ignored for version precedence — `1.11.0+ci.1` and `1.11.0+ci.2` are equal-precedence to `1.11.0`, so this never looks like a plugin update to marketplace tooling. `claude plugin validate --strict` accepts this format (verified).
- Confirm the proposed version (base + suffix) with the user via **AskUserQuestion** before proceeding — same as case A, this is still a judgment call (e.g. the user may consider a given change "content" even if it only touched files outside the usual list).

## 3. Decide whether to update the pinned MCP server version

Skip this step entirely for a Case B (`+ci.N`) release — an `.mcp.json` change makes it plugin content by definition, i.e. Case A.

Ask the user (AskUserQuestion) whether `.mcp.json`'s pinned `@dpesch/mantisbt-mcp-server@<version>` should be updated as part of this release:
- Check the current published version with `npm view @dpesch/mantisbt-mcp-server version`.
- If the user wants to update it, this is a separate fact from the plugin version bump decided in step 2 — it does not by itself justify a higher bump level unless the skills/agent content also needed changes to match new MCP capabilities.

## 4. Update files

- `.claude-plugin/plugin.json`: set `version` to the full version string agreed in step 2, **including the `+ci.N` suffix for a Case B release** (e.g. `"1.11.0+ci.1"`).
- `.mcp.json`: update the pinned MCP version if agreed in step 3 (Case A only).
- `CHANGELOG.md`: prepend a new `## [<version>] - <today's date>` section — the `<version>` here **must be the exact string without the leading `v`** (e.g. `## [1.11.0+ci.1] - 2026-07-05`, not `## [1.11.0]`), because `.gitea/workflows/ci.yml` and `.github/workflows/release.yml` both extract release notes by literally matching `## [<tag-without-v>]`; a mismatch means an empty release body. Content: Keep a Changelog style (`### Added` / `### Changed` / `### Fixed` / `### Removed`), summarizing the actual changes found in step 2's git log — for Case B this is normally a short `### Changed` note describing the CI/tooling change. Use today's date from the environment, not a guessed one.

## 5. Commit

Stage exactly the files touched in step 4 (plus any related changes the user approved in step 1) and create a commit following this repo's convention (see the global commit conventions — imperative verb, blank line, body explains *why*). Example subject: `Bereite Release v<version> vor` or an English equivalent if the repo's commit history is in English — check `git log --oneline -5` first and match the existing language.

## 6. Tag

Create an **annotated** git tag `v<version>` on the release commit: `git tag -a v<version> -m "v<version>"`. Do not create a lightweight tag — the CI release workflow and Codeberg release UI both work better with annotated tags. `+` is a valid character in git tag names, so a Case B tag like `v1.11.0+ci.1` needs no escaping or quoting beyond normal shell quoting.

## 7. Confirm before pushing

Show the user a summary: the new version, the commit, and the tag that was created locally. Then explicitly ask (plain question, not just proceeding) whether to push `main` and the tag to `origin` now. Only run `git push origin main` and `git push origin v<version>` after the user confirms in this turn. If they don't confirm, stop here — the commit and tag remain local and can be pushed later.

## 8. After pushing

If the tag was pushed, remind the user that the `.gitea/workflows/ci.yml` `validate-and-release` job (triggered only by the `v*` tag push, not by the `main` push) will create a Codeberg release automatically (provided the `CODEBERG_RELEASE_TOKEN` repository secret is configured), and that Codeberg's push-mirror will propagate `main` and the tag to the GitHub mirror shortly after. Mention that submitting/updating the listing at platform.claude.com/plugins/submit remains a manual step.
