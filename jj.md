# 🎓 Java Core + Collections: Complete Masterclass

أهلاً! أنا هنا أعلمك Java من الصفر للاحتراف. هنبدأ رحلة منظمة وشاملة. Let's go! 🚀

---

# 📚 PART 1: Java Fundamentals

---

## 1.1 Primitive Types vs Reference Types

### 🔴 Problem

```java
// What happens here? Why does this behave differently?
int a = 5;
int b = a;
b = 10;
System.out.println(a); // Still 5? Or 10?

String s1 = new String("Hello");
String s2 = s1;
// What if we "change" s2?
```

المشكلة: مش عارف ليه بعض المتغيرات بتتأثر ببعض وبعضها لأ.

---

### 💡 Concept: The City Analogy

تخيل الـ **Memory** زي مدينة فيها **حيين**:

```
┌─────────────────────────────────────────────────────┐
│                    MEMORY (RAM)                      │
│                                                      │
│  ┌──────────────────┐    ┌────────────────────────┐  │
│  │      STACK       │    │         HEAP           │  │
│  │  (الحي الصغير)   │    │    (الحي الكبير)       │  │
│  │                  │    │                        │  │
│  │  Fast access     │    │  Slower access         │  │
│  │  Fixed size      │    │  Dynamic size          │  │
│  │  Auto-managed    │    │  GC-managed            │  │
│  │                  │    │                        │  │
│  │  int age = 25    │    │  Patient{name="Ahmed"} │  │
│  │  double salary   │    │  int[] scores          │  │
│  │  boolean active  │    │  String "Hello"        │  │
│  └──────────────────┘    └────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

- **Stack** = شقة صغيرة - بتحط فيها الأشياء البسيطة مباشرة (القيمة نفسها)
- **Heap** = مخزن كبير - بتحط فيه الأشياء الكبيرة، والـ Stack بيحتفظ بـ **عنوان** المخزن

---

### Primitive Types (8 Types)

```
┌──────────┬──────────┬───────────────────────────────┬─────────────┐
│   Type   │   Size   │           Range               │   Default   │
├──────────┼──────────┼───────────────────────────────┼─────────────┤
│  byte    │  8-bit   │         -128 to 127           │      0      │
│  short   │  16-bit  │      -32,768 to 32,767        │      0      │
│  int     │  32-bit  │  -2,147,483,648 to 2,147,483  │      0      │
│  long    │  64-bit  │        -9.2E18 to 9.2E18      │      0L     │
│  float   │  32-bit  │      ~7 decimal digits        │     0.0f    │
│  double  │  64-bit  │      ~15 decimal digits       │     0.0d    │
│  char    │  16-bit  │       '\u0000' to '\uFFFF'    │   '\u0000'  │
│  boolean │  1-bit   │          true/false           │    false    │
└──────────┴──────────┴───────────────────────────────┴─────────────┘
```

---

### ✅ Code Example: Hospital Patient System

```java
// ============================================================
// PRIMITIVE TYPES - Value is stored directly in Stack
// ============================================================
public class PatientRecord {

    public static void demonstratePrimitives() {

        // 🏥 Hospital patient vitals - stored directly in Stack
        int patientAge = 35;           // Stack: [patientAge = 35]
        double bodyTemperature = 37.5; // Stack: [bodyTemperature = 37.5]
        boolean isAdmitted = true;     // Stack: [isAdmitted = true]
        char bloodType = 'A';          // Stack: [bloodType = 'A']
        long nationalId = 12345678901L; // Note the 'L' suffix for long!

        // ✅ Primitive COPY - independent values
        int originalPulse = 80;
        int copiedPulse = originalPulse; // A full COPY of the value

        copiedPulse = 120; // Changes only copiedPulse

        System.out.println("Original pulse: " + originalPulse); // 80 ✅
        System.out.println("Copied pulse: " + copiedPulse);     // 120 ✅
        // They are completely independent!
    }

    /*
     Memory view after copy:
     Stack:
     ┌─────────────────┐
     │ originalPulse=80│  ← untouched
     │ copiedPulse=120 │  ← changed independently
     └─────────────────┘
    */
}
```

```java
// ============================================================
// REFERENCE TYPES - Variable stores a MEMORY ADDRESS (pointer)
// ============================================================
public class Patient {
    String name;
    int age;
    String condition;

    public Patient(String name, int age, String condition) {
        this.name = name;
        this.age = age;
        this.condition = condition;
    }
}

public class HospitalDemo {

    public static void demonstrateReferences() {

        // Patient object lives in HEAP
        // 'patient1' variable in Stack holds the ADDRESS to Heap
        Patient patient1 = new Patient("Ahmed Mohamed", 35, "Diabetes");

        /*
         Memory view:
         Stack:               Heap:
         ┌─────────────┐     ┌──────────────────────────┐
         │ patient1    │────▶│ Patient {                 │
         │ [addr:0x100]│     │   name: "Ahmed Mohamed"  │
         └─────────────┘     │   age: 35                │
                             │   condition: "Diabetes"  │
                             │ }                        │
                             └──────────────────────────┘
        */

        // ⚠️ Reference COPY - BOTH point to SAME object!
        Patient patient2 = patient1; // Copies the ADDRESS, not the object!

        /*
         Memory view after copy:
         Stack:               Heap:
         ┌─────────────┐     ┌──────────────────────────┐
         │ patient1    │────▶│ Patient {                 │
         │ [addr:0x100]│  ┌─▶│   name: "Ahmed Mohamed"  │
         ├─────────────┤  │  │   age: 35                │
         │ patient2    │──┘  │   condition: "Diabetes"  │
         │ [addr:0x100]│     │ }                        │
         └─────────────┘     └──────────────────────────┘
         SAME ADDRESS = SAME OBJECT!
        */

        patient2.condition = "Hypertension"; // Modifies the SHARED object!

        System.out.println(patient1.condition); // "Hypertension" 😱
        System.out.println(patient2.condition); // "Hypertension"
        // Both see the change because they point to the SAME object!

        // ✅ CORRECT WAY: Create a new object (shallow copy)
        Patient patient3 = new Patient(patient1.name, patient1.age, patient1.condition);
        patient3.condition = "Fracture"; // Only affects patient3

        System.out.println(patient1.condition); // "Hypertension" ✅ unchanged
        System.out.println(patient3.condition); // "Fracture" ✅
    }
}
```

---

## 1.2 Type Casting

### 💡 Concept: Casting = التحويل بين أنواع البيانات

زي ما بتحول العملات - أحياناً بدون خسارة (دولار لسنت) وأحياناً مع خسارة (سنت لدولار بيتقرب).

```java
public class HospitalBilling {

    public static void demonstrateCasting() {

        // ✅ IMPLICIT CASTING (Widening) - No data loss, Java does it automatically
        // Small type → Large type (آمن تماماً)
        int patientAge = 35;
        long ageAsLong = patientAge;       // int → long (automatic, safe)
        double ageAsDouble = patientAge;   // int → double (automatic, safe)

        float consultationFee = 150.0f;
        double totalBill = consultationFee; // float → double (automatic, safe)

        System.out.println("Age as long: " + ageAsLong);     // 35
        System.out.println("Age as double: " + ageAsDouble); // 35.0

        // Widening order (safe direction):
        // byte → short → int → long → float → double

        // ⚠️ EXPLICIT CASTING (Narrowing) - Possible data loss, YOU must decide
        // Large type → Small type (ممكن تخسر بيانات)
        double medicationDose = 2.75; // e.g., 2.75 mg
        int roundedDose = (int) medicationDose; // Explicit cast required!
        // WARNING: Truncates decimal, doesn't round!
        System.out.println("Rounded dose: " + roundedDose); // 2 (not 3!)

        double annualRevenue = 9_999_999_999.99; // 9.9 billion
        int wrongRevenue = (int) annualRevenue;  // OVERFLOW! Data loss!
        System.out.println("Wrong revenue: " + wrongRevenue); // Garbage value!

        // ✅ Safe way to cast with validation:
        double billAmount = 500.75;
        if (billAmount >= Integer.MIN_VALUE && billAmount <= Integer.MAX_VALUE) {
            int safeBill = (int) billAmount;
            System.out.println("Safe bill: " + safeBill); // 500
        }

        // ✅ Proper rounding instead of truncation:
        int properlyRounded = (int) Math.round(medicationDose); // 3 ✅
        System.out.println("Properly rounded: " + properlyRounded);

        // String Conversions (common in real apps)
        String ageInput = "28";          // from user input/API
        int parsedAge = Integer.parseInt(ageInput);    // String → int
        double parsedFee = Double.parseDouble("299.5"); // String → double

        String ageToString = String.valueOf(parsedAge); // int → String
        String feeToString = String.valueOf(parsedFee); // double → String
        // Or: "" + parsedAge (less preferred, less clear)
    }
}
```

---

## 1.3 String Deep Dive

### 🔴 Problem

```java
// Why does this print false?!
String s1 = new String("Ahmed");
String s2 = new String("Ahmed");
System.out.println(s1 == s2);       // false 😱
System.out.println(s1.equals(s2));  // true ✅
```

### 💡 Concept: String Pool = حوض السباحة

Java عنده مكان خاص في الـ Heap اسمه **String Pool** (أو String Intern Pool). لما بتعمل `String s = "Hello"` بدون `new`، Java بيدور في الـ Pool الأول - لو لقاه بيرجع نفس المرجع!

```
┌─────────────────────────────────────────────────────────┐
│                        HEAP                             │
│                                                         │
│  ┌──────────────────────────────┐                       │
│  │        STRING POOL           │                       │
│  │  ┌──────────┐ ┌───────────┐  │                       │
│  │  │ "Ahmed"  │ │ "Mohamed" │  │                       │
│  │  └────▲─────┘ └───────────┘  │                       │
│  └───────│──────────────────────┘                       │
│          │                                              │
│  s1 ─────┘  (both point to same pool object)           │
│  s2 ─────┘                                             │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐                       │
│  │new String() │  │new String() │  ← OUTSIDE pool!      │
│  │  "Ahmed"    │  │  "Ahmed"    │  ← TWO separate objs  │
│  └─────────────┘  └─────────────┘                       │
└─────────────────────────────────────────────────────────┘
```

### ✅ Code Example: Patient Name Management

```java
public class StringDeepDive {

