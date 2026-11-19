---
title: Project Overview
cssclasses: [rtl]
tags: [overview, spring-boot, multi-tenant]
section: 1
---

# 1. Project Overview
⬅️ [[00 - Index]] | ➡️ [[02 - Business Problem]]

المشروع ده **backend** لـ **Multi-tenant Stock Movement / Inventory Management SaaS**. المنتج المقصود بيخلّي الشركة:
- تدير **products** و **categories**.
- تسجّل **stock movements** في المخازن.
- وفي مراحل بعدين: **inventory balances** و **transfers** و **adjustments** و **reporting** وتحليل بمساعدة الـ **AI**.

## 🏗️ شكل التنفيذ
الـ implementation عبارة عن **layered Spring Boot monolith**:

- **Identity و Tenancy** في الـ **shared `public` schema** (`Tenant`, `User`).
- **بيانات المخزون** في **per-tenant PostgreSQL schemas** (`categories`, `products`, `stock_mvts`).
- **JWT-based authentication** بـ **RS256**؛ الـ tenant بيتاخد من **signed claim** في التوكن، والـ legacy filter بتاع `X-TenantID` **متقفّل**.
- **Flyway** للـ schema migrations، بما فيها الـ per-tenant migrations اللي بتتشغّل **programmatically** وقت الـ provisioning.
- **Spring Data JPA repositories** و **manual DTO mappers**.

> [!tip] الهدف بعيد المدى
> **Inventory Management SaaS** تجاري فيه:
> - **tenant isolation** قوي.
> - **stock ledger** صحيح وقابل للـ audit.
> - **operational reporting**.
> - طبقة **AI read-only** عمرها ما بتعدّل المخزون.

## 📋 Project at a glance

| العنصر | القيمة |
|---|---|
| **Repository** | `saas-multi-tennant-app` (اسم الـ package محتفظ بالإملاء الأصلي *عمدًا*) |
| **Language / runtime** | Java 21 |
| **Framework** | Spring Boot 4.1.2 |
| **Web layer** | Spring MVC (`spring-boot-starter-webmvc`) |
| **Persistence** | Spring Data JPA / Hibernate, PostgreSQL |
| **Migrations** | Flyway + `flyway-database-postgresql` |
| **Security** | Spring Security, JJWT 0.12.6, RSA (RS256) |
| **API documentation** | springdoc-openapi 3.2.0 |
| **Database** | PostgreSQL 17.5 (Docker Compose) |
| **Multi-tenancy** | Schema-per-tenant (*intended, not enforced*) |
| **Test suite** | test واحد بس: `@SpringBootTest` context-load |
| **Build status** | ❌ **Failing** |
| **Production readiness** | ❌ **No** |

## 📝 الـ README ده بيعمل إيه؟
نتيجة **static audit كامل** للـ repository مع محاولة compile. أربع أهداف:

1. **توثيق الموجود** مع إثبات على مستوى الملفات (*file-level evidence*).
2. **تحديد الفجوات** في المعمارية والأمان والصحة (*correctness*).
3. **اقتراح roadmap واقعي** للمنتج والهندسة.
4. **تعريف استراتيجية AI آمنة وtenant-aware** لمنظومة Spring Boot.

> [!note] TBD
> أي معلومة مقدرناش نتحقق منها اتكتب **TBD** بدل التخمين.

---
➡️ التالي: [[02 - Business Problem]]
