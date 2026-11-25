---
name: make-plan
description: >-
  Creates an implementation plan markdown file under `.plans/` from the
  current project state and the user's request. Starts as `ready` when the
  requirement is already complete, or as `draft` with open questions when
  anything is unclear; refines with the user until the plan is `ready`.
  Does not implement the plan. Use when the user asks to make a plan,
  draft a plan, write a plan file, clarify a plan, or mentions make-plan /
  `.plans/`.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Make plan

Write one plan file for the user's request. **Create and refine the plan
only — never implement it.**

## Plan file

- Directory: `.plans/` at the project root (create if missing).
- Name: `<yyyyMMdd>-<kebab-plan-name>.md` (local date, e.g.
  `20261125-add-auth.md`).
- If that path already exists, append a random 5-letter lowercase suffix
  before `.md` (e.g. `20261125-add-auth-xkqpm.md`). Never overwrite.
- Write the body in the user's language.

### Statuses

| Status | Meaning |
| --- | --- |
| `draft` | Requirement understood, but open questions remain |
| `ready` | No open questions left; plan is executable by a later agent or the user |

`ready` plans are the end state of this skill. Do not start implementation
after writing or promoting a plan.

### Template

```markdown
# <Plan title>

- **Status**: draft | ready
- **Created**: <yyyy-MM-dd>
- **Updated**: <yyyy-MM-dd>

## Goal

<What success looks like.>

## Context

<Relevant project facts discovered while planning (stack, files, conventions).>

## Approach

<Concrete steps or design. Enough for someone else to implement without re-asking the same questions.>

## Open questions

> Omit this section entirely when Status is `ready`.

### Q1. <Question>

- **Why it matters**: <Brief.>
- **Options** (optional): A / B / …
- **Recommendation** (optional): <Preferred option and why.>

## Acceptance criteria

- <Checkable outcomes.>
```

## Workflow

1. Inspect the project enough to ground the plan (layout, existing patterns,
   related docs). Prefer discovery over assumptions.
2. Derive a short `kebab-plan-name` from the request.
3. Decide status:
   - **Complete enough to implement without further product decisions** →
     write the file as `ready` (no Open questions section).
   - **Anything material is unclear** → write as `draft` and list every open
     question. Options and a recommendation are encouraged so the user can
     choose quickly; they are not required.
4. Create `.plans/` if needed, resolve a unique path (suffix if colliding),
   write the file.
5. Reply with the path and status.
   - `draft`: summarize the open questions; ask the user to answer them
     (accept recommendations, pick options, or supply other answers). Update
     the same file as answers arrive: resolve questions, refresh Approach /
     Acceptance criteria, bump `Updated`. When none remain, set
     `Status: ready`, remove the Open questions section, and stop.
   - `ready`: stop. Do not implement.

## Do not

- Implement, scaffold, or partially code the plan under this skill.
- Leave a `draft` without an Open questions section, or a `ready` plan with
  unresolved questions.
- Overwrite an existing plan file.
- Treat this skill as a task runner or sub-agent orchestrator.
