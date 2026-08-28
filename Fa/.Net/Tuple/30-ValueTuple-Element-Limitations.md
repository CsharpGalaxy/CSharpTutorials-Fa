# آموزش جامع Tuple در Switch Expression در C#

به مستندات آموزشی این Repository خوش آمدید! در این مقاله، یکی از جذاب‌ترین و کاربردی‌ترین ویژگی‌های مدرن C#، یعنی **استفاده از Tuple در Switch Expression** را بررسی می‌کنیم. این تکنیک به شما کمک می‌کند تا منطق‌های شرطی پیچیده و چندمتغیره را به شکلی بسیار تمیز، خوانا و بهینه بنویسید.

---

## 📑 فهرست مطالب
- [سوئیچ اکسپرشن (Switch Expression) چیست؟](#سوئیچ-اکسپرشن-چیست)
- [چرا Tuple برای Switch Expression مناسب است؟](#چرا-tuple-برای-switch-expression-مناسب-است)
- [الگوی Tuple (Tuple Pattern) و مقایسه چند مقدار](#الگوی-tuple-و-مقایسه-چند-مقدار)
- [مثال ساده: بازی سنگ، کاغذ، قیچی](#مثال-ساده-بازی-سنگ-کاغذ-قیچی)
- [مثال متوسط: چراغ راهنمایی و عابر پیاده](#مثال-متوسط-چراغ-راهنمایی-و-عابر-پیاده)
- [مثال واقعی: محاسبه تخفیف فروشگاه اینترنتی](#مثال-واقعی-محاسبه-تخفیف-فروشگاه-اینترنتی)
- [ترکیب Tuple با Pattern Matching و Guards (when)](#ترکیب-tuple-با-pattern-matching-و-guards)
- [خوانایی کد و مقایسه با if/else](#خوانایی-کد-و-مقایسه-با-ifelse)
- [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
- [جمع‌بندی](#جمع‌بندی)
- [منابع رسمی](#منابع-رسمی)

---

<a id="سوئیچ-اکسپرشن-چیست"></a>
## سوئیچ اکسپرشن (Switch Expression) چیست؟

از نسخه C# 8.0، عبارت `switch` به شکل یک **Expression** (عبارت) درآمد. این یعنی برخلاف `switch statement` قدیمی که فقط برای کنترل جریان برنامه (Control Flow) استفاده می‌شد، `switch expression` مستقیماً یک **مقدار را تولید و برمی‌گرداند**.

سینتکس آن از فلش `=>` برای جداسازی الگو از مقدار بازگشتی استفاده می‌کند و بسیار فشرده‌تر از نسخه قدیمی است:

```csharp
string GetCalendarDay(DayOfWeek day) => day switch
{
    DayOfWeek.Saturday => "تعطیل است 🎉",
    DayOfWeek.Friday => "آخرین روز کاری",
    _ => "روز کاری معمولی"
};
```

<a id="چرا-tuple-برای-switch-expression-مناسب-است"></a>
## چرا Tuple برای Switch Expression مناسب است؟

در دنیای واقعی، تصمیم‌گیری‌های برنامه اغلب به **یک متغیر** وابسته نیستند. ممکن است بخواهید بر اساس ترکیبی از چند متغیر تصمیم بگیرید (مثلاً: وضعیت کاربر + مبلغ سبد خرید + روز هفته). 

در گذشته برای این کار باید از `if/else` های تو در تو یا عملگرهای منطقی (`&&`, `||`) استفاده می‌کردیم که کد را زشت و ناخوانا می‌کرد. 
**Tupleها** (تاپل‌ها) به ما اجازه می‌دهند چندین مقدار را در یک بسته واحد قرار دهیم و `Switch Expression` نیز به صورت ذاتی از **Tuple Pattern** پشتیبانی می‌کند. این ترکیب، بهشت برنامه‌نویسان برای پیاده‌سازی ماشین‌های حالت (State Machines) و منطق‌های چندشرطی است.

<a id="الگوی-tuple-و-مقایسه-چند-مقدار"></a>
## الگوی Tuple و مقایسه چند مقدار

شما می‌توانید چند متغیر را در یک Tuple قرار داده و آن‌ها را در `switch` بررسی کنید:

```csharp
var result = (variable1, variable2) switch
{
    (value1, value2) => "نتیجه اول",
    (value3, _) => "نتیجه دوم (_ یعنی هر مقداری قبول است)",
    _ => "حالت پیش‌فرض"
};
```
این ساختار به کامپایلر می‌گوید: «مقدار متغیر اول را با الگوی اول مقایسه کن، سپس متغیر دوم را...».

---

<a id="مثال-ساده-بازی-سنگ-کاغذ-قیچی"></a>
## مثال ساده: بازی سنگ، کاغذ، قیچی

بیایید منطق برنده شدن در بازی سنگ، کاغذ، قیچی را پیاده‌سازی کنیم. در اینجا دو ورودی (حرکت بازیکن اول و دوم) داریم.

```csharp
public enum Move { Rock, Paper, Scissors }

public string DetermineWinner(Move player1, Move player2)
{
    return (player1, player2) switch
    {
        (Move.Rock, Move.Paper) => "بازیکن دوم برد (کاغذ سنگ را می‌پوشاند)",
        (Move.Rock, Move.Scissors) => "بازیکن اول برد (سنگ قیچی را خراب می‌کند)",
        (Move.Paper, Move.Rock) => "بازیکن اول برد",
        (Move.Paper, Move.Scissors) => "بازیکن دوم برد",
        (Move.Scissors, Move.Rock) => "بازیکن دوم برد",
        (Move.Scissors, Move.Paper) => "بازیکن اول برد",
        (var p1, var p2) when p1 == p2 => "مساوی!",
        _ => "خطای غیرمنتظره"
    };
}
```

**توضیح نتیجه اجرا:**
- اگر فراخوانی `DetermineWinner(Move.Rock, Move.Scissors)` باشد، کامپایلر به دنبال `(Rock, Scissors)` می‌گردد و رشته `"بازیکن اول برد..."` را برمی‌گرداند.
- اگر هر دو بازیکن `Move.Paper` را انتخاب کنند، هیچ‌کدام از کیس‌های ثابت تطابق ندارند، اما کیس `(var p1, var p2) when p1 == p2` تطابق پیدا کرده و `"مساوی!"` برگردانده می‌شود.

---

<a id="مثال-متوسط-چراغ-راهنمایی-و-عابر-پیاده"></a>
## مثال متوسط: چراغ راهنمایی و عابر پیاده

فرض کنید یک سیستم هوشمند ترافیکی داریم که باید بر اساس **رنگ چراغ** و **وضعیت حضور عابر پیاده** تصمیم بگیرد خودرو چه کند.

```csharp
public enum TrafficLight { Red, Yellow, Green }

public string GetCarAction(TrafficLight light, bool isPedestrianCrossing)
{
    return (light, isPedestrianCrossing) switch
    {
        (TrafficLight.Red, _) => "توقف کامل 🛑",
        (TrafficLight.Green, false) => "حرکت با سرعت مجاز 🟢",
        (TrafficLight.Green, true) => "احتیاط، عابر پیاده در حال عبور است ⚠️",
        (TrafficLight.Yellow, _) => "آماده باش برای توقف 🟡",
        _ => "سیستم در حالت خطا قرار دارد"
    };
}
```

**توضیح نتیجه اجرا:**
- اگر `(TrafficLight.Red, true)` باشد، چون در خط اول `(TrafficLight.Red, _)` نوشته‌ایم (علامت `_` یعنی هر مقداری برای متغیر دوم قبول است)، ماشین باید توقف کند.
- اگر `(TrafficLight.Green, false)` باشد، ماشین حرکت می‌کند.
- ترتیب قرارگیری کیس‌ها مهم است؛ اگر `_` را در خط اول نمی‌گذاشتیم و به جای آن `(Red, true)` و `(Red, false)` را جداگانه می‌نوشتیم، کد طولانی‌تر می‌شد.

---

<a id="مثال-واقعی-محاسبه-تخفیف-فروشگاه-اینترنتی"></a>
## مثال واقعی: محاسبه تخفیف فروشگاه اینترنتی

در یک فروشگاه، میزان تخفیف به **نوع کاربر** (عادی، VIP، سازمانی) و **مبلغ سبد خرید** بستگی دارد.

```csharp
public decimal CalculateDiscount(string userType, decimal cartTotal)
{
    return (userType, cartTotal) switch
    {
        ("VIP", > 1000) => 0.20m,      // 20% تخفیف
        ("VIP", > 500) => 0.15m,       // 15% تخفیف
        ("VIP", _) => 0.10m,           // 10% تخفیف
        
        ("Organization", > 5000) => 0.30m, // 30% تخفیف
        ("Organization", _) => 0.25m,
        
        ("Normal", > 2000) => 0.05m,   // 5% تخفیف
        _ => 0.0m                      // بدون تخفیف
    };
}
```

**توضیح نتیجه اجرا:**
- اگر `userType` برابر `"VIP"` و `cartTotal` برابر `1500` باشد، شرط `("VIP", > 1000)` اولین تطابق است و `0.20` (بیست درصد) برمی‌گردد.
- در اینجا ما از **Relational Patterns** (`> 1000`) درون Tuple استفاده کرده‌ایم که نشان‌دهنده قدرت بی‌نظیر ترکیب Tuple و Pattern Matching است.

---

<a id="ترکیب-tuple-با-pattern-matching-و-guards"></a>
## ترکیب Tuple با Pattern Matching و Guards (when)

همانطور که در مثال‌های قبل دیدید، Tupleها به تنهایی قدرتمندند، اما وقتی با **Pattern Matching** و **Guards (کلمه کلیدی `when`)** ترکیب می‌شوند، جادو می‌کنند!

### 1. استفاده از Relational و Type Patterns در Tuple
شما می‌توانید داخل پرانتز Tuple از عملگرهای مقایسه‌ای یا بررسی نوع استفاده کنید:
```csharp
var result = (age, income) switch
{
    (< 18, _) => "خردسال",
    (>= 18 and <= 65, > 5000) => "بزرگسال با درآمد بالا",
    _ => "سایر"
};
```

### 2. استفاده از Guard (عبارت when)
اگر منطق شما پیچیده‌تر از آن است که در الگو جا شود، از `when` استفاده کنید:
```csharp
public string CheckCoordinates(int x, int y)
{
    return (x, y) switch
    {
        (0, 0) => "نقطه مبدأ",
        (var a, var b) when a == b => "روی خط y = x قرار دارد",
        (var a, var b) when Math.Abs(a) == Math.Abs(b) => "روی خط y = -x یا y = x قرار دارد",
        _ => "یک نقطه تصادفی"
    };
}
```
**توضیح:** در اینجا ما مقادیر را در `a` و `b` ذخیره کرده (`var`) و سپس با `when` شرط‌های ریاضی دلخواه را روی آن‌ها بررسی می‌کنیم.

---

<a id="خوانایی-کد-و-مقایسه-با-ifelse"></a>
## خوانایی کد و مقایسه با if/else

بیایید منطق مثال چراغ راهنمایی را با `if/else` بنویسیم تا تفاوت را احساس کنید:

**❌ روش سنتی (if/else):**
```csharp
public string GetCarActionOld(TrafficLight light, bool isPedestrian)
{
    if (light == TrafficLight.Red)
        return "توقف کامل 🛑";
    else if (light == TrafficLight.Green && !isPedestrian)
        return "حرکت با سرعت مجاز 🟢";
    else if (light == TrafficLight.Green && isPedestrian)
        return "احتیاط، عابر پیاده ⚠️";
    else if (light == TrafficLight.Yellow)
        return "آماده باش 🟡";
    else
        return "خطا";
}
```

**✅ روش مدرن (Tuple + Switch Expression):**
```csharp
public string GetCarActionNew(TrafficLight light, bool isPedestrian) => (light, isPedestrian) switch
{
    (TrafficLight.Red, _) => "توقف کامل 🛑",
    (TrafficLight.Green, false) => "حرکت با سرعت مجاز 🟢",
    (TrafficLight.Green, true) => "احتیاط، عابر پیاده ⚠️",
    (TrafficLight.Yellow, _) => "آماده باش 🟡",
    _ => "خطا"
};
```

**مزایای روش مدرن:**
1. **حذف نویز (Noise):** کلماتی مثل `if`, `else`, `return`, `&&` حذف شده‌اند.
2. **تمرکز بر داده‌ها:** چشم برنامه‌نویس مستقیماً ترکیب مقادیر `(Red, _)` را می‌بیند.
3. **تطابق خط به خط:** هر کیس دقیقاً یک خط است و خوانایی آن به شدت بالاست.

---

<a id="نکات-مهم-و-اشتباهات-رایج"></a>
## نکات مهم و اشتباهات رایج

### ⚠️ اشتباه رایج 1: فراموش کردن حالت پیش‌فرض (`_`)
برخلاف `switch` روی `enum`ها که کامپایلر می‌تواند Exhaustive بودن (پوشش تمام حالات) را بررسی کند، در **Tuple Pattern** کامپایلر معمولاً نمی‌تواند تمام ترکیبات بی‌نهایت یا پیچیده را پوشش دهد.
**راه‌حل:** همیشه در انتهای `switch` خود یک `_ => ...` قرار دهید، در غیر این صورت با خطای `CS8509: The switch expression does not handle all possible inputs` مواجه می‌شوید.

### ⚠️ اشتباه رایج 2: نادیده گرفتن ترتیب کیس‌ها
در Switch Expression، **اولین کیسی که تطابق پیدا کند اجرا می‌شود**.
```csharp
// اشتباه!
(> 0, _) => "مثبت",
(> 10, _) => "بزرگتر از ده" // این کیس هرگز اجرا نمی‌شود!
```
**راه‌حل:** همیشه کیس‌های خاص‌تر (Specific) را در بالا و کیس‌های کلی‌تر (General) را در پایین قرار دهید.

### 💡 نکته مهم: نادیده گرفته شدن نام Tupleها
در تطبیق الگو (Pattern Matching)، **نام عناصر Tuple مهم نیست**، فقط ترتیب و نوع آن‌ها مهم است:
```csharp
var point = (X: 10, Y: 20);
var result = point switch
{
    (10, 20) => "تطابق پیدا کرد!", // نیازی به نوشتن (X: 10, Y: 20) نیست
    _ => "خطا"
};
```

---

<a id="جمع‌بندی"></a>
## جمع‌بندی

استفاده از **Tuple در Switch Expression** یکی از بهترین ویژگی‌های اضافه شده به C# در سال‌های اخیر برای افزایش خوانایی و نگهداری کد است. 
- هرگاه نیاز داشتید بر اساس **ترکیب دو یا چند متغیر** تصمیم‌گیری کنید، به جای `if/else` های تو در تو، از `(var1, var2) switch { ... }` استفاده کنید.
- این تکنیک کد شما را Declarative (توصیفی) می‌کند؛ یعنی به جای اینکه به کامپیوتر بگویید «چگونه» بررسی کند، به او می‌گویید «چه» چیزی مد نظر شماست.
- با ترکیب این الگو با Relational Patterns و `when`، می‌توانید پیچیده‌ترین منطق‌های تجاری را در چند خط کدِ بسیار تمیز پیاده‌سازی کنید.

---

<a id="منابع-رسمی"></a>
## منابع رسمی

برای مطالعه بیشتر و عمیق‌تر، پیشنهاد می‌کنیم مستندات رسمی مایکروسافت را مطالعه کنید:
1. [Switch Expression - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression)
2. [Tuple Patterns - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching#tuple-pattern)
3. [Pattern Matching Overview](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)

---
*نویسنده: تیم آموزشی Repository | تاریخ بازبینی: آگوست 2026*
*اگر این مقاله برای شما مفید بود، فراموش نکنید که به این Repository ستاره (⭐) بدهید!*