# Common Interfaces & Shared Types

Core interfaces that should be reused across your entities and domain models instead of repeating the same fields.

## The Problem

Entities often repeat the same fields and patterns. Without shared interfaces, you end up with duplication and inconsistency:

```typescript
// BAD — Duplicated timestamp fields
interface User {
  id: number;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Product {
  id: number;
  name: string;
  price: number;
  createdAt: Date;    // ← Repeated
  updatedAt: Date;    // ← Repeated
}

interface Order {
  id: number;
  userId: number;
  total: number;
  createdAt: Date;    // ← Repeated again
  updatedAt: Date;    // ← Repeated again
}

// When you change timestamp format, you update 3+ places
// When new entity is created, developer forgets timestamps
// Inconsistency across the codebase
```

## Solution: Core Interfaces

Define common patterns once, reuse everywhere:

```typescript
// GOOD — Define once, reuse everywhere
interface Timestamps {
  createdAt: Date;
  updatedAt: Date;
}

interface Identifiable {
  id: number;
}

interface User extends Identifiable, Timestamps {
  name: string;
  email: string;
}

interface Product extends Identifiable, Timestamps {
  name: string;
  price: number;
  categoryId: number;
}

interface Order extends Identifiable, Timestamps {
  userId: number;
  total: number;
}

// Consistent, maintainable, DRY
```

## Common Core Interfaces

### 1. Timestamps

Used by almost every entity that persists data:

```typescript
interface Timestamps {
  createdAt: Date;
  updatedAt: Date;
}

interface SoftDeletable {
  deletedAt?: Date;
}

// Combined
interface AuditFields extends Timestamps, SoftDeletable {
  deletedAt?: Date;
  deletedBy?: number;      // User who deleted
  reason?: string;         // Why it was deleted
}
```

**Example usage:**

```typescript
interface User extends Identifiable, Timestamps {
  name: string;
  email: string;
}

interface Order extends Identifiable, Timestamps {
  userId: number;
  total: number;
}

interface Post extends Identifiable, Timestamps {
  authorId: number;
  title: string;
  content: string;
}
```

### 2. Identifiable

Every entity should have an ID:

```typescript
interface Identifiable {
  id: number;
}

interface StringIdentifiable {
  id: string;
}

interface UUIDIdentifiable {
  id: UUID;
}
```

**Example usage:**

```typescript
interface User extends Identifiable {
  name: string;
}

interface ApiResponse<T extends Identifiable> {
  items: T[];
  total: number;
}
```

### 3. Versionable

For entities that track changes:

```typescript
interface Versionable {
  version: number;      // Optimistic locking
}

interface Revisioned {
  revision: number;
  revisionHistory?: RevisionEntry[];
}
```

**Example usage:**

```typescript
interface Document extends Identifiable, Timestamps, Versionable {
  title: string;
  content: string;
  version: number;  // For conflict resolution
}
```

### 4. Activatable / Status

For entities that have activation states:

```typescript
interface Activatable {
  isActive: boolean;
}

interface Statusable {
  status: string;
}

interface StatusableEnum {
  status: OrderStatus | UserStatus | PaymentStatus;
}
```

**Example usage:**

```typescript
enum OrderStatus {
  PENDING = 'PENDING',
  PROCESSING = 'PROCESSING',
  SHIPPED = 'SHIPPED',
  DELIVERED = 'DELIVERED'
}

interface Order extends Identifiable, Timestamps {
  status: OrderStatus;
  userId: number;
  total: number;
}

interface User extends Identifiable, Timestamps, Activatable {
  name: string;
  email: string;
  isActive: boolean;
}
```

### 5. Ownable / Authorable

For entities with ownership:

```typescript
interface Ownable {
  ownerId: number;
}

interface Authorable {
  authorId: number;
  authorName?: string;    // Denormalized for convenience
}

interface CreatedBy {
  createdBy: number;      // Who created it
  lastModifiedBy?: number; // Who last modified it
}
```

