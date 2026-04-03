---
description: TypeScript best practices and patterns for full-stack development
---

# TypeScript Best Practices

## Project Configuration

### Base tsconfig.json
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "exactOptionalPropertyTypes": true,
    "skipLibCheck": true
  }
}
```

### Backend (Node.js)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### Frontend (Vite/React)
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "noEmit": true
  }
}
```

## Type System Fundamentals

### Let inference work
```typescript
// Good - inference works
const name = "Alice";
const items = ["a", "b", "c"];
const user = { id: 1, name: "Alice" };

// Required - function signatures
function calculateTotal(items: CartItem[], taxRate: number): number {
  return items.reduce((sum, item) => sum + item.price, 0) * (1 + taxRate);
}
```

### Interface vs Type
```typescript
// interface - object shapes, extendable
interface User {
  id: string;
  name: string;
}
interface Admin extends User {
  permissions: string[];
}

// type - unions, intersections, tuples, primitives
type Status = "pending" | "approved" | "rejected";
type Point = [number, number];
type UserId = string;
```

### unknown over any
```typescript
// Bad - defeats type checking
function process(data: any) {
  return data.toUpperCase();
}

// Good - forces explicit narrowing
function process(data: unknown): string {
  if (typeof data === "string") return data.toUpperCase();
  throw new Error("Expected string");
}
```

### Discriminated Unions
```typescript
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };
```

### Const Assertions
```typescript
const ROUTES = ["home", "settings", "profile"] as const;
type Route = (typeof ROUTES)[number];
```

## Async/Await Patterns

```typescript
// Always await in parallel when independent
const [user, orders] = await Promise.all([fetchUser(id), fetchOrders(id)]);

// Flat chains - never nest .then()
async function good() {
  const user = await fetchUser();
  const orders = await fetchOrders(user.id);
  return processOrders(orders);
}

// Error handling - wrap and rethrow with context
async function fetchUser(id: string): Promise<User> {
  try {
    return (await api.get(`/users/${id}`)).data;
  } catch (error) {
    if (error instanceof NotFoundError) throw new UserNotFoundError(id);
    throw error;
  }
}
```

## Runtime Validation with Zod

```typescript
import { z } from "zod";

const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(["admin", "user"]).default("user"),
});

type CreateUserInput = z.infer<typeof CreateUserSchema>;

const input = CreateUserSchema.parse(req.body);
```

### Environment Variables
```typescript
const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(["development", "test", "production"]),
});

export const env = EnvSchema.parse(process.env);
```

## React Component Typing

### Component Props
```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: "primary" | "secondary";
}

function Button({ label, onClick, disabled = false, variant = "primary" }: ButtonProps) {
  return <button onClick={onClick} disabled={disabled}>{label}</button>;
}
```

### Extending HTML Elements
```typescript
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

function Input({ label, error, ...rest }: InputProps) {
  return (
    <div>
      <label>{label}</label>
      <input {...rest} />
      {error && <span>{error}</span>}
    </div>
  );
}
```

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `any` | Disables type checking | `unknown` + narrowing, or generics |
| No strict mode | Misses whole classes of errors | `"strict": true` |
| Redundant annotations | Clutters code | Trust inference on local vars |
| Raw `process.env` access | `string \| undefined` silently | Validate with zod at startup |
| `React.FC` | Implicit children, worse generics | Plain function + explicit props |

## Quick Reference

```typescript
// Infer locals, annotate function signatures
const x = 42;
function fn(a: string): number { ... }

// Unions + narrowing
type Result<T> = { ok: true; data: T } | { ok: false; error: string };

// Zod - validate at boundaries
const Schema = z.object({ name: z.string() });
type Input = z.infer<typeof Schema>;

// React props
interface Props extends React.HTMLAttributes<HTMLDivElement> { label: string }

// Event handlers (inline needs no type)
<input onChange={(e) => setValue(e.target.value)} />

// DOM ref
const ref = useRef<HTMLInputElement>(null);

// Const assertion
const STATUSES = ["active", "inactive"] as const;
type Status = (typeof STATUSES)[number];
```
