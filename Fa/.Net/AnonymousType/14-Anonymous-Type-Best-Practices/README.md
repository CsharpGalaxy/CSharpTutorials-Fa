# راهنمای جامع و Best Practices برای Anonymous Types در C#

به نام خدا. این مقاله به‌عنوان یک مرجع کامل و کاربردی برای درک عمیق **Anonymous Types** در زبان C# نوشته شده است. هدف این مستند، عبور از دانش سطحی و بررسی دقیق معماری، محدودیت‌ها، و بهترین زمان‌های استفاده از این ویژگی قدرتمند کامپایلر است.

---

## 📑 فهرست مطالب
1. [مقدمه: Anonymous Type چیست؟](#مقدمه)
2. [مفاهیم پایه و معماری داخلی](#مفاهیم-پایه)
   - [Type Identity (هویت نوع)](#type-identity)
   - [Immutability (تغییرناپذیری)](#immutability)
   - [Equality (برابری مقادیر)](#equality)
   - [نکته مهم درباره `with` Expression](#with-expression)
3. [محدودیت‌ها و Scope](#محدودیتها)
4. [کاربردها: LINQ و Entity Framework Core](#linq-efcore)
5. [مقایسه: Anonymous Type vs Tuple vs Record vs Class](#مقایسه)
6. [معماری نرم‌افزار: API Design و Code Smells](#معماری)
7. [بررسی Performance](#performance)
8. [اشتباهات رایج (مبتدیان و حرفه‌ای‌ها)](#اشتباهات)
9. [Checklist نهایی و Decision Guide](#decision-guide)
10. [جمع‌بندی](#جمع-بندی)
11. [منابع و مطالعه بیشتر](#منابع)

---

<a name="مقدمه"></a>
## ۱. مقدمه: Anonymous Type چیست؟
تایپ‌های گمنام (Anonymous Types) در C# راهی برای بسته‌بندی مجموعه‌ای از_PROPERTY_ها در یک شیء بدون نیاز به تعریف یک `class` یا `struct` مجزا هستند. این تایپ‌ها در زمان کامپایل توسط Compiler تولید می‌شوند.

```csharp
// تعریف یک Anonymous Type
var user = new { Name = "Ali", Age = 30 };
Console.WriteLine(user.Name); // خروجی: Ali
```

<a name="مفاهیم-پایه"></a>
## ۲. مفاهیم پایه و معماری داخلی

<a name="type-identity"></a>
### 🔹 Type Identity (هویت نوع)
کامپایلر C# دو Anonymous Type را **تنها در صورتی** یکسان (Same Type) در نظر می‌گیرد که در **یک Assembly** تعریف شده باشند و شرایط زیر را داشته باشند:
1. نام Propertyها دقیقاً یکسان باشد.
2. نوع (Type) Propertyها دقیقاً یکسان باشد.
3. **ترتیب** قرارگیری Propertyها یکسان باشد.

```csharp
var a = new { X = 1, Y = "A" };
var b = new { X = 2, Y = "B" };
var c = new { Y = "C", X = 3 }; // ترتیب متفاوت است!

// a و b از یک نوع هستند، اما c نوعی کاملاً متفاوت است.
Console.WriteLine(a.GetType() == b.GetType()); // True
Console.WriteLine(a.GetType() == c.GetType()); // False
```

<a name="immutability"></a>
### 🔹 Immutability (تغییرناپذیری)
تمام Propertyهای تولید شده در Anonymous Types به‌صورت **Read-Only** هستند. شما نمی‌توانید پس از مقداردهی اولیه، آن‌ها را تغییر دهید.

```csharp
var product = new { Id = 1, Title = "Laptop" };
// product.Title = "PC"; // ❌ خطای کامپایل: Propertyها فقط خواندنی هستند
```

<a name="equality"></a>
### 🔹 Equality (برابری مقادیر)
کامپایلر به‌صورت خودکار متدهای `Equals()` و `GetHashCode()` را بر اساس **مقادیر Propertyها** (Value-based Equality) پیاده‌سازی می‌کند، نه بر اساس Reference.

```csharp
var p1 = new { Id = 1, Name = "A" };
var p2 = new { Id = 1, Name = "A" };

Console.WriteLine(p1.Equals(p2)); // True (برابری بر اساس مقدار)
```

<a name="with-expression"></a>
### 🔹 ⚠️ نکته بسیار مهم درباره `with` Expression
بسیاری از برنامه‌نویسان تصور می‌کنند می‌توانند از `with` expression برای کپی کردن Anonymous Typeها استفاده کنند. **این تصور کاملاً اشتباه است.**
عبارت `with` **فقط و فقط** مخصوص `record`ها (از C# 9 به بعد) است و کامپایلر برای Anonymous Typeها آن را پشتیبانی نمی‌کند.

```csharp
var original = new { Id = 1, Name = "Ali" };
// var copy = original with { Name = "Reza" }; // ❌ خطای کامپایل
```

<a name="محدودیتها"></a>
## ۳. محدودیت‌ها و Scope

به دلیل اینکه شما نام کلاس تولید شده را نمی‌دانید، Anonymous Typeها محدودیت‌های شدیدی در انتقال داده دارند:

1. **محدودیت Return Type:** نمی‌توانید یک Anonymous Type را به‌عنوان خروجی یک متد برگردانید (مگر اینکه از `object` یا `dynamic` استفاده کنید که **به‌شدت نهی شده است**).
2. **محدودیت Method Parameter:** نمی‌توانید آن را به‌عنوان پارامتر به متدهای دیگر پاس دهید.
3. **Scope:** استفاده از آن‌ها باید **محلی (Local)** باشد؛ یعنی در همان متدی که تعریف می‌شوند، مصرف شوند.
4. **Array:** می‌توانید آرایه‌ای از آن‌ها بسازید، به شرطی که ساختار همه عناصر دقیقاً یکسان باشد (به لطف Implicitly Typed Arrays).

```csharp
// ✅ ساخت آرایه از Anonymous Type
var users = new[] 
{ 
    new { Name = "Ali", Age = 30 }, 
    new { Name = "Sara", Age = 25 } 
};
```

<a name="linq-efcore"></a>
## ۴. کاربردها: LINQ و Entity Framework Core

### 🏆 بهترین کاربرد: LINQ Projections
اصلی‌ترین دلیل طراحی Anonymous Typeها، استفاده در متد `Select` در LINQ برای شکل‌دهی به داده‌ها (Data Shaping) است.

```csharp
var query = dbContext.Users
    .Where(u => u.IsActive)
    .Select(u => new 
    { 
        FullName = u.FirstName + " " + u.LastName, 
        u.Email 
    });
```

### 🔗 ارتباط با Entity Framework Core
وقتی در EF Core از Anonymous Type در `Select` استفاده می‌کنید، EF Core آن را به درستی به SQL ترجمه می‌کند و **فقط ستون‌های درخواستی** از دیتابیس واکشی می‌شوند (جلوگیری از Over-fetching).
*نکته:* EF Core نمی‌تواند Anonymous Typeها را در `Insert` یا `Update` استفاده کند، زیرا آن‌ها تغییرناپذیر (Immutable) هستند و EF برای ردیابی تغییرات (Change Tracking) نیاز به اشیاء Mutable دارد.

<a name="مقایسه"></a>
## ۵. مقایسه: Anonymous Type vs Tuple vs Record vs Class

چه زمانی از کدام استفاده کنیم؟

| ویژگی | Anonymous Type | Tuple (`(T1, T2)`) | Record (`record class/struct`) | Class |
| :--- | :--- | :--- | :--- | :--- |
| **نام Propertyها** | دارد (خوانا) | ندارد (مگر با نام مستعار) | دارد (خوانا) | دارد |
| **تغییرناپذیری** | ✅ کاملاً Immutable | ❌ Mutable (پیش‌فرض) | ✅ Immutable (در `record class`) | ❌ Mutable (پیش‌فرض) |
| **قابلیت برگشت از متد** | ❌ خیر | ✅ بله | ✅ بله | ✅ بله |
| **قابلیت `with`** | ❌ خیر | ❌ خیر | ✅ بله | ❌ خیر |
| **برابری (Equality)** | ✅ Value-based | ✅ Value-based | ✅ Value-based | ❌ Reference-based |
| **بهترین کاربرد** | فقط داخل LINQ و Scope محلی | برگشت چند مقدار از متد | DTOها، Stateهای دامنه | Entityها، Serviceها |

<a name="معماری"></a>
## ۶. معماری نرم‌افزار: API Design و Code Smells

### 🚫 API Design
**هرگز** Anonymous Typeها را به‌عنوان پاسخ (Response) در Web APIها برنگردانید.
اگرچه ASP.NET Core می‌تواند آن‌ها را به JSON سریالایز کند، اما این کار باعث می‌شود Contract (قرارداد) API شما پنهان بماند، مستندات خودکار (مثل Swagger/OpenAPI) به درستی تولید نشوند و تغییر ساختار آن در آینده باعث Breaking Change برای کلاینت‌ها شود.

### 🤢 Code Smellها (نشانه‌های استفاده اشتباه)
اگر کدهای زیر را دیدید، یعنی از Anonymous Type اشتباه استفاده شده است:
1. **استفاده از `dynamic` یا `object` برای دور زدن محدودیت Return Type:**
   ```csharp
   // ❌ Code Smell
   public object GetUser() => new { Name = "Ali" }; 
   ```
2. **استفاده در Domain Models:** دامنه نرم‌افزار نیاز به رفتار (Behavior) و اعتبارسنجی دارد؛ Anonymous Typeها فقط داده (Data) هستند.
3. **ذخیره در State طولانی‌مدت:** مثلاً ذخیره در `Session` یا `Cache` به‌صورت `object` و سپس Cast کردن با `dynamic`.

<a name="performance"></a>
## ۷. بررسی Performance
*   **Heap Allocation:** Anonymous Typeها در واقع `class` (Reference Type) هستند. بنابراین، ساخت آن‌ها باعث Allocation روی Heap و فشار به Garbage Collector می‌شود.
*   **Type Caching:** کامپایلر C# بهینه عمل می‌کند. اگر در یک حلقه `for` میلیون‌ها بار `new { X = i }` را با ساختار یکسان صدا بزنید، کامپایلر فقط **یک بار** کلاس آن را تولید می‌کند و در runtime فقط Instanceهای جدید از همان یک کلاس ساخته می‌شوند.
*   **نتیجه‌گیری Performance:** برای عملیات‌های محلی و LINQ کاملاً بهینه است، اما برای حلقه‌های بسیار سنگین (High-frequency allocations) که نیاز به تغییرپذیری ندارند، استفاده از `struct` یا `record struct` (اگر نیاز به نام Property دارید) یا `ValueTuple` (اگر نام Property مهم نیست) بهتر است.

<a name="اشتباهات"></a>
## ۸. اشتباهات رایج

### ❌ اشتباهات مبتدیان
1. **تلاش برای تغییر مقدار:**
   ```csharp
   var p = new { Id = 1 };
   p.Id = 2; // خطای کامپایل
   ```
2. **تلاش برای بازگرداندن از متد و استفاده از Reflection یا dynamic:** که باعث از بین رفتن Type Safety و افت شدید Performance می‌شود.

### ❌ اشتباهات برنامه‌نویسان حرفه‌ای
1. **استفاده به جای DTO در لایه‌های مختلف:** انتقال Anonymous Type بین لایه Service و Controller با استفاده از `dynamic`. این کار خوانایی و Maintainability را نابود می‌کند.
2. **نادیده گرفتن ترتیب Propertyها در LINQ:** گاهی اوقات تغییر ترتیب Propertyها در یک `Select` باعث می‌شود کامپایلر آن را یک نوع جدید در نظر بگیرد که در برخی سناریوهای خاص (مثل GroupBy یا Joinهای پیچیده در LINQ to Objects) ممکن است باعث سربار حافظه اضافی برای تولید متادیتای نوع جدید شود.

<a name="decision-guide"></a>
## ۹. Checklist نهایی و Decision Guide

### ✅ Checklist برای استفاده از Anonymous Type
- [ ] آیا فقط می‌خواهم داده‌ها را در یک متد خاص (Local Scope) شکل دهم؟
- [ ] آیا در حال نوشتن یک کوئری LINQ (مثل `Select` یا `Join`) هستم؟
- [ ] آیا نیازی به تغییر مقادیر پس از ساخت ندارم؟
- [ ] آیا نیازی به بازگرداندن این داده به متد دیگری یا لایه دیگری ندارم؟
- [ ] آیا نام Propertyها برای خوانایی کوئری برایم مهم است (نسبت به Tuple)؟

*اگر پاسخ همه موارد «بله» است، Anonymous Type انتخاب عالی است.*

---

### 🧭 اگر شک داشتم Anonymous Type استفاده کنم یا Class/Record/Tuple، چه تصمیمی بگیرم؟

برای تصمیم‌گیری سریع، از این **الگوریتم تصمیم‌گیری (Decision Guide)** استفاده کنید:

```text
آیا داده نیاز به بازگرداندن از متد (Return) یا پاس دادن به متد دیگر (Parameter) دارد؟
├── بله 👈 آیا نام Propertyها برای خوانایی مهم است؟
│   ├── بله 👈 آیا داده تغییرناپذیر (Immutable) است؟
│   │   ├── بله 👈 از `record` استفاده کنید. (بهترین انتخاب مدرن)
│   │   └── خیر 👈 از `class` استفاده کنید.
│   └── خیر 👈 از `Tuple` (ValueTuple) استفاده کنید (سبک‌ترین و سریع‌ترین).
│
└── خیر 👈 (داده فقط در Scope محلی و موقت است)
    ├── آیا در حال کار با LINQ / EF Core هستم؟
    │   ├── بله 👈 از `Anonymous Type` استفاده کنید.
    │   └── خیر 👈 آیا فقط ۲ یا ۳ مقدار ساده بدون نیاز به نام Property دارم؟
    │       ├── بله 👈 از `Tuple` استفاده کنید.
    │       └── خیر 👈 از `Anonymous Type` یا `record` محلی استفاده کنید.
```

**خلاصه قانون طلایی:**
> از **Anonymous Type** فقط برای **پروژکت کردن (Shape کردن) داده‌ها در LINQ** استفاده کنید. برای هر چیز دیگری که نیاز به انتقال بین لایه‌ها یا متدها دارد، از **`record`** (برای داده‌های Immutable) یا **`class`** (برای داده‌های Mutable) استفاده کنید.

---

<a name="جمع-بندی"></a>
## ۱۰. جمع‌بندی
Anonymous Typeها یک ویژگی کامپایلر-محور (Compiler-driven) برای تسهیل کار با LINQ و داده‌های موقت محلی هستند. آن‌ها خوانایی کد را در کوئری‌ها به‌شدت افزایش می‌دهند و از سربار تعریف کلاس‌های یک‌بار مصرف (One-off classes) جلوگیری می‌کنند. با این حال، به دلیل محدودیت در Scope و عدم پشتیبانی از `with` expression، نباید آن‌ها را با `record`ها اشتباه گرفت. رعایت مرز بین "داده‌های محلی" و "قراردادهای نرم‌افزار" کلید استفاده صحیح از این ویژگی است.

---

<a name="منابع"></a>
## ۱۱. منابع و مطالعه بیشتر

برای اطمینان از صحت مطالب، این مقاله بر اساس مستندات رسمی مایکروسافت و مشخصات زبان C# تدوین شده است:

1. **Microsoft Learn - Anonymous Types**
   * *موضوع استفاده شده:* تعاریف پایه، محدودیت‌ها، Type Identity و Immutability.
   * *لینک:* [Anonymous Types | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types)

2. **C# Language Specification (ECMA-334 / C# 11 Spec)**
   * *موضوع استفاده شده:* قوانین دقیق Type Identity (شرایط نام، نوع و ترتیب Propertyها).
   * *لینک:* [C# Language Specification - Anonymous object creation expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#117155-anonymous-object-creation-expressions)

3. **Microsoft Learn - Records (C# 9 and later)**
   * *موضوع استفاده شده:* تفاوت‌های `record` با Anonymous Type و بررسی قابلیت `with` expression.
   * *لینک:* [Record types | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)

4. **Microsoft Learn - EF Core Query Projections**
   * *موضوع استفاده شده:* نحوه ترجمه Anonymous Types به SQL در متد `Select`.
   * *لینک:* [Complex query projections - EF Core](https://learn.microsoft.com/en-us/ef/core/querying/complex-query-operators#projections)

5. **Microsoft Learn - Tuples**
   * *موضوع استفاده شده:* مقایسه با Anonymous Types و استفاده به‌عنوان جایگزین برای Return Type.
   * *لینک:* [Tuples | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)