# Module Organization & File Collocation

Every component/entity should have its own folder with all related files using the same base name but different suffixes.

## The Golden Rule: Always Singular

**Every folder, every file, every interface, every class name is SINGULAR — never plural.**

```
✅ CORRECT: Singular (standard rule)
src/user/                (not users/)
  user.ts                (not users.ts)
  user.repo.ts           (not users.repo.ts)

✅ EXCEPTIONS: Plural only for these
src/pages/               ← Contains multiple pages
src/routes/              ← Contains multiple routes

❌ WRONG: Plural (for everything else)
src/users/               ← Wrong! (entity should be singular)
  users.ts               ← Wrong!
  users.repo.ts          ← Wrong!
```

**Why?**
- **Consistency** — One standard applies everywhere
- **Clear boundaries** — One folder = one entity concept
- **Easier imports** — Natural to think "User" not "Users"
- **Follows conventions** — Domain-driven design always uses singular
- **Reflects reality** — `user.repo.ts` manages individual Users, not a collection

### Exceptions: pages/ and routes/

These two folders use **plural** because they contain collections of items:

```
src/
  pages/                    ← Plural (contains page components)
    home.tsx
    profile.tsx
    settings.tsx
    user/                   ← Singular (specific page folder)
      user-detail.tsx
      user-settings.tsx

  routes/                   ← Plural (contains route definitions)
    user.routes.ts
    product.routes.ts
    auth.routes.ts
```

**Why these are exceptions:**
- `pages/` is a **container for multiple pages** (inherently a collection)
- `routes/` is a **container for multiple route definitions** (inherently a collection)
- The folder name describes the **collection type**, not a single entity

**Everything inside these folders is still singular:**
```
✅ CORRECT
pages/
  home.tsx               (singular)
  profile.tsx            (singular)
  user-detail.tsx        (singular)

routes/
  user.routes.ts         (singular)
  product.routes.ts      (singular)
```

## The Pattern

Each domain entity or feature gets a **dedicated folder**. All files in that folder share the same base name with different extensions/suffixes:

```
src/
  user/                          ← One folder per entity
    user.ts                      ← Base interface/type
    user.service.ts              ← Service
    user.repository.ts           ← Repository
    user.controller.ts           ← Controller
    user.dto.ts                  ← DTOs (Create, Update, Response)
    user.mapper.ts               ← Mappers/Transformers
    user.validator.ts            ← Validators
    user.test.ts (or .spec.ts)   ← Tests
    index.ts                     ← Optional barrel export

  product/                       ← Another entity
    product.ts
    product.service.ts
    product.repository.ts
    product.controller.ts
    product.dto.ts
    product.validator.ts
    product.test.ts
    index.ts

  order/                         ← Another entity
    order.ts
    order.service.ts
    order.repository.ts
    order.controller.ts
    order.dto.ts
    order.mapper.ts
    order.validator.ts
    order.test.ts
    index.ts
```

## Why This Pattern?

### ✅ Benefits

1. **High Cohesion** — All related code lives together
   - Want to work on User? Everything is in `/user/`
   - No hunting through folders for User-related files

2. **Easy Discovery** — Consistent naming makes it obvious what exists
   - `user.service.ts` → UserService
   - `user.repository.ts` → UserRepository
   - No guessing about what's where

3. **Self-Documenting** — File structure tells you about the entity
   ```
   user/
     user.ts              ← I'm a User
     user.service.ts      ← I have a service
     user.controller.ts   ← I have a controller
     user.dto.ts          ← I have DTOs
   ```

4. **Atomic Refactoring** — Rename entity? Rename the folder
   ```bash
   # Rename user → account
   mv src/user src/account
   # All files moved together, no orphaned files
   ```

5. **Clear Boundaries** — No shared "utils" directories
   - UserValidator only in `/user/`
   - ProductValidator only in `/product/`
   - No conflicts or confusion

6. **Testing Proximity** — Tests live with code
   ```
   user/
     user.service.ts
     user.service.test.ts  ← Right here, not in separate folder
   ```

## File Naming Patterns

Each file type follows a consistent suffix pattern:

| File Type | Pattern | Example | Contains |
|-----------|---------|---------|----------|
| **Type/Interface** | `{name}.ts` | `user.ts` | User interface, types |
| **Service** | `{name}.service.ts` | `user.service.ts` | UserService class |
| **Repository** | `{name}.repository.ts` | `user.repository.ts` | UserRepository class |
| **Controller** | `{name}.controller.ts` | `user.controller.ts` | UserController class |
| **DTO** | `{name}.dto.ts` | `user.dto.ts` | DTOs (CreateUserDto, etc.) |
| **Validator** | `{name}.validator.ts` | `user.validator.ts` | Validation logic |
| **Mapper** | `{name}.mapper.ts` | `user.mapper.ts` | Entity ↔ DTO mapping |
| **Constants** | `{name}.constants.ts` | `user.constants.ts` | User-specific constants |
| **Tests** | `{name}.test.ts` or `.spec.ts` | `user.service.test.ts` | Test suite |
| **Barrel Export** | `index.ts` | `user/index.ts` | Public API (optional) |

