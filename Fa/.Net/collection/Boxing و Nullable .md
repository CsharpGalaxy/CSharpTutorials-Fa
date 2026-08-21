

# 🥊 رقص با حافظه: راهنمای جامع Boxing و Nullable در #C

به این آموزش خوش آمدید! اگر تا به حال با خطاهای عجیب هنگام کار با `null` مواجه شده‌اید یا کنجکاو هستید که وقتی یک متغیر را به `object` تبدیل می‌کنید، دقیقاً در پشت صحنه و در حافظه چه اتفاقی می‌افتد، این مقاله برای شماست.

ما در این آموزش از مفاهیم پایه شروع کرده و به رفتارهای عمیق CLR (محیط اجرای زبان مشترک) می‌رسیم.

---

## 📑 فهرست مطالب
1. [پیش‌نیاز: جنگ Stack و Heap](#1-پیشنیاز-جنگ-stack-و-heap)
2. [مفهوم Boxing و Unboxing چیست؟](#2-مفهوم-boxing-و-unboxing-چیست)
3. [آشنایی با Nullable Types (انواع تهی‌پذیر)](#3-آشنایی-با-nullable-types)
4. [تعامل Boxing و Nullable (قلب مقاله)](#4-تعامل-boxing-و-nullable)
   - [حالت اول: Boxing یک Nullable که مقدارش Null است](#حالت-اول-boxing-یک-nullable-که-مقدارش-null-است)
   - [حالت دوم: Boxing یک Nullable که مقدار دارد](#حالت-دوم-boxing-یک-nullable-که-مقدار-دارد)
5. [Unboxing کردن Nullable](#5-unboxing-کردن-nullable)
6. [نگاهی به زیر کاپوت: رفتار CLR هنگام Boxing](#6-نگاهی-به-زیر-کاپوت-رفتار-clr-هنگام-boxing)
7. [بهترین شیوه‌ها (Best Practices)](#7-بهترین-شیوه‌ها)
8. [منابع معتبر](#8-منابع-معتبر)

---

## 1. پیش‌نیاز: جنگ Stack و Heap
برای درک Boxing، ابتدا باید تفاوت دو نوع داده در #C را بدانیم:
* **Value Types (انواع مقداری):** مثل `int`, `double`, `bool` و `struct`. معمولاً در حافظه **Stack** (پشته) ذخیره می‌شوند. سریع هستند و اندازه ثابتی دارند.
* **Reference Types (انواع مرجعی):** مثل `class`, `string`, `array`. در حافظه **Heap** (توده) ذخیره می‌شوند و متغیر ما فقط یک "آدرس" (Reference) به آن‌ها در Stack دارد.

---

## 2. مفهوم Boxing و Unboxing چیست؟
گاهی اوقات نیاز داریم یک **Value Type** را مثل یک **Reference Type** رفتار دهیم (مثلاً بخواهیم آن را در یک متغیر از نوع `object` یا یک `ArrayList` قدیمی ذخیره کنیم).

* **Boxing (جعبه‌گذاری):** تبدیل یک Value Type به Reference Type. CLR یک "جعبه" (آبجکت) در Heap می‌سازد، مقدار را داخل آن کپی می‌کند و آدرس جعبه را برمی‌گرداند.
* **Unboxing (جعبه‌گشایی):** تبدیل یک Reference Type (که قبلاً Box شده) به Value Type.

```csharp
int myValue = 10;          // Value Type (روی Stack)
object myBox = myValue;    // BOXING: یک آبجکت در Heap ساخته می‌شود و 10 در آن کپی می‌شود.
int unboxed = (int)myBox;  // UNBOXING: مقدار از Heap خوانده شده و به Stack برمی‌گردد.
```

---

## 3. آشنایی با Nullable Types
در دنیای دیتابیس، یک فیلد عددی می‌تواند `NULL` باشد. اما در #C، متغیر `int` نمی‌تواند `null` باشد (چون Value Type است و همیشه یک مقدار پیش‌فرض مثل صفر دارد).
برای حل این مشکل، `Nullable<T>` معرفی شد.

**نکته بسیار مهم و طلایی:** `Nullable<T>` خودش یک **Value Type (Struct)** است!
```csharp
int? nullableInt = null; // معادل Nullable<int>
```
این ساختار در زیر خود دو فیلد دارد:
1. `hasValue` (یک بولین که می‌گوید آیا مقدار دارد یا خیر)
2. `value` (خود مقدار)

---

## 4. تعامل Boxing و Nullable
حالا به سوال اصلی و چالش‌برانگیز می‌رسیم: **اگر یک `Nullable` را Box کنیم چه اتفاقی می‌افتد؟**
آیا CLR کل ساختار `Nullable<T>` (شامل `hasValue` و `value`) را در Heap کپی می‌کند؟ **خیر!**
رفتار CLR در اینجا بسیار هوشمندانه و بهینه است.

### حالت اول: Boxing یک Nullable که مقدارش Null است
اگر `HasValue` برابر با `false` باشد، CLR ساختار `Nullable` را Box نمی‌کند. بلکه **مستقیماً یک `null` reference** برمی‌گرداند.

```csharp
int? nullValue = null;
object boxedNull = nullValue; 

// بررسی رفتار:
Console.WriteLine(boxedNull == null); // خروجی: True
Console.WriteLine(boxedNull is int);  // خروجی: False (چون خودش null است)
```
**نتیجه:** هیچ حافظه‌ای در Heap اشغال نمی‌شود و فقط یک اشاره‌گر `null` تحویل می‌گیریم.

### حالت دوم: Boxing یک Nullable که مقدار دارد
اگر `HasValue` برابر با `true` باشد، CLR باز هم ساختار `Nullable<T>` را Box نمی‌کند! بلکه **فقط مقدار داخلی (Underlying Value)** را استخراج کرده و همان را Box می‌کند.

```csharp
int? hasValue = 42;
object boxedValue = hasValue;

// بررسی رفتار:
Console.WriteLine(boxedValue.GetType().Name); // خروجی: Int32 (نه Nullable)
Console.WriteLine(boxedValue is int);         // خروجی: True
```
**نتیجه:** در Heap، دقیقاً همان اتفاقی می‌افتد که انگار شما یک `int` معمولی را Box کرده‌اید. هیچ سرباری بابت ساختار `Nullable` تحمیل نمی‌شود.

---

## 5. Unboxing کردن Nullable
وقتی می‌خواهید یک آبجکت را به `Nullable<T>` تبدیل کنید (Unboxing)، CLR دوباره رفتار هوشمندانه‌ای دارد:

1. **اگر آبجکت `null` باشد:** یک `Nullable<T>` ساخته می‌شود که `HasValue = false` است.
2. **اگر آبجکت حاوی مقدار `T` باشد:** یک `Nullable<T>` ساخته می‌شود که `HasValue = true` و مقدار داخل آن ریخته می‌شود.
3. **اگر نوع آبجکت با `T` همخوانی نداشته باشد:** خطای `InvalidCastException` پرتاب می‌شود.

```csharp
object objNull = null;
object objInt = 100;

int? unboxedNull = (int?)objNull;     // موفقیت‌آمیز: HasValue = false
int? unboxedInt = (int?)objInt;       // موفقیت‌آمیز: HasValue = true, Value = 100

// خطر: 
string objStr = "Hello";
int? badCast = (int?)objStr;          // پرتاب خطای InvalidCastException
```

---

## 6. نگاهی به زیر کاپوت: رفتار CLR هنگام Boxing
برای برنامه‌نویسان پیشرفته، درک این بخش حیاتی است تا از مشکلات Performance (عملکرد) جلوگیری کنند.

### سربار حافظه و GC (زباله‌روب)
هر بار که Boxing رخ می‌دهد:
1. حافظه‌ای در **Heap** تخصیص می‌یابد.
2. داده کپی می‌شود.
3. پس از اینکه متغیر Reference از Scope خارج شد، این آبجکت باید توسط **Garbage Collector** پاکسازی شود.
*این فرآیند در حلقه‌های `for` یا توابعی که میلیون‌ها بار صدا زده می‌شوند، می‌تواند باعث افت شدید عملکرد و فشار به GC شود.*

### بهینه‌سازی JIT Compiler برای Nullable
در مشخصات زبان #C و CLR، یک قانون خاص (Special Rule) برای `Nullable<T>` وجود دارد.
اگر CLR می‌خواست طبق قانون عادی عمل کند، باید `Nullable<T>` (که خودش یک Struct است) را Box می‌کرد. اما JIT Compiler در زمان کامپایل کد را بررسی می‌کند و می‌بیند که متغیر از نوع `Nullable<T>` است.
* **اگر null باشد:** به جای ساخت آبجکت، مستقیماً `null` برمی‌گرداند.
* **اگر مقدار داشته باشد:** به جای Box کردنِ Struct، مستقیماً فیلد `value` را خوانده و آن را Box می‌کند.
این یعنی **جلوگیری از Double Boxing** و بهینه‌سازی فوق‌العاده در سطح Runtime.

---

## 7. بهترین شیوه‌ها (Best Practices)
برای اینکه کدی تمیز، سریع و بدون باگ بنویسید، این نکات را رعایت کنید:

1. **تا حد امکان از Boxing جلوگیری کنید:** به جای استفاده از کالکشن‌های قدیمی مثل `ArrayList` (که آبجکت می‌گیرند)، از `List<T>` یا `IEnumerable<T>` استفاده کنید تا Genericها از Boxing جلوگیری کنند.
2. **استفاده از `as` برای Unboxing ایمن:** اگر نمی‌دانید آبجکت شما دقیقاً چه نوعی است، از Cast مستقیم `(int?)obj` پرهیز کنید. به جای آن از `as` یا Pattern Matching استفاده کنید تا از پرتاب Exception جلوگیری شود.
   ```csharp
   // روش ایمن (بدون پرتاب خطا)
   int? safeValue = obj as int?; 
   if (safeValue.HasValue) { ... }
   ```
3. **توجه به تفاوت `Nullable<T>` و Nullable Reference Types:**
   * `int?` (که در این مقاله بررسی کردیم) یک **Value Type** است و رفتارهای Boxing خاصی دارد.
   * `string?` (معرفی شده در #C 8.0) فقط یک **Annotation** برای کامپایلر است و در زمان اجرا (Runtime) هیچ تفاوتی با `string` معمولی ندارد و بحث Boxing برای آن منتفی است (چون Reference Type است).

---

## 8. منابع معتبر
برای مطالعه عمیق‌تر و ارجاع به مستندات اصلی، منابع زیر پیشنهاد می‌شوند:

1. **کتاب CLR via C# (نوشته Jeffrey Richter)**
   * *فصل 5: Primary Object Types* و *فصل 19: Nullable Types*. این کتاب "انجیل" رفتارهای حافظه در #C است.
2. **مستندات رسمی مایکروسافت (Microsoft Learn)**
   * [Nullable value types (C# reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)
   * [Boxing and Unboxing (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing)
3. **مستندات ECMA-334 (C# Language Specification)**
   * بخش *Boxing and Unboxing Conversions* که قوانین دقیق تبدیل `Nullable<T>` را دیکته می‌کند.
4. **مقالات Jon Skeet (خالق کتاب C# in Depth)**
   * [Nullable Types and Boxing](https://csharpindepth.com/) - تحلیل‌های ایشان از رفتارهای عجیب و بهینه CLR همواره مرجع برنامه‌نویسان ارشد است.

---
*اگر این آموزش برای شما مفید بود، لطفاً به ریپازیتوری ستاره (⭐) بدهید و برای توسعه‌دهندگان دیگر ارسال کنید!*
**نویسنده:** [نام شما / نام تیم شما] | **تاریخ:** August 2026