    public static void stringPoolDemo() {

        // ✅ String Literals → use String Pool
        String doctor1Name = "Dr. Ahmed";
        String doctor2Name = "Dr. Ahmed"; // Reuses same pool object!

        System.out.println(doctor1Name == doctor2Name);      // true (same reference)
        System.out.println(doctor1Name.equals(doctor2Name)); // true (same content)

        // ❌ new String() → creates NEW object OUTSIDE pool (wasteful)
        String patient1 = new String("Mohamed Ali");
        String patient2 = new String("Mohamed Ali");

        System.out.println(patient1 == patient2);      // false! Different objects!
        System.out.println(patient1.equals(patient2)); // true (same content)

        // ✅ Force into pool using .intern()
        String interned1 = patient1.intern();
        String interned2 = patient2.intern();
        System.out.println(interned1 == interned2); // true ✅

        // RULE: ALWAYS use .equals() to compare String content, NEVER ==
    }

    public static void stringImmutabilityDemo() {

        // Strings are IMMUTABLE - every "modification" creates a NEW object!
        String patientName = "Ahmed";
        System.out.println(System.identityHashCode(patientName)); // e.g., 1829164700

        patientName = patientName + " Mohamed"; // Creates NEW String object!
        System.out.println(System.identityHashCode(patientName)); // DIFFERENT hash!

        // The original "Ahmed" is now orphaned (eligible for GC)

        /*
         Memory view:
         Before: patientName → "Ahmed" [Pool]
         After:  patientName → "Ahmed Mohamed" [new Heap object]
                              "Ahmed" [still in Pool, may be GC'd]
        */

        // ❌ BAD: String concatenation in loops (creates N objects!)
        String report = "";
        String[] medications = {"Aspirin", "Metformin", "Lisinopril", "Atorvastatin"};

        for (String med : medications) {
            report = report + med + ", "; // 💀 Creates new String every iteration!
        }
        // This created 8+ String objects in the Heap!

        // ✅ GOOD: Use StringBuilder for mutable string building
        StringBuilder reportBuilder = new StringBuilder();
        for (String med : medications) {
            reportBuilder.append(med).append(", ");
        }
        String finalReport = reportBuilder.toString(); // Only ONE conversion at end
        System.out.println(finalReport);
    }

    public static void stringBuilderVsBuffer() {

        // StringBuilder: Fast, NOT thread-safe (use in single-threaded code)
        StringBuilder sb = new StringBuilder();
        sb.append("Patient: Ahmed Mohamed")
          .append("\nAge: 35")
          .append("\nDiagnosis: Hypertension")
          .append("\nMedication: Lisinopril 10mg");

        sb.insert(0, "=== MEDICAL REPORT ===\n"); // Insert at index
        sb.replace(0, 20, "=== PATIENT REPORT ==="); // Replace range
        sb.delete(0, 5); // Delete range

        System.out.println(sb.toString());
        System.out.println("Length: " + sb.length());
        System.out.println("Char at 3: " + sb.charAt(3));

        // StringBuffer: Slower, thread-safe (use when multiple threads access)
        StringBuffer threadSafeBuffer = new StringBuffer();
        threadSafeBuffer.append("Shared medical log - thread safe");
        // Every method is synchronized
    }

    public static void usefulStringMethods() {

        String patientInfo = "  Ahmed Mohamed, Age: 35, Blood: A+  ";

        // Cleaning input
        System.out.println(patientInfo.trim());           // Remove leading/trailing spaces
        System.out.println(patientInfo.strip());          // Java 11+ (handles Unicode whitespace)
        System.out.println(patientInfo.toUpperCase());
        System.out.println(patientInfo.toLowerCase());

        // Searching
        System.out.println(patientInfo.contains("Ahmed"));     // true
        System.out.println(patientInfo.startsWith("  Ahmed")); // true
        System.out.println(patientInfo.endsWith("+  "));       // true
        System.out.println(patientInfo.indexOf("Age"));        // position

        // Splitting and joining
        String csvRecord = "Ahmed,35,Male,A+,Hypertension";
        String[] fields = csvRecord.split(",");
        System.out.println(fields[0]); // Ahmed
        System.out.println(fields[4]); // Hypertension

        String rejoined = String.join(" | ", fields);
        System.out.println(rejoined); // Ahmed | 35 | Male | A+ | Hypertension

        // Replacing
        String masked = patientInfo.replace("Ahmed", "****");
        String regexMasked = patientInfo.replaceAll("\\d+", "XX"); // Replace all numbers

        // Formatting (Java 15+ Text Blocks shown later)
        String formatted = String.format(
            "Patient: %s | Age: %d | Temp: %.1f°C",
            "Ahmed", 35, 37.5
        );
        System.out.println(formatted);

        // Java 11+ useful methods
        System.out.println("   ".isBlank());  // true (only whitespace)
        System.out.println("Ahmed".isBlank()); // false

        String multiLine = "Line1\nLine2\nLine3";
        multiLine.lines().forEach(System.out::println); // Java 11+

        "Ahmed".repeat(3); // "AhmedAhmedAhmed" - Java 11+
    }
}
```

---

### 📊 String vs StringBuilder vs StringBuffer

```
┌─────────────────┬──────────────┬────────────────┬────────────────┐
│   Feature       │    String    │  StringBuilder │  StringBuffer  │
├─────────────────┼──────────────┼────────────────┼────────────────┤
│ Mutability      │  Immutable   │    Mutable     │    Mutable     │
│ Thread Safety   │    Safe*     │   Not Safe     │     Safe       │
│ Performance     │   Slowest    │    Fastest     │    Medium      │
│ Use Case        │ Fixed values │  Single thread │ Multi-thread   │
│ Since           │   Java 1     │    Java 5      │    Java 1      │
└─────────────────┴──────────────┴────────────────┴────────────────┘
* Immutable objects are inherently thread-safe
```

---

## 1.4 Autoboxing & Unboxing + Wrapper Classes

### 💡 Concept: كل primitive ليه Wrapper Class

```
┌──────────────┬──────────────┬────────────────────────────┐
│  Primitive   │   Wrapper    │   Common Methods            │
├──────────────┼──────────────┼────────────────────────────┤
│  int         │  Integer     │  parseInt(), valueOf(),     │
│              │              │  MAX_VALUE, MIN_VALUE       │
├──────────────┼──────────────┼────────────────────────────┤
│  long        │  Long        │  parseLong(), MAX_VALUE     │
├──────────────┼──────────────┼────────────────────────────┤
│  double      │  Double      │  parseDouble(), isNaN()     │
├──────────────┼──────────────┼────────────────────────────┤
│  boolean     │  Boolean     │  parseBoolean()             │
├──────────────┼──────────────┼────────────────────────────┤
│  char        │  Character   │  isDigit(), isLetter(),     │
│              │              │  toUpperCase()              │
├──────────────┼──────────────┼────────────────────────────┤
│  byte        │  Byte        │  parseByte()                │
├──────────────┼──────────────┼────────────────────────────┤
│  short       │  Short       │  parseShort()               │
├──────────────┼──────────────┼────────────────────────────┤
│  float       │  Float       │  parseFloat()               │
└──────────────┴──────────────┴────────────────────────────┘
```

```java
public class PharmacyInventory {

    public static void autoboxingDemo() {

        // Autoboxing: primitive → Wrapper (Java does it automatically)
        int stockCount = 100;
        Integer boxedStock = stockCount; // Java auto-converts: Integer.valueOf(100)

        // Unboxing: Wrapper → primitive (Java does it automatically)
        Integer reorderLevel = Integer.valueOf(20);
        int primitiveLevel = reorderLevel; // Java auto-converts: reorderLevel.intValue()

        // Where you NEED Wrapper classes (Collections only accept Objects!)
        List<Integer> stockLevels = new ArrayList<>();
        stockLevels.add(100); // Autoboxing: int → Integer automatically
        stockLevels.add(250);
        stockLevels.add(75);

        int firstStock = stockLevels.get(0); // Unboxing: Integer → int automatically

        // ⚠️ PERFORMANCE TRAP: Autoboxing in loops
        // ❌ BAD: Creates 10,000 Integer objects!
        Long totalPharmacyRevenue = 0L; // Long wrapper (NOT long primitive)
        for (long i = 0; i < 10_000; i++) {
            totalPharmacyRevenue += i; // Unbox Long → long, add, re-box to Long!
        }

        // ✅ GOOD: Use primitive for calculations
        long totalRevenue = 0L; // primitive long
        for (long i = 0; i < 10_000; i++) {
            totalRevenue += i; // No boxing/unboxing!
        }

        // ⚠️ DANGEROUS: NullPointerException with unboxing!
        Integer nullableStock = null; // Valid for wrapper
        // int primitiveStock = nullableStock; // 💀 NullPointerException!

        // ✅ Safe unboxing
        if (nullableStock != null) {
            int safeStock = nullableStock;
        }

        // ⚠️ == vs equals() with Integer cache!
        Integer a = 127;
        Integer b = 127;
        System.out.println(a == b); // true! (cached range: -128 to 127)

        Integer c = 128;
        Integer d = 128;
        System.out.println(c == d); // false! (outside cache range)
        System.out.println(c.equals(d)); // true ✅ Always use equals()!

        // Useful Wrapper methods
        System.out.println(Integer.MAX_VALUE);     // 2,147,483,647
        System.out.println(Integer.MIN_VALUE);     // -2,147,483,648
        System.out.println(Integer.toBinaryString(255)); // 11111111
        System.out.println(Integer.toHexString(255));    // ff
        System.out.println(Integer.parseInt("42"));      // 42

        // Character utilities (useful for input validation)
        char input = '5';
        System.out.println(Character.isDigit(input));   // true
        System.out.println(Character.isLetter(input));  // false
        System.out.println(Character.isUpperCase('A')); // true
        System.out.println(Character.toUpperCase('a')); // 'A'
    }
}
```

---

## 1.5 Varargs

### 💡 Concept: عدد متغير من المعطيات

```java
public class LabResults {

