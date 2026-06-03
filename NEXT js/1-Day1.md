next is react framework 
 لاننا نعرف نستخدم كل ال  hooks and apackage from react in next 
  بس مش هتحتاج بقا تحمل ال packages like react router  لان دا فريم وورك ف كل حاجه موجوده 

react is library for building user interfaces  مقدرش اقوم موقع برياكت بس لام المكاتب اللي بنزلها معاها  
  there are new features in linux that companies nowdays prefere 
  in react we used csr يعني بنرندر من ناحيه الكلاينت 
  يعني لما الفرونت بيكلم الباك ايند الباك بيبعت الداتا عن  طريق جيسون 
  الفرونت ايند هو الكلاينت الباك ايند هو السيرفر  فالكلاينت بيبعت ريكوست ويستقبل ريسبونس 
![[Pasted image 20260602094842.png]]

we have SSR  ,SSG , ISR=>Types of rendering 
csr is better as a performance which used in react  بس مش افضل حاجه في ال search 
لكن الحاجات اللي مش هحتاجها فالبحث خليك csr احسن كاداء وقرر بقا هتشتغلها ssr ,ssg,isr وكل الشركات بقت تعمل كدا عشان seo 

resct -> next 
angular -> ssr 
vue -> nuxt

rendering->display element in screen
we have prerendering in next 

Why learn Next.js?

“Next js simplifies the process of building a React application for production
- File based routing 1
- Support for css modules
“ Pre-rendering
      Next.js create html page for all at once that affect the performance
- API routes
- Dev and Prod build system
- Authentication

![[Pasted image 20260602100845.png]]
App router =>server side 
Page Router=>Client Side 
installation : 
npx create-next-app name
=>choose customize and use eslint and react compiler and src/ 
and alias 
dir 
and page router  مش هتيجي فالاختيارات تقريبا  مادام مش ابب هتبقا كدا 
and agent ai 

most sites made with page router

different between dependencies and devdependencies 
هتلاقي جوا ال src->page folder  and styles 
in pages u will find api folder in which we write backend 
and 
index.js
_app.js=>
_document=>handle html structure 
add component folder to add 

pages folder only to routing  in index.js 
![[Pasted image 20260602103435.png]]

we have 2 options to handle routing 
we can make about.js in pages and about.js in Componen  
and import the component in the one in the pages dir 


in pages we make folder named product 
and u must create index.js in it to inherit it 
and u can run it through 
/products

to make it dynamic 
![[Pasted image 20260602110941.png|542]]
![[Pasted image 20260602110905.png]] look at products folder 
and nested dynamic routing 

![[Pasted image 20260602111618.png]]
 لو عايزين اكتر الدنيا هتبوظ 
 فعندنا نوع راوتنج اسمه Catch all يعني امسك لك الديناميك راوتينج جوا فولدر واحد بس داخل فايل واحد 
 ![[Pasted image 20260602111736.png]]
![[Pasted image 20260602112151.png]]
when we install bootstrap we must import it in app.js first
![[Pasted image 20260602112925.png]]
![[Pasted image 20260602113010.png]]بنعمل ال

بنعمل الكومبوننت فوق ونستدعيه في ال pages يعني بنكريته في ال  components folder 
![[Pasted image 20260602113256.png]]
![[Pasted image 20260602113622.png]]

in app.js
![[Pasted image 20260602113638.png]] لو عايز مخليش حاجه تظهر مع الكومبوننت دا 


---
SSG 
method to prerender  for cashing 
 بعد ما تجيب الداتا بتعملها كاش 
 ![[Pasted image 20260602114632.png]]![[Pasted image 20260602114801.png]]