جميل، الـ **JDBC** من أهم المواضيع في Java، وبيجي في الانترفيوهات كتير جدًا، سواء Junior أو Mid-Level. هشرحلك من الصفر لحد مستوى الانترفيو.

---

# أولًا: يعني إيه JDBC؟

JDBC اختصار لـ

> **Java Database Connectivity**

وهي عبارة عن **API** في Java بتسمح للبرنامج إنه يتعامل مع قواعد البيانات.

بمعنى أبسط...

قبل JDBC

```
Java Program

        ❌

Database
```

الاتنين مش فاهمين بعض.

لازم يبقى فيه وسيط.

JDBC هو الوسيط ده.

```
Java Program
      |
      |
    JDBC
      |
      |
Database Driver
      |
Database
```

---

# طب هو JDBC عبارة عن إيه؟

هو مجموعة Interfaces و Classes موجودة في الـ JDK.

زي

```
Connection

Statement

PreparedStatement

CallableStatement

ResultSet

DriverManager
```

كل دول جزء من JDBC API.

---

# طب مين بينفذ الـ Interfaces دي؟

مثلا

```
Connection
```

ده Interface.

مين بيعمل Implementation؟

الـ Driver بتاع قاعدة البيانات.

مثلا

MySQL

عندها Driver

```
mysql-connector-j
```

هو اللي بيعمل Implementation للـ JDBC Interfaces.

---

# العلاقة بينهم

```
Application

↓

JDBC API

↓

JDBC Driver

↓

Database
```

---

# يعني إيه Driver؟

الـ Driver عبارة عن مكتبة.

مثلا لو هتتعامل مع MySQL

بتضيف dependency

```
mysql-connector-j
```

لو PostgreSQL

```
postgresql
```

لو Oracle

```
ojdbc
```

كل Driver بيعرف Java تتكلم مع قاعدة البيانات دي.

---

# JDBC Architecture

```
Java Application

↓

JDBC API

↓

DriverManager

↓

JDBC Driver

↓

Database
```

---

# أشهر Classes

## DriverManager

مسؤول عن إنشاء Connection.

مثال

```java
Connection connection =
DriverManager.getConnection(url, username, password);
```

هو اللي بيروح يجيبلك Connection.

---

## Connection

بتمثل الاتصال الحقيقي بقاعدة البيانات.

زي كأنك عملت Login على Database.

من خلالها تقدر

- تعمل Query
    
- تعمل Insert
    
- تعمل Update
    
- تعمل Delete
    

---

## Statement

دي أول طريقة لتنفيذ SQL.

```java
Statement stmt = connection.createStatement();

ResultSet rs =
stmt.executeQuery("SELECT * FROM users");
```

---

## ليه Statement مش كويسة؟

لو عملت

```java
String username = request.getParameter("username");
```

ثم

```java
String sql =
"SELECT * FROM users WHERE username='" + username + "'";
```

لو المستخدم كتب

```
' OR 1=1 --
```

هيبقى

```sql
SELECT * FROM users
WHERE username=''
OR 1=1
```

وده اسمه

**SQL Injection**

---

# الحل؟

PreparedStatement

وده أهم Class في JDBC.

---

# PreparedStatement

بدل

```java
String sql =
"SELECT * FROM users WHERE id = " + id;
```

بنكتب

```java
PreparedStatement ps =
connection.prepareStatement(
"SELECT * FROM users WHERE id=?");
```

بعدها

```java
ps.setInt(1, id);
```

ثم

```java
ResultSet rs =
ps.executeQuery();
```

---

# ليه PreparedStatement أفضل؟

1. يمنع SQL Injection ✅
    
2. أسرع لو نفس الـ Query هتتنفذ أكثر من مرة (لأن قاعدة البيانات قد تعيد استخدام خطة التنفيذ).
    
3. الكود أنضف.
    

---

# ResultSet

ده بيمثل البيانات الراجعة.

مثلا

```
id

name

email
```

بيكونوا جوه ResultSet.

مثال

```java
while(rs.next()){
    System.out.println(rs.getInt("id"));
    System.out.println(rs.getString("name"));
}
```

---

# executeQuery()

بتستخدم مع

```
SELECT
```

وترجع

```
ResultSet
```

---

# executeUpdate()

بتستخدم مع

```
INSERT

UPDATE

DELETE
```

وترجع

```
عدد الصفوف المتأثرة
```

مثلا

```
3
```

يعني عدلت 3 Rows.

---

# execute()

بتستخدم لو مش عارف نوع الـ Query.

