first we need to bcrypt the password 
so we will download 
npm i bcryptjs ,and we will import bcrypt from bcryptjs 

const salt = await bcrypt.gensalt(10);
const hashedpassword = await bcrypt.hash(user.password,salt)

explain what is salt
to handle login =>the flow =>we check if exists if not =>throw exception ,if yes =>exists and generate token 
but the better everything related with the auth we prefered to create an auth module
ولا انت اي رايك ؟ 
nest g mo auth 
will create an auth  folder has auth.module.ts
وكمان اعمل سيرفيز عشان يبقوا مع بعض 
وحط في السيرفيز اللوجيك بقا وابعتلي الستراكشر
واشتغل اننا هنستخدم TYpeorm  واحقنلي الريبوزتوري جوا السيرفيز وجهزها في ال module 
,وشيك بقا في اللوجيك لو اليوزر مش موجود بال best practice logic
if exists generate a token for him 
and dont forget  to import jwtmodule in module 
npm i jsonwebtoken @nestjs/jwt =>to install it 
and define it in .env file and register in in app.module
JwtModule.registerAsync
![[Pasted image 20260630012252.png]]
and inject jwtservice in authservice to genrate token 
تضيف role بقا لليوزر وكدا 
![[Pasted image 20260630013452.png]]
يعني الفانكشن اللي اسمها لوجين هتعمل كل اللوجيك دا وفي الاخر لو هو صح هتعمل التوكين اشرحلي بقا وقولي هعتعمله ازاي والخطوات من الاول اصلا والفلو بالترتيب من اول ما نزلنا الباكددج
and this function is reusable 
هنعمل بقا راوت اسمه لوجين ونستدعي فيه ال لوجين من ال authservice 
كل موديول ممكن نحط جواه exporst:[] واللي هنحطه هنا هنعرف نستخدمه في اي حته وتروحح في الموديول بناع الفولدر اللي عايز تستخدمها فيه وتعملها امبورت في ال  imports:[]  حط الموديول كله في ال imports احسن 
اللي جاي بعد كدا بقا ال  guards ودا تقريبا خاص ب ال authorization
يفضل انك تستخدمها في ال authentication  واحنا هنعمل كدا فعلا 
we will create folder called commons in src and we put in it :interceptors ,guards ,pipes ,anything related to the app and we will create a  folder called guards in it 
nest g gu guardname
 وهي عباره عن class implement interface called CanActivate and override its function  هنستخدم بقا الجارد دا فين وازاي تعالي علي اي راوت وقول @UseGuards()   
 and we pass to it a class name or an instance يعني مثلا لو ادناها كلاس ال auth علي راوت findall  
 فلازم اللي هيستخدم الراوت دا يكون عامل لوجين
 واشرحلي بقا هعمل دا اازاي جوا الكلاس بتاع الجارد واشرحلي فيه كل حاجه
 we willl inject jwtservice in this class to check token 
![[Pasted image 20260630130716.png]]
بدل بقا ما كل شويه نيجي علي كل راوت ونعمل جارد ممكن نحطها علي مستوي الكنترولر
او لو عايزها جلوبال هتحطها في ال  main.ts 
app.useglobalguards 