سأعطيك دليل كامل لاختبار نظام الـ auth في المشروع: نقاط الفحص، الأوامر (curl/Postman)، أمثلة لــ request/response، الأخطاء الشائعة وكيف تتصرف، وكمان شرح مختصر كيف التوكن بيتولد ويُستخدم.

قائمة تحقق سريعة (what you'll do)
- شغّل التطبيق محليًا.
- سجّل مستخدم جديد (POST /api/v1/auth/register).
- سجّل دخول بنفس المستخدم (POST /api/v1/auth/login) واحصل على JWT.
- استخدم الـ JWT في هيدر Authorization لأي مسارات محمية.
- جرّب حالات خطأ (email موجود، بيانات ناقصة، كلمة مرور خاطئة، توكن منتهي/مزور).

1) تشغيل التطبيق محليًا
- من جذر المشروع:
```bash
./mvnw spring-boot:run
```
أو إذا تفضل build+run:
```bash
./mvnw clean package -DskipTests
java -jar target/Skill_Hub_Learning_Platform-0.0.1-SNAPSHOT.jar
```
افتراض: التطبيق يعمل على http://localhost:8080 (إذا غيرت البورت في application.properties عدّل الـ URL).

2) نقاط الدخول (endpoints) المهمة للـ auth
- POST /api/v1/auth/register
  - يعرّف مستخدم جديد ويُرجع AuthResponse (token + user)
  - يستجيب بـ 201 CREATED على النجاح
- POST /api/v1/auth/login
  - يحقق بيانات، ويُرجع AuthResponse (token + user)
  - يستجيب بـ 200 OK على النجاح

ملاحظة: جميع المسارات الأخرى في التطبيق محمية — تطلب Authorization header بـ Bearer token.

3) DTOs المتوقعة (request bodies)
- RegisterRequest (JSON)
```json
{
  "name": "Ali",
  "email": "ali@example.com",
  "password": "secret123",
  "role": "Student"   // optional: Role can be Student/Instructor/Admin (enum)
}
```
- LoginRequest (JSON)
```json
{
  "email": "ali@example.com",
  "password": "secret123"
}
```

4) أمثلة curl لاختبار

- تسجيل مستخدم (register)
```bash
curl -i -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name":"Ali",
    "email":"ali@example.com",
    "password":"secret123",
    "role":"Student"
  }'
```
متوقع (مثال) — HTTP 201:
```json
{
  "success": true,
  "message": "Registration successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiJ9....",
    "user": {
      "id": 12,
      "name": "Ali",
      "email": "ali@example.com",
      "role": "Student"
    }
  },
  "timestamp": "2026-05-24T10:00:00",
  "statusCode": 201
}
```

