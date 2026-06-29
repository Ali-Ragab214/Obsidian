Agenda 
-validation
-built in pipes 
-custom pipes 
-services
-connecting to mongodb 
-configuration 

1-validation
through 2 ways : 1 dto , 2 pipes 

VALTDATLON ,

In NestJS, data validation is typically done using Data Transfer
Objects (DTOs) combined with validation pipes. The validation process
helps ensure that data being passed into your application meets
certain criteria before any business logic is executed.

1. DTO (Data Transfer Object)

A DTO is an object that defines the shape of datar that is expected in a
reguest. It allows you to define properties and their validation rules
using decorators provided by the class-validator and class-transformer
libraries.

npm install class-validator class-transformer


(class-validator) package give you a deocorators for valiadtion
like :
@Equals(),@IsEmpty(), @IsBoolean() , @IsEmail(),@IsNegative,@Length, .....

(class-transformer) package give a decorators for serilization 
between objects and classes  
we install these 2 packages in cli give me commands 
لازم عشان الديكوريترز دي تشتغل اروح في ال controller  اللي ال dto  دي هتستخدم فيه واحط بعد ال راوت @UsePipes

-pipes => is a class has a @Injectable() decorator 
there are built in pipes in nest js 
Validationpipe we pass it as an instance to UsePipes 


الفاليديشن هيتطبق علي الروات اللي هحط عنده البايبس دي ,,لو عايزين بقا نحط علي كل الراوتس هنيجي تحت ال decorator بتاعه @Controller ونحط البايبس دي 

and another way if we need to apply these @UsePipe on all controllers we can put it on main.ts
app.useGlobalPipes(new ValidationPipe()); we pass to it an instance from validationpipe >>this will apllay it on all project
we can give ValidationPipe() some options like whitelist we pass it on object in it 
the default is false 
and forbidNonWhiteList
look at docs on partial,pick,ommit types
يعني ممكن نستخدم ommit in partial 
ok there are built in pipes 
there are a pipe called ParseUUIDPipe built in we import it from /comm
we use this with ids if we searched for example on non  existed user 

and there  are custom pipes 
الفكره اصلا من ال  pipe    هي زي الامبوبه ..يعني اي هيعدي منها اي الانبوت واي الاوتبوت ..الانبوت هي الفاليو اللي هتطبق عليها الحاجات دي يعني ممكن تغير شكل الانبوت وتطلعه زي مانت عايز واديني ديجرام 
 if you need yur custom pipe >>cd src and create a folder called pipes and open terminal on it and genereate pipe 
 nest g pi uuid 
 uuid is the pipe name 
  دي بتبقا عباره عن كلاس واي  any pipe must implements PipeTransform interface and has @Injectable decorator    
  يوجد بداخلها transform method that has what you know ,explain it we must override this function and this function called when we used this pipe 
  واستخدام بقا ال pipe دي هيبقا زي  built in pipe في الاستخدام   