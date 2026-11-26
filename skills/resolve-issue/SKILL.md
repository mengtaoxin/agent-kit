---
name: resolve-issue
description: >-
  Picks the oldest open GitHub issue labeled `issue-status-ready` via local
  `gh`, implements it on the current branch, then commits and pushes with a
  message that references the issue (e.g. `Fix #<n>`) so GitHub can auto-close
  it. Stops if `gh` is missing or unauthorized. Use when the user asks to
  resolve an issue, pick up a ready issue, work the issue queue, or mentions
  resolve-issue.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Resolve issue

Implement the oldest **open** GitHub issue that has label
`issue-status-ready`, then commit and push on the **current** branch.

## Prerequisite: `gh`

Before fetching or changing anything, verify the local GitHub CLI:

```bash
gh auth status
gh repo view --json nameWithOwner -q .nameWithOwner
```

If `gh` is missing, not authenticated, lacks repo access, or any of these
commands fail: **stop immediately**. Tell the user to install and
authenticate `gh` (`gh auth login`) and ensure they can list issues and push
to this repository. Do not fall back to the API with a pasted token, the
web UI, or any other workaround.

## Workflow

### 1. Check `gh`

Run the prerequisite checks above. On failure, stop and prompt the user to
fix the environment.

### 2. Fetch the oldest ready issue

List open issues with label `issue-status-ready`, ordered by creation time
ascending, take the first:

```bash
gh issue list \
  --state open \
  --label issue-status-ready \
  --search "sort:created-asc" \
  --limit 1 \
  --json number,title,body,url,createdAt,labels
```

If none match: stop and say there is no open `issue-status-ready` issue.

Read the full issue (body and comments) before coding:

```bash
gh issue view <n> --comments
```

### 3. Solve it

Implement the issue in the host project. Discover conventions from the repo
(`package.json`, tests, docs, `AGENTS.md`) rather than assuming them.

If the work is a **heavy** task, apply **`task-divide-and-conquer`**
immediately — do **not** wait for user confirmation of the plan; run the
divide-and-conquer flow straight through to completion of this issue.

Otherwise implement in this conversation.

### 4. Commit and push on the current branch

Stay on the **current** branch (do not create a new branch unless the user
or host project already requires it).

1. Stage only the changes that resolve this issue.
2. Commit with a message that includes a GitHub closing keyword and the
   issue number, for example:

   ```text
   Fix #<n>: <short summary of what changed>
   ```

   Prefer `Fix #<n>` (also acceptable: `Fixes #<n>`, `Closes #<n>`). The
   keyword + `#<n>` must appear so GitHub can link and auto-close the issue
   when the commit lands on the default branch.
3. Push the current branch to its remote (`git push`, or `git push -u`
   if upstream is missing).

Follow the host project's commit / git safety rules (no force push, no
hook skips, no secrets). If commit or push fails: stop and report the
error; do not invent a workaround that drops the issue reference.

## Do not

- Invent a workaround when `gh` is unavailable or unauthorized.
- Pick a newer issue when an older open `issue-status-ready` issue exists.
- Implement issues that are closed or lack label `issue-status-ready`.
- Skip the commit message issue reference (`Fix #<n>` or equivalent).
- Wait for plan confirmation when using `task-divide-and-conquer` under
  this skill — proceed directly.