ترجع

```
boolean
```

---

# خطوات JDBC

```
1- Load Driver

↓

2- Open Connection

↓

3- Create Statement

↓

4- Execute Query

↓

5- Read ResultSet

↓

6- Close Resources
```

---

# مثال كامل

```java
Connection con =
DriverManager.getConnection(url,user,pass);

PreparedStatement ps =
con.prepareStatement(
"SELECT * FROM users WHERE id=?");

ps.setInt(1,1);

ResultSet rs=ps.executeQuery();

while(rs.next()){

System.out.println(rs.getString("name"));

}

rs.close();

ps.close();

con.close();
```

---

# ليه لازم نقفل الموارد؟

لأن الـ Connection غالي جدًا.

لو مفتوح

هيستهلك

- Memory
    
- Database Connection
    
- Network Resources
    

---

# الأفضل

```java
try(
Connection con=...

PreparedStatement ps=...

){

}
```

وده

**try-with-resources**

هيقفل كل حاجة تلقائيًا.

---

# JDBC في Spring

أنت غالبًا مش بتستخدم JDBC مباشرة.

بتستخدم

```
Spring Data JPA
```

اللي بيستخدم

```
Hibernate
```

والـ Hibernate في النهاية بيستخدم JDBC للتواصل مع قاعدة البيانات.

يعني السلسلة كده:

```
Spring Data JPA
        ↓
Hibernate
        ↓
JDBC
        ↓
Database Driver
        ↓
Database
```

---

# أشهر أسئلة الانترفيو

## 1. What is JDBC?

واجهة (API) في Java للتعامل مع قواعد البيانات، توفر Interfaces مثل `Connection` و`PreparedStatement` و`ResultSet`، بينما يقوم الـ JDBC Driver بتنفيذ هذه الواجهات للتواصل مع قاعدة بيانات معينة.

---

## 2. Difference between Statement and PreparedStatement?

|Statement|PreparedStatement|
|---|---|
|يقبل SQL مباشرة|SQL مع Placeholders (`?`)|
|معرض لـ SQL Injection|آمن ضد SQL Injection|
|أقل كفاءة مع التكرار|قد يكون أكثر كفاءة عند إعادة التنفيذ|
|يستخدم غالبًا للاستعلامات البسيطة|الأفضل في معظم الحالات|

---

## 3. What is SQL Injection?

ثغرة تحدث عند دمج مدخلات المستخدم داخل SQL مباشرة، وقد تسمح بتنفيذ أوامر غير مقصودة على قاعدة البيانات. استخدام `PreparedStatement` هو الحل المعتاد لمنعها.

---

## 4. What is DriverManager?

كلاس مسؤول عن إدارة الـ JDBC Drivers وإنشاء اتصال (`Connection`) بقاعدة البيانات.

---

## 5. What is Connection?

واجهة تمثل جلسة اتصال بين تطبيق Java وقاعدة البيانات، ومنها يتم إنشاء `Statement` أو `PreparedStatement`.

---

## 6. What is ResultSet?

كائن يمثل البيانات الناتجة من استعلام `SELECT`، ويمكن التنقل بين صفوفه باستخدام `next()`.

---

## 7. executeQuery vs executeUpdate؟

|executeQuery|executeUpdate|
|---|---|
|يستخدم مع `SELECT`|يستخدم مع `INSERT` و`UPDATE` و`DELETE`|
|يرجع `ResultSet`|يرجع عدد الصفوف المتأثرة (`int`)|

---

## 8. Why use try-with-resources?

لضمان إغلاق `Connection` و`Statement` و`ResultSet` تلقائيًا حتى في حالة حدوث Exception، مما يمنع تسرب الموارد.

---

## 9. Is JDBC a framework?

لا. JDBC هو **API** (جزء من Java Standard Library)، وليس Framework.

---

## 10. Does Hibernate replace JDBC?

لا. Hibernate لا يستبدل JDBC، بل يبني فوقه. في النهاية، Hibernate يستخدم JDBC Driver لإرسال أوامر SQL إلى قاعدة البيانات.

---

## 11. Does Spring Data JPA use JDBC?

نعم، ولكن بشكل غير مباشر:

```
Application
      ↓
Spring Data JPA
      ↓
Hibernate
      ↓
JDBC
      ↓
Database Driver
      ↓
Database
```

وده سؤال مشهور جدًا في الانترفيوهات لأنهم بيحبوا يشوفوا إنك فاهم الطبقات اللي تحت الـ Framework مش بس بتستخدمه.