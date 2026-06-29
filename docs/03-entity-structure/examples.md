# Entity Structure Examples

Real-world examples of well-structured entities in different domains.

## E-Commerce Platform

### Domain Entities (Flat)

```typescript
// Core entities
interface User {
  id: number;
  email: string;
  firstName: string;
  lastName: string;
  passwordHash: string;
  createdAt: Date;
}

interface Product {
  id: number;
  sku: string;
  name: string;
  description: string;
  price: number;
  categoryId: number;
  supplierId: number;
  inStock: boolean;
  createdAt: Date;
}

interface Order {
  id: number;
  userId: number;
  status: 'pending' | 'processing' | 'shipped' | 'delivered';
  totalPrice: number;
  createdAt: Date;
  updatedAt: Date;
}

interface OrderItem {
  id: number;
  orderId: number;
  productId: number;
  quantity: number;
  unitPrice: number;
}

interface Category {
  id: number;
  name: string;
  parentCategoryId?: number;
}
```

### DTOs for API

```typescript
// Product listing
interface ProductListDto {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

// Product detail (with nested data)
interface ProductDetailDto {
  id: number;
  sku: string;
  name: string;
  description: string;
  price: number;
  inStock: boolean;
  category: {
    id: number;
    name: string;
  };
  supplier: {
    id: number;
    name: string;
    email: string;
  };
}

// Order response
interface OrderResponseDto {
  id: number;
  status: string;
  totalPrice: number;
  createdAt: string;
  items: {
    id: number;
    product: {
      id: number;
      name: string;
      price: number;
    };
    quantity: number;
  }[];
}

// Order creation
interface CreateOrderDto {
  items: {
    productId: number;
    quantity: number;
  }[];
}
```

---

## Blog Platform

### Domain Entities (Flat)

```typescript
interface User {
  id: number;
  email: string;
  username: string;
  passwordHash: string;
  bio?: string;
  avatarUrl?: string;
  createdAt: Date;
}

interface BlogPost {
  id: number;
  authorId: number;
  title: string;
  slug: string;
  content: string;
  published: boolean;
  viewCount: number;              // Denormalized for performance
  upvoteCount: number;            // Denormalized for performance
  createdAt: Date;
  updatedAt: Date;
  publishedAt?: Date;
}

interface Comment {
  id: number;
  postId: number;
  authorId: number;
  text: string;
  parentCommentId?: number;       // For threaded comments
  upvoteCount: number;
  createdAt: Date;
  updatedAt: Date;
}

interface Tag {
  id: number;
  name: string;
  slug: string;
}

interface PostTag {
  postId: number;
  tagId: number;
}
```

### DTOs for API

```typescript
// Blog post listing
interface PostListDto {
  id: number;
  title: string;
  slug: string;
  author: {
    id: number;
    username: string;
    avatarUrl?: string;
  };
  viewCount: number;
  upvoteCount: number;
  createdAt: string;
}

// Blog post detail
interface PostDetailDto {
  id: number;
  title: string;
  slug: string;
  content: string;
  author: {
    id: number;
    username: string;
    bio?: string;
    avatarUrl?: string;
  };
  tags: {
    id: number;
    name: string;
  }[];
  viewCount: number;
  upvoteCount: number;
  createdAt: string;
  publishedAt?: string;
}

// Comment with nested replies
interface CommentDto {
  id: number;
  text: string;
  author: {
    id: number;
    username: string;
  };
  upvoteCount: number;
  createdAt: string;
  replies: CommentDto[];
}
```

---

## Task Management System

### Domain Entities (Flat)

```typescript
interface User {
  id: number;
  email: string;
  name: string;
  passwordHash: string;
  role: 'admin' | 'manager' | 'user';
  createdAt: Date;
}

interface Project {
  id: number;
  ownerId: number;
  name: string;
  description?: string;
  status: 'active' | 'archived';
  createdAt: Date;
}

interface Task {
  id: number;
  projectId: number;
  title: string;
  description?: string;
  status: 'todo' | 'in_progress' | 'done';
  priority: 'low' | 'medium' | 'high';
  assigneeId?: number;
  dueDate?: Date;
  createdBy: number;
  createdAt: Date;
  updatedAt: Date;
}

interface TaskComment {
  id: number;
  taskId: number;
  authorId: number;
  text: string;
  createdAt: Date;
  updatedAt: Date;
}

interface ProjectMember {
  projectId: number;
  userId: number;
  role: 'owner' | 'member';
  joinedAt: Date;
}
```

