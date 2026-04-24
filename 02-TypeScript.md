# TypeScript — From Fundamentals to Expert (QA Automation & Node Services)

> Subject #2 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: basic JavaScript familiarity helpful but not required. Estimated time: ~60 hours.
> Target outcome: Write production-grade TypeScript for test frameworks, Playwright suites, and small Node services.

## 1. Overview & Learning Outcomes

- Understand TypeScript's type system as a compile-time safety layer and design tool, not a runtime cost.
- Write strict TypeScript that catches common bugs at compile time (null checks, type mismatches, wrong API usage).
- Use generics, utility types, and mapped types to avoid repetition and build reusable, strongly-typed abstractions.
- Structure a Node.js monorepo with shared types; build ESM modules with proper exports and path aliases.
- Apply type-safe patterns for async code, error handling, and data validation (Zod integration).
- Author Playwright test suites with strong typing for page interactions, fixtures, and test data.
- Build small CLI tools and HTTP services in TypeScript with compile-time confidence.
- Recognize and avoid common type pitfalls: `any`, `as` type assertions, and unchecked null dereferencing.
- Debug TypeScript using source maps and VS Code's native TypeScript support.
- Interview readiness: explain TypeScript philosophy, demonstrate real-world patterns, solve type puzzles on the spot.

## 2. Detailed Syllabus

| # | Module | Hours | Topics |
|---|--------|-------|--------|
| 1 | Setup & Project Structure | 3 | Node 20 LTS, npm/pnpm, tsconfig.json, tsx, TS output targets, ESM vs CJS |
| 2 | Primitives & Basic Types | 4 | string, number, boolean, null, undefined, any, unknown, never, void |
| 3 | Collections: Arrays, Tuples, Enums | 3 | Array<T>, tuple syntax, const assertion, enum patterns, literal types |
| 4 | Objects: Interfaces & Type Aliases | 4 | Interface syntax, types, extends, merge behavior, choosing between them |
| 5 | Union & Intersection Types | 4 | Union operator, exhaustive checks, discriminated unions, intersection merging |
| 6 | Functions: Types & Overloads | 5 | Function signatures, overload declarations, rest/spread, function types as values |
| 7 | Classes & Object-Oriented Patterns | 5 | Constructors, visibility (public/private/#private), readonly, abstract, static |
| 8 | Generics 101 | 5 | Generic functions, constraints, default params, generic classes, `keyof` |
| 9 | Utility Types & Type Manipulation | 5 | Partial, Required, Readonly, Pick, Omit, Record, Exclude, Extract, Awaited, ReturnType |
| 10 | Type Narrowing & Guards | 4 | typeof, instanceof, `in`, custom type guards, assertion functions |
| 11 | Advanced Types & Mapped Types | 5 | Mapped types, conditional types, `infer`, template literal types |
| 12 | Async/Await & Promise Typing | 4 | Promise<T>, Promise.all/allSettled/race, AbortController, async iterators |
| 13 | Error Handling & Validation | 4 | Custom Error classes, Result types, Zod for runtime validation |
| 14 | Modules, Imports & Path Aliases | 3 | ESM imports, tsconfig paths, monorepo setup, package.json exports |
| 15 | Declaration Files & Ambient Types | 3 | .d.ts authoring, module augmentation, declare global, extending libraries |
| 16 | Build Tooling & Compilation | 3 | tsc, tsup, esbuild, source maps, watch mode |
| 17 | Linting, Testing & Quality | 4 | ESLint + typescript-eslint, Prettier, Vitest, type-level testing (tsd) |
| 18 | Playwright + TypeScript | 4 | Page typing, fixtures, custom test extends, data-driven tests, helpers |
| 19 | Node.js APIs (fs, streams, Buffer) | 3 | fs/promises, path, URL, fetch, streams |
| 20 | Small Projects: CLI & HTTP Service | 6 | Build a typed CLI (commander), build a typed HTTP service (Fastify) |
| | **Total** | **60** | |

## 3. Topics — Explanations with Examples

### 1. Setup & Project Structure

**Concept:**
TypeScript is compiled to JavaScript before execution. You need a working Node environment, TypeScript compiler (`tsc`), and a `tsconfig.json` to configure compilation behavior. For rapid development iteration, use `tsx` or `ts-node` to run TS files directly.

**Example — tsconfig.json for Node 20 + strict mode:**
```ts
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictPropertyInitialization": true,
    "noUncheckedIndexedAccess": true,
    "allowUnusedLabels": false,
    "allowUnreachableCode": false,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@tests/*": ["tests/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

**Example — Running TypeScript directly:**
```ts
// src/main.ts
const greet = (name: string): string => `Hello, ${name}!`;
console.log(greet("Govind"));
```

Run with `npx tsx src/main.ts` or `npx ts-node src/main.ts` (no compilation step needed for development).

**Key pitfalls:**
- Forgetting `"strict": true` — leaves the type system full of holes.
- Using `"module": "CommonJS"` when targeting modern Node/bundlers — stick to ESNext and let your bundler downcompile if needed.
- Not setting `"moduleResolution": "bundler"` — can cause import resolution differences between tsc and bundlers.

---

### 2. Primitives & Basic Types

**Concept:**
TypeScript extends JavaScript's runtime types with compile-time type annotations. Every value has a type: `string`, `number`, `boolean`, `null`, `undefined`, and the "escape hatches" `any` and `unknown`.

**Example:**
```ts
// Primitives
const name: string = "Govind";
const age: number = 28;
const isTestAutomation: boolean = true;
const nothing: null = null;
const notYetDefined: undefined = undefined;

// Unions of literals (more on this later)
const status: "pending" | "complete" | "failed" = "pending";

// any — AVOID unless integrating with untyped code
let data: any = JSON.parse('{"foo": 1}');
data.bar.baz(); // No type error, but will crash at runtime

// unknown — safer than any, forces explicit narrowing
let input: unknown = JSON.parse('{"foo": 1}');
// input.foo; // ERROR: Object is of type 'unknown'
if (typeof input === "object" && input !== null && "foo" in input) {
  console.log((input as { foo: number }).foo);
}

// never — the return type of functions that never complete
const throwError = (msg: string): never => {
  throw new Error(msg);
};

// void — function returns undefined (not useful as a variable type)
const logIt = (): void => {
  console.log("hello");
};
```

**Key pitfalls:**
- `any` silences all type checking — use `unknown` and narrow explicitly instead.
- Forgetting that `null` and `undefined` are distinct types. With `strictNullChecks: true`, you must handle both.
- Using `void` to mean "no return value" when you should use `undefined` for a value that is undefined.

---

### 3. Collections: Arrays, Tuples, Enums

**Concept:**
Arrays hold multiple values of the same type. Tuples are fixed-length arrays with specific types at each position. Enums map names to values; use `as const` for safer literal unions instead.

**Example:**
```ts
// Arrays
const numbers: number[] = [1, 2, 3];
const strings: Array<string> = ["a", "b"];
const mixed: (string | number)[] = [1, "two", 3];

// Tuples — fixed length, specific type at each position
const tuple: [string, number] = ["Govind", 28];
const tuple2: [string, number, boolean?] = ["test"]; // optional third element
const tuple3: [string, ...number[]] = ["ages", 25, 30, 35]; // rest element

// Enums — legacy, generally avoid; prefer const assertion instead
enum Status {
  Pending = "PENDING",
  Complete = "COMPLETE",
  Failed = "FAILED"
}
const s: Status = Status.Pending;

// Const assertion — safer and more tree-shake friendly
const StatusConst = {
  Pending: "PENDING",
  Complete: "COMPLETE",
  Failed: "FAILED"
} as const;
type StatusType = typeof StatusConst[keyof typeof StatusConst]; // "PENDING" | "COMPLETE" | "FAILED"

// Const assertion on array
const colors = ["red", "green", "blue"] as const;
type Color = typeof colors[number]; // "red" | "green" | "blue"

// Literal types
const literal: "only this exact string" = "only this exact string";
// literal = "anything else"; // ERROR
```

**Key pitfalls:**
- Enums are verbose and not tree-shakeable. Prefer `as const` + type extraction.
- Forgetting that tuple types are structural, not nominal — `[string, number]` and `[string, number]` are identical types.
- Assuming `array[index]` is always defined — use `noUncheckedIndexedAccess` to catch undefined array access.

---

### 4. Objects: Interfaces & Type Aliases

**Concept:**
Interfaces and type aliases both describe object shapes. Interfaces can extend other interfaces and be merged in the same file; type aliases are more flexible but don't auto-merge. For most cases, use types; use interfaces for class contracts and library augmentation.

**Example:**
```ts
// Interface — good for class contracts and public APIs
interface User {
  id: number;
  name: string;
  email?: string; // optional property
  readonly createdAt: Date; // immutable
}

// Type alias — good for unions, intersections, primitives
type UserOrAdmin = User | { role: "admin" };
type ID = string | number;
type Callback = (user: User) => void;

// Interface extending interface
interface Admin extends User {
  permissions: string[];
}

// Type extending type
type ExtendedUser = User & { preferences: Record<string, string> };

// Interface merging (same name in same scope auto-merges)
interface Config {
  debug: boolean;
}
interface Config {
  verbose: boolean; // merged into Config
}
// Config is now { debug: boolean; verbose: boolean }

// Index signature — allow arbitrary keys of a type
interface Record<K extends string | number | symbol, V> {
  [key: K]: V;
}
const settings: Record<string, string> = {
  theme: "dark",
  language: "en"
};

// Readonly properties
interface ImmutableUser {
  readonly id: number;
  readonly name: string;
}
// user.id = 99; // ERROR

// Getter/setter in interface
interface Mutable {
  get value(): number;
  set value(v: number);
}
```

**Key pitfalls:**
- Over-using interfaces when types would suffice. Types are more flexible (unions, primitives, functions).
- Forgetting that `readonly` is compile-time only; it won't prevent mutation at runtime.
- Using index signatures too liberally — they hide missing keys.

---

### 5. Union & Intersection Types

**Concept:**
Union types (`A | B`) mean "either A or B". Intersection types (`A & B`) mean "both A and B". Discriminated unions (discriminator field) allow exhaustive type narrowing and are central to many TS patterns.

**Example:**
```ts
// Union types
type Result = "success" | "error";
type Value = string | number | boolean;

// Union of objects — common but ambiguous
type Cat = { meow(): void };
type Dog = { bark(): void };
type Animal = Cat | Dog;

const pet: Animal = { meow: () => console.log("meow") };
// pet.bark(); // ERROR: Property 'bark' does not exist on type 'Cat | Dog'
// We don't know if it has bark without narrowing

// Discriminated union — add a discriminator field
type SuccessResult = { status: "success"; data: unknown };
type ErrorResult = { status: "error"; error: Error };
type Response = SuccessResult | ErrorResult;

const handle = (res: Response) => {
  if (res.status === "success") {
    console.log(res.data); // type is unknown here
  } else {
    console.log(res.error); // type is Error here
  }
};

// Intersection types
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged; // must have both name and age

const person: Person = { name: "Govind", age: 28 };

// Intersection of functions
type LoggerFn = (msg: string) => void;
type ErrorLoggerFn = (msg: string, err: Error) => void;
type CombinedLogger = LoggerFn & ErrorLoggerFn;

// Exhaustive checking with never
type Shape = { kind: "circle"; radius: number } | { kind: "square"; side: number };

const area = (shape: Shape): number => {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    // If we forget a case, the compiler catches it:
    // default: const _: never = shape; // ERROR: Type ... is not assignable to never
  }
  return 0;
};
```

**Key pitfalls:**
- Bare unions without discriminators force you to check every property to narrow. Use discriminated unions instead.
- Intersection of object types with overlapping properties — if two types have a key with incompatible types, the intersection becomes `never`.
- Forgetting the exhaustive check (`default: const _: never = shape`) — easy to miss a case.

---

### 6. Functions: Types & Overloads

**Concept:**
Functions have input and output types. Use function overloads to declare multiple valid call signatures, then provide a single implementation.

**Example:**
```ts
// Simple function with types
const add = (a: number, b: number): number => a + b;

