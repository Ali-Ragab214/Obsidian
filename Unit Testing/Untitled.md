# شرح كامل للـ Video Script – JUnit First Unit Test

خليني أشرح لك كل حاجة خطوة بخطوة يا علي 🚀

---

## 1️⃣ ليه Maven Project مش Java Project عادي؟

لما بتعمل unit tests، لازم يكون عندك **project structure معين**:

```
MyProject/
├── src/
│   ├── main/
│   │   └── java/        ← الكود الأساسي بتاعك
│   └── test/
│       └── java/        ← كود الـ Tests
└── pom.xml              ← ملف إعدادات Maven
```

لو عملت Java Project عادي، هتعمل الـ structure ده بإيدك. لو عملت **Maven Project**، بيتعمل **أوتوماتيك** ✅

---

## 2️⃣ الـ Group ID و Artifact ID إيه معناهم؟

```xml
<!-- في pom.xml -->
<groupId>com.in28minutes</groupId>      <!-- زي اسم الشركة/المشروع -->
<artifactId>JunitCourse</artifactId>    <!-- اسم الـ project نفسه -->
```

فكّرهم زي:

- **groupId** = اسم المنظمة (عكس domain: `com.ali`)
- **artifactId** = اسم الـ jar اللي هيتبني

---

## 3️⃣ الكود اللي هنعمله Testing – StringHelper

```java
package com.in28minutes.junit.helper;

public class StringHelper {

    // Method 1: بتشيل الـ A من أول مكانين بس
    public String truncateAInFirst2Positions(String str) {
        if (str.length() <= 2 && str.contains("A")) {
            return str.replace("A", "");
        }
        String first2 = str.substring(0, 2).replace("A", "");
        return first2 + str.substring(2);
    }

    // Method 2: هل أول حرفين = آخر حرفين؟
    public boolean areFirstAndLastTwoCharactersTheSame(String str) {
        if (str.length() < 2) return false;
        String first2 = str.substring(0, 2);
        String last2 = str.substring(str.length() - 2);
        return first2.equals(last2);
    }
}
```

### منطق `truncateAInFirst2Positions`:

|Input|أول 2 حروف|بعد الشيل|Output|
|---|---|---|---|
|`"AACD"`|`"AA"` → `""`|`+ "CD"`|`"CD"`|
|`"ACD"`|`"AC"` → `"C"`|`+ "D"`|`"CD"`|
|`"CDEF"`|`"CD"` → `"CD"`|`+ "EF"`|`"CDEF"`|
|`"CDAA"`|`"CD"` → `"CD"`|`+ "AA"`|`"CDAA"`|

> الـ A في المكان 3 و4 **مش بتتشال** لأن الميثود شايلة من أول 2 بس ✅

---

## 4️⃣ ليه الـ Test في `src/test/java` مش `src/main/java`؟

ده مبدأ مهم جداً:

```
❌ غلط: تحط Test code في src/main/java
   → الـ tests هتتبعت مع الـ production code للـ users!

✅ صح: src/test/java
   → Maven بيشيل الـ test files تلقائياً من الـ build النهائي
```

---

## 5️⃣ إنشاء الـ Test Class

```java
package com.in28minutes.junit.helper;  // نفس الـ package!

import org.junit.Test;
import static org.junit.Assert.*;

public class StringHelperTest {  // اسم الـ class + "Test"

    StringHelper helper = new StringHelper();

    @Test
    public void testTruncateAInFirst2Positions_AinFirst2() {
        // AACD → CD
        assertEquals("CD", helper.truncateAInFirst2Positions("AACD"));
    }

    @Test
    public void testTruncateAInFirst2Positions_AinFirst1() {
        // ACD → CD
        assertEquals("CD", helper.truncateAInFirst2Positions("ACD"));
    }

    @Test
    public void testTruncateAInFirst2Positions_NoAInFirst2() {
        // CDEF → CDEF (مفيش A خالص)
        assertEquals("CDEF", helper.truncateAInFirst2Positions("CDEF"));
    }

    @Test
    public void testTruncateAInFirst2Positions_ANotInFirst2() {
        // CDAA → CDAA (الـ A في الآخر مش في الأول)
        assertEquals("CDAA", helper.truncateAInFirst2Positions("CDAA"));
    }
}
```

---

## 6️⃣ الـ JUnit 4 Dependency في pom.xml

الطريقة الصح (مش الـ shortcut اللي عمله في الفيديو):

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>   <!-- مهم! بيخلي الـ lib دي للـ tests بس -->
    </dependency>
</dependencies>
```

> `scope=test` يعني الـ JUnit مش هتتضمن في الـ production build ✅

---

## 7️⃣ الـ Red Bar vs Green Bar

```
🔴 Red Bar  = في test فاشل
🟢 Green Bar = كل الـ tests نجحوا
```

في الفيديو ظهر **Red Bar** لأن الـ test method فاضية:

```java
@Test
public void test() {
    fail("Not yet implemented");  // ← ده بيعمل fail تلقائي
}
```

---

## 8️⃣ مبادئ Unit Testing الأساسية

```
1. Fast      → بتتشغل في milliseconds
2. Isolated  → مش بتعتمد على DB أو Network
3. Repeatable → نفس النتيجة كل مرة
4. Self-Checking → بتحدد هي نفسها Pass أو Fail
5. Timely    → بتتكتب مع الكود (أو قبله في TDD)
```

---

## الخلاصة بالترتيب 🗺️

```
1. File → New → Maven Project
2. حدد groupId + artifactId
3. حط الكود في src/main/java
4. أضف JUnit dependency في pom.xml
5. في src/test/java → New JUnit Test Case
6. اكتب @Test methods
7. Run as JUnit Test → 🟢 Green Bar
```

---

**ملاحظة للـ Spring Boot** يا علي: لما تشتغل على Spring Boot هتلاقي `spring-boot-starter-test` بيجيب JUnit 5 + Mockito + AssertJ تلقائياً، وده أقوى بكتير من اللي بيشرحه هنا 💪


```java 
public boolean isFirstAndLastTwoCharsTheSame(String str)
{
if(str.length() < 2) return false ;
String firstTwo = str.substring(0,2);
String LastTwo = str.substring(str.Length() -2 )
return firstTwo.equals(LastTwo);
}
```