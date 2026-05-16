

---

# 📚 شرح كامل: Database Setup من البداية لـ الاتصال (Step by Step)

## 🎯 الهدف النهائي
بنهاية هذا الشرح التطبيق هيتصل بـ PostgreSQL وتقدر تعمل migrations وتحديثات آمنة.

---

# 📋 الخطوات:
1. ✅ تثبيت PostgreSQL
2. ✅ إنشاء قاعدة البيانات
3. ✅ تثبيت حزم NestJS
4. ✅ إنشاء ملفات الإعدادات
5. ✅ إضافة scripts للـ migrations في package.json
6. ✅ شرح app.module.ts
7. ✅ تشغيل التطبيق والاتصال

---

# ⬇️ الخطوة 1: تثبيت PostgreSQL (على CashOS)

## أولاً: تثبيت البرنامج
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

## ثانياً: بدء خدمة PostgreSQL
```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql  # يتشغل تلقائياً عند الإقلاع
```

## ثالثاً: التحقق من التثبيت
```bash
psql --version
```

---

# 🗄️ الخطوة 2: إنشاء قاعدة البيانات وحساب جديد

## ادخل PostgreSQL:
```bash
sudo -u postgres psql 
or 
psql postgres
```

**النتيجة:**
```
psql (14.5)
Type "help" for help.

postgres=#
```

## أنشئ مستخدم جديد:
```sql
CREATE USER your_username WITH PASSWORD '12345';
```
**مثال:**
```sql
CREATE USER ali WITH PASSWORD 'MyPassword123';
```

## أنشئ قاعدة البيانات:
```sql
CREATE DATABASE nest_app OWNER your_username;
```

## أعطِ الصلاحيات:
```sql
GRANT ALL PRIVILEGES ON DATABASE nest_app TO your_username;
```

## تحقق من إنشاء القاعدة:
```sql
\l
```

**النتيجة:**
```
                                     List of databases
     Name      |     Owner     | Encoding |  Collate | Ctype |   Access privileges
  ---------------+---------------+----------+---------+-------+-----------------------
   nest_app      | your_username | UTF8     | en_US   | en_US | =Tc/your_username ...
   postgres      | postgres      | UTF8     | en_US   | en_US |
   template0     | postgres      | UTF8     | en_US   | en_US | =c/postgres
   template1     | postgres      | UTF8     | en_US   | en_US | =c/postgres
```

## اخرج من PostgreSQL:
```sql
\q
```

---

# 📦 الخطوة 3: تثبيت حزم NestJS والمكتبات المطلوبة

## تثبيت @nestjs/config:
```bash
npm install @nestjs/config
```

## تثبيت TypeORM و @nestjs/typeorm:
```bash
npm install @nestjs/typeorm typeorm
```

## تثبيت PostgreSQL driver:
```bash
npm install pg
```

## تثبيت dotenv (اختياري - للأمان):
```bash
npm install dotenv
```

## التحقق من التثبيت:
```bash
npm list @nestjs/config @nestjs/typeorm typeorm pg
```

---

# 🔐 الخطوة 4: إنشاء ملف .env

في جذر المشروع (بجانب package.json):

**الملف: .env**
```env
# معلومات التطبيق
PORT=3000
NODE_ENV=development

# معلومات قاعدة البيانات
DB_TYPE=postgres
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=ali
DB_PASSWORD=MyPassword123
DB_NAME=nest_app

# Logging
DB_LOGGING=true
```

**النقاط المهمة:**
- `DB_USERNAME` = المستخدم اللي أنشأته في PostgreSQL
- `DB_PASSWORD` = كلمة المرور اللي حددتها
- `DB_NAME` = اسم قاعدة البيانات اللي أنشأتها
- **لا تنسى تضيف .env في `.gitignore`** (ما ترفعها على GitHub)

---

# ⚙️ الخطوة 5: إنشاء ملف إعدادات قاعدة البيانات

**الملف: `src/config/database.config.ts`**

