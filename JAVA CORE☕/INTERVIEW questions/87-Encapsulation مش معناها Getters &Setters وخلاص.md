


دي نقطة مهمة في الانترفيو.

Encapsulation معناها إنك:

> ==**تجمع الـ state والbehavior المرتبطين ببعض داخل object، وتتحكم في الوصول للـ internal state.**==

يعني بدل:

```java
account.balance = -100;
```

تخلي الـ object هو اللي يتحكم:

```java
account.withdraw(100);
```

وهو يقرر هل العملية valid ولا لأ.