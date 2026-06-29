# File Type Suffixes — Be Specific, Not Generic

Using generic suffixes like `.service.ts` creates ambiguity. Use precise names that describe **what** the file does.

## The Problem

### ❌ Bad: Too Generic

```typescript
// user.service.ts — What does this do? Everything?
export class UserService {
  find() { }
  get() { }
  create() { }
  update() { }
  delete() { }
  validateEmail() { }
  validatePassword() { }
  mapToDto() { }
  cache() { }
  // ... 50 more methods
}
```

**Questions it raises:**
- Is this the database layer or business logic?
- Does it validate?
- Does it do mapping?
- Does it handle caching?
- What responsibility does it have?

### ✅ Good: Specific and Clear

```
user.repo.ts          ← Specifically repository (data access)
user.validator.ts     ← Specifically validation
user.mapper.ts        ← Specifically transformation
user.cache.ts         ← Specifically caching
user.api.ts           ← Specifically API/HTTP
user.utils.ts         ← Specifically utilities
```

### Real-World Example: HTTP Layer

**❌ Bad**: Too generic
```
http.service.ts       ← What is this "service"? Everything?
```

**✅ Good**: Specific and clear
```
http.client.ts        ← This is specifically an HTTP client
```

Why? Because:
- `http.client.ts` immediately tells you it makes HTTP requests
- `http.service.ts` could be business logic, validation, caching, or anything
- Specificity = clarity = less cognitive load

## Specific File Type Suffixes

### Data Access Layer

| Suffix | Purpose | Contains | Example |
|--------|---------|----------|---------|
| `.repo.ts` | Repository | Database queries, CRUD | `user.repo.ts` |
| `.dao.ts` | Data Access Object | Low-level DB operations | `user.dao.ts` |
| `.query.ts` | Query builder | Query construction | `user.query.ts` |
| `.store.ts` | State store | In-memory store | `user.store.ts` |
| `.cache.ts` | Cache layer | Caching logic | `user.cache.ts` |

**Examples:**

```typescript
// user.repo.ts — Repository
export class UserRepository {
  async save(user: User): Promise<User> { }
  async findById(id: number): Promise<User | null> { }
  async findAll(): Promise<User[]> { }
  async delete(id: number): Promise<void> { }
}

// user.cache.ts — Cache
export class UserCache {
  get(id: number): User | null { }
  set(id: number, user: User): void { }
  invalidate(id: number): void { }
}

// user.store.ts — State store
export class UserStore {
  currentUser: User | null = null;
  setCurrentUser(user: User): void { }
  clearCurrentUser(): void { }
}
```

### Business Logic Layer

| Suffix | Purpose | Contains | Example |
|--------|---------|----------|---------|
| `.logic.ts` | Business logic | Core business rules | `user.logic.ts` |
| `.manager.ts` | Manager/Orchestrator | Coordinates operations | `user.manager.ts` |
| `.handler.ts` | Event/Command handler | Handles specific action | `user.handler.ts` |
| `.processor.ts` | Data processor | Processes data | `user.processor.ts` |

**Examples:**

```typescript
// user.logic.ts — Business logic
export class UserLogic {
  canDeleteAccount(user: User): boolean { }
  calculateUserScore(user: User): number { }
  isEligibleForPremium(user: User): boolean { }
}

// user.manager.ts — Orchestrator
export class UserManager {
  async registerNewUser(email: string, password: string) {
    const user = await this.userLogic.validateUserData(...);
    const saved = await this.userRepo.save(user);
    await this.userCache.set(saved.id, saved);
    return saved;
  }
}

// user.handler.ts — Event handler
export class UserCreatedHandler {
  async handle(event: UserCreatedEvent): Promise<void> { }
}
```

### Validation Layer

| Suffix | Purpose | Contains | Example |
|--------|---------|----------|---------|
| `.validator.ts` | Validation | Validation rules & logic | `user.validator.ts` |
| `.schema.ts` | Schema validation | Schema definitions | `user.schema.ts` |
| `.guard.ts` | Validation guard | Pre-condition checks | `user.guard.ts` |

**Examples:**

```typescript
// user.validator.ts
export class UserValidator {
  validateEmail(email: string): boolean { }
  validatePassword(password: string): ValidationError[] { }
  validateUserData(data: CreateUserDto): ValidationResult { }
}

// user.schema.ts
export const userSchema = {
  email: { type: 'string', format: 'email' },
  password: { type: 'string', minLength: 8 },
  name: { type: 'string' }
};

// user.guard.ts
export class UserGuard {
  canDelete(user: User): boolean { }
  canEdit(user: User, target: User): boolean { }
}
```

