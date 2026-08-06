# 08 — TypeScript

## Types, Generics, Utility Types

---

### What is TypeScript?

TypeScript is JavaScript with static type checking. It catches errors at compile time, not runtime.

---

### Basic Types

```typescript
// Primitives
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;

// Arrays
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"];

// Tuples
let tuple: [string, number] = ["Alice", 30];

// Object
let user: { name: string; age: number } = { name: "Alice", age: 30 };

// Any (avoid using)
let anything: any = "hello";

// Unknown (safer than any)
let value: unknown = "hello";
if (typeof value === "string") {
    value.toUpperCase();  // OK after narrowing
}

// Void and Never
function log(msg: string): void { console.log(msg); }
function throwError(msg: string): never { throw new Error(msg); }
```

---

### Interfaces & Types

```typescript
// Interface
interface User {
    id: number;
    name: string;
    email: string;
    age?: number;           // Optional
    readonly createdAt: Date;  // Read-only
}

// Extending
interface Admin extends User {
    role: "admin";
    permissions: string[];
}

// Union types
type Status = "active" | "inactive" | "banned";
type ID = string | number;
```

---

### Generics

```typescript
// Generic function
function identity<T>(value: T): T {
    return value;
}
identity<string>("hello");
identity(42);  // Type inferred

// Generic interface
interface ApiResponse<T> {
    data: T;
    status: number;
}

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}
```

---

### Utility Types

```typescript
interface User { id: number; name: string; email: string; age: number; }

type PartialUser = Partial<User>;           // All optional
type RequiredUser = Required<PartialUser>;   // All required
type ReadonlyUser = Readonly<User>;          // All readonly
type UserPreview = Pick<User, "id" | "name">;  // Select props
type UserWithoutEmail = Omit<User, "email">;   // Exclude props
type UserRoles = Record<string, "admin" | "user">;  // Construct type
```

---

### Interview Questions

**Q: What's the difference between `interface` and `type`?**

A: "Interfaces: extendable, support declaration merging, best for objects. Types: can represent unions, intersections, primitives, more flexible. Use interfaces for object shapes; types for complex types."

**Q: What are generics?**

A: "Type variables for reusable components. function identity<T>(value: T): T. The type is specified when calling or inferred. Used for type-safe collections, API responses, and utility functions."

---

*Next: [09 — React Fundamentals](09-React-Fundamentals.md)*
