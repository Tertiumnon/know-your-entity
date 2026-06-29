# REST API Design

Principles and patterns for designing RESTful APIs.

## Resource-Oriented Design

REST APIs are organized around **resources**, not actions. Think in terms of nouns (what you're working with) rather than verbs (what you're doing).

### Typical CRUD Endpoints

For each resource, follow this pattern:

```
GET    /users           — Get all users
GET    /users/{id}      — Get one user by ID
POST   /users           — Create a new user
PUT    /users/{id}      — Replace a user entirely
PATCH  /users/{id}      — Partially update a user
DELETE /users/{id}      — Delete a user
```

### Concrete Examples

**Users**
```
GET    /users                 # List all users
POST   /users                 # Create a new user
GET    /users/{userId}        # Get specific user
PATCH  /users/{userId}        # Update specific user
DELETE /users/{userId}        # Delete specific user
```

**Products**
```
GET    /products              # List all products
POST   /products              # Create a new product
GET    /products/{productId}  # Get specific product
PATCH  /products/{productId}  # Update product
DELETE /products/{productId}  # Delete product
```

**Orders**
```
GET    /orders                # List all orders
POST   /orders                # Create a new order
GET    /orders/{orderId}      # Get specific order
PATCH  /orders/{orderId}      # Update order (e.g., status)
DELETE /orders/{orderId}      # Cancel order
```

## HTTP Methods and Semantics

| Method | Purpose | Body | Idempotent |
|--------|---------|------|-----------|
| **GET** | Retrieve data | No | Yes |
| **POST** | Create new resource | Yes | No |
| **PUT** | Replace entire resource | Yes | Yes |
| **PATCH** | Partial update | Yes | Maybe |
| **DELETE** | Remove resource | No | Yes |

### GET — Safe and Idempotent

```
GET /users/42

Response:
200 OK
{
  "id": 42,
  "name": "Alice",
  "email": "alice@example.com"
}
```

**Safe**: Does not modify server state
**Idempotent**: Multiple identical requests produce the same result

### POST — Create New Resource

```
POST /users
{
  "name": "Bob",
  "email": "bob@example.com"
}

Response:
201 Created
{
  "id": 43,
  "name": "Bob",
  "email": "bob@example.com"
}
```

**Not idempotent**: Each POST creates a new resource

### PUT — Replace Entire Resource

```
PUT /users/42
{
  "name": "Alice Smith",
  "email": "alice.smith@example.com"
}

Response:
200 OK
{
  "id": 42,
  "name": "Alice Smith",
  "email": "alice.smith@example.com"
}
```

**Idempotent**: Multiple identical PUTs produce the same result

### PATCH — Partial Update

```
PATCH /users/42
{
  "name": "Alice Johnson"
}

Response:
200 OK
{
  "id": 42,
  "name": "Alice Johnson",
  "email": "alice@example.com"
}
```

**Maybe idempotent**: Depends on how it's implemented

### DELETE — Remove Resource

```
DELETE /users/42

Response:
204 No Content
```

**Idempotent**: Multiple DELETEs of the same resource produce the same result

## Consistency: Singular vs Plural

Choose one convention for resource names and stick with it:

### Option 1: Plural Resources (Recommended)

```
GET    /products
POST   /products
GET    /products/{productId}
PATCH  /products/{productId}
DELETE /products/{productId}
```

**Advantages**: Treats collections and items uniformly

### Option 2: Singular Resources

```
GET    /product
POST   /product
GET    /product/{productId}
PATCH  /product/{productId}
DELETE /product/{productId}
```

**Less common**, but valid if consistently applied

**Choose one and apply everywhere** — Don't mix `/users` and `/product`.

## Relationships Between Resources

### Nested Routes

For related resources, use nesting to show hierarchy:

```
GET    /users/{userId}/orders               # User's orders
POST   /users/{userId}/orders               # Create order for user
GET    /users/{userId}/orders/{orderId}    # Specific order by user
DELETE /users/{userId}/orders/{orderId}    # Delete order
```

### Flat Routes (Alternative)

Use filtering instead of nesting:

```
GET    /orders?userId=42                    # User's orders
POST   /orders
GET    /orders/123                          # Specific order
DELETE /orders/123
```

Both approaches work; choose based on your API's depth.

## Query Parameters

Use query parameters for filtering, pagination, and sorting:

```
GET /users?search=alice&role=admin&limit=10&offset=0
GET /products?category=electronics&sort=price&order=desc
GET /orders?status=delivered&from=2025-01-01&to=2025-12-31
```

### Common Patterns

**Pagination**
```
GET /users?limit=20&offset=40    # Items 41-60
GET /users?page=2&pageSize=20    # Alternative pagination
```

**Filtering**
```
GET /products?category=electronics&inStock=true
GET /orders?status=pending&userId=42
```

**Sorting**
```
GET /products?sort=price&order=asc
GET /users?sort=name&order=desc
```

**Search**
```
GET /users?search=alice
GET /products?q=laptop
```

## HTTP Status Codes

Use meaningful status codes to communicate results:

| Code | Meaning | Example |
|------|---------|---------|
| **200** | OK | Successful GET, PUT, PATCH |
| **201** | Created | Successful POST creating resource |
| **204** | No Content | Successful DELETE |
| **400** | Bad Request | Invalid input data |
| **401** | Unauthorized | Missing/invalid authentication |
| **403** | Forbidden | Authenticated but not allowed |
| **404** | Not Found | Resource doesn't exist |
| **409** | Conflict | Resource conflict (e.g., duplicate) |
| **500** | Server Error | Unexpected server error |

### Examples

```
POST /users with invalid email
→ 400 Bad Request
{
  "error": "Invalid email format"
}

POST /users with duplicate email
→ 409 Conflict
{
  "error": "Email already exists"
}

GET /users/999 (doesn't exist)
→ 404 Not Found
{
  "error": "User not found"
}

DELETE /orders/123 (user not authorized)
→ 403 Forbidden
{
  "error": "You don't have permission to delete this order"
}
```

## Error Responses

Standardize error response format:

```typescript
interface ErrorResponse {
  error: string;           // Error message
  code?: string;           // Error code (e.g., "INVALID_EMAIL")
  details?: Record<string, string>;  // Field-specific errors
}
```

### Examples

```json
{
  "error": "Validation failed",
  "code": "VALIDATION_ERROR",
  "details": {
    "email": "Invalid email format",
    "password": "Must be at least 8 characters"
  }
}
```

## Request & Response Structure

### Request Body (DTO)

Keep request bodies focused — only include fields that can be set:

```typescript
// POST /users
{
  "name": "Bob",
  "email": "bob@example.com",
  "password": "securePassword123"
}

// PATCH /users/42
{
  "name": "Bob Johnson"
  // Only send fields you're updating
}
```

### Response Body (DTO)

Return only safe, relevant data:

```typescript
// POST /users → 201 Created
{
  "id": 43,
  "name": "Bob",
  "email": "bob@example.com",
  "createdAt": "2025-01-15T10:30:00Z"
  // Don't return password or internal fields
}
```

## Versioning

If your API will have breaking changes, plan for versioning:

### Option 1: URL Versioning

```
GET /v1/users
GET /v2/users           # Different response format, breaking change
```

### Option 2: Header Versioning

```
GET /users
Header: Accept: application/vnd.api+json;version=2
```

### Option 3: No Versioning (Recommended if possible)

Maintain backward compatibility and extend without breaking changes.

---

## Quick Checklist

When designing a REST API:

- [ ] Resources are nouns, not verbs?
- [ ] Using correct HTTP methods (GET, POST, PATCH, DELETE)?
- [ ] Consistent singular or plural naming?
- [ ] Meaningful HTTP status codes?
- [ ] Request/response DTOs exclude sensitive data?
- [ ] Query parameters for filtering, sorting, pagination?
- [ ] Standard error response format?
- [ ] Clear documentation (OpenAPI/Swagger)?

---

## Documentation & Tools

### OpenAPI / Swagger

Generate API documentation automatically:

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List all users
      responses:
        200:
          description: List of users
    post:
      summary: Create a new user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserDto'
```

- [Swagger / OpenAPI](https://swagger.io/)
- Tools: Swagger UI, Redoc, OpenAPI Generator

### Testing APIs

Use tools to test your endpoints:

- **Postman** — Interactive testing
- **curl** — Command-line testing
- **Jest/Supertest** — Automated testing
- **Thunder Client** — VSCode extension

---

## Related

- [Request & Response Patterns](request-response.md)
- [Error Handling](error-handling.md)
- [Service Layer](service-layer.md)