## Complete Example: User Entity

```
src/
  user/
    user.ts
    ├─ Contains: User interface, enums, types
    │
    user.service.ts
    ├─ Contains: UserService class
    │ └─ Methods: find, get, create, update, delete
    │
    user.repository.ts
    ├─ Contains: UserRepository class
    │ └─ Methods: save, findById, findAll, delete
    │
    user.controller.ts
    ├─ Contains: UserController class
    │ └─ Routes: GET /users, POST /users, GET /users/:id, etc.
    │
    user.dto.ts
    ├─ Contains: CreateUserDto, UpdateUserDto, UserResponseDto
    │
    user.validator.ts
    ├─ Contains: validateEmail, validatePassword, etc.
    │
    user.mapper.ts
    ├─ Contains: Functions to map User ↔ DTOs
    │
    user.constants.ts
    ├─ Contains: MIN_PASSWORD_LENGTH, ROLES, etc.
    │
    user.service.test.ts
    ├─ Contains: Tests for UserService
    │
    index.ts (optional)
    └─ Exports: User, UserService, UserController
```

## Frontend Components Follow Same Pattern

```
src/
  components/
    user-card/                 ← One folder per component
      user-card.tsx            ← Component
      user-card.module.css     ← Styles
      user-card.test.tsx       ← Tests
      user-card.types.ts       ← Props interface
      index.ts                 ← Export

    product-list/
      product-list.tsx
      product-list.module.css
      product-list.test.tsx
      product-list.types.ts
      index.ts

    user-profile/
      user-profile.tsx
      user-profile.module.css
      user-profile.test.tsx
      user-profile.types.ts
      user-profile.hooks.ts    ← Custom hooks
      index.ts
```

## What NOT to Do

### ❌ Don't: Scattered Files

```
src/
  interfaces/
    user.ts              ← User interface here
  services/
    user.service.ts      ← Service in different folder
  controllers/
    user.controller.ts   ← Controller in yet another folder
  dto/
    user.dto.ts          ← DTO somewhere else
  validators/
    user.validator.ts    ← Validator in another folder

# Problems:
# - To work on User, you navigate 5 different folders
# - Hard to find all User-related code
# - No clear boundaries
```

### ❌ Don't: Generic Shared Folders

```
src/
  utils/
    validators.ts        ← Everything goes here
    mappers.ts
    helpers.ts
    formatters.ts
    // 100+ functions mixed together

# Problems:
# - No organization
# - Hard to find what you need
# - Everything is global and shared
# - Becomes a dumping ground
```

### ❌ Don't: Deep Nesting

```
src/
  domain/
    entities/
      user/
        implementation/
          services/
            user.service.ts    ← Too deep!

# Problems:
# - File paths are long
# - Cognitive overload
# - Hard to navigate
# - Go against "maximum 3 levels" rule
```

## Module Organization by Feature

For larger applications, organize by feature:

```
src/
  features/
    authentication/
      auth.ts
      auth.service.ts
      auth.controller.ts
      auth.dto.ts
      auth.validator.ts
      index.ts

    user/
      user.ts
      user.service.ts
      user.controller.ts
      user.dto.ts
      user.mapper.ts
      index.ts

    product/
      product.ts
      product.service.ts
      product.controller.ts
      product.dto.ts
      product.validator.ts
      index.ts

    order/
      order.ts
      order.service.ts
      order.controller.ts
      order.dto.ts
      index.ts
```

## Shared Code (Genuinely Shared)

Only code that's **truly** shared across multiple entities goes to shared folders:

```
src/
  shared/
    types/
      pagination.types.ts    ← Used by many entities
      sorting.types.ts       ← Used by many entities
      filters.types.ts       ← Used by many entities

    utils/
      date-formatter.ts      ← No entity owns this
      string-validator.ts    ← No entity owns this
      error-handler.ts       ← No entity owns this

    middleware/
      auth.middleware.ts     ← Not entity-specific
      error.middleware.ts    ← Not entity-specific

  features/
    user/
      user.ts
      user.service.ts
      // Uses shared/utils/date-formatter.ts
      // Uses shared/types/pagination.types.ts
```

**Test**: If you could move a file into an entity folder, it doesn't belong in shared.

## Import Patterns

### From Same Module

```typescript
// user/user.service.ts
import { User } from './user';                    // Same folder
import { UserRepository } from './user.repository';
import { CreateUserDto } from './user.dto';
import { validateEmail } from './user.validator';
```