    // ❌ BAD: Multiple overloaded methods
    public static double average(double a, double b) { return (a + b) / 2; }
    public static double average(double a, double b, double c) { return (a + b + c) / 3; }
    // What if we need 4, 5, 6...? 😤

    // ✅ GOOD: Varargs - accepts any number of arguments
    public static double calculateAverageResult(double... readings) {
        // 'readings' is treated as a double[] array inside the method
        if (readings.length == 0) {
            throw new IllegalArgumentException("At least one reading required");
        }

        double sum = 0;
        for (double reading : readings) {
            sum += reading;
        }
        return sum / readings.length;
    }

    // ✅ Mixing regular params with varargs (varargs MUST be last)
    public static String formatLabReport(String patientName, String testType, double... values) {
        StringBuilder sb = new StringBuilder();
        sb.append("Patient: ").append(patientName)
          .append(" | Test: ").append(testType)
          .append(" | Values: ");

        for (double v : values) {
            sb.append(v).append(" ");
        }
        return sb.toString();
    }

    public static void main(String[] args) {
        System.out.println(calculateAverageResult(98.6));              // 1 arg
        System.out.println(calculateAverageResult(98.6, 99.1, 97.8)); // 3 args
        System.out.println(calculateAverageResult(7.2, 7.5, 7.1, 7.4, 7.6)); // 5 args

        // Can also pass an array
        double[] bloodSugarReadings = {90, 95, 88, 92, 87};
        System.out.println(calculateAverageResult(bloodSugarReadings));

        System.out.println(formatLabReport("Ahmed", "Blood Sugar", 90, 95, 88));
    }
}
```

---

# 📚 PART 2: OOP Pillars — Deep Dive

---

## 2.1 Encapsulation

### 🔴 Problem

```java
// ❌ BAD: Everything public - anyone can corrupt data!
public class BankAccount {
    public double balance = 1000.0;
    public String pin = "1234"; // Anyone can read your PIN! 😱
}

// Attacker code:
BankAccount acc = new BankAccount();
acc.balance = -999999; // Set negative balance!
acc.pin = "0000";      // Change PIN without auth!
```

### ✅ GOOD: Encapsulation with Private Fields

```java
public class BankAccount {

    // 🔒 Private fields - NOBODY can access directly from outside
    private String accountNumber;
    private String ownerName;
    private double balance;
    private String pin;
    private boolean frozen;
    private List<String> transactionHistory;

    // Constructor - controlled initialization
    public BankAccount(String accountNumber, String ownerName, String pin, double initialDeposit) {
        if (initialDeposit < 0) {
            throw new IllegalArgumentException("Initial deposit cannot be negative");
        }
        if (pin == null || pin.length() != 4 || !pin.matches("\\d{4}")) {
            throw new IllegalArgumentException("PIN must be 4 digits");
        }

        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = initialDeposit;
        this.pin = pin;
        this.frozen = false;
        this.transactionHistory = new ArrayList<>();
        this.transactionHistory.add("Account opened with balance: " + initialDeposit);
    }

    // ✅ Getter - READ ONLY access (no setter for balance!)
    public double getBalance() {
        return balance;
    }

    public String getAccountNumber() {
        return accountNumber;
    }

    public String getOwnerName() {
        return ownerName;
    }

    // ✅ Controlled WRITE access through business methods
    public void deposit(double amount, String inputPin) {
        validatePin(inputPin);
        if (frozen) throw new IllegalStateException("Account is frozen");
        if (amount <= 0) throw new IllegalArgumentException("Deposit amount must be positive");

        this.balance += amount;
        transactionHistory.add("Deposit: +" + amount + " | Balance: " + balance);
    }

    public void withdraw(double amount, String inputPin) {
        validatePin(inputPin);
        if (frozen) throw new IllegalStateException("Account is frozen");
        if (amount <= 0) throw new IllegalArgumentException("Withdrawal amount must be positive");
        if (amount > balance) throw new IllegalStateException("Insufficient funds");

        this.balance -= amount;
        transactionHistory.add("Withdrawal: -" + amount + " | Balance: " + balance);
    }

    public boolean changePin(String oldPin, String newPin) {
        validatePin(oldPin);
        if (newPin == null || !newPin.matches("\\d{4}")) {
            throw new IllegalArgumentException("New PIN must be 4 digits");
        }
        this.pin = newPin;
        return true;
    }

    // Private helper - internal use only
    private void validatePin(String inputPin) {
        if (!this.pin.equals(inputPin)) {
            throw new SecurityException("Invalid PIN");
        }
    }

    public List<String> getTransactionHistory(String inputPin) {
        validatePin(inputPin);
        return Collections.unmodifiableList(transactionHistory); // Read-only view!
    }

    @Override
    public String toString() {
        return String.format("Account[%s | Owner: %s | Balance: %.2f EGP]",
            accountNumber, ownerName, balance);
    }
}

// Usage:
// BankAccount acc = new BankAccount("ACC-001", "Ahmed", "1234", 5000);
// acc.deposit(1000, "1234");  ✅
// acc.balance = -999;          ❌ Compile error! Field is private
// acc.pin = "0000";            ❌ Compile error! Field is private
```

---

## 2.2 Inheritance

### 💡 Concept: IS-A Relationship

```
Employee (Base/Parent/Super class)
    ├── Doctor      IS-A Employee ✅
    ├── Nurse       IS-A Employee ✅
    └── AdminStaff  IS-A Employee ✅
```

```java
// ============================================================
// BASE CLASS (Parent/Super)
// ============================================================
public class Employee {

    private String employeeId;
    private String name;
    private String email;
    private double baseSalary;
    private LocalDate hireDate;

    // Constructor
    public Employee(String employeeId, String name, String email, double baseSalary) {
        this.employeeId = employeeId;
        this.name = name;
        this.email = email;
        this.baseSalary = baseSalary;
        this.hireDate = LocalDate.now();
    }

    // Methods
    public double calculateSalary() {
        return baseSalary; // Base implementation
    }

    public String generateReport() {
        return String.format("Employee[ID:%s, Name:%s, Salary:%.2f]",
            employeeId, name, calculateSalary());
    }

    // Getters
    public String getEmployeeId() { return employeeId; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public double getBaseSalary() { return baseSalary; }
    protected void setBaseSalary(double salary) { this.baseSalary = salary; } // protected!
}

// ============================================================
// CHILD CLASS 1
// ============================================================
public class Doctor extends Employee {

    private String specialization;
    private int yearsExperience;
    private double consultationFeePerHour;

    // 'super()' MUST be first line - calls parent constructor
    public Doctor(String employeeId, String name, String email,
                  double baseSalary, String specialization, int yearsExperience) {
        super(employeeId, name, email, baseSalary); // Call parent constructor!
        this.specialization = specialization;
        this.yearsExperience = yearsExperience;
        this.consultationFeePerHour = calculateConsultationFee();
    }

    private double calculateConsultationFee() {
        return 100 + (yearsExperience * 50); // More experience = higher fee
    }

    // ✅ @Override - Polymorphism! Doctor gets a different salary calculation
    @Override
    public double calculateSalary() {
        double base = super.calculateSalary(); // Call parent method
        double experienceBonus = yearsExperience * 500;
        double specializationBonus = getSpecializationBonus();
        return base + experienceBonus + specializationBonus;
    }

    private double getSpecializationBonus() {
        return switch (specialization.toLowerCase()) {
            case "cardiology", "neurology" -> 5000;
            case "oncology" -> 4000;
            case "general" -> 1000;
            default -> 2000;
        };
    }

    @Override
    public String generateReport() {
        return super.generateReport() + // Reuse parent logic
               String.format(" | Specialization:%s | Experience:%d years",
                   specialization, yearsExperience);
    }

    // Doctor-specific methods
    public void prescribeMedication(String patientId, String medication) {
        System.out.println("Dr. " + getName() + " prescribed " + medication +
                          " for patient " + patientId);
    }

    public String getSpecialization() { return specialization; }
}

// ============================================================
// CHILD CLASS 2
// ============================================================
public class Nurse extends Employee {

    private String ward;
    private boolean isHeadNurse;
    private int shiftsPerMonth;

    public Nurse(String employeeId, String name, String email,
                 double baseSalary, String ward, boolean isHeadNurse) {
        super(employeeId, name, email, baseSalary);
        this.ward = ward;
        this.isHeadNurse = isHeadNurse;
        this.shiftsPerMonth = 22; // default
    }

    @Override
    public double calculateSalary() {
        double base = super.calculateSalary();
        double headNurseBonus = isHeadNurse ? 2000 : 0;
        double shiftBonus = shiftsPerMonth * 50;
        return base + headNurseBonus + shiftBonus;
    }

    public String getWard() { return ward; }
}

// ============================================================
// USAGE & POLYMORPHISM
// ============================================================
public class HospitalPayroll {

    public static void processPayroll(List<Employee> employees) {
        // Polymorphism: each calls its OWN calculateSalary()!
        for (Employee emp : employees) {
            System.out.printf("%-20s → Salary: %.2f EGP%n",
                emp.getName(), emp.calculateSalary());
        }
    }

    public static void main(String[] args) {
        List<Employee> staff = new ArrayList<>();

        staff.add(new Doctor("DOC-001", "Dr. Ahmed Nasser", "ahmed@hospital.com",
                             10000, "Cardiology", 15));
        staff.add(new Doctor("DOC-002", "Dr. Sara Mostafa", "sara@hospital.com",
                             8000, "General", 3));
        staff.add(new Nurse("NRS-001", "Fatima Hassan", "fatima@hospital.com",
                            5000, "ICU", true));
        staff.add(new Nurse("NRS-002", "Omar Khalid", "omar@hospital.com",
                            4000, "Pediatrics", false));

        processPayroll(staff); // Polymorphism in action!

        // instanceof check before casting
        for (Employee emp : staff) {
            if (emp instanceof Doctor doctor) { // Pattern matching (Java 16+)
                doctor.prescribeMedication("PT-001", "Aspirin");
            }
        }
    }
}
```

---

## 2.3 Abstraction: Abstract Class vs Interface

### 💡 When to use which?

```
Abstract Class:                    Interface:
- IS-A relationship                - CAN-DO / HAS-ABILITY relationship
- Shared state (fields)            - No state (only constants)
- Some implemented methods         - All abstract (before Java 8)
- Single inheritance only          - Multiple implementation ✅
- Use when classes share code      - Use for contracts/capabilities

Example:
Animal (abstract class)            Printable (interface)
├── Dog                            Serializable (interface)
├── Cat                            Comparable (interface)
└── Bird                           Schedulable (interface)
```

```java
// ============================================================
// ABSTRACT CLASS - Payment base for all payment types
// ============================================================
public abstract class Payment {

