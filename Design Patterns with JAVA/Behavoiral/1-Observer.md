 مهم للميكروسيرفيز
## الـ Observer Pattern — هو إيه؟

تخيّل إنك اشتركت في قناة يوتيوب. كل ما صاحب القناة نزّل فيديو جديد، أنت وكل المشتركين بيجيلكم إشعار تلقائي. ما بتحتاجش تفضل تدخل القناة كل شوية تتأكد.

ده بالظبط الـ Observer Pattern.

- **القناة** = الـ `Subject` (اللي بيحصل فيه التغيير)
- **المشتركين** = الـ `Observers` (اللي عايزين يعرفوا بأي تغيير)
- **الإشعار** = الـ `notify()` (بيحصل أوتوماتيك)

---

## بيحل مشكلة إيه؟

لو عندك object بيتغيّر حالته، وعندك objects تانية محتاجة تعرف بالتغيير ده — من غير ما يكون في تبعية صلبة بينهم. بدل ما كل object يسأل "اتغيّر حاجة؟" كل شوية (**polling** — مضيعة وقت)، الـ Subject هو اللي يخبرهم تلقائي لما يحصل تغيير.

---

الستراكشر الأول، وبعدين الكود بالتفصيل.المثال اللي هنبنيه: **نظام البورصة**. لما سعر السهم بيتغير، كل اللي مشتركين (موبايل، إيميل) بيتبعتلهم إشعار تلقائي.

# ✅ التعريف الصح

🔹 **Subject (Publisher)**

> هو الكائن اللي **بيحصل عنده التغيير** وبيبلغ غيره

🔹 **Observer (Subscriber)**

> هو الكائن اللي **مهتم بالتغيير ده وبيستقبل إشعار لما يحصل**

---

# 🧠 بصياغة أبسط

- **Subject** = مصدر الحدث 🔔
- **Observer** = اللي بيسمع الحدث 👂

---

## الخطوة 1 — الـ Observer Interface

ده عقد بيقول: "أي حاجة عايزة تتابع التغييرات، لازم تعمل `update()`"

```java
public interface Observer {
    // بيتنادى تلقائي لما يحصل تغيير
    void update(String stockName, double price);
}
```

---

## الخطوة 2 — الـ Subject Interface

بيعرّف ازاي الـ observers بيشتركوا ويتشالوا ويتبعتلهم إشعار.

```java
public interface Subject {
    void registerObserver(Observer o);   // اشترك
    void removeObserver(Observer o);     // إلغاء اشتراك
    void notifyObservers();              // بعّت إشعار للكل
}
```

---

## الخطوة 3 — الـ Concrete Subject (البورصة)

```java
import java.util.ArrayList;
import java.util.List;

public class StockMarket implements Subject {

    // قايمة بكل المشتركين
    private List<Observer> observers = new ArrayList<>();

    private String stockName;
    private double price;

    public StockMarket(String stockName) {
        this.stockName = stockName;
    }

    // ✅ اشتراك
    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
        System.out.println("مشترك جديد انضم لمتابعة: " + stockName);
    }

    // ✅ إلغاء اشتراك
    @Override
    public void removeObserver(Observer o) {
        observers.remove(o);
        System.out.println("مشترك غادر متابعة: " + stockName);
    }

    // ✅ بيخبر كل المشتركين بالتغيير
    @Override
    public void notifyObservers() {
        for (Observer o : observers) {
            o.update(stockName, price);  // كل واحد بياخد التغيير
        }
    }

    // ⭐ ده اللي بيحصل التغيير فيه
    // لما السعر يتغير → notifyObservers بيتنادى تلقائي
    public void setPrice(double newPrice) {
        System.out.println("\n--- تغيّر سعر " + stockName + " إلى: " + newPrice + " ---");
        this.price = newPrice;
        notifyObservers();  // أخبر الكل فوراً
    }
}
```

---

## الخطوة 4 — الـ Concrete Observers

كل observer بيرد على التغيير بطريقته الخاصة.

```java
// Observer 1: إشعار الموبايل
public class MobileApp implements Observer {

    private String userName;

    public MobileApp(String userName) {
        this.userName = userName;
    }

    @Override
    public void update(String stockName, double price) {
        System.out.println("[Notification] يا " + userName +
                           "! سعر " + stockName + " = " + price);
    }
}
```

```java
// Observer 2: تنبيه الإيميل
public class EmailAlert implements Observer {

    private String email;

    public EmailAlert(String email) {
        this.email = email;
    }

    @Override
    public void update(String stockName, double price) {
        System.out.println("[Email -> " + email + "] " +
                           "تحديث: " + stockName + " وصل لـ " + price);
    }
}
```

---

## الخطوة 5 — الـ Main (اللي بيشغّل كل حاجة)

```java
public class Main {
    public static void main(String[] args) {

        // البورصة (الـ Subject)
        StockMarket apple = new StockMarket("AAPL");

        // المشتركين (الـ Observers)
        MobileApp  ahmed  = new MobileApp("Ahmed");
        MobileApp  sara   = new MobileApp("Sara");
        EmailAlert trader = new EmailAlert("trader@gmail.com");

        // ✅ اشتركوا
        apple.registerObserver(ahmed);
        apple.registerObserver(sara);
        apple.registerObserver(trader);

        // ✅ لما السعر يتغير → الكل بياخد إشعار تلقائي
        apple.setPrice(150.5);
        apple.setPrice(162.0);

        // ✅ Sara ألغت اشتراكها
        System.out.println("\n--- Sara ألغت الاشتراك ---");
        apple.removeObserver(sara);

        // دلوقتي بس Ahmed والـ trader اللي بياخدوا إشعار
        apple.setPrice(145.0);
    }
}
```

---

## الـ Output

```
مشترك جديد انضم لمتابعة: AAPL
مشترك جديد انضم لمتابعة: AAPL
مشترك جديد انضم لمتابعة: AAPL

--- تغيّر سعر AAPL إلى: 150.5 ---
[Notification] يا Ahmed! سعر AAPL = 150.5
[Notification] يا Sara! سعر AAPL = 150.5
[Email -> trader@gmail.com] تحديث: AAPL وصل لـ 150.5

--- تغيّر سعر AAPL إلى: 162.0 ---
[Notification] يا Ahmed! سعر AAPL = 162.0
[Notification] يا Sara! سعر AAPL = 162.0
[Email -> trader@gmail.com] تحديث: AAPL وصل لـ 162.0

--- Sara ألغت الاشتراك ---
مشترك غادر متابعة: AAPL

--- تغيّر سعر AAPL إلى: 145.0 ---
[Notification] يا Ahmed! سعر AAPL = 145.0
[Email -> trader@gmail.com] تحديث: AAPL وصل لـ 145.0
```

---

## الفايدة الحقيقية

الـ `StockMarket` مش بيعرف إن المشترك موبايل ولا إيميل ولا أي حاجة تانية. هو بس بيعرف إنهم `Observer`. لو حبيت تضيف observer جديد — مثلًا `SMSAlert` — بتعمل الكلاس وبتعمله `registerObserver` من غير ما تغيّر سطر واحد في `StockMarket`. ده اللي بيسموه **Open/Closed Principle** — مفتوح للإضافة، مقفول للتعديل.
 