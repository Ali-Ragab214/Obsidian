The file extension of **compiled Java bytecode** is:

### `.class`

When you compile a Java source file:

```java
Hello.java
```

using:

```bash
javac Hello.java
```

The Java compiler generates:

```text
Hello.class
```

### What happens?

```text
Hello.java
   ↓ javac
Hello.class
   ↓ JVM
Machine Code
```

- **`.java`** → Java source code written by the developer.
    
- **`.class`** → Compiled **bytecode** understood by the JVM.
    
- **JVM** → Executes the bytecode, typically by interpreting/JIT-compiling it into native machine code.
    

💡 **Interview answer:**

> "Java source code has a `.java` extension, and after compilation by `javac`, it becomes bytecode stored in a `.class` file. The JVM then executes that bytecode."


==the role of Java compiler is== : 
The **Java compiler (`javac`) converts Java source code (`.java`) into bytecode (`.class`) that can run on the JVM.**

source code => bytecode to be run on the jvm 