-what is the diference between framework and library 
![[Pasted image 20260627015246.png]] 
nest has a strict project structure 
in nest i dont have to install express ,it is default
nest   built on node js , and we use it to create server side app 
![[Pasted image 20260627015647.png]]npm i -g @nestjs/cli
to install nest cli   يعني يبقا معايا كوماند لو رنيتها يعملي اكشن معين 
يعني لما اكتب nest g >>> هينفذ اكشن معين 
nest is ssr
react csr
 -to create a  new project : nest new proj-name

(npm run start) will run the project without watching 
-tsconfig.json has the configs
tsconfig.build.json => has config but after the build

(npm run start:dev)=> watch mode

-- The entry point => is main.ts

import { Module } from '@nestjs/common';

import { AppController } from './app.controller';

import { AppService } from './app.service';

  

@Module({

imports: [],

controllers: [AppController],

providers: [AppService],

})

export class AppModule {}

//Module => يعني اقدر اعرف جواه سيبرفيز وكنترولر وههكذا

// اللي مخليه موديول انو مستخدم ديكوريتور فانكشن اسمها موديول

//we put any controller in the controllers array and any service in the providers array.

//we put any module that we want to import in the imports array.

//we put any service in the provider array

--controller has routes => give example 
--serivce has logic and call db



@Get()

findAll(@Req() req,@Query query): string {
console.log(req);
console.log(query);
return 'This action returns all users';
}
//req is the request object 
//what is query ? 


@Post()
@HttpCode(201) or  @HttpCode(HttpStatus.Created) like spring 


@Patch()
@HttpCode(200 or HttpStatus.200)

@Delete()
@HttpCode(204) => no content 

# Validation 
بيعدي ب 3 مراحل =>  Frontend 
,backend(validation , schema)


![[Pasted image 20260628132240.png]]
 ![[Pasted image 20260628132557.png]]
 /* eslint-disable prettier/prettier */

import { CreateUser } from "./CreateUser.dto";

import { PartialType } from "@nestjs/mapped-types";

  

export class UpdateUserDto extends PartialType(CreateUser) {}

// كدا لو عايزين كل ال properties في ال createUser تكون optional

//لكن لو عايز حاجات معينه هتعمل كلاس عادي وتحط فيه اللي انت عايزه

//or

//export class UpdateUserDto extends PickType(CreateUser, ['name', 'email'] ) {}

// اللي فوق دا هيعمل ابديت لحاجات معينه بس

تمام احنا كدا عملنا create and update user dto  بس لسه معملناش فاليديشن هنعمله المحاضره الجايه ب pipes 


and there is HttpException explain it 
almost we use it with exception 
HttpException('message',httpstatus code)