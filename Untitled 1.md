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