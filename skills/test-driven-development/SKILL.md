---
name: test-driven-development
description: >-
  Use Red→Green→Refactor for product behavior changes (features, bug fixes,
  domain/API logic, regression guards). Not for declarative toolchain config,
  docs, or mechanical edits. Use when the user mentions TDD, test-first,
  failing test, feature, bug fix, or behavior change in app code.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.2.0"
---

# Test-driven development (for agents)

Use Red→Green→Refactor when changing **product behavior** owned by this repo.
Do not invent ceremonial tests for config flags or mechanical edits.

Goal: prove intended product behavior with a failing test first, then make that
test pass with the smallest change. Do not claim work is done because the
implementation “looks right.”

## When TDD is required

Apply the full Red→Green→Refactor cycle for:

- Features and bug fixes in app code (application / library source, not tooling-only)
- Domain rules, mapping, validation, pure helpers with non-trivial logic
- HTTP / persistence / CRUD contracts
- Regression guards for bugs that have a meaningful automated hook

## When TDD is out of scope

Skip Red→Green (and do **not** add a test that only mirrors a literal you set)
for:

- **Docs / comments / pure formatting** — no new test; full suite not required
- **Generated or mechanical moves** (rename with tool, import reorder) — no new
  test if behavior is unchanged; still run the full suite before finishing when
  you touched code
- **Declarative toolchain / build / dev-server config** that only flips a
  framework or bundler option (e.g. Vite `strictPort`, ESLint rule, `tsconfig`
  flag, Playwright CLI already covering the same switch) — change the config;
  do not write a unit test that asserts `config.foo === true`
- **Dependency bumps** with no project logic change — rely on existing suites
- **Spike / exploration** — throwaway code may skip TDD, but delete or redo with
  Red→Green before keeping it (including the final full suite)

If unsure: TDD applies when the repo owns **logic or contracts**; it does not
apply when you are only turning on an upstream tool’s built-in switch.

## Non-negotiable cycle (when in scope)

1. **Red** — Write or extend a test that expresses the desired behavior. Run it.
   Confirm it fails for the right reason (missing behavior / wrong result), not
   because of a broken compile, wrong import, or flaky setup.
2. **Green** — Change production code only enough to make that test pass. No
   drive-by refactors in this step.
3. **Refactor** — Clean up with tests still green. Keep behavior fixed.

Do not skip Red when TDD is in scope. Writing implementation first and then a
green test is not TDD here.

If you cannot write a meaningful failing test yet, stop and clarify the
requirement. Do not invent production code without a behavior contract — and do
not invent a meaningless test just to satisfy this checklist.

## What to test at which layer

Pick the lowest layer that can lock the behavior. Prefer one focused test over a
pile of overlapping ones.

Discover project conventions first:

- Look for testing docs (`testing.md`, `docs/testing.md`, `AGENTS.md`, or similar).
- Copy style from a nearby existing test; match project naming and locations.

Rules of thumb:

- Prefer the lowest layer that can lock the contract; use end-to-end only for
  user-visible integration.
- Do not add a second test layer that only duplicates assertions already owned
  by a lower test, unless you need an integration smoke.
- Prefer real collaborators over mocks when a lower-layer test can express the
  contract cleanly.

## How to run tests

Discover how the project runs tests (`package.json` scripts, `Makefile`,
`commands.md`, CI config, or `AGENTS.md`).

- **When TDD is in scope:** run the relevant focused suite(s) on each Red/Green
  step. Before finishing, always run the full test suite; focused tests alone
  are not enough.
- **When TDD is out of scope:** still run the smallest relevant check if the
  change can break CI or local workflows (e.g. existing e2e/dev scripts). Do not
  add a new test solely to “cover” a config literal.

## Agent checklist

1. Decide **in scope / out of scope** using the sections above.
2. If adding or moving files, follow the project’s file-structure / placement
   conventions.
3. If **in scope:** state the behavior in one sentence; add the failing test;
   confirm **Red**; implement; confirm **Green**; refactor if needed.
4. If **out of scope:** make the minimal change; skip ceremonial tests.
5. Update project docs only when project-specific contracts or documented
   behavior change.
6. Before finishing: run the full suite when you changed product code, or the
   smallest relevant check for out-of-scope tooling/config. Do not mark the task
   done until those checks pass.

## Anti-patterns (do not)

- Implement first, then write a test that mirrors the code you already wrote.
- Write a test whose only assertion is that a config flag equals the value you
  just set.
- Change the test to match buggy production behavior without an explicit product
  decision.
- Broad “rewrite all tests” or unrelated snapshot churn.
- Skip the preferred lower-layer coverage because mocks “should be enough.”
- Use end-to-end tests as a substitute for a unit test of a pure helper.
- Leave tests failing, or leave Red unobserved (never ran the failing test)
  when TDD is in scope.
- Claim done after only focused tests; skip the final full suite when product
  code changed.
- Force Red→Green onto declarative toolchain config “because AGENTS.md says
  TDD.”
