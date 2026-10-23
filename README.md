# agent-kit

Reusable [Agent Skills](https://agentskills.io/) for coding agents. Install into any project with the [Skills CLI](https://skills.sh/).

## Skills

| Skill | Description |
| --- | --- |
| [`cursor-cloud-best-practices`](./skills/cursor-cloud-best-practices/) | Cursor Cloud Agent hygiene: avoid using computer use for screen recording |
| [`cursor-local-best-practices`](./skills/cursor-local-best-practices/) | Local Cursor port hygiene: free EADDRINUSE carefully, close ports this chat opened; concurrency check only when starting/stopping servers; leave commits and other chats' changes alone |
| [`electron-best-practices`](./skills/electron-best-practices/) | Electron process isolation (main / preload / renderer / shared) and IPC surface checklist |
| [`javascript-typescript-best-practices`](./skills/javascript-typescript-best-practices/) | Modern JavaScript and TypeScript best practices for writing and reviewing JS/TS |
| [`material-design-best-practices`](./skills/material-design-best-practices/) | Material Design 3 best practices for color, type, layout, components, motion, and a11y |
| [`mysql-best-practices`](./skills/mysql-best-practices/) | MySQL best practices for schemas, indexes, queries, transactions, and secure access |
| [`optimize-agent-customization`](./skills/optimize-agent-customization/) | Audit agent customization (skills, rules, MCP, AGENTS.md, docs) and recommend high-priority fixes before editing |
| [`react-best-practices`](./skills/react-best-practices/) | Browser-rendered React best practices for components, hooks, state, effects, and a11y; not for SSR stacks such as Next.js |
| [`save-web-page`](./skills/save-web-page/) | Save a URL as one offline HTML file (inlined CSS + base64 images) via the browser |
| [`task-divide-and-conquer`](./skills/task-divide-and-conquer/) | Split a big task into serial subtasks tracked in a markdown file; run one subtask per sub-agent (or per conversation if sub-agents are unavailable), end by offering a commit message |
| [`task-issue-queue`](./skills/task-issue-queue/) | File the current request as a GitHub issue labeled task-issue-queue, or drain that queue serially: one local commit per solved issue, no push; roll a failed attempt back to its baseline and continue |
| [`test-driven-development`](./skills/test-driven-development/) | Red→Green→Refactor for product behavior; not for declarative toolchain config or mechanical edits |

## Install

From another project:

```bash
npx skills add mengtaoxin/agent-kit
```

Install one skill only:

```bash
npx skills add mengtaoxin/agent-kit --skill react-best-practices
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
  cursor-cloud-best-practices/
    SKILL.md
  cursor-local-best-practices/
    SKILL.md
  electron-best-practices/
    SKILL.md
  javascript-typescript-best-practices/
    SKILL.md
  material-design-best-practices/
    SKILL.md
    references/
  mysql-best-practices/
    SKILL.md
  optimize-agent-customization/
    SKILL.md
  react-best-practices/
    SKILL.md
  save-web-page/
    SKILL.md
    scripts/
  task-divide-and-conquer/
    SKILL.md
  task-issue-queue/
    SKILL.md
  test-driven-development/
    SKILL.md
```

Each skill is a directory with a `SKILL.md` (YAML frontmatter + instructions). Agents discover and apply skills when the task matches the skill description.

## License

MIT
