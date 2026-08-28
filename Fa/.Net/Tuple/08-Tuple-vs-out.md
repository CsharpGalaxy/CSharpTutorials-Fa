# آموزش جامع Tuple در C#: هدف اصلی؛ بازگرداندن چند مقدار از یک متد

به نام خدا

به بخش **Tupleها** در آموزش C# خوش آمدید! در این مقاله از ریپازیتوری آموزشی، به یکی از کاربردی‌ترین و در عین حال جذاب‌ترین ویژگی‌های مدرن C# می‌پردازیم. اگر تا به حال با متدهایی سروکار داشته‌اید که نیاز به بازگرداندن بیش از یک مقدار داشته‌اند، این مقاله دقیقاً برای شما نوشته شده است.

---

## 📑 فهرست مطالب
1. [مشکل متدهایی که چند مقدار برمی‌گردانند](#۱-مشکل-متدهایی-که-چند-مقدار-برمی‌گردانند)
2. [روش‌های قدیمی و جایگزین (قبل از Tuple)](#۲-روشهای-قدیمی-و-جایگزین)
   - [الف) استفاده از پارامترهای out](#الف-استفاده-از-پارامترهای-out)
   - [ب) استفاده از Class](#ب-استفاده-از-class)
   - [ج) استفاده از Struct](#ج-استفاده-از-struct)
   - [د) استفاده از DTO](#د-استفاده-از-dto)
3. [انقلاب Tuple: مثال قبل و بعد](#۳-انقلاب-tuple-مثال-قبل-و-بعد)
4. [Tuple نام‌گذاری شده (Named Tuples)](#۴-tuple-نامگذاری-شده-named-tuples)
5. [استفاده از Tuple در متدهای واقعی](#۵-استفاده-از-tuple-در-متدهای-واقعی)
6. [مزایا و معایب استفاده از Tuple](#۶-مزایا-و-معایب-استفاده-از-tuple)
7. [جدول مقایسه: Tuple در برابر out](#۷-جدول-مقایسه-tuple-در-برابر-out)
8. [ماتریس تصمیم‌گیری: Tuple، DTO یا Record؟](#۸-ماتریس-تصمیمگیری-tuple-dto-یا-record)
9. [نکات مهم و اشتباهات رایج](#۹-نکات-مهم-و-اشتباهات-رایج)
10. [جمع‌بندی](#۱۰-جمع‌بندی)
11. [منابع معتبر](#۱۱-منابع-معتبر)

---

## ۱. مشکل متدهایی که چند مقدار برمی‌گردانند

در برنامه‌نویسی، یک متد (Method) طبق تعریف، تنها می‌تواند **یک مقدار** را به عنوان خروجی (Return) داشته باشد. اما در دنیای واقعی، ما اغلب به متدهایی نیاز داریم که چندین مقدار مرتبط را محاسبه یا دریافت کنند. 

**مثال:** فرض کنید متدی دارید که مختصات یک نقطه را روی صفحه پیدا می‌کند. شما هم به `X` نیاز دارید و هم به `Y`. یا متدی که اطلاعات کاربر را از دیتابیس می‌گیرد و هم `نام کاربر` و هم `وضعیت حساب` را برمی‌گرداند. چالش اصلی اینجاست: **چگونه چندین مقدار را از یک متد خارج کنیم؟**

---

## ۲. روش‌های قدیمی و جایگزین

قبل از معرفی `ValueTuple` در نسخه C# 7.0، برنامه‌نویسان مجبور بودند از راه‌حل‌های زیر استفاده کنند:

### الف) استفاده از پارامترهای `out`
در این روش، متد به جای برگرداندن مقدار، آن‌ها را در متغیرهایی که با کلمه کلیدی `out` پاس داده شده‌اند، تزریق می‌کند.

```csharp
// روش قدیمی با out
public bool TryGetUserInfo(int userId, out string userName, out int accountStatus)
{
    // ... منطق دریافت از دیتابیس
    userName = "Ali";
    accountStatus = 1;
    return true;
}

// نحوه فراخوانی
TryGetUserInfo(101, out string name, out int status);
```
**ایراد:** امضای متد (Signature) را شلوغ می‌کند، خوانایی را کاهش می‌دهد و در LINQ یا متدهای `async` به شدت آزاردهنده است.

### ب) استفاده از `Class`
می‌توانید یک کلاس جدید فقط برای نگهداری این مقادیر بسازید.

```csharp
public class UserResult
{
    public string UserName { get; set; }
    public int AccountStatus { get; set; }
}

public UserResult GetUserInfo(int userId)
{
    return new UserResult { UserName = "Ali", AccountStatus = 1 };
}
```
**ایراد:** ایجاد یک کلاس جدید فقط برای یک متد داخلی، اضافه‌کاری (Overkill) است و چون Reference Type است، سربار حافظه (Heap Allocation) دارد.

### ج) استفاده از `Struct`
دقیقاً مانند کلاس، اما به عنوان یک Value Type.

```csharp
public struct UserResult
{
    public string UserName;
    public int AccountStatus;
}
```
**ایراد:** همچنان نیاز به تعریف یک Type جدید دارید و اگر `readonly` نباشد، ممکن است در حین اجرا تغییر کند (Mutability) که باگ‌های عجیبی ایجاد می‌کند.

### د) استفاده از `DTO` (Data Transfer Object)
الگوی DTO برای انتقال داده بین لایه‌هاست، اما گاهی به اشتباه برای بازگرداندن مقادیر داخلی متدها هم استفاده می‌شود.
**ایراد:** DTOها معمولاً برای مرزهای سیستم (مثل APIها) طراحی می‌شوند. استفاده از آن‌ها برای منطق داخلی کلاس، نقض اصل Separation of Concerns است.

---

## ۳. انقلاب Tuple: مثال قبل و بعد

با ورود C# 7.0، `ValueTuple` معرفی شد تا این مشکل را برای همیشه حل کند. تاپل‌ها به شما اجازه می‌دهند چندین مقدار را در یک بسته‌بندی سبک (Value Type) بدون نیاز به تعریف کلاس یا استراکت برگردانید.

### ❌ قبل از Tuple (با استفاده از System.Tuple قدیمی)
```csharp
public Tuple<string, int> GetUserInfo(int userId)
{
    return new Tuple<string, int>("Ali", 1);
}

var result = GetUserInfo(101);
Console.WriteLine(result.Item1); // Item1 یعنی چی؟! خوانایی صفر!
```

### ✅ بعد از Tuple (با استفاده از ValueTuple مدرن)
```csharp
public (string, int) GetUserInfo(int userId)
{
    return ("Ali", 1);
}

var result = GetUserInfo(101);
Console.WriteLine(result.Item1); // هنوز هم خوانایی خوبی ندارد، اما صبر کنید...
```

---

## ۴. Tuple نام‌گذاری شده (Named Tuples)

برای حل مشکل `Item1` و `Item2`، سی‌شارپ به شما اجازه می‌دهد برای عناصر Tuple **نام** تعیین کنید. این نام‌ها فقط در زمان کامپایل (Compile-time) برای کمک به IntelliSense و خوانایی کد شما هستند.

```csharp
// تعریف متد با Tuple نام‌گذاری شده
public (string UserName, int AccountStatus) GetUserInfo(int userId)
{
    return (UserName: "Ali", AccountStatus: 1);
}

// فراخوانی و استفاده
var result = GetUserInfo(101);
Console.WriteLine(result.UserName);       // عالی! کاملاً خوانا
Console.WriteLine(result.AccountStatus);  // دقیق و بدون ابهام
```

> 💡 **نکته طلایی (Deconstruction):** شما می‌توانید Tuple را در همان لحظه دریافت، تجزیه (Deconstruct) کنید:
> ```csharp
> var (name, status) = GetUserInfo(101);
> Console.WriteLine($"Name: {name}, Status: {status}");
> ```

---

## ۵. استفاده از Tuple در متدهای واقعی

بیایید یک مثال واقعی‌تر را بررسی کنیم. فرض کنید متدی دارید که یک آدرس ایمیل را پارس می‌کند و نام کاربر و دامنه را جدا می‌کند:

```csharp
public (string User, string Domain, bool IsValid) ParseEmail(string email)
{
    if (string.IsNullOrWhiteSpace(email) || !email.Contains("@"))
    {
        return (User: string.Empty, Domain: string.Empty, IsValid: false);
    }

    var parts = email.Split('@');
    return (User: parts[0], Domain: parts[1], IsValid: true);
}

// استفاده در برنامه
var (user, domain, isValid) = ParseEmail("test@example.com");

if (isValid)
{
    Console.WriteLine($"User: {user}, Domain: {domain}");
}
```

---

## ۶. مزایا و معایب استفاده از Tuple

### ✅ مزایا
1. **خوانایی بالا:** با Named Tuples، کد خودتوضیح‌دهنده (Self-documenting) می‌شود.
2. **عملکرد عالی:** چون `ValueTuple` یک Value Type است، روی Stack (یا درون اشیاء دیگر) قرار می‌گیرد و سربار Garbage Collection ندارد.
3. **کاهش Boilerplate:** نیازی به تعریف کلاس یا استراکت‌های اضافی برای کارهای موقت نیست.
4. **سازگاری با LINQ و Async:** به راحتی در `async/await` و کوئری‌های LINQ قابل استفاده است (برخلاف `out`).

### ❌ معایب
1. **عدم شفافیت در APIهای عمومی:** اگر در متدهای Public یک Library از Tuple استفاده کنید، مصرف‌کننده کتابخانه ممکن است گیج شود (بهتر است از Record یا Class استفاده شود).
2. **نام‌ها در Runtime از بین می‌روند:** نام‌های Tuple فقط در زمان کامپایل هستند. اگر از Reflection استفاده کنید، فقط `Item1` و `Item2` را می‌بینید (مگر اینکه از Attributeها استفاده کنید که پیچیده است).
3. **محدودیت تعداد:** به طور پیش‌فرض تا ۷ عنصر را مستقیماً پشتیبانی می‌کند (بیشتر از آن نیاز به Tupleهای تودرتو دارد که خوانایی را از بین می‌برد).

---

## ۷. جدول مقایسه: Tuple در برابر `out`

| ویژگی | Tuple (ValueTuple) | پارامتر `out` |
| :--- | :--- | :--- |
| **نحوه بازگشت** | بازگشت مستقیم از طریق `return` | تزریق در متغیرهای پاس داده شده |
| **خوانایی کد** | بسیار بالا (به ویژه با Named Tuple) | پایین (امضای متد شلوغ می‌شود) |
| **سازگاری با Async** | ✅ کاملاً سازگار | ❌ در متدهای `async` پشتیبانی نمی‌شود |
| **سازگاری با LINQ** | ✅ قابل استفاده در Select و ... | ❌ غیرقابل استفاده |
| **نوع داده** | Value Type (سبک و سریع) | بستگی به نوع متغیر دارد |
| **Immutability** | ✅ (بهتر است تغییر داده نشوند) | ❌ (متغیرها قابل تغییر هستند) |

---

## ۸. ماتریس تصمیم‌گیری: Tuple، DTO یا Record؟

چه زمانی از کدام ابزار استفاده کنیم؟ این جدول راهنمای شماست:

| سناریو | بهترین انتخاب | دلیل |
| :--- | :--- | :--- |
| **بازگرداندن مقادیر در متدهای Internal/Private** | **Tuple** | سبک، سریع، بدون نیاز به تعریف Type جدید. |
| **انتقال داده بین لایه‌ها (مثلاً UI به API)** | **DTO (Class/Record)** | ساختاریافته، قابل Serialize، دارای Validation. |
| **بازگرداندن داده در Public APIها و Libraryها** | **Record (یا Class)** | دارای Contract مشخص، قابل مستندسازی، دارای Equality. |
| **گروه‌بندی موقت داده‌ها در یک متد (مثلاً در LINQ)** | **Tuple** | بسیار تمیز و سریع برای `Select(x => (x.Id, x.Name))`. |

> 🔗 **لینک داخلی:** برای درک عمیق‌تر تفاوت Tuple با [Recordها در C#](./05-Records.md)، پیشنهاد می‌کنم مقاله مربوط به Records را در این ریپازیتوری مطالعه کنید.

---

## ۹. نکات مهم و اشتباهات رایج

### ⚠️ اشتباه رایج ۱: استفاده از Tuple برای بیش از ۴ یا ۵ عنصر
**غلط:** `(int, string, bool, DateTime, double, float, Guid)`
**درست:** اگر متد شما نیاز به بازگرداندن این همه داده دارد، یعنی متد شما بیش از یک وظیفه انجام می‌دهد (نقض Single Responsibility). یک `Class` یا `Record` بسازید.

### ⚠️ اشتباه رایج ۲: فراموش کردن نام‌گذاری عناصر
استفاده از `Item1`, `Item2` در کدهای طولانی باعث می‌شود بعد از یک هفته، خودتان هم نفهمید `Item3` دقیقاً چه مقداری را نگه می‌دارد. **همیشه** از Named Tuples استفاده کنید.

### ⚠️ اشتباه رایج ۳: تغییر دادن مقادیر Tuple
تاپل‌ها باید **Immutable** (غیرقابل تغییر) در نظر گرفته شوند.
```csharp
var result = GetUserInfo(101);
result.UserName = "Reza"; // ❌ کار اشتباه! Tuple باید ثابت بماند.
```

### 💡 نکته مهم: تطبیق الگو (Pattern Matching) با Tuple
شما می‌توانید در `switch` expressions از Tupleها استفاده کنید:
```csharp
public string GetAccessLevel((bool IsAdmin, bool IsPremium) userStatus)
{
    return userStatus switch
    {
        (true, _) => "Full Access",
        (false, true) => "Premium Access",
        (false, false) => "Basic Access"
    };
}
```

---

## ۱۰. جمع‌بندی

هدف اصلی `Tuple` در سی‌شارپ، **بازگرداندن چند مقدار از یک متد** به شکلی تمیز، خوانا و با عملکرد بالا است. 
* اگر در حال نوشتن منطق داخلی (Internal) یک کلاس هستید و می‌خواهید ۲ تا ۴ مقدار مرتبط را برگردانید، **Tuple** بهترین دوست شماست.
* اگر در حال طراحی مرزهای سیستم (Public API، لایه‌های ارتباطی) هستید، به سراغ **Record** یا **DTO** بروید.

با درک درست از Tupleها، کدهای شما از حالت پر از `out` و کلاس‌های بی‌مصرف (`God Classes`) خارج شده و به کدهایی مدرن، Functional و خوانا تبدیل می‌شود.

---

## ۱۱. منابع معتبر

برای مطالعه بیشتر و عمیق‌تر، منابع زیر پیشنهاد می‌شوند:
1. **[Microsoft Learn - Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/tuples)** - مستندات رسمی و جامع مایکروسافت.
2. **[C# in Depth (4th Edition) by Jon Skeet](https://csharpindepth.com/)** - فصل مربوط به Tupleها و Value Types.
3. **[C# Language Specification - Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/structured-types)** - مشخصات دقیق زبان برای علاقه‌مندان به مباحث زیرساختی.

---
*نویسنده: تیم آموزشی ریپازیتوری C# | تاریخ بازبینی: August 2026*
*در صورت داشتن سوال یا پیشنهاد برای بهبود این مقاله، خوشحال می‌شویم [Issue](../../issues) ثبت کنید یا Pull Request بفرستید!*