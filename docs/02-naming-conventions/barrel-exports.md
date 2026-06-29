# Barrel Exports & Central Index Files — Antipattern

Barrel exports (central `index.ts` files that re-export everything) seem convenient but create hidden maintenance burden and obscure dependencies.

## The Problem

### What is a Barrel Export?

A barrel export is an `index.ts` file that imports and re-exports everything from a directory:

```typescript
// src/entities/index.ts — Central barrel export
export { User } from './user/user.interface';
export { UserService } from './user/user.service';
export { UserRepository } from './user/user.repository';
export { CreateUserDto, UpdateUserDto } from './user/user.dto';

export { Product } from './product/product.interface';
export { ProductService } from './product/product.service';
export { ProductRepository } from './product/product.repository';

export { Order } from './order/order.interface';
export { OrderService } from './order/order.service';
// ... 50 more exports
```

### Why It's an Antipattern

1. **Hidden Dependencies** — Import statements don't show actual file locations
   ```typescript
   // You import this way
   import { UserService } from '@/entities';

   // But the actual file is at
   // src/entities/user/user.service.ts

   // Find-and-replace becomes dangerous
   // IDE navigation is less accurate
   // New developers don't know where things come from
   ```

2. **Maintenance Burden** — Someone must remember to update the barrel
   ```typescript
   // You add a new service
   // src/entities/user/user.validator.ts
   // class UserValidator { }

   // You must remember to add to index.ts
   // If you forget, it's invisible to others
   // Review becomes a source of bugs
   ```

3. **Circular Dependency Risk** — Easy to create cycles
   ```typescript
   // user/index.ts exports everything
   // order/order.service.ts imports from user/index.ts
   // user/user.service.ts imports from order/index.ts
   // → Circular dependency, silent failure or build error
   ```

4. **Tree-Shaking Issues** — Bundlers can't optimize
   ```typescript
   // You only need UserService
   import { UserService } from '@/entities';

   // But barrel exports all 50+ items
   // Bundler may include unused code
   // Larger bundle size
   ```

5. **Cognitive Load** — What's exported vs what's internal?
   ```typescript
   // Is this exported from the barrel?
   // Or should I import from the specific file?
   // Inconsistency becomes a team problem
   ```

## Real-World Example: The Problem

### Initial Setup (Seems Fine)

```typescript
// src/entities/index.ts
export { User } from './user/user.interface';
export { UserService } from './user/user.service';
export { UserRepository } from './user/user.repository';

export { Product } from './product/product.interface';
export { ProductService } from './product/product.service';
```

Imports look clean:
```typescript
import { UserService, User } from '@/entities';
```

### 3 Months Later (The Problem Appears)

```typescript
// src/entities/index.ts has grown to 100 lines
export { User } from './user/user.interface';
export { UserService } from './user/user.service';
export { UserRepository } from './user/user.repository';
export { UserValidator } from './user/user.validator';  // Did someone add this?
export { UserMapper } from './user/user.mapper';
export { UserCache } from './user/user.cache';
export { UserPresenter } from './user/user.presenter';

export { Product } from './product/product.interface';
export { ProductService } from './product/product.service';
export { ProductRepository } from './product/product.repository';
export { ProductValidator } from './product/product.validator';
export { ProductMapper } from './product/product.mapper';
export { ProductCache } from './product/product.cache';
export { ProductPresenter } from './product/product.presenter';

// ... 50 more exports

// New team member adds UserCache but forgets to update index.ts
// Another developer can't find UserCache and implements it again
// Another PR updates UserService but the barrel isn't updated
// Silent bugs from outdated exports
```

## Better Alternatives

### Option 1: Direct Imports (Recommended)

Import directly from the specific file:

```typescript
// Instead of:
import { UserService } from '@/entities';

// Do this:
import { UserService } from '@/entities/user/user.service';

// Or for shorter paths in TypeScript:
import { UserService } from '@/entities/user';  // Via tsconfig path alias
```

**Advantages:**
- ✅ Explicit dependency path
- ✅ No maintenance needed
- ✅ IDE navigation works perfectly
- ✅ Tree-shaking optimization
- ✅ No circular dependency risk
- ✅ Easier to find/refactor

**Disadvantages:**
- Longer import paths (mitigate with path aliases)

