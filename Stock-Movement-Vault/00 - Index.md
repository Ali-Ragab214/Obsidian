---
title: Stock Movement Management System — Index
cssclasses: [rtl]
tags: [moc, index, stock-movement, spring-boot]
aliases: [Home, MOC, الرئيسية]
created: 2026-11-19
---

# 📦 Stock Movement Management System

> [!abstract] إيه ده؟
> **Backend** لنظام **multi-tenant** لإدارة المخزون وحركات المخزن (**Inventory / Stock Movement SaaS**)، مبني بـ **Java 21** و **Spring Boot 4.1.2** و **PostgreSQL 17.5**.
> الملف ده ترجمة وتنظيم لتقرير الـ **Audit** اللي اتعمل بتاريخ **18 نوفمبر 2026**.

> [!danger] حالة المشروع باختصار
> المشروع في مرحلة **early-stage / Work-in-Progress** — **مش جاهز للـ production**.
> - الـ **build بيفشل** (`ProvisioningServiceImpl.java:47`).
> - حتى بعد إصلاحه، الـ startup هيقف بسبب ملفات الـ **JWT keys** الفاضية (0 bytes).
> - الـ **common migration** فاضية، يعني جداول `tenants` و `users` مش موجودة.
> - الـ **REST API** الوحيد المتاح هو **Category CRUD**.
> - **Stock IN / OUT / Transfers / Adjustments / Balances** كلها **مش موجودة**؛ `StockMvtServiceImpl` فيها methods فاضية.
> - الـ **Multi-tenancy** مقصودة لكن **مش مفعّلة**: الـ tenant context بيتملى من الـ JWT بس مفيش حاجة بتستخدمه.

> [!info] ملاحظة عن التقرير
> كل حاجة في التقرير متعلّمة بوضوح: **Implemented** أو **Partially implemented** أو **Proposed / Not implemented**. الشغل المخطط له **عمره ما اتعرض كأنه موجود**.

---

## 🗺️ خريطة الملاحظات

### الجزء الأول — الوضع الحالي
- [[01 - Project Overview]] — نظرة عامة
- [[02 - Business Problem]] — المشكلة والـ Personas
- [[03 - Core Features]] — إيه اتعمل وإيه لأ
- [[04 - Technology Stack]] — الـ Stack
- [[05 - Architecture Overview]] — المعمارية

### الجزء التاني — القرارات الكبيرة
- [[06 - Multi-Tenancy Strategy]] — أهم قرار معماري ⚠️
- [[07 - Authentication and Authorization]]
- [[08 - Database Design]]
- [[09 - Stock Movement Lifecycle]] — قلب المنتج

### الجزء التالت — API والـ AI
- [[10 - API Documentation]]
- [[11 - AI Capabilities]]

### الجزء الرابع — الجودة والتشغيل
- [[12 - Security Considerations]]
- [[13 - Performance and Scalability]]
- [[14 - Testing]]
- [[15 - Deployment Instructions]]

### الجزء الخامس — المستقبل
- [[16 - Project Roadmap]]
- [[17 - Future Enhancements]]

### الملاحق
- [[Appendix A - Proposed Feature Catalog]]
- [[Appendix B - Engineering Risk Register]]
- [[Appendix C - Audit Scope and Method]]

---

## 🏷️ دليل الـ Status

| الرمز | المعنى |
|---|---|
| ✅ **Implemented** | الكود موجود ومربوط بـ entry point، ومفروض يشتغل بعد إصلاح الـ blockers |
| 🟡 **Partially implemented** | الكود موجود بس ناقص أو مش reachable أو فيه bug |
| 🧱 **Scaffolded** | الـ types أو الـ config موجودة لكن الـ feature مش قابل للاستخدام |
| ❌ **Not implemented** | مفيش أي implementation فعلية |
| 💡 **Proposed** | تصميم مقترح، مش كود موجود |

## 🎯 أولويات الـ Severity

| المستوى | المعنى |
|---|---|
| 🔴 **P0 — Critical** | blocker لازم يتصلح قبل أي حاجة |
| 🟠 **P1 — High** | مخاطر عالية على الـ production |
| 🟡 **P2 — Medium** | جودة الـ API والـ data |

---

> [!tip] إعداد Obsidian
> الـ vault دي فيها CSS snippet اسمه `rtl` متفعّل تلقائيًا. لو الاتجاه مش RTL: **Settings → Appearance → CSS snippets** وفعّل `rtl`.
> الـ Mermaid diagrams بتتعرض مباشرة في Obsidian من غير plugins.
