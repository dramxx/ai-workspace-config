---
name: project-structure
description: Guides React/Next.js/TypeScript project organization using feature-based architecture. Use when structuring new projects, reorganizing codebases, or deciding where to place new code.
---

# Project Structure: Feature-Based Architecture

## Core Principle

**Organize code by feature/domain, not by file type. Enforce unidirectional code flow: shared → features → app.**

This approach scales well for medium-to-large React, Next.js, and TypeScript projects while keeping features independent and maintainable.

## Top-Level Structure

```
src/
├── app/                # Application layer (routing, providers)
│   ├── routes/         # Route definitions / pages
│   ├── app.tsx         # Main application component
│   ├── provider.tsx    # Global providers wrapper
│   └── router.tsx      # Router configuration
├── assets/             # Static files (images, fonts)
├── components/         # Shared UI components
├── config/             # Global configuration, env variables
├── features/           # Feature-based modules (main code lives here)
├── hooks/              # Shared hooks
├── lib/                # Pre-configured libraries (axios, dayjs, etc.)
├── stores/             # Global state stores
├── testing/            # Test utilities and mocks
├── types/              # Shared TypeScript types
└── utils/              # Shared utility functions
```

## Feature Structure

Each feature is a self-contained module:

```
src/features/users/
├── api/           # API requests & React Query hooks
├── components/    # Feature-scoped UI components
├── hooks/         # Feature-scoped hooks
├── stores/        # Feature state (Zustand, etc.)
├── types/         # Feature-specific types
└── utils/         # Feature utility functions
```

**Only include folders you need.** Don't create empty folders "just in case."

## Import Rules

### Unidirectional Flow
```
shared → features → app
```

- **Shared** can be imported anywhere
- **Features** can import from shared and other features (carefully)
- **App** can import from features and shared
- **Never** import from app into features

### Import Examples

```typescript
// ✅ Good: Feature imports from shared
import { Button } from '@/components/Button';
import { useApi } from '@/hooks/useApi';

// ✅ Good: App imports from features
import { UserList } from '@/features/users/components/UserList';

// ❌ Bad: Feature imports from app
import { AppLayout } from '@/app/app'; // Don't do this
```

## Folder-by-Folder Guide

### `/app` - Application Layer
- **Purpose**: Routing, global providers, app-level concerns
- **Contents**: Pages, layouts, providers, router config
- **Imports**: Can import from anywhere

### `/features` - Feature Modules
- **Purpose**: Business logic and UI for specific domains
- **Structure**: Self-contained with api, components, hooks, types
- **Imports**: Shared utilities, other features (minimal)

### `/components` - Shared UI
- **Purpose**: Reusable UI components
- **Examples**: Button, Input, Modal, Loading
- **Rule**: No business logic, pure UI

### `/hooks` - Shared Hooks
- **Purpose**: Reusable logic across features
- **Examples**: useApi, useLocalStorage, useDebounce
- **Rule**: No feature-specific business logic

### `/lib` - Configured Libraries
- **Purpose**: Third-party libraries with custom configuration
- **Examples**: axios instance, dayjs setup, theme provider
- **Rule**: Wrappers, not direct exports

### `/stores` - Global State
- **Purpose**: Cross-feature state management
- **Examples**: auth store, theme store, cart store
- **Rule**: Minimal, prefer local state first

## Naming Conventions

### Files
- **Components**: PascalCase (`UserProfile.tsx`)
- **Hooks**: camelCase with "use" prefix (`useUserData.ts`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Types**: camelCase with "types" suffix (`userTypes.ts`)

### Folders
- **Features**: plural, domain name (`users`, `products`, `auth`)
- **Components**: singular or descriptive (`button`, `form`, `modal`)
- **Utilities**: descriptive (`api`, `formatting`, `validation`)

## When to Create a Feature

Create a new feature when you have:

1. **Multiple related components** (2+ screens/components)
2. **Business logic** that's self-contained
3. **API endpoints** specific to a domain
4. **State management** for a domain

**Examples**: users, products, auth, payments, dashboard

**Don't create features for**: shared utilities, UI components, app infrastructure

## Common Patterns

### API Organization
```typescript
// features/users/api/index.ts
export { useUsers } from './useUsers';
export { useCreateUser } from './useCreateUser';
export { useUpdateUser } from './useUpdateUser';

// features/users/api/useUsers.ts
export const useUsers = () => {
  return useQuery(['users'], fetchUsers);
};
```

### Component Organization
```typescript
// features/users/components/UserList.tsx
import { useUsers } from '../api';

export const UserList = () => {
  const { data: users, isLoading } = useUsers();
  
  if (isLoading) return <Loading />;
  
  return (
    <div>
      {users?.map(user => <UserItem key={user.id} user={user} />)}
    </div>
  );
};
```

### Type Organization
```typescript
// features/users/types/index.ts
export type { User } from './User';
export type { CreateUserInput } from './CreateUserInput';

// features/users/types/User.ts
export interface User {
  id: string;
  name: string;
  email: string;
}
```

## Migration Strategy

When migrating from file-type organization:

1. **Start with new features** using feature-based structure
2. **Gradually move existing code** feature by feature
3. **Update imports** as you move files
4. **Delete old folders** when empty

This approach prevents breaking changes and allows gradual adoption.