// Optional parameters and defaults
const greet = (name: string, greeting: string = "Hello"): string => {
  return `${greeting}, ${name}!`;
};

// Rest parameters
const sum = (...nums: number[]): number => {
  return nums.reduce((acc, n) => acc + n, 0);
};
console.log(sum(1, 2, 3, 4)); // 10

// Spread in function call
const twoNumbers = (a: number, b: number) => a + b;
const args: [number, number] = [5, 7];
twoNumbers(...args);

// Function types as values
type Transformer = (input: string) => string;
const toUpperCase: Transformer = (s) => s.toUpperCase();
const apply = (fn: Transformer, str: string): string => fn(str);

// Overloads — declare multiple signatures, then implement one
function formatDate(date: Date): string;
function formatDate(date: Date, format: "iso"): string;
function formatDate(date: Date, format: "iso" | "us"): string {
  if (format === "iso") {
    return date.toISOString();
  }
  return date.toLocaleDateString("en-US");
}

// Generic function (more on this in section 8)
function identity<T>(val: T): T {
  return val;
}
const num = identity(42); // num: number
const str = identity("hello"); // str: string

// Callback with optional context (this parameter)
interface Button {
  onClick(this: Button, event: Event): void;
}

// Nullable/optional parameters
const parseNumber = (str: string | null): number | null => {
  if (str === null) return null;
  const num = parseInt(str);
  return isNaN(num) ? null : num;
};

// Function that returns void (undefined) can return anything, but the return is ignored
const logAndReturn = (msg: string, value: number): void => {
  console.log(msg);
  return value; // "return value" is allowed but ignored
};
```

**Key pitfalls:**
- Function overloads require an implementation signature that covers all overloads — it often must be a union or very generic.
- Forgetting that optional parameters must come after required ones.
- Mixing `...rest` with positional parameters — rest must be last.

---

### 7. Classes & Object-Oriented Patterns

**Concept:**
Classes encapsulate state and behavior. TypeScript adds visibility modifiers (`public`, `private`, `protected`) and property declarations with types.

**Example:**
```ts
class User {
  // Public properties (default)
  public id: number;
  public name: string;

  // Private properties — not accessible from outside the class or subclasses
  private password: string;

