# 📘 TypeScript — شرح شامل

> **ملاحظة:** TypeScript هي JavaScript بس بـ **rules** و **safety** زيادة.  
> كل كود TypeScript بيتحوّل في النهاية لـ JavaScript عشان يشتغل في المتصفح.

---

## 🧠 ما هو TypeScript؟

```
TypeScript = JavaScript + Rules + Safety
```

- صدر سنة **2012** من **Microsoft**
- بيتكتب الكود في `.ts` وبيتحوّل (Compile) لـ `.js`
- الـ `.js` هو اللي بيشتغل في الـ HTML

### المراحل التاريخية

|الفترة|التطور|
|---|---|
|2012|الإصدار الأول من Anthropic|
|2013–2015|Classes و Interfaces|
|2016–2018|Type System → Error Detection|
|2019–2026|Performance + Developer Experience + Readability|

---

## ⚖️ TypeScript vs JavaScript

```
┌─────────────────────────────┬──────────────────────────────┐
│        TypeScript           │         JavaScript           │
├─────────────────────────────┼──────────────────────────────┤
│  Strongly Typed             │  Dynamic Typed               │
│  Errors → Compile Time      │  Errors → Run Time           │
│  Large Scale Applications   │  Small to Medium Projects    │
│  Compile قبل التشغيل         │  يشتغل مباشرة في المتصفح    │
│  Superset of JS ✅           │  -                           │
└─────────────────────────────┴──────────────────────────────┘
```

> **Superset** = TypeScript فيها كل حاجة في JavaScript + حاجات زيادة.

---

## ⚙️ التثبيت والتشغيل

```bash
# تثبيت TypeScript بشكل Global
npm install typescript -g

# Compile ملف .ts وشغّله في Watch Mode (يتابع التغييرات)
tsc fileName.ts -w

# إنشاء ملف إعدادات tsconfig.json
tsc --init
```

---

## 🔤 Data Types

### المشكلة في JavaScript

```javascript
// في JS ممكن تغير نوع المتغير براحتك ❌
var x = "ahmed"
x = true
x = 2
x = ["sd"]
```

### الحل في TypeScript

```typescript
// تحديد النوع صراحةً ✅
var fName: string = "Mohamed"
// fName = true  ❌ Error!
var adress:string = "shebin"

var age: number = 20
var flag: boolean = false
```

---

### أنواع البيانات الأساسية

#### `string` | `number` | `boolean`

```typescript
var name: string = "Sara"
var score: number = 99
var isActive: boolean = true
```

#### Union Type — أكتر من نوع

```typescript
var age: number | string
age = 20
age = "Twenty"  // كلاهما مقبول ✅



var name : string | bool  => union
name = "cr7"
name = true
```

#### Type Alias — تسمية نوع مخصص

```typescript
type Status = "pending" | "rejected" | "approved"
var orderStatus: Status = "pending"  // ✅


type Status = "pending" | "rejected" | "approved"
var orderstatus : Status = "pending"
// orderStatus = "hello"  ❌ Error!
```

#### `any` — أي نوع (مش موصى بيه)

```typescript
var z: any
z = 6
z = "hello"
z = true
// ⚠️ خطير لأنه بيلغي ميزة TypeScript
```

#### `unknown` — النسخة الآمنة من `any`

```typescript
let value: unknown = 3

// لازم تعمل Type Check الأول
if (typeof value === "string") {
    console.log(value.toUpperCase())  // ✅ آمن
}
```

```
         any          vs         unknown
    ┌────────────┐           ┌────────────┐
    │ أي عملية   │           │ لازم Check │
    │ بدون فحص   │           │ قبل الاستخدام│
    │ ⚠️ Unsafe  │           │ ✅ Safe     │
    └────────────┘           └────────────┘
```

---

### Arrays

```typescript
let scores: number[] = [9, 10, 20]
let names: string[] = ["Ahmed", "Sara"]

// Array بأكتر من نوع
let mixed: (number | string)[] = [9, "fg"]




======array======
let nums : number[] = [12,12,21]
let names : string[] = ["ali","sara"]

```

---

### Tuple — ترتيب محدد ونوع محدد

