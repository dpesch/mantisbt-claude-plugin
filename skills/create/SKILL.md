---
description: Create a new MantisBT issue with proper structure. Use when the user wants to file a bug report, feature request, or task in MantisBT — for example "create a ticket for this bug", "file an issue about X", or "add a feature request for Y".
---

Guide the user through creating a well-structured MantisBT issue using `create_issue`.

**Before calling `create_issue`**, gather the following. Use `get_issue_fields` to discover valid `select` field names, and the `mantis://enums` resource (fall back to `get_issue_enums` if the resource isn't available) to discover the valid severity, priority, status, and reproducibility values — these are configurable and may be localized per instance, so don't assume fixed values.

Required fields:
- **Project** — use `list_projects` (or the `mantis://projects` resource) to show available projects, then ask the user to pick one
- **Summary** — a concise, descriptive title (one sentence)
- **Description** — steps to reproduce (for bugs) or the full requirement (for features); this is a required field, at least one character

Optional but important:
- **Category** — use `get_project_categories` to list valid categories
- **Severity** — ask for bugs; use the enum values discovered above
- **Priority** — suggest a value based on severity and context, but let the user confirm or override it rather than hard-coding a mapping
- **Version** — affected version; use `get_project_versions` to list options
- **Target version** / **fixed-in version** — if the user mentions a release target
- **Steps to reproduce** / **additional information** / **reproducibility** — for bug reports, pass these as their own fields rather than folding them into the description
- **Custom fields** — pass via the `custom_fields` parameter (`[{field: {id|name}, value}]`) if the project defines any and the user mentions relevant values

If the user provides sufficient context in `$ARGUMENTS`, extract the information directly without asking again.

## Confirm before creating

Show a summary of all gathered fields and use **AskUserQuestion** to confirm before calling `create_issue` — offer to create, change a field, or cancel. Don't ask this as plain text; use the tool so the user gets clear options. Skip this step only if the user has explicitly said to proceed without confirmation.

## After creating

`create_issue` returns the full created issue, including its `view_url`. Output the issue ID and that URL directly — don't construct the link manually.
