# Denormalization

When and how to denormalize data for performance, with full awareness of the tradeoffs.

## What is Denormalization?

Denormalization is storing redundant data to improve performance. It's intentionally breaking normal forms.

### Normalized (Standard)

```typescript
interface Post {
  id: number;
  title: string;
  authorId: number;
}

interface User {
  id: number;
  name: string;
  email: string;
}

// To get author's name, must join tables
const post = await postRepository.getById(1);
const author = await userRepository.getById(post.authorId);
console.log(author.name);
```

### Denormalized

```typescript
interface Post {
  id: number;
  title: string;
  authorId: number;
  authorName: string;     // ← Redundant, but avoids lookup
}

// Now have author's name directly
const post = await postRepository.getById(1);
console.log(post.authorName);  // No second query needed
```

## When to Denormalize

### Good Reasons (With Caution)

1. **Read-heavy queries** — Data is read much more often than written
   ```typescript
   // ✅ Reasonable: Author name rarely changes
   interface Post {
     authorId: number;
     authorName: string;  // Read 10x per second, written 1x per day
   }
   ```

2. **Expensive calculations** — Computing value is costly
   ```typescript
   // ✅ Reasonable: Computing total price involves many joins
   interface Order {
     items: OrderItem[];
     totalPrice: number;  // Cache calculated value
   }
   ```

3. **Display-only data** — Not used for business logic
   ```typescript
   // ✅ Reasonable: For UI, not for updates
   interface UserListItem {
     id: number;
     name: string;
     postCount: number;   // For display only
   }
   ```

### Bad Reasons (Don't Do This)

1. **"Just in case"** — No actual performance issue
   ```typescript
   // ❌ Bad: Premature optimization
   interface User {
     age: number;
     ageGroup: string;    // Not needed unless there's a perf problem
   }
   ```

2. **Lazy database design** — Avoiding proper schema
   ```typescript
   // ❌ Bad: Should be separate entities
   interface User {
     address: string;
     city: string;
     state: string;
     zipCode: string;     // Store as Address entity instead
   }
   ```

3. **Convenient but unconstrained** — Fields can get out of sync
   ```typescript
   // ❌ Bad: Caches can diverge from reality
   interface Post {
     content: string;
     contentHash: string; // Can become inconsistent
   }
   ```

## Denormalization Patterns

### 1. Caching Calculations

Store a computed value to avoid recalculating:

```typescript
interface Order {
  id: number;
  items: OrderItem[];
  totalPrice: number;       // ← Cached calculation
}

class OrderService {
  async updateOrderItem(orderId: number, itemId: number, quantity: number) {
    // 1. Update the item
    await this.updateItem(orderId, itemId, quantity);

    // 2. Recalculate total
    const items = await this.getOrderItems(orderId);
    const newTotal = items.reduce((sum, item) => sum + item.price, 0);

    // 3. Update cache
    await this.orderRepository.update(orderId, { totalPrice: newTotal });
  }
}
```

**Risks**: Total can become stale if update fails partway through.

**Mitigation**: Transactions, periodic recalculation, or validation.

### 2. Copying Reference Data

Store frequently-accessed reference data:

```typescript
interface Post {
  id: number;
  title: string;
  authorId: number;
  authorName: string;       // ← Copied from User
}

class PostService {
  async createPost(data: CreatePostDto): Promise<Post> {
    const author = await this.userRepository.getById(data.authorId);

    const post = {
      ...data,
      authorName: author.name  // ← Copy at creation time
    };

    return this.postRepository.save(post);
  }

  async updatePostAuthor(postId: number, newAuthorId: number) {
    const newAuthor = await this.userRepository.getById(newAuthorId);

    return this.postRepository.update(postId, {
      authorId: newAuthorId,
      authorName: newAuthor.name  // ← Update the copy
    });
  }
}
```

**Risks**: Names can go stale if user changes their name.

**Mitigation**: Update all denormalized copies when source changes, or use read-time resolution.

### 3. Count Caching

Store counts instead of fetching every time:

