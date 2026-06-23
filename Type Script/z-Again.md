بما إنك Backend Developer وعندك خلفية في Java وSpring Boot وC# وASP.NET وبدأت تشتغل بـ React، فأنت أصلاً فاهم OOP وGenerics وDependency Injection، وبالتالي TypeScript هتكون سهلة عليك جدًا.

هديك Crash Course شامل يغطي تقريبًا كل حاجة هتحتاجها قبل NestJS.

---

# 1. TypeScript يعني إيه؟

TypeScript عبارة عن:

> JavaScript + Static Typing

يعني JS عادي لكن تضيف Types أثناء الكتابة.

مثال JS:

```js
let name = "Ali";
name = 5;
```

مفيش مشكلة.

---

في TS:

```ts
let name: string = "Ali";

name = 5;
```

خطأ:

```text
Type 'number' is not assignable to type 'string'
```

---

# ليه TypeScript؟

المشاكل في JS:

```js
function add(a, b) {
  return a + b;
}

add(5, "10");
```

النتيجة:

```js
"510"
```

مش Bug وقت Compile.

---

في TS:

```ts
function add(a: number, b: number) {
  return a + b;
}

add(5, "10");
```

Error قبل التشغيل.

---

# JS vs TS

|JS|TS|
|---|---|
|Dynamic Typing|Static Typing|
|Errors وقت التشغيل|Errors وقت التطوير|
|أقل أمانًا|أكثر أمانًا|
|لا يوجد Interfaces|يوجد Interfaces|
|لا يوجد Generics|يوجد Generics|
|لا يوجد Compile Step|يحتاج Compile|


---

# 2. Types الأساسية

## String

```ts
let name: string = "Ali";
```

---

## Number

```ts
let age: number = 22;
```

---

## Boolean

```ts
let isAdmin: boolean = true;
```

---

# Array

```ts
let nums: number[] = [1,2,3];
let nums: number[] = [1,2,3];
```

أو

```ts
let nums: Array<number> = [1,2,3];
```

---

# Tuple

عدد وترتيب ثابت

```ts
let user: [string, number];

user = ["Ali", 23];

let user:[string, number];
user = ["sara",22];
```

صح

```ts
user = [22, "Ali"];
```

خطأ

---

# Enum

يشبه Enum في Java و C#

```ts
enum Role {
  ADMIN,
  USER,
  MANAGER
}
```

---

استخدامه

```ts
let role: Role = Role.ADMIN;

let role: Role = Role.USER;
```

---

Enum بقيم مخصصة

```ts
enum Status {
  SUCCESS = 200,
  NOT_FOUND = 404
}
```

---

# Any

يلغي قوة TypeScript

```ts
let x: any = 5;

x = "Ali";
x = true;
```

تجنب استخدامه.

---

# Unknown

بديل آمن لـ Any

```ts
let value: unknown = "Ali";
let value: unknown = "sara";

if(typeof value ==="string")
{
console.log(value.toupperCase());
}
```

لازم تتأكد من النوع أولًا.

```ts
if(typeof value === "string"){
    console.log(value.toUpperCase());
}
```

---

# Void

دالة لا ترجع شيئًا

```ts
function print(): void {
   console.log("Hello");
}

function print(): void
{
console.log("Hello");
}
```

---

# Never

دالة لا تنتهي

```ts
function throwError(): never {
   throw new Error();
}
```

---

# Null & Undefined

```ts
let x: null = null;
let y: undefined = undefined;
```

---

# 3. Type Inference

TS يستنتج النوع تلقائيًا

```ts
let name = "Ali";
```

يكافئ

```ts
let name: string = "Ali";
```

---

# 4. Union Types

متغير يقبل أكثر من نوع

```ts
let id: string | number;

```

---

```ts
id = 10;
id = "10";
```

---

# 5. Literal Types

```ts
let status: "success" | "error";
```

---

```ts
status = "success";
```

صح

```ts
status = "pending";
```

خطأ

---

# 6. Type Alias

```ts
type UserId = string;
```

---

```ts
let id: UserId = "abc";
```

---

مثال أقوى

```ts
type Status = "pending" | "success" | "failed";
let process: Status = "success";
```

---

# 7. Functions

## Parameters

```ts
function add(a:number,b:number):number{
    return a+b;
}
```

---

# Optional Parameters

```ts
function greet(name?: string){
ال  name  ممكن تكون ب string or undefined
}


function greet(name?: string)
{
if(name)
{
console.log(name.toupperCase());
}
}
  بما انها ممكن تكون ب  undefined  
  فلازم نراعي عشان لو هي بيها مش هينفع نخليها كابيتال وهترمي ايرور 
```

---

# Default Parameters

```ts
function greet(name:string="Guest"){
console.log("Hello ${name}")
}
greet();=>hello Guest
greet("sara");=> hello sara
```

---

# Rest Parameters

```ts
function sum(...nums:number[]){
}
```

---

# Arrow Functions

```ts
const add = (a:number,b:number):number => {
    return a+b;
};

const sub = (a:number , b:number)=> a -b;
```

---

# 8. Objects

```ts
const user: {
    name:string;
    age:number;
} = {
    name:"Ali",
    age:22
};


type product={
id:number,
title:string,
quantity:number
}


let products : product[] = 
[
{
id:1,
title:"electronic",
quantity:3
},
{
id:2,
title:"electric",
quantity:5
}
];
```

---

# 9. Interface

أهم جزء في NestJS

```ts
interface User {
    id:number;
    name:string;
}
```

```ts
interface User{
id:number
}

interface User{
name:string 
}   
 دا صح 
```

