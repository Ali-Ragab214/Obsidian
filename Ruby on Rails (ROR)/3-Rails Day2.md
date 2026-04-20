deal with db 

---

# 🧠 أولاً: Rails بيشتغل مع DB إزاي؟

Rails فيه حاجة اسمها **Active Record ORM**

يعني:

- كل Table في الداتابيز = Model في Rails
    
- كل Row = Object
    
- كل Column = Attribute
    

مثال:

```
users table
id | name | email
```

يبقى عندك Model اسمه:

```ruby
class User < ApplicationRecord
end
```

Rails بيحول بين Ruby Objects و SQL أوتوماتيك 🔥

---

# 🧱 فكرة الـ Migrations

المـيجريشن = كود Ruby بيغيّر شكل الداتابيز (Schema)

بدل ما تكتب SQL زي:

```sql
CREATE TABLE users...
ALTER TABLE users ADD COLUMN age...
```

Rails يخليك تكتب Ruby وهو يحولها SQL 👌

الميجريشن = Version control للداتابيز  
زي Git لكن للـ DB.

---

# 📦 ملفات الداتابيز في Rails

في فولدر مهم جداً:

```
db/
 ├── migrate/     ← ملفات الميجريشن
 ├── schema.rb    ← شكل الداتابيز الحالي
 ├── seeds.rb     ← بيانات افتراضية
 └── development.sqlite3
```

---

# 🏗️ إنشاء Migration

## إنشاء Table جديدة

```bash
rails generate migration CreateUsers
```

Rails فهم إنه migration من كلمة **migration** في الأمر.

هيعمل ملف جوه:

```
db/migrate/20260417090000_create_users.rb
```

اسمه فيه Timestamp علشان ترتيب التنفيذ.

---

## كتابة الميجريشن

نفتح الملف ونكتب:

```ruby
class CreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      t.string :name
      t.string :email
      t.integer :age

      t.timestamps
    end
  end
end
```

### يعني إيه الكلام ده؟

Rails بيترجمه SQL تقريباً كده:

```sql
CREATE TABLE users(
  id INTEGER PRIMARY KEY,
  name VARCHAR,
  email VARCHAR,
  age INTEGER,
  created_at,
  updated_at
)
```

---

# ▶️ تشغيل الميجريشن

```bash
rails db:migrate
```

💥 هنا يحصل أهم حاجة:

Rails يعمل جدول اسمه:

```
schema_migrations
```

يحفظ فيه ID كل migration اتنفذت.

عشان يعرف:

- إيه اللي اتنفذ
    
- إيه اللي لسه
    

يعني Rails بيعمل Tracking للـ migrations.

---

# 🔄 Rollback يعني إيه؟

لو عملت مشكلة في الميجريشن 😅

```bash
rails db:rollback
```

يرجع آخر Migration اتنفذت.

ليه يقدر يرجع؟  
لأن Rails بيعرف يعمل **reverse** للـ change method.

مثلاً:

```
create_table ↔ drop_table
add_column ↔ remove_column
```

لو عايز ترجع 3 خطوات:

```bash
rails db:rollback STEP=3
```

---

# 📊 أهم أوامر DB في Rails

## تشغيل كل الميجريشن

```bash
rails db:migrate
```

## عرض حالة الميجريشن

```bash
rails db:migrate:status
```

هيطلع:

```
up    20240417 Create users
down  20240418 Add age to users
```

## حذف الداتابيز وإعادة إنشائها

```bash
rails db:reset
```

يعني:

```
drop + create + migrate
```

## إنشاء الداتابيز فقط

```bash
rails db:create
```

## حذف الداتابيز

```bash
rails db:drop
```

---

# 🧩 إضافة Column جديدة

في الشغل دايماً بنعدل الداتابيز.

مثلاً عايزين نضيف phone للمستخدم.

```bash
rails generate migration AddPhoneToUsers phone:string
```

Rails ذكي 👀  
فهم من الاسم:

```
AddPhoneToUsers
```

هيولد الميجريشن جاهزة:

```ruby
def change
  add_column :users, :phone, :string
end
```

ثم:

```bash
rails db:migrate
```

---

# 🗑️ حذف Column

```bash
rails generate migration RemoveAgeFromUsers age:integer
```

ينتج:

```ruby
remove_column :users, :age, :integer
```

---

# ✏️ تعديل Column

```bash
rails generate migration ChangeAgeTypeInUsers
```

ثم نكتب:

```ruby
def change
  change_column :users, :age, :string
end
```

---

# 🧮 إضافة Index (مهم في الشغل)

الأداء ⚡

```bash
rails generate migration AddIndexToUsersEmail
```

```ruby
add_index :users, :email, unique: true
```

---

# 🔗 العلاقات بين الجداول (Real Work)

خلينا نبني سيستم حقيقي 👇  
شركة عندها Users و Posts

## إنشاء Posts Table

```bash
rails generate migration CreatePosts title:string body:text user:references
```

Rails فهم العلاقة 🔥

الميجريشن:

```ruby
create_table :posts do |t|
  t.string :title
  t.text :body
  t.references :user, foreign_key: true
end
```

ده يعمل:

```
user_id column
+ foreign key
```

---

## الموديلز

User:

```ruby
class User < ApplicationRecord
  has_many :posts
end
```

Post:

```ruby
class Post < ApplicationRecord
  belongs_to :user
end
```

💥 كده بقى عندك Relation كاملة.

---

# 🌱 Seeds (بيانات افتراضية)

ملف:

```
db/seeds.rb
```

نضيف:

```ruby
User.create(name: "Ali", email: "ali@test.com")
User.create(name: "Omar", email: "omar@test.com")
```

تشغيل:

```bash
rails db:seed
```

---

# 🧾 schema.rb مهم جداً

ده شكل الداتابيز النهائي بعد كل الميجريشن.

Rails بيعيد بناء DB منه لو لزم الأمر.

---

# 🏢 سيناريو شغل حقيقي

1️⃣ عملنا Users table  
2️⃣ بعد شهر احتجنا phone → Migration جديدة  
3️⃣ بعد شهر احتجنا Posts → Migration جديدة  
4️⃣ بعد شهر حذفنا column قديم → Migration جديدة

💡 لا نعدل ميجريشن قديمة أبداً في الشغل  
دايماً نعمل Migration جديدة.

ليه؟  
عشان باقي التيم يقدر يعمل:

```
git pull
rails db:migrate
```

وتتظبط الداتابيز عندهم تلقائي 👌

---

# 🎯 الخلاصة

Rails Database Flow:

1️⃣ نكتب Migration  
2️⃣ نشغل migrate  
3️⃣ Rails يحول Ruby → SQL  
4️⃣ يسجلها في schema_migrations  
5️⃣ يقدر يعمل rollback وقت الحاجة

---

لو حابب بعد كده ندخل على **Rails Console + Queries + ActiveRecord عملي** نكمل الرحلة 😎