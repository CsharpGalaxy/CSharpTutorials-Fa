

# 📘 راهنمای جامع تبدیل‌های Nullable در سی‌شارپ (از مقدماتی تا پیشرفته)

در سی‌شارپ، مدیریت مقادیر `null` یکی از مهم‌ترین مباحث برای جلوگیری از خطاهای زمان اجرا (Runtime Errors) است. در این آموزش، ما به طور کامل مکانیزم تبدیل بین انواع داده‌ای معمولی (`T`) و انواع داده‌ای تهی‌پذیر (`T?`) را بررسی می‌کنیم.

## 📑 فهرست مطالب
1. [مفهوم پایه: چرا به Nullable نیاز داریم؟](#1-مفهوم-پایه-چرا-به-nullable-نیاز-داریم)
2. [تبدیل `T` به `T?` (تبدیل ضمنی)](#2-تبدیل-t-به-t-تبدیل-ضمنی-implicit)
3. [تبدیل `T?` به `T` (تبدیل صریح)](#3-تبدیل-t-به-t-تبدیل-صریح-explicit)
4. [نقش `Value` و احتمال وقوع Exception](#4-نقش-value-و-احتمال-وقوع-exception)
5. [استفاده امن از `GetValueOrDefault()`](#5-استفاده-امن-از-getvalueordefault)
6. [مبحث پیشرفته: تفاوت Value Type و Reference Type در Nullable](#6-مبحث-پیشرفته-تفاوت-nullable-value-types-و-nullable-reference-types)
7. [جمع‌بندی و بهترین روش‌ها (Best Practices)](#7-جمع‌بندی-و-بهترین-روش‌ها)
8. [منابع معتبر](#8-منابع-معتبر)

---

## 1. مفهوم پایه: چرا به Nullable نیاز داریم؟
در سی‌شارپ، **انواع مقداری (Value Types)** مانند `int`, `double`, `bool` و `DateTime` به طور پیش‌فرض **نمی‌توانند** مقدار `null` را بپذیرد. آن‌ها همیشه یک مقدار دارند (مثلاً `int` به طور پیش‌فرض `0` است).
اما در دنیای واقعی (مثل ارتباط با دیتابیس یا API)، ممکن است یک فیلد عددی هنوز مقداری دریافت نکرده باشد. در این شرایط ما به `T?` (تهی‌پذیر) نیاز داریم.

---

## 2. تبدیل `T` به `T?` (تبدیل ضمنی / Implicit)

تبدیل یک نوع داده‌ای معمولی به نوع تهی‌پذیر، **همیشه امن** است و نیازی به_cast_ (تبدیل صریح) ندارد. کامپایلر سی‌شارپ این کار را به صورت **ضمنی (Implicit)** برای شما انجام می‌دهد.

### 💻 مثال:
```csharp
int normalValue = 10;
int? nullableValue = normalValue; // ✅ تبدیل ضمنی - بدون نیاز به ()

// در پشت صحنه، کامپایلر این کار را می‌کند:
// Nullable<int> nullableValue = new Nullable<int>(normalValue);
```
**نکته برای مبتدیان:** چون `normalValue` قطعاً دارای یک مقدار است، قرار دادن آن درون یک ظرفِ تهی‌پذیر (`T?`) هیچ خطری ندارد و کامپایلر با خوشحالی این کار را انجام می‌دهد.

---

## 3. تبدیل `T?` به `T` (تبدیل صریح / Explicit)

برخلاف مورد قبل، تبدیل یک نوع تهی‌پذیر به نوع معمولی **همیشه با ریسک** همراه است. چون ممکن است متغیر تهی‌پذیر ما، در واقع مقدار `null` باشد. به همین دلیل، کامپایلر از شما می‌خواهد که **تبدیل صریح (Explicit)** انجام دهید تا بگویید: *"من می‌دانم چه کار می‌کنم و ریسک آن را می‌پذیرم"*.

### 💻 مثال:
```csharp
int? nullableValue = 20;
int normalValue = (int)nullableValue; // ✅ تبدیل صریح با استفاده از ()
```

⚠️ **هشدار بزرگ:** اگر `nullableValue` برابر با `null` باشد و شما سعی کنید آن را به `int` تبدیل کنید، برنامه شما **کرش (Crash)** خواهد کرد! (در بخش بعد دلیل آن را می‌خوانیم).

---

## 4. نقش `Value` و احتمال وقوع Exception

در سی‌شارپ، `T?` برای انواع مقداری (Value Types) در واقع یک ساختار (Struct) به نام `System.Nullable<T>` است. این ساختار دو عضو بسیار مهم دارد:
1. **`HasValue`**: یک `bool` که نشان می‌دهد آیا متغیر مقدار دارد یا `null` است.
2. **`Value`**: مقداری که درون آن ذخیره شده است.

### 💥 خطر استفاده مستقیم از `.Value`
اگر شما مستقیماً از `nullableValue.Value` استفاده کنید و متغیر شما `null` باشد، سی‌شارپ خطای زیر را پرتاب می‌کند:
**`InvalidOperationException: Nullable object must have a value.`**

### 💻 مثال خطرناک:
```csharp
int? myNumber = null;

// ❌ این خط باعث پرتاب InvalidOperationException می‌شود!
int result = myNumber.Value; 
```

### 🛡️ راه حل سنتی (استفاده از HasValue):
قبل از خواندن `Value`، همیشه باید چک کنید که `HasValue` برابر با `true` باشد:
```csharp
int? myNumber = null;
int result;

if (myNumber.HasValue)
{
    result = myNumber.Value; // ✅ امن
}
else
{
    result = 0; // مقدار پیش‌فرض در صورت null بودن
}
```

---

## 5. استفاده امن از `GetValueOrDefault()`

برای اینکه از شرِ چک کردن‌های طولانی `if (HasValue)` و خطر `Exception` راحت شوید، متد `GetValueOrDefault()` طراحی شده است. این متد در دو حالت استفاده می‌شود:

### حالت اول: بدون آرگومان
اگر متغیر `null` باشد، مقدار پیش‌فرضِ آن نوع داده‌ای (Default) را برمی‌گرداند (برای `int` برابر با `0`، برای `bool` برابر با `false`).
```csharp
int? myNumber = null;
int result = myNumber.GetValueOrDefault(); // نتیجه: 0
```

### حالت دوم: با آرگومان (مقدار جایگزین دلخواه)
شما می‌توانید تعیین کنید که در صورت `null` بودن، چه مقداری برگردانده شود.
```csharp
int? myNumber = null;
int result = myNumber.GetValueOrDefault(-1); // نتیجه: -1 (مقدار دلخواه ما)
```

💡 **نکته حرفه‌ای (Modern C#):** در نسخه‌های جدید سی‌شارپ (C# 2.0 به بعد)، استفاده از **عملگر Null-Coalescing (`??`)** بسیار رایج‌تر و خواناتر از `GetValueOrDefault` است:
```csharp
int result = myNumber ?? -1; // دقیقاً همان کار GetValueOrDefault(-1) را انجام می‌دهد
```

---

## 6. مبحث پیشرفته: تفاوت Nullable Value Types و Reference Types

تا اینجا در مورد `int?` صحبت کردیم که یک **Value Type** است. اما از سی‌شارپ 8.0 به بعد، مفهوم **Nullable Reference Types (NRT)** نیز اضافه شد (مثل `string?`).

### ⚠️ تفاوت‌های کلیدی که باید بدانید:
| ویژگی | Nullable Value Types (`int?`) | Nullable Reference Types (`string?`) |
| :--- | :--- | :--- |
| **ماهیت در CLR** | یک `Struct` به نام `Nullable<T>` | همان Reference معمولی (فقط یک Annotation برای کامپایلر) |
| **داشتن پراپرتی `.Value`** | ✅ بله دارد | ❌ خیر (خطای کامپایل می‌دهد) |
| **احتمال `InvalidOperationException`** | ✅ بله (در صورت استفاده از `.Value` روی null) | ❌ خیر (فقط `NullReferenceException` در زمان اجرا) |
| **تبدیل صریح `(T)`** | نیاز به Cast دارد | نیاز به Cast ندارد (فقط Warning کامپایلر) |

**مثال برای Reference Type:**
```csharp
#nullable enable
string? text = null;

// string? در واقع همان string است، پس پراپرتی .Value ندارد!
// string val = text.Value; // ❌ خطای کامپایل

string val2 = text; // ✅ بدون نیاز به Cast (اما کامپایلر Warning می‌دهد که ممکن است null باشد)
```

---

## 7. جمع‌بندی و بهترین روش‌ها (Best Practices)

برای اینکه کدی حرفه‌ای، امن و بدون باگ بنویسید، این قوانین را رعایت کنید:

1. 🚫 **هرگز** بدون چک کردن `HasValue` یا استفاده از Pattern Matching، از `.Value` استفاده نکنید.
2. 🚫 **هرگز** از تبدیل صریح `(int)myNullable` استفاده نکنید، مگر اینکه ۱۰۰٪ مطمئن باشید که متغیر `null` نیست.
3. ✅ **بهترین روش:** از عملگر `??` یا متد `GetValueOrDefault()` برای تعیین یک مقدار جایگزین (Fallback) استفاده کنید.
4. ✅ **روش مدرن (Pattern Matching):** از `is` برای چک کردن و استخراج مقدار در یک خط استفاده کنید:
   ```csharp
   int? myNumber = GetNumber();
   if (myNumber is int actualNumber)
   {
       // هم چک شد که null نیست، هم مقدار در actualNumber ریخته شد
       Console.WriteLine(actualNumber); 
   }
   ```

---

## 8. منابع معتبر

این مقاله بر اساس مستندات رسمی و کتب مرجع سی‌شارپ تدوین شده است. برای مطالعه عمیق‌تر، منابع زیر پیشنهاد می‌شوند:

1. **[Microsoft Learn - Nullable value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)**
   *مستندات رسمی مایکروسافت در مورد ساختار `Nullable<T>` و نحوه تبدیل‌ها.*
2. **[Microsoft Learn - Nullable Reference Types](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references)**
   *راهنمای کامل مایکروسافت در مورد ویژگی‌های سی‌شارپ 8.0 و مدیریت Null در Reference Typeها.*
3. **کتاب C# in Depth (نوشته Jon Skeet)**
   *فصل مربوط به Nullable Types در این کتاب مرجع، یکی از بهترین توضیحات در مورد نحوه پیاده‌سازی `Struct` برای `Nullable` در سطح CLR است.*
4. **[Microsoft Learn - Operator ?? and ??=](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator)**
   *مستندات عملگرهای جایگزین Null که جایگزین مدرن `GetValueOrDefault` هستند.*

---
*📝 **توجه برای مدیر ریپازیتوری:** این داکیومنت با فرمت استاندارد Markdown نوشته شده و لینک‌های فهرست مطالب (ToC) به صورت Anchor در گیت‌هاب به درستی کار خواهند کرد. می‌توانید این فایل را با نام `Nullable-Conversions-Guide.md` در پوشه `docs/` یا `articles/` ریپازیتوری خود قرار دهید.*