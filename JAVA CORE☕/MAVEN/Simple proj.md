
directory structure 
![[Pasted image 20260304151502.png]]
تمام يا علي 👨‍💻🔥  
خلينا نفك الـ `pom.xml` ده سطر سطر كأنك في انترفيو Maven 💪

---

# أولًا: يعني إيه `pom.xml` ؟

POM = Project Object Model

ده ملف الإعدادات الأساسي بتاع  
Apache Maven

هو اللي بيقول لـ Maven:

- اسم المشروع
    
- النسخة
    
- نوع الباكدج
    
- dependencies
    
- plugins
    
- إعدادات البناء
    

---

# نشرح الملف بتاعك 👇

---

## 1️⃣ الجزء ده

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/maven-v4_0_0.xsd">
```

ده بيحدد:

- إن ده POM version 4
    
- بيحدد الـ schema اللي Maven يفهم بيه الملف
    

مش بتغير فيه حاجة غالبًا.

---

## 2️⃣

```xml
<modelVersion>4.0.0</modelVersion>
```

نسخة نموذج الـ POM  
حاليًا 4.0.0 هي المستخدمة دايمًا.

---

## 3️⃣ تعريف المشروع

```xml
<groupId>gov.iti.jets</groupId>
<artifactId>hello-world</artifactId>
<packaging>jar</packaging>
<version>1.0-SNAPSHOT</version>
<name>hello-world</name>
```

### 🔹 groupId

ده زي اسم الشركة / الجهة المالكة  
عندك:

```
gov.iti.jets
```

ده معناه مشروع تابع لـ ITI غالبًا 👀

---

### 🔹 artifactId

اسم المشروع نفسه:

```
hello-world
```

---

### 🔹 version

```
1.0-SNAPSHOT
```

معناها:

- نسخة تطوير
    
- مش نسخة نهائية
    

لو كانت نسخة رسمية تبقى:

```
1.0
```

---

### 🔹 packaging

```
jar
```

يعني بعد ما تعمل:

```bash
mvn package
```

هيطلعلك ملف:

```
hello-world-1.0-SNAPSHOT.jar
```

---

# 4️⃣ الجزء المهم 🔥 (Plugins)

```xml
<build>
  <plugins>
```

ده الجزء اللي بيحدد أدوات إضافية أثناء البناء.

---

## الـ Plugin اللي ضيفته

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <mainClass>gov.iti.jets.App</mainClass>
    </configuration>
</plugin>
```

---

### 🔹 exec-maven-plugin

ده Plugin رسمي من:

MojoHaus

وظيفته:  
👉 يخليك تشغل main method مباشرة من Maven

يعني بدل ما تكتب:

```bash
java -cp target/classes gov.iti.jets.App
```

تكتب بس:

```bash
mvn exec:java
```

---

### 🔹 mainClass

```xml
<mainClass>gov.iti.jets.App</mainClass>
```

ده بيقول لمين يبدأ التنفيذ.

لازم:

- يكون عندك class اسمه `App`
    
- جوه package اسمها `gov.iti.jets`
    
- فيه `public static void main`
    

---

# 5️⃣ properties

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

ده بيحدد ترميز الملفات  
عشان الحروف العربي متبوظش 👌

---

# الشكل النهائي بعد ما تعمل build

```
hello-world
 ├── src
 │   └── main
 │       └── java
 │           └── gov
 │               └── iti
 │                   └── jets
 │                       └── App.java
 ├── target
 │   └── classes
 └── pom.xml
```

---

# ملخص احترافي جدًا 🔥

|الجزء|وظيفته|
|---|---|
|groupId|اسم الجهة|
|artifactId|اسم المشروع|
|version|نسخة المشروع|
|packaging|نوع الناتج (jar)|
|build|إعدادات البناء|
|plugins|أدوات إضافية|
|exec-maven-plugin|تشغيل main من Maven|

---

لو عايز أفهمك بقى دورة حياة Maven lifecycle  
(compile → test → package → install → deploy)  
بطريقة تخليك تفهم أي مشروع Spring بعد كده 👌