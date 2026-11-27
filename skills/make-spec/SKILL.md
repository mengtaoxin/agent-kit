---
name: make-spec
description: >-
  Creates or updates a local implementation spec under `.local/specs/`
  (`draft` / `ready` / `done`, YAML front matter + goal/context/approach/
  acceptance criteria). Starts as `draft` with open questions or `ready`
  when clear; refines until `ready`; may mark `done`. Never implements —
  even if the user asks to implement in the same turn after the spec is
  ready; hand off to `resolve-spec` instead. Do not use when the user
  asks only to implement, resolve, or work the ready spec queue. Use when
  the user asks to make a spec, draft a local spec, clarify a make-spec
  file, or mentions make-spec.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.1"
---

# Make spec

Capture the user's request as a **local markdown spec** under
`.local/specs/` (create or update).
**Write and refine the spec only — never implement it.**

## Stay in Agent mode

This skill is a normal Agent conversation that writes a local file, then
stops. It is **not** Cursor's Plan mode.

- Do **not** call `SwitchMode` (or any mode switch) to `plan`.
- Do **not** use Cursor's built-in Plan UI / `CreatePlan` tool.
- Do **not** switch to Ask mode.
- After the spec is `ready` / `done` (or the user has the open
  questions), end the turn. Implementation is out of scope — even if
  the user also asked to implement in the same message.

## Spec location

- Directory: `.local/specs/` at the project root (create if missing).
  Under `.local/` so specs stay out of the project's docs tree.
- Name: `<YYYYMMDD>-<kebab-slug>.md`; never overwrite an existing file
  when creating a new spec.
- Write the content in the user's language.

## Spec format

### Metadata (front matter)

| Field | Values / format |
| --- | --- |
| `status` | `draft` \| `ready` \| `done` |
| `title` | Short human title (same language as the user) |
| `created` | `yyyy-MM-dd` (set once on create; never change) |
| `updated` | `yyyy-MM-dd` (bump on every edit) |

### Statuses

| Status | Meaning |
| --- | --- |
| `draft` | Requirement understood, but open questions remain |
| `ready` | No open questions left; executable by a later agent or the user |
| `done` | Work described by the spec is complete (only when the user says so) |

`ready` is the normal end state of drafting. Do not start
implementation after writing or promoting a spec. Set `done` only
when the user asks to mark it done (no code under this skill).

### Template

```markdown
---
status: draft
title: <Short human title>
created: <yyyy-MM-dd>
updated: <yyyy-MM-dd>
---

## Goal

<What success looks like.>

## Context

<Relevant project facts discovered while drafting (stack, files, conventions).>

## Approach

<Concrete steps or design. Enough for someone else to implement without re-asking the same questions.>

## Open questions

> Omit this section entirely when status is `ready` or `done`.

### Q1. <Question>

- **Why it matters**: <Brief.>
- **Options** (optional): A / B / … — order by recommendation strength, strongest first.
- **Recommendation** (optional): <Preferred option and why.>

## Acceptance criteria

- <Checkable outcomes.>
```

## Workflow

1. Inspect the project enough to ground the spec (layout, patterns,
   related docs). Prefer discovery over assumptions.
2. Decide whether to **create** or **update**:
   - Create when the user is starting a new request and no spec path
     was given.
   - Update when the user points at an existing `.local/specs/*.md`
     path (or filename) or when refining the spec created earlier in
     this conversation.
3. Decide `status`:
   - **Complete enough to implement without further product decisions** →
     `ready` (no Open questions section).
   - **Anything material is unclear** → `draft` and list every open
     question. Options and a recommendation are encouraged; when listing
     options, strongest recommendation first.
   - **User asks to mark complete** → `done`.
4. Write or update the file:
   - On create: ensure `.local/specs/` exists; pick a unique
     `<YYYYMMDD>-<kebab-slug>.md`; set `created` and `updated` to
     today's local date.
   - On update: keep `created` and the filename; set `updated` to today;
     rewrite the body to match the template and current status.
5. Reply with the file path and status.
   - `draft`: summarize open questions; ask the user to answer them.
     As answers arrive, update the same file: resolve questions, refresh
     Approach / Acceptance criteria, bump `updated`. When none remain,
     set `status: ready`, remove Open questions, and stop (hand off —
     do not implement).
   - `ready`: tell the user they can implement with `resolve-spec`
     (e.g. resolve `.local/specs/<file>.md`); stop. Do not implement.
   - `done`: confirm the path and status; stop.

## Do not

- Switch into Cursor Plan mode, Ask mode, or use `CreatePlan` / the Plan UI.
- Implement, scaffold, or partially code the spec under this skill
  (hand off to `resolve-spec` instead).
- Leave a `draft` without Open questions, or a `ready`/`done` spec with
  unresolved questions.
- Overwrite an unrelated existing spec file when creating a new one.
- Treat this skill as a task runner or sub-agent orchestrator.