  // Protected properties — accessible in this class and subclasses
  protected internal: string = "internal data";

  // Private field syntax (JS-native, not just TS)
  #secret: string = "secret";

  // Readonly properties
  readonly createdAt: Date;

  // Constructor with parameter properties (shorthand)
  constructor(id: number, name: string, password: string) {
    this.id = id;
    this.name = name;
    this.password = password;
    this.createdAt = new Date();
  }

  // Method with type signature
  authenticate(password: string): boolean {
    return this.password === password;
  }

  // Static methods and properties
  static defaultRole = "user";
  static create(name: string): User {
    return new User(Math.random(), name, "");
  }

  // Getter and setter
  get displayName(): string {
    return this.name.toUpperCase();
  }

  set displayName(value: string) {
    this.name = value;
  }
}

// Abstract class — cannot instantiate, defines interface for subclasses
abstract class BaseHandler {
  abstract handle(data: unknown): void;

  protected log(msg: string): void {
    console.log(`[${this.constructor.name}] ${msg}`);
  }
}

class ConcreteHandler extends BaseHandler {
  handle(data: unknown): void {
    this.log("Handling data");
  }
}

// Implement multiple interfaces
interface Loggable {
  log(): void;
}

interface Serializable {
  toJSON(): string;
}

class LoggableUser implements Loggable, Serializable {
  constructor(private name: string) {}

  log(): void {
    console.log(this.name);
  }

  toJSON(): string {
    return JSON.stringify({ name: this.name });
  }
}

// Constructor shorthand with visibility modifiers
class Point {
  constructor(private x: number, public y: number) {}
  // Equivalent to:
  // private x: number;
  // public y: number;
  // constructor(x: number, y: number) { this.x = x; this.y = y; }
}

// Generic class
class Box<T> {
  constructor(private value: T) {}

  getValue(): T {
    return this.value;
  }

  setValue(value: T): void {
    this.value = value;
  }
}

const box = new Box<string>("hello");
box.getValue(); // string
```

**Key pitfalls:**
- `private` and `protected` are compile-time only; at runtime, properties are still accessible. Use `#private` fields for true privacy.
- Forgetting to initialize properties in the constructor — TypeScript will error if `strictPropertyInitialization: true`.
- Over-using inheritance when composition would be simpler.

---

### 8. Generics 101

**Concept:**
Generics let you write functions and types that work with any type while maintaining type safety. Think of them as "type parameters" that are filled in at call-time.

**Example:**
```ts
// Generic function
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const nums = first([1, 2, 3]); // nums: number | undefined
const strs = first(["a", "b"]); // strs: string | undefined

// Generic constraint — T must extend certain shape
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Govind", age: 28 };
const name = getProperty(user, "name"); // name: string
// getProperty(user, "invalid"); // ERROR

// Generic with default type
function wrap<T = string>(value: T): { value: T } {
  return { value };
}

wrap("hello"); // { value: "hello" }
wrap(42); // { value: 42 }
wrap(); // { value: string }

// Generic class
class Cache<K, V> {
  private store = new Map<K, V>();

  set(key: K, value: V): void {
    this.store.set(key, value);
  }

  get(key: K): V | undefined {
    return this.store.get(key);
  }
}

const cache = new Cache<string, number>();
cache.set("age", 28);
const age = cache.get("age"); // age: number | undefined

// keyof — extract keys of a type
interface Config {
  host: string;
  port: number;
}

type ConfigKey = keyof Config; // "host" | "port"

// Conditional generic (more on this in section 11)
type IsString<T> = T extends string ? true : false;
type A = IsString<"hello">; // true
type B = IsString<number>; // false
```

**Key pitfalls:**
- Generic constraints that are too loose (`T extends any`), defeating the purpose.
- Forgetting that generic parameters must be used somewhere in the signature, or TypeScript can't infer them.
- Over-parameterizing — if you need 5+ type parameters, consider refactoring.

---

### 9. Utility Types & Type Manipulation

**Concept:**
TypeScript provides built-in utility types to transform and extract from existing types. Use them to reduce boilerplate.

**Example:**
```ts
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

// Partial<T> — all properties optional
type PartialUser = Partial<User>;
const update: PartialUser = { name: "New Name" }; // OK, other fields optional

// Required<T> — all properties required
type RequiredUser = Required<User>;
// Opposite of Partial

// Readonly<T> — all properties readonly
type ReadonlyUser = Readonly<User>;
// user.name = "new"; // ERROR

// Pick<T, K> — select specific properties
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string }

// Omit<T, K> — exclude specific properties
type UserWithoutEmail = Omit<User, "email" | "createdAt">;
// { id: number; name: string }

// Record<K, V> — object with specific keys and value type
type Permissions = Record<"read" | "write" | "delete", boolean>;
const perms: Permissions = { read: true, write: false, delete: false };

// Exclude<T, U> — exclude types from union
type Primitive = string | number | boolean | null;
type NonNull = Exclude<Primitive, null>; // string | number | boolean

// Extract<T, U> — extract matching types from union
type OnlyString = Extract<Primitive, string>; // string

// NonNullable<T> — remove null and undefined
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>; // string

// ReturnType<T> — extract return type of function
type MyFunc = (x: number) => string;
type RetVal = ReturnType<MyFunc>; // string

// Parameters<T> — extract parameter types as tuple
type Params = Parameters<MyFunc>; // [x: number]

// InstanceType<T> — extract instance type from constructor
class MyClass {
  prop: string = "test";
}
type Instance = InstanceType<typeof MyClass>; // MyClass

// Awaited<T> — unwrap Promise
type PromiseString = Promise<string>;
type ResolvedString = Awaited<PromiseString>; // string

// Uppercase/Lowercase (literal types)
type Screaming = Uppercase<"hello">; // "HELLO"
type Quiet = Lowercase<"HELLO">; // "hello"
```

**Key pitfalls:**
- `Pick` and `Omit` with large sets — consider creating smaller types instead.
- Forgetting that `Record<string, any>` is very loose; prefer specific keys.
- Using `NonNullable` when you should use proper null checks instead.

---

### 10. Type Narrowing & Guards

**Concept:**
Type narrowing is the process of refining a variable's type from a broader type (like a union) to a narrower one. TypeScript's control flow analysis tracks this automatically in most cases.

**Example:**
```ts
// typeof narrowing
const process = (value: string | number) => {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // value: string
  } else {
    console.log(value.toFixed(2)); // value: number
  }
};

// instanceof narrowing
class Error1 extends Error {}
class Error2 extends Error {}

const handle = (err: Error1 | Error2) => {
  if (err instanceof Error1) {
    // err: Error1
  } else {
    // err: Error2
  }
};

// "in" operator narrowing
type Admin = { role: "admin"; permissions: string[] };
type User = { role: "user" };

const check = (person: Admin | User) => {
  if ("permissions" in person) {
    console.log(person.permissions); // person: Admin
  }
};

// Custom type guard — function returns type predicate
function isString(value: unknown): value is string {
  return typeof value === "string";
}

const items: unknown[] = ["a", 1, "b"];
const strings = items.filter(isString); // strings: string[]

// Type predicate with generics
function hasProperty<T, K extends PropertyKey>(obj: unknown, key: K): obj is T & Record<K, any> {
  return typeof obj === "object" && obj !== null && key in obj;
}

// Assertion function — asserts condition, narrows type
function assertString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new TypeError("Expected string");
  }
}

const data: unknown = "hello";
assertString(data);
console.log(data.toUpperCase()); // data: string

// Exhaustiveness checking
type Result = { kind: "success"; value: number } | { kind: "error"; error: Error };

const process2 = (result: Result) => {
  switch (result.kind) {
    case "success":
      return result.value;
    case "error":
      throw result.error;
    default:
      // If a new case is added to Result union, this will error:
      const exhaustive: never = result;
      return exhaustive;
  }
};

// Logical operators preserve narrowing
const validate = (str: string | null) => {
  if (str && str.length > 0) {
    // str: string (non-null, truthy, so length > 0)
  }
};
```

