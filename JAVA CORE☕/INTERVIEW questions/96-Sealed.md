بص يا صاحبي، الـ `sealed` classes في Java من الحاجات المهمة في الـ OOP، خصوصًا لما تفهم علاقتها بالـ Inheritance والـ Polymorphism.

الفكرة ببساطة: إزاي تتحكم في مين مسموح له يعمل `extends` أو `implements` للـ class أو interface بتاعتك؟

## 1. المشكلة اللي الـ `sealed` بتحلها — Why?

تخيّل إن عندك `Payment` class، وعندك نوعين بس من الـ payments مسموح بيهم في السيستم:

- `CashPayment`
    
- `CardPayment`
    

من غير `sealed`، أي حد يقدر يعمل subclass جديدة:

Java

```
class Payment { }

class CashPayment extends Payment { }

class CardPayment extends Payment { }

class CryptoPayment extends Payment { } // Allowed
```

المشكلة إنك ممكن تكون عايز تحصر أنواع الـ Payment في أنواع محددة، عشان تحافظ على business rules وتمنع الـ inheritance غير المرغوب فيه.

هنا بييجي دور `sealed`.

## 2. يعني إيه Sealed Class؟

الـ `sealed` class هي class بتسمح لك تحدد مين بالضبط مسموح له يعمل inheritance منها.

Payment

sealed class

CashPayment

Permitted

CardPayment

Permitted

الـ subclasses المسموح بيها بس هي اللي تقدر تعمل direct inheritance من Payment.

### Example

Java

```
public sealed class Payment
        permits CashPayment, CardPayment {
}

public final class CashPayment extends Payment {
}

public final class CardPayment extends Payment {
}
```

دلوقتي:

Java

```
class CryptoPayment extends Payment {
    // Compilation Error
}
```

ليه؟ لأن `CryptoPayment` مش موجودة في قائمة الـ `permits`.

ملحوظة: الـ `permits` بتحدد الـ direct subclasses فقط، مش كل الـ descendants في شجرة الـ inheritance.

## 3. أهم Rule: كل Subclass لازم تحدد مصير الـ Inheritance بتاعها

دي أهم نقطة في الموضوع كله، وغالبًا بتيجي في أسئلة الـ interviews.

أي class مباشرةً بترث من `sealed class` لازم تختار واحد من 3 modifiers:

- `final`
    ابعت 
- `sealed`
    
- `non-sealed`
    

### الحالة الأولى: `final`

يعني ممنوع أي class تانية تورث من الـ subclass دي.

Java

```
public sealed class Payment
        permits CashPayment, CardPayment {
}

public final class CashPayment extends Payment {
}

public final class CardPayment extends Payment {
}
```

هنا `CashPayment` و`CardPayment` هما آخر مستوى في الـ inheritance بالنسبة للفروع دي.

Java

```
class OnlineCashPayment extends CashPayment {
    // Compilation Error: CashPayment is final
}
```

### الحالة الثانية: `sealed`

يعني الـ subclass نفسها هتسمح بأنواع محددة من الـ subclasses تورث منها.

Java

```
public sealed class Payment
        permits CashPayment {
}

public sealed class CashPayment extends Payment
        permits OnlineCashPayment {
}

public final class OnlineCashPayment extends CashPayment {
}
```

الشجرة هنا:

`Payment` — sealed

`CashPayment` — sealed

`OnlineCashPayment` — final

### الحالة الثالثة: `non-sealed`

يعني الـ subclass بتخرج من القيود دي، وتسمح لأي class إنها تورث منها.

Java

```
public sealed class Payment
        permits CashPayment {
}

public non-sealed class CashPayment extends Payment {
}

class OnlineCashPayment extends CashPayment {
}

class MobileCashPayment extends CashPayment {
}
```

لاحظ الفرق: مش مسموح لـ `CryptoPayment` إنها تعمل `extends Payment` مباشرةً، لكن مسموح تعمل `extends CashPayment` لأن `CashPayment` بقت `non-sealed`.

احفظ القاعدة دي:

|Modifier|معناه|
|---|---|
|`final`|اقفل الـ inheritance نهائيًا|
|`sealed`|اسمح بأنواع محددة من الـ subclasses|
|`non-sealed`|افتح الـ inheritance لأي subclass|

## 4. هل ينفع أستخدم `sealed` مع Interface؟

أيوه، ينفع مع الـ `interface` كمان.

Java

```
public sealed interface Shape
        permits Circle, Rectangle {
    double area();
}

public final class Circle implements Shape {
    @Override
    public double area() {
        return Math.PI * 5 * 5;
    }
}

public final class Rectangle implements Shape {
    @Override
    public double area() {
        return 10 * 5;
    }
}
```