```ts 

type Person = {
id:number,
name:string
}
```
---

```ts
const user: User = {
    id:1,
    name:"Ali"
};
```

---

# Optional Property

```ts
interface User {
   id:number;
   phone?:string;
}
```

---

# Readonly

```ts
interface User {
   readonly id:number;
}
```

---

# Interface Inheritance

```ts
interface Person{
   name:string;
}

interface Employee extends Person{
   salary:number;
}
```

---

# 10. Type vs Interface

معظم الوقت:

```ts
interface User {}
```

أفضل للكلاسات والـ DTOs.

---

Type أقوى في:

```ts
type Status = "success" | "error";
```

---

# 11. Classes

```ts
class User {
   name:string;

   constructor(name:string){
      this.name=name;
   }
}
```

---

# Access Modifiers

## Public

```ts
public name:string;
```

---

## Private

```ts
private salary:number;
```

---

## Protected

```ts
protected age:number;
```

---

# Shorthand Constructor

```ts
class User{
   constructor(
      public name:string,
      private age:number
   ){}
}
```

NestJS يستخدمها باستمرار.

---

# Inheritance

```ts
class Person {
   constructor(public name:string){}
}

class Employee extends Person{
   constructor(name:string,
               public salary:number){
      super(name);
   }
}
```

---

# Abstract Classes

```ts
abstract class Animal{
   abstract sound():void;
}
```

---

# 12. Generics

أقوى Feature

بدون Generics:

```ts
function getValue(value:any){
   return value;
}
```

---

بـ Generics:

```ts
function getValue<T>(value:T):T{
    return value;
}
```

---

```ts
getValue<string>("Ali");
```

---

Generic Interface

```ts
interface ApiResponse<T>{
   data:T;
   success:boolean;
}
```

---

```ts
ApiResponse<User>
```

---

# 13. Utility Types

---

## Partial

```ts
interface User{
   name:string;
   age:number;
}
```

---

```ts
type UpdateUser = Partial<User>;
```

---

تصبح:

```ts
{
 name?:string;
 age?:number;
}
```

---

## Required

```ts
Required<User>
```

---

## Readonly

```ts
Readonly<User>
```

---

## Pick

```ts
Pick<User,"name">
```

---

## Omit

```ts
Omit<User,"age">
```

---

NestJS يستخدمهم كثيرًا.

---

# 14. Type Assertions

يشبه Casting

```ts
const input =
document.getElementById("name")
as HTMLInputElement;
```

---

# 15. Narrowing

```ts
function print(value:string|number){
    if(typeof value==="string"){
        console.log(value.toUpperCase());
    }
}
```

---

# 16. Modules

Export

```ts
export class User {}
```

---

Import

```ts
import { User } from "./user";
```

---

# 17. Decorators ⭐

أهم جزء لـ NestJS

```ts
function Log(
 target:any,
 propertyKey:string
){
   console.log(propertyKey);
}
```

---

```ts
class User{

   @Log
   name:string;
}
```

---

NestJS مبني بالكامل على Decorators.

---

```ts
@Controller()
```

```ts
@Get()
```

```ts
@Injectable()
```

---

كلهم Decorators.

---

# 18. Async/Await

```ts
async function getUsers(){
   const users = await fetch(...);
}
```

---

# Promise

```ts
function getUser(): Promise<string>{
   return Promise.resolve("Ali");
}
```

---

# 19. Advanced Types

## Intersection

```ts
type A = {
   name:string;
}

type B = {
   age:number;
}

type C = A & B;
```

---

النتيجة

```ts
{
 name:string;
 age:number;
}
```

---

# 20. Record

```ts
Record<string,string>
```

---

```ts
const users: Record<string,string> = {
  "1":"Ali",
  "2":"Ahmed"
};
```

---

# 21. keyof

```ts
interface User{
  name:string;
  age:number;
}
```

---

```ts
type UserKeys = keyof User;
```

---

النتيجة

```ts
"name" | "age"
```

---

# 22. typeof

```ts
const user = {
  name:"Ali"
};
```

---

```ts
type UserType = typeof user;
```

---

# 23. Mapped Types

```ts
type Optional<T> = {
   [K in keyof T]?: T[K]
}
```

---

# 24. Generics Constraints

```ts
function getLength<T extends {length:number}>
(obj:T){
   return obj.length;
}
```

---

# 25. أهم ما ستراه يوميًا في NestJS

لازم تكون مرتاح جدًا مع:

### Interfaces

```ts
interface UserDto {}
```

---

### Classes

```ts
export class CreateUserDto {}
```

---

### Generics

```ts
Promise<User>
```

---

### Utility Types

```ts
PartialType(CreateUserDto)
```

فكرتها مبنية على:

```ts
Partial<T>
```

---

### Decorators

```ts
@Controller()
```

```ts
@Post()
```

```ts
@Body()
```

```ts
@Injectable()
```

---

### Access Modifiers

```ts
constructor(
 private readonly userService: UserService
){}
```

---

# Roadmap المراجعة قبل NestJS

1. Types الأساسية
    
2. Functions
    
3. Objects
    
4. Interface
    
5. Classes
    
6. Inheritance
    
7. Generics
    
8. Utility Types
    
9. Modules
    
10. Async/Await
    
11. Decorators
    
12. Type Narrowing
    
13. keyof + typeof + Record
    
14. Mapped Types
    

لو أتقنت النقاط دي كلها بنسبة 80% فأنت جاهز تبدأ NestJS مباشرة، لأن حوالي 90% من كود NestJS عبارة عن Classes + Interfaces + Decorators + Generics + Dependency Injection.