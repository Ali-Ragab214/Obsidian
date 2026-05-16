

---

# ✅ 1) Database Configuration

- استخدام **PostgreSQL أو MySQL**
    
- إعداد الاتصال بـ **env variables**
    
- فصل config في ملف `database.config.ts`
    
- فهم الفرق بين:
    
    - `synchronize = false` في production ⚠️
        
    - استخدام **migrations بدل synchronize**
        

---

# ✅ 2) Entities Design (Database Modeling)

لازم تكون قوي في:

- Entity per module
    
- Naming conventions
    
- Column options:
    
    - unique
        
    - nullable
        
    - default
        
    - enum
        
    - indexes ⭐
        
- Audit columns:
    
    - createdAt
        
    - updatedAt
        
    - deletedAt (soft delete)
        

---

# ✅ 3) Relationships (VERY IMPORTANT)

لازم تبقى فاهم كويس:

- OneToOne
    
- OneToMany / ManyToOne ⭐
    
- ManyToMany
    
- JoinColumn / JoinTable
    
- eager vs lazy loading
    
- Cascade options
    
- Relation loading performance ⚠️
    

---

# ✅ 4) Repository Pattern

- استخدام `@InjectRepository`
    
- Custom repositories (لو محتاج logic خاص)
    
- فصل ال DB logic داخل service layer فقط
    

---

# ✅ 5) CRUD Best Practices

لازم تفهم الفرق بين:

- save vs update
    
- remove vs delete
    
- create vs preload
    

---

# ✅ 6) Query Builder (Production Queries)

لازم تعرف تعمل:

- joins
    
- complex filters
    
- search
    
- pagination
    
- sorting
    
- aggregation (count, sum)
    

ده أهم جزء في المشاريع الكبيرة 🔥

---

# ✅ 7) Pagination + Filtering + Searching

لازم تطبق:

- limit / offset
    
- cursor pagination (advanced)
    
- dynamic filters
    
- dynamic sorting
    

دي موجودة في أي API حقيقي.

---

# ✅ 8) Transactions

ضروري جدًا:

- QueryRunner
    
- commit / rollback
    
- multi-step operations
    

أي system فيه payments أو bookings محتاج ده.

---

# ✅ 9) Migrations (Production Mandatory)

لازم تعرف:

- generate migration
    
- run migrations
    
- revert migration
    
- versioning database
    

ممنوع تستخدم synchronize في production ❌

---

# ✅ 10) Soft Delete + Data Recovery

- استخدام `@DeleteDateColumn`
    
- restore soft deleted records
    
- filtering non-deleted records
    

---

# ✅ 11) Indexing & Performance

لازم تفهم:

- database indexes
    
- when to add index
    
- query optimization
    
- N+1 problem ⚠️
    

---

# ✅ 12) Error Handling

- handling DB exceptions
    
- unique constraint errors
    
- transaction errors
    

---

# ✅ 13) Seeding & Dummy Data

لازم تعرف:

- seed script
    
- generate fake data
    
- initial admin user
    

---

# 🎯 لو عرفت اللي فوق تبقى جاهز Production فعلاً

ده مستوى:

- Graduation project
    
- Internship
    
- Junior backend job
    

---

لو حابب أحول الليست دي لخطة مذاكرة step-by-step قولّي ونبدأ فيها 💪