```typescript
import { TypeOrmModuleOptions } from '@nestjs/typeorm';
import { ConfigService } from '@nestjs/config';
import { DataSource } from 'typeorm';
import * as dotenv from 'dotenv';

// حمّل .env
dotenv.config();

/**
 * هذه الدالة تُستخدم في app.module.ts
 * تقرأ من .env وترجع إعدادات الاتصال
 */
export const getDatabaseConfig = (
  configService: ConfigService,
): TypeOrmModuleOptions => ({
  type: 'postgres',  // نوع قاعدة البيانات
  
  // قراءة البيانات من .env
  host: configService.get<string>('DB_HOST'),
  port: configService.get<number>('DB_PORT'),
  username: configService.get<string>('DB_USERNAME'),
  password: configService.get<string>('DB_PASSWORD'),
  database: configService.get<string>('DB_NAME'),
  
  // الإعدادات المهمة
  autoLoadEntities: true,  // حمّل جميع الـ Entities تلقائياً
  synchronize: process.env.NODE_ENV === 'development',  // في التطوير يُعدّل تلقائياً
  logging: process.env.DB_LOGGING === 'true',  // اطبع جميع الـ queries
  
  // مسار ملفات Migrations
  migrations: ['src/migrations/*.ts'],
  
  // معرّف الهجرات
  migrationsTableName: 'migrations',
});

/**
 * هذا يُستخدم للـ migrations والـ scripts
 * الـ NestJS ORM CLI تستخدمه
 */
export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: Number(process.env.DB_PORT) || 5432,
  username: process.env.DB_USERNAME || 'postgres',
  password: process.env.DB_PASSWORD || 'password',
  database: process.env.DB_NAME || 'nest_app',
  entities: ['src/**/*.entity.ts'],  // جميع الـ Entities
  migrations: ['src/migrations/*.ts'],  // جميع الـ Migrations
  migrationsTableName: 'migrations',
  synchronize: false,  // عدم التزامن التلقائي (نستخدم migrations)
  logging: false,
});
```

**شرح كل جزء:**

| الجزء | الفائدة |
|-------|---------|
| `getDatabaseConfig()` | دالة تستخدمها app.module.ts للاتصال |
| `AppDataSource` | يستخدمه NestJS CLI للـ migrations |
| `autoLoadEntities: true` | لا تحتاج تكتب Entities يدوياً |
| `synchronize: false` | آمن في الإنتاج - نستخدم migrations |
| `migrations` | مسار ملفات التحديثات |

---

# 📝 الخطوة 6: إنشاء Entity (جدول قاعدة البيانات)

**الملف: `src/users/entities/user.entity.ts`**

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';

/**
 * @Entity('users') = اسم الجدول في قاعدة البيانات
 */
@Entity('users')
export class User {
  /**
   * معرّف فريد (Primary Key)
   * يتزايد تلقائياً 1, 2, 3, ...
   */
  @PrimaryGeneratedColumn()
  id: number;

  /**
   * عمود من نوع string
   * { length: 100 } = أقصى عدد أحرف
   */
  @Column({ length: 100 })
  name: string;

  /**
   * عمود بريد إلكتروني فريد
   * { unique: true } = لا يمكن تكرار نفس الإيميل
   */
  @Column({ unique: true })
  email: string;

  /**
   * عمود كلمة المرور
   */
  @Column()
  password: string;

  /**
   * عمود القالب النشط
   * { default: true } = القيمة الافتراضية true
   */
  @Column({ default: true })
  isActive: boolean;

  /**
   * التاريخ والوقت الذي تم الإنشاء
   * يُعيّن تلقائياً عند الإنشاء
   */
  @CreateDateColumn()
  createdAt: Date;

  /**
   * التاريخ والوقت للتحديث الأخير
   * يُعيّن تلقائياً عند أي تحديث
   */
  @UpdateDateColumn()
  updatedAt: Date;
}
```

---

# 🔗 الخطوة 7: تحديث `src/app.module.ts` (الربط الكامل)

**الملف: `src/app.module.ts`**

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { getDatabaseConfig } from './config/database.config';
import { ClientModule } from './client/client.module';
import { UsersModule } from './users/users.module';

/**
 * @Module = هذا الكلاس هو الوحدة الرئيسية
 * imports = الحاجات اللي نستخدمها
 */
@Module({
  imports: [
    /**
     * الخطوة 1: تحميل متغيرات البيئة من .env
     * 
     * ConfigModule.forRoot() = قول لـ NestJS اقرأ ملف .env
     * { isGlobal: true } = اجعل ConfigService متاح في كل المشروع
     * 
     * بدون هذا ما تقدر تقرأ من .env
     */
    ConfigModule.forRoot({ 
      isGlobal: true,  // متاح في كل الوحدات
      envFilePath: '.env',  // مسار ملف .env
    }),

    /**
     * الخطوة 2: الاتصال بقاعدة البيانات
     * 
     * TypeOrmModule.forRootAsync() = الاتصال بشكل غير متزامن
     * - لماذا async؟ لأننا نتنظر تحميل .env أولاً
     * 
     * inject: [ConfigService] = أعطِ الدالة ConfigService
     * - يعني: "يا NestJS، مرّر ConfigService للدالة getDatabaseConfig"
     * 
     * useFactory: getDatabaseConfig = استخدم هذه الدالة
     * - الدالة تقرأ من .env وترجع الإعدادات الكاملة
     */
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],  // حقن ConfigService
      useFactory: getDatabaseConfig,  // الدالة من database.config.ts
    }),

    /**
     * الخطوة 3: الوحدات الفرعية
     * 
     * كل وحدة تحتوي على:
     * - Module ملف
     * - Service خدمة
     * - Controller متحكم
     * - Entity جدول في قاعدة البيانات
     */
    ClientModule,  // وحدة Clients
    UsersModule,   // وحدة Users
  ],
})
export class AppModule {}
```

