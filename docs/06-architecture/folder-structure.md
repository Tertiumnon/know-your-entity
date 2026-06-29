# Folder Structure: 4-Level Architecture

A consistent, scalable folder structure that organizes code by responsibility and layer.

## The Pattern

```
src/
├── core/                    ← Framework setup, configuration, shared utilities
├── entities/                ← Domain entities, models, business logic
├── components/ (or libs/)   ← Reusable components, services, middleware
└── pages/ (or routes/)      ← Application pages, routes, views
```

Each folder has a specific responsibility:

| Folder | Purpose | Contains |
|--------|---------|----------|
| **core/** | Bootstrapping & shared | App initialization, config, shared utils |
| **entities/** | Domain models | Data entities, business logic, repositories |
| **components/** (or **libs/**) | Reusable pieces | UI components, services, utilities |
| **pages/** (or **routes/**) | Application structure | Pages, routes, views |

---

## Detailed Breakdown

### 1. Core Layer (`src/core/`)

**Purpose**: Application foundation and shared configuration.

**Contains**:
- Application initialization and bootstrap
- Global configuration
- Environment variables and settings
- Shared utilities (not entity-specific)
- Global types and interfaces
- Middleware and interceptors (not entity-specific)

**Examples**:

```
src/core/
├── app.ts                      ← App initialization
├── app.config.ts               ← App configuration
├── environment.config.ts       ← Environment setup
├── logger.config.ts            ← Logger configuration
├── database.config.ts          ← Database connection
├── index.ts                    ← Main exports
├── utils/
│   ├── date-formatter.ts
│   ├── error-handler.ts
│   ├── string-validator.ts
│   └── http-client.ts          ← Shared HTTP client (not entity-specific)
├── middleware/
│   ├── auth.middleware.ts      ← Authentication (global)
│   ├── error.middleware.ts     ← Error handling (global)
│   └── cors.middleware.ts      ← CORS configuration
├── types/
│   ├── pagination.types.ts
│   ├── sorting.types.ts
│   └── common.types.ts
└── constants/
    ├── http-codes.ts
    └── app-constants.ts
```

**Key Rules**:
- ✅ Shared across multiple entities
- ✅ No entity-specific logic
- ✅ Configuration and setup code
- ❌ Don't add entity-specific files here
- ❌ Don't create "utils" that only one entity uses

---

### 2. Entities Layer (`src/entities/`)

**Purpose**: Domain entities and their business logic.

**Contains**:
- Entity models and interfaces
- Repositories and data access
- Business logic and services
- Validators
- Mappers and transformers
- Entity-specific constants

**Pattern**: One folder per entity, all files use the same base name.

**Examples**:

```
src/entities/
├── user/
│   ├── user.ts                 ← User interface/type
│   ├── user.repo.ts            ← Database access
│   ├── user.logic.ts           ← Business logic
│   ├── user.validator.ts       ← Validation
│   ├── user.mapper.ts          ← DTO mapping
│   ├── user.cache.ts           ← Caching
│   ├── user.constants.ts       ← User constants
│   ├── user.service.test.ts    ← Tests
│   └── index.ts                ← Exports (optional)
│
├── product/
│   ├── product.ts
│   ├── product.repo.ts
│   ├── product.logic.ts
│   ├── product.validator.ts
│   ├── product.mapper.ts
│   └── index.ts
│
├── order/
│   ├── order.ts
│   ├── order.repo.ts
│   ├── order.logic.ts
│   ├── order.manager.ts        ← Orchestrates multiple operations
│   ├── order.validator.ts
│   └── index.ts
│
└── comment/
    ├── comment.ts
    ├── comment.repo.ts
    ├── comment.logic.ts
    └── index.ts
```

**Key Rules**:
- ✅ One folder per entity
- ✅ All files in folder start with entity name (singular)
- ✅ Use specific suffixes: `.repo.ts`, `.logic.ts`, `.validator.ts`, etc.
- ✅ Keep entity folders independent (no cross-entity imports in logic)
- ❌ Never use plural folder names (it's `user/` not `users/`)
- ❌ Never use generic `.service.ts` suffix

**See Also**: [Module Organization](module-organization.md), [File Type Suffixes](../02-naming-conventions/file-type-suffixes.md)

---

### 3. Components/Libs Layer (`src/components/` or `src/libs/`)

**Purpose**: Reusable application components and utilities.

**Use `components/`** for:
- UI components (React, Vue, Angular, etc.)
- Visual elements and layouts
- Frontend-specific modules

**Use `libs/`** for:
- Business services (not entity-specific)
- Utility libraries
- Cross-cutting concerns
- Backend-focused organization

**Examples with `components/`**:

```
src/components/
├── user-card/
│   ├── user-card.tsx           ← Component
│   ├── user-card.module.css    ← Styles
│   ├── user-card.types.ts      ← Props
│   ├── user-card.test.tsx      ← Tests
│   └── index.ts                ← Export
│
├── product-list/
│   ├── product-list.tsx
│   ├── product-list.module.css
│   ├── product-list.types.ts
│   └── index.ts
│
├── modal/
│   ├── modal.tsx
│   ├── modal.module.css
│   └── index.ts
│
└── form-field/
    ├── form-field.tsx
    ├── form-field.module.css
    └── index.ts
```

**Examples with `libs/`**:

```
src/libs/
├── auth-service/               ← Cross-entity auth logic
│   ├── auth.ts
│   ├── auth.service.ts
│   ├── auth.validator.ts
│   └── index.ts
│
├── file-upload/                ← File handling (used by multiple entities)
│   ├── file-upload.service.ts
│   ├── file-upload.validator.ts
│   └── index.ts
│
├── notifications/              ← Notification system (cross-cutting)
│   ├── notification.service.ts
│   ├── notification.types.ts
│   └── index.ts
│
└── state-management/           ← Global state (if using Vuex/Redux/etc.)
    ├── store.ts
    ├── user-module.ts
    ├── product-module.ts
    └── index.ts
```

**Key Rules**:
- ✅ For `components/`: UI elements and visual components
- ✅ For `libs/`: Business services not owned by an entity
- ✅ Reusable across multiple features
- ✅ Can have entity-agnostic services here
- ❌ Don't put single-entity logic here (goes in entities/)
- ❌ Don't duplicate entity-specific code

**When to use `components/` vs `libs/`**:
- **Frontend-heavy projects** → Use `components/` for UI, `libs/` for non-UI reusables
- **Backend-heavy projects** → Use `libs/` for shared services, `components/` not needed
- **Full-stack** → Both folders, different purposes

---

### 4. Pages/Routes Layer (`src/pages/` or `src/routes/`)

**Purpose**: Application structure and navigation.

**Use `pages/`** for:
- Page components (in frameworks like Next.js, Nuxt, Gatsby)
- Full-page layouts
- Top-level views

**Use `routes/`** for:
- Route definitions and configuration
- Backend route handlers
- API endpoint definitions

**Examples with `pages/`**:

```
src/pages/
├── home.tsx                    ← Home page
├── not-found.tsx               ← 404 page
├── user/
│   ├── user-list.tsx           ← List all users
│   ├── user-detail.tsx         ← Single user page
│   └── user-settings.tsx       ← User settings page
├── product/
│   ├── product-list.tsx
│   ├── product-detail.tsx
│   └── product-search.tsx
└── cart/
    ├── cart.tsx
    └── checkout.tsx
```

**Examples with `routes/`**:

```
src/routes/
├── user.routes.ts             ← User routes
│   ├── GET /users
│   ├── GET /users/:id
│   ├── POST /users
│   ├── PUT /users/:id
│   └── DELETE /users/:id
│
├── product.routes.ts          ← Product routes
│   ├── GET /products
│   ├── GET /products/:id
│   ├── POST /products
│   └── ...
│
├── order.routes.ts            ← Order routes
├── auth.routes.ts             ← Auth routes
└── index.ts                   ← Combine all routes
```

**Key Rules**:
- ✅ `pages/` folder is PLURAL (contains multiple pages)
- ✅ `routes/` folder is PLURAL (contains multiple routes)
- ✅ Files inside are still SINGULAR: `user-detail.tsx`, `user.routes.ts`
- ✅ Each page/route maps to a URL path
- ❌ Don't put business logic in pages (import from entities/)
- ❌ Don't use nested page folders beyond one level

**See Also**: [Module Organization](module-organization.md#exceptions-pages-and-routes)

---

## Complete Example Structure

```
src/
│
├── core/                           ← Application foundation
│   ├── app.ts
│   ├── app.config.ts
│   ├── environment.config.ts
│   ├── logger/
│   │   ├── logger.config.ts
│   │   └── logger.types.ts
│   ├── database/
│   │   ├── database.config.ts
│   │   └── database.connection.ts
│   ├── utils/
│   │   ├── date-formatter.ts
│   │   ├── string-validator.ts
│   │   └── error-handler.ts
│   ├── types/
│   │   ├── pagination.types.ts
│   │   └── common.types.ts
│   └── index.ts
│
├── entities/                       ← Domain logic
│   ├── user/
│   │   ├── user.ts
│   │   ├── user.repo.ts
│   │   ├── user.logic.ts
│   │   ├── user.validator.ts
│   │   ├── user.mapper.ts
│   │   ├── user.constants.ts
│   │   └── index.ts
│   │
│   ├── product/
│   │   ├── product.ts
│   │   ├── product.repo.ts
│   │   ├── product.logic.ts
│   │   ├── product.validator.ts
│   │   └── index.ts
│   │
│   ├── order/
│   │   ├── order.ts
│   │   ├── order.repo.ts
│   │   ├── order.logic.ts
│   │   ├── order.manager.ts
│   │   ├── order.validator.ts
│   │   └── index.ts
│   │
│   └── comment/
│       ├── comment.ts
│       ├── comment.repo.ts
│       ├── comment.logic.ts
│       └── index.ts
│
├── components/                     ← Reusable UI components
│   ├── user-card/
│   │   ├── user-card.tsx
│   │   ├── user-card.module.css
│   │   ├── user-card.types.ts
│   │   └── index.ts
│   │
│   ├── product-list/
│   │   ├── product-list.tsx
│   │   ├── product-list.module.css
│   │   └── index.ts
│   │
│   ├── modal/
│   │   ├── modal.tsx
│   │   ├── modal.module.css
│   │   └── index.ts
│   │
│   └── form-field/
│       ├── form-field.tsx
│       ├── form-field.module.css
│       └── index.ts
│
├── libs/                           ← Shared services
│   ├── auth/
│   │   ├── auth.service.ts
│   │   ├── auth.validator.ts
│   │   └── index.ts
│   │
│   ├── notifications/
│   │   ├── notification.service.ts
│   │   ├── notification.types.ts
│   │   └── index.ts
│   │
│   └── file-upload/
│       ├── file-upload.service.ts
│       └── index.ts
│
└── pages/                          ← Application pages
    ├── home.tsx
    ├── not-found.tsx
    ├── user/
    │   ├── user-list.tsx
    │   ├── user-detail.tsx
    │   └── user-settings.tsx
    ├── product/
    │   ├── product-list.tsx
    │   ├── product-detail.tsx
    │   └── product-search.tsx
    └── cart/
        ├── cart.tsx
        └── checkout.tsx
```

---

## Maximum 3 Levels Deep

This structure enforces a **maximum of 3 folder levels** to prevent cognitive overload:

```
✅ GOOD — 3 levels max
src/
  entities/          ← Level 1
    user/            ← Level 2
      user.repo.ts   ← Level 3 (file)

✅ GOOD — 2 levels max
src/
  components/        ← Level 1
    user-card/       ← Level 2
      user-card.tsx  ← Level 3 (file)

❌ BAD — 4+ levels (too deep)
src/
  domain/            ← Level 1
    entities/        ← Level 2
      user/          ← Level 3
        repository/  ← Level 4 ← TOO DEEP!
          user.repo.ts
```

**Why this matters**:
- Easier navigation and file discovery
- Clearer mental model of codebase
- Faster development velocity
- Reduced context switching

---

## Folder Naming Rules

### Always Singular (except pages/ and routes/)

```
✅ CORRECT — Singular
src/user/
src/product/
src/comment/
src/entities/

✅ EXCEPTIONS — Plural only for these
src/pages/          (contains multiple pages)
src/routes/         (contains multiple routes)

❌ WRONG — Plural (for everything else)
src/users/          (should be user/)
src/products/       (should be product/)
src/entities/user/  (should be entities/)
```

**Why**: Singular is clearer conceptually. A folder named `user/` represents the User concept, not a collection of users.

---

## Import Patterns

### Within Same Entity

```typescript
// user/user.logic.ts
import { User } from './user';
import { UserRepository } from './user.repo';
import { validateEmail } from './user.validator';
```

### From Other Entity

```typescript
// order/order.logic.ts
// Importing User from user entity
import { User } from '../user';        // Via barrel
// or
import { User } from '../user/user';   // Direct import
```

### From Core

```typescript
// entities/user/user.logic.ts
// Using shared utilities
import { formatDate } from '../../core/utils/date-formatter';
import { Pagination } from '../../core/types/pagination.types';
```

### From Libs/Components

```typescript
// entities/order/order.logic.ts
// Using shared service
import { AuthService } from '../../libs/auth';

// pages/user-list.tsx
// Using component
import { UserCard } from '../../components/user-card';
```

---

## Variations by Project Type

### Backend Project (Node.js/Express/Nest.js)

```
src/
├── core/              ← App setup, config
├── entities/          ← Domain models and logic
├── libs/              ← Shared services
├── routes/            ← API route definitions
└── middleware/        ← Global middleware (or in core/)
```

**Note**: Skip `pages/` and `components/` folders entirely.

---

### Frontend Project (React/Vue/Angular)

```
src/
├── core/              ← Config, shared utils
├── entities/          ← Domain models (optional)
├── components/        ← UI components
└── pages/             ← Application pages
```

**Note**: `libs/` only if you have shared services.

---

### Full-Stack Project

```
src/
├── core/              ← Shared config, utils
├── entities/          ← Shared domain models
├── components/        ← UI components (frontend)
├── libs/              ← Shared services (both)
├── pages/             ← Application pages (frontend)
└── routes/            ← API routes (backend)
```

Or split into separate backend/frontend folders:

```
backend/
├── core/
├── entities/
├── libs/
└── routes/

frontend/
├── core/
├── components/
├── pages/
└── libs/
```

---

## Migrating to This Structure

### From Scattered Folders

**Before** (bad structure):

```
src/
├── interfaces/
│   └── user.ts
├── services/
│   └── user.service.ts
├── controllers/
│   └── user.controller.ts
├── dto/
│   └── user.dto.ts
├── validators/
│   └── user.validator.ts
└── utils/
    └── user-helpers.ts
```

**After** (organized):

```
src/
├── core/
│   └── utils/
│       └── date-formatter.ts
├── entities/
│   └── user/
│       ├── user.ts
│       ├── user.repo.ts
│       ├── user.logic.ts
│       ├── user.mapper.ts
│       └── user.validator.ts
└── pages/
    └── user/
        ├── user-list.tsx
        └── user-detail.tsx
```

**Steps**:
1. Create `entities/user/` folder
2. Move all user-related files into it
3. Rename files to use consistent pattern: `user.{type}.ts`
4. Update import paths
5. Repeat for other entities

---

## Quick Checklist

When organizing your project:

- [ ] Is your project split into 4 levels: `core/`, `entities/`, `components/`, `pages/`?
- [ ] Is each entity in its own folder under `entities/`?
- [ ] Does each entity folder use singular naming?
- [ ] Do all files in an entity folder start with entity name?
- [ ] Is folder depth never more than 3 levels?
- [ ] Are `pages/` and `routes/` the only plural folders?
- [ ] Are shared utilities in `core/`, not scattered?
- [ ] Are reusable components in `components/`?
- [ ] Are cross-entity services in `libs/`?

---

## Related

- [Module Organization](module-organization.md)
- [File Type Suffixes](../02-naming-conventions/file-type-suffixes.md)
- [Files & Folders Naming](../02-naming-conventions/files-folders.md)
- [No Nesting Rule](no-nesting-rule.md)
