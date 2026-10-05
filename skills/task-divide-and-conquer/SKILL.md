---
name: task-divide-and-conquer
description: >-
  Splits a large user request into a parent task with serial subtasks tracked
  in a markdown file under `docs-task-divide-and-conquer/`, then executes one
  subtask per conversation so no single chat grows too long or runs too long.
  Use when the user asks to divide and conquer a big task, split work across
  multiple conversations, create a parent task / subtasks, continue or resume
  a task file, or cancel one; also when they mention task-divide-and-conquer.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Task divide and conquer

One parent task → N subtasks, executed **serially**, **one subtask per
conversation**. A markdown task file is the only shared state between
conversations, so it must always be accurate.

## Task file

- Directory: `docs-task-divide-and-conquer/` at the project root (create if
  missing).
- Name: `<YYYY-MM-DD>-<kebab-slug>.md`; never overwrite an existing file.
- Write the content in the user's language. No task file exists unless a
  parent task was created.

### Parent statuses

| Status | Meaning |
| --- | --- |
| `In progress` | Created; subtasks remain |
| `Completed` | All subtasks done, including the commit |
| `Failed` | Task design problem found; reason recorded |
| `Cancelled` | Stopped, usually by the user |

`Failed`, `Completed`, and `Cancelled` are terminal: never execute subtasks of
such a task. A redesign means a new parent task (new file).

### Template

```markdown
# <Parent task title>

- **Status**: In progress
- **Failure reason**: —
- **Created**: <YYYY-MM-DD>
- **Updated**: <YYYY-MM-DD>
- **Progress**: 1/<N> subtasks done

## Requirement

<The user's request, constraints, and overall acceptance criteria.>

## Checklist

- [x] 1. Create parent task
- [ ] 2. <Subtask title>
- [ ] <N>. Commit changes

## Subtasks

### 1. Create parent task

- **Status**: Done
- **Goal**: Analyze the request and create this task file.
- **Result**: <Key findings: relevant files, conventions, decisions.>

### 2. <Subtask title>

- **Status**: Pending
- **Goal**: <One outcome.>
- **Scope**: <Files / modules / areas to touch.>
- **Steps**: <Short ordered list.>
- **Acceptance**: <How to verify: tests, build, manual check.>
- **Result**: —

### <N>. Commit changes

- **Status**: Pending
- **Goal**: Commit all changes made by this task.
- **Acceptance**: Commit created; working tree clean for task-related files.
- **Result**: —
```

Subtask statuses: `Pending`, `Done`, `Failed`.

## Mode A: create the parent task

Conversation 1 only plans; it does not start subtask 2.

1. Clarify the requirement if it is ambiguous; explore the codebase enough to
   plan (conventions, affected areas, test/build commands).
2. Split into serial subtasks. Each subtask must:
   - fit comfortably in one conversation (bounded files, bounded time);
   - depend only on earlier subtasks;
   - have concrete acceptance criteria;
   - leave the project in a working state where practical.
3. Fixed bookends: subtask 1 is **Create parent task** (already `Done` when
   the file is written); the last subtask is **Commit changes**.
4. Write the file from the template with status `In progress`.
5. Reply with the file path, the subtask list, and the prompt to continue in
   a **new** conversation, e.g.
   `Continue task docs-task-divide-and-conquer/<file>.md (task-divide-and-conquer)`.

## Mode B: run the next subtask

1. Locate the task file: use the one the user named; otherwise list files in
   `docs-task-divide-and-conquer/` with status `In progress` — use it if there
   is exactly one, else ask.
2. If the status is terminal, report it and stop.
3. Pick the **first** `Pending` subtask. Read the requirement and the
   `Result` of earlier subtasks for context.
4. Execute only that subtask and verify its acceptance criteria.
5. Update the file:
   - subtask `Status: Done`, fill `Result` (what changed, files, anything the
     next subtask must know);
   - tick its checklist box, bump `Progress` and `Updated`.
6. Stop. Reply with what was done and the prompt for the next conversation.
   Do not start the next subtask, even if time allows.

### Commit subtask (last)

1. Review the diff; stage files changed by this task (include the task file
   unless the host project ignores `docs-task-divide-and-conquer/`). Follow
   the host repo's commit message style and hooks.
2. Before committing, mark the subtask `Done`, the parent `Completed`, and
   update `Progress`/`Updated`, so the committed file is final.
3. If the commit fails (hook, conflict), fix it if the fix is in scope;
   otherwise revert those marks to `Pending` / `In progress`, record the
   problem in `Result`, and report. A failing hook is not a design failure.
4. Do not push unless the user asks.

## Edge cases

### Task design is wrong

Examples: a subtask's premise is false, the order is impossible, the scope is
far larger than planned, or the requirement contradicts the codebase.

1. Do not silently redesign or add subtasks.
2. Mark the current subtask `Failed` with details in `Result`.
3. Set parent `Status: Failed` and `Failure reason`: which subtask, what was
   wrong, and what a redesigned task should change.
4. **Do not revert** local changes already made; list them in the reply.
5. Stop and report.

Ordinary bugs or failing tests inside a subtask's scope are not design
failures — fix them as part of the subtask.

### User cancels

Set parent `Status: Cancelled` (add the reason if given) and `Updated`. Do not
revert local changes unless the user asks.

### File and reality disagree

If the file says `Pending` but the work appears done (e.g. a previous chat
ended before updating), verify against the acceptance criteria, then mark it
`Done` with a note instead of redoing it.
