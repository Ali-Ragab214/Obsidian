
---

# 🟩 NestJS Modules — Complete Guide

---

# 🎯 ما هو الـ Module في NestJS ؟

الـ Module هو **وحدة تنظيم (Organizational Unit)** بتجمع كل حاجة تخص Feature معينة مع بعض.

يعني بدل ما المشروع يبقى فوضى:

```
controllers
services
repositories
...
```

نقسمه Features:

```
Users Module
Orders Module
Auth Module
Payments Module
```

💡 فكر فيها كأنها **Package / Feature Folder** لكن ذكية ومرتبطة بالـ DI Container.

---

# 🧠 أهم فكرة لازم تفهمها

Nest مش بيشغل Classes وخلاص…

هو بيبني حاجة اسمها:

## 🔗 Application Dependency Graph

ده جراف فيه:

- Modules
    
- Providers
    
- Controllers
    
- العلاقات بينهم
    

وكل ده يبدأ من:

## Root Module (AppModule)

---

# 🏁 Root Module — نقطة بداية التطبيق

كل تطبيق Nest لازم يكون فيه Module واحد على الأقل.

```ts
@Module({
  imports: [UsersModule, OrdersModule],
})
export class AppModule {}
```

Nest يبدأ من هنا ويبني باقي التطبيق.

---

# 🧩 Module Decorator Anatomy

```ts
@Module({
  providers: [],
  controllers: [],
  imports: [],
  exports: [],
})
export class UsersModule {}
```

دي أهم 4 مفاهيم في Nest Modules 👇

---

# 1️⃣ providers

الـ services / repositories اللي Nest هيعملهم instantiate داخل الموديول.

```ts
@Module({
  providers: [UsersService, UsersRepository],
})
```

💡 أي Provider هنا متاح **داخل الموديول فقط**.

---

# 2️⃣ controllers

الـ controllers اللي تستقبل HTTP requests.

```ts
@Module({
  controllers: [UsersController],
})
```

---

# 3️⃣ imports

لو محتاج Providers من Module تاني.

```ts
@Module({
  imports: [AuthModule],
})
```

يعني:

> أنا محتاج الخدمات اللي الـ AuthModule بيقدمها.

---

# 4️⃣ exports (مهم جداً)

الـ providers اللي تسمح لباقي الموديولات تستخدمهم.

```ts
@Module({
  providers: [UsersService],
  exports: [UsersService],
})
```

💡 exports = Public API of the module.

---

# ⚠️ أهم قاعدة في Modules

الـ Provider مش متاح خارج الموديول إلا لو عملت له export.

دي نقطة ناس كتير بتغلط فيها.

---

# 🧱 Feature Modules

أغلب التطبيقات بتكون عبارة عن Feature Modules.

## مثال Users Module

```
users/
 ├── users.controller.ts
 ├── users.service.ts
 ├── users.repository.ts
 ├── users.module.ts
```

```ts
@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],
})
export class UsersModule {}
```

---

# 🧠 ليه بنستخدم Feature Modules ؟

- تنظيم المشروع
    
- فصل المسؤوليات (SOLID)
    
- سهولة التوسع
    
- فرق العمل تشتغل بشكل مستقل
    

---

# 🔄 Shared Modules

الموديولات في Nest = **Singletons**

يعني لو اتعمل import لنفس الموديول في 10 أماكن → نفس instance من providers.

---

## مثال مشاركة Service

### UsersModule

```ts
@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

### OrdersModule

```ts
@Module({
  imports: [UsersModule],
})
export class OrdersModule {}
```

اللي حصل:

```
OrdersService → inject UsersService
```

وكل الموديولات تشارك نفس instance.

💡 ده يمنع:

- استهلاك ذاكرة زائد
    
- مشاكل state inconsistency
    

---

# 🔁 Module Re-Exporting

ممكن Module يمرر Module تاني.

## CoreModule

```ts
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

أي Module يستورد CoreModule يقدر يستخدم CommonModule.

ده اسمه **Re-exporting**.

---

# 🧪 هل الموديول نفسه ممكن يعمل Inject ؟

نعم 👇

```ts
@Module({
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

لكن ⚠️  
لا يمكن حقن Module داخل Module آخر.

الموديول مش Provider.

---

# 🌍 Global Modules

لو عندك Module محتاجه في كل المشروع.

مثال:

- ConfigService
    
- Logger
    
- Database
    

---

## جعله Global

```ts
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}
```

دلوقتي تقدر تعمل inject للـ ConfigService في أي مكان بدون imports.

---

### ⚠️ مهم جداً

الإفراط في Global Modules = Design سيء.

الأفضل:

```
Explicit imports > Global modules
```

---

# 🚀 Dynamic Modules (أقوى مفهوم)

تخيل Module يقبل Config وقت التشغيل 😮

زي:

- DatabaseModule
    
- AuthModule
    
- MailModule
    

---

## مثال DatabaseModule

```ts
@Module({
  providers: [Connection],
  exports: [Connection],
})
export class DatabaseModule {

  static forRoot(entities = []): DynamicModule {
    const providers = createDatabaseProviders(entities);

    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

---

## استخدامه في AppModule

```ts
@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

💡 هنا بنمرر Config للموديول وقت التشغيل.

ده نفس فكرة:

- `TypeOrmModule.forRoot()`
    
- `JwtModule.register()`
    

---

# 🧠 ليه Dynamic Modules مهمة ؟

لأنك تقدر:

- تبني Modules قابلة لإعادة الاستخدام
    
- تعمل Modules قابلة للتهيئة (configurable)
    
- تبني Libraries
    

---

# 🧭 Lifecycle — إزاي Nest يبني التطبيق

```
start app
   ↓
Nest يقرأ AppModule
   ↓
يقرأ imports recursively
   ↓
يبني Module Graph
   ↓
يبني Provider Graph
   ↓
ينشئ Singletons
   ↓
App ready
```

---

# 🆚 مقارنة مع Spring Boot

|NestJS|Spring|
|---|---|
|Module|Package / Configuration class|
|providers|Beans|
|exports|public beans|
|imports|@Import|
|Global module|@Configuration + ComponentScan|

💡 Nest متأثر جداً بـ Spring.

---

# 🧠 الخلاصة النهائية

Module مسؤول عن:

- تنظيم التطبيق Features
    
- عزل Providers
    
- مشاركة Providers عبر exports
    
- بناء Dependency Graph
    
- إدارة Scope وLifecycle
    

---

لو فهمت Providers + Modules  
يبقى انت فهمت **قلب NestJS فعلاً** 🔥

لو حابب نكمل بعد كده  
Dependency Graph & Injection Tokens بعمق رهيب 👌