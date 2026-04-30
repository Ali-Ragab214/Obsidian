# Part 1 — History & Background

---

## What is TypeScript and why was it created?

TypeScript is a **superset of JavaScript** — meaning it is JavaScript, but with extra features on top. The most important addition is **static typing**, which means you tell the code what type each variable should be, and TypeScript checks for errors **before** you even run the code.

```typescript
// TypeScript catches this error before running
let age: number = 25;
age = "hello"; // ❌ Error: Type 'string' is not assignable to type 'number'
```

In JavaScript, this error would only appear at **runtime** — meaning your app crashes in front of the user. TypeScript catches it at **compile time** — before the app even runs.

---

## Who created it and when?

TypeScript was created by **Anders Hejlsberg** at **Microsoft** and released publicly in **October 2012**. Anders is also the creator of C#, which is why TypeScript feels very similar to C# in many ways. Microsoft built it because they were struggling to maintain large-scale JavaScript applications internally.

---

## What problems does it solve that JavaScript could not?

JavaScript was originally built in **10 days** in 1995 for simple browser interactions — not for building massive applications like Gmail, Netflix, or enterprise systems. As apps grew bigger, JavaScript started showing serious problems:

```javascript
// JavaScript — no errors shown, but this is a bug
function getUser(id) {
  return id.toUpperCase(); // what if id is a number? App crashes at runtime
}

getUser(123); // crashes only when user runs it
```

```typescript
// TypeScript — catches the bug immediately
function getUser(id: string): string {
  return id.toUpperCase();
}

getUser(123); // ❌ Error at compile time — caught before shipping
```

The main problems TypeScript solves:

**1. No type safety in JavaScript** — you can pass anything to any function and JavaScript won't complain until it crashes.

**2. No autocompletion in large codebases** — without types, your editor doesn't know what properties an object has.

**3. Refactoring is dangerous in JavaScript** — rename a function and you won't know what broke until runtime. TypeScript tells you instantly.

**4. Team collaboration is hard** — when 10 developers work together, without types nobody knows what a function expects or returns.

---

## How does TypeScript relate to JavaScript under the hood?

TypeScript **compiles down to plain JavaScript**. The browser and Node.js do not understand TypeScript — they only understand JavaScript. So TypeScript goes through a step called **transpilation**:

```
Your TypeScript code (.ts)
        ↓
TypeScript Compiler (tsc)
        ↓
Plain JavaScript code (.js)
        ↓
Browser / Node.js runs it
```

This means TypeScript types **disappear at runtime** — they only exist to help you during development.

```typescript
// What you write in TypeScript
function add(a: number, b: number): number {
  return a + b;
}
```

```javascript
// What gets compiled to JavaScript
function add(a, b) {
  return a + b;
}
```

All the type information is stripped out. TypeScript is purely a **development tool** — a safety net while you write code.

---

## Where and why is TypeScript used in the real world?

TypeScript is used by some of the biggest companies and frameworks in the world:

**Frameworks built with TypeScript:**

- **Angular** — Google's frontend framework, written 100% in TypeScript
- **NestJS** — the Node.js backend framework you are learning, built entirely in TypeScript
- **Next.js** — fully supports TypeScript out of the box

**Companies using TypeScript:**

- Microsoft, Google, Airbnb, Slack, Asana, Stripe

**Why they chose TypeScript:**

- Large teams need type safety to avoid bugs
- Better IDE support means faster development
- Easier onboarding for new developers — the code is self-documenting
- Catches entire categories of bugs before production

---

**This is exactly why you need TypeScript before NestJS** — NestJS uses TypeScript decorators, interfaces, and types everywhere. Without understanding TypeScript deeply, NestJS will feel like magic you can't control. With TypeScript, every NestJS concept will make perfect sense.

---

Do you want to continue to Part 2 (TypeScript vs JavaScript) or do you need more examples on Part 1? 🚀

---
# Part 2 — TypeScript vs JavaScript

---

## The same code — side by side

```javascript
// ❌ JavaScript — no safety at all
function createUser(name, age, email) {
  return {
    name: name,
    age: age,
    email: email
  };
}

const user = createUser("Ali", "not a number", null);
// No error shown — bug goes to production
```

```typescript
// ✅ TypeScript — fully protected
function createUser(name: string, age: number, email: string): object {
  return {
    name: name,
    age: age,
    email: email
  };
}

const user = createUser("Ali", "not a number", null);
// ❌ Error caught immediately before running
```

---

## Key Differences

### 1. Static Typing vs Dynamic Typing

```javascript
// JavaScript — type can change anytime
let data = "hello";
data = 42;        // ✅ no error
data = true;      // ✅ no error
data = [];        // ✅ no error — JavaScript doesn't care
```

```typescript
// TypeScript — type is locked
let data: string = "hello";
data = 42;        // ❌ Error
data = true;      // ❌ Error
data = [];        // ❌ Error — TypeScript protects you
```

---

### 2. Function Parameters

```javascript
// JavaScript — call with anything
function multiply(a, b) {
  return a * b;
}

multiply(5, 3);        // 15 ✅
multiply("5", 3);      // "555" ❌ wrong but no error shown
multiply(5);           // NaN ❌ silent bug
```

```typescript
// TypeScript — enforces correct usage
function multiply(a: number, b: number): number {
  return a * b;
}

multiply(5, 3);        // 15 ✅
multiply("5", 3);      // ❌ Error caught immediately
multiply(5);           // ❌ Error — missing argument
```

---

### 3. Object Shape

```javascript
// JavaScript — no structure enforced
const product = {
  name: "Laptop",
  price: 999
};

console.log(product.category); // undefined — no error, silent bug
```

```typescript
// TypeScript — shape is defined and enforced
interface Product {
  name: string;
  price: number;
}

const product: Product = {
  name: "Laptop",
  price: 999
};

console.log(product.category); // ❌ Error — property doesn't exist
```

---

### 4. Autocompletion & IntelliSense

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

const user: User = { name: "Ali", age: 22, email: "ali@gmail.com" };

user. // your editor shows: name, age, email instantly
      // in JavaScript you get nothing
```

This alone saves hours of debugging every week.

---

### 5. Catching Errors Before Running

```javascript
// JavaScript — you find out when users complain
const users = null;
users.forEach(u => console.log(u)); // crashes at runtime
```

```typescript
// TypeScript — you find out while writing code
const users: string[] | null = null;
users.forEach(u => console.log(u)); // ❌ Error: users is possibly null
                                     // Fix it before shipping
