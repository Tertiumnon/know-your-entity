# Naming Conventions Overview

Good naming makes code understandable, consistent, and simple to maintain. This guide provides recommendations and practical examples to establish a unified style for your team.

## General Principles

- **Name reflects entity and purpose, not implementation** — Don't include types, formats, or technical details in names
- **Clarity over brevity** — Prefer full, clear names over shortened ones. Readability is more important than saving a few characters
- **Consistent style** — Use one style throughout your project:
  - `camelCase` for variables/functions
  - `PascalCase` for classes/interfaces
  - `kebab-case` or `camelCase` for files (depending on project convention)
- **Team agreement** — Agree on rules with your team and document them in your repository's style guide

## When to Use Abbreviations

Use abbreviations sparingly, only when the meaning is clear from context:
- Acceptable: `id`, `msg`, `err`
- Avoid: Making up custom abbreviations without explanation

In short functions or obvious contexts (like `users.map(u => u.name)`), local abbreviations are acceptable.

## Naming by Category

- **[Variables](variables.md)** — Primitives, objects, and collections
- **[Functions & Methods](functions-methods.md)** — Action verbs and semantic clarity
- **[Classes & Interfaces](classes-interfaces.md)** — PascalCase and meaningful suffixes
- **[Files & Folders](files-folders.md)** — Alignment with content and consistency
- **[Common Anti-Patterns](anti-patterns.md)** — Mistakes to avoid

---

## Quick Checklist

When naming something, ask:

- [ ] Is the name clear without looking at the implementation?
- [ ] Does it reflect the entity/purpose, not the type?
- [ ] Is it consistent with other names in the codebase?
- [ ] Would a new team member understand it?
- [ ] Does it follow the team's established style?

---

## Related

- [Variables](variables.md)
- [Functions & Methods](functions-methods.md)
- [Classes & Interfaces](classes-interfaces.md)
- [Files & Folders](files-folders.md)
- [Anti-Patterns](anti-patterns.md)
