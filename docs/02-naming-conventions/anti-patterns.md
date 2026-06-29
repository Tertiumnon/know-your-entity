# Common Naming Anti-Patterns

Learn from common mistakes that make code harder to maintain.

## 1. Type in the Name

**Problem**: Repeating type information in variable names adds noise and complicates refactoring.

```typescript
// BAD — Type is in the name
const ageNumber: number = 30;
const userArray: User[] = [];
const userListArray: User[] = [];
const isActiveBoolean: boolean = true;

// GOOD — Let types be in annotations
const age = 30;
const users: User[] = [];
const isActive: boolean = true;
```

**Why it's bad**: If you change `users` from `User[]` to `UserCollection`, the name `userListArray` becomes misleading.

## 2. Redundant Entity Names in Properties

**Problem**: Repeating the entity name in property names creates clutter without adding information.

```typescript
// BAD — Entity name repeated
class User {
  userId: number;
  userName: string;
  userEmail: string;
  userCreatedAt: Date;
}

// GOOD — Clean properties
class User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}
```

**Same problem in API responses:**

```json
// BAD
{
  "userName": "Bob",
  "userEmail": "bob@example.com",
  "userCreatedAt": "2025-01-01"
}

// GOOD
{
  "name": "Bob",
  "email": "bob@example.com",
  "createdAt": "2025-01-01"
}
```

## 3. Generic, Non-Descriptive Names

**Problem**: Names like `data`, `info`, `result`, `temp` don't convey meaning.

```typescript
// BAD — No context
const data = await fetch('/users');
const info = process.getInfo();
const tmp = 42;
const res = calculateSomething();

// GOOD — Specific and clear
const userResponse = await fetch('/users');
const userInfo = process.getUserInfo();
const maxRetries = 42;
const totalPrice = calculateOrderTotal();
```

## 4. Unclear Abbreviations

**Problem**: Making up abbreviations without explanation confuses team members.

```typescript
// BAD — What does 'upd' mean? Update? Upload?
function upd(item) { }
const usr = getUser();
const prd = product;
const calc_res = calculate();

// GOOD — Clear and explicit
function update(item) { }
const user = getUser();
const product = product;
const calculationResult = calculate();
```

## 5. Double Negatives

**Problem**: Negating a name makes conditions confusing.

```typescript
// BAD — Double negative, hard to read
const isNotActive = false;
if (!isNotActive) { }  // What does this mean?

const unsetUser = true;  // Confusing

// GOOD — Positive assertions
const isActive = true;
if (isActive) { }      // Clear

const user = getUser();
```

## 6. Inconsistent Verb Choices

**Problem**: Using different verbs for the same operation confuses the API.

```typescript
// BAD — Inconsistent across services
class UserService {
  find(filters): User[] { }       // One service uses 'find'
}

class ProductService {
  search(filters): Product[] { }  // Another uses 'search'
}

class OrderService {
  query(filters): Order[] { }     // A third uses 'query'
}

// GOOD — Consistent verb usage
class UserService {
  find(filters): User[] { }
}

class ProductService {
  find(filters): Product[] { }
}

class OrderService {
  find(filters): Order[] { }
}
```

## 7. Mixing Naming Conventions

**Problem**: Using different case styles in the same codebase causes inconsistency.

```typescript
// BAD — Mixed conventions
const user_name = 'Bob';      // snake_case
const userId = 42;            // camelCase
const ProductId = 5;          // PascalCase

const getUserName = () => { } // camelCase
const Get_User = () => { }    // Mixed

// GOOD — Consistent style
const userName = 'Bob';       // All camelCase
const userId = 42;
const productId = 5;

const getUserName = () => { }
const getUser = () => { }
```

## 8. Files Don't Match Content

**Problem**: File names don't correspond to the classes/components they export.

```
// BAD — File name doesn't match content
user-list.component.ts  → exports UserTableComponent
app-utils.ts            → only has date formatting
common.ts               → 50 unrelated utilities

// GOOD — File names match content
user-list.component.ts  → exports UserListComponent
date-formatter.ts       → date formatting functions
string-validator.ts     → string validation functions
```

## 9. Generic Folder Names at Root Level

