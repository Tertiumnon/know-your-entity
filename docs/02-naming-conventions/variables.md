# Naming Variables

## Rule: Don't Include Type in the Name

Never duplicate the type in a variable name — this adds noise and complicates maintenance. Types belong in annotations or are obvious from context and your IDE.

```typescript
// BAD — Type is in the name
const ageNumber: number = 30;
const userArray: User[] = [];
const userListArray: User[] = [];

// GOOD — Clean names, let types be inferred or annotated
const age = 30;
const users: User[] = [];
```

## Primitives

### Use Full, Meaningful Names

Avoid ambiguous abbreviations (`res`, `tmp`, `data`) — use clear names like `response`, `result`, `users`. Short abbreviations are acceptable in limited scopes (like loop variables), but prefer clarity.

```typescript
// BAD
const a = 30;
const res = await fetch('/users');
const tmp = process.data;

// GOOD
const age = 30;
const response = await fetch('/users');
const userData = process.data;
```

### Example: `result` vs `response`

The difference matters when clarity is important:

```typescript
const getUserNames = async () => {
  const result: string[] = [];           // Final result we're building
  const response = await fetch('/users'); // HTTP response object
  const users = await response.json();    // Parsed users data

  if (users?.data) {
    result.push(...users.data.map(user => user.name));
  }

  return result;
}
```

## Collections

Use **plural names** for collections:

```typescript
// BAD
const user: User[] = [];         // Singular suggests one item

// GOOD
const users: User[] = [];        // Plural clearly shows multiple items
const userEmails: string[] = []; // Array of specific items
```

## Objects and Complex Values

### Describe the Content, Not the Type

```typescript
// BAD — Type is obvious
const userObject = { name: 'Bob', email: 'bob@example.com' };
const userMap = new Map<string, User>();

// GOOD — Name reflects what it contains
const user = { name: 'Bob', email: 'bob@example.com' };
const usersByEmail = new Map<string, User>();
```

### Parameters as Objects

When you have multiple related parameters, create a dedicated type instead of long parameter lists:

```typescript
// BAD
function createOrder(userId, productId, quantity, discount, shippingAddress, billingAddress) { }

// GOOD
interface CreateOrderParams {
  userId: number;
  productId: number;
  quantity: number;
  discount?: number;
  shippingAddress: Address;
  billingAddress: Address;
}

function createOrder(params: CreateOrderParams) { }
```

Use singular form for the type, plural for collections:

```typescript
interface Filter {
  priceFrom?: number;
  priceTo?: number;
}

const filter: Filter = { priceFrom: 0, priceTo: 99 };
const filters: Filter[] = [filter];
```

For named filters with additional metadata:

```typescript
interface NamedFilter {
  name: string;
  valueFrom?: number;
  valueTo?: number;
}

const priceFilter: NamedFilter = {
  name: 'price',
  valueFrom: 0,
  valueTo: 99
};
```

## Boolean Variables

Use affirmative names or verbs that make the value obvious:

```typescript
// GOOD
const isActive = true;
const hasPermission = false;
const canEdit = true;
const shouldUpdate = true;

// AVOID
const inactive = false;  // Double negative is confusing
const notFound = true;   // Hard to read in conditions
```

## Temporary and Loop Variables

Short abbreviations are acceptable in limited scopes:

```typescript
// Acceptable in a short loop
users.map(u => u.name)
[1, 2, 3].forEach(i => total += i)

// Not acceptable in longer functions — be explicit
users.forEach(user => {
  const profile = user.profile;
  const settings = user.settings;
  // ... 20 more lines of logic
  // Better to use full names here
});
```

---

## Related

- [Overview](overview.md)
- [Functions & Methods](functions-methods.md)
- [Classes & Interfaces](classes-interfaces.md)
