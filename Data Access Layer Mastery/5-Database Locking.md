pessimestic ,optimistic
اولا ال concurency  هي :  
ان البرنامج يتعامل مع اكتر من تاسك في نفس الوقت وبيتنقل بينهم بحيث كل مهمه تاخد وقت من ال cpu  
لو عندي اكتر من transaction بيحاولوا يغيروا نفس الداتا فنفس الوقت هينتج عن دا conflict 
فاحنا عايزين نعالج دا 

فنتخيل انا فالجيم وعايز احط حاجتي فاللوكر ومحدش غيري يحط ف هقفله ب lock  واما اجي اخدها هفك ال lock واخدها وبكدا هضمن سلامتها 

عندنا نوعين بقا من ال locking  : 
pessimestic ,optimistic 

-Optimistic
![[Pasted image 20260811195434.png]]الاوبتميستك محترم يعني لما بيغير حاجه فالداتا بيز بيضيف مثلا فيرجن او تايم ستامب اي حاجه تدل  عالتغيير  
وبيتجنب انو يعمل locking  علي ال record in the db فبالتالي بيستجيب للكويري بشكل اسرع 
وبالتالي  better performance 

# we use it in systems that have little number of conflicts 
 عيوبه لو كان في عدد كتير من ال concurrent update  لانو هيحصل كونفليكت 

 **أنا مش همنع transaction تانية من قراءة الـ record، لكن هتحقق قبل الـ UPDATE إن البيانات اللي قرأتها لسه هي نفسها.**

وده اسمه:

> Optimistic Locking
----




2-Pessimistic  هو بردوا وسيله لتوفير حل عشان تمنع الكونفليكتس الناتجه من ال  concurrent updates 

بس الفرق هنا ان لو في transaction try to make an update on a specific row it will ==lock== it on this record and no one can modify it until it is finished 

![[Pasted image 20260811200220.png]]فكدا ممكن نعتبر ان دا شخص متشائم متوقع اسوء حاجه هتحصل وياخذ احتياطاته من البدايه 

# it preferd to use it in the systems that have a lot of concurrent updates 


![[Pasted image 20260811200501.png]]
 البيرفورمانس هنا مش احسن حاجه بسبب ال locking  اللي محطوط علي rows  كتيره ولفترات زمنيه طويله و high resource utilization 

---
يعني مثال كدا عندنا لو انا روحت الجيم وروحت علي لوكر احط فيه حاجتي وفتحته وقولت هروح  اشرب واجي احط الحاجه بتاعتي بس قفلته بقفل لحد ما ارجع ف دا pessimestic   لكن لو مقفلتوش ف دا  optimistic 