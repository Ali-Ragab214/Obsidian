
جميل 🔥 كده دخلت عالم Rails API بجد.  
هنعمل شرح كامل خطوة بخطوة **Rails API + Relations + Scaffold** بمثال عملي:

هنبني API بسيط فيه:

- User
    
- Post  
    و العلاقة بينهم:
    

> User has many Posts  
> Post belongs to User

وده أشهر مثال في Rails 👌

---

# 🎯 أولاً: إنشاء Rails API Project

لو عايز API فقط بدون Views:

```bash
rails new blog_api --api
cd blog_api
```

شغل السيرفر للتأكد:

```bash
rails s
```

---

# 🧠 ثانياً: يعني اي Model Relation؟

احنا عايزين:

- المستخدم يقدر ينشر Posts
    
- كل Post لازم يكون ليه User
    

يعني:

```
User 1 ---- * Posts
```

دي اسمها:

- One To Many Relationship
    

---

# ⚙️ ثالثاً: إنشاء Models باستخدام Scaffold

هننشئ User الأول 👇

```bash
rails g scaffold User name:string email:string
```

هننشئ Post مرتبط بـ User 👇

```bash
rails g scaffold Post title:string body:text user:references
```

ركز في دي 👇  
`user:references`  
دي أهم كلمة في الدرس كله 💣

Rails هيعمل:

- عمود user_id في جدول posts
    
- foreign key
    
- index
    
- relation جاهزة
    

بعد كده نشغل الميجريشن:

```bash
rails db:migrate
```

---

# 🗄️ شكل قاعدة البيانات بعد الميجريشن

## جدول users

| id | name | email |

## جدول posts

| id | title | body | user_id |

`user_id` ده هو الرابط بين الجدولين 🔗

---

# 🧩 رابعاً: إضافة العلاقات في Models

افتح Model بتاع User:

### app/models/user.rb

```ruby
class User < ApplicationRecord
  has_many :posts
end
```

افتح Model بتاع Post:

### app/models/post.rb

```ruby
class Post < ApplicationRecord
  belongs_to :user
end
```

بس كده 👏  
Rails فهم العلاقة.

---

# 🔥 خامساً: تجربة العلاقة من Rails Console

افتح الكونسول:

```bash
rails c
```

إنشاء User:

```ruby
u = User.create(name: "Ali", email: "ali@mail.com")
```

إنشاء Post مرتبط به:

```ruby
u.posts.create(title: "First Post", body: "Hello Rails API")
```

نجرب نجيب بوستات اليوزر:

```ruby
u.posts
```

نجرب نجيب صاحب البوست:

```ruby
p = Post.first
p.user
```

🔥 Rails بيعمل الربط تلقائي.

---

# 🌍 سادساً: تجربة الـ API Endpoints

Scaffold عمل Controllers جاهزين 👌

## إنشاء User

POST

```
/users
```

Body JSON:

```json
{
  "user": {
    "name": "Ali",
    "email": "ali@mail.com"
  }
}
```

---

## إنشاء Post مربوط ب User

POST

```
/posts
```

```json
{
  "post": {
    "title": "Rails API",
    "body": "Learning Rails",
    "user_id": 1
  }
}
```

---

## جلب كل Posts

GET

```
/posts
```

---

## جلب Post واحد

GET

```
/posts/1
```

---

# 💎 سابعاً: نخلي الـ API يرجع العلاقة مع البيانات (Nested JSON)

دلوقتي لو جبت posts مش هيظهر اليوزر.

نعدل Controller 👇

### app/controllers/posts_controller.rb

غير index:

```ruby
def index
  posts = Post.includes(:user).all
  render json: posts.to_json(include: :user)
end
```

وجرب:

```
GET /posts
```

هيرجع:

```json
[
  {
    "id": 1,
    "title": "Rails API",
    "body": "Learning Rails",
    "user": {
      "id": 1,
      "name": "Ali",
      "email": "ali@mail.com"
    }
  }
]
```

🔥 ده اسمه Eager Loading + Nested JSON.

---

# 🧠 ملخص سريع تحفظه

أهم commands في Rails API:

```bash
rails new app --api
rails g scaffold User name email
rails g scaffold Post title body user:references
rails db:migrate
rails s
rails c
```

العلاقات:

```ruby
has_many :posts
belongs_to :user
```

---

لو حابب الدرس الجاي نعمل  
Nested Routes  
زي:  
`/users/1/posts`  
وده الجزء المهم في APIs الاحترافية 😎