---
name: swift-best-practices
description: >-
  Use when writing or reviewing Swift: API shape, value and reference types,
  optionals, errors, access control, and concurrency. Triggers on .swift edits,
  Swift packages or app targets, API design, or when the user mentions Swift
  best practices. Does not prescribe UI architecture (SwiftUI, UIKit) or a
  concurrency migration.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
  docs: https://www.swift.org/documentation/api-design-guidelines/
---

# Swift best practices

Apply these practices when writing or changing Swift. Prefer project conventions when they conflict; discover them first (language mode, `Package.swift` or the Xcode target, formatter, existing types).

Official naming guidance: [API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/).

## Defaults

1. Match the target's **Swift language mode** and existing concurrency style. Do not switch a target to Swift 6, or rewrite callbacks, Combine, or GCD to `async`, unless that change is the task.
2. Prefer **clarity at the call site** over brevity. Names should read as a phrase where the call is written.
3. Prefer **small types** with one clear role. Do not introduce a new architecture.
4. Prefer the **narrowest access** that callers actually need.
5. Do not invent style rules that fight the repo's formatter.

## API shape

Follow the API Design Guidelines for new names:

- Omit words that repeat the type or parameter. Keep words that prevent ambiguity (`remove(at:)`).
- Name side effects with an imperative verb (`sort()`). Name the nonmutating counterpart with a past participle or noun (`sorted()`).
- Factory methods start with `make`. An initializer's first argument does not repeat the type name.
- Booleans and predicates read as assertions (`isEmpty`, `hasPrefix`).
- Protocols that name a kind of thing are nouns (`Collection`). Capability protocols use `-able`, `-ible`, or `-ing` (`Equatable`).
- Document every **public** declaration. Skip comments that restate the signature.
- Prefer a defaulted parameter over a near-duplicate overload. Do not overload only on return type.

```swift
view.remove(cancelButton)
let ordered = names.sorted()
names.sort()
```

## Types

- Prefer a **struct** when the value has no identity. Use a **class** when identity, inheritance, or an existing reference-type API requires it.
- Prefer `let`. Use `var` only when the value changes.
- Prefer `enum` for a closed set of cases. Do not use sentinel strings or magic numbers for that set.
- Prefer standard-library protocols (`Equatable`, `Hashable`, `Codable`, `Comparable`) when the project already models the type that way. Synthesize them; hand-write only when synthesis is wrong.
- Do not add a protocol, generic, or wrapper that has a single implementation.

## Optionals and errors

- Prefer `if let`, `guard let`, `?.`, and `??`. Avoid force unwrap (`!`), `try!`, and `as!` unless the invariant is local and obvious.
- Reserve implicitly unwrapped optionals for framework lifecycles that require them. Do not add new ones for convenience.
- Use `throw` for recoverable failures. Use `precondition` / `fatalError` only for programmer errors.
- Use `try?` when failure means "absent" and the error is intentionally discarded. Do not use an empty `catch` to swallow a failure.
- Prefer the project's error type. Do not introduce a new error-handling style for one call site.

## Concurrency

Apply this section to code that already uses Swift concurrency, or to new async code the task asks for.

- Do not block a thread waiting on a task.
- Prefer structured concurrency (`async let`, task groups) over an unstructured `Task` when the work shares the caller's lifetime. Store a `Task` only when the work must outlive the caller, and cancel it when that owner goes away.
- Cross isolation boundaries with `Sendable` values or an actor. Do not mark types `@unchecked Sendable` to silence a diagnostic.
- Respect the target's actor isolation, including a default isolation the build already sets. Do not annotate a whole file `@MainActor` unless neighboring types do.
- Resume a checked continuation **exactly once**.
- Leave `@preconcurrency` for compatibility edges the project already uses. Do not add it to hide a new data race.

## Access, modules, and memory

- Keep types and helpers at `private` or `fileprivate` until another file needs them. Use `public`, `package`, or `open` only on the intended API surface.
- For packages, prefer the project's import style. When adding a dependency, keep it off the public surface unless clients must see it.
- Break reference cycles in classes: use a capture list (`[weak self]`, or `unowned` when the lifetime is guaranteed) for closures the instance stores. Do not add `[weak self]` to every closure.

## Tooling

- If the project formats with `swift-format`, SwiftFormat, or Xcode, follow that and format the files you touch. Do not add a second formatter.
- Build and test with the project's command (`swift test`, `swift build`, or the repo's script). After non-trivial changes, run the smallest check that covers them.

## Anti-patterns

- Force unwraps, `try!`, and `as!` to get past the compiler.
- `@unchecked Sendable`, `@preconcurrency`, or a language-mode downgrade to silence a race in new code.
- Rewriting unrelated callback or Combine code to `async` in the same change.
- Public API with no documentation comment, or comments that only repeat the name.
- A new protocol or generic with one conformer.
- Stringly-typed identifiers where the module already has a dedicated type.

## Agent checklist

Before finishing Swift work:

1. New names read clearly at the call site and match neighboring APIs.
2. No new force unwrap, `try!`, or `as!` without a local invariant.
3. Language mode and concurrency style match the target; no drive-by migration.
4. Access is no wider than callers need. New public declarations are documented.
5. Stored closures do not retain cycles.
6. Formatter and the project's build or tests are clean for the files you touched.
