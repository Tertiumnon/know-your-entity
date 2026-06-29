# Flat Entities — Core Principle

The most important rule for entity design: **keep domain models flat**.

## The Rule

Your core domain entities should be **strictly flat** — their fields should be primitives or identifiers of other entities, never nested objects containing full models of other entities.

## Why Flatten?

Flat entities provide:

- **Simple updates** — Change one field without touching related data
- **Clear boundaries** — Easy to see what belongs to each entity
- **Easier validation** — Each field has independent rules
- **Flexible querying** — No need to fetch nested objects
- **Clean migrations** — Schema changes are straightforward
- **No confusion** — One source of truth for each entity

## Nested vs Flat

### ❌ Bad: Nested Domain Model

```typescript
interface User {
  id: number;
  name: string;
  profile: {           // ← Nested object
    age: number;
    bio: string;
  };
  contact: {           // ← Nested object
    email: string;
    phone: string;
  };
}
```

**Problems**:
- Hard to update just the email without touching all contact fields
- If contact is reused elsewhere, you have duplication
- Unclear: Is contact always fetched with user? Is it optional?
- Validation logic for contact is buried inside User

### ✅ Good: Flat Domain Model

```typescript
interface User {
  id: number;
  name: string;
  age: number;
  bio: string;
  email: string;
  phone: string;
}
```

**Benefits**:
- Each field is independent
- Clear what the User entity contains
- Update any field without touching others

### ✅ Also Good: Separate Entities

When related data is conceptually distinct or reused, extract it:

```typescript
interface User {
  id: number;
  name: string;
  profileId: number;    // ← Reference, not nested object
  contactId: number;    // ← Reference, not nested object
}

interface UserProfile {
  id: number;
  userId: number;
  age: number;
  bio: string;
}

interface UserContact {
  id: number;
  userId: number;
  email: string;
  phone: string;
}
```

**Benefits**:
- User profile and contact can be updated independently
- Can be reused if needed
- Clear relationships via IDs
- Easier to query — fetch profile only if needed

## The Nesting Spectrum

| Nesting Level | Use Case | Example |
|---------------|----------|---------|
| **0 levels (Flat)** | Domain entities | `User { id, name, email }` |
| **1 level (DTO)** | API responses | `UserDto { user: { name }, profile: { age } }` |
| **2 levels (DTO)** | Complex API responses | `Order { items: [{ product: { name } }] }` |
| **3+ levels** | ❌ Avoid everywhere | Never good idea |

## Domain Model vs DTO

### Domain Model (Flat, in Database)

```typescript
// In database: User table
interface User {
  id: number;
  name: string;
  email: string;
  profileId: number;
}

// In database: UserProfile table
interface UserProfile {
  id: number;
  userId: number;
  bio: string;
}
```

### DTO (Can be nested, for API responses)

```typescript
// Response from API — can have 1-2 levels of nesting
interface UserResponseDto {
  id: number;
  name: string;
  email: string;
  profile: {           // ← 1 level of nesting is OK in DTO
    bio: string;
  };
}
```

The key difference: **DTOs can nest** (for client convenience), but **domain entities should stay flat**.

## When to Nest in DTOs

Nesting in API responses is acceptable when:

1. **Client convenience** — The UI wants related data together
2. **Reducing requests** — Avoids multiple API calls
3. **1-2 levels maximum** — Don't nest deeply

Example:

```json
// GET /posts/{id}
{
  "id": 1,
  "title": "My Post",
  "author": {              // ← 1 level of nesting, OK in DTO
    "id": 2,
    "name": "Alice"
  },
  "comments": [
    {
      "id": 10,
      "text": "Great!",
      "author": {          // ← 2 levels total, still acceptable
        "id": 3,
        "name": "Bob"
      }
    }
  ]
}
```

But the underlying domain entities remain flat:

```typescript
// Domain: Flat entities
interface Post {
  id: number;
  title: string;
  authorId: number;       // ← ID, not nested object
}

interface Comment {
  id: number;
  postId: number;         // ← ID
  authorId: number;       // ← ID
  text: string;
}

interface User {
  id: number;
  name: string;
}
```

## Rules of Thumb

### Rule 1: No Nested Objects in Domain Entities

```typescript
// ❌ Bad
class User {
  address: { city: string; zip: string };
}

// ✅ Good
class User {
  addressId: number;
}
```

### Rule 2: 1-2 Levels OK in DTOs

```typescript
// ✅ Good DTO with nesting
interface PostWithAuthor {
  id: number;
  title: string;
  author: {           // ← 1 level
    id: number;
    name: string;
  };
}

// ⚠️ Avoid 3+ levels even in DTOs
interface DeepNesting {
  post: {
    comments: {
      author: {      // ← Too deep
        profile: {
          // ...
        }
      }
    }
  };
}
```

### Rule 3: Extract Reused Concepts

If something appears in multiple entities, extract it:

```typescript
// ❌ Bad: Address duplicated
interface User {
  name: string;
  address: { city: string; zip: string };
}

interface Company {
  name: string;
  address: { city: string; zip: string };  // ← Duplicated
}

// ✅ Good: Separate Address entity
interface Address {
  id: number;
  city: string;
  zip: string;
}

interface User {
  name: string;
  addressId: number;
}

interface Company {
  name: string;
  addressId: number;
}
```

## Examples

### E-Commerce Product

```typescript
// ❌ Bad: Too much nesting
interface Product {
  id: number;
  name: string;
  category: {
    id: number;
    name: string;
  };
  supplier: {
    id: number;
    name: string;
    contact: {
      email: string;
      phone: string;
    };
  };
  inventory: {
    warehouse1: number;
    warehouse2: number;
  };
}

// ✅ Good: Flat with references
interface Product {
  id: number;
  name: string;
  categoryId: number;
  supplierId: number;
}

interface Supplier {
  id: number;
  name: string;
  email: string;
  phone: string;
}

interface ProductInventory {
  id: number;
  productId: number;
  warehouseId: number;
  quantity: number;
}
```

### Blog Post

```typescript
// ❌ Bad: Nested in domain model
interface BlogPost {
  id: number;
  title: string;
  author: {
    id: number;
    name: string;
    email: string;
  };
  comments: {
    id: number;
    text: string;
    author: {
      name: string;
    };
  }[];
}

// ✅ Good: Flat domain entities
interface BlogPost {
  id: number;
  title: string;
  authorId: number;
  createdAt: Date;
}

interface Comment {
  id: number;
  postId: number;
  authorId: number;
  text: string;
  createdAt: Date;
}

interface User {
  id: number;
  name: string;
  email: string;
}

// ✅ DTO for API can nest (1-2 levels)
interface BlogPostDto {
  id: number;
  title: string;
  author: {          // ← 1 level
    id: number;
    name: string;
  };
  comments: [
    {
      id: number;
      text: string;
      author: {      // ← 2 levels total
        name: string;
      };
    }
  ];
}
```

---

## Quick Checklist

When designing a domain entity:

- [ ] Are all fields primitives or IDs?
- [ ] No nested objects?
- [ ] Related data is separate entity or DTO?
- [ ] Could fields be updated independently?
- [ ] Is there a clear reason for each field?
- [ ] Does it map to a real domain concept?

---

## Related

- [DTO vs Domain Model](dto-vs-domain.md)
- [Relationships & References](relationships.md)
- [Denormalization](denormalization.md)
- [Examples](examples.md)