```

---

## What TypeScript adds to JavaScript

|Feature|JavaScript|TypeScript|
|---|---|---|
|Static Types|❌|✅|
|Interfaces|❌|✅|
|Generics|❌|✅|
|Enums|❌|✅|
|Decorators|❌|✅|
|Access Modifiers|❌|✅|
|Compile-time errors|❌|✅|
|Autocompletion|weak|strong|

---

## Important rule to remember

> Every valid JavaScript file is also a valid TypeScript file.

This means you can take any `.js` file, rename it to `.ts`, and it will work. Then you start adding types gradually. This is why big companies can **migrate from JavaScript to TypeScript** slowly without rewriting everything.

```javascript
// this is valid in both JavaScript AND TypeScript
function greet(name) {
  return "Hello " + name;
}
```

---

## Why this matters for NestJS

NestJS is written in TypeScript and forces you to use it. Every controller, service, and module in NestJS uses:

- **Interfaces** to define data shapes
- **Decorators** like `@Controller()`, `@Get()`, `@Injectable()`
- **Types** everywhere for dependency injection

Without TypeScript, NestJS literally cannot work.

```typescript
// This is real NestJS code — see how TypeScript powers everything
@Controller('users')          // Decorator
export class UsersController {
  constructor(
    private usersService: UsersService  // Type used for injection
  ) {}

  @Get()                      // Decorator
  findAll(): User[] {         // Return type
    return this.usersService.findAll();
  }
}
```

Every single piece of this uses TypeScript features you will learn in the next parts.

---

Do you want to continue to **Part 3 — Core Concepts** or do you need more examples here? 🚀

---
# Part 3 — TypeScript Core Concepts (Zero to Hero)

---

## 1. Types

### string

```typescript
let firstName: string = "Ali";
let lastName: string = 'Ragab';
let fullName: string = `${firstName} ${lastName}`; // template literal

firstName = 42; // ❌ Error
```

---

### number

```typescript
let age: number = 22;
let price: number = 99.99;
let hex: number = 0xf00d;    // hexadecimal
let binary: number = 0b1010; // binary

age = "22"; // ❌ Error
```

---

### boolean

```typescript
let isLoggedIn: boolean = true;
let hasPermission: boolean = false;

isLoggedIn = 1; // ❌ Error — 1 is not boolean in TypeScript
```

---

### any

`any` turns off TypeScript completely for that variable. It is like going back to JavaScript. **Avoid it as much as possible.**

```typescript
let data: any = "hello";
data = 42;        // ✅ no error
data = true;      // ✅ no error
data = [];        // ✅ no error

// The danger
data.toUpperCase(); // no error shown — but if data is a number this crashes
```

**When is `any` acceptable?** Only when working with third-party libraries that have no types, or during quick migration from JavaScript.

---

### unknown

`unknown` is the **safe version of `any`**. You can assign anything to it, but you cannot use it until you check its type first.

```typescript
let input: unknown = "hello";

// ❌ Cannot use directly
input.toUpperCase(); // Error — TypeScript doesn't know the type yet

// ✅ Must check type first
if (typeof input === "string") {
  input.toUpperCase(); // ✅ Now TypeScript knows it's a string
}
```

**Rule:** Always prefer `unknown` over `any` when you don't know the type yet.

---

### void

Used for functions that **return nothing**.

```typescript
function logMessage(message: string): void {
  console.log(message);
  // no return statement needed
}

// In NestJS you will see this a lot
async function saveUser(user: User): Promise<void> {
  await database.save(user);
  // returns nothing
}
```

---

### never

Used for functions that **never finish** — they either throw an error or run forever.

```typescript
// Function that always throws — never returns
function throwError(message: string): never {
  throw new Error(message);
}

// Function that runs forever — never returns
function infiniteLoop(): never {
  while (true) {
    // runs forever
  }
}
```

**Where you see `never` in NestJS:**

```typescript
// When all cases are handled, remaining type is never
function handleStatus(status: "active" | "inactive") {
  if (status === "active") return "user is active";
  if (status === "inactive") return "user is inactive";

  const impossible: never = status; // TypeScript ensures all cases covered
}
```

---

### null and undefined

```typescript
let name: string = null;      // ❌ Error in strict mode
let age: number = undefined;  // ❌ Error in strict mode

// To allow null — use union type
let name: string | null = null;       // ✅
let age: number | undefined = undefined; // ✅

// Optional chaining — very common in NestJS
const user = null;
console.log(user?.name); // ✅ returns undefined instead of crashing
```

---

## 2. Type Inference

TypeScript is smart enough to **figure out the type automatically** without you writing it.

```typescript
// You don't need to write the type — TypeScript infers it
let name = "Ali";       // inferred as string
let age = 22;           // inferred as number
let active = true;      // inferred as boolean

name = 42;  // ❌ Error — TypeScript already knows name is string

// Inferred return type
function add(a: number, b: number) {
  return a + b; // TypeScript infers return type is number
}

const result = add(5, 3); // result is inferred as number
result.toUpperCase();     // ❌ Error — TypeScript knows result is number
```

**Rule:** For simple variables, let TypeScript infer the type. Only write types explicitly for function parameters and complex objects.

---

## 3. Type Aliases and Interfaces

Both are used to define the shape of an object. They look similar but have important differences.

### Type Alias

```typescript
type User = {
  name: string;
  age: number;
  email: string;
};

const user: User = {
  name: "Ali",
  age: 22,
  email: "ali@gmail.com"
};

// Type alias for primitives — interfaces cannot do this
type ID = string | number;
type Status = "active" | "inactive" | "banned";

let userId: ID = "abc123";
userId = 456; // ✅ both are valid
```

### Interface

```typescript
interface User {
  name: string;
  age: number;
  email: string;
  address?: string; // optional property
  readonly id: number; // cannot be changed after creation
}

const user: User = {
  name: "Ali",
  age: 22,
  email: "ali@gmail.com",
  id: 1
};

user.id = 2; // ❌ Error — readonly
user.address = "Cairo"; // ✅ optional but can be set
```

### Key Difference — Interfaces can be extended and merged

```typescript
// Extending interface
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

