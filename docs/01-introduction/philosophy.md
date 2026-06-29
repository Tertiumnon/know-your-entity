# Philosophy: Entity Thinking

Why entity-driven development matters beyond just code structure.

---

## What is Entity Thinking?

**Entity thinking** is a mental model for understanding systems—both in software and in real life.

Rather than thinking about:
- "How do I implement this feature?"
- "What functions do I need?"
- "What technologies should I use?"

Entity thinking asks:
- "What are the core things (entities) in this system?"
- "What properties define them?"
- "What relationships exist between them?"
- "What can you do with them?"

This leads to clearer design, better code, and easier maintenance.

---

## Real-World Example: A Coffee Shop

### Without Entity Thinking

```
Feature request: "Customers should be able to order coffee"

Implementation: Build a form, process input, store somewhere, send emails...
```

Problems:
- Unclear what data is important
- Easy to add too much or too little information
- Hard to extend later (what about payment? ratings? rewards?)
- No clear responsibility boundaries

### With Entity Thinking

```
What are the core things?
  - Customer (entity)
  - Coffee (entity)
  - Order (entity)
  - Payment (entity)

Customer: { id, name, email, phone }
Coffee: { id, name, price, size }
Order: { id, customerId, coffeeId, quantity, status, createdAt }
Payment: { id, orderId, amount, method, status }

What can we do?
  - Customer: create, update, delete
  - Coffee: list, get details
  - Order: create, view, cancel, update status
  - Payment: process, refund, check status
```

Benefits:
- Clear what data matters
- Easy to add features (loyalty program → add field to Customer)
- Clear who's responsible for what (CustomerService, OrderService, etc.)
- Easy to test each part independently

---

## In Programming: Order Management System

### Before Entity Thinking (Procedural)

```javascript
// How do I implement "place an order"?
function placeOrder(customerId, items, shippingAddress, paymentMethod) {
  // Validate everything
  if (!customerId) throw error;
  if (!items || items.length === 0) throw error;
  // ... 50 more validations ...

  // Create order record
  db.insert('orders', { customerId, items, shippingAddress, paymentMethod, status: 'pending' });

  // Process payment
  let cardToken = processCard(paymentMethod.card);
  // ... payment logic ...

  // Send confirmation email
  email.send(customer.email, 'Order Confirmation', ...);

  // Update inventory
  items.forEach(item => {
    db.update('products', { id: item.productId }, { stock: stock - item.quantity });
  });

  // ... and 100 more lines of mixed logic ...
}
```

Problems:
- One giant function does everything
- Hard to test individual pieces
- Mixing concerns (validation, business logic, communication, persistence)
- Where do we add new features?

### After Entity Thinking (Structured)

```typescript
// Define entities
interface Customer {
  id: number;
  name: string;
  email: string;
}

interface Product {
  id: number;
  name: string;
  price: number;
  stock: number;
}

interface Order {
  id: number;
  customerId: number;
  status: 'pending' | 'confirmed' | 'shipped' | 'delivered';
  totalPrice: number;
  createdAt: Date;
}

interface OrderItem {
  orderId: number;
  productId: number;
  quantity: number;
  price: number;
}

// Create services for each entity
class CustomerService {
  get(id: number): Customer { }
  create(data): Customer { }
}

class ProductService {
  get(id: number): Product { }
  decrementStock(id: number, quantity: number): Product { }
}

class OrderService {
  create(customerId: number, items: OrderItem[]): Order { }
  updateStatus(id: number, status: string): Order { }
}

class PaymentService {
  process(orderId: number, amount: number, method: string): Payment { }
}

class NotificationService {
  sendOrderConfirmation(order: Order, customer: Customer): void { }
}

// Now "place an order" is simple
async function placeOrder(customerId, items, paymentMethod) {
  // Verify customer exists
  const customer = await customerService.get(customerId);

  // Create order
  const order = await orderService.create(customerId, items);

  // Process payment
  await paymentService.process(order.id, order.totalPrice, paymentMethod);

  // Update inventory
  for (const item of items) {
    await productService.decrementStock(item.productId, item.quantity);
  }

  // Notify customer
  await notificationService.sendOrderConfirmation(order, customer);

  return order;
}
```

Benefits:
- Each service has one job
- Easy to test `OrderService` without touching `PaymentService`
- New feature? Add to the appropriate service
- Clear data flow and responsibilities

---

## Key Benefits of Entity Thinking

### 1. Clarity

Everyone understands what you're talking about:
- "Update the User entity" vs. "Fix the user thing"
- Clear boundaries: what's part of Order, what's part of Payment?

### 2. Maintainability

When requirements change, you know exactly where to look:

```
New requirement: "Users should have addresses"
→ Add to User entity
→ Update UserService
→ Add migration
→ Done
```

Without entity thinking: "Where do I put address logic?" (everywhere?)

### 3. Testability

Test each entity independently:

```typescript
// Test User service
test('UserService.create creates user with email');
test('UserService.update updates only changed fields');

// Test Order service
test('OrderService.create creates order for valid customer');
test('OrderService.create throws for invalid customer');

// No dependencies on Payment, Notification, etc.
```

### 4. Scalability

As your system grows, entity structure doesn't break:

```
Small system:
  - 5 entities
  - 1 database
  - 1 service per entity

Large system:
  - 50 entities
  - 10 microservices
  - Each entity still has clear responsibility
  - Easy to split into services
```

### 5. Communication

