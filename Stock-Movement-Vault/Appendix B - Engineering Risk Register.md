---
title: "Appendix B: Engineering Risk Register"
cssclasses: [rtl]
tags: [appendix, risks, p0, p1, p2]
section: B
---

# Appendix B: Engineering Risk Register
⬅️ [[Appendix A - Proposed Feature Catalog]] | ➡️ [[Appendix C - Audit Scope and Method]]

السجل ده بيلخّص الـ findings المرتبة من [[12 - Security Considerations]] §12.2. الأدلة وتفاصيل الإصلاح موجودة هناك.

| ID | Severity | Area | الملخص | Target phase |
|---|---|---|---|---|
| **P0-1** | 🔴 Critical | Build | فشل compile في الـ provisioning | Phase 1 |
| **P0-2** | 🔴 Critical | Secrets | JWT keys فاضية | Phase 1 |
| **P0-3** | 🔴 Critical | Database | Common migration فاضية | Phase 1 |
| **P0-4** | 🔴 Critical | Multi-tenancy | العزل مش متفرض؛ resolver بيفشل مفتوح (fail-open) | Phase 1 |
| **P0-5** | 🔴 Critical | Inventory | مفيش stock engine ولا balance ولا حماية من المخزون السالب | Phase 1 |
| **P0-6** | 🔴 Critical | Auth | مفيش login/إصدار tokens | Phase 1 |
| **P0-7** | 🔴 Critical | Security | Schema-name injection و rollback هدّام | Phase 1 |
| **P0-8** | 🔴 Critical | Security | تعرّض الـ password hash في الـ DTOs | Phase 1 |
| **P0-9** | 🔴 Critical | Authorization | Privilege escalation في role validation | Phase 1 |
| **P1-1** | 🟠 High | Tenancy | دورة onboarding/approval مكسورة | Phase 1 |
| **P1-2** | 🟠 High | Persistence | غياب الـ auditor بيكسر الـ inserts | Phase 1 |
| **P1-3** | 🟠 High | Products | الـ update بيترفض دايمًا / بيضيّع audit fields | Phase 1 |
| **P1-4** | 🟠 High | Data lifecycle | Hard delete و soft delete غير مستخدم | Phase 1–2 |
| **P1-5** | 🟠 High | Validation | Constraints وحدود pagination ناقصة | Phase 1 |
| **P1-6** | 🟠 High | API | فجوات exception mapping | Phase 1 |
| **P1-7** | 🟠 High | Authorization | مفيش permissions على مستوى الـ method | Phase 1 |
| **P1-8** | 🟠 High | Auth | الـ tenants الـ suspended والمستخدمين المحذوفين لسه صالحين | Phase 1 |
| **P1-9** | 🟠 High | Auth | مفيش refresh/revocation؛ tokens طويلة العمر | Phase 2 |
| **P1-10** | 🟠 High | Perimeter | CORS wildcard، مفيش rate limiting | Phase 2 |
| **P1-11** | 🟠 High | Quality | مفيش tests مفيدة ولا CI | Phase 1–2 |
| **P1-12** | 🟠 High | Operations | مفيش observability؛ SQL logging في الـ default profile | Phase 2 |
| **P1-13** | 🟠 High | Performance | Indexes ناقصة | Phase 2 |
| **P2-1..8** | 🟡 Medium | API/data quality | REST conventions، pagination، OpenAPI، dead code، duplication، soft delete، case sensitivity، eager mapping | Phase 2–3 |

> [!tip] ترتيب الهجوم المقترح
> ابدأ بـ **P0-1 → P0-2 → P0-3** (عشان التطبيق يقوم أصلًا)، وبعدها **P0-4** (العزل) قبل أي feature، وبالتوازي **P0-7 / P0-8 / P0-9** (ثغرات أمنية سريعة الإصلاح).

---
⬅️ [[Appendix A - Proposed Feature Catalog]] | ➡️ [[Appendix C - Audit Scope and Method]]
