# Material Design implementation notes

Stack-specific guidance and token examples for the Material Design skill. Prefer host-project theme and components when they conflict. Official guidance: [m3.material.io](https://m3.material.io/).

## Token example (CSS roles)

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

## By stack

### MUI (Material UI)

Prefer theme `palette` / MD3 theme extras, `sx` or tokens over one-off styles; use MUI components before custom CSS that fights the system.

### Material Web

Use documented component APIs and design tokens; avoid restyling shadow DOM internals.

### Jetpack Compose

Use `MaterialTheme` color/type/shape; prefer Material3 composables (`FilledTonalButton`, `Scaffold`, etc.).

### Flutter

Use `ThemeData` / `ColorScheme.fromSeed` and Material 3 widgets; avoid hard-coded `Colors.*` in product UI.

When unsure about a token or component API, check [m3.material.io](https://m3.material.io/) and the library docs for the project’s stack.