### Transformation Layer

| Suffix | Purpose | Contains | Example |
|--------|---------|----------|---------|
| `.mapper.ts` | Mapping | Entity ↔ DTO conversion | `user.mapper.ts` |
| `.converter.ts` | Format conversion | Type conversion | `user.converter.ts` |
| `.transformer.ts` | Data transformation | Complex transformation | `user.transformer.ts` |
| `.serializer.ts` | Serialization | Object serialization | `user.serializer.ts` |

**Examples:**

```typescript
// user.mapper.ts
export class UserMapper {
  toDto(user: User): UserResponseDto { }
  fromDto(dto: CreateUserDto): User { }
  toPersistence(user: User): UserEntity { }
}

// user.converter.ts
export class UserConverter {
  stringToUser(json: string): User { }
  userToString(user: User): string { }
}

// user.transformer.ts
export class UserTransformer {
  enrichWithMetadata(users: User[]): EnrichedUser[] { }
  flattenNested(user: NestedUser): FlatUser { }
}
```

### API/HTTP Layer

| Suffix | Purpose | Contains | Example |
|--------|---------|----------|---------|
| `.controller.ts` | HTTP controller | Route handlers | `user.controller.ts` |
| `.client.ts` | HTTP client | External API calls | `http.client.ts` |
| `.api.ts` | API client | API-specific client | `user.api.ts` |
| `.routes.ts` | Route definitions | Route configuration | `user.routes.ts` |
| `.middleware.ts` | Middleware | HTTP middleware | `auth.middleware.ts` |

**Examples:**

```typescript
// http.client.ts — Generic HTTP client
export class HttpClient {
  async get<T>(url: string): Promise<T> { }
  async post<T>(url: string, data: any): Promise<T> { }
  async put<T>(url: string, data: any): Promise<T> { }
  async delete(url: string): Promise<void> { }
}

// user.controller.ts — HTTP routes
export class UserController {
  @Get('/users/:id')
  async getUser(req: Request): Promise<UserResponseDto> { }

  @Post('/users')
  async createUser(req: Request): Promise<UserResponseDto> { }
}

// user.api.ts — API-specific operations
export class UserApi {
  async fetchUser(id: number): Promise<User> { }
  async updateUser(id: number, data: Partial<User>): Promise<User> { }
}

// auth.middleware.ts — HTTP middleware
export const authMiddleware = (req: Request, res: Response, next: NextFunction) => { }
```

### ❌ Don't Use Generic `.service.ts` for HTTP

**Bad example:**
```typescript
// ❌ http.service.ts — Too vague!
export class HttpService {
  async request() { }
}
```

**Good example:**
```typescript
// ✅ http.client.ts — Clear and specific
export class HttpClient {
  async request() { }
}
```

### Utilities

| Suffix | Purpose | Contains | Example |
|--------|---------|----------|---------|
| `.utils.ts` | Utilities | Helper functions | `user.utils.ts` |
| `.helpers.ts` | Helpers | Helper functions | `user.helpers.ts` |
| `.constants.ts` | Constants | Constant values | `user.constants.ts` |
| `.types.ts` | Types | Type definitions | `user.types.ts` |

**Examples:**

```typescript
// user.utils.ts
export const formatUserName = (user: User): string => { }
export const isAdult = (user: User): boolean => { }

// user.constants.ts
export const MIN_PASSWORD_LENGTH = 8;
export const ADMIN_ROLES = ['admin', 'super-admin'];

// user.types.ts
export type UserStatus = 'active' | 'suspended' | 'deleted';
export type UserRole = 'admin' | 'user';
```

### Testing

| Suffix | Purpose | Contains | Example |
|--------|---------|----------|---------|
| `.test.ts` | Test suite | Unit/integration tests | `user.repo.test.ts` |
| `.mock.ts` | Mock objects | Test mocks | `user.mock.ts` |
| `.fixture.ts` | Test data | Test fixtures | `user.fixture.ts` |

**Examples:**

```typescript
// user.repo.test.ts
describe('UserRepository', () => {
  it('should find user by id', () => { })
})

// user.mock.ts
export const mockUserRepository = {
  findById: jest.fn(),
  save: jest.fn()
}

// user.fixture.ts
export const validUserFixture = {
  id: 1,
  email: 'test@example.com',
  name: 'Test User'
}
```

## Complete Example: User Module