const dog: Dog = {
  name: "Rex",
  breed: "German Shepherd"
};

// Interface merging — unique to interfaces
interface User {
  name: string;
}

interface User {
  age: number; // TypeScript merges both declarations
}

const user: User = {
  name: "Ali",
  age: 22   // must have both now
};
```

**Rule for NestJS:** Use **interfaces** for object shapes and data transfer objects (DTOs). Use **type aliases** for unions and primitives.

---

## 4. Union Types and Intersection Types

### Union Types — value can be ONE of these types

```typescript
// Variable can be string OR number
let id: string | number;
id = "abc123"; // ✅
id = 456;      // ✅
id = true;     // ❌ Error

// Function accepting multiple types
function formatId(id: string | number): string {
  if (typeof id === "string") {
    return id.toUpperCase();
  }
  return id.toString();
}

// Real NestJS example
type ApiResponse = SuccessResponse | ErrorResponse;
```

### Intersection Types — value must have ALL of these types

```typescript
interface HasName {
  name: string;
}

interface HasAge {
  age: number;
}

// User must have BOTH name AND age
type User = HasName & HasAge;

const user: User = {
  name: "Ali",
  age: 22
  // missing either one = ❌ Error
};

// Real NestJS use case
type AdminUser = User & { permissions: string[] };
```

---

## 5. Enums

Enums let you define a set of named constants. Very common in NestJS for status, roles, and directions.

```typescript
// Numeric enum — default starts at 0
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right  // 3
}

console.log(Direction.Up);    // 0
console.log(Direction.Right); // 3

// String enum — preferred in real apps
enum UserRole {
  Admin = "ADMIN",
  User = "USER",
  Moderator = "MODERATOR"
}

function checkAccess(role: UserRole): void {
  if (role === UserRole.Admin) {
    console.log("Full access granted");
  }
}

checkAccess(UserRole.Admin); // ✅
checkAccess("ADMIN");        // ❌ Error — must use enum value

// NestJS real example
enum OrderStatus {
  Pending = "PENDING",
  Processing = "PROCESSING",
  Shipped = "SHIPPED",
  Delivered = "DELIVERED",
  Cancelled = "CANCELLED"
}
```

---

## 6. Tuples and Arrays

### Arrays

```typescript
// Array of strings
let names: string[] = ["Ali", "Ahmed", "Sara"];
let names2: Array<string> = ["Ali", "Ahmed", "Sara"]; // same thing

// Array of numbers
let scores: number[] = [95, 87, 92];

// Array of objects
let users: User[] = [
  { name: "Ali", age: 22, email: "ali@gmail.com", id: 1 }
];

names.push(42); // ❌ Error — only strings allowed
```

### Tuples — fixed length, fixed types at each position

```typescript
// Exactly 2 elements: first is string, second is number
let person: [string, number] = ["Ali", 22];

person[0] = "Ahmed"; // ✅
person[1] = 25;      // ✅
person[0] = 42;      // ❌ Error — first must be string
person[2] = "extra"; // ❌ Error — only 2 elements allowed

// Real use case — function returning multiple values
function getMinMax(numbers: number[]): [number, number] {
  return [Math.min(...numbers), Math.max(...numbers)];
}

const [min, max] = getMinMax([3, 1, 9, 2, 7]);
console.log(min); // 1
console.log(max); // 9
```

---

## 7. Functions

```typescript
// Basic typed function
function greet(name: string, age: number): string {
  return `Hello ${name}, you are ${age} years old`;
}

// Optional parameter — use ?
function greet(name: string, age?: number): string {
  if (age) {
    return `Hello ${name}, you are ${age} years old`;
  }
  return `Hello ${name}`;
}

greet("Ali");       // ✅
greet("Ali", 22);   // ✅

// Default parameter
function createUser(name: string, role: string = "user"): object {
  return { name, role };
}

createUser("Ali");           // { name: "Ali", role: "user" }
createUser("Ali", "admin");  // { name: "Ali", role: "admin" }

// Arrow function with types
const multiply = (a: number, b: number): number => a * b;

// Function type definition
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4, 5); // 15
```

---

## 8. Generics

Generics are the most important concept for NestJS. They let you write **reusable code that works with any type** while still being type-safe.

```typescript
// Without generics — must repeat for every type
function getFirstString(arr: string[]): string {
  return arr[0];
}
function getFirstNumber(arr: number[]): number {
  return arr[0];
}

// With generics — one function works for all types
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

getFirst<string>(["Ali", "Ahmed"]);  // returns string
getFirst<number>([1, 2, 3]);         // returns number
getFirst<boolean>([true, false]);    // returns boolean

// TypeScript can also infer the generic type
getFirst(["Ali", "Ahmed"]); // T is inferred as string

// Generic interface — used heavily in NestJS
interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

// Reuse for any data type
const userResponse: ApiResponse<User> = {
  data: { name: "Ali", age: 22, email: "ali@gmail.com", id: 1 },
  message: "User fetched successfully",
  success: true
};

const productsResponse: ApiResponse<Product[]> = {
  data: [{ name: "Laptop", price: 999 }],
  message: "Products fetched",
  success: true
};

// Generic with constraint — T must have certain properties
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Ali", age: 22 };
getProperty(user, "name");  // ✅ returns "Ali"
getProperty(user, "email"); // ❌ Error — email doesn't exist on user
```

**In NestJS you will see generics everywhere:**

```typescript
// Repository pattern in NestJS
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User> // Generic!
  ) {}

  findAll(): Promise<User[]> { // Generic!
    return this.usersRepository.find();
  }
}
```

---

## 9. Type Assertions and Type Guards

### Type Assertions — you tell TypeScript what type something is

```typescript
// Sometimes you know more than TypeScript
const input = document.getElementById("username");
// TypeScript thinks this is HTMLElement | null

// Assert it is specifically an HTMLInputElement
const inputElement = input as HTMLInputElement;
inputElement.value; // ✅ now you can access .value

// Non-null assertion — you guarantee it's not null
const inputElement2 = input!; // ! means "trust me it's not null"
```

### Type Guards — check the type at runtime safely

```typescript
// typeof guard
function processInput(input: string | number): string {
  if (typeof input === "string") {
    return input.toUpperCase(); // TypeScript knows it's string here
  }
  return input.toFixed(2); // TypeScript knows it's number here
}

