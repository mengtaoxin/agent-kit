---
name: optimize-agent-customization
description: >-
  Audit agent customization for the current project: skills, Cursor rules,
  MCP config, AGENTS.md, and docs. Finds oversized markdown, stale structure
  docs, contradictions, unclear guidance, and weak skill metadata, then
  recommends the high-priority fixes. Apply edits only after the user agrees
  which items to change. Use when the user asks to optimize agent
  customization, clean up agent docs/rules/skills, or audit AGENTS.md / MCP —
  not for application bug fixes, product features, or program-entry scripts.
license: MIT
metadata:
  author: mengtaoxin
  version: "3.0.0"
---

# Optimize agent customization

Audit the current project's **agent customization** surface and recommend the highest-priority fixes. These files shape later agent behavior, so recommend first and edit only after the user chooses what to apply.

## Scope (in scope)

Only these paths (and close equivalents if the project uses a variant layout):

- `./agents/skills/**` (also `./skills/**`, `.cursor/skills/**`, `.agents/skills/**` when that is where skills live)
- `.cursor/rules/**`
- `.cursor/mcp.json`
- `AGENTS.md`
- `docs/**`

`scripts/**` is **out of scope**. Many projects use it for main program entries, so do not inventory, rewrite, or clean up those files as agent customization. A customization file may still name a script; if that reference is wrong, the recommendation belongs in the customization file, not under `scripts/**`.

Do **not** treat application source code, product features, or unrelated refactors as in scope unless a customization file must change to stay truthful.

## What to look for

Prioritize issues like:

- **Oversized markdown** — a single file is too long for reliable agent use; split or progressive-disclose
- **Stale structure docs** — documented layout/paths diverge from the real tree
- **Contradictions** — docs (or docs vs rules/skills/`AGENTS.md`) disagree
- **Unclear guidance** — vague, ambiguous, or missing when/how-to-apply instructions
- **Dead or duplicate guidance** — unused skills, overlapping rules, repeated copy that drifts
- **Broken references** — links, paths, script names, or MCP server entries that do not exist
- **Weak skill metadata** — missing/vague `description`, poor trigger terms, inconsistent naming

## Workflow

Copy and track:

```text
Optimize progress:
- [ ] 1. Inventory
- [ ] 2. Assess
- [ ] 3. Recommend (stop for confirmation)
- [ ] 4. Apply
- [ ] 5. Verify
```

### 1. Inventory

Map what exists under the scope paths. Note missing expected files (e.g. no `AGENTS.md`) without inventing content unless the user asked for it.

### 2. Assess

Compare docs/rules/skills against the **actual** repo structure and each other. Record findings as concrete items:

| ID  | Severity     | Path(s) | Problem                 | Proposed fix |
| --- | ------------ | ------- | ----------------------- | ------------ |
| F1  | high/med/low | `path`  | short problem statement | short fix    |

Severity guide:

- **high** — contradiction, wrong paths, broken MCP/script refs that mislead agents
- **med** — unclear or stale docs that cause wrong defaults
- **low** — length/style/duplication that still works

### 3. Recommend

Stop here. Do not edit yet.

A one-line fix in a skill, rule, `AGENTS.md`, doc, or MCP config can still change how later agents behave, so there is no “small fix” exception.

Present:

1. **High-priority recommendations** — every **high** finding, with the proposed fix and why it matters. This is the default suggestion set.
2. **Other findings** — **med** and **low**, briefly, so they stay visible without being the proposed edit.

If nothing is high, say so. You may point at the strongest medium items as optional. Still do not edit.

Wait until the user names which items to apply. Delivering the recommendations is not approval.

### 4. Apply

Only after the user explicitly chooses items, or asks to apply a named set. Implement only those items. Prefer the smallest edit that removes the ambiguity:

- Split long markdown; keep a short entry file and link deeper detail one level deep
- Align structure docs with the real tree
- Resolve contradictions (pick the truth from the repo; update all dependents)
- Clarify vague instructions; tighten skill `description` WHAT + WHEN (third person)
- Fix or remove dead links and references in customization files
- Keep project voice and existing skill/rule conventions

Do not expand scope into product code “while you’re here.” Do not apply findings the user did not accept.

### 5. Verify

Skip this step when no files were edited. Otherwise re-check touched files:

- [ ] No remaining contradictions among edited docs/rules/skills
- [ ] Paths and referenced names in customization files resolve
- [ ] Markdown splits have working relative links
- [ ] Skill frontmatter still valid (`name`, `description`)
- [ ] Summary of what changed and what was deferred

## Output format

After assessment, before any edit, report:

1. **Summary** — one short paragraph
2. **High-priority recommendations** — the **high** findings (or “none”)
3. **Other findings** — **med** / **low** (or “none”)
4. **Changes made** — none until the user chooses items

After apply, report:

1. **Summary** — one short paragraph
2. **Changes made** — files touched
3. **Deferred** — findings not applied, and why

## Anti-patterns

- Editing customization files before the user confirms which findings to apply, including one-line or “obvious” fixes
- Rewriting the whole docs tree when a few path fixes suffice
- Adding new skills/rules unrelated to found issues
- “Improving” wording so much that project conventions are lost
- Leaving two sources of truth after resolving a contradiction
- Scope creep into application source or `scripts/**` outside customization paths
