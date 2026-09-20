---
name: cursor-local-best-practices
description: >-
  Local Cursor session hygiene for concurrent chats and ports. Before work,
  wait out other in-repo conversations (5 min × 5 checks) or abort. Free occupied
  ports by stopping the holder; before ending, close ports this chat opened.
  Use when running Cursor locally, starting agent work in a repo, hitting
  EADDRINUSE / port-in-use, starting or stopping dev servers, or finishing a
  local conversation.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Cursor local best practices

Apply when the agent runs in a **local** Cursor environment (same machine and
workspace as the user). Goal: avoid colliding with other chats in the same
repo and leave ports free for the next chat or the user’s own debugging.

## Rules

1. **Before starting work**, check whether the current repo has other
   conversations in progress. If so, wait for them to finish. After each check,
   wait 5 minutes before checking again. Check at most 5 times. If others are
   still not finished, terminate the current conversation.
2. **During the conversation**, if a port is occupied, stop the target process.
3. **Before the conversation ends**, close ports that were opened during the
   conversation, so they do not affect the next conversation or the user’s own
   debugging.

## Before starting work — concurrent conversations

Do this **before** edits, installs, servers, or other repo-mutating work.

1. Identify the current workspace / repo root.
2. Detect other in-progress conversations for **this same repo**, for example:
   - Other agent chats actively working in this workspace (not this chat).
   - Project `terminals` metadata showing an active command from another
     session (header fields such as `pid`, `cwd`, and a running command).
   - Ignore idle/historical terminals and finished chats.
3. If **no** other conversation is in progress → proceed.
4. If another conversation **is** in progress:
   - Tell the user you are waiting on the other chat.
   - Wait **5 minutes**, then re-check.
   - Repeat until clear, up to **5 checks total** (including the first).
5. After 5 checks, if another conversation is still in progress:
   - **Terminate this conversation**: stop all work, do not start servers or
     edits, and tell the user why. Do not continue on a later turn unless the
     user explicitly asks to retry after the other chat finishes.

Do not skip the wait loop to “save time.” Do not work in parallel on the same
repo while another chat is still active.

## During the conversation — occupied ports

When binding a port fails (e.g. `EADDRINUSE`, “port already in use”) or a needed
port is already taken:

1. Find the process listening on that port (`lsof`, `ss`, `netstat`, or OS
   equivalent).
2. **Stop the target process** (the one holding the port), then retry.
3. Prefer a graceful stop when practical; escalate only if it does not exit.
4. Do not leave the conflict unresolved and switch to a random alternate port
   unless the user or project config requires a different port.

Track every port **this conversation** successfully opens (dev servers,
preview, proxies, one-off listeners).

## Before ending — close ports this chat opened

Before the final reply / handoff for this conversation:

1. Stop processes this chat started that still hold ports (dev servers,
   watchers, tunnels, etc.).
2. Confirm those ports are free.
3. Do **not** kill unrelated user processes or servers started outside this
   chat, unless rule 2 required freeing a specific occupied port.

If the user asked to leave a server running, say so and leave it; otherwise
default to closing ports opened here.