```typescript
let user: [string, number] = ["kareem", 22]
let location: [number, number] = [30.62828, 32.27282]
```

```
Tuple vs Array:
┌──────────────────────────────────┐
│  Array: أي عدد من عناصر نفس النوع │
│  Tuple: عدد محدد + نوع لكل موضع  │
└──────────────────────────────────┘
```

---

## 🔧 Functions

### الشكل الأساسي

```typescript
function add(x: number, y: number): number {
    return x + y
}
```

### Return Type — `void` لو مفيش return

```typescript
function greet(name: string): void {
    console.log("Hello " + name)
}
```

### Default Parameter

```typescript
function greet(name: string, age: number = 0) {
    console.log(name, age)
}

greet("Ahmed", 23)   // Ahmed 23
greet("Mohamed")     // Mohamed 0
```

### Optional Parameter — `?`

```typescript
function greet(name: string, age?: number) {
    console.log(name, age)
}

greet("Ahmed", 23)  // ✅
greet("Mohamed")    // ✅ age = undefined
```

---

### Overloading — تعريف الدالة بطرق مختلفة

```typescript
// Signatures
function combine(num1: number, num2: number): number
function combine(name1: string, name2: string): string

// Implementation
function combine(a: any, b?: any): any {
    return a + b
}

combine(2, 3)          // ✅ number
combine("ddd", "ddd")  // ✅ string
```

```
         Overloading Flow:
    combine(2, 3) ──────► number signature ──► number result
    combine("a","b") ───► string signature ──► string result
```

---

### Generic Functions — `<T>` النوع المرن

```typescript
// بدل ما تكتب دالة لكل نوع
function echo<T>(x: T): T {
    return x
}

echo<string>("hamada")   // ✅
echo<number>(3)          // ✅
echo<boolean>(true)      // ✅

// مثال عملي
function reverseArray<T>(arr: T[]): T[] {
    return arr.reverse()
}

reverseArray([2, 3, 10, 20])            // number[]
reverseArray(["Mohamed", "Ahmed"])       // string[]
```

```
    Generic = Type Placeholder
    
    echo<string>  ──► T becomes string
    echo<number>  ──► T becomes number
    echo<boolean> ──► T becomes boolean
```

---

## 🏗️ Class

```typescript
class Person { //مختصر
    constructor(public name: string, private age: number) {
        // public name و private age بيتعملوا تلقائياً
    }

    greet() {
        console.log("Hello " + this.name)
    }
}

class Person 
{
constructor(public name:string , private age :number)
{}

greet()
{
}
}


let p1 = new Person("Sara", 22)
p1.greet()    // ✅
p1.name       // ✅ public
// p1.age     // ❌ private
```

### Access Modifiers

```
┌────────────┬───────────────────────────────────────────┐
│  Modifier  │  الوصول                                   │
├────────────┼───────────────────────────────────────────┤
│  public    │  من أي مكان ✅                              │
│  private   │  داخل الـ Class فقط 🔒                     │
│  protected │  داخل الـ Class والـ SubClasses فقط 🛡️     │
└────────────┴───────────────────────────────────────────┘
```

---

### Inheritance — الوراثة

```typescript
class Animal {
    constructor(public name: string) {}

    move() {
        console.log(this.name + " is Moving")
    }
}

class Dog extends Animal {
    bark() {
        console.log(this.name + " says Woof")
    }

    override move() {
        console.log(this.name + " Dog is Moving")
    }
}

class Cat extends Animal {
    meow() {
        console.log(this.name + " says Meow")
    }
}

let d1 = new Dog("Rex")
d1.move()  // Rex Dog is Moving (override)
d1.bark()  // Rex says Woof
```

```
         Animal
        /      \
      Dog      Cat
    (bark)    (meow)
    (move*)          * = override
```

---

### Static Properties & Methods

```typescript
class Calculator {
    static PI: number = 3.14

    static square(x: number): number {
        return x * x
    }
}

// بدون ما تعمل new
Calculator.PI         // 3.14
Calculator.square(4)  // 16
```

```
    Static = ينتمي للـ Class مش للـ Object
    
    ❌ new Calculator().PI
    ✅ Calculator.PI
```