**Example usage:**

```typescript
interface Document extends Identifiable, Timestamps, Ownable, CreatedBy {
  title: string;
  content: string;
  ownerId: number;
  createdBy: number;
}

interface BlogPost extends Identifiable, Timestamps, Authorable {
  title: string;
  content: string;
  authorId: number;
}
```

### 6. Nestable (Hierarchical)

For entities with parent-child relationships:

```typescript
interface Nestable {
  parentId?: number;
}

interface Hierarchical extends Nestable {
  parentId?: number;
  level: number;
  path: string;  // e.g., "/1/2/3" for breadcrumb
}
```

**Example usage:**

```typescript
interface Category extends Identifiable, Nestable {
  name: string;
  parentId?: number;  // Parent category
}

interface Comment extends Identifiable, Timestamps, Nestable {
  text: string;
  authorId: number;
  parentId?: number;  // Parent comment (threaded)
}
```

### 7. Publishable

For content entities:

```typescript
interface Publishable {
  published: boolean;
  publishedAt?: Date;
}

interface DraftAware {
  isDraft: boolean;
  status: 'draft' | 'review' | 'published';
}
```

**Example usage:**

```typescript
interface BlogPost extends Identifiable, Timestamps, Publishable, Authorable {
  title: string;
  content: string;
  authorId: number;
  published: boolean;
  publishedAt?: Date;
}
```

### 8. Taggable / Categorizable

For entities with tags or categories:

```typescript
interface Taggable {
  tags: string[];
}

interface Tagged {
  tags: Tag[];
}

interface Categorizable {
  categoryId: number;
}
```

**Example usage:**

```typescript
interface BlogPost extends Identifiable, Timestamps, Taggable {
  title: string;
  content: string;
  tags: string[];  // ["typescript", "architecture"]
}

interface Product extends Identifiable, Timestamps, Categorizable {
  name: string;
  categoryId: number;
}
```

### 9. Countable

For denormalized counts:

```typescript
interface Countable {
  viewCount: number;
  likeCount: number;
  commentCount: number;
}

interface WithStats {
  totalViews: number;
  totalLikes: number;
  totalShares: number;
}
```

**Example usage:**

```typescript
interface BlogPost extends Identifiable, Timestamps, Countable {
  title: string;
  viewCount: number;      // Denormalized count
  likeCount: number;      // Denormalized count
  commentCount: number;   // Denormalized count
}
```

### 10. Sluggable

For entities with URL-friendly slugs:

```typescript
interface Sluggable {
  slug: string;
}

interface Slugged {
  slug: string;
  slugUpdatedAt?: Date;
}
```

**Example usage:**

```typescript
interface BlogPost extends Identifiable, Timestamps, Sluggable {
  title: string;
  slug: string;  // "my-blog-post"
  content: string;
}
```

## Complete Example: Rich Entity

Combining multiple interfaces:

```typescript
interface User
  extends Identifiable,
           Timestamps,
           Activatable,
           Statusable {
  id: number;
  email: string;
  name: string;
  isActive: boolean;
  status: 'active' | 'suspended' | 'deleted';
  createdAt: Date;
  updatedAt: Date;
}

interface BlogPost
  extends Identifiable,
          Timestamps,
          Authorable,
          Publishable,
          Sluggable,
          Taggable,
          Countable {
  id: number;
  title: string;
  content: string;
  authorId: number;
  authorName: string;
  published: boolean;
  publishedAt?: Date;
  slug: string;
  tags: string[];
  viewCount: number;
  likeCount: number;
  commentCount: number;
  createdAt: Date;
  updatedAt: Date;
}

interface Comment
  extends Identifiable,
          Timestamps,
          Authorable,
          Nestable,
          Countable {
  id: number;
  text: string;
  authorId: number;
  authorName: string;
  parentId?: number;
  likeCount: number;
  createdAt: Date;
  updatedAt: Date;
}
```