**Key pitfalls:**
- Custom type guards with incorrect logic — the guard must actually check what it claims.
- Forgetting that `null` is an object in JavaScript, so `typeof null === "object"`.
- Assertion functions that don't actually throw when the assertion fails.

---

### 11. Advanced Types & Mapped Types

**Concept:**
Mapped types and conditional types enable meta-programming: writing types that derive from other types. These are powerful but easy to misuse.

**Example:**
```ts
// Mapped types — iterate over keys and transform
interface User {
  name: string;
  age: number;
  email: string;
}

// Create a type where all properties are getters
type Getters<T> = {
  [K in keyof T]: () => T[K];
};

type UserGetters = Getters<User>;
// {
//   name: () => string;
//   age: () => number;
//   email: () => string;
// }

// Readonly version of all properties
type ReadonlyVersion<T> = {
  readonly [K in keyof T]: T[K];
};

// Nullable version
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

// Conditional types — choose type based on condition
type IsString<T> = T extends string ? true : false;
type A = IsString<"hello">; // true
type B = IsString<number>; // false

// Conditional with unions — distributes over union
type Flatten<T> = T extends Array<infer U> ? U : T;
type NumOrNum = Flatten<number[]>; // number
type StrOrStr = Flatten<string>; // string

// infer keyword — capture type variable in conditional
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;
type MyFuncReturn = ReturnTypeOf<(x: number) => string>; // string

// Template literal types
type Size = "small" | "medium" | "large";
type Color = "red" | "blue";
type ClassName = `${Size}-${Color}`;
// "small-red" | "small-blue" | "medium-red" | ... (all 6 combinations)

// Recursive mapped type — DeepPartial
type DeepPartial<T> = T extends object ? {
  [K in keyof T]?: DeepPartial<T[K]>;
} : T;

type PartialUser = DeepPartial<{
  name: string;
  address: { street: string; city: string };
}>;
// {
//   name?: string;
//   address?: {
//     street?: string;
//     city?: string;
//   };
// }

// Extracting paths — type-safe object navigation
type PathsOf<T, Prefix extends string = ""> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? `${Prefix}${K}` | PathsOf<T[K], `${Prefix}${K}.`>
        : never;
    }[keyof T]
  : never;

type UserPaths = PathsOf<{
  name: string;
  address: { street: string; city: string };
}>;
// "name" | "address" | "address.street" | "address.city"
```

**Key pitfalls:**
- Mapped types with `keyof T` can become unwieldy if `T` is very large.
- Conditional types that distribute incorrectly over unions. Wrap in `[]` to disable distribution: `[T] extends [string]`.
- Using `infer` in a position where it can't capture (e.g., in a union type).

---

### 12. Async/Await & Promise Typing

**Concept:**
Promises represent async work. TypeScript tracks what a Promise resolves to via `Promise<T>`. Async functions are syntactic sugar for returning Promises.

**Example:**
```ts
// Promise basics
const fetchUser = (id: number): Promise<{ id: number; name: string }> => {
  return fetch(`/api/user/${id}`).then((res) => res.json());
};

// Async function — returns Promise implicitly
async function getUser(id: number): Promise<{ id: number; name: string }> {
  const res = await fetch(`/api/user/${id}`);
  return res.json();
}

// Type inference in async
const getConfig = async () => {
  // return type is Promise<{ debug: boolean }>
  return { debug: true };
};

// Promise.all — wait for multiple promises
const [user, config, permissions] = await Promise.all([
  fetchUser(1),
  fetch("/config").then((r) => r.json()),
  fetch("/perms").then((r) => r.json())
]);
// TypeScript infers tuple type correctly

// Promise.allSettled — all promises settle (success or fail)
const results = await Promise.allSettled([
  fetchUser(1),
  fetchUser(999) // may fail
]);

results.forEach((result) => {
  if (result.status === "fulfilled") {
    console.log(result.value); // success
  } else {
    console.log(result.reason); // error
  }
});

// Promise.race — first to settle
const timeout = new Promise<never>((_, reject) => {
  setTimeout(() => reject(new Error("timeout")), 5000);
});
const user2 = await Promise.race([fetchUser(1), timeout]);

// Async iterator
async function* generate(): AsyncGenerator<number> {
  let i = 0;
  while (i < 3) {
    yield i++;
    await new Promise((r) => setTimeout(r, 100));
  }
}

for await (const value of generate()) {
  console.log(value); // 0, 1, 2
}

// AbortController for cancellation
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000);

try {
  const res = await fetch("/api/data", { signal: controller.signal });
  const data = await res.json();
} catch (err) {
  if ((err as Error).name === "AbortError") {
    console.log("Request aborted");
  }
}

// Error handling in async
async function safeGetUser(id: number): Promise<{ id: number; name: string } | null> {
  try {
    return await fetchUser(id);
  } catch (err) {
    console.error("Failed to fetch user", err);
    return null;
  }
}

// Typing async function as value
type AsyncFetcher = () => Promise<unknown>;
const call: AsyncFetcher = async () => {
  return { data: "test" };
};

// Awaited utility — unwrap Promise/Promise chains
type PromiseChain = Promise<Promise<string>>;
type Unwrapped = Awaited<PromiseChain>; // string
```

**Key pitfalls:**
- Forgetting to `await` an async function — returns a Promise, not the value.
- `Promise.all` rejects if any promise rejects. Use `Promise.allSettled` if you need to handle partial failures.
- `async/await` can make errors harder to trace — add stack traces where possible.

---

### 13. Error Handling & Validation

**Concept:**
TypeScript doesn't enforce error handling at compile time (unlike checked exceptions). Use type-safe patterns: custom Error classes, Result types (success/error unions), or Zod for validation.

