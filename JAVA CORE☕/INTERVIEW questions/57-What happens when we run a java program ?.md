

- **What happens when you run a Java program?**
    
- **Explain the Java execution process.**
    
- **How does the JVM execute a Java program?**
    

وده سؤال بيختبر فهمك للـ **JDK, JVM, JRE, Class Loader, Bytecode, JIT Compiler**.
                  Java Source Code
                     (Main.java)
                          │
                          ▼
                 javac (Java Compiler)
                          │
                          ▼
                 Bytecode (.class file)
                          │
                          ▼
                java Main (Starts JVM)
                          │
                          ▼
              Class Loader Subsystem
          (Loads Main.class & Required Classes)
                          │
                          ▼
               Bytecode Verifier
      (Checks Security & Bytecode Validity)
                          │
                          ▼
              JVM Runtime Memory Creation
        ┌──────────────────────────────────┐
        │ Heap        → Objects            │
        │ Stack       → Local Variables    │
        │ Metaspace   → Class Metadata     │
        └──────────────────────────────────┘
                          │
                          ▼
          JVM Finds & Invokes static main()
                          │
                          ▼
          Interpreter Executes Bytecode
                          │
             Hot Method? (Executed Often)
                    Yes          No
                     │            │
                     ▼            │
         JIT Compiler Compiles    │
        Bytecode → Native Machine │
                 Code             │
                     │            │
                     └──────┬─────┘
                            ▼
                   Native Code Executes
                            │
                            ▼
            Garbage Collector Runs When Needed
                            │
                            ▼
                     Program Terminates

---

# Interview Question

## What happens when we run a Java program?

### Short Answer

When we run a Java program, the source code is first compiled into -> bytecode by the Java compiler (`javac`). Then, the Java Virtual Machine (JVM) loads, verifies, and executes the bytecode. Frequently executed code is optimized by the Just-In-Time (JIT) compiler into native machine code for better performance.

---

# Step 1: Write the Java Source Code

Suppose we have:

```java
public class Main {

    public static void main(String[] args) {
        System.out.println("Hello World");
    }

}
```

The file is saved as:

```text
Main.java
```

At this stage, it's just **human-readable source code**.

---

# Step 2: Compilation (`javac`)

When you run:

```bash
javac Main.java
```

The Java compiler (`javac`) checks:

- Syntax errors
    
- Type checking
    
- Compile-time errors
    

If everything is correct, it generates:

```text
Main.class
```

This `.class` file contains **Bytecode**, not machine code.

Bytecode is platform-independent.

---

# Step 3: Running the Program

When you execute:

```bash
java Main
```

The `java` launcher starts the **JVM**.

At this point, no Java code has been executed yet.

The JVM begins preparing the application.

---

# Step 4: Class Loading

The **Class Loader Subsystem** loads the required classes into memory.

Examples:

- Main.class
    
- java.lang.Object
    
- java.lang.String
    
- java.lang.System
    
- java.io.PrintStream
    

The JVM doesn't load every class in Java.

It loads classes **only when needed**.

---

# Step 5: Bytecode Verification

Before execution, the **Bytecode Verifier** checks that the bytecode is safe.

It verifies things like:

- No illegal memory access
    
- Stack usage is correct
    
- Bytecode format is valid
    
- Type safety
    

This is one reason Java is considered secure.

---

# Step 6: Memory Allocation

The JVM creates runtime memory areas.

Some important ones:

### Heap

Stores:

- Objects
    
- Arrays
    

Example:

```java
String name = new String("Ali");
```

The `String` object is stored in the Heap.

---

### Stack

Each thread has its own Stack.

Stores:

- Local variables
    
- Method parameters
    
- Method calls
    

Example:

```java
int age = 22;
```

`age` is stored in the Stack.

---

### Method Area (Metaspace)

Stores:

- Class metadata
    
- Static variables
    
- Method information
    
- Runtime constant pool
    

---

# Step 7: Locate the `main()` Method

The JVM searches for:

```java
public static void main(String[] args)
```

This is the entry point of every Java application.

Since `main()` is `static`, the JVM can invoke it **without creating an object**.

---

# Step 8: Interpretation

Initially, the JVM executes the bytecode using the **Interpreter**.

The interpreter reads bytecode instruction by instruction.

This is simple but slower.

---

# Step 9: JIT Compilation

If the JVM notices that a method is executed many times (a **hot method**), it sends it to the **Just-In-Time (JIT) Compiler**.

The JIT compiles the bytecode into **native machine code**.

Next time the method runs, the JVM executes the native code directly.

This makes Java much faster.

---

# Step 10: Garbage Collection

While the program is running, the **Garbage Collector (GC)** automatically removes objects that are no longer reachable.

Example:

```java
Person p = new Person();

p = null;
```

When no references point to that object anymore, it becomes eligible for garbage collection.

---

# Step 11: Program Termination

When `main()` finishes and there are no non-daemon threads running:

- The JVM releases memory.
    
- Stops all threads.
    
- Terminates the process.
    

---

# Complete Flow

```text
Write Source Code (.java)
          │
          ▼
Compile using javac
          │
          ▼
Bytecode (.class)
          │
          ▼
Start JVM
          │
          ▼
Class Loader loads classes
          │
          ▼
Bytecode Verifier checks safety
          │
          ▼
Runtime Memory is created
          │
          ▼
JVM calls static main()
          │
          ▼
Interpreter executes bytecode
          │
          ▼
JIT compiles hot methods
          │
          ▼
Native Machine Code
          │
          ▼
Garbage Collector runs when needed
          │
          ▼
Program Ends
```

---

# Common Interview Mistakes

❌ **"Java compiles directly to machine code."**

Incorrect.

Java compiles to **Bytecode**, then the JVM executes it.

---

❌ **"`javac` is the JVM."**

Incorrect.

- `javac` = Compiler.
    
- JVM = Runtime Environment that executes bytecode.
    

---

❌ **"The JIT compiles the entire program before execution."**

Incorrect.

The JVM starts by interpreting bytecode. The JIT compiles only **frequently executed ("hot") methods** during runtime.

---

# Interview Answer (1 Minute)

> **When we run a Java program, the source code (`.java`) is first compiled by the `javac` compiler into platform-independent bytecode (`.class`). When we execute the program using the `java` command, the JVM starts and loads the required classes through the Class Loader. The Bytecode Verifier checks the bytecode for security and correctness. The JVM then creates the runtime memory areas such as the Heap, Stack, and Metaspace, locates the static `main()` method, and begins execution. Initially, the Interpreter executes the bytecode, and frequently executed methods are optimized by the JIT Compiler into native machine code for better performance. During execution, the Garbage Collector automatically frees memory occupied by unreachable objects. Finally, when the program finishes and no non-daemon threads remain, the JVM terminates.**

دي إجابة تعتبر **Senior-level** لأنها مش بتقول "بيتحول لـ bytecode وخلاص"، لكن بتشرح دورة حياة تنفيذ البرنامج كاملة.