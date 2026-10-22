---
name: task-issue-queue
description: >-
  Files the user's current request as a GitHub issue labeled task-issue-queue,
  or drains that queue from a parent agent: eligible open issues only, oldest
  first, one sub-agent per issue, serial. Close a solved issue with no commit.
  Mark an unsolvable issue failed, record the reason, and continue. Use when
  the user asks to file or queue the current request, to process or continue
  the task-issue-queue, or mentions task-issue-queue.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Task issue queue

GitHub issues labeled `task-issue-queue` are the queue. One turn runs one mode.

- **File**: turn the user's current request into one issue. Do not solve it.
- **Solve**: the parent drains eligible open issues, oldest first, one sub-agent at a time. It passes the issue number and URL only. The sub-agent closes a solved issue, or marks an unsolvable one failed and leaves it open.

Use `gh` against the current repository. If `gh` cannot access it, stop and report that. Do not invent another queue.

Write issue titles, bodies, close comments, and failure reasons in the user's language.

## File

1. Create the label when it is missing:

```bash
gh label create task-issue-queue --description "Serial queue drained by the task-issue-queue skill"
```

2. Open one issue. The title is a short summary. The body is the user's request, with its constraints intact.

```bash
gh issue create --title "<title>" --label task-issue-queue --body-file <body-file>
```

3. Reply with the issue number and URL. Stop. Do not implement the request, launch a sub-agent, close the issue, commit, or start Solve.

## Solve

This conversation is the parent. It chooses the next issue and passes the number and URL to one sub-agent. It does not read the issue body or implement it.

An issue is eligible when it is open, has `task-issue-queue`, and does not have `task-issue-queue-failed`.

1. List open issues with the queue label:

```bash
gh issue list --state open --label task-issue-queue --limit 200 --json number,title,createdAt,labels,url
```

Skip issues that already have `task-issue-queue-failed`. A missing queue label, or no eligible issue, means the workable queue is clear. Say so and stop.

2. Choose the earliest `createdAt`. Break ties with the lower issue number.
3. Launch **one** sub-agent with the prompt below. Wait until it finishes. Do not start the next issue, and do not do the work here.
4. If this host cannot launch sub-agents, stop and say Solve needs a sub-agent. Do not solve the issue in the parent.
5. Look at that issue again. Closed, or still open with `task-issue-queue-failed`, means the sub-agent recorded the outcome. Post one short status line, then go back to step 1.
6. If it is still eligible, the sub-agent returned without recording an outcome. Create `task-issue-queue-failed` when it is missing, mark the issue failed, leave it open, then go back to step 1.

```bash
gh label create task-issue-queue-failed --description "Could not be solved by the task-issue-queue skill"
gh issue comment <number> --body "The sub-agent returned without closing this issue or marking it failed."
gh issue edit <number> --add-label task-issue-queue-failed
```

Stop when a fresh list has no eligible issue. Report how many issues this run closed and how many it marked failed.

Sub-agent prompt (use the real issue number and URL):

```text
Solve GitHub issue #<number> (<url>) (task-issue-queue). Read it with gh and do only that issue. You share the checkout; leave other uncommitted work untouched. Do not commit, stage, push, or open a pull request. Do not launch sub-agents or start another issue.

When it is done, close it with a short summary:
gh issue close <number> --reason completed --comment "<summary>"

When it cannot be completed, leave it open. Create task-issue-queue-failed if missing, comment the reason, and add that label. Use this when the premise is false, the request contradicts the repository, required access or information is unavailable, or you cannot finish it. Problems inside the issue's scope are part of solving it. Do not revert partial edits unless the user asks.

Return one line: closed #<number>, or failed #<number> and the reason.
```