1-we use typeof 
// instanceof guard
class Dog {
  bark() { return "Woof"; }
}
class Cat {
  meow() { return "Meow"; }
}

2- we use instanceof 
function makeSound(animal: Dog | Cat): string {
  if (animal instanceof Dog) {
    return animal.bark(); // TypeScript knows it's Dog here
  }
  return animal.meow(); // TypeScript knows it's Cat here
}

// Custom type guard — most powerful
interface Admin {
  role: "admin";
  permissions: string[];
}
interface RegularUser {
  role: "user";
}

function isAdmin(user: Admin | RegularUser): user is Admin {
  return user.role === "admin";
}

function doSomething(user: Admin | RegularUser) {
  if (isAdmin(user)) {
    console.log(user.permissions); // ✅ TypeScript knows it's Admin
  }
}
```

---

## 10. Utility Types

TypeScript has built-in types that transform existing types. Used constantly in NestJS.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  age: number;
}

// Partial — makes all properties optional (done)
type UpdateUserDto = Partial<User>;
// same as: { id?: number; name?: string; email?: string; ... }

const update: UpdateUserDto = { name: "New Name" }; // ✅ only update what changed

// Required — makes all properties required
type RequiredUser = Required<User>;
// all properties must be provided — no optionals

// Pick — select only specific properties
type PublicUser = Pick<User, "id" | "name" | "email">;

// { id: number; name: string; email: string }
// password is removed — never send password to frontend!

// Omit — exclude specific properties
type UserWithoutPassword = Omit<User, "password">;
// { id: number; name: string; email: string; age: number }

// Record — create object type with specific keys and values
type UserRoles = Record<string, "admin" | "user" | "moderator">;
const roles: UserRoles = {
  ali: "admin",
  ahmed: "user",
  sara: "moderator"
};

// Readonly — all properties become read-only
type FrozenUser = Readonly<User>;
const frozenUser: FrozenUser = { id: 1, name: "Ali", email: "ali@gmail.com", password: "123", age: 22 };
frozenUser.name = "Ahmed"; // ❌ Error — cannot modify readonly
```

**Real NestJS pattern using utility types:**

```typescript
// Create DTO — all fields required
interface CreateUserDto {
  name: string;
  email: string;
  password: string;
}

// Update DTO — all fields optional
type UpdateUserDto = Partial<CreateUserDto>;

// Response DTO — never include password
type UserResponseDto = Omit<User, "password">;
```

---

## 11. Decorators

Decorators are the **heart of NestJS**. They are special functions that add behavior to classes, methods, and properties.

```typescript
// Enable in tsconfig.json:
// "experimentalDecorators": true

// Class decorator
function Injectable(constructor: Function) {
  console.log(`${constructor.name} is now injectable`);
}

@Injectable
class UsersService {
  // This is exactly what NestJS does internally
}

// Method decorator
function Log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${key} with`, args);
    return original.apply(this, args);
  };
  return descriptor;
}

class UsersController {
  @Log
  findUser(id: number) {
    return { id, name: "Ali" };
  }
}

// This is EXACTLY how NestJS decorators work
@Controller('users')    // sets route prefix
class UsersController {

  @Get(':id')           // sets GET /users/:id
  findOne(@Param('id') id: string) { // extracts URL parameter
    return id;
  }
}
```

---

## 12. tsconfig.json — option by option

```json
{
  "compilerOptions": {

    "target": "ES2020",
    // What JavaScript version to compile to
    // ES2020 supports async/await, optional chaining natively

    "module": "commonjs",
    // Module system — commonjs for Node.js/NestJS
    // use "ESNext" for browser apps

    "strict": true,
    // Enables ALL strict checks — always use this
    // Catches the most bugs possible

    "strictNullChecks": true,
    // null and undefined are not assignable to other types
    // included in "strict": true

    "noImplicitAny": true,
    // Variables cannot have implicit any type
    // included in "strict": true

    "experimentalDecorators": true,
    // REQUIRED for NestJS — enables @Decorator syntax

    "emitDecoratorMetadata": true,
    // REQUIRED for NestJS — enables dependency injection

    "outDir": "./dist",
    // Where compiled JavaScript files go

    "rootDir": "./src",
    // Where your TypeScript source files are

    "esModuleInterop": true,
    // Allows default imports from CommonJS modules
    // import express from 'express' instead of = require('express')

    "skipLibCheck": true,
    // Skip type checking of node_modules
    // Makes compilation faster

    "declaration": true,
    // Generate .d.ts type definition files
    // Needed when building libraries

    "sourceMap": true
    // Generate source maps for debugging
    // Maps compiled JS back to original TS lines
  }
}
```

---

Do you want to continue to **Part 4 — OOP in TypeScript** or do you need more examples on any concept here? 🚀

---
# Part 4 — OOP in TypeScript

---

## 1. Classes and Objects

```typescript
// Basic class
class User {
  name: string;
  age: number;
  email: string;

  constructor(name: string, age: number, email: string) {
    this.name = name;
    this.age = age;
    this.email = email;
  }

  greet(): string {
    return `Hello, my name is ${this.name}`;
  }
}

// Create object from class
const user = new User("Ali", 22, "ali@gmail.com");
console.log(user.greet()); // Hello, my name is Ali
console.log(user.name);    // Ali
```

---

## 2. Constructors — Shorthand Syntax

TypeScript has a shortcut that saves you from writing properties twice:

```typescript
// ❌ Long way — repeating yourself
class User {
  name: string;
  age: number;
  email: string;

  constructor(name: string, age: number, email: string) {
    this.name = name;
    this.age = age;
    this.email = email;
  }
}

// ✅ Short way — TypeScript shorthand
class User {
  constructor(
    public name: string,
    public age: number,
    public email: string
  ) {}
  // TypeScript automatically creates and assigns the properties
}

const user = new User("Ali", 22, "ali@gmail.com");
console.log(user.name); // Ali ✅
```

This shorthand is used **everywhere in NestJS.**

---

## 3. Access Modifiers

### public — accessible from anywhere (default)

```typescript
class User {
  public name: string; // same as just: name: string

  constructor(name: string) {
    this.name = name;
  }
}

const user = new User("Ali");
console.log(user.name); // ✅ accessible anywhere
user.name = "Ahmed";    // ✅ can be changed
```

### private — only accessible inside the class

```typescript
class BankAccount {
  private balance: number;

