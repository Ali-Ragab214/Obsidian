![[Pasted image 20260725153006.png]]تمام، خليني أشرحلك الـ Advisors بالتفصيل من الـ script.

## إيه هو الـ Advisor أصلاً؟

الفكرة الأساسية: الـ advisors دول زي الـ interceptors أو الـ middleware للـ prompt flow بتاعك، بالظبط زي الـ filters في التطبيقات الويب التقليدية. يعني لو انت متعود على الـ filters أو الـ interceptors في Spring MVC، دي نفس الفكرة بالظبط بس مطبقة على الـ AI requests.

بمعنى تاني: هو **middleware layer** بيقدر يمسك الـ request قبل ما يوصل للـ LLM، وكمان يمسك الـ response بعد ما يرجع من الـ LLM.

## إيه اللي بيعمله بالظبط؟

بيسمحلك تعمل pre-processing أو post-processing logic على بيانات الـ prompt، وتضيف custom logging أو auditing requirements، وتحقن additional behavior من غير ما تعدل الـ core logic، وتعمل chain لأكتر من behavior بطريقة نضيفة عن طريق تعريف أكتر من advisor.

يعني باختصار: أي **cross-cutting concern** (حاجة مش جزء من الـ business logic الأساسي بس محتاجها في كل الـ requests) بتتعمل هنا.

## إزاي الـ Flow بيمشي؟

ده أهم جزء تفهمه كويس:

1. المستخدم بيعمل call لـ Rest API
2. الـ request بيوصل لـ **ChatClient bean**
3. من هنا، قبل ما الـ request يوصل للـ LLM، الـ advisors المُعرّفة بتمسك الـ request وتنفذ اللي متعرف جواها
4. تقدر تعرّف أي عدد من الـ advisors، وكلهم بيشتغلوا في شكل chain (واحد ورا التاني)
5. بعد ما الـ advisors تخلص، الـ request بيتبعت للـ LLM فعليًا
6. الـ response بيرجع، وتاني مرة الـ advisors بتمسك الـ response، وبعد ما كل الـ advisors تنفذ الـ logic بتاعها، الـ response يوصل للمستخدم

يعني الـ advisors بتشتغل في **الاتجاهين**: ذاهب للـ LLM وراجع منه.

## الـ Best Practices اللي المفروض تلتزم بيها

### 1. خليهم Stateless

لازم تخلي الـ advisors stateless أو request-scoped بس. متحاولش تحتفظ بالداتا عبر أكتر من request أو أكتر من session.

**ليه ده مهم؟** لو الـ advisor بيحتفظ بحالة (state) بين الـ requests، ده هيسبب مشاكل concurrency خطيرة في بيئة multi-user، وده حاجة إنت فاهمها كويس أصلاً من خبرتك في Spring (زي الفرق بين singleton beans و request-scoped beans).

### 2. Single Responsibility لكل Advisor

الأفضل إنك تعمل أكتر من advisor، كل واحد فيهم يركز على حاجة واحدة بس (cross-cutting concern واحد)، مش تحط كل حاجة في advisor واحد. مثلاً advisor لل logging، وadvisor تاني منفصل للـ security.

ده نفس مبدأ الـ Single Responsibility Principle اللي انت شغال بيه في SOLID، بس مطبق هنا على الـ advisors.

### 3. متغيرش معنى الـ Prompt

لما بتعمل implement للـ advisors دي، لازم تتجنب إنك تغير معنى الـ prompts إلا لو ده مقصود، لأن الـ advisors بتدي صلاحيات كبيرة للـ developer إنه يتدخل في الـ request قبل ما يوصل للـ LLM — لكن ده مش معناه إنك تغير المعنى الكامل بتاع الـ prompt data اللي المستخدم بعته.

يعني لو المستخدم كتب سؤال معين، الـ advisor ميعملش تعديل جذري في نية السؤال إلا لو فيه سبب business واضح.

### 4. متحطش الـ Business Logic هنا

متحاولش تكتب الـ core business logic بتاعك جوا الـ advisors — دي المفروض تكون دايمًا جوا الـ controller Rest APIs بتاعتك.

**ده مهم جدًا ليك خصوصًا**: زي ما انت متعود تفصل الـ concerns في Spring Boot (Controller/Service/Repository)، هنا نفس المبدأ — الـ advisors مكانها الـ housekeeping، مش الـ actual logic بتاع الـ feature.

## إزاي بتعرّفها في الكود؟

فيه طريقتين للـ configuration:

1. **على مستوى الـ ChatClient bean كله (Default)**: بتقدر تعرّف advisor على مستوى الـ ChatClient bean باستخدام default advisor، عن طريق ChatClientBuilder — يعني الـ advisor ده هيشتغل مع **كل** الـ requests اللي بتعدي من الـ ChatClient ده.
    
2. **على مستوى request معين بس**: أو تعرّف advisor خاص بـ request واحد بس عن طريق invoke method اسمه advisors() — ده شبيه جدًا بطريقة default() و system() اللي اتشرحوا قبل كده.
    

## الـ Advisors الجاهزة (Built-in)

من الأمثلة على الـ built-in advisors: SimpleLoggerAdvisor، SafeguardAdvisor، PromptChatMemoryAdvisor.

من الأسامي، تقدر تتوقع وظيفة كل واحد:

- **SimpleLoggerAdvisor**: تسجيل الـ requests/responses
- **SafeguardAdvisor**: حماية من محتوى غير آمن أو مخالف
- **PromptChatMemoryAdvisor**: الاحتفاظ بسياق المحادثة (memory) عبر الـ requests

## لو عايز تعمل Advisor خاص بيك

تقدر تعمل implement لـ interfaces زي CallAdvisor و StreamAdvisor عشان تبني advisor مخصص بيك.

---

## إزاي ده ممكن يفيدك في StockSavvy؟

بما إن عندك RAG assistant وكذا AI agent (forecasting, reordering, negotiation, anomaly detection)، لو كنت هتطبق نفس الفكرة في NestJS (بدون Spring AI طبعًا)، فكر في الـ advisors كـ **NestJS Interceptors** — تقدر تعمل interceptor للـ logging، وواحد تاني للـ rate limiting على الـ AI calls، وواحد للـ audit trail (مهم جدًا مع الـ HITL approval flow بتاعك قبل ما أي PO يتبعت). المبدأ نفسه: كل concern لوحده، وميتلخبطش مع الـ business logic.