    // Shared fields (interfaces can't have these)
    protected String paymentId;
    protected double amount;
    protected LocalDateTime timestamp;
    protected String currency;

    // Constructor (interfaces can't have constructors)
    protected Payment(String paymentId, double amount, String currency) {
        this.paymentId = paymentId;
        this.amount = amount;
        this.currency = currency;
        this.timestamp = LocalDateTime.now();
    }

    // ✅ Abstract method - MUST be implemented by subclasses
    public abstract boolean processPayment();
    public abstract String getPaymentType();

    // ✅ Concrete method - shared by ALL subclasses (DRY principle)
    public void generateReceipt() {
        System.out.println("=== PAYMENT RECEIPT ===");
        System.out.println("ID: " + paymentId);
        System.out.println("Type: " + getPaymentType());
        System.out.println("Amount: " + amount + " " + currency);
        System.out.println("Time: " + timestamp);
        System.out.println("Status: " + (processPayment() ? "SUCCESS" : "FAILED"));
    }

    public double getAmount() { return amount; }
    public String getPaymentId() { return paymentId; }
}

// Concrete implementations
public class CreditCardPayment extends Payment {
    private String cardNumber;
    private String cvv;
    private String cardHolderName;

    public CreditCardPayment(String paymentId, double amount, String currency,
                             String cardNumber, String cvv, String cardHolderName) {
        super(paymentId, amount, currency);
        this.cardNumber = maskCardNumber(cardNumber);
        this.cvv = cvv;
        this.cardHolderName = cardHolderName;
    }

    @Override
    public boolean processPayment() {
        // Real implementation: call payment gateway API
        System.out.println("Processing credit card payment: " + cardNumber);
        return amount > 0 && amount <= 50000; // simplified logic
    }

    @Override
    public String getPaymentType() { return "Credit Card"; }

    private String maskCardNumber(String num) {
        return "****-****-****-" + num.substring(num.length() - 4);
    }
}

public class CashPayment extends Payment {
    private double amountReceived;

    public CashPayment(String paymentId, double amount, String currency, double amountReceived) {
        super(paymentId, amount, currency);
        this.amountReceived = amountReceived;
    }

    @Override
    public boolean processPayment() {
        return amountReceived >= amount;
    }

    @Override
    public String getPaymentType() { return "Cash"; }

    public double getChange() { return amountReceived - amount; }
}

// ============================================================
// INTERFACE - capabilities, not identity
// ============================================================
public interface Refundable {
    // All methods are public abstract by default
    boolean processRefund(String reason);
    double calculateRefundAmount();

    // Default method (Java 8+) - optional override
    default String getRefundPolicy() {
        return "Standard 30-day refund policy";
    }

    // Static method (Java 8+) - belongs to interface
    static boolean isRefundEligible(LocalDateTime purchaseDate) {
        return purchaseDate.isAfter(LocalDateTime.now().minusDays(30));
    }
}

public interface Notifiable {
    void sendNotification(String email, String message);
    void sendSMS(String phoneNumber, String message);
}

// ✅ Multiple interface implementation (impossible with abstract class!)
public class CreditCardPaymentFull extends Payment
        implements Refundable, Notifiable {

    private String cardNumber;

    public CreditCardPaymentFull(String paymentId, double amount,
                                  String currency, String cardNumber) {
        super(paymentId, amount, currency);
        this.cardNumber = cardNumber;
    }

    @Override public boolean processPayment() { return true; }
    @Override public String getPaymentType() { return "Credit Card"; }

    // Refundable implementation
    @Override
    public boolean processRefund(String reason) {
        System.out.println("Refunding " + amount + " to card: " + cardNumber);
        return true;
    }

    @Override
    public double calculateRefundAmount() {
        return amount * 0.95; // 5% processing fee
    }

    // Notifiable implementation
    @Override
    public void sendNotification(String email, String message) {
        System.out.println("Email to " + email + ": " + message);
    }

    @Override
    public void sendSMS(String phoneNumber, String message) {
        System.out.println("SMS to " + phoneNumber + ": " + message);
    }
}
```

---

## 2.4 SOLID Principles (Spring Boot Context)

```java
// ============================================================
// S - Single Responsibility Principle
// ============================================================

// ❌ BAD: One class doing too many things
public class OrderServiceBAD {
    public void placeOrder(Order order) { /* business logic */ }
    public void sendEmail(Order order) { /* email sending */ }  // ❌ not its job
    public void saveToDatabase(Order order) { /* DB logic */ }  // ❌ not its job
    public void generatePDF(Order order) { /* PDF generation */ } // ❌ not its job
}

// ✅ GOOD: Each class has ONE reason to change
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final EmailNotificationService emailService;
    private final InvoiceService invoiceService;

    public Order placeOrder(CreateOrderRequest request) {
        Order order = createOrderFromRequest(request);
        Order saved = orderRepository.save(order);
        emailService.sendOrderConfirmation(saved);    // Delegates!
        invoiceService.generateInvoice(saved);        // Delegates!
        return saved;
    }

    private Order createOrderFromRequest(CreateOrderRequest req) {
        // Only business logic here
        return new Order(req.getCustomerId(), req.getItems());
    }
}

@Service
public class EmailNotificationService {
    public void sendOrderConfirmation(Order order) { /* only email logic */ }
    public void sendShippingUpdate(Order order) { /* only email logic */ }
}

// ============================================================
// O - Open/Closed Principle
// ============================================================

// ❌ BAD: Must modify class to add new discount type
public class DiscountCalculatorBAD {
    public double calculate(Order order, String discountType) {
        if (discountType.equals("STUDENT")) return order.getTotal() * 0.1;
        if (discountType.equals("SENIOR")) return order.getTotal() * 0.15;
        // Every new discount type requires MODIFYING this class! ❌
        return 0;
    }
}

// ✅ GOOD: Open for extension, closed for modification
public interface DiscountStrategy {
    double calculate(Order order);
    boolean isApplicable(Customer customer);
}

@Component
public class StudentDiscount implements DiscountStrategy {
    @Override
    public double calculate(Order order) { return order.getTotal() * 0.10; }
    @Override
    public boolean isApplicable(Customer c) { return c.isStudent(); }
}

@Component
public class SeniorDiscount implements DiscountStrategy {
    @Override
    public double calculate(Order order) { return order.getTotal() * 0.15; }
    @Override
    public boolean isApplicable(Customer c) { return c.getAge() >= 60; }
}

// Adding NEW discount = just add new class, no modification!
@Component
public class LoyaltyDiscount implements DiscountStrategy {
    @Override
    public double calculate(Order order) {
        return order.getTotal() * (order.getCustomer().getLoyaltyYears() * 0.02);
    }
    @Override
    public boolean isApplicable(Customer c) { return c.getLoyaltyYears() >= 2; }
}

@Service
public class DiscountService {
    private final List<DiscountStrategy> strategies; // Spring injects all implementations!

    public DiscountService(List<DiscountStrategy> strategies) {
        this.strategies = strategies;
    }

    public double getBestDiscount(Order order, Customer customer) {
        return strategies.stream()
            .filter(s -> s.isApplicable(customer))
            .mapToDouble(s -> s.calculate(order))
            .max()
            .orElse(0);
    }
}

// ============================================================
// L - Liskov Substitution Principle
// ============================================================

// ❌ BAD: Subclass breaks parent's contract
public class Rectangle {
    protected int width, height;
    public void setWidth(int w) { this.width = w; }
    public void setHeight(int h) { this.height = h; }
    public int getArea() { return width * height; }
}

public class SquareBAD extends Rectangle {
    @Override
    public void setWidth(int w) { this.width = w; this.height = w; } // Breaks contract!
    @Override
    public void setHeight(int h) { this.width = h; this.height = h; } // Breaks contract!
}
// Code expecting Rectangle behaves WRONG with Square!

// ✅ GOOD: Both implement common interface
public interface Shape {
    int getArea();
}
public class RectangleGood implements Shape {
    private int width, height;
    public RectangleGood(int w, int h) { this.width = w; this.height = h; }
    @Override public int getArea() { return width * height; }
}
public class SquareGood implements Shape {
    private int side;
    public SquareGood(int side) { this.side = side; }
    @Override public int getArea() { return side * side; }
}

// ============================================================
// I - Interface Segregation Principle
// ============================================================

// ❌ BAD: Fat interface forces unnecessary implementations
public interface HospitalEmployee {
    void treatPatient();      // Only doctors do this
    void giveInjection();     // Only nurses do this
    void processPayroll();    // Only HR does this
    void cleanWards();        // Only cleaners do this
}
// Doctor must implement giveInjection, cleanWards, etc.? NO! ❌

// ✅ GOOD: Small, focused interfaces
public interface PatientCare { void treatPatient(); }
public interface NursingCare { void giveInjection(); void monitorVitals(); }
public interface HRCapable { void processPayroll(); }
public interface FacilityMaintenance { void cleanWards(); }

public class Doctor implements PatientCare { /* only relevant */ }
public class Nurse implements NursingCare { /* only relevant */ }
public class HRStaff implements HRCapable { /* only relevant */ }

// ============================================================
// D - Dependency Inversion Principle
// ============================================================

// ❌ BAD: High-level depends on low-level (concrete)
public class PatientServiceBAD {
    private MySQLPatientRepository repo = new MySQLPatientRepository(); // ❌ Tight coupling!
    // Can't switch to MongoDB without changing this class!
}

