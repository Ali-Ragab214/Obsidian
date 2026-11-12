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
TenantContext المسؤول عن تخزين ال current tenant 
get ,set ,clear 
TenantFilter  => قراءه ال  tenant_id 
وتخزينه في ال tenant context <<>>
TenantHibernateFilter => activate hibernate filter  عشان نرجع بيانات ال current tenant 


TenantContext :to store current tenant 
TenantFilter:read tenant id and store it in tenant context 
HibernateTenantFilter: activate hibernate filter and apply the filter on the retrieved data

```java 
public class TenantContext 
{
public static final ThreadLocal<String> CURRENT_TENANT = new ThreadLocal<>();


public static void setCurrentTenant(final String tenant)
{
CURRENT_TENANT.set(tenant);
}


public static String getCurrentTenant()
{
return CURRENT_TENANT.get();
}

public static void clearCurrentTenant() 
{

CURRENT_TENANT.remove();
}
}
```

%s
%
println => normal paste 
printf للطباعه بفورمات معين 
String.format 
String hour = 30 ? "passed" : "not passed";