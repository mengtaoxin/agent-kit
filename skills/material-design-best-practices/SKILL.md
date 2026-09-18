---
name: material-design-best-practices
description: >-
  Apply Material Design 3 (Material You) best practices when designing or
  building UI. Covers color roles, typography, layout, elevation, motion,
  components, theming, dark mode, and accessibility. Use when implementing
  Material UI (MUI), Material Web, Jetpack Compose Material3, Flutter Material,
  or when the user mentions Material Design, Material 3, Material You, MD3,
  or Material Design best practices.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.0.0"
  docs: https://m3.material.io/
---

# Material Design best practices

Apply these practices when designing or implementing Material Design UI. Prefer **Material Design 3 (Material You / MD3)** unless the project is locked to Material 2. Prefer project conventions when they conflict; discover them first (theme tokens, MUI/Compose/Flutter theme, existing components).

Official guidance: [m3.material.io](https://m3.material.io/).

## Defaults

1. Design with **roles and tokens**, not hard-coded hex values or one-off shadows.
2. Prefer **filled / tonal / outlined / text** component variants from the system over custom chrome.
3. Keep **layout and spacing on an 8dp grid** (4dp for tight icon/text alignment).
4. Treat **accessibility** (contrast, touch targets, focus, motion) as part of the design, not polish.
5. Match the platform library already in use (MUI, Material Web, Compose Material3, Flutter Material) instead of inventing parallel patterns.

## Color & theming

- Build from a **seed / brand color** into a full tonal palette (primary, secondary, tertiary, error, neutral, neutral-variant) when the stack supports dynamic color.
- Use **color roles** (`primary`, `on-primary`, `primary-container`, `on-primary-container`, `surface`, `on-surface`, `surface-variant`, `outline`, etc.) — never put raw brand hex on text or icons without an `on-*` pair.
- Prefer **surface containers** (`surface-container-lowest` … `highest`) for layered backgrounds instead of stacking semi-transparent black/white.
- Support **light and dark** schemes from the same roles; do not invert colors by hand.
- Limit accent usage: one primary emphasis path per view; secondary/tertiary for supporting accents only.
- Prefer theme tokens / CSS variables / Compose `ColorScheme` / Flutter `ColorScheme` over scattered literals.

```css
/* Prefer roles */
.button-filled {
  background: var(--md-sys-color-primary);
  color: var(--md-sys-color-on-primary);
}

/* Avoid */
.button-filled {
  background: #6750a4;
  color: white;
}
```

## Typography

- Use the MD3 **type scale** roles: `display`, `headline`, `title`, `body`, `label` (large/medium/small) — map product copy to roles, do not invent ad-hoc sizes.
- Prefer **one typeface family** (or the platform default) with weight/size via the scale; avoid mixing many display fonts.
- Keep line length readable (~40–60 characters for body on large screens); use type roles, not raw `px`, for hierarchy.
- Do not encode meaning with color alone; pair with type weight/size or icons.

## Layout, spacing & shape

- Prefer **8dp spacing rhythm**; use 4dp only for fine alignment (icon to label).
- Follow responsive **window size classes** / breakpoints: compact, medium, expanded — adapt navigation (bar → rail → drawer) rather than shrinking a phone layout.
- Prefer **canonical layouts** (list-detail, feed, supporting pane) when they fit the task.
- Use MD3 **shape scale** (corner radii per component role). Do not mix random radii; keep shape family consistent.
- Prefer padding and inset over decorative dividers; use `outline-variant` dividers sparingly.

## Elevation, surfaces & state

- Prefer **tonal surface elevation** (surface containers + tint) over heavy drop shadows (MD3 default).
- Use elevation/shadow when the platform still expects it (temporary sheets, menus), but keep levels few and consistent.
- Interactive components need clear **state layers** (hover, focus, pressed, dragged, disabled) via the system overlay model — do not invent opaque background swaps that break contrast.
- Disabled content must remain readable enough to recognize, with reduced emphasis — do not rely only on low-contrast gray.

## Components

- Prefer **standard Material components** before custom widgets: buttons, icon buttons, FABs, fields, chips, cards, dialogs, sheets, snackbars, navigation bar/rail/drawer, tabs, menus, lists.
- **Buttons**: filled = primary action; tonal/outlined = secondary; text = low emphasis; FAB = single prominent create/navigate action per screen.
- Prefer **one primary action** per view region; avoid competing filled buttons.
- **Text fields**: outlined or filled per theme; always show labels; use supporting/error text, not placeholder-as-label.
- **Dialogs** for critical decisions; **bottom sheets** for optional tasks or mobile pickers; **snackbars** for brief confirmations (not errors that need durable UI).
- **Cards** for contained, actionable content groups — not as default page chrome. Prefer surface layout without cards when content is the page itself.
- Navigation: keep destinations stable; use badges sparingly; label icons in compact nav when space allows.

## Motion

- Prefer **MD3 motion** (easing, duration tokens, shared-axis / fade / container transforms) over arbitrary CSS bounce.
- Motion should clarify hierarchy or spatial continuity; skip decoration-only animation.
- Respect **reduced motion** (`prefers-reduced-motion` / platform accessibility settings): provide instant or fade-only alternatives.
- Keep durations short for micro-interactions; longer only for large layout transitions.

## Accessibility

- Meet contrast for text and essential icons against their surface roles (aim for WCAG AA as a floor).
- Touch targets ≥ **48×48dp** (or platform minimum), with adequate spacing between targets.
- Every interactive control must be **keyboard reachable** with a visible focus indicator (use the system focus ring / state layer).
- Prefer semantic structure (headings, lists, landmarks, native controls) before ARIA.
- Do not convey state by color alone (pair with icons, text, or shape).
- Announce dynamic updates (snackbars, validation) in a way the platform screen reader expects.

## Implementation notes (by stack)

- **MUI (Material UI)**: prefer theme `palette` / MD3 theme extras, `sx` or tokens over one-off styles; use MUI components before custom CSS that fights the system.
- **Material Web**: use documented component APIs and design tokens; avoid restyling shadow DOM internals.
- **Jetpack Compose**: use `MaterialTheme` color/type/shape; prefer Material3 composables (`FilledTonalButton`, `Scaffold`, etc.).
- **Flutter**: use `ThemeData` / `ColorScheme.fromSeed` and Material 3 widgets; avoid hard-coded `Colors.*` in product UI.

When unsure about a token or component API, check [m3.material.io](https://m3.material.io/) and the library docs for the project’s stack.

## Anti-patterns (do not)

- Hard-coded brand colors on text/icons without `on-*` / contrast-safe pairs.
- Mixing Material 2 elevation metaphors and MD3 tonal surfaces inconsistently in the same flow.
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
