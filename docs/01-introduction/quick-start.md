# 5-Minute Quick Start

Essentials of entity-driven development in one page.

---

## Core Rules

### 1. Name Things by What They Are, Not How

```typescript
// ❌ BAD (encodes implementation)
const ageNumber = 30;
const userListArray: User[] = [];
const isActiveBoolean = true;

// ✅ GOOD (describes what it is)
const age = 30;
const users: User[] = [];
const isActive = true;
```

### 2. Keep Domain Entities Flat

```typescript
// ❌ BAD (nested in domain model)
interface User {
  id: number;
  profile: {
    age: number;
    bio: string;
  };
}

// ✅ GOOD (flat domain model)
interface User {
  id: number;
  age: number;
  bio: string;
}
```

Note: Nesting is OK in DTOs for API responses (max 1-2 levels).

### 3. Use IDs for References, Not Full Objects

```typescript
// ❌ BAD (embedding entire object)
interface Order {
  userId: {
    id: number;
    name: string;
    email: string;
  };
}

// ✅ GOOD (reference by ID)
interface Order {
  id: number;
  userId: number;
}
```

### 4. Use Clear Verbs for Methods

| Verb | Meaning | Return Type |
|------|---------|-------------|
| `get()` | Synchronous retrieval or error | Throws if not found |
| `find()` | Search with optional results | Returns array or null |
| `fetch()` | Async network request | Promise<Entity[]> |
| `create()` | Create new entity | Promise<Entity> |
| `update()` | Modify existing entity | Promise<Entity> |
| `delete()` | Remove entity | Promise<boolean> |

### 5. Avoid Repeating Entity Name in Properties

```typescript
// ❌ BAD (redundant prefixes)
class User {
  userId: number;
  userName: string;
  userEmail: string;
}

// ✅ GOOD (names are already in class)
class User {
  id: number;
  name: string;
  email: string;
}
```

---

## Architecture Structure

```
src/
├── core/                      # Shared utilities
│   ├── http/
│   └── router/
│
├── entities/                  # Business logic by entity
│   ├── user/
│   │   ├── user.interface.ts
│   │   ├── user.service.ts
│   │   ├── user.service.spec.ts
│   │   ├── user.mockup.ts
│   │   └── user.constant.ts
│   ├── product/
│   └── order/
│
├── components/                # Reusable UI components
│   ├── user-card/
│   ├── product-list/
│   └── navbar/
│
└── pages/                     # Full-page components
    ├── home/
    ├── products/
    └── checkout/
```

**Rules:**
- Max 3 folder levels deep
- Tests, mocks, constants live with their code
- Each entity in its own folder
- Clear separation: core → entities → components → pages

---

## Service Pattern

Every entity has a service with standard operations:

```typescript
class UserService {
  // Find multiple (returns empty array if none)
  async find(filters?: Filter[]): Promise<User[]> {
    // ...
  }

  // Get one (throws if not found)
  async get(id: number): Promise<User> {
    const user = await this.find({ id });
    if (!user) throw new Error(`User ${id} not found`);
    return user;
  }

  // Create
  async create(dto: UserCreateDto): Promise<User> {
    // ...
  }

  // Update
  async update(id: number, dto: UserUpdateDto): Promise<User> {
    // ...
  }

  // Delete
  async delete(id: number): Promise<boolean> {
    // ...
    return true;
  }
}
```

---

## REST API Patterns

### Standard CRUD Endpoints

```
GET    /user              → Get all users (UserService.find())
GET    /user/:id          → Get one user (UserService.get(id))
POST   /user              → Create user (UserService.create(dto))
PATCH  /user/:id          → Update user (UserService.update(id, dto))
DELETE /user/:id          → Delete user (UserService.delete(id))
```

### Response DTOs (OK to nest slightly)

```typescript
// Entity (flat)
interface User {
  id: number;
  name: string;
  email: string;
}

// Create DTO
interface UserCreateDto {
  name: string;
  email: string;
  password: string;
}

// Update DTO (partial)
interface UserUpdateDto {
  name?: string;
  email?: string;
}

// Response DTO (no password)
interface UserResponseDto {
  id: number;
  name: string;
  email: string;
}

// Aggregated response (OK with 1 level nesting for convenience)
interface ProfileResponseDto {
  user: UserResponseDto;
  address: AddressDto;
  preferences: PreferencesDto;
}
```

---

## Naming Quick Reference

| Item | Pattern | Example |
|------|---------|---------|
| **Entity** | PascalCase | `User`, `Product`, `Order` |
| **Entity file** | kebab-case | `user.interface.ts` |
| **Service** | PascalCase + Service | `UserService` |
| **Service file** | kebab-case | `user.service.ts` |
| **Variable** | camelCase | `user`, `products`, `orderId` |
| **Function** | camelCase + verb | `getUser()`, `createOrder()` |
| **Constant** | UPPER_SNAKE_CASE | `MAX_NAME_LENGTH`, `API_BASE_URL` |
| **Folder** | kebab-case | `user-card/`, `order-list/` |
| **CSS class** | kebab-case + BEM-like | `product-card`, `product-card__title` |

---

## Common Anti-Patterns

| ❌ Wrong | ✅ Right | Why |
|---------|---------|-----|
| `const userArray: User[]` | `const users: User[]` | Type is obvious from variable |
| `user.userId` | `user.id` | No need to repeat entity name |
| `fetchCurrentUser()` → sync | `getCurrentUser()` → sync | "fetch" implies async/network |
| `/api/users/getAll` | `/api/user` | Standard REST convention |
| Nested entities in domain | Flat domain + separate DTOs | Easier to update and maintain |
| No tests collocated | Tests with code | Easier to navigate |
| `const tmp = ...` | `const processedData = ...` | Clarity for future readers |

---

## File Organization Example

For User entity:

```
src/entities/user/
├── user.interface.ts          # Entity definition
├── user.dto.ts                # DTO definitions
├── user.service.ts            # Business logic
├── user.service.spec.ts       # Tests
├── user.mockup.ts             # Mock data for testing
└── user.constant.ts           # Constants
```

For UserCard component:

```
src/components/user-card/
├── user-card.component.ts     # Component logic
├── user-card.component.spec.ts # Tests
├── user-card.html             # Template
├── user-card.css              # Styles
└── user-card.constant.ts      # Component-specific constants (if needed)
```

---

## Checklist

Before you commit, verify:

**Naming**
- [ ] No type in variable names (`ageNumber`, `userArray`)
- [ ] No entity prefix in properties (`user.id` not `user.userId`)
- [ ] Methods use clear verbs (`get`, `find`, `fetch`, `create`)
- [ ] File names match content
- [ ] Consistent casing throughout

**Structure**
- [ ] Entity is flat (no nesting in domain model)
- [ ] References use IDs, not full objects
- [ ] Service has standard operations (find, get, create, update, delete)
- [ ] Tests/mocks/constants collocated with code
- [ ] Max 3 folder levels

**API**
- [ ] Endpoints follow REST convention (GET, POST, PATCH, DELETE)
- [ ] Response uses DTO (no password, sensitive data)
- [ ] Error responses are consistent

---

## Learn More

- **Full guide to entities**: [What is an Entity?](what-is-entity.md)
- **Naming examples**: [../02-naming-conventions/](../02-naming-conventions/)
- **Architecture deep dive**: [../06-architecture/](../06-architecture/)
- **Working code**: [entity-driven-architecture](https://github.com/Tertium/entity-driven-architecture)