**الترتيب الزمني عند تشغيل التطبيق:**

```
1. NestJS يبدأ
   ↓
2. ConfigModule.forRoot() → يقرأ .env
   ↓
3. ConfigService يصير متاح ← الآن ممكن نقرأ البيانات
   ↓
4. TypeOrmModule.forRootAsync() → يستدعي getDatabaseConfig
   ↓
5. getDatabaseConfig(configService) → تقرأ من .env
   ↓
6. TypeORM يتصل بـ PostgreSQL ← الاتصال!
   ↓
7. جميع الـ Entities تُحمّل ← autoLoadEntities: true
   ↓
8. التطبيق يشتغل والـ APIs جاهزة ✅
```

---

# 🔄 الخطوة 8: إضافة Scripts للـ Migrations في package.json

**ابحث عن الجزء `"scripts"` في package.json وأضف هذه الأسطر:**

```json
{
  "name": "first-nest-app",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  
  "scripts": {
    // الأوامر الأساسية (موجودة بالفعل)
    "build": "nest build",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    
    // 👇 أضف هذه الأوامر الجديدة للـ Migrations:
    
    /**
     * توليد migration جديدة تلقائياً
     * مثال: npm run migration:generate -- src/migrations/CreateUsersTable
     */
    "migration:generate": "typeorm migration:generate",
    
    /**
     * إنشاء ملف migration فارغ
     * مثال: npm run migration:create -- src/migrations/CreateUsersTable
     */
    "migration:create": "typeorm migration:create",
    
    /**
     * تشغيل جميع Migrations
     * هذا يحدّث قاعدة البيانات بناءً على الملفات
     */
    "migration:run": "typeorm migration:run -d src/config/database.config.ts",
    
    /**
     * التراجع عن آخر migration
     * إذا عملت خطأ، يرجع الخطوة السابقة
     */
    "migration:revert": "typeorm migration:revert -d src/config/database.config.ts",
    
    /**
     * عرض حالة Migrations
     * أي migrations تم تشغيلها وأيها لم يتم
     */
    "migration:show": "typeorm migration:show -d src/config/database.config.ts",
  },

  "dependencies": {
    "@nestjs/common": "^11.0.1",
    "@nestjs/config": "^4.0.4",
    "@nestjs/core": "^11.0.1",
    "@nestjs/typeorm": "^11.0.1",
    "typeorm": "^0.3.29",
    "pg": "^8.20.0",
    // ... الباقي
  }
}
```

**ملاحظة مهمة:** كل السطور اللي تبدأ بـ `//` و `/**` هي تعليقات (comments). احذفها عند النسخ!

---

# 📖 الخطوة 9: فهم الـ Migrations (التحديثات الآمنة)

## ما هي Migrations؟

تخيل:
```
الأسبوع 1: جدول users فيه: id, name, email
الأسبوع 2: أضفت عمود phone
الأسبوع 3: حذفت عمود password
```

**بدون Migrations:** تعدل الكود مباشرة ← قد تفقد البيانات ❌

**مع Migrations:** كل تغيير مسجل وآمن ✅

## كيفية العمل:

### 1️⃣ عدّل الـ Entity:
```typescript
// كان:
@Column()
email: string;

// أصبح:
@Column()
email: string;

@Column()  // ✨ جديد
phone: string;
```

### 2️⃣ توليد Migration تلقائياً:
```bash
npm run migration:generate -- src/migrations/AddPhoneToUser
```