### Option 2: Minimal, Focused Barrels

Only export primary/public API:

```typescript
// src/entities/user/index.ts — Small, focused
export { User } from './user.interface';
export { UserService } from './user.service';
// That's it! No internal exports

// src/entities/index.ts — Only top-level
export * from './user';   // Re-export from user barrel
export * from './product';
export * from './order';
```

**Advantages:**
- ✅ Cleaner than mega-barrel
- ✅ Clear public API separation
- ✅ Less maintenance than full barrel

**Disadvantages:**
- Still requires barrel files
- Still some indirection

### Option 3: Explicit Folder Imports (Best for monorepos)

Rely on file structure, not barrels:

```typescript
// File structure speaks for itself
src/
  entities/
    user/
      user.interface.ts         ← Import directly
      user.service.ts           ← Import directly
      user.repository.ts        ← Import directly

import { UserService } from '@/entities/user/user.service';
import { UserRepository } from '@/entities/user/user.repository';
```

**Advantages:**
- ✅ No barrel files needed
- ✅ Clear structure from filesystem
- ✅ No maintenance
- ✅ Perfect tree-shaking
- ✅ IDE knows exactly what's where

## When Barrels Are Acceptable

### 1. Library Exports (Public API)

For published libraries/packages, a barrel is good:

```typescript
// my-library/src/index.ts
// This IS the public API contract
export { User } from './entities/user';
export { UserService } from './services/user.service';
export { CreateUserDto } from './dto/create-user.dto';
```

✅ Acceptable because:
- Users import from the library, not internals
- Clear separation of public/private
- Defined API contract

### 2. Framework/App Entry Points

Exporting features/modules from an app is OK:

```typescript
// app/features/authentication/index.ts
export { LoginComponent } from './components/login';
export { AuthService } from './services/auth.service';
export { useAuth } from './hooks/use-auth';
```

✅ Acceptable because:
- Clear feature boundaries
- Intentional public API
- Not meant to be internal-only

## Checking for Barrel Antipattern

Use this checklist to catch problematic barrels:

```
Is this index.ts/barrel file:
  - [ ] Growing faster than individual files?
  - [ ] Exporting internal utilities not meant for outside use?
  - [ ] Frequently forgotten when new files are added?
  - [ ] Creating circular dependency issues?
  - [ ] Making import paths shorter, but reality harder?
  - [ ] Exported from parent barrel too (chain of barrels)?

If YES to any:
  → It's an antipattern
  → Refactor to direct imports
  → Remove the barrel
```

## Refactoring Strategy

### Phase 1: Identify

```bash
# Find all index.ts files
find src -name "index.ts" -type f | wc -l

# Check what's exported
cat src/entities/index.ts | grep "export"
```

### Phase 2: Update Imports

Replace barrel imports with direct imports:

```typescript
// Before
import { UserService, ProductService } from '@/entities';

// After
import { UserService } from '@/entities/user/user.service';
import { ProductService } from '@/entities/product/product.service';
```

### Phase 3: Remove Barrel

Delete `src/entities/index.ts` and any redundant barrels

### Phase 4: Update Path Aliases (Optional)

If paths are too long, use TypeScript path aliases:

```json
{
  "compilerOptions": {
    "paths": {
      "@/entities/*": ["src/entities/*"],
      "@/entities/user/*": ["src/entities/user/*"]
    }
  }
}
```

Then imports can be:
```typescript
import { UserService } from '@/entities/user/user.service';
```

---

## Quick Summary

| Approach | Maintenance | Dependencies | Tree-Shake | Refactor |
|----------|-------------|--------------|-----------|----------|
| **Mega Barrel** | ❌ High | ❌ Hidden | ❌ Poor | ❌ Hard |
| **Focused Barrel** | ⚠️ Medium | ⚠️ Some | ⚠️ Okay | ⚠️ Moderate |
| **Direct Imports** | ✅ None | ✅ Clear | ✅ Perfect | ✅ Easy |

**Recommendation**: Use direct imports. Only use barrels for intentional public APIs.

---

## Related

- [Files & Folders](files-folders.md)
- [Anti-Patterns](anti-patterns.md)
- [Module Organization](../06-architecture/) (when available)