// ✅ GOOD: Both depend on abstraction
public interface PatientRepository {
    Patient findById(String id);
    Patient save(Patient patient);
    List<Patient> findAll();
}

@Repository
public class MySQLPatientRepository implements PatientRepository { /* MySQL impl */ }

@Repository
public class MongoPatientRepository implements PatientRepository { /* MongoDB impl */ }

@Service
public class PatientService {
    private final PatientRepository repository; // Depends on abstraction!

    // Spring injects the right implementation via @Qualifier or @Primary
    public PatientService(PatientRepository repository) {
        this.repository = repository;
    }
    // Switch from MySQL to MongoDB = just change Spring config, NO code change! ✅
}
```

---

# 📚 PART 3: Java 8+ Features

---

## 3.1 Lambda Expressions & Functional Interfaces

### 💡 Concept: Lambda = Anonymous Function

```
Traditional:              Lambda:
new Runnable() {          () -> System.out.println("Hello")
  public void run() {
    System.out.println("Hello");
  }
}
```

```java
public class LambdaDeepDive {

    // ============================================================
    // Predicate<T> - Tests a condition, returns boolean
    // ============================================================
    public static void predicateExamples() {

        // Hospital: Filter patients by various criteria
        List<Patient> patients = getPatients();

        Predicate<Patient> isAdult = p -> p.getAge() >= 18;
        Predicate<Patient> isAdmitted = p -> p.isAdmitted();
        Predicate<Patient> isCritical = p -> p.getRiskLevel().equals("CRITICAL");

        // Composing predicates (AND, OR, NOT)
        Predicate<Patient> isAdultAndAdmitted = isAdult.and(isAdmitted);
        Predicate<Patient> isAdultOrCritical = isAdult.or(isCritical);
        Predicate<Patient> isMinor = isAdult.negate();

        List<Patient> criticalAdults = patients.stream()
            .filter(isAdult.and(isCritical))
            .collect(Collectors.toList());

        System.out.println("Critical adults: " + criticalAdults.size());

        // BiPredicate - takes 2 arguments
        BiPredicate<Patient, Doctor> isAssignedTo =
            (patient, doctor) -> patient.getDoctorId().equals(doctor.getId());
    }

    // ============================================================
    // Function<T, R> - Takes T, returns R (transformation)
    // ============================================================
    public static void functionExamples() {

        // Transform Patient to PatientDTO (for API response)
        Function<Patient, PatientDTO> toDTO = patient -> new PatientDTO(
            patient.getId(),
            patient.getName(),
            patient.getAge()
            // Don't expose sensitive medical data!
        );

        Function<String, Integer> parseAge = Integer::parseInt; // Method reference

        // Composing functions
        Function<Patient, String> getPatientSummary =
            toDTO.andThen(dto -> dto.getName() + " (Age: " + dto.getAge() + ")");

        List<Patient> patients = getPatients();
        List<String> summaries = patients.stream()
            .map(getPatientSummary)
            .collect(Collectors.toList());

        // BiFunction - takes 2 args, returns 1
        BiFunction<String, Integer, Patient> createPatient =
            (name, age) -> new Patient(name, age);
    }

    // ============================================================
    // Consumer<T> - Takes T, returns nothing (side effects)
    // ============================================================
    public static void consumerExamples() {

        Consumer<Patient> printPatientInfo = p ->
            System.out.println("Patient: " + p.getName() + " | Age: " + p.getAge());

        Consumer<Patient> sendReminder = p ->
            System.out.println("SMS sent to: " + p.getPhone());

        // Chaining consumers (andThen)
        Consumer<Patient> fullNotification = printPatientInfo.andThen(sendReminder);

        List<Patient> patients = getPatients();
        patients.forEach(fullNotification); // forEach takes Consumer!

        // BiConsumer - takes 2 args
        BiConsumer<Patient, String> logAction =
            (patient, action) -> System.out.println(patient.getName() + " → " + action);

        logAction.accept(patients.get(0), "Admitted to ICU");
    }

    // ============================================================
    // Supplier<T> - Takes nothing, returns T (lazy evaluation)
    // ============================================================
    public static void supplierExamples() {

        // Generate a new patient ID
        Supplier<String> patientIdGenerator =
            () -> "PT-" + System.currentTimeMillis();

        // Lazy initialization (expensive object, create only if needed)
        Supplier<List<Patient>> expensiveQuery =
            () -> fetchAllPatientsFromDatabase(); // Called only when needed!

        // Use with Optional
        Patient patient = findPatient("PT-001")
            .orElseGet(() -> createDefaultPatient()); // Supplier here!

        String id1 = patientIdGenerator.get(); // "PT-1234567890"
        String id2 = patientIdGenerator.get(); // "PT-1234567891"
    }

    private static Optional<Patient> findPatient(String id) { return Optional.empty(); }
    private static Patient createDefaultPatient() { return new Patient("Unknown", 0); }
    private static List<Patient> fetchAllPatientsFromDatabase() { return new ArrayList<>(); }
    private static List<Patient> getPatients() { return new ArrayList<>(); }
}
```

---

## 3.2 Stream API — Full Deep Dive

### 💡 Concept: Stream = Pipeline of operations on data

```
Data Source → [filter] → [map] → [sorted] → [collect]
                ↓           ↓        ↓           ↓
           Intermediate  Intermed  Intermed   Terminal
           (lazy)        (lazy)    (lazy)     (triggers execution)
```

```java
@Service
public class HospitalAnalyticsService {

    private final List<Patient> patients;
    private final List<Doctor> doctors;
    private final List<Appointment> appointments;

    // ============================================================
    // FILTERING
    // ============================================================
    public List<Patient> getCriticalAdultPatients() {
        return patients.stream()
            .filter(p -> p.getAge() >= 18)
            .filter(p -> p.getRiskLevel().equals("CRITICAL"))
            .filter(Patient::isAdmitted)           // Method reference
            .collect(Collectors.toList());
    }

    // ============================================================
    // MAPPING (Transformation)
    // ============================================================
    public List<PatientDTO> getAllPatientsAsDTO() {
        return patients.stream()
            .map(p -> new PatientDTO(p.getId(), p.getName(), p.getAge(), p.getCondition()))
            .collect(Collectors.toList());
    }

    public List<String> getAllPatientNames() {
        return patients.stream()
            .map(Patient::getName)  // Method reference
            .sorted()
            .collect(Collectors.toList());
    }

    // ============================================================
    // FLATMAP (Flatten nested collections)
    // ============================================================
    public List<String> getAllMedicationsAcrossPatients() {
        // Each patient has List<String> medications
        return patients.stream()
            .flatMap(p -> p.getMedications().stream()) // Flatten nested lists!
            .distinct()
            .sorted()
            .collect(Collectors.toList());

        // Without flatMap you'd get Stream<List<String>>
        // With flatMap you get Stream<String> ✅
    }

    // ============================================================
    // COLLECTING - Different ways to collect results
    // ============================================================
    public Map<String, List<Patient>> groupPatientsByCondition() {
        return patients.stream()
            .collect(Collectors.groupingBy(Patient::getCondition));
        // Result: {"Diabetes": [p1, p3], "Hypertension": [p2, p5], ...}
    }

    public Map<String, Long> countPatientsByCondition() {
        return patients.stream()
            .collect(Collectors.groupingBy(
                Patient::getCondition,
                Collectors.counting()
            ));
        // Result: {"Diabetes": 15, "Hypertension": 23, ...}
    }

    public Map<String, Double> averageAgeByCondition() {
        return patients.stream()
            .collect(Collectors.groupingBy(
                Patient::getCondition,
                Collectors.averagingInt(Patient::getAge)
            ));
    }

    public Map<String, Patient> patientsByIdMap() {
        return patients.stream()
            .collect(Collectors.toMap(
                Patient::getId,      // key
                patient -> patient,  // value
                (existing, newer) -> existing // merge function (handle duplicate keys)
            ));
    }

    public Map<Boolean, List<Patient>> partitionByAdmissionStatus() {
        return patients.stream()
            .collect(Collectors.partitioningBy(Patient::isAdmitted));
        // Result: {true: [admitted patients], false: [outpatients]}
    }

    // ============================================================
    // REDUCING & AGGREGATING
    // ============================================================
    public double getTotalMedicationCost() {
        return patients.stream()
            .mapToDouble(Patient::getMedicationCost)
            .sum();
    }

    public OptionalDouble getAveragePatientAge() {
        return patients.stream()
            .mapToInt(Patient::getAge)
            .average();
    }

    public Optional<Patient> getOldestPatient() {
        return patients.stream()
            .max(Comparator.comparingInt(Patient::getAge));
    }

    public long countAdmittedPatients() {
        return patients.stream()
            .filter(Patient::isAdmitted)
            .count();
    }

    // Custom reduce
    public double calculateTotalBill(String patientId) {
        return appointments.stream()
            .filter(a -> a.getPatientId().equals(patientId))
            .map(Appointment::getCost)
            .reduce(0.0, Double::sum); // identity + accumulator
    }

    // ============================================================
    // FINDING
    // ============================================================
    public Optional<Patient> findFirstCriticalPatient() {
        return patients.stream()
            .filter(p -> p.getRiskLevel().equals("CRITICAL"))
            .findFirst(); // Returns first match (deterministic)
    }

    public Optional<Patient> findAnyCriticalPatient() {
        return patients.stream()
            .filter(p -> p.getRiskLevel().equals("CRITICAL"))
            .findAny(); // Faster in parallel streams (any match)
    }

    // ============================================================
    // MATCHING
    // ============================================================
    public boolean hasAnyEmergencyPatient() {
        return patients.stream()
            .anyMatch(p -> p.getRiskLevel().equals("EMERGENCY"));
    }

    public boolean areAllPatientsAdmitted() {
        return patients.stream()
            .allMatch(Patient::isAdmitted);
    }

    public boolean hasNoPediatricPatients() {
        return patients.stream()
            .noneMatch(p -> p.getAge() < 12);
    }

