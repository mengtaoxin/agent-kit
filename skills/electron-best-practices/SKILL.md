---
name: electron-best-practices
description: >-
  Use when editing Electron process boundaries or IPC: main / preload /
  renderer / shared, contextBridge, channel constants, and ipcMain/ipcRenderer
  handlers. Triggers on Electron main/preload/renderer work, contextBridge APIs,
  or when the user mentions Electron, preload, IPC, contextBridge, or process
  boundaries.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.2.0"
  docs: https://www.electronjs.org/docs/latest/
---

# Electron best practices

Apply these practices when writing or changing Electron apps. Prefer host-project
conventions when they conflict; discover them first (`package.json`, existing
main/preload/renderer layout, IPC channel modules, `AGENTS.md` / docs in the
target repo).

Official docs: [electronjs.org/docs](https://www.electronjs.org/docs/latest/).

## Process boundaries

Keep process roles strict. Do not blur them for convenience.

| Process | Owns | Must not |
| --- | --- | --- |
| **Main** | App lifecycle, `BrowserWindow`, Node/OS APIs, native modules, privileged I/O, `ipcMain` handlers | UI components, React trees, renderer-only state libraries as the source of truth for privileged work |
| **Preload** | Thin `contextBridge` API; forward invokes/events using shared channel names/types | Business logic, DB access, direct Node filesystem beyond what the bridge must expose |
| **Renderer** | UI, client state, routing | Node built-ins (`fs`, `child_process`, …), native modules, importing main-process modules |
| **Shared** (optional but recommended) | Isomorphic DTOs, channel name constants, pure helpers safe on both sides | Electron main APIs, React, Node filesystem |

Hard rules:

1. **Renderer never imports main.** No direct imports from main-process trees into renderer, preload, or UI packages.
2. **Renderer talks to main only through preload.** Prefer `contextBridge.exposeInMainWorld` + `ipcRenderer.invoke` / subscriptions — not ad-hoc globals or enabling Node in the renderer.
3. **Preload stays thin.** Validate/forward; put real work in main modules called from `ipcMain` handlers.
4. **Shared is isomorphic only.** If a module needs Electron main or Node FS, it belongs in main — not in shared.
5. **Do not put UI in main.** Dialogs and OS integration stay in main; visual chrome stays in renderer.

Discover the host layout (common patterns: `src/main`, `src/preload`, `src/renderer`, or Forge/Vite multi-entry configs) and place new code in the matching process.

## IPC surface checklist

When **adding or changing** a cross-process API, update every layer in one change set. Skipping a layer leaves a half-wired surface.

Do these in order (adapt names to the host project):

1. **Channel constants** — Define invoke/event channel strings once in a shared module. Do not hardcode magic strings in main and preload separately.
2. **Types / DTOs** — Put request and response shapes in shared types (or an equivalent contract module).
3. **Main handlers** — Register `ipcMain.handle` / `ipcMain.on` in the main IPC module; implement privileged work in main services.
4. **Preload bridge** — Expose a typed method on the bridged API that invokes the same channel; subscribe to push events if needed and return an unsubscribe function.
5. **Renderer call sites** — Call only the bridged API (e.g. `window.<api>.…`), never `ipcRenderer` from UI code.

Push/events (main → renderer):

- Main sends on a dedicated channel (`webContents.send` / equivalent).
- Preload subscribes via `ipcRenderer.on` and exposes a small subscribe API to the renderer.
- Renderer registers through the bridge and cleans up on unmount.

Security defaults:

- Keep `contextIsolation: true` and `nodeIntegration: false` unless the project already documents an exception.
- Expose the minimum API surface on the bridge; do not mirror all of Node or Electron.
- Validate and authorize privileged inputs in **main**, not only in the UI.

## Anti-patterns

- Importing main-process modules from React/Vue/Svelte UI files.
- Using `ipcRenderer` directly from renderer code when a preload bridge exists.
- Duplicating channel string literals in main and preload.
- Growing preload into a second business layer (DB, multi-step workflows, heavy parsing).
- Putting React components or router screens under the main-process tree.
- Shipping only a main handler or only a preload method without the matching contract and call site.

## Agent checklist

Before finishing Electron IPC or process-boundary work:

1. New/changed APIs update channel constants, shared types, main handlers, preload bridge, and renderer call sites together.
2. Renderer never imports main; UI talks to main only through the preload bridge.
3. Preload stays thin; privileged work lives in main.
4. Shared modules stay isomorphic (no Electron main / Node FS).
5. `contextIsolation: true` and `nodeIntegration: false` unless the project documents an exception.
6. Privileged inputs are validated/authorized in main.