- تسجيل دخول (login)
```bash
curl -i -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email":"ali@example.com",
    "password":"secret123"
  }'
```
متوقع — HTTP 200:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiJ9....",
    "user": {
      "id": 12,
      "name": "Ali",
      "email": "ali@example.com",
      "role": "Student"
    }
  },
  "timestamp": "2026-05-24T10:05:00",
  "statusCode": 200
}
```

5) كيف تستخدم التوكن للوصول لمسار محمي
بعد حصولك ��لى token من login/register، ضعه في هيدر Authorization:
- مثال:
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
     -H "Content-Type: application/json" \
     http://localhost:8080/api/v1/some-protected-endpoint
```
(بدّل /some-protected-endpoint بمسار حقيقي في مشروعك — أي مسار غير /api/v1/auth/** محمي حسب `SecurityConfig`)

6) حالات خطأ شائعة وكيـف تبدو الاستجابة
- Email already exists (register):
  - AuthServiceImpl يرمي IllegalStateException("Email already exists")
  - GlobalExceptionHandler يعيد 400 Bad Request م��:
```json
{
  "success": false,
  "message": "Email already exists",
  "data": null,
  "timestamp": "...",
  "statusCode": 400
}
```
- Validation error (مثلاً email غير صالح أو password ناقص) من @Valid:
  - GlobalExceptionHandler يعيد 400 مع رسالة "Validation failed" و`data` حقل خريطة الأخطاء:
```json
{
  "success": false,
  "message": "Validation failed",
  "data": { "email": "Email should be valid", "password": "must not be blank" },
  "timestamp":"...",
  "statusCode":400
}
```
- Wrong credentials (login):
  - Spring Security يرمى BadCredentialsException → GlobalExceptionHandler يعيد 401:
```json
{
  "success": false,
  "message": "Invalid email or password",
  "data": null,
  "timestamp":"...",
  "statusCode":401
}
```
- Access to protected endpoint without/with invalid token:
  - عادة تحصل 401 Unauthorized (أو 403) من Spring Security. الـ JwtAuthFilter يحاول قراءة Authorization header ويُهيئ SecurityContext إذا التوكن صالح.

7) كيف التوكن بيتولد وما الذي يحتويه (ببساطة)
- `JwtServiceImpl.generateToken(userDetails)` يبني JWT بـ:
  - subject = userDetails.getUsername() → في التطبيق هذا هو email
  - issuedAt = now
  - expiration = now + قيمة `application.security.jwt.expiration` (في application.properties هو 86400000 ms = 1 يوم)
  - يوقّع التوكن باستخدام HMAC-SHA مع المفتاح السري الموج��د في `application.security.jwt.secret-key` (base64)
- التحقق: `JwtAuthFilter` يقرأ هيدر Authorization، يستخرج التوكن، يستخرج username من التوكن، يحمل UserDetails من DB عبر `CustomUserDetailsService`, ثم `jwtService.isTokenValid(jwt, userDetails)` يتأكد من أن subject يطابق ولم ينتهي صلاحية التوكن.

8) فحص التوكن يدوياً
- انسخ التوكن وادخله في https://jwt.io/ (قسم Verify Signature يحتاج نفس الـ secret key base64) — سترى الـ payload مثلاً يحتوي "sub": "ali@example.com", "exp": timestamp.

9) اقتراحات لاختبارات أوتوماتيكية سريعة
- Integration test باستخدام MockMvc:
  - POST /api/v1/auth/register → assert status 201 والـ data.token غير فارغ
  - POST /api/v1/auth/login → assert status 200 والـ data.token غير فارغ
  - GET محمي مع Authorization: Bearer <token> → assert 200
  - POST login بكلمة مرور خاطئة → assert 401
إذا تريد، أقدر أكتب لك مثال اختبار JUnit + MockMvc جاهز.

10) استخدام Swagger / UI للتجربة
- إذا ضفت springdoc-openapi، افتح:
  - http://localhost:8080/swagger-ui.html
  - أو http://localhost:8080/swagger-ui/index.html
- يمكنك تجربة المسارات من هناك، واضغط Authorize وأدخل: Bearer <token> ثم اختبر المسارات المحمية.

11) Troubleshooting سريع
- 401 بعد إرسال توكن صالح:
  - تأكد التوكن كامل (بدون علامات اقتباس) وبتنسيق "Bearer <space>token".
  - تأكد secret-key في application.properties يطابق الذي استعملته عند إنشاء التوكن.
  - تأكد التوكن لم ينقضِ (expiration).
- 400 من register بسبب email موجود:
  - استخدم بريد جديد أو امسح السجل من DB.
- LazyInitializationException عند تجربة بيانات المستخدم المرتبطة:
  - ليس متعلق بالـ auth نفسه، بل بالعرض (serialization) لعلاقات كيانات خارج سياق Hibernate. الحل: استخدم DTOs أو fetch joins.

12) إذا تريد أمثلة جاهزة
- أقدر أعمل لك:
  - Postman collection JSON جاهز لـ Register/Login + مثال محمي.
  - ملف اختبار JUnit + MockMvc يتأكد من عمل register/login + الوصول لمسار محمي.
  - مثال curl جاهز لاختبار خطأ (bad credentials, duplicate email, validation error).

هل أجهز لك الآن:
- a) Postman collection (JSON) جاهز للتصدير؟  
- b) مثال JUnit + MockMvc لاختبارات اندماجية؟  
- c) أم تريده فقط كملف README شامل مع كل الأمثلة أعلاه؟

اختر واحد وانا أجهزه لك فورًا (Postman collection أو اختبار JUnit أو README).