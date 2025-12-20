# Frontend Style Guide (HTML / CSS / JS)

This guide covers conventions for HTML structure, CSS (including LESS examples), and basic JavaScript naming used by the project.

## Elements

Elements are the building blocks of the UI. Keep markup simple and scoped.

Example markup:

```html
<span class="logo">
  <img src="logo.svg" alt="Logo" />
</span>
```

LESS example:

```less
.logo {
  img {}
}
```

If multiple logos exist, be explicit with a more descriptive class:

```html
<span class="main_logo">
  <img src="logo.svg" alt="Main logo" />
</span>
```

## Modifiers

Modifiers represent element variations or states (color, size, hidden, etc.). Use two forms:

- Generic modifiers applied to any element: `{{element}} {{modifier}}` (e.g. `.hidden`).
- Element-specific (BEM-like): `{{element}}--{{modifier}}` (e.g. `.main_logo--xl`).

Examples:

```html
<span class="main_logo hidden">
  <img src="logo.svg" alt="Hidden logo" />
</span>

<span class="main_logo main_logo--xs">
  <img src="logo.svg" alt="Small logo" />
</span>
```

CSS example with modifiers:

```less
.main_logo {
  img {}
  &.main_logo--xs { img {} }
  &.main_logo--xl { img {} }
  &.main_logo--dark { img {} }
}
```

## Naming

- Use short, meaningful names. Prefer lowercase for HTML/CSS classes.
- Use underscores in HTML `name` attributes if needed (e.g. `secret_name`).
- Use double underscore to show parent→child relationship: `parent__child`.
- In JavaScript prefer `camelCase` for variables and identifiers.

Example JavaScript selector:

```javascript
const secretName = document.querySelector('input[name="secret_name"]');
```

## References

- [BEM](https://bem.info/)
- [Atomic Design](https://atomicdesign.bradfrost.com/)
