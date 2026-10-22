
this is an example about variable hiding 


```java
class Bike { int speedLimit = 90; }
class Honda3 extends Bike { int speedLimit = 150; }

Bike obj = new Honda3();
System.out.println(obj.speedLimit);           // 90  (Reference = Bike)
System.out.println(((Honda3) obj).speedLimit); // 150 (بعد الـ cast، Reference بقى Honda3)
```


the rule is that the variable follow refrence ,but the method follow object 

### مثال 8: اختلاف الـ Type مع نفس الاسم



java

```java
class Bike { int speedLimit = 90; }
class Honda3 extends Bike { String speedLimit = "fast"; }

Bike b = new Honda3();
Honda3 h = new Honda3();

System.out.println(b.speedLimit); // 90
System.out.println(h.speedLimit); // fast
```

الـ hiding بيشتغل حتى لو الـ type مختلف تماماً، لأن المهم هو **الاسم**. مهمه 