هنا الـ `Shape` بتحدد إن الأنواع اللي مسموح لها تعمل `implements` مباشرةً هي `Circle` و`Rectangle` فقط.

وبالتالي:

Java

```
class Triangle implements Shape {
    // Compilation Error
}
```

السبب: `Triangle` مش ضمن الـ permitted direct implementations.

## 5. Why use Sealed? — إمتى أستخدمها؟

تخيّل إنك بتعمل Spring Boot application، وعندك نتيجة عملية دفع ممكن تكون واحدة من 3 حالات معروفة:

Java

```
public sealed interface PaymentResult
        permits Success, Failure, Pending {
}

public record Success(String transactionId)
        implements PaymentResult {
}

public record Failure(String reason)
        implements PaymentResult {
}

public record Pending(String transactionId)
        implements PaymentResult {
}
```

هنا استخدام `sealed` مفيد لأن أنواع النتائج محددة ومعروفة.

من المميزات المهمة إن Java تقدر تستفيد من النوع ده في الـ pattern matching، وتساعدك تتأكد إنك بتتعامل مع كل الحالات المتوقعة.

مثال باستخدام `switch` في Java 21:

Java

```
static String handle(PaymentResult result) {
    return switch (result) {
        case Success s -> "Payment completed";
        case Failure f -> "Payment failed";
        case Pending p -> "Payment pending";
    };
}
```

الـ `switch` هنا exhaustive، يعني غطّى كل الأنواع المسموح بيها، ومش محتاج `default` في المثال ده.

لو ضفت نوع جديد مسموح به للـ `PaymentResult`، فالـ compiler يقدر ينبهك إن الـ `switch` محتاج يتحدّث، لو الـ pattern matching مش مغطّي النوع الجديد.

### فوائدها الأساسية

- Controlled inheritance: تتحكم في الأنواع اللي تقدر ترث من الـ parent.
    
- Safer domain modeling: تمنع إنشاء subclasses غير متوقعة.
    
- Better pattern matching: تساعد الـ compiler في التأكد من تغطية الحالات.
    
- Clear design: أي developer يقرأ الكود يعرف الأنواع المسموح بيها.
    

## 6. Why NOT use Sealed? — إمتى أتجنبها؟

مش معنى إن `sealed` مفيدة إننا نستخدمها في كل مكان.

ما تستخدمهاش لو:

1. عايز extensibility مفتوحة: لو بتعمل library وعايز developers يقدروا يعملوا implementations خاصة بيهم، الـ `sealed` ممكن تمنعهم.
    
2. الأنواع بتتغير باستمرار: كل ما تضيف subclass جديدة، ممكن تحتاج تعدّل الـ `permits` والـ `switch` statements المرتبطة بيها.
    
3. مفيش احتياج فعلي لتقييد الـ inheritance: لو الـ `abstract class` أو الـ `interface` العادية كفاية، مش محتاج تزود قيود من غير سبب.
    

مثال: لو بتصمم `Notification` interface لمكتبة عامة، ومحتاج أي developer يقدر يضيف `EmailNotification` أو `WhatsAppNotification`، فالـ interface العادية غالبًا أنسب من `sealed interface`.

## 7. قواعد مهمة للـ Java Interview

- الـ `sealed` اتقدمت كميزة نهائية في Java 17.
    
- لازم تحدد الـ permitted direct subclasses باستخدام `permits`، إلا لو الأنواع واضحة للـ compiler من الـ source file نفسه.
    
- الـ permitted subclasses لازم تكون في نفس الـ module؛ ولو الكود في unnamed module، لازم تكون في نفس الـ package.
    
- كل direct subclass لازم تكون `final` أو `sealed` أو `non-sealed`.
    
- الـ `sealed class` عادي جدًا تعمل منها reference وتستخدم الـ polymorphism:
    

Java

```
Payment payment = new CashPayment();
```

ده صحيح، بشرط إن `CashPayment` نوع مسموح به ومتاح للإنشاء.

## 8. اختبر نفسك

1. Can a class extend a sealed class without being listed in permits?
    

Yes, always

No, not as a direct subclass

Only if it is abstract

2. What does non-sealed mean?
    

The class cannot be instantiated

The class cannot have subclasses

The class allows unrestricted further inheritance

3. Can we create a reference of a sealed abstract class?
    

Yes, pointing to an appropriate concrete subclass

No, sealed classes cannot have references

Only with static methods

Try again

الخلاصة يا صاحبي: الـ `abstract` بتسمح لك تعمل inheritance مع إمكانية وجود methods من غير implementation، أما الـ `sealed` فبتحدد مين مسموح له يعمل inheritance. الاتنين ممكن يجتمعوا في نفس الـ class، وكل واحد ليه وظيفة مختلفة.