**Example:**
```ts
// Custom Error classes
class ValidationError extends Error {
  constructor(public field: string, public value: unknown) {
    super(`Validation failed for ${field}`);
    this.name = "ValidationError";
  }
}

class NotFoundError extends Error {
  constructor(public resource: string, public id: unknown) {
    super(`${resource} not found: ${id}`);
    this.name = "NotFoundError";
  }
}

// Type-safe error with discriminator
type AppError =
  | { type: "validation"; field: string; message: string }
  | { type: "notfound"; resource: string; id: unknown }
  | { type: "server"; code: number; message: string };

// Result type — Either success or error
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

const createUser = async (name: string): Promise<Result<{ id: number; name: string }>> => {
  if (!name || name.length === 0) {
    return { ok: false, error: new ValidationError("name", name) };
  }
  // simulate API call
  return { ok: true, value: { id: 1, name } };
};

// Usage
const result = await createUser("Govind");
if (result.ok) {
  console.log(result.value.name); // value: { id: number; name: string }
} else {
  console.log(result.error.field); // error: ValidationError
}

// Zod — runtime validation with type inference
import { z } from "zod";

const UserSchema = z.object({
  id: z.number(),
  name: z.string().min(1),
  email: z.string().email().optional(),
  age: z.number().int().min(0).max(150)
});

type User = z.infer<typeof UserSchema>; // { id: number; name: string; email?: string; age: number }

const parse = (data: unknown): Result<User> => {
  try {
    const parsed = UserSchema.parse(data);
    return { ok: true, value: parsed };
  } catch (err) {
    return { ok: false, error: err as z.ZodError };
  }
};

// Discriminated error result
type ParseResult =
  | { status: "success"; data: User }
  | { status: "error"; errors: z.ZodError["errors"] };

const validate = (data: unknown): ParseResult => {
  try {
    return { status: "success", data: UserSchema.parse(data) };
  } catch (err) {
    if (err instanceof z.ZodError) {
      return { status: "error", errors: err.errors };
    }
    throw err;
  }
};

// Never-throw async with implicit error handling
type AsyncResult<T> = Promise<T | Error>;
const fetch2 = async (): AsyncResult<string> => {
  try {
    return "success";
  } catch (err) {
    return err instanceof Error ? err : new Error(String(err));
  }
};

const val = await fetch2();
if (val instanceof Error) {
  // Handle error
} else {
  // val: string
}
```

**Key pitfalls:**
- Throwing anything (strings, objects) instead of Error instances — always `throw new Error(msg)`.
- Zod schema changes without updating types — use `z.infer<>` to keep them in sync.
- Forgetting to check `result.ok` before accessing `.value`.

---

### 14. Modules, Imports & Path Aliases

**Concept:**
TypeScript supports ES modules. Use path aliases (via `tsconfig.json` `paths`) to avoid `../../../` imports and improve refactoring.

**Example:**
```ts
// tsconfig.json paths
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@types/*": ["src/types/*"],
      "@tests/*": ["tests/*"],
      "@/lib/*": ["src/lib/*"]
    }
  }
}

// src/lib/utils.ts
export const sleep = (ms: number): Promise<void> => {
  return new Promise((resolve) => setTimeout(resolve, ms));
};

// src/types/user.ts
export interface User {
  id: number;
  name: string;
}

// src/main.ts — use @ aliases
import { User } from "@types/user";
import { sleep } from "@/lib/utils";

// Absolute exports from package
// package.json
{
  "exports": {
    ".": "./dist/index.js",
    "./types": "./dist/types/index.d.ts"
  }
}

// Re-export from index
// src/index.ts
export * from "./types/user";
export { sleep } from "./lib/utils";

// Consumer can do:
// import type { User } from "my-package/types";
// import { sleep } from "my-package";

// ESM and CJS side-by-side (with proper configuration)
// src/index.mjs (ESM)
export const greet = (name) => `Hello, ${name}`;

// src/index.cjs (CJS)
module.exports.greet = (name) => `Hello, ${name}`;

// package.json
{
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs"
    }
  }
}

// Monorepo with pnpm workspaces
// pnpm-workspace.yaml
packages:
  - "packages/*"

// packages/shared-types/package.json
{
  "name": "@myorg/shared-types",
  "exports": {
    ".": "./dist/index.d.ts"
  }
}

// packages/api/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "paths": {
      "@myorg/shared-types": ["../shared-types/src/index.ts"]
    }
  }
}
```

**Key pitfalls:**
- Path aliases that are too generic (`@/*` for everything) defeat organization.
- Circular imports — check with `madge` or similar.
- Forgetting to update `tsconfig.json` path aliases when moving files.

---

### 15. Declaration Files & Ambient Types

**Concept:**
`.d.ts` files declare types for JavaScript code (or type-only exports). Use them to type third-party untyped libraries or define global augmentations.

**Example:**
```ts
// src/utils.ts
export function add(a: number, b: number): number;
export function multiply(a: number, b: number): number;
export const PI: number;

// src/utils.d.ts (alternative to inline types)
export function add(a: number, b: number): number;
export function multiply(a: number, b: number): number;
export declare const PI: number;

// Global augmentation
// types/global.d.ts
declare global {
  interface Window {
    myCustomAPI: { getValue(): string };
  }

  namespace NodeJS {
    interface ProcessEnv {
      DATABASE_URL: string;
      API_KEY: string;
    }
  }
}

export {}; // Make this a module

// Usage in code
console.log(process.env.DATABASE_URL); // no error

// Module augmentation — extend third-party module
// types/express-custom.d.ts
declare global {
  namespace Express {
    interface Request {
      user?: { id: number; role: string };
    }
  }
}

// Usage
app.get("/", (req, res) => {
  const userId = req.user?.id; // req.user: { id: number; role: string } | undefined
});

// Declare ambient modules (for untyped libraries)
declare module "untyped-library" {
  export function doSomething(input: string): void;
  export interface Config {
    debug: boolean;
  }
}

// Usage
import { doSomething, Config } from "untyped-library";
const cfg: Config = { debug: true };
```

**Key pitfalls:**
- Forgetting `export {}` in global augmentation — it won't be recognized.
- Augmenting too many global types — centralizes them in a `types/global.d.ts`.
- Untyped libraries that have broken DefinitelyTyped — write a `.d.ts` shim.

---

### 16. Build Tooling & Compilation

**Concept:**
`tsc` compiles TypeScript to JavaScript. For performance and modern output, use `tsup` (esbuild-based) or `swc`. Always generate source maps for debugging.

**Example:**
```ts
// Package.json scripts
{
  "scripts": {
    "build": "tsc",
    "build:fast": "tsup src/index.ts --format esm,cjs --sourcemap",
    "dev": "tsx watch src/index.ts",
    "type-check": "tsc --noEmit"
  }
}

// tsconfig.json for production
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "moduleResolution": "bundler"
  },
  "include": ["src"],
  "exclude": ["node_modules", "tests", "dist"]
}

// tsup.config.ts — bundler config
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm", "cjs"],
  dts: true,
  sourcemap: true,
  clean: true,
  splitting: false,
  shims: true,
  external: ["react", "react-dom"]
});

// esbuild.mjs — lower-level bundling
import * as esbuild from "esbuild";

await esbuild.build({
  entryPoints: ["src/index.ts"],
  bundle: true,
  outfile: "dist/index.js",
  format: "esm",
  sourcemap: true,
  target: "ES2020"
});

// Incremental compilation (watch mode)
tsc --watch

// Type-only build (no runtime compilation)
tsc --noEmit
```

**Key pitfalls:**
- Forgetting source maps in production — makes debugging impossible.
- Using `tsc` for bundling — it doesn't bundle; use a bundler.
- `outDir` + file organization that creates a mess — keep src structure simple.

---

### 17. Linting, Testing & Quality

