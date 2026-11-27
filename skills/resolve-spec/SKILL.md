---
name: resolve-spec
description: >-
  Implements a local `.local/specs/` markdown spec with front matter
  `status: ready`: if the user names a path or filename, do that spec;
  otherwise pick any ready spec. Implements on the current branch,
  leaves changes uncommitted locally, then deletes the spec file. Never
  drafts or refines specs (`make-spec`); if the target is not ready,
  stop and point to `make-spec`. Heavy work may use
  `task-divide-and-conquer` under resolve-owned close-out. Use when the
  user asks to resolve a spec, pick up a ready spec, work the spec
  queue, or mentions resolve-spec.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.2.0"
---

# Resolve spec

Implement a **local** `.local/specs/*.md` file whose front matter
`status` is `ready`, leave the changes uncommitted on the **current**
branch, then delete the spec file.

## Workflow

### 1. Choose the spec

- If the user named a path or filename under `.local/specs/`: use that
  file. It must exist and have front matter `status: ready`; otherwise
  stop, say why, and if it is still a draft, tell them to clarify or
  promote it with `make-spec` first.
- Otherwise list `.local/specs/*.md` files with `status: ready` and
  pick any one.

If none match: stop and say there is no `ready` spec under
`.local/specs/`.

Read the full file before coding.

### 2. Solve it

Implement the spec in the host project. Discover conventions from the
repo (`package.json`, tests, docs, `AGENTS.md`) rather than assuming
them.

Treat the work as **heavy** (use `task-divide-and-conquer`) when any of
these hold: multiple modules or layers, large unknown surface, or a plan
that clearly needs more than ~5 bounded steps. Otherwise implement in
this conversation.

#### When using `task-divide-and-conquer`

Plan the work with that skill. Under this skill:

1. **Skip its plan-confirmation wait** — after Mode A writes the task
   file, become the orchestrator (or run the fallback chain) immediately
   and run through to parent `Completed`. The user already asked to
   resolve the spec.
2. After parent `Completed`, run step 3 here (delete the spec file).
3. On parent `Failed` or `Cancelled`: stop; do not delete the spec.
   Report what remains.

If `task-divide-and-conquer` is not installed, implement in this
conversation instead, then run step 3 here.

### 3. Delete the spec

After implementation succeeds, delete the same `.local/specs/` markdown
file. Leave all code changes uncommitted locally — do not commit or
push.

If the delete fails: stop and report the error; do not leave a
successful code change paired with a still-present `ready` spec without
telling the user.

## Do not

- When the user **did** name a path / filename, ignore that target in
  favor of another ready spec (unless that file is ineligible).
- Implement specs that are missing, `draft`, or already `done` (point
  the user at `make-spec` when the named file still needs drafting).
- Draft, create, or refine a make-spec-style file under this skill.
- Wait for plan confirmation when using `task-divide-and-conquer` under
  this skill — proceed directly.
- Commit or push as part of this skill.
- Mark the spec `done` instead of deleting it.