**النتيجة:** ملف جديد في `src/migrations/`:
```typescript
// src/migrations/1234567890-AddPhoneToUser.ts
export class AddPhoneToUser1234567890 {
  async up(queryRunner: QueryRunner): Promise<void> {
    // هذا يضيف العمود في قاعدة البيانات
    await queryRunner.query(`
      ALTER TABLE "users" ADD COLUMN "phone" character varying
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    // هذا يحذف العمود (للتراجع)
    await queryRunner.query(`
      ALTER TABLE "users" DROP COLUMN "phone"
    `);
  }
}
```

### 3️⃣ تشغيل Migration:
```bash
npm run migration:run
```

**النتيجة:**
- ✅ عمود `phone` أُضيف لجدول `users`
- ✅ جميع البيانات القديمة محفوظة
- ✅ جدول `migrations` يسجل هذا التغيير

### 4️⃣ التراجع (لو حصلت مشكلة):
```bash
npm run migration:revert
```
يعيد الحالة كما كانت قبل آخر migration.

---

# 🚀 الخطوة 10: تشغيل التطبيق والاتصال

## الخطوة أ: تأكد من صحة الملفات:
```
src/
├── config/
│   └── database.config.ts ✅
├── users/
│   ├── entities/
│   │   └── user.entity.ts ✅
│   └── users.module.ts
├── client/
│   └── client.module.ts
├── app.module.ts ✅
└── main.ts
.env ✅
package.json ✅
```

## الخطوة ب: شغّل التطبيق:
```bash
npm run start:dev
```

**النتيجة المتوقعة:**
```
[Nest] 12345  05/17/2026, 2:30:45 PM     LOG [NestFactory] Starting Nest application...
[Nest] 12345  05/17/2026, 2:30:46 PM     LOG [InstanceLoader] TypeOrmModule dependencies initialized
[Nest] 12345  05/17/2026, 2:30:46 PM     LOG [InstanceLoader] AppModule dependencies initialized
[Nest] 12345  05/17/2026, 2:30:46 PM     LOG [NestApplication] Nest application successfully started
[Nest] 12345  05/17/2026, 2:30:46 PM     LOG [NestApplication] Listening on port 3000
```

✅ **التطبيق متصل بقاعدة البيانات!**

## الخطوة ج: تحقق من قاعدة البيانات:

**في terminal جديد:**
```bash
psql -U ali -d nest_app -h localhost
```

**اكتب:**
```sql
\dt
```

**النتيجة:**
```
               List of relations
 Schema | Name      |   Type   | Owner
--------+-----------+----------+-------
 public | migrations | table    | ali
 public | users     | table    | ali
(2 rows)
```

✅ **الجداول موجودة!**

### عرض بيانات جدول users:
```sql
SELECT * FROM users;
```

**النتيجة:**
```
 id | name | email | password | isActive | createdAt | updatedAt
----+------+-------+----------+----------+-----------+-----------
(0 rows)
```

✅ **الجدول فارغ لكنه موجود!**

### الخروج:
```sql
\q
```

---

# 📊 الملخص الكامل: من البداية لـ الاتصال

| الخطوة | الأمر/الملف | الفائدة |
|--------|-----------|---------|
| 1 | `sudo apt install postgresql` | تثبيت PostgreSQL |
| 2 | `CREATE DATABASE nest_app` | إنشاء قاعدة البيانات |
| 3 | `npm install @nestjs/config typeorm pg` | تثبيت المكتبات |
| 4 | .env | حفظ البيانات الحساسة |
| 5 | database.config.ts | إعدادات الاتصال |
| 6 | user.entity.ts | تعريف الجدول |
| 7 | app.module.ts | ربط كل شيء |
| 8 | package.json - scripts | أوامر Migrations |
| 9 | `npm run migration:run` | تشغيل التحديثات |
| 10 | `npm run start:dev` | تشغيل التطبيق |

---

# ⚡ أوامر سريعة (Copy & Paste)

### تشغيل التطبيق:
```bash
npm run start:dev
```

### توليد migration جديدة:
```bash
npm run migration:generate -- src/migrations/CreateUsersTable
```

### تشغيل migrations:
```bash
npm run migration:run
```

### عرض حالة migrations:
```bash
npm run migration:show
```

### الدخول لـ PostgreSQL:
```bash
psql -U ali -d nest_app -h localhost
```

### عرض الجداول:
```sql
\dt
```

### عرض بيانات جدول:
```sql
SELECT * FROM users;
```

### الخروج من PostgreSQL:
```sql
\q
```

---

# 🎓 نقاط مهمة جداً:

1. ✅ **`ConfigModule.forRoot()` أولاً** ← تحميل .env
2. ✅ **`TypeOrmModule.forRootAsync()` ثانياً** ← الاتصال بـ DB
3. ✅ **`synchronize: false`** ← استخدم Migrations
4. ✅ **`autoLoadEntities: true`** ← حمّل Entities تلقائياً
5. ✅ **`AppDataSource`** ← ضروري للـ Migrations
6. ✅ **Scripts في package.json** ← للتحكم بـ Migrations

---

**الآن أنت جاهز! شغّل التطبيق وشوف الجداول تظهر في PostgreSQL! 🎉**You've used 82% of your weekly rate limit. Your weekly rate limit will reset on May 18 at 3:00 AM. [Learn More](https://aka.ms/github-copilot-rate-limit-error)

Similar code found with 2 license types