    // ============================================================
    // SORTING
    // ============================================================
    public List<Patient> getPatientsSortedByAge() {
        return patients.stream()
            .sorted(Comparator.comparingInt(Patient::getAge))
            .collect(Collectors.toList());
    }

    public List<Patient> getPatientsSortedByMultipleCriteria() {
        return patients.stream()
            .sorted(Comparator.comparing(Patient::getRiskLevel)
                .thenComparingInt(Patient::getAge)
                .reversed())
            .collect(Collectors.toList());
    }

    // ============================================================
    // LIMIT & SKIP (Pagination)
    // ============================================================
    public List<Patient> getPatientPage(int pageNumber, int pageSize) {
        return patients.stream()
            .sorted(Comparator.comparing(Patient::getName))
            .skip((long) pageNumber * pageSize)  // Skip previous pages
            .limit(pageSize)                      // Take current page
            .collect(Collectors.toList());
    }

    // ============================================================
    // PEEK (Debugging - doesn't affect stream)
    // ============================================================
    public List<Patient> getCriticalPatientsWithLogging() {
        return patients.stream()
            .filter(p -> p.getRiskLevel().equals("CRITICAL"))
            .peek(p -> System.out.println("Processing critical patient: " + p.getName()))
            .map(p -> { p.setAlertSent(true); return p; })
            .peek(p -> System.out.println("Alert sent for: " + p.getName()))
            .collect(Collectors.toList());
        // peek() is great for debugging pipelines!
    }

    // ============================================================
    // DISTINCT
    // ============================================================
    public List<String> getUniqueConditions() {
        return patients.stream()
            .map(Patient::getCondition)
            .distinct()  // Removes duplicates (uses equals/hashCode)
            .sorted()
            .collect(Collectors.toList());
    }

    // ============================================================
    // PARALLEL STREAMS (Use carefully!)
    // ============================================================
    public Map<String, Long> getConditionCountsParallel() {
        return patients.parallelStream() // Split work across CPU cores!
            .collect(Collectors.groupingByConcurrent(
                Patient::getCondition,
                Collectors.counting()
            ));
        // ⚠️ Only use for large data sets (>10,000 items) AND CPU-bound tasks
        // ⚠️ Thread-safe collectors required!
        // ⚠️ Order not guaranteed!
    }

    // ============================================================
    // COMPLETE REAL-WORLD EXAMPLE
    // ============================================================
    public HospitalReport generateDailyReport() {
        LocalDate today = LocalDate.now();

        List<Appointment> todayAppointments = appointments.stream()
            .filter(a -> a.getDate().equals(today))
            .collect(Collectors.toList());

        Map<String, Long> appointmentsByDoctor = todayAppointments.stream()
            .collect(Collectors.groupingBy(
                Appointment::getDoctorName,
                Collectors.counting()
            ));

        double totalRevenue = todayAppointments.stream()
            .mapToDouble(Appointment::getCost)
            .sum();

        List<String> criticalPatients = patients.stream()
            .filter(p -> p.getRiskLevel().equals("CRITICAL"))
            .filter(Patient::isAdmitted)
            .map(Patient::getName)
            .sorted()
            .collect(Collectors.toList());

        OptionalDouble avgWaitTime = todayAppointments.stream()
            .mapToDouble(Appointment::getWaitTimeMinutes)
            .average();

        return new HospitalReport(
            today,
            todayAppointments.size(),
            totalRevenue,
            appointmentsByDoctor,
            criticalPatients,
            avgWaitTime.orElse(0.0)
        );
    }
}
```

---

## 3.3 Optional<T> — Proper Usage

```java
public class PatientLookupService {

    // ============================================================
    // ❌ WRONG ways to use Optional
    // ============================================================
    public Optional<Patient> findByIdBAD(String id) {
        Patient p = repository.findById(id);

        // ❌ Don't use Optional.get() without checking!
        Patient patient = findByIdBAD("123").get(); // NoSuchElementException if empty!

        // ❌ Don't use isPresent() + get() (defeats the purpose)
        Optional<Patient> opt = findByIdBAD("123");
        if (opt.isPresent()) {
            return opt; // Just use old null check style then!
        }

        // ❌ Don't use Optional as method parameter
        // public void update(Optional<Patient> patient) {} // Anti-pattern!

        // ❌ Don't use Optional for collections (return empty list instead)
        // public Optional<List<Patient>> getAll() {} // Anti-pattern!

        return Optional.ofNullable(p);
    }

    // ============================================================
    // ✅ CORRECT ways to use Optional
    // ============================================================
    public Optional<Patient> findById(String id) {
        return Optional.ofNullable(repository.findById(id));
    }

    public void demonstrateOptionalUsage() {

        // ✅ orElse - provide a default value
        Patient patient = findById("PT-001")
            .orElse(new Patient("Unknown", 0)); // Always creates default obj

        // ✅ orElseGet - lazy default (better if object creation is expensive)
        Patient patient2 = findById("PT-001")
            .orElseGet(() -> createDefaultPatient()); // Only creates if empty

        // ✅ orElseThrow - throw custom exception
        Patient patient3 = findById("PT-001")
            .orElseThrow(() -> new PatientNotFoundException("Patient PT-001 not found"));

        // ✅ map - transform if present
        Optional<String> patientName = findById("PT-001")
            .map(Patient::getName);

        // ✅ flatMap - when mapper returns Optional
        Optional<Doctor> assignedDoctor = findById("PT-001")
            .flatMap(p -> findDoctorById(p.getDoctorId()));

        // ✅ filter - apply condition
        Optional<Patient> criticalPatient = findById("PT-001")
            .filter(p -> p.getRiskLevel().equals("CRITICAL"));

        // ✅ ifPresent - do something if present
        findById("PT-001")
            .ifPresent(p -> sendAlertEmail(p.getEmail()));

        // ✅ ifPresentOrElse (Java 9+)
        findById("PT-001")
            .ifPresentOrElse(
                p -> System.out.println("Found: " + p.getName()),
                () -> System.out.println("Patient not found")
            );

        // ✅ Chaining
        String doctorName = findById("PT-001")
            .map(Patient::getDoctorId)
            .flatMap(this::findDoctorById)
            .map(Doctor::getName)
            .orElse("No doctor assigned");

        System.out.println("Doctor: " + doctorName);
    }

    private Optional<Doctor> findDoctorById(String doctorId) { return Optional.empty(); }
    private void sendAlertEmail(String email) { }
    private Patient createDefaultPatient() { return new Patient("Default", 0); }

    // Mock repository
    private PatientRepository repository = new PatientRepository() {
        @Override public Patient findById(String id) { return null; }
        @Override public Patient save(Patient p) { return p; }
        @Override public List<Patient> findAll() { return List.of(); }
    };
}
```

---

# 📚 PART 4: Collections Framework

---

## 4.1 List — ArrayList vs LinkedList

### 💡 Internal Structure

```
ArrayList (Dynamic Array):
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │   │   │ ← contiguous memory
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
                              ↑ size=6, capacity=10

LinkedList (Doubly Linked):
null ← [A|●|●] ↔ [B|●|●] ↔ [C|●|●] ↔ [D|●|null]
        prev next   prev next   prev next
```

```java
@Service
public class AppointmentListService {

    // ============================================================
    // ArrayList - Best for: random access, iteration
    // ============================================================
    public void arrayListDemo() {

        // Default capacity: 10. When full, creates new array of size * 1.5!
        ArrayList<Appointment> appointments = new ArrayList<>();

        // ✅ If you know approximate size, pre-size it (avoids resizing)
        ArrayList<Appointment> bigList = new ArrayList<>(1000);

        // Operations & Performance:
        appointments.add(new Appointment("Dr. Ahmed", "Patient A", LocalTime.of(9, 0)));    // O(1) amortized
        appointments.add(0, new Appointment("Dr. Sara", "Patient B", LocalTime.of(8, 0)));   // O(n) - shifts all!

        Appointment first = appointments.get(0);    // O(1) ✅ Fast random access!
        appointments.remove(0);                      // O(n) - shifts elements
        appointments.remove(appointments.size() - 1); // O(1) - just remove last

        // Iteration (preferred - uses index directly)
        for (Appointment apt : appointments) {
            System.out.println(apt.getTime() + ": " + apt.getPatientName());
        }

        // Searching
        boolean hasEmergency = appointments.stream()
            .anyMatch(a -> a.isEmergency());

        // Bulk operations
        List<Appointment> morningSlots = List.of(
            new Appointment("Dr. A", "P1", LocalTime.of(8, 0)),
            new Appointment("Dr. A", "P2", LocalTime.of(8, 30))
        );
        appointments.addAll(morningSlots);
        appointments.removeIf(a -> a.isCancelled()); // Remove with condition
        appointments.sort(Comparator.comparing(Appointment::getTime));
    }

    // ============================================================
    // LinkedList - Best for: frequent insert/delete at start or middle
    // ============================================================
    public void linkedListDemo() {

        LinkedList<Patient> waitingQueue = new LinkedList<>();

        // Deque operations (use as queue)
        waitingQueue.addLast(new Patient("Ahmed", 35));   // Add to end: O(1)
        waitingQueue.addFirst(new Patient("Emergency!", 45)); // Add to start: O(1)!

        Patient next = waitingQueue.pollFirst();  // Remove from front: O(1)
        Patient last = waitingQueue.pollLast();   // Remove from end: O(1)

        Patient peek = waitingQueue.peekFirst();  // Look without removing

        // ⚠️ LinkedList is SLOW for random access!
        Patient p = waitingQueue.get(5); // O(n) - traverses nodes! ❌ Avoid!

        // ✅ Use LinkedList when:
        // - Implementing a Queue or Deque
        // - Frequent insertions/deletions at beginning
        // - You DON'T need random access
    }
}
```

### 📊 Performance Comparison Table

```
┌───────────────┬────────────────┬────────────────┬─────────────────────────┐
│   Operation   │   ArrayList    │   LinkedList   │      Winner             │
├───────────────┼────────────────┼────────────────┼─────────────────────────┤
│ get(index)    │     O(1) ✅    │     O(n) ❌    │ ArrayList (always)      │
│ add at end    │  O(1) amort.   │     O(1)       │ Tie                     │
│ add at start  │     O(n) ❌    │     O(1) ✅    │ LinkedList              │
│ add at middle │     O(n)       │     O(n)       │ Tie (LinkedList finds   │
│               │                │                │ position slower)        │
│ remove at end │     O(1)       │     O(1)       │ Tie                     │
│ remove start  │     O(n) ❌    │     O(1) ✅    │ LinkedList              │
│ contains()    │     O(n)       │     O(n)       │ Tie                     │
│ Memory        │   Less ✅      │   More ❌      │ ArrayList               │
│ Iteration     │   Faster ✅    │   Slower       │ ArrayList (cache)       │
└───────────────┴────────────────┴────────────────┴─────────────────────────┘

