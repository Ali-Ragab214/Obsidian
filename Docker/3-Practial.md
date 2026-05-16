problem? : البروجكت شغال علي جهاز والتاني لا لاسباب مثلا زي وليكن عندك node:20.0 
فلو وديت البروجكت علي ماشيين تانيه هتحتاج  same versions للداتا بيزس وكل حاجه 

docker put all of that in # Container =>
the app, dependencies ,runtime ,environment 
all of that packed on container  
دا اللي بنشيره عالاجهزه التانيه 
goal : run the same app in the same way everywhere 


-Docker : platform for packaging and running applications inside isolated env called container 
the container will be supported everywhere 

-**docker look at both code and env 
-> Core Concepts : 
= 
=>Image(class) : Blueprint used to create a  container(object) 
ex:  nginx ,node ,postgress ,ubuntu  
-**we pull images from different places like registery 
image is not running 

=>Container : is the running instance from image 
we can : 
start     stop    pause    delete 
it is a running  procces



=>Docker file : file has some instructions written by u 
to build ur image like 

FROM node:20 
COPY .. =>copy all files and put it in container
RUN  npm install =>  شغله وسطب الديبندنسيز
CMD ["npm" ,"start"]





docker run "hello-world"  
لو ال image is not found localy in ur machine it will be pulled بس الحروف لازم تكون كلها small letters  


docker ps =>Running containers 
docker ps -a =>All Containers 
docker run -d  -p 8080:80 --name my-nginx  nginx 
-d لما تعمل الكونتينر رجعني للتيرمنال بتاعي مش عايز اروح للي انت عملته 
-p :ports 
8080  your port 
80   inside container 
اللي بعد النيم هو الاسم عندي والاخير  اسم ال container 


docker stop container_id  هيوقفه 


==We will build our own image 
هنعمل image from app 

we dont create image from scratch we use image from docker hub وبنبني فوقها 

docker build -t  my-node-app .   (1)
to build an image 

docker run -p 8081:8081 --name nodeApp my-node-app (2)

اسم الاميج : my-node-app 

Dockerfile : 
FROM node:20-alpine

WORKDIR /app

COPY app.js .

EXPOSE 8081

CMD ["node", "app.js"] 
دا في ابليكشن بسيط بجافا سكريبت وعملناه ران بنود جي اس علي  كونتينر وشغال عندنا 

  

#FROM  to get the base image from which we want to build our image. Here we are using node:20-alpine as our base image which is a lightweight version of Node.js.

#WORKDIR to set the working directory inside the container. Here we are setting it to /app.

#COPY to copy files from the host machine to the container. Here we are copying app.js from the current directory on the host machine to the /app directory in the container.

#EXPOSE to specify the port on which the application will run. Here we are exposing port 8081.

#CMD to specify the command that will run when the container starts. Here we are running the


الداتا بتبقا عندك انت عالجهاز والكونتينر بيستخدمها  مينفعش الداتا بيز تبقا جواه  بتبقا في volume عندنا عالجهاز والكونتينر بيستخدمها 
-Volume : Store Data outside container 
--مينفعش تحط اي داتا مهمه جوا الدوكر فايل 



-docker run -e POSTGRES_PASSWORD=123456 postgres
will bring postgress image and run it and give it password in env variables  ودا بردوا مش طبيعي 
فهنعمل compose 




---
# Composer 
-manage multi container apps using one file 
Docker Compose :  هل الابليكشن بيكون سيرفر بس ؟ 
لا طبعا بيكون فيه سيرفر و db و monitoring tool  شغاله يعني اكتر من حاجه ...يعني هعمل كونتينر لكل حاجه فعايزهم يكلموا بعض فبنستخدم ال compose 

-we will create file and all services we will need and its docker files and how to build it   
d يعني المشكله والحل 
يعني بضغطه هنشغل كل حاجه 
we write docker compose in yaml syntax 
services:
  app:
     build:   .
     ports:
       - "3030:3030"

     depends_on:
     - db  => كدا بنقوله هيعتمد عالكونتينر بتاع الداتا بيز 

db:
    image:postgress
    environment:
      - POSTGRES_PASSWORD=123456
      - POSTGRESS_DB=ةغ-شحح
      
     PORTS:
        -"5432:5432"
      volumes:
          -where i will restore it on my device 



docker compose up -d  =>run all 
docker compose down =>stop
docker compose down -v =>stop and remove volumes 
volumes : الداتا اللي في الداتا بيز 
