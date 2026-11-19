---
title: Future Enhancements
cssclasses: [rtl]
tags: [future, ideas, backlog]
section: 17
---

# 17. Future Enhancements
⬅️ [[16 - Project Roadmap]] | ➡️ [[Appendix A - Proposed Feature Catalog]]

دي تحسينات **قيّمة لكنها متأجّلة عمدًا** بعد أول مرحلتين. بعضها متجدول في **Phase 3** لو القطاع المستهدف محتاجه؛ الباقي لاحقًا.

## 📦 Inventory depth
- **Batch/lot و serial-number tracking** بتتبّع كامل (traceability).
- **Expiry dates** و **FEFO/FIFO picking**.
- **Barcode/QR scanning** و **mobile warehouse workflow**.
- **Kitting/bundles** وتجميع خفيف (light assembly).
- **Consignment stock** ونماذج ملكية المخازن.
- **Drop-shipping** و in-transit inventory.
- **Returns** و reverse logistics.
- **Supplier management** و purchase orders و goods receipt.
- **Multi-currency costs** وتوزيع **landed-cost**.

## 📊 Analytics
- **Stock turnover** ومقاييس days-of-inventory.
- تصنيف **fast/slow/dead stock**.
- أداء المخازن واستغلال السعة.
- تقارير **service-level** و **fill-rate**.
- تحليل **ABC/XYZ**.
- **Demand sensing** بالـ promotions والإشارات الخارجية.
- **Scenario simulation** لسياسات الـ reorder ونقل المخازن.

## ☁️ SaaS platform
- **Self-service signup** و tenant onboarding.
- **Subscription plans** و trials و coupons و dunning.
- **Usage metering** و overage handling.
- **SSO/SAML** و **SCIM** و **MFA** وإدارة الـ sessions.
- **Roles وصلاحيات مخصصة** بتفصيل دقيق.
- **Branding** وإعدادات مخصصة لكل tenant.
- إشعارات داخل التطبيق و email digests.
- **Webhooks** و public API بـ API keys.
- **Audit export** وتقارير الامتثال.
- **Data residency** وسياسات retention لكل tenant.

## 🛠️ Engineering
- **Event-driven integrations** عبر transactional outbox.
- **Read model / warehouse** للـ analytics.
- **Feature flags** و progressive delivery.
- **Chaos و failure-injection tests**.
- **Cost observability** لكل tenant.
- **Public SDK** وسياسة API versioning/deprecation.

## 🤖 AI
- **Forecasting** بإشارات promotions و seasonality.
- **Prescriptive replenishment** بقيود الميزانية.
- **Supplier lead-time prediction**.
- **Computer vision** للاستلام/الجرد (**بس لو الـ ROI اتثبت**).
- **Voice-driven warehouse operations** (قيّم الأمان والدقة بحذر).

---

## 🤝 Contributing

مفيش `CONTRIBUTING.md` لسه. لحد ما يتعمل:

1. اعمل **feature branch**.
2. خلّي التغييرات **صغيرة ومركّزة**.
3. ضيف **tests** لأي تغيير في السلوك، خصوصًا tenant isolation و stock invariants.
4. شغّل `./mvnw verify` قبل فتح pull request.
5. **ماتعملش commit** لأسرار أو مفاتيح مولَّدة أو بيانات production.

## 📄 License

**TBD.** ملف `pom.xml` فيه license metadata فاضية، ومفيش ملف `LICENSE` في الـ repository. اختار license **قبل** ما تقبل مساهمات خارجية.

---
⬅️ [[16 - Project Roadmap]] | ➡️ [[Appendix A - Proposed Feature Catalog]]
