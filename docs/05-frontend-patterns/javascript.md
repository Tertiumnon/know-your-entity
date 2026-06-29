# JavaScript Naming & Patterns

Conventions for JavaScript/TypeScript code in the frontend.

## Naming in JavaScript

JavaScript uses **camelCase** for variables and identifiers, while HTML uses **snake_case** for attribute names.

### Selecting Elements

Use camelCase for JavaScript variables, even when accessing HTML elements with snake_case names:

```typescript
// HTML has snake_case names
<input name="secret_name" />
<div class="user_profile">Profile</div>

// JavaScript uses camelCase
const secretName = document.querySelector('input[name="secret_name"]');
const userProfile = document.querySelector('.user_profile');
```

### Variable Naming

Follow the general naming conventions (see [Variables](../02-naming-conventions/variables.md)):

```typescript
// Good
const userName = 'Alice';
const isActive = true;
const userEmails: string[] = [];
const currentUser = await fetchUser();

// Bad
const user_name = 'Alice';           // snake_case in JS
const uName = 'Alice';               // Unclear abbreviation
const userNameString: string = '';   // Type in name
```

## DOM Manipulation

### Safe Element Selection

```typescript
// Get single element
const button = document.querySelector('.submit_btn');
if (button) {
  button.addEventListener('click', handleSubmit);
}

// Get multiple elements
const items = document.querySelectorAll('.list_item');
items.forEach(item => {
  item.addEventListener('hover', handleHover);
});
```

### Update Classes

Use the `classList` API for managing modifiers:

```typescript
// Add/remove classes
element.classList.add('active');
element.classList.remove('hidden');
element.classList.toggle('disabled');

// Check if class exists
if (element.classList.contains('selected')) {
  // ...
}
```

### Data Attributes

Use `data-*` attributes for custom data:

```html
<div class="card" data-card-id="123" data-status="active">
  Card content
</div>
```

```typescript
const card = document.querySelector('.card');
const cardId = card.dataset.cardId;      // "123"
const status = card.dataset.status;      // "active"
```

## Component Structure (Frontend)

### React Example

```typescript
// components/UserCard/UserCard.tsx
interface UserCardProps {
  userId: number;
  isSelected?: boolean;
}

export const UserCard: React.FC<UserCardProps> = ({
  userId,
  isSelected = false
}) => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  if (!user) return <div>Loading...</div>;

  return (
    <div className={`user_card ${isSelected ? 'user_card--selected' : ''}`}>
      <div className="user_card__avatar">
        <img src={user.avatar} alt={user.name} />
      </div>
      <div className="user_card__content">
        <h3>{user.name}</h3>
        <p className="user_card__email">{user.email}</p>
      </div>
    </div>
  );
};
```

### Vue Example

```vue
<template>
  <div :class="['user_card', { 'user_card--selected': isSelected }]">
    <div class="user_card__avatar">
      <img :src="user.avatar" :alt="user.name" />
    </div>
    <div class="user_card__content">
      <h3>{{ user.name }}</h3>
      <p class="user_card__email">{{ user.email }}</p>
    </div>
  </div>
</template>

<script>
import { reactive, computed } from 'vue';

export default {
  props: {
    userId: Number,
    isSelected: Boolean
  },
  setup(props) {
    const user = reactive({});

    const fetchUser = async () => {
      // Load user data
    };

    return {
      user
    };
  }
};
</script>

<style scoped lang="less">
.user_card {
  display: flex;
  gap: 12px;

  &.user_card--selected {
    border: 2px solid var(--primary-color);
  }

  .user_card__avatar {
    width: 48px;
    height: 48px;
  }

  .user_card__content {
    flex: 1;
  }

  .user_card__email {
    color: var(--text-secondary);
  }
}
</style>
```

## Event Handlers

Use clear names that indicate the action:

```typescript
const handleClick = (event: MouseEvent) => {
  // Handle click
};

const handleSubmit = (data: FormData) => {
  // Handle form submission
};

const handleInputChange = (value: string) => {
  // Handle input change
};

const handleUserSelected = (user: User) => {
  // Handle user selection
};
```

