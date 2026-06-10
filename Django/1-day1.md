![[Pasted image 20260604092612.png]]venv isolate my project 
venv file is not sharable we mustnot push it to github 
![[Pasted image 20260604092948.png]]![[Pasted image 20260604093001.png]]
![[Pasted image 20260604093027.png]]![[Pasted image 20260604093310.png]]

settings.py => put secrets like secret key 

INSTALLED_APPS = [

'django.contrib.admin',

'django.contrib.auth',

'django.contrib.contenttypes',

'django.contrib.sessions',

'django.contrib.messages',

'django.contrib.staticfiles',

]

![[Pasted image 20260604100237.png]] 
to create a new app and put it in installed apps  
add it 
'blog'  =>small projects
or 
'blog.apps.BlogConfig'=> large proj

# reverse url (search)
---
---

python -m venv venv => create new virtual env 
venv\scripts\activate =>to activate it 
pip install django =>to install django 
django-admin startproject moviestore =>create project
cd moviestore
python manage.py runserver

![[Pasted image 20260608092257.png]]

![[Pasted image 20260608092818.png]]![[Pasted image 20260608103445.png]]![[Pasted image 20260608103811.png]]![[Pasted image 20260608104110.png]]![[Pasted image 20260608104358.png]]
أيوه بالظبط! في طريقتين:

---

**الطريقة الأولى — صور من الإنترنت (أسرع):**

```python
'poster': {'url': 'https://image.tmdb.org/t/p/w500/q6y0Go1tsGEsmtFryDOJo3dEmqu.jpg'},
```

بس حط لينك صورة حقيقية من أي موقع.

---

**الطريقة الثانية — صور من جهازك (الأصح):**

**١. حط الصور في:**

```
your_project/
└── static/
    └── movies/
        ├── shawshank.jpg
        ├── godfather.jpg
        └── dark_knight.jpg
```

**٢. غير الـ `poster` في الـ views:**

```python
'poster': {'url': 'movies/shawshank.jpg'},   # مش محتاج /static/ هنا
```

**٣. في الـ template استخدم `{% static %}`:**

```html
{% load static %}
<img src="{% static movie.poster.url %}" alt="{{ movie.title }}">
```

---

**نصيحة:** لو المشروع هيكبر، الأحسن تعمل **Django Model** وتستخدم `ImageField` بدل الـ dictionary — ده اللي بيتعامل مع الصور بشكل احترافي أكتر. عايز أساعدك تعمل ده؟

---
----
---
---
![[Pasted image 20260608111007.png]]
![[Pasted image 20260608113821.png]]
def __str__ (self):
    return self.name
pip install pillow to deal with image 
python manage.py migrate
![[Pasted image 20260608114036.png]]
![[Pasted image 20260608114730.png]]
![[Pasted image 20260608115156.png]]![[Pasted image 20260608120043.png]]
![[Pasted image 20260608120347.png]]
![[Pasted image 20260608120413.png]]

![[Pasted image 20260608121900.png]]![[Pasted image 20260608122015.png]]![[Pasted image 20260608122525.png]]![[Pasted image 20260608122739.png]]![[Pasted image 20260608123459.png]]![[Pasted image 20260608124127.png]] od