### DTOs for API

```typescript
// Task list for project
interface TaskListDto {
  id: number;
  title: string;
  status: string;
  priority: string;
  assignee?: {
    id: number;
    name: string;
  };
  dueDate?: string;
}

// Task detail
interface TaskDetailDto {
  id: number;
  title: string;
  description?: string;
  status: string;
  priority: string;
  assignee?: {
    id: number;
    name: string;
    email: string;
  };
  createdBy: {
    id: number;
    name: string;
  };
  dueDate?: string;
  createdAt: string;
  updatedAt: string;
  comments: {
    id: number;
    text: string;
    author: {
      id: number;
      name: string;
    };
    createdAt: string;
  }[];
}

// Project overview
interface ProjectOverviewDto {
  id: number;
  name: string;
  description?: string;
  status: string;
  owner: {
    id: number;
    name: string;
  };
  taskStats: {
    total: number;
    done: number;
    inProgress: number;
    todo: number;
  };
  members: {
    id: number;
    name: string;
    role: string;
  }[];
}
```

---

## Social Network

### Domain Entities (Flat)

```typescript
interface User {
  id: number;
  email: string;
  username: string;
  displayName: string;
  bio?: string;
  avatarUrl?: string;
  followerCount: number;          // Denormalized
  followingCount: number;         // Denormalized
  createdAt: Date;
}

interface Post {
  id: number;
  authorId: number;
  content: string;
  likeCount: number;              // Denormalized
  commentCount: number;           // Denormalized
  createdAt: Date;
}

interface Comment {
  id: number;
  postId: number;
  authorId: number;
  content: string;
  likeCount: number;
  createdAt: Date;
}

interface Like {
  id: number;
  userId: number;
  targetType: 'post' | 'comment';
  targetId: number;
  createdAt: Date;
}

interface Follow {
  followerId: number;
  followingId: number;
  createdAt: Date;
}
```

### DTOs for API

```typescript
// Feed post
interface FeedPostDto {
  id: number;
  author: {
    id: number;
    username: string;
    displayName: string;
    avatarUrl?: string;
  };
  content: string;
  likeCount: number;
  commentCount: number;
  isLikedByMe: boolean;
  createdAt: string;
}

// Post with comments
interface PostWithCommentsDto {
  id: number;
  author: {
    id: number;
    username: string;
    displayName: string;
    avatarUrl?: string;
  };
  content: string;
  likeCount: number;
  isLikedByMe: boolean;
  createdAt: string;
  comments: {
    id: number;
    author: {
      id: number;
      username: string;
      avatarUrl?: string;
    };
    content: string;
    likeCount: number;
    isLikedByMe: boolean;
    createdAt: string;
  }[];
}

// User profile
interface UserProfileDto {
  id: number;
  username: string;
  displayName: string;
  bio?: string;
  avatarUrl?: string;
  followerCount: number;
  followingCount: number;
  isFollowedByMe: boolean;
  postCount: number;
  createdAt: string;
}
```

---

## Key Patterns Across Examples

1. **Flat domain entities** — All have primitive fields or IDs only
2. **Denormalized counts** — Used for display/performance, not business logic
3. **DTOs have nesting** — API responses nest related data for convenience
4. **IDs for relationships** — Never embed full objects in entities
5. **Enum values** — Status, role, priority stored as strings/enums
6. **Timestamps** — Created and updated times on all entities
7. **Optional fields** — Marked with `?` to show they may be null

---

## Related

- [Flat Entities](flat-entities.md)
- [DTO vs Domain Model](dto-vs-domain.md)
- [Relationships & References](relationships.md)
- [Denormalization](denormalization.md)
