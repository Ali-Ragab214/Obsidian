 بنختاره علي حسب السيستم بتاعنا ممكن لو اختارناه غلط يسبب مشاكل 
 ![[Pasted image 20260622031045.png]]Cache hit -> روحت اقري البيانات من الميموري ولقيتها 
 Cache miss -> ملقيتش البيانات متكيشه لما روحت اقراها 
# TYpes of Reading strategies 

# 1-Cache Aside
![[Pasted image 20260622031520.png]]
 -دا اللي احنا استخدمناه ونعرفه من اسمه كدا ال  cache بجانب الداتا سورس بتاعي 
 its very simple 
 - الريكوست بيروح للكاش لو لقي الداتا بيرجعها ملقهاش بيروح للداتا سورس دا المعظم والاشهر 
- وهنا ال backend  هو اللي بيتحكم في اللوجيك بتاع ال caching  وهو اللي متحكم تمام فيه 
- ودا عرضه لل stale data 
- مناسب اكتر لل read heavy apps لو بتعمل عمليات قرائه كتير فال stale data هتقل 


# 2-Read Through
 مماثل لل cach aside لكن المتحكم هو ال cache  يعني ال app هيروح يطلب البيانات من الباكيند فالباكيند هيروح للكاش لو لقي الداتا هيرجعها لو ملقهاش ف ال cache نفسه هو اللي هيروح للداتا سورس ويجيب الداتا ويعمل ابديت لنفسه ويرجعها 
 ![[Pasted image 20260622032406.png]]


# 3-Refresh Ahead 
![[Pasted image 20260622032654.png]]


---
---
---
---
---

# Reading Strategies


# 1-Write THrough 
![[Pasted image 20260622032830.png]]

-بروح اكتب البيانات في ال cache and data source synchronously ولو حصل failure  في اي منهم الاتنين هيفشلوا 
-> benefits 
![[Pasted image 20260622032953.png]]

-no stale data because we always will update caching so it will not retrieve 
an old data 

-its suitable for Read-After-Write consistency use cases 



# 2- Write Behind
![[Pasted image 20260622033314.png]]