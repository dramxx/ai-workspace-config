---
name: typescript-best-practices
description: Comprehensive TypeScript guidance for both frontend and backend development. Covers strict configuration, type system patterns, async code, runtime validation with zod, React component typing, Node.js tsconfig, and common pitfalls. Always trigger for any TypeScript task — .ts/.tsx files, tsconfig questions, type design, zod schemas, React prop types, or any TS error.
---

# TypeScript Best Practices

Full-stack TypeScript guide covering both Node.js backend and React frontend patterns.

---

## 1. Project Configuration

### Base (shared)

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

> Match `target` to your Node version: Node 18 → `ES2022`, Node 20+ → `ES2023`.
> `NodeNext` module resolution is required for ESM and `.js` extension imports.

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

---

## 2. Type System Fundamentals

### Let inference work — annotate only where it can't

```typescript
// Good — inference works
const name = "Alice";
const items = ["a", "b", "c"];
const user = { id: 1, name: "Alice" };

// Required — function signatures and class properties
function calculateTotal(items: CartItem[], taxRate: number): number {
  return items.reduce((sum, item) => sum + item.price, 0) * (1 + taxRate);
}
```

### Interface vs Type

```typescript
// interface — object shapes, extendable, declaration merging
interface User {
  id: string;
  name: string;
}
interface Admin extends User {
  permissions: string[];
}

// type — unions, intersections, tuples, mapped types, primitives
type Status = "pending" | "approved" | "rejected";
type Point = [number, number];
type UserId = string;
type ReadonlyUser = Readonly<User>;
```

### unknown over any

```typescript
// Bad — defeats type checking
function process(data: any) {
  return data.toUpperCase(); // no error, may crash
}

// Good — forces explicit narrowing
function process(data: unknown): string {
  if (typeof data === "string") return data.toUpperCase();
  throw new Error("Expected string");
}
```

### Discriminated Unions for State

```typescript
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

function render<T>(state: AsyncState<T>) {
  switch (state.status) {
    case "success":
      return state.data; // data is T
    case "error":
      return state.error.message; // error is Error
  }
}
```

### Const Assertions

```typescript
const ROUTES = ["home", "settings", "profile"] as const;
type Route = (typeof ROUTES)[number]; // "home" | "settings" | "profile"

const config = { apiUrl: "https://api.example.com", timeout: 5000 } as const;
```

### Satisfies — validate without widening

```typescript
// satisfies checks compatibility but preserves literal types
const handlers = {
  click: (e: MouseEvent) => console.log("clicked"),
  focus: (e: FocusEvent) => console.log("focused"),
} satisfies Record<string, (e: Event) => void>;
```

---

## 3. Async/Await

```typescript
// Always await in parallel when independent
const [user, orders] = await Promise.all([fetchUser(id), fetchOrders(id)]);

// Flat chains — never nest .then()
async function good() {
  const user = await fetchUser();
  const orders = await fetchOrders(user.id);
  return processOrders(orders);
}

// Error handling — wrap and rethrow with context
async function fetchUser(id: string): Promise<User> {
  try {
    return (await api.get(`/users/${id}`)).data;
  } catch (error) {
    if (error instanceof NotFoundError) throw new UserNotFoundError(id);
    throw error;
  }
}
```

---

## 4. Runtime Validation with Zod (BE + FE)

The model knows nothing about incoming data — validate at boundaries (API input, env vars, form data).

```typescript
import { z } from "zod";

// Define schema
const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(["admin", "user"]).default("user"),
  age: z.number().int().min(0).optional(),
});

// Derive type from schema — single source of truth
type CreateUserInput = z.infer<typeof CreateUserSchema>;

// Parse — throws ZodError on invalid input
const input = CreateUserSchema.parse(req.body);

// SafeParse — returns { success, data } | { success: false, error }
const result = CreateUserSchema.safeParse(req.body);
if (!result.success) {
  return res.status(400).json({ errors: result.error.flatten() });
}
```

### Environment Variables

```typescript
// Never trust process.env — always validate
const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(["development", "test", "production"]),
  API_SECRET: z.string().min(32),
});

export const env = EnvSchema.parse(process.env);
// env.PORT is number, env.NODE_ENV is a literal union — fully typed
```

---

## 5. Guard Clauses

