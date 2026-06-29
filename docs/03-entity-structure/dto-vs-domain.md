# DTO vs Domain Model

Understanding the difference between domain entities and data transfer objects is crucial for clean architecture.

## Quick Definition

| Aspect | Domain Entity | DTO |
|--------|---------------|-----|
| **Purpose** | Business logic, data source | Data transfer over APIs |
| **Structure** | Flat, simple | Can be nested, optimized for clients |
| **Nesting** | ❌ No nested objects | ✅ 1-2 levels acceptable |
| **Fields** | Only what's essential | Only what client needs |
| **Mutation** | Can have methods | Usually data-only |
| **Location** | Database, business logic | API responses, requests |

## Domain Entity

A domain entity is the **core business model**. It represents a real concept in your system.

### Characteristics

- **Flat structure** — Only primitives and IDs
- **Single source of truth** — One entity per concept
- **Business rules** — May contain validation, constraints
- **Database-backed** — Typically maps to a table
- **Focused** — Only essential fields

### Example

```typescript
interface User {
  id: number;
  email: string;
  name: string;
  passwordHash: string;
  createdAt: Date;
}
```

**This is the real User** — it has everything needed internally, including sensitive fields like `passwordHash`.

## DTO (Data Transfer Object)

A DTO is **what you show to clients**. It's optimized for external consumption.

### Characteristics

- **Tailored** — Only fields the client needs
- **Safe** — Excludes sensitive data
- **Structured** — Can be nested for convenience
- **Transport-focused** — Used in API requests/responses
- **Multiple versions** — Different DTOs for different use cases

### Example

```typescript
interface UserResponseDto {
  id: number;
  email: string;
  name: string;
  createdAt: string;  // ← Different format for API
  // ← No passwordHash!
}

interface UserCreateDto {
  email: string;
  name: string;
  password: string;   // ← Clear intent, not hashed
}

interface UserUpdateDto {
  email?: string;
  name?: string;
}
```

## Key Differences

### Sensitive Data

**Domain Entity** includes everything:

```typescript
// Internal User entity — has all data
class User {
  id: number;
  email: string;
  passwordHash: string;
  twoFactorSecret?: string;
  subscriptionStatus: 'active' | 'canceled';
  billingEmail: string;
  // ... other sensitive fields
}
```

**DTO** excludes sensitive fields:

```typescript
// DTO sent to client
interface UserResponseDto {
  id: number;
  email: string;
  name: string;
  createdAt: string;
  // ← No sensitive data exposed
}
```

### Nesting

**Domain Entity** — Flat:

```typescript
// Domain: User references Address by ID
class User {
  id: number;
  name: string;
  addressId: number;  // ← Just an ID
}
```

**DTO** — Can nest for convenience:

```typescript
// DTO: Include full address details for client
interface UserResponseDto {
  id: number;
  name: string;
  address: {          // ← Nested for convenience
    city: string;
    zipCode: string;
    country: string;
  };
}
```

### Methods

**Domain Entity** — Can have behavior:

```typescript
class User {
  id: number;
  email: string;

  isEmailValid(): boolean {
    return isValidEmail(this.email);
  }

  changePassword(oldPassword: string, newPassword: string): void {
    // Validation and update logic
  }
}
```

**DTO** — Usually just data:

```typescript
interface UserResponseDto {
  id: number;
  email: string;
  name: string;
}
// No methods, just data
```

## Real World Example: User Registration

### 1. Client Sends DTO

```typescript
// Client sends this
interface UserCreateDto {
  email: string;
  name: string;
  password: string;
}
```

### 2. Backend Creates Domain Entity

```typescript
// Backend internally creates this
class User {
  id: number;                                    // ← Generated
  email: string;                                 // ← From DTO
  name: string;                                  // ← From DTO
  passwordHash: string;                          // ← Hashed (not from DTO!)
  emailVerified: boolean = false;                // ← Default
  subscriptionStatus: 'trial' = 'trial';        // ← Default
  createdAt: Date = new Date();                 // ← Server time
}
```

### 3. Backend Returns DTO

```typescript
// Backend returns this to client
interface UserResponseDto {
  id: number;
  email: string;
  name: string;
  createdAt: string;
}

// ← No passwords, no verification status, no subscription details
```

## Multiple DTOs for Different Contexts

You often need different DTOs for different scenarios:

```typescript
// For registration
interface UserCreateDto {
  email: string;
  name: string;
  password: string;
}

// For profile update
interface UserUpdateDto {
  name?: string;
  email?: string;
  // Note: password change is separate
}

// For API response
interface UserResponseDto {
  id: number;
  email: string;
  name: string;
  createdAt: string;
}

// For admin view
interface UserAdminDto {
  id: number;
  email: string;
  name: string;
  createdAt: string;
  subscriptionStatus: string;
  emailVerified: boolean;
  lastLogin: string;
}

// For user list
interface UserListItemDto {
  id: number;
  name: string;
  email: string;
}

// For detailed profile
interface UserProfileDto {
  id: number;
  email: string;
  name: string;
  bio?: string;
  avatar?: string;
  createdAt: string;
  addresses: {
    id: number;
    city: string;
  }[];
}
```

## Mapping Between Entity and DTO

Your service layer should handle the mapping:

```typescript
class UserService {
  async createUser(dto: UserCreateDto): Promise<UserResponseDto> {
    // 1. Validate DTO
    validate(dto);

    // 2. Create domain entity
    const user = new User();
    user.email = dto.email;
    user.name = dto.name;
    user.passwordHash = await hashPassword(dto.password);
    user.createdAt = new Date();

    // 3. Persist
    const savedUser = await this.userRepository.save(user);

    // 4. Return DTO (not the entity!)
    return this.toResponseDto(savedUser);
  }

  private toResponseDto(user: User): UserResponseDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt.toISOString(),
    };
  }
}
```

## When to Nest in DTOs

1-2 levels of nesting in DTOs is acceptable when:

```typescript
// ✅ Good: 1 level of nesting
interface PostResponseDto {
  id: number;
  title: string;
  author: {           // ← 1 level
    id: number;
    name: string;
  };
}

// ✅ Good: 2 levels of nesting
interface OrderDto {
  id: number;
  items: [
    {
      id: number;
      product: {      // ← 2 levels
        id: number;
        name: string;
      };
    }
  ];
}

// ❌ Bad: 3+ levels
interface DeepDto {
  post: {
    author: {
      company: {      // ← Too deep
        employees: {
          // ...
        }
      }
    }
  };
}
```

---

## Quick Checklist

When designing entities and DTOs:

**Domain Entity**:
- [ ] All fields are primitives or IDs?
- [ ] No sensitive data exposure needed?
- [ ] Flat structure?
- [ ] Single source of truth?

**DTO**:
- [ ] Only includes fields client needs?
- [ ] Sensitive data removed?
- [ ] Maximum 2 levels of nesting?
- [ ] Optimized for transport?
- [ ] Clear naming (ResponseDto, CreateDto, etc.)?

---

## Related

- [Flat Entities](flat-entities.md)
- [Relationships & References](relationships.md)
- [Examples](examples.md)
