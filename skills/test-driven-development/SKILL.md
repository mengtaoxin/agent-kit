---
name: test-driven-development
description: >-
  Mandatory Red→Green→Refactor workflow for agents changing behavior. Use when
  implementing features, fixing bugs, adding regression guards, or any code
  change that alters observable behavior. Triggers on TDD, test-first,
  failing test, behavior change, or before claiming work is done.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
---

# Test-driven development (for agents)

Mandatory workflow when changing behavior. Follow this guide before and while editing code.

Goal: prove the intended behavior with a failing test first, then make that test pass with the smallest change. Do not claim work is done because the implementation “looks right.”

## Non-negotiable cycle

For every behavior change (feature, bug fix, regression guard):

1. **Red** — Write or extend a test that expresses the desired behavior. Run it. Confirm it fails for the right reason (missing behavior / wrong result), not because of a broken compile, wrong import, or flaky setup.
2. **Green** — Change production code only enough to make that test pass. No drive-by refactors in this step.
3. **Refactor** — Clean up with tests still green. Keep behavior fixed.

Do not skip Red. Writing implementation first and then a green test is not TDD here.

If you cannot write a meaningful failing test yet, stop and clarify the requirement. Do not invent production code without a behavior contract.

## What to test at which layer

Pick the lowest layer that can lock the behavior. Prefer one focused test over a pile of overlapping ones.

Discover project conventions first:

- Look for testing docs (`testing.md`, `docs/testing.md`, `AGENTS.md`, or similar).
- Copy style from a nearby existing test; match project naming and locations.

Rules of thumb:

- Prefer the lowest layer that can lock the contract; use end-to-end only for user-visible integration.
- Do not add a second test layer that only duplicates assertions already owned by a lower test, unless you need an integration smoke.
- Prefer real collaborators over mocks when a lower-layer test can express the contract cleanly.

## How to run tests

Discover how the project runs tests (`package.json` scripts, `Makefile`, `commands.md`, CI config, or `AGENTS.md`). Run the relevant suite(s) on each Red/Green step. Before finishing, always run the full test suite; do not mark the task done until it passes. Focused tests alone are not enough to finish.

## Agent checklist (every behavior change)

1. If adding or moving files, follow the project’s file-structure / placement conventions.
2. State the behavior in one sentence (observable outcome, not implementation).
3. Add/adjust the failing test at the right layer; run it; confirm **Red**.
4. Implement the minimal fix; run the same test; confirm **Green**.
5. Refactor if needed; keep tests green.
6. Update project docs only when project-specific contracts or documented behavior change.
7. Run the full test suite. Do not mark the task done until it passes.

## Anti-patterns (do not)

- Implement first, then write a test that mirrors the code you already wrote.
- Change the test to match buggy production behavior without an explicit product decision.
- Broad “rewrite all tests” or unrelated snapshot churn.
- Skip the preferred lower-layer coverage because mocks “should be enough.”
- Use end-to-end tests as a substitute for a unit test of a pure helper.
- Leave tests failing, or leave Red unobserved (never ran the failing test).
- Claim done after only focused tests; skip the final full suite.

## When a thin exception applies

TDD still applies to behavior. These are not free passes to skip tests:

- **Docs / comments / pure formatting** with no behavior change — no new test required; full suite not required.
- **Generated or mechanical moves** (rename with tool, import reorder) — no new test if behavior is unchanged; still run the full suite before finishing.
- **Spike / exploration** — throwaway code may skip TDD, but delete or redo with Red→Green before keeping it (including the final full suite).

If a bug has no reliable automated hook yet, add the smallest test that would have failed before the fix, then fix it.