### Event Listener Management

```typescript
class FormManager {
  private handleSubmit = (event: Event) => {
    event.preventDefault();
    this.submitForm();
  };

  private handleReset = (event: Event) => {
    event.preventDefault();
    this.resetForm();
  };

  mount() {
    const form = document.querySelector('form');
    form?.addEventListener('submit', this.handleSubmit);
    form?.addEventListener('reset', this.handleReset);
  }

  unmount() {
    const form = document.querySelector('form');
    form?.removeEventListener('submit', this.handleSubmit);
    form?.removeEventListener('reset', this.handleReset);
  }
}
```

## State Management

### Using Local State

```typescript
// React
const [isOpen, setIsOpen] = useState(false);
const [selectedUser, setSelectedUser] = useState<User | null>(null);
const [users, setUsers] = useState<User[]>([]);

// Vue
const state = reactive({
  isOpen: false,
  selectedUser: null as User | null,
  users: [] as User[]
});
```

### Constants

Keep UI constants organized:

```typescript
const UI_CONSTANTS = {
  ANIMATION_DURATION: 300,
  DEBOUNCE_DELAY: 500,
  MAX_ITEMS_PER_PAGE: 20,
  MIN_PASSWORD_LENGTH: 8
};

const MESSAGES = {
  ERROR_NETWORK: 'Network error. Please try again.',
  ERROR_VALIDATION: 'Please check your input.',
  SUCCESS_SAVED: 'Changes saved successfully.'
};
```

## Utilities

Keep utility functions focused and well-named:

```typescript
// formatters.ts
export const formatDate = (date: Date): string => {
  return date.toLocaleDateString('en-US');
};

export const formatCurrency = (amount: number): string => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(amount);
};

// validators.ts
export const isValidEmail = (email: string): boolean => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
};

export const isValidPassword = (password: string): boolean => {
  return password.length >= UI_CONSTANTS.MIN_PASSWORD_LENGTH;
};

// api.ts
export const fetchUser = async (id: number): Promise<User> => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

export const updateUser = async (id: number, data: Partial<User>): Promise<User> => {
  const response = await fetch(`/api/users/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return response.json();
};
```

## Hooks (React)

Hooks should follow a clear naming pattern:

```typescript
// useUser.ts
export const useUser = (userId: number) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const loadUser = async () => {
      try {
        const data = await fetchUser(userId);
        setUser(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    loadUser();
  }, [userId]);

  return { user, loading, error };
};

// useForm.ts
export const useForm = <T extends Record<string, any>>(initialValues: T) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});

  const handleChange = (field: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [field]: value }));
  };

  const handleSubmit = async (onSubmit: (values: T) => Promise<void>) => {
    try {
      await onSubmit(values);
    } catch (err) {
      setErrors({ ...errors });
    }
  };

  return { values, errors, handleChange, handleSubmit };
};

// Usage
const MyForm = () => {
  const { values, handleChange, handleSubmit } = useForm({
    email: '',
    name: ''
  });

  return (
    <form onSubmit={e => {
      e.preventDefault();
      handleSubmit(async (data) => {
        await submitForm(data);
      });
    }}>
      {/* form fields */}
    </form>
  );
};
```

## Type Safety

Use TypeScript interfaces for component props and data:

```typescript
interface UserCardProps {
  user: User;
  onSelect?: (user: User) => void;
  isSelected?: boolean;
}

interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
}

const UserCard: React.FC<UserCardProps> = ({
  user,
  onSelect,
  isSelected = false
}) => {
  return (
    <div
      className={`user_card ${isSelected ? 'user_card--selected' : ''}`}
      onClick={() => onSelect?.(user)}
    >
      {/* render user */}
    </div>
  );
};
```

---

## Related

- [HTML & CSS](html-css.md)
- [Variables Naming](../02-naming-conventions/variables.md)
- [Functions Naming](../02-naming-conventions/functions-methods.md)