  constructor(initialBalance: number) {
    this.balance = initialBalance;
  }

  deposit(amount: number): void {
    this.balance += amount;
  }

  withdraw(amount: number): void {
    if (amount > this.balance) {
      throw new Error("Insufficient funds");
    }
    this.balance -= amount;
  }

  getBalance(): number {
    return this.balance; // only way to read balance from outside
  }
}

const account = new BankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500 ✅
console.log(account.balance);      // ❌ Error — private
account.balance = 999999;          // ❌ Error — private, cannot cheat
```

### protected — accessible inside class and subclasses

```typescript
class Animal {
  protected name: string;

  constructor(name: string) {
    this.name = name;
  }
}

class Dog extends Animal {
  bark(): string {
    return `${this.name} says Woof!`; // ✅ protected — accessible in subclass
  }
}

const dog = new Dog("Rex");
console.log(dog.bark());  // Rex says Woof! ✅
console.log(dog.name);    // ❌ Error — protected, not accessible outside
```

### readonly — can only be set once in constructor

```typescript
class User {
  readonly id: number;
  name: string;

  constructor(id: number, name: string) {
    this.id = id;     // ✅ set in constructor
    this.name = name;
  }

  changeName(newName: string): void {
    this.name = newName; // ✅ name can change
    this.id = 999;       // ❌ Error — readonly cannot change after constructor
  }
}

const user = new User(1, "Ali");
user.name = "Ahmed"; // ✅
user.id = 2;         // ❌ Error — readonly
```

---

## 4. Inheritance and extends

```typescript
// Parent class
class Animal {
  constructor(
    public name: string,
    public age: number
  ) {}

  move(distance: number): string {
    return `${this.name} moved ${distance} meters`;
  }

  toString(): string {
    return `Animal: ${this.name}, Age: ${this.age}`;
  }
}

// Child class inherits from Animal
class Dog extends Animal {
  constructor(
    name: string,
    age: number,
    public breed: string
  ) {
    super(name, age); // MUST call super() first — calls parent constructor
  }

  bark(): string {
    return `${this.name} says Woof!`;
  }

  // Override parent method
  toString(): string {
    return `Dog: ${this.name}, Breed: ${this.breed}`;
  }
}

class Bird extends Animal {
  fly(height: number): string {
    return `${this.name} flew to ${height} meters high`;
  }
}

const dog = new Dog("Rex", 3, "German Shepherd");
console.log(dog.move(10));    // Rex moved 10 meters ✅ inherited
console.log(dog.bark());      // Rex says Woof! ✅ own method
console.log(dog.toString());  // Dog: Rex, Breed: German Shepherd ✅ overridden

const bird = new Bird("Tweety", 1);
console.log(bird.fly(100));   // Tweety flew to 100 meters high ✅
console.log(bird.move(5));    // Tweety moved 5 meters ✅ inherited
```

---

## 5. Abstract Classes

Abstract classes are **blueprints** — you cannot create an object directly from them. They exist only to be extended. Every subclass **must** implement the abstract methods.

```typescript
abstract class Shape {
  abstract getArea(): number;    // no implementation — subclass MUST implement
  abstract getPerimeter(): number;

