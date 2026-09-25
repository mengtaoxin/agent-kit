---
name: material-design-best-practices
description: >-
  Use when designing or building Material Design 3 (Material You / MD3) UI:
  color roles, typography, layout, elevation, motion, components, theming,
  dark mode, and accessibility. Triggers on MUI, Material Web, Jetpack Compose
  Material3, Flutter Material, or when the user mentions Material Design,
  Material 3, Material You, or MD3. Do not use for generic UI without Material.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.1.0"
  docs: https://m3.material.io/
---

# Material Design best practices

Apply these practices when designing or implementing Material Design UI. Prefer **Material Design 3 (Material You / MD3)** unless the project is locked to Material 2. Prefer project conventions when they conflict; discover them first (theme tokens, MUI/Compose/Flutter theme, existing components).

Official guidance: [m3.material.io](https://m3.material.io/).

Stack-specific notes and token examples: [implementation by stack](./references/implementation.md).

## Defaults

1. Design with **roles and tokens**, not hard-coded hex values or one-off shadows.
2. Prefer **filled / tonal / outlined / text** component variants from the system over custom chrome.
3. Keep **layout and spacing on an 8dp grid** (4dp for tight icon/text alignment).
4. Treat **accessibility** (contrast, touch targets, focus, motion) as part of the design, not polish.
5. Match the platform library already in use (MUI, Material Web, Compose Material3, Flutter Material) instead of inventing parallel patterns.

## Color & theming

- Build from a **seed / brand color** into a full tonal palette when the stack supports dynamic color.
- Use **color roles** (`primary`, `on-primary`, `primary-container`, `surface`, `on-surface`, etc.) — never put raw brand hex on text or icons without an `on-*` pair.
- Prefer **surface containers** for layered backgrounds instead of stacking semi-transparent black/white.
- Support **light and dark** schemes from the same roles; do not invert colors by hand.
- Limit accent usage: one primary emphasis path per view; secondary/tertiary for supporting accents only.
- Prefer theme tokens / CSS variables / Compose `ColorScheme` / Flutter `ColorScheme` over scattered literals.

## Typography

- Use the MD3 **type scale** roles: `display`, `headline`, `title`, `body`, `label` (large/medium/small).
- Prefer **one typeface family** (or the platform default) with weight/size via the scale.
- Keep line length readable (~40–60 characters for body on large screens).
- Do not encode meaning with color alone; pair with type weight/size or icons.

## Layout, spacing & shape

- Prefer **8dp spacing rhythm**; use 4dp only for fine alignment.
- Follow responsive **window size classes**: compact, medium, expanded — adapt navigation (bar → rail → drawer).
- Prefer **canonical layouts** (list-detail, feed, supporting pane) when they fit.
- Use MD3 **shape scale**; keep shape family consistent.
- Prefer padding and inset over decorative dividers; use `outline-variant` sparingly.

## Elevation, surfaces & state

- Prefer **tonal surface elevation** over heavy drop shadows (MD3 default).
- Use elevation/shadow when the platform still expects it (sheets, menus), but keep levels few.
- Interactive components need clear **state layers** (hover, focus, pressed, dragged, disabled).
- Disabled content must remain recognizable — do not rely only on low-contrast gray.

## Components

- Prefer **standard Material components** before custom widgets.
- **Buttons**: filled = primary; tonal/outlined = secondary; text = low emphasis; FAB = one create/navigate action per screen.
- Prefer **one primary action** per view region; avoid competing filled buttons.
- **Text fields**: always show labels; use supporting/error text, not placeholder-as-label.
- **Dialogs** for critical decisions; **bottom sheets** for optional tasks; **snackbars** for brief confirmations (not durable errors).
- **Cards** for contained actionable groups — not default page chrome.
- Navigation: keep destinations stable; label icons in compact nav when space allows.

## Motion

- Prefer **MD3 motion** tokens over arbitrary CSS bounce.
- Motion should clarify hierarchy or spatial continuity; skip decoration-only animation.
- Respect **reduced motion**: provide instant or fade-only alternatives.
- Keep durations short for micro-interactions; longer only for large layout transitions.

## Accessibility

- Meet contrast for text and essential icons (WCAG AA as a floor).
- Touch targets ≥ **48×48dp** (or platform minimum), with adequate spacing.
- Every interactive control must be **keyboard reachable** with a visible focus indicator.
- Prefer semantic structure before ARIA.
- Do not convey state by color alone.
- Announce dynamic updates the way the platform screen reader expects.

## Anti-patterns (do not)

- Hard-coded brand colors on text/icons without `on-*` / contrast-safe pairs.
- Mixing Material 2 elevation and MD3 tonal surfaces inconsistently in the same flow.
- Multiple filled primary buttons competing on one screen.
- Placeholder-only fields with no visible label.
- Cards wrapping every section “because UI”.
- Custom components that duplicate Material ones with worse a11y/keyboard support.
- Motion that ignores reduced-motion preferences.
- Touch targets under ~48dp or tightly packed icon-only controls without labels/tooltips.

## Agent checklist

Before finishing Material Design UI work:

1. Colors use roles/tokens; light and dark still work.
2. Type uses the type scale roles, not one-off sizes.
3. Spacing follows 8dp rhythm; layout adapts across widths when required.
4. Components use standard variants and one clear primary action.
5. States (hover/focus/pressed/disabled) are visible and contrast-safe.
6. A11y: targets, focus, labels, contrast, reduced motion.
7. Matches the project’s Material library and existing theme — no parallel design system.
