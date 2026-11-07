يعني هنكتب ال tenant id in the request header ,
ي Aspect لوجيك اضافي بيتنفذ عند نقاط معينه 
@FilterDef(// بنعرف الفلتر 
name = "tenantFilter",
parameters = @ParamDef(
name ="tenantId",
type=String.Class
)
defaultCondition +

)



يعني اي thread and threadlocal 