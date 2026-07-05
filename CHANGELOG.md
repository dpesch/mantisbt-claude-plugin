# Changelog

All notable changes to this plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.11.0+ci.2] - 2026-07-05

Cosmetic-only release — `displayName` rewording and a new privacy document, no functional change. See [DEVELOPMENT.md](DEVELOPMENT.md#infra-only-releases-cin) for what this suffix means.

### Added
- `PRIVACY.md` describing the plugin's data handling (needed for the Anthropic plugin directory submission)
- Privacy section linking `PRIVACY.md` from both READMEs

### Changed
- Renamed the plugin's `displayName` from "MantisBT" to "MantisBT Integration" to avoid implying it *is* MantisBT itself

## [1.11.0+ci.1] - 2026-07-05

Infra-only release — no change to the shipped plugin content. See [DEVELOPMENT.md](DEVELOPMENT.md#infra-only-releases-cin) for what this suffix means.

### Changed
- CI reduced to a single `validate-and-release` job, triggered only by `v*` tag pushes or PRs (plain pushes to `main` no longer run CI)
- Added `.github/workflows/release.yml` so tagged releases also produce a GitHub release on the new push-mirror
- Introduced the `+ci.N` build-metadata scheme for infra-only releases (this one), including a regex escaping fix in both release workflows for versions containing `+`

## [1.11.0] - 2026-07-05

### Added
- `sync` skill for checking and refreshing the MantisBT metadata cache
- `setup` skill: guided setup, status check, and troubleshooting for the optional semantic search
- Explicit read-only tool allowlist on the `mantis-researcher` agent
- README section documenting the optional local semantic search (env vars, defaults)

### Changed
- `research` and `create` skills updated for MCP server v1.11.0: MCP resources (`mantis://...`), `select` field projection, batch `get_issues`, guided confirmation before creating an issue, `custom_fields` support
- Pinned MantisBT MCP server to v1.11.0

## [1.10.3] - 2026-07-04

### Added
- Initial plugin structure: MCP server configuration, `research` and `create` skills, `mantis-researcher` agent
