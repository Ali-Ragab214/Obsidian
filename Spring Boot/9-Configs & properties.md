

---

# 🔷 application.properties vs application.yml في Spring Boot

---

# 🔹 أولًا: الفكرة العامة

## 📌 Definition (EN)

**Configuration Files:**  
Files used to define application settings such as server port, database connection, logging, etc.

👉 في Spring Boot عندك طريقتين تكتب بيهم الـ config:

- `application.properties`
    
- `application.yml`
    

---

# 🔷 1. application.properties

## 📌 Definition (EN)

A **key-value pair based configuration file** where each property is defined as `key=value`.

---

## 🔹 Example

```properties
server.port=8081

spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=1234

spring.jpa.hibernate.ddl-auto=update
```

---

## 🔹 الشكل العام

```id="n5c7cb"
key=value
key=value
key=value
```

---

## 🔹 مميزاته

✔️ سهل وبسيط  
✔️ مناسب للبدايات  
✔️ واضح لأي حد

---

## 🔹 عيوبه

❌ صعب مع البيانات الكبيرة (nested)  
❌ مفيش تنظيم Hierarchy واضح

---

# 🔷 2. application.yml (YAML)

## 📌 Definition (EN)

A **hierarchical configuration format** that uses indentation (spaces) to represent nested structure.

---

## 🔹 نفس المثال بـ YAML

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: 1234

  jpa:
    hibernate:
      ddl-auto: update
```

---

## 🔹 الشكل العام

```id="nccz1c"
parent:
  child:
    key: value
```

---

## 🔹 مميزاته

✔️ Organized جدًا  
✔️ مناسب للـ nested config  
✔️ بيقلل التكرار

---

## 🔹 عيوبه

❌ حساس جدًا للمسافات (indentation) 😅  
❌ لو غلطت في space → error

---

# 🔥 مقارنة مباشرة

|Feature|properties|yaml|
|---|---|---|
|الشكل|key=value|hierarchy|
|القراءة|سهلة|أسهل في المشاريع الكبيرة|
|التعقيد|ضعيف|قوي|
|المسافات|مش مهمة|مهمة جدًا|
|nested data|صعب|سهل جدًا|

---

# 🔷 Diagram يوضح الفرق 🔥

## properties

```id="eq7xtg"
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
```

---

## yaml

```id="l5fjqk"
spring
 └── datasource
      ├── url
      ├── username
      └── password
```

---

# 🔷 Spring بيقرأهم إزاي؟

```id="v94tp4"
Spring Boot
     |
     v
Reads Config File
     |
     v
Creates Beans & Settings
```

---

# 🔴 مهم جدًا

❗ مينفعش تستخدم الاتنين مع بعض بنفس الوقت  
لو الاتنين موجودين:

👉 Spring بي prioritize:

```id="4y0t6z"
application.yml > application.properties
```

---

# 🔷 Example واقعي (Binding مع كلاس)

## YAML

```yaml
app:
  name: MyApp
  version: 1.0
```

---

## Java Class

```java
@Component
@ConfigurationProperties(prefix = "app")
class AppConfig {
    private String name;
    private double version;

    // getters & setters
}
```

---

# 🔥 Flow كامل

```id="scc68k"
application.yml
      |
      v
Spring Environment
      |
      v
@ConfigurationProperties
      |
      v
Java Object (Bean)
```

---

# 🔥 إمتى أستخدم كل واحد؟

## ✅ استخدم properties لما:

- المشروع صغير
    
- config بسيط
    

## ✅ استخدم YAML لما:

- project كبير 🔥
    
- فيه nested config
    
- عايز readability أحسن
    

---

# 💡 الخلاصة التقيلة

- الاتنين نفس الوظيفة ✅
    
- الفرق في الشكل والتنظيم
    
- YAML أقوى في المشاريع الكبيرة
    
- properties أسهل في البداية
    

---

لو عايز نكمل 🔥  
أشرحلك:

- Profiles (dev / prod)
    
- أو Configuration Binding بشكل احترافي
    
- أو نعمل Project Spring Boot كامل
    

قولّي التوبيك اللي بعده 💪