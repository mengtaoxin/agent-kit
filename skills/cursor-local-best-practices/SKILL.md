---
name: cursor-local-best-practices
description: >-
  Local Cursor port and session hygiene. Free occupied ports on EADDRINUSE,
  close ports this chat opened before ending, and only check concurrent chats
  when starting or stopping servers. Use when hitting EADDRINUSE / port-in-use,
  starting or stopping a local server, finishing a conversation that opened
  ports, or when the user asks for local session / port hygiene — not for
  ordinary edits, Q&A, or docs-only work.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.1.0"
---

# Cursor local best practices

Apply when the agent runs in a **local** Cursor environment (same machine and
workspace as the user). Goal: leave ports free for the next chat or the user’s
own debugging, without blocking ordinary parallel chats.

## When to apply

Apply this skill only when one of these is true:

- Binding or freeing a **port** (dev server, preview, proxy, tunnel)
- Hitting **`EADDRINUSE`** / “port already in use”
- **Ending** a conversation that opened ports in this chat
- The user **explicitly** asks for local session / port hygiene

Do **not** apply for ordinary code edits, reviews, Q&A, docs, or tests that do
not start a long-lived listener. Parallel chats in the same repo are allowed
for that work.

## Rules

1. **Before starting or stopping a server** (or other long-lived port binder),
   check whether another in-repo chat is already running a server that would
   collide. If so, coordinate — do not blindly start a second listener on the
   same port.
2. **During the conversation**, if a port this chat needs is occupied, free it
   carefully (see below); prefer not to kill the user’s intentional servers.
3. **Before the conversation ends**, close ports that **this chat** opened,
   unless the user asked to leave them running.

## Before starting a server — light concurrency check

Do this only when about to start a **dev server, preview, proxy, or other
long-lived port binder** — not before every edit.

1. Identify the current workspace / repo root.
2. Check project `terminals` metadata (and similar) for an **active** command
   in this repo that already holds the target port or the same kind of server.
   Ignore idle/historical terminals and finished chats.
3. If the port / server is free → proceed.
4. If another chat (or the user) already holds it:
   - Tell the user about the conflict.
   - Prefer reusing that server, or ask whether to stop it — do **not** enter a
     multi-minute wait loop, and do **not** abort the whole conversation for
     unrelated edits.
5. Never block ordinary (non-server) work because another chat is active.

## During the conversation — occupied ports

When binding a port fails (e.g. `EADDRINUSE`, “port already in use”) or a needed
port is already taken:

1. Find the process listening on that port (`lsof`, `ss`, `netstat`, or OS
   equivalent).
2. If the holder was **started by this chat** → stop it, then retry.
3. If the holder looks like the **user’s** or **another chat’s** intentional
   server → tell the user; stop it only if they confirm, or if they already
   asked you to free that port. Prefer a graceful stop when practical.
4. Do not leave the conflict unresolved and switch to a random alternate port
   unless the user or project config requires a different port.

Track every port **this conversation** successfully opens (dev servers,
preview, proxies, one-off listeners).

## Before ending — close ports this chat opened

Before the final reply / handoff for this conversation (when this chat opened
ports, or the user asked for cleanup):

1. Stop processes this chat started that still hold ports (dev servers,
   watchers, tunnels, etc.).
2. Confirm those ports are free.
3. Do **not** kill unrelated user processes or servers started outside this
   chat, unless the user confirmed freeing that port.

If the user asked to leave a server running, say so and leave it; otherwise
default to closing ports opened here.
