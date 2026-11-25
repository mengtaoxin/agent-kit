---
name: make-spec
description: >-
  Creates an implementation spec markdown file under `.specs/` from the
  current project state and the user's request. Runs in normal Agent
  (edit) mode — never Cursor Plan mode, Ask mode, or CreatePlan.
  Starts as `ready` when the requirement is already complete, or as
  `draft` with open questions when anything is unclear; refines with the
  user until the spec is `ready`. Does not implement the spec. Use when
  the user asks to make a spec, draft a spec, write a spec file, clarify
  a spec, or mentions make-spec / `.specs/`.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Make spec

Write one spec file for the user's request. **Create and refine the spec
only — never implement it.**

## Stay in Agent mode

This skill is a normal Agent conversation that edits a `.specs/*.md`
file, then stops. It is **not** Cursor's Plan mode.

- Do **not** call `SwitchMode` (or any mode switch) to `plan`.
- Do **not** use Cursor's built-in Plan UI / `CreatePlan` tool.
- Do **not** switch to Ask mode. Stay in Agent so you can write the
  spec file.
- After the spec is `ready` (or the user has the open questions), end
  the turn. Who reads or implements the spec later is out of scope for
  this conversation.

## Spec file

- Directory: `.specs/` at the project root (create if missing).
- Name: `<yyyyMMdd>-<kebab-spec-name>.md` (local date, e.g.
  `20261125-add-auth.md`).
- If that path already exists, append a random 5-letter lowercase suffix
  before `.md` (e.g. `20261125-add-auth-xkqpm.md`). Never overwrite.
- Write the body in the user's language.

### Statuses

| Status | Meaning |
| --- | --- |
| `draft` | Requirement understood, but open questions remain |
| `ready` | No open questions left; spec is executable by a later agent or the user |

`ready` specs are the end state of this skill. Do not start implementation
after writing or promoting a spec.

### Template

```markdown
# <Spec title>

- **Status**: draft | ready
- **Created**: <yyyy-MM-dd>
- **Updated**: <yyyy-MM-dd>

## Goal

<What success looks like.>

## Context

<Relevant project facts discovered while drafting (stack, files, conventions).>

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

1. Inspect the project enough to ground the spec (layout, existing patterns,
   related docs). Prefer discovery over assumptions.
2. Derive a short `kebab-spec-name` from the request.
3. Decide status:
   - **Complete enough to implement without further product decisions** →
     write the file as `ready` (no Open questions section).
   - **Anything material is unclear** → write as `draft` and list every open
     question. Options and a recommendation are encouraged so the user can
     choose quickly; they are not required.
4. Create `.specs/` if needed, resolve a unique path (suffix if colliding),
   write the file.
5. Reply with the path and status.
   - `draft`: summarize the open questions; ask the user to answer them
     (accept recommendations, pick options, or supply other answers). Update
     the same file as answers arrive: resolve questions, refresh Approach /
     Acceptance criteria, bump `Updated`. When none remain, set
     `Status: ready`, remove the Open questions section, and stop.
   - `ready`: stop. Do not implement.

## Do not

- Switch into Cursor Plan mode, Ask mode, or use `CreatePlan` / the Plan UI.
- Implement, scaffold, or partially code the spec under this skill.
- Leave a `draft` without an Open questions section, or a `ready` spec with
  unresolved questions.
- Overwrite an existing spec file.
- Treat this skill as a task runner or sub-agent orchestrator.