  // Regular method — shared by all shapes
  describe(): string {
    return `This shape has area: ${this.getArea()} and perimeter: ${this.getPerimeter()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  getArea(): number {
    return Math.PI * this.radius ** 2;
  }

  getPerimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rectangle extends Shape {
  constructor(
    private width: number,
    private height: number
  ) {
    super();
  }

  getArea(): number {
    return this.width * this.height;
  }

  getPerimeter(): number {
    return 2 * (this.width + this.height);
  }
}

const circle = new Circle(5);
console.log(circle.describe());
// This shape has area: 78.53 and perimeter: 31.41

const rect = new Rectangle(4, 6);
console.log(rect.describe());
// This shape has area: 24 and perimeter: 20

const shape = new Shape(); // ❌ Error — cannot instantiate abstract class
```

**NestJS uses abstract classes for base services and repositories:**

```typescript
// NestJS pattern
abstract class BaseService<T> {
  abstract findAll(): Promise<T[]>;
  abstract findOne(id: number): Promise<T>;
  abstract create(data: Partial<T>): Promise<T>;
  abstract delete(id: number): Promise<void>;
}

class UsersService extends BaseService<User> {
  // Must implement all 4 methods or TypeScript gives error
  async findAll(): Promise<User[]> { ... }
  async findOne(id: number): Promise<User> { ... }
  async create(data: Partial<User>): Promise<User> { ... }
  async delete(id: number): Promise<void> { ... }
}
```

---

## 6. Interfaces with Classes

```typescript
interface Flyable {
  fly(height: number): string;
  maxHeight: number;
}

interface Swimmable {
  swim(distance: number): string;
  maxDepth: number;
}

// Class can implement MULTIPLE interfaces
class Duck implements Flyable, Swimmable {
  maxHeight: number = 100;
  maxDepth: number = 5;

  constructor(public name: string) {}

  fly(height: number): string {
    return `${this.name} is flying at ${height} meters`;
  }

  swim(distance: number): string {
    return `${this.name} swam ${distance} meters`;
  }

  quack(): string {
    return `${this.name} says Quack!`;
  }
}

const duck = new Duck("Donald");
console.log(duck.fly(50));    // Donald is flying at 50 meters
console.log(duck.swim(10));   // Donald swam 10 meters
console.log(duck.quack());    // Donald says Quack!
```

**Difference between interface and abstract class:**

||Interface|Abstract Class|
|---|---|---|
|Implementation|No|Yes (partial)|
|Multiple|✅ can implement many|❌ can only extend one|
|Constructor|❌|✅|
|Access Modifiers|❌|✅|

---

## 7. Static Members

Static members belong to the **class itself**, not to instances.

```typescript
class MathHelper {
  static PI: number = 3.14159;
  static instanceCount: number = 0;

  constructor() {
    MathHelper.instanceCount++;
  }

  static square(n: number): number {
    return n * n;
  }

  static cube(n: number): number {
    return n * n * n;
  }
}

// Access without creating an instance
console.log(MathHelper.PI);          // 3.14159
console.log(MathHelper.square(4));   // 16
console.log(MathHelper.cube(3));     // 27

const m1 = new MathHelper();
const m2 = new MathHelper();
console.log(MathHelper.instanceCount); // 2 — shared across all instances

// Real world use case — Singleton pattern
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private connected: boolean = false;

  private constructor() {} // private — cannot use new directly

  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }

  connect(): void {
    this.connected = true;
    console.log("Connected to database");
  }
}

const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true — same instance
```

---

## 8. Polymorphism

Same method name, different behavior depending on the object type.

```typescript
abstract class Payment {
  abstract processPayment(amount: number): string;

  printReceipt(amount: number): void {
    console.log(`Receipt: ${this.processPayment(amount)}`);
  }
}

class CreditCardPayment extends Payment {
  constructor(private cardNumber: string) {
    super();
  }

  processPayment(amount: number): string {
    return `Paid $${amount} with credit card ending in ${this.cardNumber.slice(-4)}`;
  }
}

class PayPalPayment extends Payment {
  constructor(private email: string) {
    super();
  }

  processPayment(amount: number): string {
    return `Paid $${amount} via PayPal account ${this.email}`;
  }
}

class CashPayment extends Payment {
  processPayment(amount: number): string {
    return `Paid $${amount} in cash`;
  }
}

// Polymorphism in action
const payments: Payment[] = [
  new CreditCardPayment("1234567890123456"),
  new PayPalPayment("ali@gmail.com"),
  new CashPayment()
];

// Same method call — different behavior
payments.forEach(payment => payment.printReceipt(100));

// Receipt: Paid $100 with credit card ending in 3456
// Receipt: Paid $100 via PayPal account ali@gmail.com
// Receipt: Paid $100 in cash
```

---

## 9. Encapsulation

Hide internal details, expose only what is needed.

```typescript
class UserAccount {
  private _password: string;
  private _loginAttempts: number = 0;
  private _isLocked: boolean = false;

  constructor(
    public readonly username: string,
    password: string
  ) {
    this._password = this.hashPassword(password);
  }

  // Private helper — hidden from outside
  private hashPassword(password: string): string {
    return `hashed_${password}`; // simplified
  }

  // Controlled access through methods
  login(password: string): boolean {
    if (this._isLocked) {
      throw new Error("Account is locked");
    }

    if (this.hashPassword(password) === this._password) {
      this._loginAttempts = 0;
      return true;
    }

    this._loginAttempts++;
    if (this._loginAttempts >= 3) {
      this._isLocked = true;
    }
    return false;
  }

  // Getter — read-only access to private data
  get isLocked(): boolean {
    return this._isLocked;
  }

  get loginAttempts(): number {
    return this._loginAttempts;
  }

  // Setter — controlled modification
  set password(newPassword: string) {
    if (newPassword.length < 8) {
      throw new Error("Password must be at least 8 characters");
    }
    this._password = this.hashPassword(newPassword);
  }
}

const account = new UserAccount("ali", "secret123");

account.login("wrongpass");
account.login("wrongpass");
account.login("wrongpass");

console.log(account.isLocked);      // true
console.log(account.loginAttempts); // 3
console.log(account._password);     // ❌ Error — private
account.password = "new";           // ❌ Error — too short
account.password = "newpassword";   // ✅ setter validates
```

---

## 10. Real World OOP Example — NestJS Style

```typescript
// Entity
class User {
  constructor(
    public readonly id: number,
    public name: string,
    public email: string,
    private password: string,
    public role: UserRole = UserRole.User
  ) {}

  validatePassword(password: string): boolean {
    return this.password === password; // simplified
  }

  toPublicProfile(): Omit<User, "password" | "validatePassword"> {
    const { password, validatePassword, ...profile } = this as any;
    return profile;
  }
}

// Repository interface
interface IUserRepository {
  findAll(): Promise<User[]>;
  findById(id: number): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<User>;
  delete(id: number): Promise<void>;
}

// Service
class UsersService {
  constructor(private userRepository: IUserRepository) {}

  async getAllUsers(): Promise<User[]> {
    return this.userRepository.findAll();
  }

  async getUserById(id: number): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error(`User with id ${id} not found`);
    }
    return user;
  }

  async createUser(
    name: string,
    email: string,
    password: string
  ): Promise<User> {
    const existing = await this.userRepository.findByEmail(email);
    if (existing) {
      throw new Error("Email already in use");
    }
    const user = new User(Date.now(), name, email, password);
    return this.userRepository.save(user);
  }
}
```

This is **exactly** how NestJS structures its code — you are already thinking in NestJS patterns.

---

Do you want to continue to **Part 5 — Advanced Tricks & Patterns** or do you need more examples? 🚀
---
# Part 5 — Advanced Tricks & Patterns

---

## 1. Mapped Types

Mapped types let you **transform every property** in an existing type automatically.

```typescript
// Manual way — repeating yourself
interface UserReadonly {
  readonly id: number;
  readonly name: string;
  readonly email: string;
}

// Mapped type — automatic transformation
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

// TypeScript already has this built in as Readonly<T>
type ReadonlyUser = MyReadonly<User>;
// { readonly id: number; readonly name: string; readonly email: string }

// Make all properties optional
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

// Make all properties nullable
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

interface User {
  id: number;
  name: string;
  email: string;
}

type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; email: string | null }

// Change all properties to a specific type
type Stringify<T> = {
  [K in keyof T]: string;
};

type StringifiedUser = Stringify<User>;
// { id: string; name: string; email: string }

// Real NestJS use case — make specific keys required, rest optional
type RequireOnly<T, K extends keyof T> = 
  Partial<T> & Required<Pick<T, K>>;

type CreateUserDto = RequireOnly<User, "name" | "email">;
// name and email are required, everything else optional
```

---

## 2. Conditional Types

Types that change based on a condition — like an if statement but for types.

```typescript
// Basic syntax: T extends Condition ? TypeIfTrue : TypeIfFalse
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>;  // "yes"
type B = IsString<number>;  // "no"
type C = IsString<boolean>; // "no"

// Practical example — unwrap Promise type
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type A = UnwrapPromise<Promise<string>>;  // string
type B = UnwrapPromise<Promise<User>>;    // User
type C = UnwrapPromise<string>;           // string (not a promise, returns as is)

// Real world use case
async function fetchUser(): Promise<User> {
  return { id: 1, name: "Ali", email: "ali@gmail.com" };
}

type FetchUserReturn = UnwrapPromise<ReturnType<typeof fetchUser>>;
// User — not Promise<User>

// NonNullable — remove null and undefined
type NonNullable<T> = T extends null | undefined ? never : T;

type A = NonNullable<string | null | undefined>; // string
type B = NonNullable<number | null>;             // number

// Flatten array type
type Flatten<T> = T extends Array<infer U> ? U : T;

type A = Flatten<string[]>;  // string
type B = Flatten<number[]>;  // number
type C = Flatten<string>;    // string (not array, returns as is)
```

---

## 3. Template Literal Types

Combine string types like template literals.

```typescript
type Direction = "top" | "bottom" | "left" | "right";
type Color = "red" | "blue" | "green";

// Generate all combinations automatically
type ColoredDirection = `${Color}-${Direction}`;
// "red-top" | "red-bottom" | "red-left" | "red-right" |
// "blue-top" | "blue-bottom" | ... | "green-right"

// Real use case — CSS properties
type CSSProperty = 
  | `margin-${"top" | "bottom" | "left" | "right"}`
  | `padding-${"top" | "bottom" | "left" | "right"}`;

const prop: CSSProperty = "margin-top";    // ✅
const prop2: CSSProperty = "margin-center"; // ❌ Error

// Event handlers
type EventName = "click" | "focus" | "blur" | "change";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur" | "onChange"

// API routes generator
type HttpMethod = "get" | "post" | "put" | "delete";
type ApiRoute = `/${string}`;
type ApiEndpoint = `${Uppercase<HttpMethod>} ${ApiRoute}`;

const endpoint: ApiEndpoint = "GET /users";   // ✅
const endpoint2: ApiEndpoint = "FETCH /users"; // ❌ Error

// NestJS use case — dynamic property names
type GetterName<T extends string> = `get${Capitalize<T>}`;
type SetterName<T extends string> = `set${Capitalize<T>}`;

type UserGetter = GetterName<"name" | "email">;
// "getName" | "getEmail"
```

---

## 4. Index Signatures

When you don't know the property names in advance but you know their types.

```typescript
// Basic index signature
interface StringMap {
  [key: string]: string;
}

const headers: StringMap = {
  "Content-Type": "application/json",
  "Authorization": "Bearer token123",
  "X-Custom-Header": "value"
  // any string key with string value ✅
};

// Number index signature
interface NumberArray {
  [index: number]: string;
}

const fruits: NumberArray = {
  0: "apple",
  1: "banana",
  2: "cherry"
};

// Mix known and dynamic properties
interface Config {
  name: string;          // known property — required
  version: number;       // known property — required
  [key: string]: unknown; // any additional properties allowed
}

const config: Config = {
  name: "MyApp",
  version: 1,
  debug: true,           // ✅ additional dynamic property
  maxRetries: 3,         // ✅ additional dynamic property
};

// Real NestJS use case — dynamic query params
interface QueryParams {
  page?: number;
  limit?: number;
  [filter: string]: string | number | undefined;
}

function buildQuery(params: QueryParams): string {
  return Object.entries(params)
    .filter(([_, v]) => v !== undefined)
    .map(([k, v]) => `${k}=${v}`)
    .join("&");
}

buildQuery({ page: 1, limit: 10, status: "active", role: "admin" });
// "page=1&limit=10&status=active&role=admin"
```

---

## 5. Declaration Merging

TypeScript merges multiple declarations of the same name into one.

```typescript
// Interface merging — add properties to existing interfaces
interface User {
  name: string;
}

interface User {
  age: number;
}

interface User {
  email: string;
}

// Result — TypeScript merges all three
const user: User = {
  name: "Ali",
  age: 22,
  email: "ali@gmail.com"
  // all three are required
};

// Real world use case — extending third party types
// Imagine Express doesn't have user on Request by default
import { Request } from "express";

declare global {
  namespace Express {
    interface Request {
      user?: User; // add user to every request
    }
  }
}

// Now in NestJS/Express controllers:
app.get("/profile", (req: Request) => {
  console.log(req.user); // ✅ TypeScript knows user exists on request
});

// Module augmentation — extend existing modules
declare module "some-library" {
  interface SomeClass {
    newMethod(): void; // add method to existing class
  }
}
```

---

## 6. Strict Mode Best Practices

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true // enables all of these:
    // strictNullChecks — null/undefined must be handled
    // noImplicitAny — no silent any types
    // strictFunctionTypes — function parameter types checked
    // strictPropertyInitialization — class properties must be initialized
    // noImplicitThis — this must have a known type
  }
}
```

```typescript
// strictNullChecks in action
function getUser(id: number): User | null {
  return id === 1 ? { id: 1, name: "Ali", email: "ali@gmail.com" } : null;
}

