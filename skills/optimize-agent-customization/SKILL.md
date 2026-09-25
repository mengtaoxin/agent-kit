---
name: optimize-agent-customization
description: >-
  Audit and improve agent customization for the current project: skills,
  Cursor rules, MCP config, AGENTS.md, docs, and scripts. Finds oversized
  markdown, stale structure docs, contradictions, unclear guidance, and weak
  skill metadata; proposes a plan; then applies fixes. Use when the user asks
  to optimize agent customization, clean up agent docs/rules/skills, or audit
  AGENTS.md / MCP — not for application bug fixes or product features.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.1.0"
---

# Optimize agent customization

Optimize the current project's **agent customization** surface so agents get accurate, concise, non-contradictory guidance.

## Scope (in scope)

Only these paths (and close equivalents if the project uses a variant layout):

- `./agents/skills/**` (also `./skills/**`, `.cursor/skills/**`, `.agents/skills/**` when that is where skills live)
- `.cursor/rules/**`
- `.cursor/mcp.json`
- `AGENTS.md`
- `docs/**`
- `scripts/**`

Do **not** treat application source code, product features, or unrelated refactors as in scope unless they are required to keep a customization file truthful.

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
- [ ] 3. Plan (get confirmation if destructive/large)
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

### 3. Plan

Present the findings table and a short ordered plan (high → low). Prefer minimal edits that remove ambiguity.

Ask before large rewrites, mass deletes, or changing MCP servers the user may rely on. For small, clearly correct fixes (broken path, obvious contradiction), proceed after stating the plan.

### 4. Apply

Implement the agreed plan inside the scope:

- Split long markdown; keep a short entry file and link deeper detail one level deep
- Align structure docs with the real tree
- Resolve contradictions (pick the truth from the repo; update all dependents)
- Clarify vague instructions; tighten skill `description` WHAT + WHEN (third person)
- Fix or remove dead links/scripts/references
- Keep project voice and existing skill/rule conventions

Do not expand scope into product code “while you’re here.”

### 5. Verify

Re-check touched files:

- [ ] No remaining contradictions among edited docs/rules/skills
- [ ] Paths and script names resolve
- [ ] Markdown splits have working relative links
- [ ] Skill frontmatter still valid (`name`, `description`)
- [ ] Summary of what changed and what was deferred

## Output format

After assessment (and again after apply), report:

1. **Summary** — one short paragraph
2. **Findings** — the table above (or “none”)
3. **Changes made** — bullet list of files touched
4. **Deferred** — items skipped and why

## Anti-patterns

- Rewriting the whole docs tree when a few path fixes suffice
- Adding new skills/rules unrelated to found issues
- “Improving” wording so much that project conventions are lost
- Leaving two sources of truth after resolving a contradiction
- Scope creep into application source outside customization paths
