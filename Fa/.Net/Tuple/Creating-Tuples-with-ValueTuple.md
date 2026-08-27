# آموزش جامع ایجاد Tuple با ValueTuple در C# (از مقدماتی تا پیشرفته)

به مقاله آموزشی **ایجاد Tuple با ValueTuple** خوش آمدید! در این مقاله از مجموعه آموزش‌های C#، به بررسی یکی از کاربردی‌ترین و جذاب‌ترین ویژگی‌های اضافه‌شده در C# 7.0، یعنی `ValueTuple` می‌پردازیم. اگر تا به حال نیاز داشته‌اید که چندین مقدار را از یک متد برگردانید یا داده‌های موقتی را در یک ساختار سبک گروه‌بندی کنید، این مقاله دقیقاً برای شما نوشته شده است.

---

## 📑 فهرست مطالب
1. [مقدمه: Tuple چیست و چرا ValueTuple؟](#مقدمه)
2. [سینتکس و ایجاد Tuple با پرانتز](#۱-سینتکس-و-ایجاد-tuple-با-پرانتز)
3. [Tuple بدون نام (Unnamed) در مقابل نام‌گذاری‌شده (Named)](#۲-tuple-بدون-نام-و-نامگذاریشده)
4. [تعیین Type اعضا و استفاده از var](#۳-تعیین-type-اعضا-و-استفاده-از-var)
5. [مقداردهی اولیه و دسترسی به اعضا](#۴-مقداردهی-اولیه-و-دسترسی-به-اعضا)
6. [تغییر مقدار اعضا (Mutability)](#۵-تغییر-مقدار-اعضا-در-valuetuple)
7. [تفاوت Syntax پرانتز `()` با `new ValueTuple`](#۶-تفاوت-syntax-پرانتز-با-new-valuetuple)
8. [نحوه استنتاج Type توسط کامپایلر](#۷-نحوه-استنتاج-type-توسط-کامپایلر)
9. [مثال‌های عملی (ساده تا پیشرفته)](#۸-مثالهای-عملی)
10. [نکات مهم و اشتباهات رایج](#۹-نکات-مهم-و-اشتباهات-رایج)
11. [جمع‌بندی](#جمع‌بندی)
12. [منابع معتبر](#منابع-معتبر)

---

<a name="مقدمه"></a>
## مقدمه: Tuple چیست و چرا ValueTuple؟
در زبان C#، **Tuple** یک ساختار داده‌ای سبک است که به شما اجازه می‌دهد چندین فیلد با نوع‌های مختلف را در یک آبجکت واحد گروه‌بندی کنید. 
پیش از C# 7.0، ما از کلاس `System.Tuple` استفاده می‌کردیم که یک **Reference Type** (کلاس) بود و روی Heap حافظه تخصیص می‌یافت. اما در C# 7.0 مایکروسافت `System.ValueTuple` را معرفی کرد که یک **Struct** (Value Type) است، روی Stack (در اکثر مواقع) قرار می‌گیرد و سربار حافظه و عملکرد (Performance) بسیار کمتری دارد.

> 💡 **نکته:** از این به بعد در این مقاله، هر جا کلمه Tuple را می‌بینید، منظورمان همان `ValueTuple` مدرن است.

---

<a name="۱-سینتکس-و-ایجاد-tuple-با-پرانتز"></a>
## ۱. سینتکس و ایجاد Tuple با پرانتز
ساده‌ترین راه برای ایجاد یک Tuple، استفاده از پرانتز `()` است. این سینتکس که در C# 7.0 معرفی شد، خوانایی کد را به شدت افزایش می‌دهد.

```csharp
// ایجاد یک Tuple با دو عضو
var myTuple = (10, "Ali");
```

---

<a name="۲-tuple-بدون-نام-و-نامگذاریشده"></a>
## ۲. Tuple بدون نام و نام‌گذاری‌شده

### الف) Tuple بدون نام (Unnamed)
اگر برای اعضا نامی تعیین نکنید، کامپایلر به‌صورت پیش‌فرض نام‌های `Item1`، `Item2` و... را به آن‌ها اختصاص می‌دهد.

```csharp
var unnamedTuple = (42, true, "Hello");
Console.WriteLine(unnamedTuple.Item1); // خروجی: 42
Console.WriteLine(unnamedTuple.Item2); // خروجی: True
```

### ب) Tuple نام‌گذاری‌شده (Named)
برای افزایش خوانایی، می‌توانید برای هر عضو یک نام اختصاصی در نظر بگیرید.

```csharp
var namedTuple = (Id: 100, IsActive: true, Message: "Hello");
Console.WriteLine(namedTuple.Id);      // خروجی: 100
Console.WriteLine(namedTuple.Message); // خروجی: Hello
```

---

<a name="۳-تعیین-type-اعضا-و-استفاده-از-var"></a>
## ۳. تعیین Type اعضا و استفاده از var

شما می‌توانید نوع داده‌ها را به‌صورت صریح (Explicit) مشخص کنید یا از کامپایلر بخواهید خودش آن‌ها را حدس بزند (Implicit).

### تعیین صریح Type:
```csharp
(int X, double Y, string Name) point = (10, 20.5, "Origin");
```

### استفاده از `var` (استنتاج نوع):
وقتی از `var` استفاده می‌کنید، کامپایلر بر اساس مقادیر سمت راست، نوع Tuple را تشخیص می‌دهد.
```csharp
var point = (10, 20.5, "Origin"); 
// کامپایلر این را به عنوان (int, double, string) در نظر می‌گیرد
```

---

<a name="۴-مقداردهی-اولیه-و-دسترسی-به-اعضا"></a>
## ۴. مقداردهی اولیه و دسترسی به اعضا

شما می‌توانید متغیرهای از پیش تعریف‌شده را درون یک Tuple قرار دهید. در این حالت، نام متغیرها به‌عنوان نام اعضای Tuple در نظر گرفته می‌شود (اگر نام صریحی تعیین نکرده باشید).

```csharp
int age = 25;
string name = "Sara";

// نام متغیرها (age و name) به عنوان نام اعضای Tuple ذخیره می‌شوند
var user = (age, name); 

Console.WriteLine(user.age);  // دسترسی از طریق نام متغیر
Console.WriteLine(user.Item1); // دسترسی از طریق Item1 (همان age است)
```

---

<a name="۵-تغییر-مقدار-اعضا-در-valuetuple"></a>
## ۵. تغییر مقدار اعضا در ValueTuple
برخلاف بسیاری از Structهای دیگر در C# که Immutable (غیرقابل تغییر) هستند، `ValueTuple`ها **Mutable** (قابل تغییر) طراحی شده‌اند. شما می‌توانید مقدار اعضا را پس از ایجاد تغییر دهید.

```csharp
var coordinates = (X: 10, Y: 20);
Console.WriteLine($"Before: X={coordinates.X}, Y={coordinates.Y}");

// تغییر مقدار اعضا
coordinates.X = 50;
coordinates.Y = 100;

Console.WriteLine($"After: X={coordinates.X}, Y={coordinates.Y}");
```

---

<a name="۶-تفاوت-syntax-پرانتز-با-new-valuetuple"></a>
## ۶. تفاوت Syntax پرانتز `()` با `new ValueTuple`

سینتکس پرانتز `()` در واقع **Syntactic Sugar** (شکر سینتکسی) برای استفاده از `new ValueTuple<T1, T2, ...>` است. کامپایلر کد شما را به شکل زیر ترجمه می‌کند:

```csharp
// روش مدرن و پیشنهادی (Syntactic Sugar)
var t1 = (1, "Test");

// روش قدیمی و معادل آن در سطح IL (که کامپایلر تولید می‌کند)
var t2 = new ValueTuple<int, string>(1, "Test");
```
**تفاوت:** استفاده از پرانتز بسیار تمیزتر است، اما اگر نیاز داشته باشید Tuple را به یک متد پاس دهید که دقیقاً `ValueTuple` می‌پذیرد یا در شرایط خاصی از Reflection استفاده کنید، ممکن است با ساختار `new ValueTuple` روبرو شوید.

---

<a name="۷-نحوه-استنتاج-type-توسط-کامپایلر"></a>
## ۷. نحوه استنتاج Type توسط کامپایلر

کامپایلر C# بسیار هوشمند عمل می‌کند. اگر یک متد Tuple برگرداند، کامپایلر نوع آن را از روی Return Type متد استنتاج می‌دهد:

```csharp
public static (int Code, string Status) GetOrderStatus()
{
    return (200, "Success");
}

// استنتاج Type در متغیر گیرنده
var result = GetOrderStatus(); 
// کامپایلر می‌فهمد که result از نوع (int Code, string Status) است
Console.WriteLine(result.Status); // خروجی: Success
```

---

<a name="۸-مثالهای-عملی"></a>
## ۸. مثال‌های عملی

### مثال ساده: محاسبه مساحت و محیط مستطیل
```csharp
public static (int Area, int Perimeter) CalculateRectangle(int width, int height)
{
    int area = width * height;
    int perimeter = 2 * (width + height);
    return (area, perimeter); // بازگرداندن چندین مقدار بدون نیاز به کلاس اضافی
}

// استفاده:
var (area, perimeter) = CalculateRectangle(5, 10); // استفاده از Deconstruction
Console.WriteLine($"Area: {area}, Perimeter: {perimeter}");
```

### مثال پیشرفته: شبیه‌سازی دیکشنری با Tupleها
```csharp
var users = new[]
{
    (Id: 1, Name: "Ali", Role: "Admin"),
    (Id: 2, Name: "Sara", Role: "User"),
    (Id: 3, Name: "Reza", Role: "User")
};

// فیلتر کردن و پیدا کردن کاربر
var admin = users.FirstOrDefault(u => u.Role == "Admin");
Console.WriteLine($"Admin Name: {admin.Name}");
```

---

<a name="۹-نکات-مهم-و-اشتباهات-رایج"></a>
## ۹. نکات مهم و اشتباهات رایج

### ⚠️ اشتباه رایج ۱: استفاده از Tuple در Public API
هرگز از Tupleها برای بازگرداندن داده در متدهای **Public** یک کتابخانه یا لایه سرویس‌دهی استفاده نکنید. 
**دلیل:** نام اعضای Tuple (مثل `Name` یا `Id`) فقط در **زمان کامپایل (Compile-Time)** وجود دارند. در زمان اجرا (Runtime)، این نام‌ها از بین می‌روند و فقط `Item1` و `Item2` باقی می‌مانند. بنابراین اگر کد شما از طریق Reflection یا در پروژه‌های دیگری که به اسم‌ها وابسته‌اند استفاده شود، می‌شکند.
✅ **راه‌حل:** برای APIهای عمومی از `class`، `struct` یا `record` استفاده کنید.

### ⚠️ اشتباه رایج ۲: توهم وجود نام‌ها در Runtime
```csharp
var myTuple = (FirstName: "Ali", LastName: "Alavi");
// در زمان اجرا، myTuple فقط دارای فیلدهای Item1 و Item2 است.
// نام‌های FirstName و LastName فقط metadata برای IDE و کامپایلر هستند.
```

### 💡 نکته مهم: محدودیت ۸ تایی
ساختار `ValueTuple` به‌صورت پیش‌فرض تا ۷ عضو را پشتیبانی می‌کند (`ValueTuple<T1..T7>`). اگر به عضو هشتم نیاز داشته باشید، کامپایلر به‌صورت خودکار از `ValueTuple<T1..T7, TRest>` استفاده می‌کند که `TRest` خودش یک Tuple دیگر است. دسترسی به اعضای بعد از هفتم کمی پیچیده می‌شود، بنابراین اگر بیش از ۷ عضو نیاز دارید، **حتماً** از یک `class` یا `record` استفاده کنید.

---

<a name="جمع‌بندی"></a>
## جمع‌بندی
`ValueTuple` یک ویژگی فوق‌العاده در C# است که به شما اجازه می‌دهد بدون تعریف کلاس‌های اضافی (DTOهای موقت)، چندین مقدار را با هم گروه‌بندی کنید. 
* از پرانتز `()` برای ساخت سریع استفاده کنید.
* برای خوانایی بهتر، حتماً اعضا را نام‌گذاری کنید.
* به یاد داشته باشید که Tupleها برای **استفاده‌های داخلی و خصوصی (Private/Internal)** عالی هستند، اما برای **مدل‌های دامنه و APIهای عمومی** باید از `Record`ها یا `Class`ها استفاده کنید.

---

<a name="منابع-معتبر"></a>
## منابع معتبر
برای مطالعه عمیق‌تر و بررسی دقیق‌تر مشخصات زبان، منابع زیر پیشنهاد می‌شوند:

1. **[Microsoft Learn - Tuple types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)**
   * مرجع اصلی مایکروسافت برای بررسی سینتکس، استنتاج نوع و محدودیت‌های ValueTuple.
2. **[Microsoft Learn - Deconstruction](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct)**
   * مقاله‌ای درباره نحوه تجزیه Tupleها به متغیرهای مجزا (مبحث مرتبط).
3. **[C# Language Specification - Tuple Expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1281-tuple-expressions)**
   * مشخصات رسمی زبان C# (ECMA) برای درک نحوه رفتار کامپایلر در سطح پایین.
4. **[Source Code: System.ValueTuple](https://source.dot.net/System.Private.CoreLib/ValueTuple.cs.html)**
   * بررسی سورس کد اصلی ValueTuple در مخزن .NET Runtime برای درک ساختار داخلی آن.

---
🔗 **لینک‌های داخلی مرتبط در این Repository:**
* [لینک به مقاله: تفاوت Struct و Class در C#](#)
* [لینک به مقاله: آشنایی با Record Types در C# 9.0](#)
* [لینک به مقاله: Deconstruction و الگوهای تطبیق (Pattern Matching)](#)

---
*نویسنده: تیم آموزشی Repository | تاریخ بازبینی: August 2026*
*اگر این مقاله برای شما مفید بود، فراموش نکنید که به Repository ما ستاره (⭐) بدهید!*