# What is an Entity?

## Definition

An **entity** is a base unit that exists as a model—a data structure with properties that characterize it.

In programming, this means:

- **Entity** = Object/Structure with properties
- **Entity** = A domain model in your business logic
- **Entity** = Usually corresponds to a database table
- **Entity** = The "nouns" in your application (User, Product, Order, etc.)

Entities are the foundation of entity-driven development. They define what data you have and how it's organized.

---

## Examples

### User Entity

The most common example:

```typescript
interface User {
  id: number;
  email: string;
  name: string;
  createdAt: Date;
}
```

Properties: `id`, `email`, `name`, `createdAt`
Responsibility: Represents a person registered in the system

### Product Entity

For e-commerce:

```typescript
interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  categoryId: number;        // Reference to another entity
  inStock: boolean;
  createdAt: Date;
}
```

Notice: `categoryId` is an ID (reference), not a full Category object

### Order Entity

Complex entity with relationships:

```typescript
interface Order {
  id: number;
  userId: number;            // References User
  totalPrice: number;
  status: OrderStatus;       // Enum: 'pending' | 'processing' | 'shipped' | 'delivered'
  itemCount: number;
  createdAt: Date;
  updatedAt: Date;
}
```

---

## Why Entities Matter

Understanding entities gives you understanding of:

### 1. Processes

How does data flow through your system?

```
User (entity) → signs up → Service creates user → User stored in database
Product (entity) → added to cart → Service updates cart → Order (entity) created
```

### 2. Code Organization

Where does logic belong?

```typescript
// USER SERVICE - handles user operations
class UserService {
  async create(data): Promise<User> { }
  async find(filters): Promise<User[]> { }
  async update(id, data): Promise<User> { }
}

// PRODUCT SERVICE - handles product operations
class ProductService {
  async find(filters): Promise<Product[]> { }
  async get(id): Promise<Product> { }
}

// ORDER SERVICE - handles order operations
class OrderService {
  async create(userId, items): Promise<Order> { }
}
```

Each entity has its own service with clear responsibilities.

### 3. Naming

How to name variables, functions, and files?

```typescript
// Variables reflect entities
const user = await userService.get(userId);
const products = await productService.find();
const orders = await orderService.find({ userId });

// Functions use clear verbs
userService.create()      // create a user
userService.find()        // find users
userService.get(id)       // get specific user

// Files match entities
user.service.ts           // UserService
product.interface.ts      // Product interface
order.dto.ts              // Order DTOs
```

### 4. Architecture

How to structure folders and modules?

```
src/
  entities/
    user/           ← User entity files
      user.interface.ts
      user.service.ts
    product/        ← Product entity files
      product.interface.ts
      product.service.ts
    order/          ← Order entity files
      order.interface.ts
      order.service.ts
  components/       ← UI components use entities
    user-card/
    product-list/
  pages/            ← Pages combine components
    products/
    checkout/
```

---

## Entity vs Other Concepts

Different software concepts and how they relate to entities:

| Concept | Purpose | Example | Relationship |
|---------|---------|---------|---------------|
| **Entity** | Core data model | User, Product, Order | Base definition |
| **DTO** | Data transfer object | UserCreateDto, ProductResponseDto | Transfers entity data |
| **Service** | Business logic | UserService, ProductService | Operates on entities |
| **Repository** | Data access | UserRepository | Persists entities |
| **Component** | UI rendering | UserCard, ProductList | Displays entities |
| **Controller** | HTTP routing | UserController | Accepts/returns entities |

### Entity vs DTO

**Entity** - The real thing:
```typescript
interface User {
  id: number;
  email: string;
  name: string;
  password: string;           // Sensitive!
  createdAt: Date;
}
```

**DTO** - What you show to clients:
```typescript
interface UserResponseDto {
  id: number;
  email: string;
  name: string;
  createdAt: string;          // No password!
}
```

Same entity, different views depending on the context.

### Entity vs Database Table

They usually correspond:

```
Database Table: users
↓
Entity: User
↓
Service: UserService
↓
Components that use it: UserCard, UserList, UserProfile
```

