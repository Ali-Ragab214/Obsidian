spring.ai.openai.api-key=
spring.ai.openai.chat.options.model
ChatModel class is low level and responsible for dealing with Ai Provider 
and ChatCLient is the abstraction of it and a high level api to        make a call or deal with LLM 

prompts 
![[Pasted image 20261102222110.png]]

# Spring AI – Prompt Engineering & Prompt Management

ملخص محاضرة Dan Vega – Working with Prompts in Spring AI

## 1. Prompt Engineering

هو إنك تكتب Instructions للـ AI بطريقة واضحة ومنظمة عشان تحصل على الـ Output المطلوب.

مثال:

```
You are a Java instructor.
Explain Spring Boot to a beginner.
Use simple examples and Java code.
```

كل ما كان الـ Prompt واضح ومحدد، زادت فرص الحصول على Response مناسب.

معلومة: فيه Prompt Collections جاهزة على الإنترنت تقدر تستخدمها وتعدل عليها بدل ما تبدأ من الصفر.

## 2. Prompt Templates

Spring AI بتدعم استخدام Text Templates لإنشاء Prompts قابلة لإعادة الاستخدام باستخدام Dynamic Variables.

مثال:

```
Recommend a movie in the {genre} genre.
```

وقت التنفيذ:

```
genre = Action
```

النتيجة:

```
Recommend a movie in the Action genre.
```

الفوائد:

- Reusability.
    
- Dynamic Variables.
    
- سهولة إدارة وتعديل الـ Prompts.
    
- عدم تكرار نفس النص في كل مرة.
    

## 3. Prompt Class & Messages

الـ `Prompt` في Spring AI مش مجرد String، لكنه بيمثل مجموعة من Messages اللي هتتبعت للـ AI Model.

أهم أنواع الـ Messages:

|Message|الوظيفة|
|---|---|
|`SystemMessage`|تحديد سلوك وقواعد الـ AI|
|`UserMessage`|تمثيل طلب المستخدم|
|`AssistantMessage`|تمثيل رد الـ AI|

مثال:

System Message:

```
You are a professional Java instructor.
Explain concepts simply.
```

User Message:

```
What is Dependency Injection?
```

Assistant Message:

```
Dependency Injection is a design pattern
where an object receives its dependencies
from an external source.
```

## 4. Prompt vs Prompt Template

|Prompt|Prompt Template|
|---|---|
|يمثل الرسائل المرسلة للـ AI|قالب لإنشاء Prompt|
|يحتوي على Messages|يحتوي على Dynamic Variables|
|يمثل الـ Input النهائي|يساعد في تكوين الـ Input|

## 5. Prompt في Spring MVC vs Spring AI

Dan شبّه الـ Prompt بالـ View في Spring MVC.

- في Spring MVC: الـ Controller بيجهز البيانات ويحطها في Model، والـ View بتستخدمها.
    
- في Spring AI: الـ Application بتجهز Dynamic Variables، والـ Prompt Template بتستخدمها لإنشاء الرسالة النهائية للـ AI.
    

## 6. هل Spring AI مجرد REST API Wrapper؟

لأ، هو بيوفر Abstractions وComponents بتساعدك في التعامل مع تحديات بناء AI Applications، زي:

- Prompt Engineering.
    
- Prompt Management.
    
- التعامل مع اختلاف AI Providers.
    
- تحويل AI Responses إلى Java Objects.
    
- إدارة الـ Messages والـ Conversations.
    

## أهم المصطلحات

|Term|Meaning|
|---|---|
|LLM|Large Language Model|
|Prompt|Input sent to an AI Model|
|Prompt Engineering|Designing effective prompts|
|Prompt Template|Reusable prompt with variables|
|Dynamic Variables|Values injected at runtime|
|System Message|Instructions for AI behavior|
|User Message|User's input|
|Assistant Message|AI's response|
|Prompt Management|Organizing and managing prompts|

## الخلاصة اللي لازم تخرج بيها

احفظ وفهم الخمس نقاط دول:

1. Prompt Engineering هو تصميم Instructions فعالة للـ AI.
    
2. Prompt Templates بتسمح بإعادة استخدام Prompts مع Dynamic Variables.
    
3. Prompt في Spring AI بيمثل مجموعة Messages.
    
4. كل Message ليها Role، وأهمهم System وUser وAssistant.
    
5. Spring AI مش مجرد REST API Wrapper، لكنه بيوفر أدوات وAbstractions لبناء AI Applications.
    

## Checklist للمراجعة

- أفهم Prompt Engineering.
    
- أفهم Prompt Templates.
    
- أفهم Dynamic Variables.
    
- أفرق بين SystemMessage وUserMessage وAssistantMessage.
    
- أفهم العلاقة بين Prompt وPrompt Template.
    
- أقدر أكتب Prompt Template بنفسي.
    

نصيحتي: احتفظ بالملخص ده في Obsidian، ومتدخلش في تفاصيل Implementation قبل ما تفهم المفاهيم دي. الخطوة التالية في المذاكرة هي تطبيق `Prompt`, `SystemMessage`, `UserMessage` و`PromptTemplate` عمليًا باستخدام Spring AI.