**Concept:**
Lint with `eslint` + `typescript-eslint` for type-aware rules. Format with Prettier. Test with Vitest (faster than Jest for modern TS). Use `tsd` for type-level tests.

**Example:**
```ts
// .eslintrc.json
{
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 2022,
    "sourceType": "module",
    "project": "./tsconfig.json"
  },
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-types": "off",
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/no-misused-promises": "error"
  }
}

// .prettierrc
{
  "semi": true,
  "singleQuote": false,
  "printWidth": 100,
  "trailingComma": "es5"
}

// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true,
    environment: "node",
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"]
    }
  }
});

// src/math.ts
export const add = (a: number, b: number): number => a + b;

// src/math.test.ts
import { describe, it, expect } from "vitest";
import { add } from "./math";

describe("math", () => {
  it("adds two numbers", () => {
    expect(add(1, 2)).toBe(3);
  });
});

// Type-level testing with tsd
// src/utils.d.ts
export function getData(id: string): Promise<{ id: string; name: string }>;
export function getData(id: number): Promise<{ id: number; name: string }>;

// src/utils.test-d.ts
import { expectType } from "tsd";
import { getData } from "./utils";

const result1: Promise<{ id: string; name: string }> = getData("1");
expectType<Promise<{ id: string; name: string }>>(result1);

const result2: Promise<{ id: number; name: string }> = getData(1);
expectType<Promise<{ id: number; name: string }>>(result2);
```

**Key pitfalls:**
- `@typescript-eslint/no-floating-promises` — catch unhandled async.
- Not running `tsc --noEmit` in CI — types won't be checked before tests run.
- Test snapshots with complex types — brittle; test behavior, not types.

---

### 18. Playwright + TypeScript

**Concept:**
Playwright provides strong typing out of the box. Use custom fixtures with generics and type-safe test data to build scalable test suites.

**Example:**
```ts
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",
  use: { baseURL: "http://localhost:3000" },
  projects: [
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox", use: { ...devices["Desktop Firefox"] } }
  ]
});

// tests/user.spec.ts
import { test, expect } from "@playwright/test";

test("login flow", async ({ page }) => {
  await page.goto("/login");
  await page.fill('input[name="email"]', "user@example.com");
  await page.fill('input[name="password"]', "pass123");
  await page.click('button[type="submit"]');

  // page is typed as Page, all methods are type-safe
  await expect(page).toHaveURL(/\/dashboard/);
});

// Custom fixture with type parameter
import { test as base } from "@playwright/test";

interface AuthenticatedContext {
  authenticatedPage: typeof page;
  user: { id: number; name: string };
}

export const test = base.extend<{ authenticatedPage: typeof base }>({
  authenticatedPage: async ({ page }, use) => {
    await page.goto("/login");
    await page.fill('input[name="email"]', "user@example.com");
    await page.fill('input[name="password"]', "pass123");
    await page.click('button[type="submit"]');
    await page.waitForURL(/\/dashboard/);

    await use(page);
  }
});

// Usage
test("access dashboard", async ({ authenticatedPage }) => {
  await expect(authenticatedPage.locator("h1")).toContainText("Dashboard");
});

// Typed test data
interface TestUser {
  email: string;
  password: string;
  expectedRole: "admin" | "user";
}

const testUsers: TestUser[] = [
  { email: "admin@test.com", password: "admin123", expectedRole: "admin" },
  { email: "user@test.com", password: "user123", expectedRole: "user" }
];

test.describe("permissions", () => {
  testUsers.forEach((user) => {
    test(`${user.expectedRole} can perform expected actions`, async ({ page }) => {
      await page.goto("/login");
      await page.fill('input[name="email"]', user.email);
      await page.fill('input[name="password"]', user.password);
      await page.click('button[type="submit"]');

      // type-safe: user.expectedRole is "admin" | "user"
      if (user.expectedRole === "admin") {
        await expect(page.locator('[data-testid="admin-panel"]')).toBeVisible();
      }
    });
  });
});

// Helper function with generic page locator typing
async function fillForm<T extends Record<string, string>>(
  page: typeof test.page,
  selectors: Record<keyof T, string>,
  data: T
): Promise<void> {
  for (const [key, selector] of Object.entries(selectors)) {
    await page.fill(selector, data[key as keyof T]);
  }
}

// Usage
const formSelectors = {
  email: 'input[name="email"]',
  password: 'input[name="password"]'
} as const;

await fillForm(page, formSelectors, { email: "test@example.com", password: "pass123" });
```

**Key pitfalls:**
- Not using `test.describe.parallel` for independent tests — missing performance gains.
- Hardcoding selectors instead of extracting to constants — brittle across refactors.
- Mixing page interactions with assertions — separate concerns for clarity.

---

### 19. Node.js APIs (fs, streams, Buffer)

**Concept:**
Node.js provides typed APIs for files, paths, URLs, and streams. Use `fs/promises` for async file operations and proper type imports.

**Example:**
```ts
// File system operations
import { readFile, writeFile, readdir, mkdir } from "fs/promises";
import { existsSync, statSync } from "fs";
import path from "path";

// Read file (async)
const content: string = await readFile("./data.txt", "utf-8");

// Write file
await writeFile("./output.txt", "Hello, World!");

// Check existence (sync, use sparingly)
if (existsSync("./file.txt")) {
  const stats = statSync("./file.txt");
  console.log(stats.size, stats.isDirectory());
}

// List directory
const files: string[] = await readdir("./src");

// Create directory recursively
await mkdir("./output", { recursive: true });

// Path manipulation
const joined = path.join("/home", "govind", "files");
const resolved = path.resolve("./relative/path");
const dirname = path.dirname("/home/govind/file.txt"); // /home/govind
const basename = path.basename("/home/govind/file.txt"); // file.txt
const ext = path.extname("file.txt"); // .txt

// URL handling (Node 18+)
const url = new URL("file:///home/govind/data.json");
const pathname = url.pathname; // /home/govind/data.json

// Read as file path
import { fileURLToPath } from "url";
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// Buffer operations
const buf = Buffer.from("Hello", "utf-8");
const str: string = buf.toString("utf-8");

const json = Buffer.from(JSON.stringify({ key: "value" }));
const parsed = JSON.parse(json.toString());

// Streams for large files
import { createReadStream, createWriteStream } from "fs";

const readStream = createReadStream("./large.txt", { encoding: "utf-8" });
const writeStream = createWriteStream("./output.txt");

readStream.on("data", (chunk: string) => {
  console.log(`Received: ${chunk.length} bytes`);
});

readStream.on("end", () => {
  console.log("Read complete");
});

readStream.on("error", (err: Error) => {
  console.error(err);
});

// Pipe for efficiency
readStream.pipe(writeStream);

// Fetch API (Node 18+, typed)
const response: Response = await fetch("https://api.example.com/data");
const data: unknown = await response.json();

// Type the response
interface ApiResponse {
  id: number;
  name: string;
}

const typed: ApiResponse = data as ApiResponse;
```

**Key pitfalls:**
- Using sync file operations in hot paths — stick to `fs/promises` async.
- Not handling stream errors — attach `error` listeners.
- Reading entire files into memory for large datasets — use streams.

---

### 20. Small Projects: CLI & HTTP Service

#### CLI Tool with Commander