```typescript
interface User {
  id: number;
  name: string;
  postCount: number;        // ← Cached count
}

class UserService {
  async getUserProfile(userId: number) {
    // Fetch user and their post count, using cached value
    return this.userRepository.getById(userId);
  }

  async createPost(userId: number, postData: PostData) {
    // 1. Create post
    await this.postRepository.create({ ...postData, userId });

    // 2. Increment user's post count
    const user = await this.userRepository.getById(userId);
    await this.userRepository.update(userId, {
      postCount: user.postCount + 1
    });
  }

  async deletePost(postId: number, userId: number) {
    // 1. Delete post
    await this.postRepository.delete(postId);

    // 2. Decrement user's post count
    const user = await this.userRepository.getById(userId);
    await this.userRepository.update(userId, {
      postCount: Math.max(0, user.postCount - 1)
    });
  }
}
```

**Risks**: Count can become inconsistent if updates fail.

**Mitigation**: Transactions, regular audit jobs, or event-driven updates.

## Strategies to Keep Denormalized Data Consistent

### 1. Transactions

Keep normalized and denormalized data in sync:

```typescript
async function updatePost(postId: number, newAuthor: User) {
  return await transaction(async (trx) => {
    // Both updates happen atomically
    await trx('posts')
      .where('id', postId)
      .update({
        authorId: newAuthor.id,
        authorName: newAuthor.name
      });
  });
}
```

### 2. Events

Use domain events to update denormalized copies:

```typescript
class UserService {
  async updateUserName(userId: number, newName: string) {
    // 1. Update user
    const user = await this.userRepository.update(userId, {
      name: newName
    });

    // 2. Emit event
    this.eventBus.emit('user:name-changed', {
      userId: user.id,
      newName: user.name
    });

    return user;
  }
}

class PostService {
  constructor(eventBus: EventBus) {
    // Listen for user name changes
    eventBus.on('user:name-changed', async (event) => {
      // Update all posts by this user
      await this.postRepository.updateMany(
        { authorId: event.userId },
        { authorName: event.newName }
      );
    });
  }
}
```

### 3. Periodic Recalculation

Periodically recompute denormalized values:

```typescript
class AuditService {
  async recalculateUserCounts() {
    const users = await this.userRepository.getAll();

    for (const user of users) {
      const postCount = await this.postRepository.countByUserId(user.id);
      const commentCount = await this.commentRepository.countByUserId(user.id);

      await this.userRepository.update(user.id, {
        postCount,
        commentCount
      });
    }
  }
}

// Run daily
schedule.every('1 day').do(() => auditService.recalculateUserCounts());
```

### 4. Materialized Views

Use a separate table for denormalized data:

```sql
-- Base tables (normalized)
CREATE TABLE posts (
  id INTEGER PRIMARY KEY,
  title VARCHAR(255),
  authorId INTEGER REFERENCES users(id)
);

CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name VARCHAR(255)
);

-- Materialized view (denormalized, read-only)
CREATE TABLE post_summaries (
  id INTEGER PRIMARY KEY,
  title VARCHAR(255),
  authorName VARCHAR(255),
  createdAt DATETIME
);

-- Refresh the view periodically
REFRESH MATERIALIZED VIEW post_summaries;
```

## Decision Framework

Should you denormalize?

```
Does the data:
  - Get read much more than written?        → More likely
  - Require expensive joins to assemble?    → More likely
  - Change when source data changes?        → Less likely
  - Need absolute consistency?              → Less likely
  - Only used for display, not logic?       → More likely

If YES to more than 2 of the top ones:
  → Consider denormalization
  → But measure first! Don't guess
  → Implement sync strategy
```

## Example: Appropriate Denormalization

```typescript
interface BlogPost {
  id: number;
  title: string;
  content: string;
  authorId: number;
  authorName: string;           // ← OK: Author rarely changes name
  authorEmail: string;          // ← OK: Email rarely changes
  publishedAt: Date;
  viewCount: number;            // ← OK: Display only, no business logic
  upvoteCount: number;          // ← OK: Expensive to calculate each time
}
```

These are reasonable denormalizations because:
- Author name/email don't change frequently
- View/upvote counts don't affect business logic
- Benefits are clear (avoid expensive joins)
- Consistency can be maintained through events

---

## Related

- [Flat Entities](flat-entities.md)
- [Relationships & References](relationships.md)
- [DTO vs Domain Model](dto-vs-domain.md)