RULE: Use ArrayList by default. Use LinkedList only for Queue/Deque patterns.
```

---

## 4.2 Set — HashSet, LinkedHashSet, TreeSet

```java
public class HospitalRegistrationService {

    // ============================================================
    // HashSet - No order, No duplicates, O(1) operations
    // ============================================================
    public void hashSetDemo() {

        // Track unique medical conditions in hospital
        Set<String> registeredConditions = new HashSet<>();

        registeredConditions.add("Diabetes");       // Added
        registeredConditions.add("Hypertension");   // Added
        registeredConditions.add("Diabetes");       // IGNORED (duplicate!)

        System.out.println(registeredConditions.size()); // 2, not 3!

        // ✅ Fastest for: contains(), add(), remove() - all O(1)
        boolean hasCondition = registeredConditions.contains("Diabetes"); // O(1)!

        // Use case: unique patient IDs, unique email addresses
        Set<String> activePatientIds = new HashSet<>();
        activePatientIds.add("PT-001");
        activePatientIds.add("PT-002");
        boolean alreadyRegistered = activePatientIds.contains("PT-001"); // O(1)

        // ⚠️ Order NOT guaranteed!
        System.out.println(registeredConditions); // Could print in ANY order
    }

    // ============================================================
    // LinkedHashSet - Insertion order preserved, No duplicates
    // ============================================================
    public void linkedHashSetDemo() {

        // Track patient check-in order (unique patients, preserved order)
        Set<String> checkInOrder = new LinkedHashSet<>();

        checkInOrder.add("Ahmed Mohamed");
        checkInOrder.add("Sara Hassan");
        checkInOrder.add("Mohamed Ali");
        checkInOrder.add("Ahmed Mohamed"); // Duplicate - ignored, order not changed

        System.out.println(checkInOrder);
        // [Ahmed Mohamed, Sara Hassan, Mohamed Ali] - INSERTION ORDER preserved! ✅

        // Slightly slower than HashSet due to ordering maintenance
    }

    // ============================================================
    // TreeSet - Sorted order, No duplicates, O(log n) operations
    // ============================================================
    public void treeSetDemo() {

        // Sorted set of available time slots
        TreeSet<LocalTime> availableSlots = new TreeSet<>();
        availableSlots.add(LocalTime.of(14, 0));
        availableSlots.add(LocalTime.of(9, 0));
        availableSlots.add(LocalTime.of(11, 30));
        availableSlots.add(LocalTime.of(16, 0));
        availableSlots.add(LocalTime.of(9, 0)); // Duplicate - ignored

        System.out.println(availableSlots);
        // [09:00, 11:30, 14:00, 16:00] - SORTED automatically! ✅

        // NavigableSet operations (very useful!)
        LocalTime firstSlot = availableSlots.first(); // 09:00
        LocalTime lastSlot = availableSlots.last();   // 16:00

        // Find next available slot after 10:00
        LocalTime nextAfter10 = availableSlots.higher(LocalTime.of(10, 0)); // 11:30

        // Find all slots between 10:00 and 15:00
        NavigableSet<LocalTime> afternoon = availableSlots.subSet(
            LocalTime.of(10, 0), true,  // inclusive
            LocalTime.of(15, 0), true   // inclusive
        );

        // Custom sorting with TreeSet + Comparator
        TreeSet<Doctor> doctorsByExperience = new TreeSet<>(
            Comparator.comparingInt(Doctor::getYearsExperience).reversed()
        );
    }
}
```

---

## 4.3 Map — The Most Important Collection

```java
public class HospitalCacheService {

    // ============================================================
    // HashMap - Most used, no order, O(1) average
    // ============================================================
    public void hashMapDeepDive() {

        // Patient lookup cache: patientId → Patient
        Map<String, Patient> patientCache = new HashMap<>();

        // Basic operations
        patientCache.put("PT-001", new Patient("Ahmed", 35));
        patientCache.put("PT-002", new Patient("Sara", 28));

        Patient p = patientCache.get("PT-001");        // O(1) ✅
        boolean exists = patientCache.containsKey("PT-003"); // O(1) ✅
        patientCache.remove("PT-001");                  // O(1) ✅

        // ✅ Modern put operations (Java 8+)
        // putIfAbsent - put only if key doesn't exist
        patientCache.putIfAbsent("PT-001", new Patient("New Patient", 25));

        // getOrDefault - safe get with fallback
        Patient unknown = patientCache.getOrDefault("PT-999",
            new Patient("Unknown", 0));

        // computeIfAbsent - create value if absent (great for cache!)
        Map<String, List<Appointment>> doctorAppointments = new HashMap<>();
        doctorAppointments.computeIfAbsent("DOC-001", k -> new ArrayList<>())
                          .add(new Appointment("DOC-001", "PT-001", LocalTime.now()));
        // No more: if (!map.containsKey(k)) { map.put(k, new ArrayList()); }!

        // merge - update existing or put new value
        Map<String, Integer> conditionCount = new HashMap<>();
        List<String> conditions = List.of("Diabetes", "Hypertension", "Diabetes", "Flu");

        for (String condition : conditions) {
            conditionCount.merge(condition, 1, Integer::sum);
            // If key absent: put(key, 1)
            // If key present: update value with Integer::sum
        }
        System.out.println(conditionCount); // {Diabetes=2, Hypertension=1, Flu=1}

        // Iterating a Map (3 ways)
        // 1. entrySet (preferred when you need both key and value)
        for (Map.Entry<String, Patient> entry : patientCache.entrySet()) {
            System.out.println(entry.getKey() + " → " + entry.getValue().getName());
        }

        // 2. forEach (Java 8+ - cleaner)
        patientCache.forEach((id, patient) ->
            System.out.println(id + " → " + patient.getName()));

        // 3. keySet (only if you need keys)
        for (String id : patientCache.keySet()) {
            System.out.println("ID: " + id);
        }
    }

    // ============================================================
    // LinkedHashMap - Insertion order preserved (great for LRU cache)
    // ============================================================
    public Map<String, Patient> createLRUCache(int maxSize) {
        // Override removeEldestEntry to create LRU cache!
        return new LinkedHashMap<>(maxSize, 0.75f, true) { // true = access order
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, Patient> eldest) {
                return size() > maxSize;
            }
        };
    }

    // ============================================================
    // TreeMap - Sorted by key, O(log n)
    // ============================================================
    public void treeMapDemo() {

        // Appointments sorted by time (key = LocalTime)
        TreeMap<LocalTime, Appointment> schedule = new TreeMap<>();

        schedule.put(LocalTime.of(14, 0), new Appointment("DOC-1", "PT-1", LocalTime.of(14, 0)));
        schedule.put(LocalTime.of(9, 0),  new Appointment("DOC-1", "PT-2", LocalTime.of(9, 0)));
        schedule.put(LocalTime.of(11, 30), new Appointment("DOC-1", "PT-3", LocalTime.of(11, 30)));

        // Always sorted by time! ✅
        System.out.println(schedule.firstKey());  // 09:00
        System.out.println(schedule.lastKey());   // 14:00

        // Next appointment after 10:00
        LocalTime nextKey = schedule.higherKey(LocalTime.of(10, 0)); // 11:30

        // All appointments in the morning
        SortedMap<LocalTime, Appointment> morning = schedule.headMap(LocalTime.of(12, 0));
    }

    // ============================================================
    // ConcurrentHashMap - Thread-safe, high performance
    // ============================================================
    private final ConcurrentHashMap<String, Patient> activePatients =
        new ConcurrentHashMap<>();

    public void updatePatientStatus(String patientId, String status) {
        // Thread-safe atomic update!
        activePatients.computeIfPresent(patientId, (id, patient) -> {
            patient.setStatus(status);
            return patient;
        });
    }

    // ============================================================
    // Spring Boot: Real caching example
    // ============================================================
    @Service
    public class PatientCacheService {

        // Simple in-memory cache (production: use Redis/Caffeine)
        private final Map<String, Patient> cache = new ConcurrentHashMap<>();
        private final Map<String, LocalDateTime> cacheTimestamps = new ConcurrentHashMap<>();
        private static final Duration CACHE_TTL = Duration.ofMinutes(30);

        public Patient getPatient(String id) {
            // Check if cached and not expired
            if (cache.containsKey(id)) {
                LocalDateTime cached = cacheTimestamps.get(id);
                if (cached.plus(CACHE_TTL).isAfter(LocalDateTime.now())) {
                    return cache.get(id); // Cache hit!
                }
            }

            // Cache miss - fetch from DB
            Patient patient = patientRepository.findById(id).orElseThrow();
            cache.put(id, patient);
            cacheTimestamps.put(id, LocalDateTime.now());
            return patient;
        }

        public void evict(String id) {
            cache.remove(id);
            cacheTimestamps.remove(id);
        }

        @Autowired private PatientRepository patientRepository;
    }
}
```

---

## 4.4 Queue, PriorityQueue & Deque

```java
@Service
public class AppointmentSchedulingService {

    // ============================================================
    // PriorityQueue - Patients sorted by priority (min-heap)
    // ============================================================

    // Custom patient priority: EMERGENCY first, then by waiting time
    private final PriorityQueue<Patient> emergencyQueue = new PriorityQueue<>(
        Comparator.comparingInt(Patient::getPriorityLevel) // Lower number = higher priority
                  .thenComparing(Patient::getArrivalTime)  // FIFO within same priority
    );

