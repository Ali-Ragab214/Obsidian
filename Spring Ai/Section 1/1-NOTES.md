lets build intelligent apps 
we will start with basics we will build our first spring Ai app with  : 
-OpenAi
-Ollama
-Docker Model Runner
-Aws Bedrock 
![[Pasted image 20260722144331.png]]
![[Pasted image 20260722154838.png]]
![[Pasted image 20260722154916.png]]to help you to interact with an LLM  model  
![[Pasted image 20260722174447.png]]

> **Why does Spring AI inject `ChatClient.Builder` instead of `ChatClient`?**

جاوبه:

> **Because `ChatClient` is designed to be configurable. Spring injects a preconfigured `ChatClient.Builder`, allowing each component to customize prompts, advisors, tools, memory, and other settings before creating its own `ChatClient` instance using `build()`. This follows the Builder Pattern, making object creation flexible and readable.**

logging.pattern.console=%green(%d{HH:mm:ss.SSS}) %blue(%-5level) %red([%thread]) %yellow(%logger{15}) - %msg%n

السطر  هو Pattern مخصص لتنسيق الـ Logs في Spring Boot، بحيث يلوّن:

- الوقت باللون الأخضر.
- مستوى الـ Log (INFO, DEBUG...) بالأزرق.
- اسم الـ Thread بالأحمر.
- اسم الـ Logger بالأصفر.
- ثم يطبع الرسالة (`%msg`).