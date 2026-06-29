# Naming Classes & Interfaces

Use **PascalCase** for classes and interfaces. Names should be specific and describe the concept they represent, not implementation details.

## Basic Naming

### Use Specific, Concrete Names

Each class or interface represents a specific concept. Avoid generic or vague names.

```typescript
// BAD — Generic, unclear
class Data { }
class Model { }
class Resource { }
class Info { }

// GOOD — Specific, meaningful
class User { }
class Product { }
class Order { }
class Comment { }
```

## Don't Duplicate Entity Name in Properties

Don't include the entity name as a prefix on properties — it's redundant:

```typescript
// BAD — Properties repeat the class name
class User {
  userId: number;
  userName: string;
  userEmail: string;
}

// GOOD — Clean, no repetition
class User {
  id: number;
  name: string;
  email: string;
}
```

This same principle applies to DTOs and API responses:

```typescript
// BAD — Field names repeat entity name
{
  "userName": "Bob",
  "userEmail": "bob@example.com"
}

// GOOD — Clean field names
{
  "name": "Bob",
  "email": "bob@example.com"
}
```

## Multiple Entities in Response

When returning multiple entities, use clear wrappers:

```typescript
// BAD — Unclear structure
{
  "name": "Bob",
  "email": "bob@example.com",
  "city": "New York",
  "zipCode": "10001"
}

// GOOD — Clear entity boundaries
{
  "user": {
    "name": "Bob",
    "email": "bob@example.com"
  },
  "address": {
    "city": "New York",
    "zipCode": "10001"
  }
}
```

## Service, Controller, Repository Suffixes

For classes with specific roles, use meaningful suffixes. Prefer full names for clarity:

```typescript
// Preferred — Full names
class UserService { }
class UserController { }
class UserRepository { }
class ProductService { }

// Abbreviated (if your team agrees)
class UserSvc { }
class UserCtrl { }
class UserRepo { }
```

Choose one pattern and apply it consistently across the project.

## DTO Naming

Use `Dto` or more specific suffixes for data transfer objects:

```typescript
// By purpose
class UserCreateDto { }
class UserUpdateDto { }
class UserResponseDto { }
class UserListDto { }

// By suffixes
interface CreateUserDto { }
interface UpdateUserDto { }
interface UserResponse { }
interface UserListResponse { }
```

Pick a convention and stick with it.

## Type/Interface Suffixes

You can use suffixes to clarify intent, but keep them consistent:

```typescript
// No suffix (common in modern TypeScript)
interface User { }
interface Filter { }
interface Settings { }

// With I prefix (older convention, less common now)
interface IUser { }
interface IFilter { }

// With suffix
interface UserInterface { }
interface FilterSpec { }
```

## Enum Naming

Use **PascalCase** for enums, **UPPER_CASE** for enum values:

```typescript
enum UserRole {
  ADMIN = 'ADMIN',
  MODERATOR = 'MODERATOR',
  USER = 'USER'
}

enum OrderStatus {
  PENDING = 'PENDING',
  PROCESSING = 'PROCESSING',
  SHIPPED = 'SHIPPED',
  DELIVERED = 'DELIVERED'
}
```

## Examples by Category

### Core Entities

```typescript
class User { id: number; name: string; email: string; }
class Product { id: number; name: string; price: number; }
class Order { id: number; userId: number; total: number; }
```

### Services

```typescript
class UserService { create(); find(); get(); update(); delete(); }
class ProductService { find(); get(); create(); }
class OrderService { create(); find(); get(); }
```

### Repositories

```typescript
class UserRepository { save(); findById(); findAll(); delete(); }
class ProductRepository { save(); findById(); findAll(); }
```

### DTOs

```typescript
class CreateUserDto { email: string; name: string; }
class UpdateUserDto { name?: string; email?: string; }
class UserResponseDto { id: number; name: string; email: string; }
```

### Controllers

```typescript
class UserController { create(); find(); get(); update(); delete(); }
class ProductController { find(); get(); create(); }
```

---

## Related

- [Overview](overview.md)
- [Functions & Methods](functions-methods.md)
- [Files & Folders](files-folders.md)
- [File Type Suffixes](file-type-suffixes.md)
- [Anti-Patterns](anti-patterns.md)