    public void addPatient(Patient patient) {
        emergencyQueue.offer(patient); // O(log n)
    }

    public Patient treatNextPatient() {
        return emergencyQueue.poll(); // Returns highest priority, O(log n)
    }

    public Patient peekNextPatient() {
        return emergencyQueue.peek(); // Look without removing, O(1)
    }

    // ============================================================
    // ArrayDeque - Use as Queue OR Stack (modern replacement)
    // ============================================================
    public void arrayDequeDemo() {

        // As a Queue (FIFO - First In, First Out)
        Deque<Patient> waitingRoom = new ArrayDeque<>();

        // Enqueue
        waitingRoom.offerLast(new Patient("Ahmed", 35));   // Add to tail
        waitingRoom.offerLast(new Patient("Sara", 28));    // Add to tail
        waitingRoom.offerFirst(new Patient("Emergency", 60)); // Add to head

        // Dequeue
        Patient next = waitingRoom.pollFirst(); // Remove from head (FIFO)

        // As a Stack (LIFO - Last In, First Out)
        Deque<String> undoHistory = new ArrayDeque<>();

        // Push
        undoHistory.push("Created appointment");    // pushFirst()
        undoHistory.push("Updated patient info");   // pushFirst()
        undoHistory.push("Deleted old record");     // pushFirst()

        // Pop
        String lastAction = undoHistory.pop(); // "Deleted old record" (LIFO)

        // ✅ ArrayDeque is FASTER than Stack and LinkedList for both Queue and Stack!
    }

    // ============================================================
    // Spring Boot: Appointment Scheduler
    // ============================================================
    @Component
    public class HospitalAppointmentScheduler {

        private final PriorityQueue<Appointment> upcomingAppointments;

        public HospitalAppointmentScheduler() {
            // Sort by appointment time, then by priority
            this.upcomingAppointments = new PriorityQueue<>(
                Comparator.comparing(Appointment::getScheduledTime)
                          .thenComparingInt(Appointment::getPriority)
            );
        }

        public void schedule(Appointment appointment) {
            upcomingAppointments.offer(appointment);
            System.out.println("Scheduled: " + appointment.getPatientName() +
                             " at " + appointment.getScheduledTime());
        }

        public List<Appointment> getNextAppointments(int count) {
            List<Appointment> next = new ArrayList<>();
            // ⚠️ Don't poll from the original queue if you just want to view!
            // Copy first for viewing
            PriorityQueue<Appointment> copy = new PriorityQueue<>(upcomingAppointments);
            for (int i = 0; i < count && !copy.isEmpty(); i++) {
                next.add(copy.poll());
            }
            return next;
        }

        @Scheduled(fixedDelay = 60000) // Run every minute
        public void processNextAppointment() {
            Appointment next = upcomingAppointments.peek();
            if (next != null && next.getScheduledTime().isBefore(LocalDateTime.now().plusMinutes(5))) {
                Appointment apt = upcomingAppointments.poll();
                notifyDoctorAndPatient(apt);
            }
        }

        private void notifyDoctorAndPatient(Appointment apt) {
            System.out.println("Appointment starting soon: " + apt.getPatientName());
        }
    }
}
```

---

## 4.5 Comparator vs Comparable

```java
// ============================================================
// Comparable - Natural ordering (ONE default sort)
// "I know how to compare myself"
// ============================================================
public class Patient implements Comparable<Patient> {

    private String id;
    private String name;
    private int age;
    private int priorityLevel; // 1=Emergency, 2=Urgent, 3=Normal

    // Natural ordering: by priority first, then name
    @Override
    public int compareTo(Patient other) {
        int priorityCompare = Integer.compare(this.priorityLevel, other.priorityLevel);
        if (priorityCompare != 0) return priorityCompare;
        return this.name.compareTo(other.name);
    }
}

// ============================================================
// Comparator - External, flexible sorting (MANY sort options)
// "Someone else knows how to sort me"
// ============================================================
public class PatientComparators {

    // Multiple independent sort strategies
    public static final Comparator<Patient> BY_NAME =
        Comparator.comparing(Patient::getName);

    public static final Comparator<Patient> BY_AGE =
        Comparator.comparingInt(Patient::getAge);

    public static final Comparator<Patient> BY_PRIORITY =
        Comparator.comparingInt(Patient::getPriorityLevel);

    public static final Comparator<Patient> BY_PRIORITY_THEN_NAME =
        Comparator.comparingInt(Patient::getPriorityLevel)
                  .thenComparing(Patient::getName);

    public static final Comparator<Patient> OLDEST_FIRST =
        Comparator.comparingInt(Patient::getAge).reversed();
}

// Usage:
// Collections.sort(patients);                                   // Uses Comparable
// patients.sort(PatientComparators.BY_PRIORITY_THEN_NAME);      // Uses Comparator
// patients.stream().sorted(PatientComparators.OLDEST_FIRST)...  // In streams
// new TreeSet<>(PatientComparators.BY_PRIORITY)                 // In TreeSet
// new PriorityQueue<>(PatientComparators.BY_PRIORITY)           // In PriorityQueue
```

---

## 4.6 Iterator & Fail-Fast vs Fail-Safe

```java
public class IteratorDemo {

    public void iteratorExamples(List<Patient> patients) {

        // ✅ Iterator - manual control, safe removal during iteration
        Iterator<Patient> it = patients.iterator();
        while (it.hasNext()) {
            Patient p = it.next();
            if (p.isDischargedMoreThan30DaysAgo()) {
                it.remove(); // ✅ Safe removal during iteration!
            }
        }

        // ✅ ListIterator - bidirectional, can add/set during iteration
        ListIterator<Patient> lit = patients.listIterator();
        while (lit.hasNext()) {
            Patient p = lit.next();
            if (p.needsFollowUp()) {
                Patient updated = updatePatient(p);
                lit.set(updated); // Replace current element ✅
            }
        }

        // ❌ FAIL-FAST: ConcurrentModificationException!
        for (Patient p : patients) {
            if (p.isDiscovered()) {
                patients.remove(p); // 💀 ConcurrentModificationException!
            }
        }

        // ✅ Fail-Safe: CopyOnWriteArrayList (thread-safe, iteration safe)
        List<Patient> safeList = new CopyOnWriteArrayList<>(patients);
        for (Patient p : safeList) {
            if (p.isDiscovered()) {
                safeList.remove(p); // ✅ Works! Iterates over a snapshot
            }
        }
        // ⚠️ CopyOnWriteArrayList creates copy on every write - expensive for frequent writes!

        // ✅ Modern: removeIf (cleanest approach)
        patients.removeIf(Patient::isDischargedMoreThan30DaysAgo);
    }

    private Patient updatePatient(Patient p) { return p; }
}
```

---

# 📊 Master Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────────────┐
│              JAVA COLLECTIONS - WHEN TO USE WHAT                        │
├──────────────────┬─────────────────────────────────────────────────────┤
│   Need           │   Use                                               │
├──────────────────┼─────────────────────────────────────────────────────┤
│ Ordered list     │ ArrayList (default), LinkedList (queue pattern)     │
│ No duplicates    │ HashSet (fast), LinkedHashSet (ordered), TreeSet (sorted) │
│ Key-value pairs  │ HashMap (default), LinkedHashMap (ordered), TreeMap (sorted) │
│ Thread-safe list │ CopyOnWriteArrayList (read-heavy), Collections.synchronizedList │
│ Thread-safe map  │ ConcurrentHashMap (always prefer over Hashtable)   │
│ Priority queue   │ PriorityQueue (min-heap by default)                │
│ Stack/Deque      │ ArrayDeque (faster than Stack, LinkedList)          │
│ Null-safe value  │ Optional<T>                                         │
│ Sorted iteration │ TreeSet / TreeMap                                   │
│ Enum keys/values │ EnumMap / EnumSet (most efficient for enums)       │
└──────────────────┴─────────────────────────────────────────────────────┘

Big O Summary:
┌──────────────┬──────────┬────────────┬──────────┬────────────┬──────────┐
│ Operation    │ArrayList │ LinkedList │ HashSet  │  TreeSet   │ HashMap  │
├──────────────┼──────────┼────────────┼──────────┼────────────┼──────────┤
│ get/contains │  O(1)    │   O(n)     │  O(1)    │  O(log n)  │  O(1)   │
│ add (end)    │  O(1)*   │   O(1)     │  O(1)    │  O(log n)  │  O(1)   │
│ add (start)  │  O(n)    │   O(1)     │  O(1)    │  O(log n)  │  O(1)   │
│ remove       │  O(n)    │   O(1)**   │  O(1)    │  O(log n)  │  O(1)   │
│ sort         │ O(n log n)│ O(n log n)│    -     │     -      │    -    │
└──────────────┴──────────┴────────────┴──────────┴────────────┴──────────┘
*amortized  **if you have the node
```

---

# 🎯 Top Interview Questions

```
1. ArrayList vs LinkedList - when to use each? (Performance tradeoffs)
2. How does HashMap work internally? (hashing, buckets, collision)
3. What is the difference between == and .equals() for Strings?
4. Explain fail-fast vs fail-safe iterators with examples
5. When would you use ConcurrentHashMap over HashMap?
6. What is the contract between equals() and hashCode()?
7. Explain lambda and functional interfaces with examples
8. What is Optional and how do you use it properly?
9. Difference between abstract class and interface?
10. Explain SOLID principles with real examples
11. How does Stream API differ from a for-loop? (lazy evaluation)
12. What is autoboxing and when can it cause bugs?
13. Comparable vs Comparator - when to use each?
14. What is the String Pool and why does it matter?
15. Explain PriorityQueue and give a real-world use case
```

---

هذا هو المنهج الكامل! كل موضوع اتشرح بـ:

- 🔴 المشكلة
- 💡 المفهوم والتشبيه
- ✅ كود واقعي (hospital/bank/e-commerce)
- ⚡ الأداء
- ❌ الأخطاء الشائعة
- 🔗 ربط بـ Spring Boot

تريد أعمق في موضوع معين؟ أو نبدأ بتمارين عملية على أي section؟ 🚀