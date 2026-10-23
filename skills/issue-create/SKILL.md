---
name: issue-create
description: >-
  Creates one open GitHub issue for the user's current request and applies a
  label the user names. Leaves the issue open and does not implement the
  request. Use when the user asks to file or open the current request as a
  GitHub issue with a specific label, or mentions issue-create.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Issue create

Create one open GitHub issue that carries the label the user names. Then stop.

The label comes from the user. If they do not name one, ask. Do not invent a label.

Use `gh` against the current repository. If `gh` cannot access it, stop and report that.

Write the title and body in the user's language.

1. Ensure the label exists. Match the name exactly. Create it only when it is missing. Leave an existing label unchanged. If create fails, stop.

```bash
gh label list --limit 200 --search "<label>" --json name
gh label create "<label>" --description "Applied by issue-create"
```

2. Open one issue. The title is a short summary. The body is the user's request, with its constraints intact. The issue stays open.

```bash
gh issue create --title "<title>" --label "<label>" --body-file <body-file>
```

3. Reply with the issue number and URL. Stop. Do not implement the request, launch a sub-agent, close the issue, or commit.
