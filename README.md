# agent-kit

Reusable [Agent Skills](https://agentskills.io/) for coding agents. Install into any project with the [Skills CLI](https://skills.sh/).

## Skills

| Skill | Description |
| --- | --- |
| [`cursor-cloud-best-practices`](./skills/cursor-cloud-best-practices/) | Cursor Cloud Agent hygiene: avoid using computer use for screen recording |
| [`cursor-local-best-practices`](./skills/cursor-local-best-practices/) | Local Cursor port hygiene: free EADDRINUSE carefully, close ports this chat opened; concurrency check only when starting/stopping servers; leave commits and other chats' changes alone |
| [`electron-best-practices`](./skills/electron-best-practices/) | Electron process isolation (main / preload / renderer / shared) and IPC surface checklist |
| [`issue-create`](./skills/issue-create/) | Create one open GitHub issue for the current request, with a label the user names; leave it open and do not implement it |
| [`issue-queue-process`](./skills/issue-queue-process/) | Drain open GitHub issues that carry a label the user names, one sub-agent at a time: one local commit per solved issue, no push; roll a failed attempt back to its baseline and continue |
| [`javascript-typescript-best-practices`](./skills/javascript-typescript-best-practices/) | Modern JavaScript and TypeScript best practices for writing and reviewing JS/TS |
| [`material-design-best-practices`](./skills/material-design-best-practices/) | Material Design 3 best practices for color, type, layout, components, motion, and a11y |
| [`mysql-best-practices`](./skills/mysql-best-practices/) | MySQL best practices for schemas, indexes, queries, transactions, and secure access |
| [`react-best-practices`](./skills/react-best-practices/) | Browser-rendered React best practices for components, hooks, state, effects, and a11y; not for SSR stacks such as Next.js |
| [`save-web-page`](./skills/save-web-page/) | Save a URL as one offline HTML file (inlined CSS + base64 images) via the browser |
| [`swift-best-practices`](./skills/swift-best-practices/) | Swift API shape, value and reference types, optionals, errors, access control, and concurrency; no UI architecture or concurrency migration |
| [`task-divide-and-conquer`](./skills/task-divide-and-conquer/) | Split a big task into serial subtasks tracked in a markdown file; run one subtask per sub-agent (or per conversation if sub-agents are unavailable), end by offering a commit message |
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
  issue-create/
    SKILL.md
  issue-queue-process/
    SKILL.md
  javascript-typescript-best-practices/
    SKILL.md
  material-design-best-practices/
    SKILL.md
    references/
  mysql-best-practices/
    SKILL.md
  react-best-practices/
    SKILL.md
  save-web-page/
    SKILL.md
    scripts/
  swift-best-practices/
    SKILL.md
  task-divide-and-conquer/
    SKILL.md
  test-driven-development/
    SKILL.md
```

Each skill is a directory with a `SKILL.md` (YAML frontmatter + instructions). Agents discover and apply skills when the task matches the skill description.

## License

MIT
