---
name: cursor-local-best-practices
description: >-
  Local Cursor session hygiene for finishing work: do not inspect the working
  tree at the end unless asked; leave other chats' and the user's changes
  alone. Use when finishing a conversation that changed files, or when the
  user asks for local session hygiene. For port and long-lived process
  cleanup, use port-and-process-management.
license: MIT
metadata:
  author: mengtaoxin
  version: "2.1.0"
---

# Cursor local best practices

Apply when the agent runs in a **local** Cursor environment (same machine and
workspace as the user). Goal: leave the working tree alone for the user and
other chats.

Port and long-lived process cleanup lives in **`port-and-process-management`**
— install and apply that skill when starting servers or freeing ports.

## When to apply

Apply this skill only when one of these is true:

- **Finishing** a conversation that changed files
- The user **explicitly** asks for local session hygiene (working tree), not
  port cleanup

## Finishing — working tree

Other chats and the user often edit the same working tree in parallel. Unless
the user or the host project's workflow asks otherwise:

- Do not run `git status` / `git diff` to review the tree at the end, and do not
  inspect or comment on changes this chat did not make.
