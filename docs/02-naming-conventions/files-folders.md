# Naming Files & Folders

File and folder names should clearly reflect their content and follow a consistent style throughout your project.

## General Principles

- **Match content name** — File names should correspond to the main class, interface, or component they export
- **Consistent style** — Choose one naming convention and apply it everywhere:
  - `kebab-case` (e.g., `user-service.ts`) — Popular in modern JavaScript/TypeScript projects
  - `camelCase` (e.g., `userService.ts`) — Alternative convention
  - `snake_case` (e.g., `user_service.ts`) — Less common in JS projects
  - `PascalCase` (e.g., `UserService.ts`) — Used by some teams, matches class names
- **Be specific** — Avoid generic names like `utils`, `helpers`, `common` at the root level

## Entity Files — Always Singular

**Golden Rule**: Entity names are ALWAYS singular, never plural.

Files for specific entities should use the entity name in singular form:

```
// User entity files — SINGULAR
src/
  user/                       ← Singular (not users/)
    user.ts                   ← Singular (not users.ts)
    user.service.ts           ← Singular (not users.service.ts)
    user.controller.ts        ← Singular (not users.controller.ts)
    user.repository.ts        ← Singular (not users.repository.ts)
    user.dto.ts               ← Singular (not users.dto.ts)
    user.interface.ts         ← Singular (not users.interface.ts)
```

**Why always singular?**
- **Consistency** — One rule, no exceptions
- **Clarity** — "user" represents the User concept, not a collection
- **DDD** — Domain-Driven Design uses singular entity names
- **Code** — `class User {}`, not `class Users {}`
- **Imports** — `import User from './user'` feels natural

**Wrong (Don't do this):**

```
❌ src/users/              (Plural is wrong)
❌ user.ts in users/ folder
❌ class Users { }
❌ interface Users { }
```

### Exceptions: pages/ and routes/

These two folder names are **plural** because they contain collections:

```
✅ CORRECT — These use plural
src/pages/                 (Contains multiple page components)
  home.tsx
  profile.tsx
  user-detail.tsx          (Individual page still singular)

src/routes/                (Contains multiple route definitions)
  user.routes.ts
  product.routes.ts        (Individual routes still singular)
```

Everything **inside** pages/ and routes/ uses **singular** naming.

## Folder Structure by Entity

Group all files related to an entity together:

```
src/
  entities/
    user/
      user.interface.ts
      user.service.ts
      user.controller.ts
      user.repository.ts
      user.dto.ts
    product/
      product.interface.ts
      product.service.ts
      product.controller.ts
      product.repository.ts
      product.dto.ts
    order/
      order.interface.ts
      order.service.ts
      order.controller.ts
      order.repository.ts
      order.dto.ts
```

## Component Files (Frontend)

For UI components, match the component name:

```
// React/Vue component
components/
  user-card/
    user-card.component.ts      // or .tsx/.vue
    user-card.module.css        // or .scss/.less
    user-card.types.ts          // Types/Props
    index.ts                    // Export

  product-list/
    product-list.component.ts
    product-list.module.css
    product-list.types.ts
    index.ts
```

Or with different naming conventions:

```
// Alternative: PascalCase
components/
  UserCard/
    UserCard.tsx
    UserCard.module.css
    UserCard.types.ts
    index.ts

  ProductList/
    ProductList.tsx
    ProductList.module.css
    ProductList.types.ts
    index.ts
```

## Utility and Helper Files

Be specific about what the utility does, don't use generic names:

```
// BAD
utils/
  helpers.ts
  utils.ts
  common.ts

// GOOD
utils/
  date-formatter.ts           // Date formatting functions
  string-validator.ts         // String validation functions
  error-handler.ts            // Error handling utilities
  api-client.ts               // API request utilities
```

## Configuration Files

Keep configuration file names clear and consistent:

```
config/
  app.config.ts               // Main app configuration
  database.config.ts          // Database configuration
  logger.config.ts            // Logger configuration
  environment.config.ts       // Environment-specific config
```

## Test Files

Test files should match the file they test, with a `.test` or `.spec` suffix:

```
// Jest/Vitest convention
src/
  user/
    user.service.ts
    user.service.test.ts        // or .spec.ts

  product/
    product.controller.ts
    product.controller.spec.ts   // or .test.ts
```

## Type Definition Files

Use `.types.ts` or `.interface.ts` for type-only files:

```
types/
  common.types.ts              // Common types
  api.types.ts                 // API-related types

user/
  user.types.ts                // User-specific types
  user.interface.ts            // User interface (alternative)
```

## Naming Conventions Comparison

Choose one and stick with it:

| Pattern | Example | Best For |
|---------|---------|----------|
| kebab-case | `user-service.ts` | Most modern JS/TS projects |
| camelCase | `userService.ts` | Some teams prefer this |
| snake_case | `user_service.ts` | Less common, some Python projects |
| PascalCase | `UserService.ts` | Matches class names, less common |

## Index Files

⚠️ **Warning**: While index files seem convenient, they can become an antipattern. See [Barrel Exports](barrel-exports.md) for detailed analysis.

### When Index Files Are Appropriate

Use `index.ts` only for intentional, minimal public APIs:

```typescript
// user/index.ts — Small, focused, clear intent
export { User } from './user.interface';
export { UserService } from './user.service';
// Only export what's part of the public API

// consumers import from the folder
import { User, UserService } from './entities/user';
```

### What to Avoid

❌ **Don't**: Create central barrels that export everything

```typescript
// entities/index.ts — BAD! Central barrel (maintenance burden)
export { User } from './user/user.interface';
export { UserService } from './user/user.service';
export { UserValidator } from './user/user.validator';
export { UserMapper } from './user/user.mapper';
// ... 50 more exports
// → Hidden dependencies, requires constant updates
```

✅ **Do**: Use direct imports instead

```typescript
import { UserService } from './entities/user/user.service';
// → Clear, explicit, zero maintenance
```

**[Read the full analysis of why barrel exports are problematic →](barrel-exports.md)**

## Directories with Multiple Concepts

When a folder contains multiple related items, name it by the category:

```
src/
  middleware/
    auth-middleware.ts
    error-middleware.ts
    cors-middleware.ts

  validators/
    email-validator.ts
    password-validator.ts
    phone-validator.ts

  hooks/ (React)
    use-user-data.ts
    use-form-validation.ts
    use-theme.ts
```

---

## Quick Checklist

When naming a file or folder:

- [ ] Does it clearly reflect the content?
- [ ] Is it consistent with other files in the project?
- [ ] Is it specific (not generic like `utils` or `helpers`)?
- [ ] Would a new team member understand what it contains?
- [ ] Does it use the agreed-upon naming convention?

---

## Related

- [Overview](overview.md)
- [Classes & Interfaces](classes-interfaces.md)
- [Anti-Patterns](anti-patterns.md)