---

## 📋 Interface

الـ Interface بيحدد **شكل** الـ Object أو الـ Class.

```typescript
interface IPerson {
    id: number
    name: string
    age: number
}

let user: IPerson = {
    id: 2,
    name: "ahmed",
    age: 79
}
```

### Optional Property في Interface

```typescript
interface IProduct {
    name: string
    price: number
    description?: string  // اختياري
}

let p1: IProduct = {
    name: "Product1",
    price: 23
    // description مش لازم ✅
}
```

### Class implements Interface

```typescript
interface Animal {
    name: string
    makeSound(): void
}

class Dog implements Animal {
    constructor(public name: string) {}

    makeSound(): void {
        console.log(this.name + " says Woof!")
    }
}
```

```
    Interface = Contract (عقد)
    Class = بتوقع على العقد وتنفذه

    interface Animal ──► makeSound() مطلوب
         ↑
    class Dog ──implements──► لازم تحط makeSound()
```

### Array من Interface

```typescript
interface IStudent {
    name: string
    grade: number
}

let students: IStudent[] = [
    { name: "Ahmed", grade: 99 },
    { name: "Sara", grade: 85 },
    { name: "Mohamed", grade: 91 },
]

let students : IStudent[] = 
[
{},{},{}
]
```

---

## 🔢 Enum

بيحدد مجموعة ثابتة من القيم المسماة.

### Numeric Enum (الافتراضي)

```typescript
enum Days {
    Monday = 11,
    Tuesday = 12,
    Wednesday = 13
}

let day = Days.Tuesday
console.log(day)  // 12
```

### String Enum

```typescript
enum Gender {
    Male = "Male",
    Female = "Female"
}
```

### مثال عملي — Order Status

```typescript
enum OrderStatus {
    Pending = 10,
    Shipped,     // 11
    Delivered,   // 12
    Cancelled    // 13
}

let myOrder = OrderStatus.Shipped
console.log(myOrder)  // 11
```

```
    Enum Flow:
    
    OrderStatus.Pending   → 10
    OrderStatus.Shipped   → 11 (auto-increment)
    OrderStatus.Delivered → 12
    OrderStatus.Cancelled → 13
```

---

## 📦 Modules

بتقسّم الكود على ملفات منفصلة.

```typescript
// math.ts
export function add(a: number, b: number): number {
    return a + b
}

export const PI = 3.14
```

```typescript
// app.ts
import { add, PI } from "./math"

console.log(add(2, 3))  // 5
console.log(PI)          // 3.14
```

### Compile مع Modules

```bash
tsc --module es2024 module.ts
```

```
    module.ts ──compile──► module.js
    
    Import/Export = نظّم كودك في ملفات
```

---

## 🗺️ خريطة المفاهيم الكاملة

```
                    TypeScript
                        │
        ┌───────────────┼───────────────┐
        │               │               │
    Data Types       OOP Topics      Utilities
        │               │               │
   ┌────┴────┐     ┌────┴────┐     ┌───┴───┐
 string    Tuple  Class   Interface Enum  Modules
 number    Array  Inheritance        
 boolean   any    Static     
 unknown   union  Override   
        │         
   Generic <T>    
   Functions      
```

---

## 💡 ملخص سريع

|المفهوم|الاستخدام|
|---|---|
|`string / number / boolean`|أنواع البيانات الأساسية|
|`union ( \| )`|أكتر من نوع لمتغير واحد|
|`type alias`|اسم مخصص لنوع معين|
|`tuple`|array بترتيب ونوع ثابت|
|`any`|أي نوع (تجنبه)|
|`unknown`|أي نوع لكن بـ type check|
|`Generic <T>`|دالة تتكيف مع أي نوع|
|`class`|نموذج لإنشاء Objects|
|`interface`|عقد لشكل الـ Object أو الـ Class|
|`enum`|مجموعة قيم مسماة وثابتة|
|`modules`|تقسيم الكود على ملفات|

---

> 📌 **نصيحة:** TypeScript مش بتتشغلش مباشرة — لازم `tsc` تحوّلها لـ JavaScript أولاً.