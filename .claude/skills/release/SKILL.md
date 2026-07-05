---
description: Prepare and cut a release of this plugin — validates the manifest, decides the version, updates plugin.json/CHANGELOG.md, commits, and tags. Use when the user asks to "release", "cut a release", "prepare a release", or "publish a new version" of this plugin.
---

Prepare a release of the `mantisbt` Claude Code plugin. Follow every step in order. **Never push to `origin` or push a tag without an explicit final confirmation from the user in this conversation** — a prior approval to run this skill is not approval to push.

## 1. Pre-flight checks

- Run `git status --porcelain`. If there are uncommitted changes unrelated to this release, stop and ask the user how to proceed (commit, stash, or abort) — do not silently include unrelated work in the release commit.
- Run `claude plugin validate . --strict`. If it fails, stop and fix the reported issues (or ask the user) before continuing — do not release a plugin that fails strict validation.

## 2. Decide the version

The plugin's base SemVer version **always mirrors the pinned MantisBT MCP server version** in `.mcp.json`. There is no independent Patch/Minor/Major decision for the plugin itself — the only question that determines the version is: *does this release change the MCP pin?*

- Check the currently pinned version in `.mcp.json` and the latest published version with `npm view @dpesch/mantisbt-mcp-server version`.
- Ask the user (**AskUserQuestion**) whether to update the pin as part of this release.

### If the MCP pin changes (to version `X.Y.Z`)

- The release's version is the **plain** tag `vX.Y.Z` — no suffix, regardless of what else changed in this release (new skills, doc fixes, breaking changes, anything). This holds even for breaking changes to the plugin itself; there is no separate Major/Minor path.
- The `+ci.N` counter resets: the next release after this one that does *not* also change the MCP pin starts again at `+ci.1`.

### If the MCP pin does not change

- **Base version** stays exactly the current pin's version (e.g. `1.11.0`) — this applies to *any* other change in this release: new/removed/renamed skills or agents, `userConfig` changes, doc/CI/tooling housekeeping, cosmetic wording — all of it, without exception.
- **Suffix**: SemVer build metadata, `+ci.<N>`, where `<N>` is one more than the highest existing `+ci.N` tag for that base version (count with `git tag -l 'v<base>+ci.*'`; if none exist yet, `N=1`).
- The resulting version string is e.g. `1.11.0+ci.3`, tagged as `v1.11.0+ci.3`.
- Per SemVer, build metadata is ignored for version precedence — `1.11.0+ci.1` and `1.11.0+ci.3` are equal-precedence to `1.11.0`, so this never looks like a plugin update to marketplace tooling. `claude plugin validate --strict` accepts this format (verified).

Present the resulting version to the user via **AskUserQuestion** before proceeding — confirm, don't decide silently.

## 3. Update files

- `.claude-plugin/plugin.json`: set `version` to the full version string agreed in step 2 (including the `+ci.N` suffix when applicable, e.g. `"1.11.0+ci.3"`).
- `.mcp.json`: update the pinned MCP version if that's what triggered this release.
- `CHANGELOG.md`: prepend a new `## [<version>] - <today's date>` section — the `<version>` here **must be the exact string without the leading `v`** (e.g. `## [1.11.0+ci.3] - 2026-07-05`, not `## [1.11.0]`), because `.gitea/workflows/ci.yml` and `.github/workflows/release.yml` both extract release notes by literally matching `## [<tag-without-v>]`; a mismatch means an empty release body. Content: Keep a Changelog style (`### Added` / `### Changed` / `### Fixed` / `### Removed`), summarizing what actually changed since the last tag (`git log <last-tag>..HEAD --oneline`). For a `+ci.N` release, briefly note it's a non-MCP-triggered release so the CHANGELOG stays self-explanatory (a one-line note above the bullets is enough, no fixed wording required).

## 4. Commit

Stage exactly the files touched in step 3 (plus any related changes the user approved in step 1) and create a commit following this repo's convention (see the global commit conventions — imperative verb, blank line, body explains *why*). Example subject: `Bereite Release v<version> vor` or an English equivalent if the repo's commit history is in English — check `git log --oneline -5` first and match the existing language.

## 5. Tag

Create an **annotated** git tag `v<version>` on the release commit: `git tag -a v<version> -m "v<version>"`. Do not create a lightweight tag — the CI release workflow and Codeberg release UI both work better with annotated tags. `+` is a valid character in git tag names, so a `+ci.N` tag like `v1.11.0+ci.3` needs no escaping or quoting beyond normal shell quoting.

## 6. Confirm before pushing

Show the user a summary: the new version, the commit, and the tag that was created locally. Then explicitly ask (plain question, not just proceeding) whether to push `main` and the tag to `origin` now. Only run `git push origin main` and `git push origin v<version>` after the user confirms in this turn. If they don't confirm, stop here — the commit and tag remain local and can be pushed later.

## 7. After pushing

If the tag was pushed, remind the user that the `.gitea/workflows/ci.yml` `validate-and-release` job (triggered only by the `v*` tag push, not by the `main` push) will create a Codeberg release automatically (provided the `CODEBERG_RELEASE_TOKEN` repository secret is configured), and that Codeberg's push-mirror will propagate `main` and the tag to the GitHub mirror shortly after, where `.github/workflows/release.yml` creates the matching GitHub release independently. Mention that submitting/updating the listing at platform.claude.com/plugins/submit or claude.ai's directory submissions (depending on account type) remains a manual step for the very first submission — after that, new tags are picked up automatically.
