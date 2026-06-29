بنفصل ال business logic for the app in it  زي ما كنا شغالين 
nest  g s users => to generate a service 
will genearate a service and put it in provider 
we use @Injectabele decorator function above the calss to use the service everwhere 
  ونحط هنا بقا كل اللوجيك بتاعه الفانيكشن بحيث ان الكنترولر ميبقاش فيه لوجيك 
عشان بقا نشتغل علي داتا بيز حقيقيه 
npm i @nestjs/mongose mongose
 عشان نكونكت بقا هتروح في ال imports array وتحط فيها  mongoosemodule 
 and pass the connection url forroot("url")
 in app.module
 and then create a schema  وبنعمل فيها فاليديشن بردوا 
 هنروح جوا الفولدر مثلا بتاع اليوزر ونعمل سكيما فولدر schemas
  to say that this class is a schema you must out 
  @Schema 
  each variable has 
  @Prop 
 
  er EET NEL EE TE EED VY EE

import ( Prop, Schema | from "@nestjs/mongoose";

@Schema()

export class Userl

?
@Prop(ireguired:true,unigue:true))
name:string;

@Prop(ireguired:true))
email:string;

@Prop(ireguired:true))
password:string;
schemafactory.createforclass(classname)
export const userschema =  schemafactory.createforclass(classname)
وهنيجي فالموديول بتاع اليوزر ونعملهم امبورت in mongoosemodule in forfeature  
 وهنيجي في السيرفيز بقا ونعمل inject for the model in the constructor 
 uctor(@InjectModel(User.name) private userModel :Modelklyserb)) ()
   الفانكشنز بقا هتبقا async  وهترجع  promise  من الحاجه اللي هترجع وهنستخدم فيها await 
 المفروض بقا نحط اليوزر نيم والباسورد بتاعه ال db  وكدا في .env file and db url 
  هنعمل بقا الكونفيجيريشن بتاعه اول حاجه بنعمله هننزل  
  npm i --save @nestjs/config 
  step 2 
  هنروح ل app.module.ts and we will import COnfigModule.forroot 
  وهنديله  envfilepath as a parameter in array  يعني اسمه الفايل 
  عشان نوصل لاي متغير في ال   .env we must inject a variable called ConfigService 
   
  // MongooseModule.forRoot ("mongodb://localhost/nest"),
MongooseModule. forRootAsync(f
p imports: [ConfigModule],
useFactory:async. (configSservice:Configservice) s1
EE [ return furi:configservice.getsstring” ("DB URL"))
inject:[ConfigSservice]
DOP