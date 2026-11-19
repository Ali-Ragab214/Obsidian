//Stereotype annotations are a specialized markers derived from @Component for any Spring bean,for example :@Service we put this on class level to write business logic in it ,@RestController to mark the calss as a Http Requests reciever ,@Repository used on Data Layer

بيسجلوا ال class as a spring bean 
@RestController is a composite annotation from @Controller and @ResponseBody 


 "@RestController is a composite meta-annotation combining @Controller and @ResponseBody. ==🟣@Controller registers the class as a Spring web bean==, while @ResponseBody instructs Spring to serialize the return object directly into the HTTP response body as JSON or XML rather than rendering a view template."

---
spring boot is an opinionated framework that built on spring framework 
spring-boot-starter-web
@SpringBootApplication : =>
@SpringBootConfiguration 
@EnableAutoConfiguration 
@ComponentScan 

spring-boot-starter-web

يعني كدا spring boot is built over spring framework to rapid application development 
through reduce configuration and come with embedded web server and starter dependencies 
@EnableAutoConfiguration 


@SpringBootConfiguration is derived from @Configuration 
@EnableAutoConfiguration 
@ComponentScan => بتخلي سبرينج  يدور علي ال  stereotype annotations 


rr  doesnot make health checks 
lb algorithms => -rr
-least connections 

A component that distributes the incoming trrafic across multiple servers to improve availabaility ,scalability and reliability 


Synch Replica : the request go to the replica and the response has data consistency as there is no lag in i,
Asynch replica : the request read tha data from the replica and this data maybe stale data  يعني ممكن الداتا تبقا قديمه لسه متحدثتش من البريماري 
وعيبه ان ممكن يبقا في  replication lag اللي يؤدي الي stale data 
ولنظام payment هختار synchronous طبعا عشان اضمن ال data consistency  ول socail media هتخار asynchronus عشان مقبول فيه التاخير في performance شويه 


High Availabilty ان السيرفر يكون متاح اكبر فتره ممكنه من غير ما يقع 
Fault tolerance :  ان السيرفر يفضل شغال حتي لو جزء منه وقع يعني مثلا لو عندي 
 primary and replica db 
  وال primary وقعت يبقا ال rplica شغاله 

### ls

#### سؤال 4 من 30

**Replication بتحسّن Read Scalability ولا Write Scalability؟ ولو DB بتتحمل 10,000 read/sec وعملت 3 Replicas، ليه الزيادة مش بالضرورة Linear؟**

اكتب إجابتك تحت 👇
Replication improve  Read Scalability not write , 
if i have  DB بتتحمل 10,000 read/sec وعملت 3  replicas   مش هيفرق في حاجه لان كله هيقرا من ال primary  بردوا 