const user = getUser(1);
console.log(user.name); // ❌ Error — user might be null

// Fix 1 — optional chaining
console.log(user?.name); // ✅ returns undefined if null

// Fix 2 — null check
if (user) {
  console.log(user.name); // ✅ TypeScript knows user is not null here
}

// Fix 3 — nullish coalescing
console.log(user?.name ?? "Unknown"); // ✅ returns "Unknown" if null

// noImplicitAny in action
function process(data) { // ❌ Error — data implicitly has type any
  return data;
}

function process(data: unknown) { // ✅ explicit type
  return data;
}

// strictPropertyInitialization
class UserService {
  private users: User[]; // ❌ Error — not initialized

  // Fix 1 — initialize in declaration
  private users: User[] = [];

  // Fix 2 — initialize in constructor
  constructor() {
    this.users = [];
  }

  // Fix 3 — definite assignment assertion (use carefully)
  private users!: User[]; // tells TypeScript "trust me, I'll set this"
}
```

---

## 7. Common TypeScript Patterns in NestJS and Angular

### Pattern 1 — Repository Pattern with Generics

```typescript
interface Repository<T, ID> {
  findAll(): Promise<T[]>;
  findById(id: ID): Promise<T | null>;
  save(entity: T): Promise<T>;
  update(id: ID, data: Partial<T>): Promise<T>;
  delete(id: ID): Promise<void>;
}

class UsersRepository implements Repository<User, number> {
  private users: User[] = [];

  async findAll(): Promise<User[]> {
    return this.users;
  }

