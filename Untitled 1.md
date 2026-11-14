اي هو ال servlet => 
java مجرد لغه برمجه مش هتعرف تتعامل مع ال http requests 
ولكن اللي بيتعامل مع ال http requests is the servlet specification 
request-> server -> servlet (dispatcher servlet) -> endpoint 
 وال dispatcher دي is a servlet and named in this way عشان بتوزع كل  request علي مكانه الصح front controller pattern 



web server is an app that recieves a requests from the internet 
ننقل الداتا من موديول لموديول او نستقبل او نعرض للكلاينت

```java

```

input validation ,business logic,authentication,db constraints 

يعني الثريد لما الريكوست يوصله 
we have 
TenantContext:to store tenant id ,and clear it from threadlocal
TenantFilter:to get the tenantid from the header of the request 
HibernateTenantFilter:to make database querors 

---
class<=>table
object<=> row
field<=>column
jpa is a specification 
hibernate is an implementation 
jpa => specification like interfaces and annotations has no implementation code 
Jpa => is the specification for the orm defining the interfaces and the annotations ,Hibernate is the actual orm provider that implements jpa 
sdjpa is an additional layer that works abovehibernate and jpa
----
query parser that read the name and convert it into sql  بنشتق السيكوال من اسم الميثود 

derived query methods allows spring data jpa to parse repository method name int sql queries through keywords like (findby,orderby ,or and and
)

@Query لما الكويري تكون معقده  وفيه joins كتير 
JPQL 
---
-------
native sql :we target database columns and write native sql queries 
jpql :we target java entites and attributes  and wirte native sql too   وبنستخدم معاهم الاتنين 
@Query annotation
اللي بنستخدمها لما تكون الكويري معقده وفي joins كتير 
----------------
---

N+1 problem is a performance problem ,imagine that we have one user with 100 orders and we need to fetch the user with its orders 
it will be one query for fetch the user and a query for each order so ==🔴we will have 101 queries==  

how to solve it ? 
first approach through Join Fetch in JPQL 
@Query("---")
sec approach through :
@EntityGraph(attributePaths ={"orders"})
List<User> findAll(); 
هيرجع الاوردرز مع اليوزرس

----
all these systems has a related data 

CREATE INDEX idx_name
ON  users(Email);
O (log n) instead of O(n)

SELECT  * 
FROM  users u 
join Orders o 
ON  u.id = o.userId

INNER ,LEFT ,RIGHT , FULL 

set of processes  either all of them executed successfully or none of them executed (Rollback)


link a table with another table 