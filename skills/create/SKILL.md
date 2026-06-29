---
description: Create a new MantisBT issue with proper structure. Use when the user wants to file a bug report, feature request, or task in MantisBT — for example "create a ticket for this bug", "file an issue about X", or "add a feature request for Y".
---

Guide the user through creating a well-structured MantisBT issue using `create_issue`.

**Before calling `create_issue`**, gather the following. Use `get_issue_fields` and `get_issue_enums` to discover the available options for the target project.

Required fields:
- **Project** — use `list_projects` to show available projects, then ask the user to pick one
- **Summary** — a concise, descriptive title (one sentence)
- **Description** — steps to reproduce (for bugs) or the full requirement (for features)

Optional but important:
- **Category** — use `get_project_categories` to list valid categories
- **Priority** — normal by default; ask only if the user has a preference
- **Severity** — minor/major/crash etc.; ask for bugs
- **Version** — affected version; use `get_project_versions` to list options

If the user provides sufficient context in `$ARGUMENTS`, extract the information directly without asking again. Always confirm the summary and project before submitting unless the user explicitly says to proceed.

After creating the issue, output the issue ID and URL so the user can navigate directly to it.
