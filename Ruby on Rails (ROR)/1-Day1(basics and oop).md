حلو جدًا 👌 كدا انت ماشي صح قبل Rails لازم تبقى فاهم Ruby كويس.  
هنعمل لك شرح مرتب من الصفر لحد OOP بشكل منظم + أمثلة كتير.

خلينا نبدأ 🚀

---

# 🔰 أولاً: يعني ايه Ruby ؟

Ruby لغة برمجة:

- سهلة القراءة جدًا
    
- مرنة
    
- Object Oriented بنسبة 100%
    
- Rails معمول بيها
    

فكرة Ruby الأساسية:

> **Everything is an object**  
> حتى الأرقام والـ strings 👀

---

# 🟢 1️⃣ تشغيل Ruby

في التيرمنال:

```bash
irb
```

دي interactive console تجرب فيها الكود مباشرة.

---

# 🟡 2️⃣ Variables & Data Types

## ✔️ المتغيرات

في Ruby مفيش types قبل المتغير.

```ruby
name = "Ali"
age = 24
price = 10.5
is_admin = true
```

Ruby بتحدد النوع تلقائيًا.

---

## ✔️ أنواع البيانات الأساسية

### 🔹 String

```ruby
name = "Ali"
```

دمج النصوص:

```ruby
puts "Hello #{name}"
```

⚠️ دي اسمها interpolation

---

### 🔹 Numbers

```ruby
x = 10      # Integer
y = 3.5     # Float
```

عمليات:

```ruby
puts 5 + 3
puts 10 / 2
puts 10 % 3
```

---

### 🔹 Boolean

```ruby
true
false
```

---

### 🔹 Nil (زي null)

```ruby
value = nil
```

---

# 🟢 3️⃣ Input / Output

## الطباعة

```ruby
puts "Hello"
print "Hello"
p "Hello"
```

الفرق:

- puts ينزل سطر جديد
    
- print لا
    
- p يعرض القيمة + النوع
    

---

## إدخال من المستخدم

```ruby
name = gets.chomp
puts "Hello #{name}"
```

---

# 🟡 4️⃣ Conditions

## if

```ruby
age = 20

if age >= 18
  puts "Adult"
else
  puts "Child"
end
```

⚠️ لازم end

---

## elsif

```ruby
grade = 85

if grade >= 90
  puts "A"
elsif grade >= 80
  puts "B"
else
  puts "C"
end
```

---

## unless (عكس if)

```ruby
age = 15

unless age >= 18
  puts "Not adult"
end
```

---

# 🟢 5️⃣ Loops

## while

```ruby
i = 1
while i <= 5
  puts i
  i += 1
end
```

---

## each (مهم جدًا في Ruby 🔥)

```ruby
5.times do
  puts "Hello"
end
```

---

## loop على Array

```ruby
numbers = [1,2,3,4]

numbers.each do |n|
  puts n
end
```

---

# 🟡 6️⃣ Arrays

```ruby
numbers = [1,2,3,4]
names = ["Ali","Sara","Omar"]
```

أهم العمليات:

```ruby
numbers.push(5)
numbers.pop
numbers.length
numbers.include?(3)
```

---

# 🟢 7️⃣ Hash (زي Dictionary)

مهم جدًا في Rails 🔥

```ruby
user = {
  name: "Ali",
  age: 24,
  city: "Cairo"
}

puts user[:name]
```

---

# 🟣 8️⃣ Methods

```ruby
def say_hello
  puts "Hello"
end

say_hello
```

---

## method ب parameters

```ruby
def greet(name)
  puts "Hello #{name}"
end

greet("Ali")
```

---

## return

```ruby
def add(a,b)
  return a + b
end

puts add(2,3)
```

⚠️ في Ruby آخر سطر بيرجع تلقائي حتى بدون return

---

# 🔵 9️⃣ Blocks (مهم جدًا جدًا 🔥)

```ruby
3.times do
  puts "Hello"
end
```

مع variable:

```ruby
3.times do |i|
  puts i
end
```

---

# 🟥 نخش بقى على OOP 🔥🔥🔥

Ruby = Object Oriented بالكامل

---

# 🟣 1️⃣ إنشاء Class

```ruby
class User
end
```

إنشاء object:

```ruby
user = User.new
```

---

# 🟣 2️⃣ Constructor (initialize)

```ruby
class User
  def initialize(name, age)
    @name = name
    @age = age
  end
end

user = User.new("Ali",24)
```

@ = instance variable

---

# 🟣 3️⃣ Methods داخل Class

```ruby
class User
  def initialize(name)
    @name = name
  end

  def say_hello
    puts "Hello #{@name}"
  end
end

user = User.new("Ali")
user.say_hello
```

---

# 🟣 4️⃣ Getter & Setter

بدل ما تكتبهم يدوي 👇

```ruby
class User
  attr_accessor :name, :age
end
```

أنواعهم:

- attr_reader → قراءة فقط
    
- attr_writer → كتابة فقط
    
- attr_accessor → الاتنين
    

---

# 🟣 5️⃣ Inheritance

```ruby
class Person
  def speak
    puts "Hello"
  end
end

class Student < Person
end

s = Student.new
s.speak
```

---

# 🟣 6️⃣ Method Override

```ruby
class Person
  def speak
    puts "Hello"
  end
end

class Student < Person
  def speak
    puts "Hi from student"
  end
end
```

---

# 🟣 7️⃣ Super

```ruby
class Person
  def speak
    puts "Hello"
  end
end

class Student < Person
  def speak
    super
    puts "Student here"
  end
end
```

---

# 🟣 8️⃣ Encapsulation

```ruby
class BankAccount
  def initialize(balance)
    @balance = balance
  end

  def show_balance
    puts @balance
  end
end
```

private:

```ruby
private
def secret
end
```

---

# 🟣 9️⃣ Polymorphism

```ruby
class Dog
  def speak
    puts "Woof"
  end
end

class Cat
  def speak
    puts "Meow"
  end
end

animals = [Dog.new, Cat.new]
animals.each { |a| a.speak }
```

نفس الميثود → سلوك مختلف 🔥

---

# 💥 خلاصة اللي لازم تكون فهمته قبل Rails

لازم تكون فاهم:

- variables
    
- methods
    
- arrays & hash
    
- blocks
    
- classes
    
- inheritance
    
- encapsulation
    
- polymorphism
    

وده بالظبط اللي Rails بيعتمد عليه.

---

لو حابب نعمل الجزء التاني قبل Rails (Modules + Mixins + Symbols + Gems + Bundler) قولّي 👌  
ده الجزء اللي بعده مباشرة وهيفرق معاك جدًا بكرا قبل المحاضرة.