### From Other Module

```typescript
// product/product.service.ts
// Importing from user module
import { User } from '../user';              // Via barrel
// or
import { User } from '../user/user';         // Direct import
```

### From Shared Code

```typescript
// user/user.service.ts
// Using shared utilities
import { formatDate } from '../../shared/utils/date-formatter';
import { Pagination } from '../../shared/types/pagination.types';
```

## Naming Consistency Rules

### ⭐ Rule 0: ALWAYS SINGULAR (with 2 exceptions)

This is the fundamental rule. Everything is singular: folders, files, class names, interface names.

**Exceptions:** `pages/` and `routes/` are plural because they contain collections.

```
✅ SINGULAR (Standard rule)
src/user/
src/product/
src/order/
src/comment/

src/user/user.ts
src/user/user.repo.ts
src/product/product.ts

class User { }
class UserRepository { }
interface CreateUserDto { }

✅ PLURAL (Only exceptions)
src/pages/              ← Contains multiple pages
src/routes/             ← Contains multiple routes

❌ PLURAL (Always wrong — for everything else)
src/users/            ← WRONG! (entity should be singular)
src/products/         ← WRONG!
src/orders/           ← WRONG!

src/users/users.ts    ← WRONG!
class Users { }       ← WRONG!
```

**Why this rule is nearly absolute:**
- Consistency across entire codebase
- No ambiguity about naming
- Matches DDD (Domain-Driven Design) conventions
- Easier to remember ("always singular, except pages/ and routes/")
- Only 2 clear exceptions that describe container collections

### Rule 1: Base name matches the entity

```
Entity: User
  ✅ user.ts
  ✅ user.repo.ts
  ✅ user.validator.ts
  ❌ users.ts (plural is wrong)
  ❌ user-entity.ts (redundant)
```

**Rule 2**: File type suffixes are consistent

```
  ✅ user.service.ts, product.service.ts, order.service.ts
  ❌ user.service.ts, productService.ts, order-service.ts (inconsistent)
```

**Rule 3**: All files in a folder start with folder name

```
folder: src/user/
  ✅ user.ts
  ✅ user.service.ts
  ✅ user.controller.ts
  ❌ types.ts (should be user.ts)
  ❌ service.ts (should be user.service.ts)
  ❌ user-entity.ts (should be user.ts)
```

## Refactoring: Merging Entities

When two entities should merge:

```
Before:
  src/
    user/
      user.ts
      user.service.ts
      ...
    profile/
      profile.ts
      profile.service.ts
      ...

Decision: User and Profile are too coupled, merge them

After:
  src/
    user/
      user.ts          ← Now contains both User and Profile
      user.service.ts
      index.ts

  rm -rf src/profile/  ← Removed entire folder
```

## Refactoring: Splitting Entities

When one entity becomes too complex:

```
Before:
  src/
    user/
      user.ts
      user.service.ts
      user.controller.ts
      user.dto.ts
      user.mapper.ts
      user.validator.ts
      user.authentication.ts        ← Getting too big
      user.profile-settings.ts
      user.notifications.ts
      user.audit-log.ts

After:
  src/
    user/
      user.ts
      user.service.ts
      user.controller.ts
      user.dto.ts
      index.ts

    user-authentication/           ← Split out
      user-authentication.ts
      user-authentication.service.ts
      index.ts

    user-settings/                 ← Split out
      user-settings.ts
      user-settings.service.ts
      index.ts

    user-notifications/            ← Split out
      user-notifications.ts
      user-notifications.service.ts
      index.ts
```

## Quick Checklist

When creating a new entity/component:

**Singular Naming Rule:**
- [ ] **Entity name is SINGULAR** (user, not users)
- [ ] Folder name is singular: `src/user/` (not `src/users/`)
- [ ] File base name is singular: `user.ts` (not `users.ts`)
- [ ] Interface/class name is singular: `User` (not `Users`)
- [ ] ⚠️ **Exceptions**: `pages/` and `routes/` folders are plural (only these!)

**File Organization:**
- [ ] All files use base name: `{entity-name}.{type}.ts`
- [ ] Has interface file: `{entity-name}.ts`
- [ ] Has repo file: `{entity-name}.repo.ts` (not .service.ts)
- [ ] Has tests: `{entity-name}.*.test.ts` or `.spec.ts`
- [ ] Optional index.ts for exports (minimal, not barrel)
- [ ] No files with generic names (utils.ts, types.ts, etc.)
- [ ] No shared logic — moved to `/shared/` if truly common

---

## Related

- [Flat Entities](../03-entity-structure/flat-entities.md)
- [Common Interfaces](../03-entity-structure/common-interfaces.md)
- [Files & Folders Naming](../02-naming-conventions/files-folders.md)
