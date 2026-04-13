![[Pasted image 20260412231926.png]]             
![[Pasted image 20260412232025.png]] 



![[Pasted image 20260412232621.png]  





---
-WHAT IS SECURITY?

Security is for protecting your data and business
logic inside your web applications.

--planning ->designing -->development -->testing -->deployment 

---
-what is servlet :kind of web server  or app server like :  tomcat 
![[Pasted image 20260413000257.png]]## Servlets & Filters في Java Web Applications

---

### 📌 السيناريو الطبيعي داخل تطبيق الويب

لما بيجي **HTTP Request** من المتصفح أو أي client، الـ **Web Server** (زي Apache Tomcat) مش بيفهم HTTP مباشرةً كـ Java code.

فبيحصل الآتي:

1. الـ **Servlet Container** (اللي هو Tomcat) بيحول الـ HTTP message لـ **ServletRequest**
2. بيبعتها للـ **Servlet** كـ parameter
3. الـ Servlet بيعمل المنطق ويرجع **ServletResponse**
4. الـ Container بيرجع الـ Response للـ Client

> يعني كل حاجة بتكتبها في Java Web App بتتحكم فيها **Servlets**

---

### 🔒 دور الـ Filters

الـ **Filters** هي طبقة بتقف **قبل** ما الـ Request يوصل للـ Servlet.

- بتعمل **اعتراض** لكل Request/Response
- بتعمل شغل مسبق **قبل** ما Business Logic تشتغل
- **Spring Security** بيعتمد عليها عشان يطبق قواعد الأمان (Authentication, Authorization...) على التطبيق

---

### 🖼️ الرسمة بتوضح إيه؟

```
Client → [Filter 1 → Filter 2 → Filter 3] → [Servlet 1 / 2 / 3 / 4]
                    ↑
              Web/App Server
```

الـ Filters بتيجي الأول، وبعدين الـ Request بيعدي عليها كلها قبل ما يوصل للـ Servlet المناسب.

---

**خلاصة:** Filters = حارس البوابة | Servlets = اللي بيشتغل الفعلي