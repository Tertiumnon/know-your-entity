# Naming Functions & Methods

Use verbs that reflect the semantic meaning of the action. Consistent verb choices across your codebase make functions predictable and intuitive.

## Standard Verbs

### `get` — Return an existing value

Usually synchronous or from cache. Implies the value already exists.

```typescript
function getUser(id: number): User | undefined { }
function getConfig(): Config { }
function getFromCache(key: string): any { }
```

### `set` — Assign or modify a value

```typescript
function setUser(user: User): void { }
function setConfig(config: Config): void { }
function setActiveUser(user: User): void { }
```

### `find` — Locate an element matching criteria

Returns `null` or `undefined` if not found. May return multiple results.

```typescript
function findUser(filters: Filter[]): User | null { }
function findUsers(email: string): User[] { }
function findById(id: number): Entity | undefined { }
```

### `fetch` / `load` — Retrieve data (usually async)

Implies a network request or async operation. Different from `get` which is synchronous.

```typescript
async function fetchUsers(): Promise<User[]> { }
async function loadConfig(): Promise<Config> { }
async function loadUserProfile(id: number): Promise<UserProfile> { }
```

### CRUD Operations

Standard database/API operations should use clear verbs:

```typescript
// CREATE
async function create(data: CreateUserDto): Promise<User> { }
async function createUser(data: CreateUserDto): Promise<User> { }

// READ / GET
async function get(id: number): Promise<User> { }
async function getById(id: number): Promise<User> { }
async function find(filters: Filter[]): Promise<User[]> { }

// UPDATE
async function update(id: number, data: UpdateUserDto): Promise<User> { }
async function updateUser(id: number, data: UpdateUserDto): Promise<User> { }

// DELETE
async function delete(id: number): Promise<void> { }
async function deleteUser(id: number): Promise<void> { }
```

## Compound Operations

When combining verbs, use clear sequences:

```typescript
function findAndRemove(items: Item[], predicate: (item: Item) => boolean): Item[] {
  // Find matching items and remove them
}

function getOrCreate(id: number): User {
  // Get existing or create new
}

function fetchAndCache(url: string): Promise<Data> {
  // Fetch from remote and store locally
}
```

## Negative Actions

For deletion or deactivation, be explicit:

```typescript
// GOOD
function remove(item: Item): void { }
function deactivate(user: User): void { }
function disable(feature: string): void { }
function clear(cache: Cache): void { }

// AVOID — Double negatives are confusing
function unset() { }        // What's being unset?
function notActive() { }    // Is this a getter or setter?
```

## Boolean-Returning Functions

Use `is`, `has`, `can`, or `should`:

```typescript
function isActive(user: User): boolean { }
function hasPermission(user: User, permission: string): boolean { }
function canEdit(user: User, item: Item): boolean { }
function shouldUpdate(oldData: Data, newData: Data): boolean { }
```

## Examples in a Service

```typescript
class UserService {
  // Get existing user (sync/cached)
  getUser(id: number): User | undefined { }

  // Find users by criteria
  find(filters: Filter[]): Promise<User[]> { }

  // Get one user (could be sync or async, context-dependent)
  get(id: number): Promise<User> { }

  // Fetch from remote API (definitely async)
  async fetchUsers(): Promise<User[]> { }

  // Check conditions
  isAdmin(user: User): boolean { }
  canDelete(user: User, target: User): boolean { }

  // Create/Update/Delete
  async create(data: CreateUserDto): Promise<User> { }
  async update(id: number, data: UpdateUserDto): Promise<User> { }
  async delete(id: number): Promise<void> { }
}
```

## Consistency is Key

Choose a verb set and apply it consistently across your project. If `find` is used in one service, don't switch to `search` in another without reason.

```typescript
// INCONSISTENT — Confusing
class UserService {
  find(filters: Filter[]): User[] { }
}

class ProductService {
  search(filters: Filter[]): Product[] { }
}

// CONSISTENT — Clear and predictable
class UserService {
  find(filters: Filter[]): User[] { }
}

class ProductService {
  find(filters: Filter[]): Product[] { }
}
```

---

## Related

- [Overview](overview.md)
- [Variables](variables.md)
- [Classes & Interfaces](classes-interfaces.md)
