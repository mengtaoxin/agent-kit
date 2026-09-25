---
name: react-best-practices
description: >-
  Apply modern React best practices when writing or reviewing React code.
  Covers components, hooks, state, effects, composition, performance, and
  accessibility. Use when editing .tsx/.jsx files, building UI, refactoring
  React components, fixing hooks/effects bugs, or when the user mentions
  React, hooks, JSX, or React best practices.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.1.0"
---

# React best practices

Apply these practices when writing or changing React code. Prefer project conventions when they conflict; discover them first (`package.json`, ESLint/Biome React plugins, React Compiler config, existing components).

## Defaults

1. Prefer **function components** and hooks. Do not introduce class components unless the codebase already requires them.
2. Keep components **small and focused** — one clear responsibility per component.
3. Prefer **composition** over inheritance and over deep prop drilling when context or a shared hook fits better.
4. Colocate state with the nearest consumer; lift state only when siblings must share it.
5. Match the project's React version APIs and patterns (Compiler, Server Components, router, data library).

## Components & JSX

- Prefer named function declarations or `const` function components consistent with the file/repo.
- Keep render paths pure: same props/state → same UI. Side effects belong in effects, event handlers, or framework data APIs.
- Prefer early returns for loading/error/empty states over deeply nested ternaries.
- Do not put heavy logic inline in JSX; extract helpers or custom hooks.
- Prefer children and render slots for flexible composition instead of boolean prop matrices (`showX`, `isY`, `variantZ` piles).
- Keys: use stable ids. Never use array index as key when the list can reorder, insert, or filter.

```tsx
// Prefer derived UI from data, not boolean prop piles
function UserCard({ user }: { user: User }) {
  if (!user) return null;
  return <article>{user.name}</article>;
}
```

## State

- Prefer **derived values** over storing the same information twice. If it can be computed from props/state, do not mirror it in `useState`.
- Prefer updating state with a functional updater when the next value depends on the previous (`setX(prev => ...)`).
- Group related state; prefer one object/reducer when fields always change together.
- Prefer controlled inputs when the parent must own the value; uncontrolled (`ref` / defaultValue) for simple forms that do not need live sync.
- Do not sync props into state with `useEffect` unless intentionally capturing an initial value; prefer fully controlled or a `key` remount.

```tsx
// Prefer derived
const fullName = `${user.first} ${user.last}`;

// Avoid
// const [fullName, setFullName] = useState("");
// useEffect(() => setFullName(`${user.first} ${user.last}`), [user]);
```

## Hooks

- Treat the **Rules of Hooks** as hard constraints (top level only; same order every render). Do not disable `exhaustive-deps` casually.
- Prefer custom hooks to share stateful logic; name them `useXxx`.
- Keep hooks focused; a hook that manages five unrelated concerns should be split.
- Prefer `useReducer` when state transitions are complex or next state depends on previous in multiple ways.

## Effects

- Use `useEffect` for **synchronizing with external systems** (DOM APIs, subscriptions, non-React widgets, imperative libraries) — not for deriving state or chaining “do this after that” app logic.
- Prefer calculating during render, event handlers, or data-fetching libraries over effect-driven data flow.
- Always clean up subscriptions, timers, and listeners in the effect return.
- List the real dependencies; fix the design instead of empty dep arrays or disabled lint rules.
- Prefer `useEffectEvent` (when available in the project) for event-like logic inside effects that should not re-subscribe on every render.

```tsx
useEffect(() => {
  const id = setInterval(onTick, 1000);
  return () => clearInterval(id);
}, [onTick]);
```

## Events & transitions

- Put user-triggered side effects in **event handlers**, not effects.
- Prefer `startTransition` for non-urgent updates (list filtering, tab switches) when the project already uses concurrent patterns.
- Prefer `useDeferredValue` for expensive UI that can lag behind urgent input when appropriate and already used in the team’s patterns.
- Prefer functional, declarative updates over imperative DOM manipulation.

## Performance

- **Do not** add `useMemo` / `useCallback` / `React.memo` by default. Follow the project's **React Compiler** guidance when enabled; otherwise memoize only with a measured or obvious expensive re-render problem.
- Prefer fixing state location and component splits before memoization.
- Avoid creating new object/array identities in hot paths only when they are genuine deps of children that rely on reference equality (rare if Compiler is on).
- Prefer `lazy` + `Suspense` for large, rarely used routes/panels when code-splitting is already a project pattern.
- Prefer virtualization for very long lists when the project already uses a list library.

## Lists, forms, and data

- Prefer stable keys and predictable list identity.
- Prefer controlled forms with a clear single source of truth, or the project's form library (React Hook Form, Conform, etc.).
- Prefer schemas already used in the project (Zod, etc.) for validation at boundaries.
- Do not fetch in random child effects if the app has a data library (React Query, SWR, router loaders, Server Components). Match existing data patterns.

## Server Components & client boundaries (when applicable)

- Default to Server Components in frameworks that support them; add `"use client"` only when browser APIs, state, or effects are required.
- Keep client leaves small; pass serializable props across the server/client boundary.
- Do not import server-only modules into client components.

## Accessibility

- Use semantic HTML (`button`, `label`, `nav`, headings) before ARIA.
- Prefer native controls; add ARIA only when HTML cannot express the role.
- Every interactive control must be keyboard reachable and operable.
- Images need meaningful `alt` (or empty `alt` if decorative).
- Do not rely on color alone for meaning; preserve focus visibility.

## Error & loading UX

- Prefer Error Boundaries (or framework error UI) for render failures; do not catch render errors in event handlers alone.
- Prefer explicit loading and empty states near the data source.
- Prefer Suspense boundaries where the project already structures async UI that way.

## Anti-patterns (do not)

- Derived state mirrored into `useState` + syncing `useEffect`.
- Effects that only set state from props/state (re-render loops waiting to happen).
- Prop drilling through many layers when a hook or context is clearer.
- Giant components (hundreds of lines) with unrelated UI and data logic.
- Disabling `react-hooks/exhaustive-deps` to silence a real dependency bug.
- Blanket `useMemo`/`useCallback` “for performance” without evidence or Compiler guidance.
- Index keys for dynamic lists; `Math.random()` keys.
- Nested ternaries that obscure loading/error/success UI.

## Agent checklist

Before finishing React work:

1. Components are pure where expected; effects only sync to external systems.
2. No redundant state; derived values are computed, not synced.
3. Hooks follow the rules; dependencies are honest.
4. A11y basics for new interactive UI (semantics, labels, keyboard).
5. No default memoization spam; matches project Compiler / performance norms.
6. Lints for touched files are clean.

Behavior changes: follow the project's test / TDD skill or docs for writing and running tests — this skill does not define that workflow.
