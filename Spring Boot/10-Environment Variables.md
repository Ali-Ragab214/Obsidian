

---

# 🧠 يعني ايه Environment Variables؟
**Environment Variables (متغيرات البيئة)**  
هي متغيرات بنخزن فيها إعدادات أو قيم خارج الكود، علشان نقدر نغيّرها حسب البيئة (Environment) من غير ما نعدل الكود نفسه.
---

## 🎯 ليه نستخدمها؟

بدل ما تعمل كده ❌

```java
String dbPassword = "123456";
```

تعمل كده ✅

```java
String dbPassword = System.getenv("DB_PASSWORD");
```

---

# 🔥 الفكرة الأساسية

✔ تخلي الكود:

- آمن 🔐
    
- مرن 🔄
    
- سهل التغيير بدون تعديل الكود
    

---

# 🏗️ استخدام Environment Variables في Java

## ✅ الطريقة الأساسية

```java
String port = System.getenv("PORT");
```

---

# 🚀 في Spring Boot (الأهم ليك)

Spring Boot بيسهّل الموضوع جدًا 👌

---

## ✅ 1. في application.properties

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
```
${take the value from env variable}
👉 `${}` معناها:  
خد القيمة من Environment Variables


---

## ✅ 3. باستخدام application.yml

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASS}
```

---

# 💡 Default Value (مهم جدًا)

```properties
server.port=${PORT:8080}
```

👉 لو `PORT` مش موجود → يستخدم 8080

---

# 🧠 استخدام @Value

```java
@Value("${DB_USER}")
private String username;
```

---

# 🧠 استخدام @ConfigurationProperties (احترافي)

```java
@ConfigurationProperties(prefix = "app")
public class AppConfig {
    private String name;
}
```

وفي properties:

```properties
app.name=${APP_NAME}
```

---

# 🔐 Use Cases حقيقية

## ✅ 1. Database Credentials

- username
    
- password
    

---

## ✅ 2. API Keys

```properties
api.key=${API_KEY}
```

---

## ✅ 3. Ports

```properties
server.port=${PORT:8080}
```

---

## ✅ 4. Environments

```properties
spring.profiles.active=${ENV:dev}
```

---

# ⚠️ أخطاء شائعة

## ❌ تحط secrets في الكود

غلط كبير جدًا

---

## ❌ تنسى default value

ممكن التطبيق يقع

---

## ❌ تعتمد على env مش موجود

يعمل crash

---

# 🔥 Production vs Development

|Environment|القيم|
|---|---|
|dev|local DB|
|prod|real DB|
|test|test DB|

---

# 🚀 ربط الكلام بشغلك

في مشروعك (Services App):

هتستخدم env variables في:

- DB connection
    
- JWT Secret
    
- Payment keys
    
- Email config
    

---

# 💣 مثال Real كامل

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}

jwt.secret=${JWT_SECRET}

server.port=${PORT:8080}
```

---

# 🧠 ليه الشركات بتحبها؟

✔ Security  
✔ Flexibility  
✔ CI/CD friendly  
✔ Docker / Kubernetes compatible

---

# 🔥 مستوى أعلى شوية

لو ناوي تبقى جامد فعلًا 👇

- استخدم `.env` files
    
- استخدم Docker secrets
    
- استخدم Vault (زي HashiCorp Vault)
    

---

# 💣 سؤال انترفيو

ليه Environment Variables أحسن من:

```properties
application.properties
```

في الـ production؟ 🤔

---
اازاي بقا استخدمها واعملها 




---

# 🚀 الهدف

تشغل Spring Boot باستخدام **Environment Variables** بدل ما تحط الداتا في الكود

---

# 🟢 STEP 1: جهز `application.properties`

افتح الملف:

```id="a1"
src/main/resources/application.properties
```

وخليه كده:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
```

---

# 🟢 STEP 2: افتح Run Configuration

في IntelliJ:

👉 من فوق اضغط:

```
Run → Edit Configurations
```

---

# 🟢 STEP 3: اعمل Configuration جديد

لو مفيش واحد:

1. اضغط ➕
    
2. اختار **Spring Boot**
    
3. اختار الـ Main class (اللي فيه `@SpringBootApplication`)
    

---

# 🟢 STEP 4: فعل Environment Variables

📍 دي أهم خطوة:

1. اضغط:  
    👉 **Modify options**
    
2. فعل:  
    👉 ✅ **Environment variables**
    

---

# 🟢 STEP 5: ضيف القيم

هتظهرلك خانة اسمها:

👉 **Environment variables**
حط بقا الحاجات المهمه هنا زي ال Api Keys  وخلافه من الداتا الحساسه

اضغط Edit وحط:

```bash
DB_USER=root;DB_PASS=1234;DB_URL=jdbc:mysql://localhost:3306/mydb
```

---

# 🟢 STEP 6: شغل البرنامج

اضغط:

👉 Run 🚀

---

# 🧠 STEP 7: اتأكد إنه شغال

ممكن تجرب تطبع:

```java
System.out.println(System.getenv("DB_USER"));
```

لو طلع `root` → كده تمام 🔥

---

# ⚠️ حاجات مهمة جدًا

## ❗ 1. لو غيرت القيم

لازم:  
👉 Restart للـ app

---

## ❗ 2. متحطش القيم في الكود

❌ غلط:

```properties
spring.datasource.password=1234
```

✅ صح:

```properties
spring.datasource.password=${DB_PASS}
```

---

## ❗ 3. لو مش شغال

راجع:

- كتبت الاسم صح؟ (DB_USER)
    
- فعلت Environment variables؟
    
- عملت Restart؟
    

---

# 🧠 الصورة النهائية

```id="a7"
IntelliJ Env Variables
        ↓
Spring Boot (${DB_USER})
        ↓
Application شغال 🔥
```

---

# 💥 كده انت وصلت لمستوى الشركات 👏

لو عايز نكمل:

- نعمل **dev / prod profiles**
    
- أو نستخدمها مع **Docker**
    

قولّي وهننقل لليفل أعلى 🔥