But not always 1:1. Sometimes:
- Multiple entities share one table (inheritance)
- One entity spans multiple tables (denormalization)
- Views or projections create temporary entities

---

## The Entity-Driven Mindset

When designing any feature, start by asking:

### 1. What are my entities?

List the nouns in your requirements:

> "Users can create accounts, browse products, and place orders"

Entities: **User**, **Product**, **Order**

### 2. What properties do they have?

```typescript
// User
{ id, email, name, createdAt }

// Product
{ id, name, price, categoryId, inStock }

// Order
{ id, userId, totalPrice, status, createdAt }
```

### 3. What relationships exist?

```
User —has-many→ Orders
Product —has-one→ Category
Order —has-many→ OrderItems
```

Expressed as foreign keys:
```typescript
Order: { userId, productId, ... }
OrderItem: { orderId, productId, ... }
```

### 4. What operations are needed?

```
User:
  - create(email, name, password)
  - find(filters)
  - get(id)
  - update(id, data)
  - delete(id)

Product:
  - find(categoryId)
  - get(id)
  - create(data)
  - update(id, data)

Order:
  - create(userId, items)
  - find(userId)
  - get(id)
  - update(id, status)
```

This naturally leads to services with clear responsibilities.

---

## Common Mistakes

### Mistake 1: Too Many Properties

❌ **Bad**: Entity with everything

```typescript
interface User {
  id: number;
  email: string;
  name: string;
  phone: string;
  address: string;
  city: string;
  zipCode: string;
  country: string;
  birthDate: Date;
  avatar: string;
  bio: string;
  // ... 20 more properties
}
```

✅ **Better**: Separate entities for related concepts

```typescript
interface User {
  id: number;
  email: string;
  name: string;
}

interface UserProfile {
  userId: number;
  phone: string;
  birthDate: Date;
  avatar: string;
  bio: string;
}

interface UserAddress {
  userId: number;
  city: string;
  zipCode: string;
  country: string;
}
```

### Mistake 2: Nesting Other Entities

❌ **Bad**: Domain model with nested objects

```typescript
interface User {
  id: number;
  name: string;
  address: {
    city: string;
    zipCode: string;
  };
  company: {
    name: string;
    industry: string;
  };
}
```

✅ **Good**: Flat domain model with references

```typescript
interface User {
  id: number;
  name: string;
  addressId: number;      // Reference only
  companyId: number;      // Reference only
}
```

(Nesting is OK only in DTOs returned from APIs)

### Mistake 3: Unclear Entity Names

❌ **Bad**: Generic names

```typescript
class Data { }
class Model { }
class Resource { }
class Info { }
```

✅ **Good**: Specific entity names

```typescript
class User { }
class Product { }
class Order { }
class Comment { }
```

---

## Next Steps

Now that you understand what entities are:

1. **Naming entities correctly**: Read [Naming Conventions](../02-naming-conventions/overview.md)
2. **Designing entities well**: Read [Flat Entities](../03-entity-structure/flat-entities.md)
3. **Organizing code by entity**: Read [Architecture](../06-architecture/folder-structure.md)
4. **Building services for entities**: Read [Service Layer](../04-backend-patterns/service-layer.md)
5. **See working examples**: [entity-driven-architecture](https://github.com/Tertium/entity-driven-architecture)

---

## Quick Checklist

When designing an entity, verify:

- [ ] Entity name is specific (User, not Data)
- [ ] Properties are primitive or IDs (no nested objects)
- [ ] Related data is separate entity (not embedded)
- [ ] Each property is necessary (avoid bloat)
- [ ] Entity maps to a real domain concept
- [ ] Service exists for entity operations (find, get, create, update, delete)

---

## Related

- [Philosophy: Entity Thinking](philosophy.md)
- [Naming: Classes and Interfaces](../02-naming-conventions/classes-interfaces.md)
- [Structure: Flat Entities](../03-entity-structure/flat-entities.md)
- [Architecture: core/entities folder](../06-architecture/folder-structure.md)
