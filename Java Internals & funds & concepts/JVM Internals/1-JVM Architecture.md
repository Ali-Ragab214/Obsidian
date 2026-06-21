ممتاز. 🔥

**JVM Architecture** من أهم المواضيع في Java لأنها تشرح ماذا يحدث منذ لحظة تشغيل البرنامج وحتى تنفيذ الكود وإدارة الذاكرة.

ممكن تضع الشرح التالي مباشرة في Obsidian:

# JVM Architecture

## What is JVM?

JVM اختصار لـ:

```text
Java Virtual Machine
```

وهي البيئة التي تقوم بتشغيل ملفات Java Bytecode.

عندما نكتب:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello JVM");
    }
}
```

يقوم الـ Compiler بتحويله إلى:

```text
Main.class
```

وهو ملف يحتوي على:

```text
Bytecode
```

الـ JVM هي المسؤولة عن قراءة هذا الـ Bytecode وتنفيذه.

---

# Why JVM?

الهدف الأساسي هو:

```text
Write Once, Run Anywhere
```

أي أن البرنامج يُكتب مرة واحدة ويمكن تشغيله على أي نظام يحتوي على JVM.

مثال:

```text
Windows JVM
Linux JVM
Mac JVM
```

نفس ملف الـ `.class` يعمل على الجميع.

---

# High Level Architecture

```text
          Java Source Code
                   |
                   v
              Java Compiler
                   |
                   v
                Bytecode
                   |
                   v
+--------------------------------+
|              JVM               |
|                                |
|  Class Loader                  |
|  Runtime Data Areas            |
|  Execution Engine              |
|  Garbage Collector             |
+--------------------------------+
                   |
                   v
            Operating System
```

---

# Components of JVM

## 1. Class Loader

مسؤول عن تحميل ملفات الـ Classes إلى الذاكرة.

مثال:

```java
User user = new User();
```

عند أول استخدام لـ User:

```text
User.class
```

يتم تحميله بواسطة:

```text
Class Loader
```

---

### مراحل عمل Class Loader

#### Loading

تحميل ملف:

```text
User.class
```

---

#### Linking

يتكون من:

```text
Verification
Preparation
Resolution
```

---

#### Initialization

تنفيذ:

```java
static {
    System.out.println("Loaded");
}
```

---

# 2. Runtime Data Areas

هذه هي الذاكرة الخاصة بالـ JVM.

---

## Method Area

تخزن:

- Class Metadata
    
- Static Variables
    
- Method Information
    

مثال:

```java
public static int count = 0;
```

يتم تخزينه هنا.

---

## Heap Memory

أكبر جزء في الذاكرة.

تخزن فيه جميع Objects.

مثال:

```java
User user = new User();
```

الكائن:

```java
new User()
```

يذهب إلى Heap.

---

### Important

تقريباً:

```text
All Objects → Heap
```

---

## Stack Memory

كل Thread يمتلك Stack خاص به.

تخزن فيه:

- Local Variables
    
- Method Calls
    

مثال:

```java
public void printName() {
    String name = "Ali";
}
```

المتغير:

```java
name
```

يُخزن داخل Stack.

---

## PC Register

اختصار:

```text
Program Counter Register
```

يحفظ عنوان التعليمة الحالية التي ينفذها الـ JVM.

---

## Native Method Stack

عند استدعاء كود Native مثل:

```java
System.arraycopy()
```

يستخدم هذا الجزء.

---

# Heap vs Stack

## Example

```java
public static void main(String[] args) {

    String name = "Ali";

    User user = new User();

}
```

### Stack

```text
name
user reference
```

---

### Heap

```text
User Object
```

---

### Visualization

```text
STACK                     HEAP

user  ------------->     User Object

name = "Ali"
```

---

# 3. Execution Engine

بعد تحميل الـ Bytecode يبدأ التنفيذ.

---

## Interpreter

يقوم بقراءة الـ Bytecode سطرًا سطرًا.

```text
Bytecode
   ↓
Execute
```

عيبه:

```text
Slow
```

---

## JIT Compiler

اختصار:

```text
Just In Time Compiler
```

يقوم بتحويل الأجزاء المستخدمة كثيرًا إلى:

```text
Native Machine Code
```

بدلاً من تفسيرها كل مرة.

---

### Example

لو عندك Loop تعمل مليون مرة:

```java
for(int i = 0; i < 1000000; i++) {
    process();
}
```

يقوم JIT بتحويلها إلى Native Code.

---

### Benefit

```text
Much Faster Execution
```

---

# 4. Garbage Collector

مسؤول عن تنظيف الذاكرة.

---

## Problem

إذا أنشأنا:

```java
User user = new User();
```

ثم لم نعد نستخدمه:

```java
user = null;
```

فالكائن أصبح:

```text
Unreachable
```

---

## Solution

Garbage Collector

يقوم بحذف الكائنات غير المستخدمة تلقائيًا.

---

## Benefit

المبرمج لا يحتاج إلى:

```cpp
delete object;
free(object);
```

كما في C++.

---

# JVM Execution Flow

عند تشغيل البرنامج:

```text
Main.java
      |
      v
javac
      |
      v
Main.class
      |
      v
Class Loader
      |
      v
Runtime Memory
      |
      v
Execution Engine
      |
      v
Output
```

---

# Interview Questions

### What is JVM?

بيئة تشغيل تقوم بتنفيذ Java Bytecode.

---

### What is the difference between JDK, JRE and JVM?

```text
JDK
 └── JRE
      └── JVM
```

- JVM → تشغيل البرنامج.
    
- JRE → JVM + Libraries.
    
- JDK → أدوات التطوير + JRE.
    

---

### Where are Objects stored?

```text
Heap Memory
```

---

### Where are Local Variables stored?

```text
Stack Memory
```

---

### What is JIT Compiler?

محول يقوم بتحويل Bytecode إلى Native Code لتحسين الأداء.

---

### What is Garbage Collection?

عملية إزالة الكائنات غير المستخدمة من الذاكرة تلقائيًا.

---

# Quick Summary

```text
JVM
│
├── Class Loader
│
├── Runtime Data Areas
│   ├── Heap
│   ├── Stack
│   ├── Method Area
│   ├── PC Register
│   └── Native Method Stack
│
├── Execution Engine
│   ├── Interpreter
│   └── JIT Compiler
│
└── Garbage Collector
```

احفظ الرسم الأخير جيدًا، لأنه يمثل **الصورة الكاملة للـ JVM Architecture**، ومنه سننتقل بسهولة إلى مواضيع أعمق مثل **Heap vs Stack**, **Garbage Collection Internals**, **Class Loaders**, و **JIT Compiler**.