## Organization

Where to keep these shared interfaces:

### Option 1: Separate types file

```
src/
  types/
    core-interfaces.ts    ← All common interfaces
    common-types.ts
```

### Option 2: Shared folder

```
src/
  shared/
    interfaces/
      timestamps.ts
      identifiable.ts
      timestamps.ts
      publishable.ts
```

### Option 3: By category

```
src/
  types/
    audit-interfaces.ts   ← createdAt, updatedAt, deletedAt
    ownership-interfaces.ts ← ownerId, authorId, createdBy
    status-interfaces.ts  ← status, isActive
    content-interfaces.ts ← slug, tags, published
```

## Best Practices

### ✅ Do

1. **Define once, reuse everywhere**
   ```typescript
   interface Timestamps { createdAt: Date; updatedAt: Date; }
   interface User extends Timestamps { }
   interface Product extends Timestamps { }
   ```

2. **Compose interfaces for flexibility**
   ```typescript
   interface BlogPost extends Identifiable, Timestamps, Publishable, Taggable { }
   ```

3. **Use meaningful names**
   ```typescript
   interface Timestamps { }       // ✅ Clear
   interface TimeFields { }       // ⚠️ Less clear
   interface DateFields { }       // ❌ Too generic
   ```

4. **Document the purpose**
   ```typescript
   /**
    * Timestamps are automatically managed by the database
    * Use for audit trails and sorting by recency
    */
   interface Timestamps {
     createdAt: Date;
     updatedAt: Date;
   }
   ```

### ❌ Don't

1. **Don't create mega-interfaces with everything**
   ```typescript
   // BAD — Not every entity needs all these
   interface BaseEntity
     extends Identifiable,
             Timestamps,
             Activatable,
             Publishable,
             Taggable,
             Countable,
             Nestable { }
   ```

2. **Don't duplicate fields across interfaces**
   ```typescript
   // BAD — createdAt in two places
   interface Timestamps { createdAt: Date; }
   interface BaseFields { createdAt: Date; updatedAt: Date; }
   ```

3. **Don't mix concerns**
   ```typescript
   // BAD — Mixing business logic with structure
   interface Product {
     id: number;
     calculateDiscount(): number;  // Methods don't belong in data interfaces
   }
   ```

## Type-Safe Collections

Use interfaces for type-safe collections:

```typescript
// Generic collection interface
interface Entity extends Identifiable {
  // Base interface for all entities
}

interface CollectionResult<T extends Entity> {
  items: T[];
  total: number;
  page: number;
  limit: number;
}

interface Service<T extends Entity> {
  find(filters?: any): Promise<CollectionResult<T>>;
  get(id: number): Promise<T>;
  create(data: Partial<T>): Promise<T>;
  update(id: number, data: Partial<T>): Promise<T>;
  delete(id: number): Promise<void>;
}

// Usage
const userResult = await userService.find();
// userResult.items is User[]
// Type-safe!
```

---

## Quick Checklist

When creating a new entity:

- [ ] Does it have an ID? Use `Identifiable`
- [ ] Does it persist? Use `Timestamps`
- [ ] Can it be deleted? Use `SoftDeletable`
- [ ] Can it be active/inactive? Use `Activatable`
- [ ] Does it have status? Use `Statusable`
- [ ] Does it have an owner? Use `Ownable` or `Authorable`
- [ ] Can it have children? Use `Nestable`
- [ ] Is it publishable? Use `Publishable`
- [ ] Does it have tags? Use `Taggable`
- [ ] Does it have counts? Use `Countable`
- [ ] Does it have a URL slug? Use `Sluggable`

---

## Related

- [Flat Entities](flat-entities.md)
- [DTO vs Domain Model](dto-vs-domain.md)
- [Relationships & References](relationships.md)
- [Examples](examples.md)