**Example:**
```ts
// src/cli.ts
import { program } from "commander";
import { readFile, writeFile } from "fs/promises";

interface Config {
  inputFile: string;
  outputFile: string;
  format: "json" | "csv";
}

program
  .name("converter")
  .description("Convert data between formats")
  .version("1.0.0");

program
  .command("convert")
  .option("-i, --input <file>", "Input file path", "input.json")
  .option("-o, --output <file>", "Output file path", "output.csv")
  .option("-f, --format <format>", "Output format", "csv")
  .action(async (options: Partial<Config>) => {
    const config: Config = {
      inputFile: options.inputFile || "input.json",
      outputFile: options.outputFile || "output.csv",
      format: (options.format as "json" | "csv") || "csv"
    };

    try {
      const data = await readFile(config.inputFile, "utf-8");
      const transformed = transform(data, config.format);
      await writeFile(config.outputFile, transformed);
      console.log(`Converted to ${config.outputFile}`);
    } catch (err) {
      console.error("Conversion failed:", err);
      process.exit(1);
    }
  });

const transform = (data: string, format: "json" | "csv"): string => {
  if (format === "csv") {
    return `"id","name"\n"1","test"`;
  }
  return JSON.stringify({ data: [] });
};

program.parse(process.argv);
```

#### HTTP Service with Fastify

**Example:**
```ts
// src/server.ts
import Fastify from "fastify";
import { z } from "zod";

// Type-safe request/response
const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email()
});

type User = z.infer<typeof UserSchema>;

const CreateUserSchema = UserSchema.omit({ id: true });

const app = Fastify();

// GET endpoint
app.get<{ Reply: User }>("/:id", async (request, reply) => {
  const id = parseInt(request.params.id);
  if (isNaN(id)) {
    return reply.status(400).send({ error: "Invalid id" });
  }

  const user: User = { id, name: "Test User", email: "test@example.com" };
  return reply.send(user);
});

// POST endpoint with validation
app.post<{ Body: z.infer<typeof CreateUserSchema>; Reply: User }>(
  "/",
  async (request, reply) => {
    const parsed = CreateUserSchema.safeParse(request.body);
    if (!parsed.success) {
      return reply.status(400).send({ error: parsed.error.errors });
    }

    const user: User = { id: Math.random(), ...parsed.data };
    return reply.status(201).send(user);
  }
);

const start = async () => {
  try {
    await app.listen({ port: 3000 });
    console.log("Server running on http://localhost:3000");
  } catch (err) {
    console.error(err);
    process.exit(1);
  }
};

start();
```

---

## 4. Exercises

### Beginner

**Exercise 1: Strict TypeScript Bootstrap**
Create a new project with `strict: true` tsconfig. Write a module exporting:
- A typed function `fetchWithRetry(url: string, maxAttempts: number): Promise<Response>`
- A custom error class `NetworkError` extending Error with a `statusCode` property
- A helper type `Unwrap<T>` that extracts the value from `Promise<T>`

Ensure zero `any` types and full coverage of all branches with type narrowing.

**Exercise 2: Typed Fetch Wrapper**
Build a wrapper around `fetch()` that:
- Accepts generic type parameter for response body
- Validates response status and throws custom errors
- Supports retry logic with exponential backoff
- Returns `Promise<T>` where T is the parsed JSON

