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