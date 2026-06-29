# Relationships & References

How to represent relationships between entities using IDs instead of nested objects.

## Core Principle

In domain entities, use **IDs and foreign keys** to reference other entities, not nested objects.

```typescript
// ❌ Bad: Nested object
class Order {
  customer: Customer;  // Nested object
}

// ✅ Good: Reference via ID
class Order {
  customerId: number;  // Just the ID
}
```

## One-to-One Relationships

One entity relates to exactly one other entity.

### Example: User and UserProfile

```typescript
// Domain entities
interface User {
  id: number;
  email: string;
  profileId: number;   // ← Reference to profile
}

interface UserProfile {
  id: number;
  userId: number;      // ← Reference back to user
  bio: string;
  avatarUrl: string;
}
```

### In the Database

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  email VARCHAR(255),
  profileId INTEGER UNIQUE REFERENCES user_profiles(id)
);

CREATE TABLE user_profiles (
  id INTEGER PRIMARY KEY,
  userId INTEGER UNIQUE REFERENCES users(id),
  bio TEXT,
  avatarUrl VARCHAR(255)
);
```

### Loading Related Data

When you need both, explicitly fetch them:

```typescript
class UserService {
  async getUserWithProfile(userId: number) {
    const user = await this.userRepository.getById(userId);
    const profile = await this.profileRepository.getByUserId(userId);

    return {
      user,
      profile
    };
  }
}
```

## One-to-Many Relationships

One entity has many related entities.

### Example: User and Orders

```typescript
// Domain entities
interface User {
  id: number;
  name: string;
  // ← No 'orders' array here!
}

interface Order {
  id: number;
  userId: number;      // ← Reference to user
  total: number;
  createdAt: Date;
}
```

### In the Database

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name VARCHAR(255)
);

CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  userId INTEGER REFERENCES users(id),
  total DECIMAL(10, 2),
  createdAt DATETIME
);
```

### Loading Related Data

```typescript
class OrderService {
  async getOrdersByUser(userId: number): Promise<Order[]> {
    return this.orderRepository.findByUserId(userId);
  }
}
```

### In API Responses (DTOs Can Nest)

```typescript
interface UserWithOrdersDto {
  id: number;
  name: string;
  orders: {            // ← Can nest in DTO
    id: number;
    total: number;
    createdAt: string;
  }[];
}
```

## Many-to-Many Relationships

Multiple entities can relate to multiple other entities.

### Example: Students and Courses

```typescript
// Domain entities
interface Student {
  id: number;
  name: string;
  // ← No courses list
}

interface Course {
  id: number;
  name: string;
  // ← No students list
}

// Join table entity
interface Enrollment {
  id: number;
  studentId: number;
  courseId: number;
  enrolledAt: Date;
}
```

### In the Database

```sql
CREATE TABLE students (
  id INTEGER PRIMARY KEY,
  name VARCHAR(255)
);

CREATE TABLE courses (
  id INTEGER PRIMARY KEY,
  name VARCHAR(255)
);

CREATE TABLE enrollments (
  id INTEGER PRIMARY KEY,
  studentId INTEGER REFERENCES students(id),
  courseId INTEGER REFERENCES courses(id),
  enrolledAt DATETIME
);
```

### Querying Many-to-Many

```typescript
class EnrollmentService {
  async getStudentCourses(studentId: number): Promise<Course[]> {
    const enrollments = await this.enrollmentRepository
      .findByStudentId(studentId);

    const courseIds = enrollments.map(e => e.courseId);
    return this.courseRepository.findByIds(courseIds);
  }

  async getCoursesWithStudents(courseId: number): Promise<StudentDto[]> {
    const enrollments = await this.enrollmentRepository
      .findByCourseId(courseId);

    const studentIds = enrollments.map(e => e.studentId);
    return this.studentRepository.findByIds(studentIds);
  }
}
```

## Circular References

When two entities reference each other, handle carefully:

```typescript
// Domain entities
interface Author {
  id: number;
  name: string;
  // ← Don't include books here
}

interface Book {
  id: number;
  title: string;
  authorId: number;    // ← Reference to author
}
```

### In Responses (DTO Can Resolve)

```typescript
// DTO for book with author details
interface BookDetailedDto {
  id: number;
  title: string;
  author: {
    id: number;
    name: string;
  };
}

// DTO for author with their books
interface AuthorWithBooksDto {
  id: number;
  name: string;
  books: {
    id: number;
    title: string;
  }[];
}
```

### Avoid Circular Nesting

```typescript
// ❌ BAD: Circular reference in DTO
interface BadDto {
  author: {
    id: number;
    books: {
      id: number;
      author: {        // ← Circular! Infinite nesting
        id: number;
        books: { }
      }
    }
  }
}

// ✅ Good: Resolved at one level
interface GoodDto {
  id: number;
  title: string;
  author: {
    id: number;
    name: string;
    // ← Don't include books here
  }
}
```

## Composite Keys

Sometimes an entity uses multiple foreign keys:

```typescript
// Domain entity with composite key
interface OrderItem {
  orderId: number;     // Part of key
  productId: number;   // Part of key
  quantity: number;
  price: number;
}
```

### In the Database

```sql
CREATE TABLE order_items (
  orderId INTEGER REFERENCES orders(id),
  productId INTEGER REFERENCES products(id),
  quantity INTEGER,
  price DECIMAL(10, 2),
  PRIMARY KEY (orderId, productId)
);
```

## Self-References

An entity can reference itself (like comments on comments).

```typescript
// Domain entity
interface Comment {
  id: number;
  text: string;
  parentCommentId?: number;  // ← Can reference another comment
  authorId: number;
}
```

### In the Database

```sql
CREATE TABLE comments (
  id INTEGER PRIMARY KEY,
  text TEXT,
  parentCommentId INTEGER REFERENCES comments(id),
  authorId INTEGER REFERENCES users(id)
);
```

### Querying Hierarchies

```typescript
class CommentService {
  async getCommentThread(commentId: number): Promise<Comment[]> {
    // Get all replies to this comment
    return this.commentRepository.findByParentId(commentId);
  }

  async getCommentPath(commentId: number): Promise<Comment[]> {
    // Get all parent comments (walk the tree upward)
    const result = [];
    let currentId: number | undefined = commentId;

    while (currentId) {
      const comment = await this.commentRepository.getById(currentId);
      result.unshift(comment);
      currentId = comment.parentCommentId;
    }

    return result;
  }
}
```

## Soft Deletes and References

When using soft deletes, consider reference behavior:

```typescript
interface Comment {
  id: number;
  text: string;
  authorId: number;
  deletedAt?: Date;    // Soft delete
}

// When querying, exclude deleted
async function getCommentsByPost(postId: number) {
  return comments.filter(c => !c.deletedAt);
}
```

## Foreign Key Constraints

Database foreign keys ensure referential integrity:

```sql
-- Strict: Prevent orphaned records
CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  customerId INTEGER REFERENCES customers(id) ON DELETE RESTRICT
);

-- Cascade: Delete related records
CREATE TABLE order_items (
  id INTEGER PRIMARY KEY,
  orderId INTEGER REFERENCES orders(id) ON DELETE CASCADE
);

-- Soft: Allow orphaned records (handle in code)
CREATE TABLE comments (
  id INTEGER PRIMARY KEY,
  authorId INTEGER REFERENCES users(id)  -- No constraint
);
```

---

## Quick Checklist

When designing relationships:

- [ ] Uses IDs for references, not nested objects?
- [ ] Foreign keys defined in database?
- [ ] Circular references avoided in domain entities?
- [ ] Can nest only in DTOs, not entities?
- [ ] Delete behavior considered (cascade, restrict)?
- [ ] Query methods explicit about loading related data?

---

## Related

- [Flat Entities](flat-entities.md)
- [DTO vs Domain Model](dto-vs-domain.md)
- [Denormalization](denormalization.md)
- [Examples](examples.md)