Use this to fetch real data from JSONPlaceholder (https://jsonplaceholder.typicode.com/users/1).

**Exercise 3: FizzBuzz with Types**
Implement FizzBuzz where:
- Input range is a tuple type `[start: number, end: number]`
- Output is a discriminated union: `{ type: "fizz" | "buzz" | "fizzbuzz" | "number"; value: number }`
- Use type narrowing in a consumer function to handle each case

---

### Intermediate

**Exercise 1: Typed In-Memory TODO API**
Build a small Fastify server with:
- Zod schemas for Todo input/output
- POST `/todos` to create (returns `{ id: number; title: string; done: boolean }`)
- GET `/todos/:id` to retrieve (narrowing validation)
- PATCH `/todos/:id` to update (Partial<Todo> with validation)
- DELETE `/todos/:id` to remove
- All responses typed and validated before sending

Test with curl or a simple test file.

**Exercise 2: Generic Retry/Backoff Helper**
Implement a function:
```ts
async function retry<T>(
  fn: () => Promise<T>,
  options: { maxAttempts: number; backoffMs: number; backoffMultiplier?: number }
): Promise<T>
```

Handle exponential backoff, logging, and custom error types. Use it to wrap a flaky API call.

**Exercise 3: Utility Types Deep Dive**
Implement these custom utility types:
- `DeepPartial<T>` — all properties and nested properties optional
- `DeepReadonly<T>` — all properties and nested properties readonly
- `PathsOf<T>` — extract all paths through an object as string literals (e.g., "user.address.street")

Test with a complex nested type like:
```ts
type Config = {
  database: { host: string; port: number; credentials: { user: string; password: string } };
  logging: { level: "debug" | "info" | "error"; format: "json" | "text" };
};
```

---

### Advanced

**Exercise 1: Typed Event Emitter**
Build a strongly-typed event emitter:
```ts
class EventEmitter<T extends Record<string, any>> {
  on<K extends keyof T>(event: K, listener: (data: T[K]) => void): void
  emit<K extends keyof T>(event: K, data: T[K]): void
}

type MyEvents = {
  "user:login": { userId: number };
  "user:logout": { userId: number };
  "error": Error;
};

const emitter = new EventEmitter<MyEvents>();
emitter.on("user:login", (data) => console.log(data.userId)); // data is typed correctly
emitter.emit("user:login", { userId: 1 });
```

**Exercise 2: Module Augmentation**
Extend Express Request with a typed `user` property:
```ts
declare global {
  namespace Express {
    interface Request {
      user?: { id: number; role: "admin" | "user" };
    }
  }
}
```

Build a middleware that validates JWT and sets `req.user`. Ensure downstream handlers access it with full type safety.

**Exercise 3: Migrate JS to Strict TS**
Take a small existing JavaScript project (or create a mock one with 200+ lines). Migrate to strict TypeScript:
- Add types to all functions
- Fix implicit `any` issues
- Handle null/undefined properly
- Extract types into a shared module
- Add a `tsconfig.json` with strict checks

Commit the before/after and explain the type improvements.

---

### Capstone: Typed API Contract Testing Framework

Build a framework that:

1. **Reads an OpenAPI 3.0 spec** (JSON file)
2. **Generates TypeScript types** from it (use `openapi-typescript` library)
3. **Defines test cases** in a strongly-typed format:
   ```ts
   const tests: ApiTest[] = [
     {
       name: "GET /users/:id should return user",
       method: "GET",
       path: "/users/1",
       expectedStatus: 200,
       expectedSchema: UserSchema // Zod schema or interface
     }
   ];
   ```
4. **Executes requests** against a real API (use a mock server like `prism` or local Node server)
5. **Validates responses** against schemas and assertions
6. **Produces a JUnit XML report** with pass/fail/skip results

**Deliverables:**
- `src/framework.ts` — core test runner with generics
- `src/types.ts` — shared types, generated from OpenAPI
- `tests/api.test.ts` — example test suite
- `report.xml` — JUnit output
- README explaining how to add new tests

**Bonus:**
- Support parametrized tests (multiple URLs, payloads)
- Custom assertion builders (e.g., `response.body.should.have.property("id", "number")`)
- Parallel test execution

---

## 5. Additional Reading & Resources

### Books
- **Effective TypeScript** by Dan Vanderkam — Covers 62 patterns; excellent for deepening understanding.
- **Programming TypeScript** by Boris Cherny — Comprehensive, covers types, patterns, and tooling.
- **Type-Driven Development with TypeScript** — Practical guide to using types to guide design.

### Official Documentation
- **TypeScript Handbook** (https://www.typescriptlang.org/docs/) — Official reference; sections on types, advanced types, module resolution.
- **TypeScript 5.x Release Notes** (https://www.typescriptlang.org/docs/handbook/release-notes/) — Latest features (const type parameters, decorators, etc.).

### Courses & Learning
- **Total TypeScript by Matt Pocock** (https://www.totaltypescript.com/) — Video-based, practical, excellent for interviews.
- **Matt Pocock's YouTube Channel** — Free advanced typing tutorials.
- **type-challenges** (https://github.com/type-challenges/type-challenges) — 500+ type puzzles, from easy to extreme.

### Tools & Libraries
- **Vitest** (https://vitest.dev/) — Modern test runner for TypeScript.
- **Playwright** (https://playwright.dev/) — E2E testing with strong TypeScript support.
- **Zod** (https://zod.dev/) — Runtime schema validation with type inference.
- **typescript-eslint** (https://typescript-eslint.io/) — Linting for TypeScript.
- **tsup** (https://tsup.egoist.dev/) — Simple bundler for TypeScript.
- **tsx** (https://github.com/esbuild-kit/tsx) — Run TypeScript without compilation step.

### Blogs & Articles
- **The TypeScript Handbook Blog** (https://devblogs.microsoft.com/typescript/) — Updates on new features.
- **Dev.to (TypeScript tag)** — Community articles on patterns and pitfalls.
- **Marius Schulz's Blog** — Deep dives into TypeScript types.

### YouTube
- **Matt Pocock** — Advanced typing, Playwright, TypeScript patterns.
- **Wes Bos** — Practical TypeScript for Node.js and web.
- **Jack Herrington** — TypeScript patterns and generics.

---

## 6. Index

| Topic | Section | Key Concepts |
|-------|---------|--------------|
| Assertion functions | 10 | `asserts x is Y` syntax, narrowing from assertions |
| Async iterators | 12 | `AsyncGenerator<T>`, `for await...of` |
| Bare unions | 5 | Union types without discriminators; ambiguous narrowing |
| Classes | 7 | Visibility, static, abstract, constructors, readonly |
| CLI tools | 20 | Commander, yargs, option parsing, typed actions |
| Conditional types | 11 | `T extends U ? A : B`, `infer`, distribution |
| Constructor shorthand | 7 | `constructor(public x: number)` syntax |
| Custom type guards | 10 | `is` predicate, return type typing |
| Declaration files | 15 | `.d.ts` files, ambient types, module augmentation |
| Decorators | — | Stage 3, `experimentalDecorators`, class/method enhancement |
| Discriminated unions | 5 | Discriminator field, exhaustive narrowing, `never` |
| Enums | 3 | Pitfalls, prefer `as const` |
| Error handling | 13 | Custom Error classes, Result types, Zod validation |
| ESM vs CJS | 1 | Module formats, dual exports, path aliases |
| Exclude/Extract | 9 | Union type filtering utilities |
| Function overloads | 6 | Multiple signatures, single implementation |
| Generics | 8 | Type parameters, constraints, defaults, `keyof` |
| Global augmentation | 15 | `declare global`, extending Window/NodeJS.ProcessEnv |
| Generators | — | `function*`, `yield`, iteration protocol |
| HTTP services | 20 | Fastify, Express, typed handlers, validation |
| `in` operator | 10 | Type guard for property existence |
| Index signatures | 4 | `[key: string]: value`, Record<K, V> |
| `instanceof` | 10 | Type narrowing for classes and constructors |
| Interfaces | 4 | Extends, merge behavior, class contracts |
| Intersection types | 5 | `A & B`, combined properties, merging |
| JSON parsing | 13 | Type-safe with Zod, parse safety |
| Literal types | 3 | String/number/boolean literals as types |
| Mapped types | 11 | Iterate keys, transform properties |
| Module paths | 14 | `baseUrl`, `paths`, tsconfig aliases, monorepos |
| `never` type | 2 | Functions that never return, exhaustive checks |
| Node.js APIs | 19 | fs/promises, path, URL, streams, Buffer |
| `null` vs `undefined` | 2 | Distinct types, `strictNullChecks` handling |
| Omit/Pick | 9 | Property selection utilities |
| Optional chaining | — | `obj?.prop`, safe navigation |
| Optional properties | 4 | `key?: type`, Partial<T> |
| Partial/Required | 9 | Make properties optional/required |
| Path aliases | 14 | `@/*` imports, refactoring-safe |
| Playwright | 18 | Custom fixtures, typing, test data, helpers |
| Promise<T> | 12 | Generic type, async/await, Promise utilities |
| Record | 9 | `Record<K, V>` for object literals with keys |
| Readonly | 4, 7 | Compile-time immutability, `readonly` modifier |
| Rest parameters | 6 | `...rest`, type as array |
| Result type | 13 | Discriminated union for success/error |
| ReturnType | 9 | Extract function return type |
| Strict mode | 1 | `strict: true`, enabling all strict checks |
| Tuples | 3 | Fixed-length arrays, `[T, U]`, optional elements |
| Type aliases | 4 | `type X = ...`, flexibility vs interfaces |
| Type narrowing | 10 | Refining broader to narrower types |
| typeof | 10 | Type guard, primitive checks |
| Unions | 5 | `A | B`, ambiguous without discriminators |
| `unknown` | 2 | Safer than `any`, requires narrowing |
| Utility types | 9 | Partial, Required, Pick, Omit, Record, etc. |
| Zod | 13 | Runtime validation, schema inference |

---

## 7. References

1. TypeScript Handbook — https://www.typescriptlang.org/docs/
2. TypeScript 5.0 Release Notes — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html
3. Effective TypeScript by Dan Vanderkam — https://effectivetypescript.com/
4. Programming TypeScript by Boris Cherny — https://www.oreilly.com/library/view/programming-typescript/9781492037644/
5. Total TypeScript by Matt Pocock — https://www.totaltypescript.com/
6. type-challenges Repository — https://github.com/type-challenges/type-challenges
7. Vitest Documentation — https://vitest.dev/
8. Playwright Documentation — https://playwright.dev/
9. Zod Documentation — https://zod.dev/
10. Fastify TypeScript Guide — https://www.fastify.io/docs/latest/Guides/TypeScript/
11. typescript-eslint Documentation — https://typescript-eslint.io/
12. tsup Documentation — https://tsup.egoist.dev/
13. tsx GitHub — https://github.com/esbuild-kit/tsx
14. OpenAPI TypeScript Generator — https://openapi-ts.dev/
15. Node.js fs/promises API — https://nodejs.org/docs/latest/api/fs.html#fs_promises_api
