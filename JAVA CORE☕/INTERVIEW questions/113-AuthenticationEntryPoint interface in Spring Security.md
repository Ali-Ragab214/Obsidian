```java
package ali.ragab.saas_multi_tennant_app.security;

import ali.ragab.saas_multi_tennant_app.responses.ErrorResponse;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.http.MediaType;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;
import tools.jackson.databind.json.JsonMapper;

import java.io.IOException;
import java.time.LocalDateTime;

@Component
@RequiredArgsConstructor
public class ApiAuthenticationEntryPoint implements AuthenticationEntryPoint {

    private final JsonMapper jsonMapper;

    @Override
    public void commence(final HttpServletRequest request,
                         final HttpServletResponse response,
                         final AuthenticationException authException) throws IOException, ServletException {

        final ErrorResponse error = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpServletResponse.SC_UNAUTHORIZED)
                .error("UNAUTHORIZED")
                .message("Full authentication is required to access this resource")
                .path(request.getRequestURI())
                .build();

        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.getWriter().write(this.jsonMapper.writeValueAsString(error));
    }
}
 
الكلاس دا بيعمل اي ؟
```

بص، الكلاس ده مسؤول عن التعامل مع حالة معينة في Spring Security، وهي:

**لما User يحاول يدخل على Endpoint محتاج Authentication، لكنه مش عامل Authentication أو الـ Authentication فشل، نرجعله HTTP 401 برسالة JSON منظمة بدل ما نرجعله Response فاضي أو صفحة HTML.**

خلينا نفهمه واحدة واحدة.

## **1. يعني إيه AuthenticationEntryPoint؟**

في Spring Security عندنا Interface اسمها `AuthenticationEntryPoint`.

دي Spring بتستدعيها لما يحصل Authentication Failure ويكون المفروض المستخدم يعمل Authentication عشان يقدر يوصل للـ Resource.

مثال:

عندك Endpoint:

```java
@GetMapping("/api/v1/auth/me")
public UserResponse me() {
    // Return current user
}
```

والـ Endpoint ده محتاج JWT.

لو المستخدم بعت Request من غير Token:

```
GET /api/v1/auth/me
```

Spring Security هتمنع الوصول للـ Controller، وتستدعي:

```
commence(request, response, authException)
```

وهنا يبدأ دور الكلاس بتاعنا.

## **2. نشرح الـ commence()**

```
@Override
public void commence(
        final HttpServletRequest request,
        final HttpServletResponse response,
        final AuthenticationException authException
)
```

دي الـ method اللي Spring Security بتناديها وقت الـ Authentication Failure.

عندنا 3 Parameters:

- `request`: الـ HTTP Request اللي المستخدم بعته.
    
- `response`: الـ HTTP Response اللي هنرجعه للمستخدم.
    
- `authException`: الاستثناء اللي حصل أثناء عملية الـ Authentication.
    

## **3. بناء الـ Error Response**

```
final ErrorResponse error = ErrorResponse.builder()
        .timestamp(LocalDateTime.now())
        .status(HttpServletResponse.SC_UNAUTHORIZED)
        .error("UNAUTHORIZED")
        .message("Full authentication is required to access this resource")
        .path(request.getRequestURI())
        .build();
```

هنا بنجهز Object يمثل الخطأ.

كل Field ليه وظيفة:

|Field|وظيفته|
|---|---|
|timestamp|وقت حدوث الخطأ|
|status|HTTP Status Code = 401|
|error|نوع الخطأ|
|message|رسالة توضح المشكلة|
|path|الـ Endpoint اللي المستخدم حاول يوصل له|

## **4. تحديد HTTP Status**

```
response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
```

بنقول للـ Client إن الـ HTTP Response بتاعنا حالته:

```
HTTP/1.1 401 Unauthorized
```

خلي بالك: `401` معناها إن المستخدم مش authenticated، مش إن صلاحياته غير كافية.

## **5. تحديد نوع الـ Response**

```
response.setContentType(MediaType.APPLICATION_JSON_VALUE);
```

بنحدد إن الـ Response اللي راجع هيكون JSON.

ليه؟

عشان الـ Frontend يعرف يتعامل معاه باعتباره JSON، بدل ما يستقبل HTML أو Response غير منظم.

## **6. تحويل الـ Object إلى JSON**

```
response.getWriter().write(
    this.jsonMapper.writeValueAsString(error)
);
```

إحنا عندنا Java Object اسمه `error`.

لكن الـ HTTP Response محتاج بيانات بصيغة JSON.

فـ `JsonMapper` بتحول الـ Object إلى JSON String، وبعد كده بنكتبها في الـ Response Body.

مثلاً المستخدم ممكن يستقبل:

```
{
  "timestamp": "2026-11-25T17:20:00",
  "status": 401,
  "error": "UNAUTHORIZED",
  "message": "Full authentication is required to access this resource",
  "path": "/api/v1/auth/me"
}
```

## **7. ليه استخدمنا** `@Component` **و** `@RequiredArgsConstructor`**؟**

```
@Component
@RequiredArgsConstructor
public class ApiAuthenticationEntryPoint
```

- `@Component`: بتخلي Spring تعمل Bean من الكلاس وتقدر تحقنه في `SecurityConfig`.
    
- `@RequiredArgsConstructor`: Lombok بتعمل Constructor للـ `final` fields، وبالتالي Spring تقدر تعمل Constructor Injection للـ `JsonMapper`.
    

## **8. الفرق بين AuthenticationEntryPoint و AccessDeniedHandler**

دي نقطة مهمة جدًا في الإنترفيو:

|AuthenticationEntryPoint|AccessDeniedHandler|
|---|---|
|المستخدم غير authenticated|المستخدم authenticated|
|بيرجع 401|بيرجع 403|
|محتاج يعمل Login أو يقدم Credentials صحيحة|محتاج صلاحيات أعلى|

مثال:

- User مش باعت JWT → `401 Unauthorized`.
    
- Student باعت JWT صحيح وحاول يدخل Admin Endpoint → `403 Forbidden`.
    

## **اختبر فهمك**

**لو Student معاه JWT صحيح وحاول يدخل Endpoint محمي بـ** `ROLE_PLATFORM_ADMIN`**، مين المفروض يتعامل مع الطلب؟**

AuthenticationEntryPoint

AccessDeniedHandler

GlobalExceptionHandler

تأكيد الإجابة

**إجابة صحيحة!**

المستخدم Authenticated بالفعل، لكن معندوش الصلاحية المطلوبة، فبنرجع `403 Forbidden`.

**الخلاصة:** الكلاس ده مش مسؤول عن التحقق من صحة الـ JWT نفسه؛ هو مسؤول عن تجهيز وإرجاع `401 JSON Response` لما Spring Security تقرر إن الطلب غير مصادق عليه.