**Problem**: Folders like `utils`, `helpers`, `common` become dumping grounds.

```
// BAD — Too generic
src/
  utils/
  common/
  helpers/
  shared/
  (Nobody knows what's in there until they look)

// GOOD — Specific folders
src/
  formatters/
    date-formatter.ts
    currency-formatter.ts
  validators/
    email-validator.ts
    password-validator.ts
  middleware/
    auth-middleware.ts
    error-middleware.ts
```

## 10. One-Word Generic Classes

**Problem**: Classes with vague, one-word names don't convey purpose.

```typescript
// BAD
class Data { }
class Model { }
class Resource { }
class Manager { }
class Helper { }
class Processor { }

// GOOD
class User { }
class Product { }
class OrderService { }
class EmailValidator { }
class DateFormatter { }
```

## Quick Fix Checklist

When reviewing naming, ask:

- [ ] Is the type information redundant in the name?
- [ ] Does the name repeat the entity/object name unnecessarily?
- [ ] Is the name generic or unclear? (`data`, `info`, `tmp`)
- [ ] Are abbreviations explained or obvious?
- [ ] Are there double negatives?
- [ ] Is the verb choice consistent with similar operations?
- [ ] Is the naming convention consistent across the project?
- [ ] Does the file name match its content?
- [ ] Are folder names specific and descriptive?
- [ ] Can a new team member understand what this is without looking at the code?

If you answered "yes" to any of these questions, consider refactoring the name.

---

## 11. Central/Barrel Exports (Hidden Maintenance Burden)

**Problem**: Central `index.ts` files that export everything from a directory create an invisible maintenance layer.

```typescript
// BAD — Central barrel (maintenance nightmare)
// src/entities/index.ts
export { User } from './user/user.interface';
export { UserService } from './user/user.service';
export { UserRepository } from './user/user.repository';
export { UserValidator } from './user/user.validator';
export { UserMapper } from './user/user.mapper';
// ... 50 more exports that must be kept in sync

// GOOD — Direct imports (zero maintenance)
import { UserService } from './entities/user/user.service';
```

**Why it's bad**:
- Dependencies are hidden (don't know where imports come from)
- Requires manual updates when files change
- Easy to forget to update the barrel
- Creates circular dependency risk
- Harms tree-shaking and bundle optimization

**Read more**: [Barrel Exports Antipattern](barrel-exports.md)

---

## 12. Generic File Type Suffixes (.service.ts, .handler.ts)

**Problem**: Using `.service.ts` or `.handler.ts` is too vague and doesn't tell you what the file actually does.

```typescript
// BAD — What does this "service" do?
// user.service.ts
export class UserService {
  async save() { }          // Database? Or business logic?
  async validateEmail() { } // Validation?
  toDto() { }               // Mapping?
  async cache() { }         // Caching?
  isAdmin() { }             // Guards?
}

// GOOD — Specific, clear purpose
// user.repo.ts
export class UserRepository { }    // Specifically database

// user.validator.ts
export class UserValidator { }     // Specifically validation

// user.mapper.ts
export class UserMapper { }        // Specifically transformation

// user.logic.ts
export class UserLogic { }         // Specifically business rules
```

**Why it's bad**:
- "Service" is too generic (could be anything)
- Hard to understand file responsibility
- Creates god objects (one class doing everything)
- Mixing layers (data access + validation + mapping)
- Violates single responsibility principle

**Better suffixes**:
- `.repo.ts` → Repository (database)
- `.validator.ts` → Validation logic
- `.mapper.ts` → DTO mapping
- `.logic.ts` → Business logic
- `.cache.ts` → Caching
- `.api.ts` → External API calls
- `.guard.ts` → Permission checks
- `.utils.ts` → Utility functions

**Read more**: [File Type Suffixes](file-type-suffixes.md)

---

## Related

- [Overview](overview.md)
- [Variables](variables.md)
- [Functions & Methods](functions-methods.md)
- [Classes & Interfaces](classes-interfaces.md)
- [Files & Folders](files-folders.md)
- [File Type Suffixes](file-type-suffixes.md)
- [Barrel Exports Antipattern](barrel-exports.md)