  async findById(id: number): Promise<User | null> {
    return this.users.find(u => u.id === id) ?? null;
  }

  async save(user: User): Promise<User> {
    this.users.push(user);
    return user;
  }

  async update(id: number, data: Partial<User>): Promise<User> {
    const index = this.users.findIndex(u => u.id === id);
    this.users[index] = { ...this.users[index], ...data };
    return this.users[index];
  }

  async delete(id: number): Promise<void> {
    this.users = this.users.filter(u => u.id !== id);
  }
}
```

### Pattern 2 — DTO Pattern (Data Transfer Object)

```typescript
// Used in every NestJS controller
class CreateUserDto {
  name: string;
  email: string;
  password: string;
  role?: UserRole;
}

class UpdateUserDto extends PartialType(CreateUserDto) {
  // All fields from CreateUserDto become optional automatically
}

class UserResponseDto {
  id: number;
  name: string;
  email: string;
  role: UserRole;
  // NO password — never expose this

  constructor(user: User) {
    this.id = user.id;
    this.name = user.name;
    this.email = user.email;
    this.role = user.role;
  }
}
```

### Pattern 3 — Builder Pattern

```typescript
class QueryBuilder<T> {
  private filters: Partial<T> = {};
  private sortField?: keyof T;
  private sortOrder: "ASC" | "DESC" = "ASC";
  private _limit: number = 10;
  private _page: number = 1;

  where(filters: Partial<T>): this {
    this.filters = { ...this.filters, ...filters };
    return this; // return this for chaining
  }

  orderBy(field: keyof T, order: "ASC" | "DESC" = "ASC"): this {
    this.sortField = field;
    this.sortOrder = order;
    return this;
  }

  limit(limit: number): this {
    this._limit = limit;
    return this;
  }

  page(page: number): this {
    this._page = page;
    return this;
  }

  build(): object {
    return {
      filters: this.filters,
      sort: this.sortField ? { field: this.sortField, order: this.sortOrder } : null,
      pagination: { limit: this._limit, page: this._page }
    };
  }
}

// Clean chaining syntax
const query = new QueryBuilder<User>()
  .where({ role: UserRole.Admin })
  .orderBy("name", "ASC")
  .limit(20)
  .page(2)
  .build();
```

### Pattern 4 — Observer Pattern (used in Angular)

```typescript
type Listener<T> = (data: T) => void;

class EventEmitter<T> {
  private listeners: Listener<T>[] = [];

  subscribe(listener: Listener<T>): () => void {
    this.listeners.push(listener);

    // Return unsubscribe function
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }

  emit(data: T): void {
    this.listeners.forEach(listener => listener(data));
  }
}

// Usage
const userCreated = new EventEmitter<User>();

const unsubscribe = userCreated.subscribe((user) => {
  console.log(`New user created: ${user.name}`);
  // send welcome email, update analytics, etc.
});

userCreated.emit({ id: 1, name: "Ali", email: "ali@gmail.com" });
// New user created: Ali

unsubscribe(); // stop listening
```

### Pattern 5 — Decorator Factory (exactly how NestJS works)

```typescript
// Simple decorator
function Log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function(...args: any[]) {
    console.log(`→ Calling ${key}(${args.join(", ")})`);
    const result = original.apply(this, args);
    console.log(`← ${key} returned: ${JSON.stringify(result)}`);
    return result;
  };
  return descriptor;
}

// Decorator factory — decorator with parameters
function Validate(minLength: number) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function(...args: any[]) {
      const str = args[0];
      if (typeof str === "string" && str.length < minLength) {
        throw new Error(`${key}: argument must be at least ${minLength} characters`);
      }
      return original.apply(this, args);
    };
    return descriptor;
  };
}

// Role guard decorator — like NestJS @Roles()
function Roles(...roles: UserRole[]) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function(currentUser: User, ...args: any[]) {
      if (!roles.includes(currentUser.role)) {
        throw new Error(`Access denied. Required roles: ${roles.join(", ")}`);
      }
      return original.apply(this, [currentUser, ...args]);
    };
    return descriptor;
  };
}

class UsersController {
  @Log
  @Validate(3)
  findByName(name: string) {
    return { name };
  }

  @Roles(UserRole.Admin)
  deleteUser(currentUser: User, userId: number) {
    console.log(`Deleting user ${userId}`);
  }
}

const controller = new UsersController();
controller.findByName("Ali");
// → Calling findByName(Ali)
// ← findByName returned: {"name":"Ali"}

controller.findByName("Al");
// ❌ Error: findByName: argument must be at least 3 characters
```

---

## Putting it all together — Full NestJS-ready service

```typescript
enum UserRole {
  Admin = "ADMIN",
  User = "USER"
}

interface IUser {
  id: number;
  name: string;
  email: string;
  role: UserRole;
}

type CreateUserDto = Omit<IUser, "id">;
type UpdateUserDto = Partial<CreateUserDto>;
type UserResponse = Readonly<IUser>;

interface IUserService {
  findAll(): Promise<UserResponse[]>;
  findById(id: number): Promise<UserResponse>;
  create(dto: CreateUserDto): Promise<UserResponse>;
  update(id: number, dto: UpdateUserDto): Promise<UserResponse>;
  delete(id: number): Promise<void>;
}

class UserService implements IUserService {
  private users: IUser[] = [];
  private nextId: number = 1;

  async findAll(): Promise<UserResponse[]> {
    return this.users;
  }

  async findById(id: number): Promise<UserResponse> {
    const user = this.users.find(u => u.id === id);
    if (!user) throw new Error(`User ${id} not found`);
    return user;
  }

  async create(dto: CreateUserDto): Promise<UserResponse> {
    const user: IUser = { id: this.nextId++, ...dto };
    this.users.push(user);
    return user;
  }

  async update(id: number, dto: UpdateUserDto): Promise<UserResponse> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) throw new Error(`User ${id} not found`);
    this.users[index] = { ...this.users[index], ...dto };
    return this.users[index];
  }

  async delete(id: number): Promise<void> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) throw new Error(`User ${id} not found`);
    this.users.splice(index, 1);
  }
}
```

هذا هو بالضبط الكود اللي هتكتبه في NestJS — كل concept اتعلمته في الـ parts السابقة موجود هنا.

---

Do you want to continue to **Part 6 — Interview Questions**? 🚀