# Changelog

All notable changes to this plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.11.0] - 2026-07-04

### Added
- `sync` skill for checking and refreshing the MantisBT metadata cache
- Explicit read-only tool allowlist on the `mantis-researcher` agent

### Changed
- `research` and `create` skills updated for MCP server v1.11.0: MCP resources (`mantis://...`), `select` field projection, batch `get_issues`, guided confirmation before creating an issue, `custom_fields` support
- Pinned MantisBT MCP server to v1.11.0

## [1.10.3] - 2026-07-04

### Added
- Initial plugin structure: MCP server configuration, `research` and `create` skills, `mantis-researcher` agent
