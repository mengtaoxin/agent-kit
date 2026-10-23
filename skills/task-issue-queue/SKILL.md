---
name: task-issue-queue
description: >-
  Files the user's current request as a GitHub issue labeled task-issue-queue,
  or drains that queue from a parent agent. File when the user states work to
  queue. Solve when the user asks to process, drain, or continue the queue and
  does not state new work: oldest eligible open issue first, one sub-agent
  each, serial. A solved issue becomes one local commit, then is closed, with
  no push. A failed issue is marked failed, its partial edits are rolled back
  to this attempt's baseline, and the queue continues. Earlier commits stay.
  Use when the user asks to file or queue the current request, to process or
  continue the task-issue-queue, or mentions task-issue-queue.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Task issue queue

GitHub issues labeled `task-issue-queue` are the queue. One turn runs one mode.

- **File**: the user states work to queue, including work that names this skill. Create one issue and stop.
- **Solve**: the user asks to process, drain, or continue the queue, and does not state a new piece of work.

A message that states work is File. "处理登录失败" states work. "继续处理队列" does not. If a message can be either mode, ask. Do not do both.

Use `gh` against the current repository. If `gh` cannot access it, stop and report that. Do not invent another queue.

Write issue titles, bodies, close comments, and failure reasons in the user's language.

## File

1. Ensure the label exists. `--force` updates it when it is already there:

```bash
gh label create task-issue-queue --force --description "Serial queue drained by the task-issue-queue skill"
```

2. Open one issue. The title is a short summary. The body is the user's request, with its constraints intact.

```bash
gh issue create --title "<title>" --label task-issue-queue --body-file <body-file>
```

3. Reply with the issue number and URL. Stop. Do not implement the request, launch a sub-agent, close the issue, commit, or start Solve.

## Solve

This conversation is the parent. It passes the issue number, URL, and baseline commit to one sub-agent. It does not read the issue body or implement it.

If this host cannot launch sub-agents, stop. Solve needs a sub-agent.

An issue is eligible when it is open, has `task-issue-queue`, and does not have `task-issue-queue-failed`.

1. List eligible issues, oldest first:

```bash
gh issue list --limit 30 --search 'label:"task-issue-queue" -label:"task-issue-queue-failed" state:open sort:created-asc' --json number,title,createdAt,url
```

No issue means the workable queue is clear. Say so and stop.

2. Take the first issue. If several share its `createdAt`, pick the lowest number.
3. Stop if `git stash list` already contains `task-issue-queue baseline`. Record the baseline with `git rev-parse HEAD`. If `git status --porcelain` is non-empty, stash it, including untracked files:

```bash
git stash push -u -m "task-issue-queue baseline"
```

If the tree is still dirty, stop and report that. Do not launch a sub-agent on someone else's uncommitted work.
4. Launch **one** sub-agent with the prompt below, including the baseline commit. Wait until it finishes. Do not start the next issue, and do not do the work here.
5. Read the outcome from GitHub, not from the sub-agent's summary:

```bash
gh issue view <number> --json state,labels
```

6. If it is still eligible, mark it failed and leave it open:

```bash
gh label create task-issue-queue-failed --force --description "Could not be solved by the task-issue-queue skill"
gh issue comment <number> --body "The sub-agent returned without closing this issue or marking it failed."
gh issue edit <number> --add-label task-issue-queue-failed
```

7. If the issue has the failed label, confirm the baseline is `HEAD` or an ancestor of `HEAD`. A commit is an ancestor of itself. If this check fails, stop and report both commits. Leave the baseline stash in place. Do not reset.

```bash
git merge-base --is-ancestor <baseline> HEAD
```

When the check succeeds, discard this attempt. Commits at or before the baseline stay, pushed or not. Do not push.

```bash
git reset --hard <baseline>
git clean -fd
```

8. If the issue is closed and the tree has uncommitted changes, create the one missing commit for this issue, in the repository's commit style, mentioning the issue number. Do not push. A clean tree and an unchanged `HEAD` means there was nothing to commit.
9. When this run stashed a baseline, restore it with `git stash pop`. If the pop conflicts, stop and report that. Do not continue with a conflicted tree.
10. Post one short status line, then go back to step 1.

Stop when a fresh list has no eligible issue. Report how many issues this run closed and how many it marked failed.

Sub-agent prompt (use the real issue number, URL, and baseline):

```text
Solve GitHub issue #<number> (<url>) (task-issue-queue). The baseline commit is <baseline>. Do only that issue. Do not push, amend, or reset commits at or before the baseline. Do not launch sub-agents or start another issue. Do not touch the stash named "task-issue-queue baseline".

Read the issue:

gh issue view <number> --comments

When it is done, make one local commit of this issue's changes, using the repository's commit message style and mentioning the issue number. If nothing changed, skip the commit. Then close it:

gh issue close <number> --reason completed --comment "<summary>"

When it cannot be completed, do not commit. Problems inside the issue's scope are part of solving it. Mark it failed when the premise is false, the request contradicts the repository, required access or information is unavailable, or you cannot finish it. First confirm the baseline is HEAD or an ancestor of HEAD. If that check fails, do not reset or clean; report both commits. When it succeeds, roll this attempt back, then leave the issue open:

git merge-base --is-ancestor <baseline> HEAD
git reset --hard <baseline>
git clean -fd
gh label create task-issue-queue-failed --force --description "Could not be solved by the task-issue-queue skill"
gh issue comment <number> --body "<failure reason>"
gh issue edit <number> --add-label task-issue-queue-failed

Return one line: closed #<number>, or failed #<number> and the reason.
```
