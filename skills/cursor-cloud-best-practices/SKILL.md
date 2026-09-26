---
name: cursor-cloud-best-practices
description: >-
  Cursor Cloud Agent hygiene for demos and computer use. Avoid using computer
  use for screen recording. Use when running as a Cursor Cloud Agent / in a
  cloud VM, producing walkthrough or demo artifacts, or when considering
  computer use / RecordScreen — not for local Cursor chats.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Cursor cloud best practices

Apply when the agent runs in a **Cursor Cloud** environment (Cloud Agent /
isolated cloud VM). Do **not** apply in local Cursor chats (see
`cursor-local-best-practices` for local port hygiene).

## When to apply

Apply this skill only when one of these is true:

- The run is a **Cloud Agent** / cloud VM session
- Producing **walkthrough** / demo **artifacts** (video, screenshots)
- Considering **computer use** or screen recording (`RecordScreen`, etc.)

## Rules

1. **Avoid using computer use to record the screen.** Do not drive the desktop
   via computer use as part of a screen-recording / walkthrough capture loop.
   Prefer screenshots, logs, terminal output, or screen recording **without**
   computer-use click-through. Pairing RecordScreen with computer use often
   hangs the run and surfaces as “stuck on recording.”
