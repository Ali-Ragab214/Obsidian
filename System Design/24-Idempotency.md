نمنع تكرار البيانات  
safe methods 
Idempotent methods =>GET,HEAD,PUT ,DELETE,OPTIONS,TRACE 
يعني مهما عملت get  مش هاثر علي السيرفر 
يعني مثلا لو عميل بيبعت ريكوست انو يدفع فاتوره وبعت الريكوست والفلوس اتسحبت ومتبعتلوش الريسبونس فهيحاول تاني ..فلو انت مش عامل حساب كدا الفلوس هتتسحب تاني 
وغالبا ال request for payment هيكون POST  
ودا خطا من اليوزر بس احنا شغلنا اننا نراعي اخطاء اليوزرس 
and post is not idempotent  
 فهنهدل دا ازاي هنحط our safe guard  فنخلي مثلا payment  اللي من نوع  POST  هنحط معاها  unique identifier uuid فلو حصل فيلر هنبعت نفس ال uuid مره تانيه عشان العمليه متحصلش مرتين ومش هنخلي الريكوست يوصل للداتا بيز 
 ودا بنسميه  idempotent key   ودا دوره ي   manage  ال  duplication
 هناخد مثال علي سبرينج بوت 
 هنعمل config folder 
  جواه IdempotencyINterceptor class that implements HandlerInterceptor interface and override preHandleMethod 
  ![[Pasted image 20260706163013.png]]
  and this is interceptor 
  لو الهيدر مش موجود يبقا خلاص كاننا مشوفناش الانتيرسيبتور
  ![[Pasted image 20260706163151.png]]
  ![[Pasted image 20260706163304.png]]
يعني كله يتمحور حول ال  idempotency key 