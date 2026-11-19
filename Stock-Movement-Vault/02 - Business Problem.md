---
title: Business Problem
cssclasses: [rtl]
tags: [business, personas, product]
section: 2
---

# 2. Business Problem
⬅️ [[01 - Project Overview]] | ➡️ [[03 - Core Features]]

الموزّعين والتجار والمصنّعين الصغار والمتوسطين غالبًا بيدّيروا المخزون بـ **spreadsheets** أو أدوات بسيطة. المشاكل المتكررة:

> [!bug] المشاكل
> 1. **Stockouts** في الأصناف سريعة الحركة و **overstock** في البطيئة، بسبب غياب **balance** موثوق ولحظي.
> 2. **مفيش audit trail** يتوثّق فيه؛ التصحيحات بتتم بتعديل الكميات، فمحدش يعرف *مين غيّر إيه ومتى وليه*.
> 3. **المخازن المتعددة** بتتدار بشكل غير رسمي، فالـ transfers وملكية المخزون غامضة.
> 4. **مفيش فصل بين الـ tenants**. الـ SaaS vendor لازم يضمن إن شركة مستحيل تقرا أو تعدّل مخزون شركة تانية.
> 5. **الـ reporting يدوي**: المدير مش قادر يجاوب بسرعة على أسئلة زي "أنهي منتجات هتخلص الشهر ده؟" أو "أنهي مخزن فيه أعلى قيمة؟"
> 6. **الـ Reconciliation** مؤلمة: الجرد الفعلي بيختلف عن النظام ومفيش workflow مضبوط للـ adjustment.
> 7. **القرارات reactive**: كميات إعادة الطلب (**reorder quantities**) بتتخمّن بدل ما تتحسب من الطلب والـ **lead time**.

## 👥 الـ Personas

| الـ Persona | الاحتياج الأساسي |
|---|---|
| **Warehouse operator** | تسجيل receipts و issues و transfers و counts بسرعة وأمان |
| **Inventory manager** | شوف الـ balances وتحذيرات low-stock وتاريخ الحركات والفروقات |
| **Company administrator** | إدارة المستخدمين والـ roles والمخازن وإعدادات الـ tenant |
| **Finance / operations analyst** | تصدير الـ valuations وتقارير الحركات |
| **Platform administrator** | Onboard / suspend / support للـ tenants **من غير** ما يدخل على بياناتهم التجارية من غير داعي |

## 🎯 Product Positioning

> [!important] النطاق التجاري الواقعي الأول
> **Inventory control و stock movement للـ SMBs** — **مش ERP كامل**.
> الـ Accounting و Procurement و Manufacturing BOMs و E-commerce order management منتجات مجاورة، ولازم تفضل **out of scope** لحد ما نواة المخزون تبقى موثوقة.

## ✨ الـ Differentiators اللي تستاهل البناء ناحيتها

1. **Correctness و Auditability** لكل تغيير في المخزون.
2. **Tenant isolation حقيقي** بـ isolation model واضح.
3. **Replenishment قابل للتنفيذ**: reorder points و forecasts واقتراحات redistribution.
4. **AI assistant آمن** يجاوب أسئلة تشغيلية **من غير ما يقدر يغيّر المخزون**.

---
⬅️ [[01 - Project Overview]] | ➡️ [[03 - Core Features]]
