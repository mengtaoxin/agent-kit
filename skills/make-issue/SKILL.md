---
name: make-issue
description: >-
  Creates or updates a GitHub issue (via local `gh`) that captures an
  implementation spec: YAML front matter for status/created/updated,
  body with goal/context/approach/acceptance criteria. Starts as
  `draft` with open questions or `ready` when clear; refines until
  `ready`; may mark `done`. Never implements — even if the user asks
  to implement in the same turn after the issue is ready; hand off to
  `resolve-issue` instead. Do not use when the user asks only to
  implement, resolve, or work the ready issue queue. Use when the user
  asks to make an issue, draft a spec as an issue, clarify a
  make-issue issue, or mentions make-issue.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.4.0"
---

# Make issue

Capture the user's request as a **GitHub issue** (create or update).
**Write and refine the issue only — never implement it.**

Do **not** create local markdown files under `.specs/` or elsewhere.

## Stay in Agent mode

This skill is a normal Agent conversation that uses `gh` to create or
edit an issue, then stops. It is **not** Cursor's Plan mode.

- Do **not** call `SwitchMode` (or any mode switch) to `plan`.
- Do **not** use Cursor's built-in Plan UI / `CreatePlan` tool.
- Do **not** switch to Ask mode.
- After the issue is `ready` / `done` (or the user has the open
  questions), end the turn. Implementation is out of scope — even if
  the user also asked to implement in the same message.

## Prerequisite: `gh`

Before any create/update, verify the local GitHub CLI works in this
repo:

```bash
gh auth status
gh repo view --json nameWithOwner -q .nameWithOwner
```

If `gh` is missing, not authenticated, lacks repo access, or any of
these commands fail: **stop immediately**. Tell the user to install
and authenticate `gh` (`gh auth login`) and ensure they can create
issues on this repository. Do not fall back to local files, the API
with a pasted token, or any other workaround.

## Issue body format

Issue **title**: short human title (same language as the user).

Issue **body**: YAML front matter, then markdown sections. Write the
body in the user's language.

### Metadata (front matter)

| Field | Values / format |
| --- | --- |
| `status` | `draft` \| `ready` \| `done` |
| `created` | `yyyy-MM-dd` (set once on create; never change) |
| `updated` | `yyyy-MM-dd` (bump on every edit) |

### Statuses

| Status | Meaning |
| --- | --- |
| `draft` | Requirement understood, but open questions remain |
| `ready` | No open questions left; executable by a later agent or the user |
| `done` | Work described by the issue is complete (only when the user says so) |

`ready` is the normal end state of drafting. Do not start
implementation after writing or promoting an issue. Set `done` only
when the user asks to mark it done (no code under this skill).

### Label `issue-status-ready`

Keep the GitHub label in sync with front matter `status`:

| Front matter | Label action |
| --- | --- |
| `ready` | Ensure `issue-status-ready` is present (create the label in the repo if missing, then attach) |
| `draft` or `done` | Remove `issue-status-ready` if present |

When marking `done`, also close the GitHub issue with `gh issue close <n>`.

### Template

```markdown
---
status: draft
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

1. Run the `gh` prerequisite checks. On failure, stop and prompt the user.
2. Inspect the project enough to ground the issue (layout, patterns,
   related docs). Prefer discovery over assumptions.
3. Decide whether to **create** or **update**:
   - Create when the user is starting a new request and no issue number/URL
     was given.
   - Update when the user points at an existing issue (`#n`, URL) or when
     refining the issue created earlier in this conversation.
4. Decide `status`:
   - **Complete enough to implement without further product decisions** →
     `ready` (no Open questions section).
   - **Anything material is unclear** → `draft` and list every open
     question. Options and a recommendation are encouraged; when listing
     options, strongest recommendation first.
   - **User asks to mark complete** → `done`, then unlabel and close.
5. Create or update via `gh` (prefer `--body-file` for multiline bodies):

   ```bash
   # create — always assign to the authenticated user
   gh issue create --title "<title>" --body-file <path> --assignee @me
   # if status is ready:
   # gh issue create --title "<title>" --body-file <path> --assignee @me --label issue-status-ready

   # update body (and title if needed)
   gh issue edit <n> --body-file <path>
   # optional: gh issue edit <n> --title "<title>"
   # if status is (now) ready:
   # gh issue edit <n> --add-label issue-status-ready
   # if status is draft or done (and label may exist):
   # gh issue edit <n> --remove-label issue-status-ready
   # if status is done:
   # gh issue close <n>
   ```

   On create, set `created` and `updated` to today's local date, and
   always pass `--assignee @me` so the issue is assigned to the
   authenticated GitHub user. On update, keep `created`, set `updated`
   to today; do not change assignees unless the user asks. Apply the
   label table above on every create/update. If `gh issue create` /
   `edit` / `close` / `label` fails (permissions, network, missing
   issue): **stop** and tell the user to fix `gh` / repo permissions.
   Do not retry with a different storage approach.
6. Reply with the issue URL (or `#n`) and status.
   - `draft`: summarize open questions; ask the user to answer them.
     As answers arrive, update the same issue: resolve questions, refresh
     Approach / Acceptance criteria, bump `updated`. When none remain,
     set `status: ready`, remove Open questions, add label
     `issue-status-ready`, and stop (hand off — do not implement).
   - `ready`: confirm label `issue-status-ready`; tell the user they can
     implement with `resolve-issue` (e.g. resolve `#n`); stop. Do not
     implement.
   - `done`: confirm unlabelled and closed; stop.

## Do not

- Write local `.specs/` (or any other) markdown files for this skill.
- Switch into Cursor Plan mode, Ask mode, or use `CreatePlan` / the Plan UI.
- Implement, scaffold, or partially code the issue under this skill
  (hand off to `resolve-issue` instead).
- Leave a `draft` without Open questions, or a `ready`/`done` issue with
  unresolved questions.
- Leave `issue-status-ready` on a `draft` or `done` issue, or omit it on
  `ready`.
- Invent a workaround when `gh` is unavailable or unauthorized.
- Treat this skill as a task runner or sub-agent orchestrator.