Entity-based thinking matches how humans think about domains:

Accountant: "A customer has multiple orders and addresses"
Developer: "Customer entity has customerOrders and customerAddresses tables"
Manager: "Orders have a status: pending, shipped, delivered"
Developer: `interface Order { status: OrderStatus }`

Same mental model, different languages.

---

## Entity Thinking in Different Domains

### E-Commerce

**Entities**: Customer, Product, Category, Cart, Order, Payment, Review, Shipment

**Relationships**: Customer has many Orders. Order has many OrderItems. Product belongs to Category.

**Operations**: Customer.browse(), Product.get(), Cart.add(), Order.create(), Payment.process()

### Social Network

**Entities**: User, Post, Comment, Like, Follow, Message, Notification

**Relationships**: User has many Posts. Post has many Comments. User has many Followers.

**Operations**: Post.create(), Comment.add(), Like.toggle(), Follow.add()

### Task Management

**Entities**: Workspace, User, Project, Task, Label, Assignment, Activity

**Relationships**: Workspace has many Projects. Project has many Tasks. Task has many Labels.

**Operations**: Task.create(), Task.assign(), Task.complete(), Label.add()

---

## Common Pitfalls

### Pitfall 1: Too Many Entities

❌ Creating an entity for every little thing

```typescript
interface User { ... }
interface UserName { ... }
interface UserEmail { ... }
interface UserAddress { ... }
interface UserPreferences { ... }
```

✅ One User entity with all relevant properties

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  addressId: number;    // Reference if complex
  preferences: UserPreferences;
}
```

### Pitfall 2: Entities Without Clear Purpose

❌ Vague entities

```typescript
interface Data { ... }
interface Info { ... }
interface Thing { ... }
```

✅ Specific entities

```typescript
interface Product { ... }
interface Order { ... }
interface Review { ... }
```

### Pitfall 3: Mixing Entity Concerns

❌ One entity doing everything

```typescript
interface User {
  // User properties
  id: number;
  name: string;

  // But also...
  orders: Order[];           // Should UserService handle this?
  preferences: {
    theme: string;
    language: string;
    notifications: boolean;
  };
  activityLog: Activity[];   // Should UserService handle this?
}
```

✅ Separate entities with references

```typescript
interface User {
  id: number;
  name: string;
}

interface UserPreferences {
  userId: number;
  theme: string;
  language: string;
  notifications: boolean;
}

interface UserActivity {
  userId: number;
  action: string;
  timestamp: Date;
}
```

---

## How to Start Thinking in Entities

### Step 1: List the Nouns

Read your requirements and identify nouns (things):

> "Users create accounts, browse products, add them to a cart, and place orders. Admins manage product categories and prices."

Nouns: **User**, **Account**, **Product**, **Cart**, **Order**, **Admin**, **Category**, **Price**

### Step 2: Identify Core Entities

Not all nouns need to be entities. Some are:
- Attributes (Account → part of User)
- Relationships (Cart → temporary, not persistent)
- Enums (Price → just a number, Category → reference in Product)

Core entities: **User**, **Product**, **Category**, **Order**, **OrderItem**

### Step 3: Define Properties

For each entity, what describes it?

```typescript
User: { id, name, email, createdAt }
Product: { id, name, description, price, categoryId, inStock }
Category: { id, name, description }
Order: { id, userId, totalPrice, status, createdAt }
OrderItem: { orderId, productId, quantity, price }
```

### Step 4: Identify Relationships

How do entities relate?

```
User —has-many→ Orders (one user, many orders)
Product —has-one→ Category (one product, one category)
Order —has-many→ OrderItems (one order, many items)
OrderItem —references→ Product
```

### Step 5: Design Services

For each entity, what operations are needed?

```typescript
UserService: create, get, find, update, delete
ProductService: get, find, update
CategoryService: get, find, create, update, delete
OrderService: create, get, find, updateStatus
```

---

## Entity Thinking Enables...

### Better Design

> "We need a new feature for returning products"

With entity thinking:
```
Does this affect Order entity? ✓ (add returnStatus)
Does this affect Product entity? ✗
Does this affect User entity? ✗
Add new Return entity? ✓ (track what was returned, why, when)
```

Clear impact analysis.

### Better Testing

```typescript
// Test Order entity independently
describe('OrderService', () => {
  test('create validates required fields');
  test('create calculates total correctly');
  test('updateStatus rejects invalid transitions');
  // No need to test Payment, Notification, etc.
});
```

### Better Communication

Developer to Manager: "The Order entity has 5 properties and 6 operations"
Manager to Developer: "Customers need to track order status"
Both understand immediately.

### Better Code

Services are small and focused:
```typescript
// UserService doesn't know about Orders
// OrderService doesn't know about Payments
// Each service is easy to read, test, and modify
```

---

## The Essence

**Entity-driven development is about understanding your domain first, then structuring code to reflect that understanding.**

Not:
- "Build an API" → "Create endpoints" → "Implement logic"

But:
- "Understand entities" → "Design relationships" → "Implement services"

This naturally produces clean, maintainable, scalable code.

---

## Next Steps

- **Understand entities deeply**: [What is an Entity?](what-is-entity.md)
- **Apply to your code**: [5-Minute Quick Start](quick-start.md)
- **See working examples**: [entity-driven-architecture](https://github.com/Tertium/entity-driven-architecture)
- **Master naming**: [Naming Conventions](../02-naming-conventions/)
