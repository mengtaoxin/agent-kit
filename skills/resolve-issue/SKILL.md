---
name: resolve-issue
description: >-
  Implements an open GitHub issue labeled `issue-status-ready` via local
  `gh`: if the user names `#n` or an issue URL, do that issue; otherwise
  pick the oldest ready. Implements on the current branch, commits and
  pushes with `Fix #<n>` (push is required by this skill), then sets
  front matter `status: done`, removes `issue-status-ready`, and closes
  the issue (do not rely only on auto-close). Never drafts or refines
  specs (`make-issue`); if the target is not ready, stop and point to
  `make-issue`. Use when the user asks to resolve an issue, pick up a
  ready issue, work the issue queue, or mentions resolve-issue.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.3.0"
---

# Resolve issue

Implement an **open** GitHub issue labeled `issue-status-ready`, then
commit and push on the **current** branch, and close it out of the ready
queue.

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

### 2. Choose the issue

- If the user named an issue (`#n`, URL, or number): use that issue. It
  must be **open** and have label `issue-status-ready`; otherwise stop,
  say why, and if it is still a draft/spec, tell them to clarify or promote
  it with `make-issue` first.
- Otherwise list open issues with label `issue-status-ready`, oldest
  first, and take the first:

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

Treat the work as **heavy** (use `task-divide-and-conquer`) when any of
these hold: multiple modules or layers, large unknown surface, or a plan
that clearly needs more than ~5 bounded steps. Otherwise implement in
this conversation.

#### When using `task-divide-and-conquer`

Plan the work with that skill. Under this skill:

1. **Skip its plan-confirmation wait** — after Mode A writes the task
   file, become the orchestrator (or run the fallback chain) immediately
   and run through to parent `Completed`. The user already asked to
   resolve the issue.
2. **Include a final commit / push subtask** — last work subtask should
   land the changes per step 4 below (`Fix #<n>`, then push). Do not leave
   commit / push to a post-hoc step outside the task file.
3. After parent `Completed`, run step 5 here (mark done, unlabel, close).
4. On parent `Failed` or `Cancelled`: stop; do not commit, push, or close
   the GitHub issue. Report what remains.

If `task-divide-and-conquer` is not installed, implement in this
conversation instead, then run steps 4–5 here.

### 4. Commit and push on the current branch

Stay on the **current** branch (do not create a new branch unless the user
or host project already requires it). When using
`task-divide-and-conquer`, this step is the final subtask; otherwise run
it in this conversation after implementation.

This skill **requires** a push after a successful commit so the remote and
the issue queue stay aligned — even when a general user rule says not to
push unless asked.

1. Stage only the changes that resolve this issue.
2. Commit with a message that includes a GitHub closing keyword and the
   issue number, for example:

   ```text
   Fix #<n>: <short summary of what changed>
   ```

   Prefer `Fix #<n>` (also acceptable: `Fixes #<n>`, `Closes #<n>`). Keep
   the keyword + `#<n>` so GitHub can link the commit (and auto-close if
   it later lands on the default branch).
3. Push the current branch to its remote (`git push`, or `git push -u`
   if upstream is missing).

Follow the host project's commit / git safety rules (no force push, no
hook skips, no secrets). If commit or push fails: stop and report the
error; do not invent a workaround that drops the issue reference.

### 5. Mark done and leave the ready queue

After a successful push, **always** close out the issue — do not rely only
on `Fix #<n>` auto-close (that only runs when the commit reaches the
default branch; a feature-branch push often leaves the issue open and
still labeled ready).

1. Update the issue body via `--body-file`: set front matter
   `status: done`, keep `created`, set `updated` to today's local date.
   Leave Goal / Context / Approach / Acceptance criteria intact; do not
   reintroduce Open questions.
2. Remove the ready label:

   ```bash
   gh issue edit <n> --remove-label issue-status-ready
   ```

3. If the issue is still open, close it:

   ```bash
   gh issue close <n>
   ```

If body edit, unlabel, or close fails: stop and report the error; do not
leave a successful code push paired with a still-`ready` open issue without
telling the user.

## Do not

- Invent a workaround when `gh` is unavailable or unauthorized.
- When the user did **not** name an issue, pick a newer ready issue while
  an older open `issue-status-ready` issue exists.
- When the user **did** name `#n` / a URL, ignore that target in favor of
  queue order (unless that issue is ineligible).
- Implement issues that are closed or lack label `issue-status-ready`
  (point the user at `make-issue` when the named issue still needs
  drafting).
- Skip the commit message issue reference (`Fix #<n>` or equivalent).
- Skip push, or skip the post-push done / unlabel / close step after a
  successful push.
- Draft, create, or refine a make-issue-style spec under this skill.
- Wait for plan confirmation when using `task-divide-and-conquer` under
  this skill — proceed directly and include commit / push as the final
  subtask.