```typescript
// Bad — deep nesting
function processOrder(order: Order | null) {
  if (order) {
    if (order.items.length > 0) {
      if (order.status === "pending") {
        // actual logic buried here
      }
    }
  }
}

// Good — guard clauses flatten the function
function processOrder(order: Order | null): ProcessedOrder {
  if (!order) throw new Error("Order required");
  if (order.items.length === 0) throw new Error("Order must have items");
  if (order.status !== "pending") throw new Error("Order already processed");

  return { ...order, status: "processed" };
}
```

---

## 6. Advanced Types (use when they pay their weight)

```typescript
// Conditional types
type ArrayElement<T> = T extends (infer U)[] ? U : never;
type Awaited<T> = T extends Promise<infer U> ? U : T;

// Template literal types
type EventName = `on${Capitalize<string>}`;
type ApiRoute = `/api/${string}`;

// Branded types — prevent mixing IDs of different entities
type Brand<T, B> = T & { __brand: B };
type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

function getUser(id: UserId) { ... }

const orderId = "123" as OrderId;
getUser(orderId); // TS error — can't pass OrderId where UserId expected

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

---

## 7. React Component Typing (FE)

### Component Props

```typescript
// Prefer plain function over React.FC — no implicit children, better generics
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: "primary" | "secondary";
}

function Button({ label, onClick, disabled = false, variant = "primary" }: ButtonProps) {
  return <button onClick={onClick} disabled={disabled}>{label}</button>;
}

// When you need children
interface CardProps {
  title: string;
  children: React.ReactNode;   // any renderable — string, element, fragment, null
}
```

### Extending HTML Elements

```typescript
// Extend native element props — don't redefine what's already there
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

// ComponentProps — extract props from an existing component
type ButtonProps = React.ComponentProps<typeof Button>;
type DivProps = React.ComponentProps<"div">;
```

### Event Handlers

```typescript
// Always type events explicitly
function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
  setValue(e.target.value);
}

function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
  e.preventDefault();
}

function handleClick(e: React.MouseEvent<HTMLButtonElement>) {
  e.stopPropagation();
}

// Inline — TS infers from JSX context, no annotation needed
<input onChange={(e) => setValue(e.target.value)} />
```

### useRef

```typescript
// DOM ref — initialize with null, assert non-null on use
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus(); // safe — optional chaining
inputRef.current!.focus(); // if you're certain it's mounted

// Mutable value ref (not a DOM node)
const timerRef = useRef<ReturnType<typeof setTimeout> | null>(null);
```

### Generic Components

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

---

## 8. Imports

```typescript
// Type-only imports — stripped at compile time, better tree-shaking
import type { User, Order } from "./types";
import { fetchUser } from "./api";

// Re-exports
export type { User, Order };

// Prefer direct imports — barrel files hide module boundaries and cause cycles
import { UserService } from "./user/user.service"; // good
import { UserService } from "./user"; // use only if project already does this
```

---

## 9. Dependency Injection for Testability

```typescript
interface PaymentGateway {
  charge(amount: number): Promise<boolean>;
}

class PaymentProcessor {
  constructor(private gateway: PaymentGateway) {}

  async processPayment(amount: number): Promise<boolean> {
    if (amount <= 0) throw new Error("Amount must be positive");
    return this.gateway.charge(amount);
  }
}

// Test
const mockGateway: PaymentGateway = {
  charge: jest.fn().mockResolvedValue(true),
};
const processor = new PaymentProcessor(mockGateway);
```

---

## Common Mistakes

| Mistake                    | Problem                           | Fix                                |
| -------------------------- | --------------------------------- | ---------------------------------- |
| `any`                      | Disables type checking            | `unknown` + narrowing, or generics |
| No strict mode             | Misses whole classes of errors    | `"strict": true`                   |
| Redundant annotations      | Clutters code                     | Trust inference on local vars      |
| Raw `process.env` access   | `string \| undefined` silently    | Validate with zod at startup       |
| `React.FC`                 | Implicit children, worse generics | Plain function + explicit props    |
| `useRef<T>()` without null | Wrong type for DOM refs           | `useRef<T>(null)`                  |
| Nested async chains        | Hard to follow                    | Flat `await` statements            |
| Barrel files everywhere    | Import cycles, slower builds      | Direct imports by default          |

---

## Quick Reference

```typescript
// Infer locals, annotate function signatures
const x = 42;
function fn(a: string): number { ... }

// Unions + narrowing
type Result<T> = { ok: true; data: T } | { ok: false; error: string };

// Zod — validate at boundaries
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

// Null safety
const len = str?.length ?? 0;
```

---

## References

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [Zod Docs](https://zod.dev)
- [TypeScript Performance Wiki](https://github.com/microsoft/TypeScript/wiki/Performance)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
