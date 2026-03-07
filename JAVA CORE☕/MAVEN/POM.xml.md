is the central piece of information 
شايل كل المعلومات  فيه كل المعلومات الخاص بكل حاجه في البروجكت  

# شرح POM في Maven

## إيه هو الـ POM؟

الـ **POM** اختصار لـ **Project Object Model**، وهو ملف XML اسمه `pom.xml` بيكون في root directory لأي مشروع Maven.

ببساطة: هو **ملف تعريفي للمشروع** بيقول لـ Maven كل حاجة عن المشروع ده.

---

## العناصر الأساسية اللي في الصورة

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>gov.iti.jets</groupId>
    <artifactId>hello-world</artifactId>
    <version>1.0-SNAPSHOT</version>
</project>
```

|العنصر|معناه|المثال|
|---|---|---|
|`modelVersion`|إصدار ملف الـ POM نفسه (دايماً 4.0.0)|`4.0.0`|
|`groupId`|اسم المؤسسة أو الشركة (زي package name)|`gov.iti.jets`|
|`artifactId`|اسم المشروع نفسه|`hello-world`|
|`version`|إصدار المشروع|`1.0-SNAPSHOT`|

> الـ **SNAPSHOT** معناها إن المشروع لسه في مرحلة تطوير، مش إصدار نهائي.

---

## Maven, JDK9+, and JPMS

دي 3 حاجات مختلفة بس بتتشغل مع بعض:

### 1. Maven 🏗️

نظام بناء (Build Tool) مهمته:

- تحميل الـ dependencies تلقائياً
- بناء المشروع (compile → test → package)
- إدارة الـ `pom.xml`

### 2. JDK 9+ ☕

من Java 9 فأكثر، اتضافت فيها فيتشر مهمة جداً اسمها **JPMS**

### 3. JPMS (Java Platform Module System) 📦

اسمها كمان **Project Jigsaw**، وهي نظام الـ **Modules** في Java.

**قبل Java 9:** الكود كله كان في classpath وكل حاجة تشوف حاجة.

**بعد Java 9:** بقى فيه ملف اسمه `module-info.java` بتحدد فيه:

- إيه اللي هيـ **export** (يتشاف من برا)
- إيه اللي هيـ **require** (dependencies على modules تانية)

```java
// module-info.java
module gov.iti.jets.helloworld {
    requires java.base;
    exports gov.iti.jets.helloworld;
}
```

---

## العلاقة بين التلاتة

```
Maven (يدير المشروع)
  └── pom.xml (يعرّف المشروع)
        └── يستخدم JDK 9+
              └── اللي فيه JPMS (نظام الـ modules)
```

Maven بدأ يدعم JPMS من إصدارات معينة، وبيتعامل مع `module-info.java` تلقائياً لو موجود في المشروع.




# الفرق بين الـ 3 أنواع

## 1. Super POM 👴

- هو الـ **parent الأساسي** لكل مشاريع Maven
- موجود **جوه Maven نفسه** (مش بتكتبه إنت)
- بيحتوي على **الإعدادات الافتراضية** كلها زي:
    - مكان الـ source code: `src/main/java`
    - مكان الـ tests: `src/test/java`
    - مكان الـ output: `target`

---

## 2. POM 📄

- ده اللي **إنت بتكتبه** في مشروعك
- بيحتوي بس على اللي **إنت محتاجه** وعايز تخصصه
- هو الـ `pom.xml` العادي اللي شفناه

---

## 3. Effective POM ⚡

- هو **الناتج النهائي** = Super POM + POM بتاعك مع بعض
- Maven بيعمله في الذاكرة وقت البناء
- تقدر تشوفه بالأمر:

```bash
mvn help:effective-pom
```

---

## تلخيص سريع 🎯

```
Super POM  (افتراضيات Maven)
    +
   POM      (اللي إنت كتبته)
    =
Effective POM (اللي Maven بيشتغل بيه فعلاً)
```

> ببساطة: إنت بتكتب الـ POM، Maven بيدمجه مع الـ Super POM، والناتج هو الـ Effective POM.



#POM STRUCTURE 
project information معظمه optional

![[Pasted image 20260304145532.png]]
mvn help:effective-pom => this command عشان اعرض ال super pom 