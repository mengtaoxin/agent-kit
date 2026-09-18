# agent-kit

Reusable [Agent Skills](https://agentskills.io/) pack. Skills here are installed into other projects (via [skills.sh](https://skills.sh/)); this repo is not an application.

## Rules

- **Skills are general-purpose.** Every skill must be useful across many projects. Do not encode a single app’s paths, domain names, product workflows, repo layout, or team-only conventions.
- **No project-specific coupling.** Prefer discovering host-project conventions at use time (`package.json`, existing tests, docs, `AGENTS.md` in the *target* repo). Do not hardcode another repository’s structure or commands.
- **Prefer stack/domain guidance over product guidance.** Technology and workflow skills (e.g. React, Fastify, MySQL, TDD) are in scope; one product’s feature playbook is not.
- **Discover, then apply.** When a skill conflicts with the host project’s conventions, prefer the host project and discover those conventions first.
- **Keep skills installable.** Each skill lives under `skills/<name>/SKILL.md` with valid frontmatter (`name`, `description`). Keep `description` third-person with clear WHAT + WHEN triggers.
- **Stay concise.** Prefer short `SKILL.md` files; put deep detail in linked sibling files only when needed (one level deep).
- **Keep README in sync.** When adding, renaming, or removing a skill, update the Skills table and Layout tree in `README.md`.

## Layout

```text
skills/
  <skill-name>/
    SKILL.md
```

See `README.md` for the current skill list and install commands.

## Out of scope for this repo

- Application / product source code
- Project-only agent rules meant for a single consumer repo
- Secrets, credentials, or environment-specific deploy steps
