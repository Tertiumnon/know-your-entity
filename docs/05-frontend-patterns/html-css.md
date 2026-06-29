# HTML & CSS Patterns

Conventions for HTML structure, CSS/LESS styling, and class naming in the frontend.

## Elements

Elements are the building blocks of the UI. Keep markup simple and scoped to avoid conflicts and make styling predictable.

### Basic Structure

```html
<span class="logo">
  <img src="logo.svg" alt="Logo" />
</span>
```

### LESS/CSS Example

```less
.logo {
  img {
    // styles for image inside logo
  }
}
```

### Multiple Similar Elements

Be explicit with descriptive class names instead of generic ones:

```html
<span class="main_logo">
  <img src="logo.svg" alt="Main logo" />
</span>

<span class="secondary_logo">
  <img src="secondary.svg" alt="Secondary logo" />
</span>
```

## Modifiers

Modifiers represent element variations or states (color, size, hidden, disabled, etc.).

### Two Types of Modifiers

**1. Generic modifiers** — Applied to any element

```html
<span class="logo hidden">
  <img src="logo.svg" alt="Hidden logo" />
</span>

<div class="box hidden">Content</div>
```

Pattern: `{{element}} {{modifier}}`

**2. Element-specific (BEM-like)** — Variations of a specific element

```html
<span class="main_logo main_logo--xl">
  <img src="logo.svg" alt="Extra large logo" />
</span>

<span class="main_logo main_logo--xs">
  <img src="logo.svg" alt="Small logo" />
</span>

<span class="main_logo main_logo--dark">
  <img src="logo.svg" alt="Dark logo" />
</span>
```

Pattern: `{{element}}--{{modifier}}`

### CSS/LESS Styles

```less
.main_logo {
  display: inline-block;
  vertical-align: middle;

  img {
    max-width: 100%;
    height: auto;
  }

  // Element-specific modifiers
  &.main_logo--xs {
    width: 32px;
    img {
      // styles for small variant
    }
  }

  &.main_logo--xl {
    width: 120px;
    img {
      // styles for large variant
    }
  }

  &.main_logo--dark {
    filter: brightness(0.8);
  }
}

// Generic modifiers
.hidden {
  display: none;
}

.disabled {
  opacity: 0.5;
  pointer-events: none;
}

.active {
  font-weight: bold;
  color: var(--primary-color);
}
```

### State Modifiers

Use modifiers to represent interactive states:

```html
<!-- Button states -->
<button class="btn btn--primary">Default</button>
<button class="btn btn--primary btn--hover">Hover</button>
<button class="btn btn--primary btn--disabled">Disabled</button>
<button class="btn btn--primary btn--active">Active</button>
```

```less
.btn {
  &.btn--primary {
    background: var(--primary-color);
  }

  &.btn--hover {
    background: var(--primary-hover);
  }

  &.btn--disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  &.btn--active {
    box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.2);
  }
}
```

## Naming Conventions

### Classes for HTML/CSS

- Use **lowercase** for HTML/CSS classes
- Use **underscores** for multi-word names: `main_logo`, `user_avatar`
- Use **double underscore** for parent→child relationships: `card__header`, `card__body`
- Use **double dash** for modifiers: `card--large`, `card--dark`

### Examples

```html
<!-- Parent-child structure -->
<div class="card">
  <div class="card__header">
    <h2>Title</h2>
  </div>
  <div class="card__body">
    <p>Content</p>
  </div>
  <div class="card__footer">
    <button>Action</button>
  </div>
</div>

<!-- Modifiers -->
<div class="card card--large">
  <!-- Large variant -->
</div>

<div class="card card--dark">
  <!-- Dark variant -->
</div>
```

### CSS Structure

```less
.card {
  padding: 16px;
  border: 1px solid var(--border-color);
  border-radius: 4px;

  // Child elements
  .card__header {
    border-bottom: 1px solid var(--border-color);
    margin-bottom: 12px;

    h2 {
      margin: 0;
      font-size: 18px;
    }
  }

  .card__body {
    margin-bottom: 12px;
  }

  .card__footer {
    border-top: 1px solid var(--border-color);
    padding-top: 12px;
  }

  // Modifiers
  &.card--large {
    padding: 24px;
  }

  &.card--dark {
    background: var(--dark-bg);
    color: var(--dark-text);
  }

  &.card--elevated {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  }
}
```

## Naming for HTML Attributes

Use underscores in HTML `name` attributes (for form handling):

```html
<input type="text" name="user_email" />
<input type="text" name="billing_address" />
<select name="product_category">
  <option>Electronics</option>
  <option>Clothing</option>
</select>
```

This keeps consistency when retrieving form data:

```typescript
const formData = new FormData(document.querySelector('form'));
const userEmail = formData.get('user_email');
const billingAddress = formData.get('billing_address');
```

## Practical Layout Example

```html
<div class="page_container">
  <header class="page__header">
    <div class="page__header__logo">
      <img src="logo.svg" alt="Logo" />
    </div>
    <nav class="page__header__nav">
      <a class="nav__link nav__link--active">Home</a>
      <a class="nav__link">About</a>
      <a class="nav__link">Contact</a>
    </nav>
  </header>

  <main class="page__content">
    <div class="hero hero--large">
      <h1>Welcome</h1>
      <p class="hero__subtitle">This is the main message</p>
    </div>

    <section class="features">
      <div class="feature_card feature_card--highlighted">
        <h3>Feature 1</h3>
        <p>Description</p>
      </div>
      <div class="feature_card">
        <h3>Feature 2</h3>
        <p>Description</p>
      </div>
    </section>
  </main>

  <footer class="page__footer">
    <p>&copy; 2025. All rights reserved.</p>
  </footer>
</div>
```

## Scoped Styles

Keep CSS rules scoped to avoid conflicts:

```less
// Good: Scoped to a component
.user_profile {
  .profile__avatar {
    width: 100px;
    height: 100px;
  }

  .profile__name {
    font-weight: bold;
    font-size: 18px;
  }

  .profile__bio {
    color: var(--text-secondary);
  }
}

// Bad: Generic names that conflict
.avatar {      // Could conflict with other .avatar classes
  width: 100px;
}

.name {        // Too generic
  font-weight: bold;
}
```

---

## References

- [BEM](https://bem.info/) — Block Element Modifier methodology
- [Atomic Design](https://atomicdesign.bradfrost.com/) — Design system thinking
- [SMACSS](https://smacss.com/) — Scalable and Modular Architecture for CSS

---

## Related

- [JavaScript Naming](javascript.md)
- [Component Structure](../06-architecture/components.md)
