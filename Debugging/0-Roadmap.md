بص، دي من أكتر الحاجات اللي الشركات بتبص عليها، لأن أي Software Engineer بيقضي جزء كبير من يومه **مش بيكتب كود... بيصلح Bugs**.

لو دخلت Interview وسألوك عن Debugging، فالمفروض تبقى عارف الحاجات دي:

---

# 1. يعني إيه Debugging؟

الإجابة المثالية:

> Debugging is the process of identifying, analyzing, and fixing bugs or unexpected behavior in an application.

---

# 2. الفرق بين Bug و Error و Exception

- **Bug**: مشكلة في الكود بسبب Logic غلط.
    
- **Error**: أي مشكلة تمنع البرنامج يشتغل أو تؤدي لسلوك غير صحيح.
    
- **Exception**: Runtime Error تقدر تتعامل معاه في الكود.
    

مثال:

```java
int x = 10 / 0;
```

ده Exception.

لكن:

```java
if(age > 18)
```

والمفروض

```java
if(age >= 18)
```

ده Bug.

---

# 3. إزاي تعمل Debug؟

المفروض تقول:

1. Reproduce the issue.
    
2. Read logs.
    
3. Analyze stack trace.
    
4. Use breakpoints.
    
5. Inspect variables.
    
6. Find the root cause.
    
7. Fix the issue.
    
8. Test again.
    

وده ترتيب محترف.

---

# 4. يعني إيه Breakpoint؟

دي أكتر حاجة بيسألوا عليها.

Breakpoint يعني:

> نقطة توقف للبرنامج.

مثلاً

```java
int x = 10;

int y = x * 5;

System.out.println(y);
```

لو حطيت Breakpoint على السطر التاني

البرنامج هيقف هناك.

وساعتها تقدر تشوف

```
x = 10
```

قبل ما يكمل.

---

# 5. أهم Debug Actions

لازم تعرفهم في IntelliJ أو VS Code.

### Step Over (F8)

ينفذ السطر الحالي.

---

### Step Into (F7)

يدخل جوه الميثود.

```java
calculate();
```

بدل ما ينفذها مرة واحدة

يدخل يشوف الكود بتاعها.

---

### Step Out

يطلع من الميثود الحالية.

---

### Resume

يكمل تشغيل البرنامج لحد Breakpoint تاني.

---

# 6. Watch

ممكن تراقب Variable.

مثلاً

```
balance
```

كل ما تتغير تشوف قيمتها.

---

# 7. Evaluate Expression

ميزة قوية جدًا.

أثناء الـ Debug

تقدر تكتب

```java
user.getAge()
```

ويحسبهولك.

أو

```java
orders.size()
```

من غير ما تغير الكود.

---

# 8. Stack Trace

من أهم أسئلة الانترفيو.

مثلاً

```
NullPointerException

at UserService.save()

at UserController.create()
```

ده معناه

الخطأ بدأ في

```
UserService.save()
```

واتنادى من

```
UserController.create()
```

لازم تعرف تقراه.

---

# 9. Logging

الفرق بين

```java
System.out.println()
```

و

```java
log.info()
```

و

```java
log.debug()
```

و

```java
log.error()
```

وليه بنستخدم Logging Framework زي SLF4J + Logback بدل `System.out.println()`.

---

# 10. Remote Debugging

لو سألك Senior.

يعني تعمل Debug لتطبيق شغال على Server.

مش لازم تعرفه بالتفصيل كـ Junior، لكن اعرف الفكرة.

---

# 11. Debugging في Spring Boot

لازم تعرف تدور على:

- Application Logs.
    
- Stack Trace.
    
- HTTP Request.
    
- HTTP Response.
    
- Exception Handler.
    
- SQL Queries.
    
- Hibernate Logs.
    
- Spring Security Logs.
    

---

# 12. Debugging Database

لو البيانات غلط.

بتعمل:

- تشوف الـ SQL.
    
- تشوف الـ Parameters.
    
- تتأكد من الـ Transaction.
    
- تتأكد إن البيانات اتحفظت.
    

---

# 13. Debugging APIs

لو الـ API رجعت

```
500
```

هتعمل إيه؟

الإجابة:

- أشوف الـ Logs.
    
- أشوف الـ Stack Trace.
    
- أحط Breakpoints.
    
- أتأكد من الـ Request.
    
- أتأكد من الـ Service.
    
- أتأكد من الـ Repository.
    
- أتأكد من الـ Database.
    

---

# 14. Common Interview Questions

ممكن يتسألك:

- What is debugging?
    
- What is a breakpoint?
    
- Difference between Step Into and Step Over?
    
- What is a stack trace?
    
- How do you debug a NullPointerException?
    
- How do you debug a REST API?
    
- How do you investigate a 500 Internal Server Error?
    
- Why use logging instead of `System.out.println()`?
    
- Difference between INFO, DEBUG, WARN, and ERROR logs?
    
- How do you debug a Spring Boot application?
    

---

## لو عايز تبقى قوي فعلًا

بما إنك Backend Java/Spring، أنصحك تذاكر Debugging بالترتيب ده:

1. **IntelliJ Debugger** (Breakpoints, Step Into, Step Over, Watches, Evaluate).
    
2. **Stack Trace** وازاي تقرأه.
    
3. **Logging (SLF4J + Logback)** ومستويات الـ Logs.
    
4. **Debugging Spring Boot** (Controllers, Services, Repositories).
    
5. **Debugging Hibernate & SQL**.
    
6. **Debugging REST APIs** باستخدام Postman + Logs.
    
7. **Remote Debugging** (كمفهوم).
    

لو أتقنت السبع نقاط دول، هتبقى جاهز لمعظم أسئلة الـ Debugging اللي بتتسأل في انترفيوهات الـ Java Backend، سواء Junior أو Mid-Level.