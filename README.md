
# know-your-entity

A short project description: consolidated style guides and naming conventions for the project.

See the main article: `KNOW_YOUR_ENTITY.md` for the full guide. Other resources:

- Frontend guide: `FRONTEND_STYLEGUIDE.md`
- Git guide: `GIT_STYLEGUIDE.md`
- Release examples: `mastering-releases/`

---

## REST API Style Guide

### Design — Typical CRUD patterns

- GET `/user` — get all users
- GET `/user/{id}` — get one user by ID
- POST `/user` — create a new user
- PUT `/user/{id}` — update a user (replace)
- PATCH `/user/{id}` — update a user (partial)
- DELETE `/user/{id}` — delete a user

Reference:

- Swagger: <https://swagger.io/>

---

## Git Style Guide

The Git Style Guide has been moved to a separate file: `GIT_STYLEGUIDE.md`.

See `GIT_STYLEGUIDE.md` for branch, commit and release guidance.

---

## Naming (variables, objects, methods, classes, files)

Good naming makes code understandable and maintainable. Below are recommendations and examples.

### Principles

- Names should reflect the entity and purpose, not implementation details.
- Prefer clear, descriptive names over short abbreviations.
- Use consistent casing: `camelCase` for variables/functions, `PascalCase` for classes/interfaces.
- Agree rules with your team and document them.

### Variables (primitives)

Do not encode types in names.

```ts
// BAD
const ageNumber: number = 30;

// GOOD
const age = 30;
```

Prefer descriptive names over ambiguous ones.

```ts
// BAD
const a = 30;

// GOOD
const age = 30;
```

Example showing `response` vs `result`:

```ts
const getUserNames = async () => {
  const result: string[] = [];
  const response = await fetch('/users');
  const users = await response.json();
  if (users?.data) result.push(...users.data.map(user => user.name));
  return result;
}
```

Use abbreviations sparingly and only when well-understood (`id`, `err`).

### Objects and classes

Classes should describe an entity (e.g. `User`). Avoid duplicating the entity name in its properties (`userId` inside `User` is redundant).

```ts
// BAD
class User {
  userId: number;
  userName: string;
}

// GOOD
class User {
  id: number;
  name: string;
}
```

DTO example (corrected JSON):

```json
// GET /user/{id}
{
  "name": "Bob",
  "email": "bob@bob.bo"
}
```

If returning multiple entities:

```json
// GET /user/{id}
{
  "user": {
    "name": "Bob",
    "email": "bob@bob.bo"
  },
  "address": {
    "city": "City",
    "zipCode": "111111"
  }
}
```

### Parameters as objects

Use a dedicated type/interface for parameter groups and use singular type names.

```ts
interface Filter {
  priceFrom?: number;
  priceTo?: number;
}

const filter: Filter = { priceFrom: 0, priceTo: 99 };
const filters: Filter[] = [filter];
```

### Functions and methods

Use verbs that reflect semantics:

- `get` — return an existing value
- `set` — set or change a value
- `find` — locate an item (may return `null`)
- `fetch` / `load` — perform async network calls
- `create` / `update` / `delete` — CRUD

```ts
function getAppUser(): User | undefined {}
function setAppUser(user: User): void {}
function findUser(filters: Filter[]): User | null {}
async function fetchUsers(): Promise<User[]> {}
```

### Classes and suffixes

Use clear suffixes for pattern roles: `Controller`, `Service`, `Repository`. Prefer full names for clarity.

### File naming

File names should reflect contents. Use a consistent style across the project (kebab-case, snake_case, or camelCase depending on conventions).

### Examples & anti-patterns

Bad example (duplicate type in name):

```ts
// BAD
const userListArray: User[] = [];
```

Good example:

```ts
const users: User[] = [];
function findUserByName(name: string) { return users.find(u => u.name === name); }
```

### Naming checklist

- [ ] Name reflects entity and purpose
- [ ] Type is not encoded in the name
- [ ] No redundant duplication in properties
- [ ] Use clear verbs for actions
- [ ] File names match contents and project style

---

If you'd like, I can:

- produce a one-page condensed ruleset (`CONTRIBUTING.md` or `STYLEGUIDE.md`), or
- generate ESLint/TS rules to automate some checks.

Tell me which of those you'd prefer next.
