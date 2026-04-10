الفكرة هنا: هنبني مثال **Home Theater System** — نظام سينما البيت. الكود بتاعك بدل ما يشغّل كل جهاز لوحده، بيقول للـ `HomeTheaterFacade` كلمة واحدة وهو بيتكلم مع كل الأجهزة.

هنشوف الستراكشر الأول، وبعدين الكود كامل.دلوقتي الكود الكامل خطوة بخطوة:

---

##  تقليل الـ Coupling 🔗

الـ Controller أو الـ Client:

❌ قبل Facade:

- مرتبط بـ 4 أو 5 services

✔️ بعد Facade:

- مرتبط بكلاس واحد فقط

👉 ده مهم جدًا في الـ Clean Architecture

## سهولة التعديل بدون كسر النظام 🔧

لو غيرت:

- طريقة الدفع
- طريقة الشحن
- أو أي Service

مش هتلمس الـ Controller  
بس تعدل داخل الـ Facade أو الخدمات نفسها
![[Pasted image 20260410115450.png]]

![[Pasted image 20260410115322.png]]
## الخطوة 1 — الـ Subsystem Classes (الأنظمة الداخلية)

دي الكلاسات المعقدة اللي الـ Facade هيتعامل معاها بدلًا منك.

```java
// نظام الصوت
public class Amplifier {
    public void on() {
        System.out.println("Amplifier: تشغيل");
    }
    public void setVolume(int level) {
        System.out.println("Amplifier: الصوت على " + level);
    }
    public void off() {
        System.out.println("Amplifier: إيقاف");
    }
}
```

```java
// مشغل الأفلام
public class DVDPlayer {
    public void on() {
        System.out.println("DVDPlayer: تشغيل");
    }
    public void play(String movie) {
        System.out.println("DVDPlayer: يشغّل فيلم -> " + movie);
    }
    public void stop() {
        System.out.println("DVDPlayer: وقف");
    }
    public void off() {
        System.out.println("DVDPlayer: إيقاف");
    }
}
```

```java
// البروجيكتور
public class Projector {
    public void on() {
        System.out.println("Projector: تشغيل");
    }
    public void wideScreenMode() {
        System.out.println("Projector: وضع الشاشة العريضة");
    }
    public void off() {
        System.out.println("Projector: إيقاف");
    }
}
```

```java
// الإضاءة
public class TheaterLights {
    public void dim(int level) {
        System.out.println("Lights: تعتيم على " + level + "%");
    }
    public void on() {
        System.out.println("Lights: إضاءة كاملة");
    }
}
```

---

## الخطوة 2 — الـ Facade Class ⭐

دي أهم حاجة — بتمسك الـ subsystems كلها وبتوفر methods بسيطة.

```java
public class HomeTheaterFacade {

    // بيحتفظ بمرجع لكل نظام
    private Amplifier amp;
    private DVDPlayer dvd;
    private Projector projector;
    private TheaterLights lights;

    // بياخدهم في الـ constructor
    public HomeTheaterFacade(Amplifier amp,
                              DVDPlayer dvd,
                              Projector projector,
                              TheaterLights lights) {
        this.amp       = amp;
        this.dvd       = dvd;
        this.projector = projector;
        this.lights    = lights;
    }

    // ✅ method واحدة بسيطة بدل 7 استدعاءات
    public void watchMovie(String movie) {
        System.out.println("\n--- جهّز نفسك للفيلم! ---");
        lights.dim(10);          // عتّم الأضواء
        projector.on();          // شغّل البروجيكتور
        projector.wideScreenMode(); // وضع سينما
        amp.on();                // شغّل الصوت
        amp.setVolume(80);       // ارفع الصوت
        dvd.on();                // شغّل الـ DVD
        dvd.play(movie);         // العب الفيلم
    }

    // ✅ method واحدة تقفل كل حاجة
    public void endMovie() {
        System.out.println("\n--- تصبح على خير! ---");
        dvd.stop();
        dvd.off();
        amp.off();
        projector.off();
        lights.on();             // رجّع الإضاءة
    }
}
```

---

## الخطوة 3 — الـ Client (كودك)

```java
public class Main {
    public static void main(String[] args) {

        // بنعمل الأجهزة
        Amplifier    amp       = new Amplifier();
        DVDPlayer    dvd       = new DVDPlayer();
        Projector    projector = new Projector();
        TheaterLights lights   = new TheaterLights();

        // بنعمل الـ Facade ونديله كل الأجهزة
        HomeTheaterFacade homeTheater =
            new HomeTheaterFacade(amp, dvd, projector, lights);

        // ✅ استدعاء واحد بسيط — مش محتاج تعرف أي تفاصيل
        homeTheater.watchMovie("Inception");

        // ... شوف الفيلم ...

        homeTheater.endMovie();
    }
}
```

---

## الـ Output

```
--- جهّز نفسك للفيلم! ---
Lights: تعتيم على 10%
Projector: تشغيل
Projector: وضع الشاشة العريضة
Amplifier: تشغيل
Amplifier: الصوت على 80
DVDPlayer: تشغيل
DVDPlayer: يشغّل فيلم -> Inception

--- تصبح على خير! ---
DVDPlayer: وقف
DVDPlayer: إيقاف
Amplifier: إيقاف
Projector: إيقاف
Lights: إضاءة كاملة
```

---

## الفايدة بالظبط

|بدون Facade|مع Facade|
|---|---|
|7 استدعاءات في كل مرة|استدعاء واحد `watchMovie()`|
|الكود يعرف تفاصيل كل جهاز|الكود يعرف الـ Facade بس|
|لو غيّرت جهاز، بتغيّر في كل الكود|بتغيّر جوه الـ Facade بس|

الـ Facade مش بيمنعك من الوصول للأجهزة مباشرة لو احتجت — بس بيوفرلك الطريق السهل.