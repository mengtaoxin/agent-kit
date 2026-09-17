# agent-kit

Reusable [Agent Skills](https://agentskills.io/) for coding agents. Install into any project with the [Skills CLI](https://skills.sh/).

## Skills

| Skill | Description |
| --- | --- |
| [`fix`](./skills/fix/) | Assess and improve agent customization (skills, rules, MCP, AGENTS.md, docs, scripts) |
| [`test-driven-development`](./skills/test-driven-development/) | Mandatory Red→Green→Refactor workflow when changing behavior |

## Install

From another project:

```bash
npx skills add mengtaoxin/agent-kit
```

Install one skill only:

```bash
npx skills add mengtaoxin/agent-kit --skill test-driven-development
```

Install globally (user-level):

```bash
npx skills add mengtaoxin/agent-kit -g
```

List skills in this repo without installing:

```bash
npx skills add mengtaoxin/agent-kit -l
```

## Layout

```text
skills/
  fix/
    SKILL.md
  test-driven-development/
    SKILL.md
```

Each skill is a directory with a `SKILL.md` (YAML frontmatter + instructions). Agents discover and apply skills when the task matches the skill description.

## License

MIT
