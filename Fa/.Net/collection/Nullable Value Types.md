

# 📦 راهنمای جامع Nullable Value Types در سی‌شارپ (از مقدماتی تا پیشرفته)

به مستند آموزشی **Nullable Value Types** خوش آمدید! این راهنما برای درک عمیق یکی از مهم‌ترین مفاهیم سی‌شارپ، یعنی «چگونه با مقادیر خالی (Null) در انواع مقداری کار کنیم» طراحی شده است.

## 📑 فهرست مطالب
1. [مقدمه: چرا به Nullable نیاز داریم؟](#1-مقدمه-چرا-به-nullable-نیاز-داریم)
2. [مفهوم Nullable Value Types و سینتکس `?`](#2-مفهوم-nullable-value-types-و-سینتکس-)
3. [ساختار داخلی `Nullable<T>` (زیر کاپوت)](#3-ساختار-داخلی-nullablet-زیر-کاپوت)
4. [ویژگی‌های کلیدی: `HasValue` و `Value`](#4-ویژگیهای-کلیدی-hasvalue-و-value)
5. [رفتار Nullable در حالت Null و مقدار پیش‌فرض](#5-رفتار-nullable-در-حالت-null-و-مقدار-پیشفرض)
6. [خطر `InvalidOperationException` و نحوه جلوگیری از آن](#6-خطر-invalidoperationexception-و-نحوه-جلوگیری-از-آن)
7. [راه‌حل امن: متد `GetValueOrDefault()`](#7-راه‌حل-امن-متد-getvalueordefault)
8. [مفهوم `default(T?)`](#8-مفهوم-defaultt)
9. [نکات پیشرفته و مدرن (C# 9 به بعد)](#9-نکات-پیشرفته-و-مدرن)
10. [منابع معتبر برای مطالعه بیشتر](#10-منابع-معتبر-برای-مطالعه-بیشتر)

---

## 1. مقدمه: چرا به Nullable نیاز داریم؟
در سی‌شارپ، ما دو دسته اصلی نوع داده داریم:
* **انواع مرجعی (Reference Types):** مثل `string` یا کلاس‌ها. این‌ها می‌توانند مقدار `null` (تهی) را بپذیرند.
* **انواع مقداری (Value Types):** مثل `int`, `bool`, `DateTime`. این‌ها **نمی‌توانند** `null` باشند. یک `int` همیشه باید یک عدد داشته باشد.

**مشکل کجاست؟** 
تصور کنید در حال کار با یک دیتابیس هستید. ستونی در دیتابیس وجود دارد که ممکن است مقدار نداشته باشد (NULL). اما وقتی می‌خواهید آن را در یک متغیر `int` در سی‌شارپ بخوانید، با خطا مواجه می‌شوید، چون `int` سی‌شارپ نمی‌تواند خالی باشد! 
اینجاست که **Nullable Value Types** وارد میدان می‌شوند.

---

## 2. مفهوم Nullable Value Types و سینتکس `?`
کلمه `Nullable` به این معناست که «این متغیر از نوع مقداری است، اما **اجازه دارد** که خالی (null) باشد».

برای تعریف آن، کافیست یک علامت سوال `?` به انتهای نوع مقداری اضافه کنید:

```csharp
int regularInt = 10;       // مجاز
// int nullInt = null;      // ❌ خطا! انواع مقداری نمی‌توانند null باشند.

int? nullableInt = 10;     // ✅ مجاز
nullableInt = null;        // ✅ مجاز! حالا می‌تواند خالی باشد.
```
> 💡 **نکته مبتدی:** علامت `?` فقط یک میانبر (Syntax Sugar) است. در واقعیت، کامپایلر آن را به `Nullable<int>` تبدیل می‌کند.

---

## 3. ساختار داخلی `Nullable<T>` (زیر کاپوت)
بسیاری از مبتدیان فکر می‌کنند `Nullable` یک جادوی کامپایلر است، اما اینطور نیست! 
`Nullable<T>` در واقع یک **ساختار (Struct)** در کتابخانه پایه .NET است.

```csharp
// ساختار تقریبی Nullable<T> در دات‌نت:
public struct Nullable<T> where T : struct
{
    private readonly bool hasValue;
    private readonly T value;
    // ...
}
```
**نکات مهم ساختاری:**
1. چون خودش یک `struct` است، پس **Value Type** محسوب می‌شود.
2. محدودیت `where T : struct` یعنی شما فقط می‌توانید آن را با **انواع مقداری** استفاده کنید (نمی‌توانید `Nullable<string>` بنویسید چون string مرجعی است).
3. این ساختار، مقدار اصلی شما را درون خود **پکیج (Wrap)** می‌کند.

---

## 4. ویژگی‌های کلیدی: `HasValue` و `Value`
ساختار `Nullable<T>` دو عضو بسیار مهم دارد که باید مثل کف دستتان بشناسید:

### الف) `HasValue` (آیا مقداری وجود دارد؟)
یک خاصیت `bool` است که به شما می‌گوید آیا متغیر شما مقدار دارد یا خالی (null) است.
```csharp
int? myNumber = null;
Console.WriteLine(myNumber.HasValue); // خروجی: False

myNumber = 42;
Console.WriteLine(myNumber.HasValue); // خروجی: True
```

### ب) `Value` (مقدار واقعی)
خاصیتی که مقدار اصلی (پکیج شده) را برمی‌گرداند.
```csharp
int? myNumber = 42;
Console.WriteLine(myNumber.Value); // خروجی: 42
```

---

## 5. رفتار Nullable در حالت Null و مقدار پیش‌فرض
وقتی یک `Nullable` را `null` می‌کنید، در واقع در پشت صحنه، `HasValue` را روی `false` قرار داده‌اید. 

**مقدار پیش‌فرض (Default):**
اگر یک متغیر Nullable را بدون مقدار اولیه تعریف کنید، مقدار پیش‌فرض آن `null` است (نه صفر!).
```csharp
int? defaultNumber; 
Console.WriteLine(defaultNumber.HasValue); // False
Console.WriteLine(defaultNumber == null);  // True
```

---

## 6. خطر `InvalidOperationException` و نحوه جلوگیری از آن
بزرگترین دام برای مبتدیان، دسترسی مستقیم به `.Value` بدون بررسی `.HasValue` است.

```csharp
int? myNumber = null;

// ❌ این کد در زمان اجرا (Runtime) کرش می‌کند و خطا می‌دهد:
int result = myNumber.Value; 
// خطا: System.InvalidOperationException: Nullable object must have a value.
```

**✅ روش صحیح (سنتی):**
همیشه قبل از خواندن `.Value`، چک کنید که `HasValue` درست باشد:
```csharp
int? myNumber = null;

if (myNumber.HasValue)
{
    int safeResult = myNumber.Value;
    Console.WriteLine(safeResult);
}
else
{
    Console.WriteLine("مقداری وجود ندارد!");
}
```

---

## 7. راه‌حل امن: متد `GetValueOrDefault()`
برای اینکه از شر `if/else` و خطر `InvalidOperationException` خلاص شوید، سی‌شارپ یک متد فوق‌العاده به نام `GetValueOrDefault()` دارد.

این متد دو حالت دارد:

**حالت اول: بدون آرگومان**
اگر متغیر مقدار داشته باشد، همان را برمی‌گرداند. اگر `null` باشد، **مقدار پیش‌فرضِ نوعِ اصلی** (مثلاً `0` برای `int`) را برمی‌گرداند.
```csharp
int? myNumber = null;
int result = myNumber.GetValueOrDefault(); 
Console.WriteLine(result); // خروجی: 0 (بدون هیچ خطایی!)
```

**حالت دوم: با آرگومان (پیشنهادی)**
می‌توانید به متد بگویید اگر `null` بود، چه مقداری را به عنوان جایگزین برگرداند:
```csharp
int? myNumber = null;
int result = myNumber.GetValueOrDefault(-1); 
Console.WriteLine(result); // خروجی: -1 (مقدار پیش‌فرض دلخواه ما)
```

---

## 8. مفهوم `default(T?)`
کلمه کلیدی `default` در سی‌شارپ برای تولید «مقدار پیش‌فرض» یک نوع داده استفاده می‌شود.
برای انواع مقداری معمولی، `default` یعنی `0` یا `false`. اما برای `Nullable<T>` داستان متفاوت است!

مقدار پیش‌فرض برای `Nullable<T>` همیشه **`null`** است.

```csharp
int? a = default;       // دقیقاً معادل int? a = null;
bool? b = default;      // دقیقاً معادل bool? b = null;
DateTime? c = default;  // دقیقاً معادل DateTime? c = null;

Console.WriteLine(a.HasValue); // False
```
> 🧠 **نکته پیشرفته:** در واقع `default(Nullable<T>)` یک ساختار `Nullable` برمی‌گرداند که در آن `hasValue = false` و `value = default(T)` تنظیم شده است. اما چون `hasValue` منفی است، کل شیء `null` در نظر گرفته می‌شود.

---

## 9. نکات پیشرفته و مدرن
برای اینکه کدهای شما حرفه‌ای و تمیز (Clean) باشد، از عملگرهای مدرن سی‌شارپ استفاده کنید:

### الف) عملگر Null-Coalescing (`??`)
این عملگر جایگزین بسیار شیکی برای `GetValueOrDefault()` است:
```csharp
int? myNumber = null;
int result = myNumber ?? 100; // اگر myNumber نال بود، 100 را در result بریز
```

### ب) تطبیق الگو (Pattern Matching) در C# 9+
شما می‌توانید همزمان هم `null` بودن را چک کنید و هم مقدار را استخراج کنید:
```csharp
int? myNumber = 42;

if (myNumber is int actualValue)
{
    // این بلاک فقط اجرا می‌شود که myNumber نال نباشد
    Console.WriteLine($"مقدار واقعی: {actualValue}"); 
}
```

---

## 10. منابع معتبر برای مطالعه بیشتر
این مستند بر اساس منابع رسمی و کتاب‌های مرجع دات‌نت تهیه شده است. برای عمیق‌تر شدن، حتماً این منابع را بررسی کنید:

1. **[Microsoft Learn - Nullable value types (Official Docs)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)**
   * *توضیح:* مستندات رسمی مایکروسافت؛ بهترین نقطه شروع برای درک سینتکس و رفتار کامپایلر.
2. **[Microsoft Learn - Nullable<T> Structure](https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1?view=net-8.0)**
   * *توضیح:* بررسی دقیق APIها، متدها و ساختار داخلی `Nullable<T>` در دات‌نت.
3. **کتاب "C# in Depth" نوشته Jon Skeet (فصل مربوط به Nullable Types)**
   * *توضیح:* جان اسکیث (نویسنده افسانه‌ای StackOverflow) بهترین توضیحات را در مورد تفاوت‌های زیرپوستی و رفتار کامپایلر با `Nullable` ارائه داده است.
4. **کتاب "C# via CLR" نوشته Jeffrey Richter (فصل 17 و 19)**
   * *توضیح:* برای درک اینکه `Nullable` چگونه در سطح CLR (Common Language Runtime) و در حافظه (Stack/Heap) مدیریت می‌شود، این کتاب مرجع نهایی است.

---
*تهیه و تنظیم شده برای ریپازیتوری آموزشی سی‌شارپ | آخرین بروزرسانی: مرداد ۱۴۰۵ (August 2026)*
*اگر این آموزش برای شما مفید بود، فراموش نکنید که به ریپازیتوری ستاره (⭐) بدهید!*