```
src/user/
├── user.ts                 ← Interfaces and types
├── user.types.ts           ← Additional type definitions
├── user.constants.ts       ← Constants (ROLES, STATUSES)
├── user.repo.ts            ← Database repository
├── user.cache.ts           ← Caching layer
├── user.validator.ts       ← Validation rules
├── user.guard.ts           ← Pre-condition checks
├── user.mapper.ts          ← DTO mapping
├── user.logic.ts           ← Business logic
├── user.manager.ts         ← Orchestration (coordinates repo, cache, logic)
├── user.controller.ts      ← HTTP routes
├── user.api.ts             ← External API calls (if needed)
├── user.utils.ts           ← Helper functions
├── user.repo.test.ts       ← Repository tests
├── user.logic.test.ts      ← Logic tests
├── user.validator.test.ts  ← Validator tests
├── user.mock.ts            ← Test mocks
├── user.fixture.ts         ← Test data
└── index.ts                ← Optional exports
```

## Suffix Priority & Layering

**Database Layer** (most specific about data)
```
.repo.ts → .dao.ts → .query.ts
```

**Business Logic Layer**
```
.logic.ts → .manager.ts → .handler.ts
```

**Validation Layer**
```
.validator.ts → .schema.ts → .guard.ts
```

**Transformation Layer**
```
.mapper.ts → .converter.ts → .transformer.ts
```

**API Layer**
```
.controller.ts → .api.ts → .routes.ts
```

**Utilities**
```
.utils.ts → .helpers.ts → .constants.ts
```

## Common Suffix Mistakes

### ❌ Wrong Suffix Usage

```
user.service.ts        ← Too generic (what service?)
user.handler.ts        ← For business logic (should be .logic.ts)
user.factory.ts        ← For DTO creation (should be .mapper.ts)
user.provider.ts       ← For repository (should be .repo.ts)
user.adapter.ts        ← For API client (should be .api.ts)
```

### ✅ Correct Suffix Usage

```
user.repo.ts           ← Database operations
user.logic.ts          ← Business rules
user.mapper.ts         ← Entity ↔ DTO conversion
user.api.ts            ← External API calls
user.controller.ts     ← HTTP routes
user.validator.ts      ← Input validation
```

## When to Create a New File vs Adding to Existing

### ✅ Create new file when:

1. **Different responsibility**
   ```
   user.logic.ts      ← Business logic
   user.validator.ts  ← Validation (separate)
   user.mapper.ts     ← Mapping (separate)
   ```

2. **File gets too large** (>200 lines)
   ```
   user.repo.ts       ← Repository only
   user.cache.ts      ← Caching only (split out)
   ```

3. **Different testing strategy**
   ```
   user.repo.ts       ← Database tests
   user.logic.ts      ← Logic tests (different setup)
   ```

### ❌ Don't create new file when:

1. **Tiny utility function**
   ```
   // DON'T: user.formatName.ts (too small)
   // DO: Add to user.utils.ts
   ```

2. **Only one related function**
   ```
   // DON'T: user.isAdmin.ts (too specific)
   // DO: Add to user.guard.ts
   ```

## Migration Guide: From .service.ts

If you have existing `.service.ts` files, split them:

### Before:
```typescript
// user.service.ts (500 lines!)
export class UserService {
  // Database operations
  async save(user: User) { }
  async findById(id: number) { }

  // Business logic
  canDelete(user: User) { }
  isEligibleForPremium(user: User) { }

  // Validation
  validateEmail(email: string) { }
  validatePassword(password: string) { }

  // Mapping
  toDto(user: User) { }
  fromDto(dto: CreateUserDto) { }

  // Caching
  invalidateCache(id: number) { }
}
```

### After:
```
user/
├── user.repo.ts           ← Database operations
├── user.logic.ts          ← Business logic
├── user.validator.ts      ← Validation
├── user.mapper.ts         ← Mapping
├── user.cache.ts          ← Caching
└── user.manager.ts        ← Orchestrates all of above
```

---

## Quick Checklist

When creating a new file, ask:

- [ ] Does it access the database? → `.repo.ts`
- [ ] Does it validate data? → `.validator.ts`
- [ ] Does it transform data? → `.mapper.ts`
- [ ] Does it handle HTTP? → `.controller.ts`
- [ ] Does it implement business logic? → `.logic.ts`
- [ ] Does it call external APIs? → `.api.ts`
- [ ] Does it have helper functions? → `.utils.ts`
- [ ] Does it define constants? → `.constants.ts`
- [ ] Does it orchestrate multiple layers? → `.manager.ts`

---

## Related

- [Files & Folders Naming](files-folders.md)
- [Anti-Patterns](anti-patterns.md)
- [Module Organization](../06-architecture/module-organization.md)
