

## فهرست مطالب
1. [مقدمه و مفاهیم پایه](#1-مقدمه-و-مفاهیم-پایه)
2. [چرا Attribute به وجود آمد؟](#2-چرا-attribute-به-وجود-آمد)
3. [Attribute و Metadata](#3-attribute-و-metadata)
4. [چرخه حیات: Compiler و Runtime](#4-چرخه-حیات-compiler-و-runtime)
5. [تعریف کلاس Attribute](#5-تعریف-کلاس-attribute)
6. [قرارداد نام‌گذاری (Naming Convention)](#6-قرارداد-نام‌گذاری-naming-convention)
7. [Attributeهای آماده و کاربردی .NET](#7-attributeهای-آماده-و-کاربردی-net)
8. [ساخت Custom Attribute](#8-ساخت-custom-attribute)
9. [AttributeUsage و AttributeTargets](#9-attributeusage-و-attributetargets)
10. [آرگومان‌های Attribute (Positional و Named)](#10-آرگومان‌های-attribute-positional-و-named)
11. [انواع مجاز در آرگومان‌های Attribute](#11-انواع-مجاز-در-آرگومان‌های-attribute)
12. [اعمال Attribute روی عناصر مختلف کد](#12-اعمال-attribute-روی-عناصر-مختلف-کد)
13. [Reflection و خواندن Attributeها](#13-reflection-و-خواندن-attributeها)
14. [Attribute Constructor و لحظه اجرا](#14-attribute-constructor-و-لحظه-اجرا)
15. [Roslyn و AttributeData](#15-roslyn-و-attributedata)
16. [مقایسه Reflection و Roslyn](#16-مقایسه-reflection-و-roslyn)
17. [Attribute در اکوسیستم مدرن .NET](#17-attribute-در-اکوسیستم-مدرن-net)
18. [معماری، الگوها و طراحی](#18-معماری-الگوها-و-طراحی)
19. [Performance، AOT و Trimming](#19-performance-aot-و-trimming)
20. [اشتباهات رایج](#20-اشتباهات-رایج)
21. [مقایسه‌های کلیدی](#21-مقایسه‌های-کلیدی)
22. [پروژه عملی: Mini Framework](#22-پروژه-عملی-mini-framework)
23. [تست و Benchmark](#23-تست-و-benchmark)
24. [Cheat Sheet و راهنمای تصمیم‌گیری](#24-cheat-sheet-و-راهنمای-تصمیم‌گیری)
25. [جمع‌بندی](#25-جمع‌بندی)
26. [منابع رسمی](#26-منابع-رسمی)

---

## 1. مقدمه و مفاهیم پایه

### Attribute چیست؟
**Attribute (ویژگی)** یک نوع خاص از کلاس در C# است که برای افزودن **Metadata (فراداده)** یا اطلاعات توصیفی به عناصر کد (مانند کلاس‌ها، متدها، پراپرتی‌ها و...) استفاده می‌شود. Attribute به‌تنهایی هیچ رفتار اجرایی (Behavior) ندارد، بلکه مانند یک "برچسب" یا "یادداشت" عمل می‌کند که به Compiler، Runtime یا Frameworkها می‌گوید با آن عنصر کد چگونه رفتار کنند.

### مثال ساده
```csharp
[Obsolete("این متد منسوخ شده است، از NewMethod استفاده کنید.")]
public void OldMethod()
{
    // ...
}
```
در اینجا، `[Obsolete]` به کامپایلر می‌گوید که اگر جایی در کد `OldMethod` فراخوانی شد، یک هشدار (Warning) یا خطا (Error) نمایش دهد.

### Attribute چه مشکلی را حل می‌کند؟
قبل از Attributeها، توسعه‌دهندگان برای افزودن اطلاعات به کد مجبور بودند از فایل‌های پیکربندی خارجی (مثل XML) یا کامنت‌های متنی استفاده کنند که قابلیت اطمینان پایینی داشتند و با تغییر کد، همگام‌سازی نمی‌شدند. Attribute این اطلاعات را **مستقیماً به کد می‌چسباند**.

### تفاوت Data و Metadata
* **Data (داده):** اطلاعاتی که برنامه در حین اجرا پردازش می‌کند (مثلاً مقدار یک متغیر `string name = "Ali";`).
* **Metadata (فراداده):** "داده‌ای درباره داده". اطلاعاتی که ساختار، رفتار یا ویژگی‌های کد را توصیف می‌کند (مثلاً "این کلاس قابل Serialization است").

### نقش Attribute در سه سطح:
1. **Compiler:** ممکن است رفتار کامپایلر را تغییر دهد (مثل `[Obsolete]` یا `[Conditional]`).
2. **Runtime:** از طریق Reflection قابل خواندن است و می‌تواند رفتار برنامه را در حین اجرا تغییر دهد.
3. **Frameworkها:** فریم‌ورک‌هایی مثل ASP.NET Core یا Entity Framework از Attributeها برای پیکربندی Declarative (اعلامی) استفاده می‌کنند (مثل `[HttpGet]` یا `[Key]`).

---

## 2. چرا Attribute به وجود آمد؟

برای درک ارزش Attribute، روش‌های قدیمی افزودن اطلاعات به کد را مقایسه می‌کنیم:

| روش | توضیح | معایب |
| :--- | :--- | :--- |
| **1. اضافه کردن Keyword** | افزودن کلمات کلیدی جدید به زبان (مثل `virtual` یا `sealed`). | زبان را پیچیده می‌کند. برای هر نیاز جدید نمی‌توان یک Keyword جدید به C# اضافه کرد. |
| **2. فایل Configuration** | ذخیره اطلاعات در فایل‌های XML یا JSON جداگانه. | احتمال ناهماهنگی بین کد و کانفیگ (Refactoring کد، کانفیگ را آپدیت نمی‌کند). |
| **3. استفاده از Attribute** | چسباندن Metadata مستقیماً به عنصر کد در Source Code. | **راه‌حل ایده‌آل:** اطلاعات همیشه همراه کد است، توسط کامپایلر بررسی می‌شود و از Refactoring پشتیبانی می‌کند. |

---

## 3. Attribute و Metadata

### Metadata چیست؟
وقتی کد C# کامپایل می‌شود، خروجی فقط کد ماشین (IL) نیست. کامپایلر یک بخش جداگانه به نام **Metadata** تولید می‌کند که شامل اطلاعاتی درباره Assembly، Typeها، متدها، پراپرتی‌ها و پارامترهاست.

### سطوح Metadata:
* **Assembly Metadata:** نام، نسخه، فرهنگ (Culture) و اطلاعات شرکت سازنده.
* **Type Metadata:** نام کلاس، کلاس‌های پایه، اینترفیس‌های پیاده‌سازی شده.
* **Method/Property Metadata:** نام، نوع بازگشتی، پارامترها و سطح دسترسی.
* **Attribute Metadata:** لیست Attributeهای اعمال شده به هر یک از عناصر فوق.

### تفاوت Source Code، Metadata، IL و Runtime Object:
```text
1. Source Code: کدی که شما می‌نویسید (C#).
2. Metadata + IL: خروجی کامپایلر (فایل .dll یا .exe). Metadata توصیف‌کننده ساختار است و IL دستورالعمل‌های اجرایی.
3. Runtime Object: وقتی برنامه اجرا می‌شود، CLR از روی Metadata و IL، اشیاء واقعی (Instances) را در حافظه (Heap) می‌سازد.
```
Attributeها در مرحله ۲ (داخل Metadata) ذخیره می‌شوند و در مرحله ۳ توسط Reflection قابل بازیابی هستند.

---

## 4. چرخه حیات: Compiler و Runtime

چرخه مفهومی پردازش Attribute به شرح زیر است:

```mermaid
graph TD
    A[C# Source Code <br/> شامل [Attribute]] --> B(Roslyn Compiler)
    B --> C[Metadata + IL Code]
    C --> D[Assembly .dll/.exe]
    D --> E{Runtime Execution}
    E -->|Reflection / Framework| F[خواندن Metadata]
    F --> G[ساخت Instance از کلاس Attribute]
    G --> H[اعمال رفتار بر اساس Attribute]
```
**نکته کلیدی:** کامپایلر معمولاً Attribute را اجرا نمی‌کند (مگر در موارد خاص مثل `[Conditional]`). کامپایلر فقط آرگومان‌های Attribute را در Metadata **کدگذاری (Encode)** می‌کند. ساخت واقعی شیء Attribute (Instantiation) در Runtime و توسط Reflection یا Framework انجام می‌شود.

---

## 5. تعریف کلاس Attribute

هر کلاسی که بخواهد به عنوان Attribute استفاده شود، باید مستقیماً یا غیرمستقیماً از کلاس پایه `System.Attribute` ارث‌بری کند.

```csharp
// روش صحیح
public class DescriptionAttribute : System.Attribute
{
    public string Text { get; set; }
}

// روش مخفف (معادل بالا)
public class DescriptionAttribute : Attribute
{
}
```
**چرا ارث‌بری از `System.Attribute` الزامی است؟**
این یک قرارداد در CLR (Common Language Runtime) است. کامپایلر و Reflection فقط کلاس‌هایی را که این زنجیره ارث‌بری را دارند به عنوان Attribute معتبر می‌شناسند.

---

## 6. قرارداد نام‌گذاری (Naming Convention)

طبق قرارداد .NET، نام کلاس Attribute باید با پسوند **`Attribute`** تمام شود.
```csharp
public class ObsoleteAttribute : Attribute { }
```
اما هنگام **استفاده** از آن در کد، کامپایلر به شما اجازه می‌دهد پسوند `Attribute` را حذف کنید. هر دو حالت زیر معتبر و معادل هستند:
```csharp
[Obsolete] // کامپایلر خودکار آن را به ObsoleteAttribute تبدیل می‌کند
[ObsoleteAttribute] // نام کامل Type
```
**هشدار Ambiguity:** اگر دو کلاس با نام‌های `MyAttribute` و `My` (بدون پسوند) در یک Namespace وجود داشته باشند، حذف پسوند می‌تواند باعث ابهام شود. همیشه نام کلاس را با `Attribute` تمام کنید.

---

## 7. Attributeهای آماده و کاربردی .NET

### 1. `ObsoleteAttribute`
* **کاربرد:** علامت‌گذاری عناصر منسوخ شده.
* **مثال:** `[Obsolete("Use V2", true)]` (پارامتر `true` باعث خطای کامپایل می‌شود، نه فقط هشدار).

### 2. `ConditionalAttribute`
* **کاربرد:** کامپایل شرطی یک متد. اگر Symbol مشخص شده تعریف نشده باشد، فراخوانی‌های آن متد حذف می‌شوند.
* **مثال:** `[Conditional("DEBUG")]` (فقط در حالت Debug اجرا می‌شود).

### 3. `DebuggerDisplayAttribute`
* **کاربرد:** تغییر نحوه نمایش یک شیء در پنجره Debugger ویژوال استودیو.
* **مثال:** `[DebuggerDisplay("Name = {Name}, Age = {Age}")]`

### 4. `CallerMemberNameAttribute`، `CallerFilePathAttribute`، `CallerLineNumberAttribute`
* **کاربرد:** تزریق خودکار نام متد، مسیر فایل و شماره خط فراخوانی‌کننده در زمان کامپایل (بسیار مفید برای Logging و `INotifyPropertyChanged`).
* **مثال:**
  ```csharp
  public void Log(string message, [CallerMemberName] string memberName = "")
  {
      Console.WriteLine($"{memberName}: {message}");
  }
  ```

### 5. `FlagsAttribute`
* **کاربرد:** نشان می‌دهد که یک `enum` می‌تواند به صورت ترکیب بیتی (Bitwise) استفاده شود.
* **مثال:** `[Flags] public enum Permissions { Read = 1, Write = 2 }`

### 6. `AttributeUsageAttribute`
* **کاربرد:** مشخص می‌کند که یک Custom Attribute روی چه عناصری قابل اعمال است (در بخش 9 توضیح داده می‌شود).

---

## 8. ساخت Custom Attribute

ساخت یک Attribute سفارشی بسیار ساده است. بیایید یک Attribute برای توصیف کلاس‌ها بسازیم:

```csharp
using System;

// 1. ارث‌بری از System.Attribute
// 2. اضافه کردن پسوند Attribute
public class DescriptionAttribute : Attribute
{
    // 3. Constructor برای آرگومان‌های Positional
    public DescriptionAttribute(string description)
    {
        Description = description;
    }

    // 4. Property برای آرگومان‌های Named
    public string Author { get; set; }
    public int Version { get; set; } = 1;

    // 5. داده اصلی
    public string Description { get; }
}
```
**استفاده:**
```csharp
[Description("این کلاس مدیریت مشتریان را بر عهده دارد", Author = "Abolfazl", Version = 2)]
public class Customer
{
}
```

---

## 9. AttributeUsage و AttributeTargets

برای کنترل نحوه استفاده از Custom Attribute خود، باید از `[AttributeUsage]` روی خودِ کلاس Attribute استفاده کنید.

```csharp
[AttributeUsage(
    AttributeTargets.Class | AttributeTargets.Method, // کجاها مجاز است؟
    AllowMultiple = true,                             // آیا می‌توان چند بار روی یک عنصر استفاده کرد؟
    Inherited = true                                  // آیا به کلاس‌های مشتق‌شده ارث می‌رسد؟
)]
public class DescriptionAttribute : Attribute
{
    // ...
}
```

### بررسی `AttributeTargets` (مهم‌ترین مقادیر):
* `Class`, `Struct`, `Interface`, `Enum`, `Delegate`, `Record` (در C# 9+)
* `Method`, `Constructor`, `Property`, `Field`, `Event`
* `Parameter`, `ReturnValue`
* `Assembly`, `Module`
* `GenericParameter` (برای پارامترهای ژنریک)

*نکته:* می‌توانید چندین Target را با عملگر بیتی `|` ترکیب کنید.

---

## 10. آرگومان‌های Attribute (Positional و Named)

Attributeها دو نوع آرگومان می‌پذیرند:

### 1. Positional Arguments (آرگومان‌های موقعیتی)
مستقیماً به **Constructor** کلاس Attribute نگاشت می‌شوند. ترتیب آن‌ها مهم است.
```csharp
[Description("توضیحات اصلی")] // "توضیحات اصلی" به constructor فرستاده می‌شود
```

### 2. Named Arguments (آرگومان‌های نام‌دار)
به **Public Field** یا **Public Property** کلاس Attribute نگاشت می‌شوند. ترتیب مهم نیست و با علامت `=` مشخص می‌شوند.
```csharp
[Description("توضیحات", Author = "Ali", Version = 3)]
```

| ویژگی | Positional Argument | Named Argument |
| :--- | :--- | :--- |
| **مقصد** | Constructor Parameters | Public Properties / Fields |
| **ترتیب** | مهم است | مهم نیست |
| **الزام** | معمولاً Required (بسته به Constructor) | معمولاً Optional |
| **مثال** | `[My("value")]` | `[My(Name = "value")]` |

---

## 11. انواع مجاز در آرگومان‌های Attribute

کامپایلر C# به‌شدت محدود می‌کند که چه نوع داده‌هایی می‌توانند به عنوان آرگومان به Attribute پاس داده شوند. دلیل این محدودیت این است که این مقادیر باید بتوانند در **Metadata** ذخیره شوند.

**انواع مجاز:**
1. انواع اولیه (Primitive Types): `bool`, `byte`, `char`, `double`, `float`, `int`, `long`, `short`, `string`
2. `System.Type` (با استفاده از `typeof`)
3. `enum`
4. آرایه‌های یک‌بعدی از انواع مجاز بالا (مثلاً `int[]` یا `string[]`)
5. مقدار `null` (برای انواع مرجع مجاز)

**انواع غیرمجاز:**
* `object`
* کلاس‌های سفارشی (Custom Classes)
* `dynamic`
* آرایه‌های چندبعدی

**مثال معتبر:**
```csharp
[MyAttribute("Text", typeof(int), MyEnum.Value, new int[] { 1, 2 })]
```
**مثال نامعتبر:**
```csharp
[MyAttribute(new DateTime(2023, 1, 1))] // خطا: DateTime نوع مجازی نیست (مگر اینکه به string یا long تبدیل شود)
```

---

## 12. اعمال Attribute روی عناصر مختلف کد

### روی Class / Struct / Record / Interface / Enum
```csharp
[Description("موجودیت کاربر")]
public record User(string Name);
```

### روی Method
```csharp
[Authorize]
[AuditLog(Action = "Create")]
public void CreateUser() { }
```

### روی Property و Field
```csharp
[Required]
[MaxLength(50)]
public string Email { get; set; }
```

### روی Backing Field (بسیار مهم در C# مدرن)
گاهی می‌خواهید Attribute به جای Property، به Fielد پنهان (Backing Field) یک Auto-Property اعمال شود (مثلاً برای EF Core یا Serialization).
```csharp
[field: JsonPropertyName("user_name")]
public string UserName { get; set; }
```
*سایر Targetهای صریح:* `[property: ...]`, `[param: ...]`, `[return: ...]`

### روی Parameter
```csharp
public IActionResult Get([FromQuery] int id, [FromBody] User user)
```

### روی Return Value
```csharp
[return: NotNullIfNotNull("input")]
public string? Process(string? input) => input;
```

### روی Generic Parameter
```csharp
public class Repository<[MustBeEntity] T> where T : class
{
}
```

### روی Lambda (معرفی شده در C# 10)
در C# 10 به بعد، می‌توانید مستقیماً روی Lambda Expression یا پارامترهای آن Attribute بگذارید:
```csharp
var action = [Obsolete] (int x) => Console.WriteLine(x);

// یا روی پارامتر Lambda
Func<int, int> square = ([Description("Input number")] int x) => x * x;
```

### روی Assembly و Module
این Attributeها معمولاً در فایل `Program.cs` یا `AssemblyInfo.cs` قرار می‌گیرند و روی کل Assembly اعمال می‌شوند.
```csharp
[assembly: AssemblyTitle("My Awesome App")]
[assembly: AssemblyVersion("1.0.0.0")]
```
*نکته:* در SDK-style projects (.NET Core/5+)، بسیاری از این‌ها به‌صورت خودکار از فایل `.csproj` تولید می‌شوند.

---

## 13. Reflection و خواندن Attributeها

**Reflection (بازتاب)** مکانیزمی در .NET است که اجازه می‌دهد Metadata برنامه را در حین اجرا (Runtime) بررسی کنیم.

### APIهای اصلی خواندن Attribute:
فرض کنید کلاس `Customer` داریم که `[Description("Test")]` دارد.

```csharp
using System.Reflection;

// 1. دریافت یک Attribute خاص (تک)
var attr = typeof(Customer).GetCustomAttribute<DescriptionAttribute>();
Console.WriteLine(attr?.Description);

// 2. دریافت تمام Attributeهای یک نوع خاص (لیست)
var attrs = typeof(Customer).GetCustomAttributes<DescriptionAttribute>();

// 3. دریافت تمام Attributeها (بدون فیلتر نوع)
var allAttrs = typeof(Customer).GetCustomAttributes();

// 4. بررسی سریع وجود Attribute (بهینه‌تر از گرفتن لیست)
bool hasDesc = typeof(Customer).IsDefined(typeof(DescriptionAttribute), inherit: true);
```

### خواندن Attribute از متد یا پارامتر:
```csharp
MethodInfo method = typeof(Customer).GetMethod("CreateUser");
var methodAttr = method.GetCustomAttribute<AuditLogAttribute>();

// خواندن از پارامتر
ParameterInfo param = method.GetParameters()[0];
var paramAttr = param.GetCustomAttribute<FromBodyAttribute>();
```

---

## 14. Attribute Constructor و لحظه اجرا

یک سوءتفاهم رایج: **"آیا Constructor Attribute در زمان کامپایل اجرا می‌شود؟"**
**پاسخ:** خیر.

1. **Compile Time:** کامپایلر بررسی می‌کند که آیا نوع Attribute و آرگومان‌های آن معتبر هستند. سپس مقادیر را در Metadata به صورت باینری **Encode** می‌کند. هیچ کدی از Constructor اجرا نمی‌شود.
2. **Runtime:** زمانی که شما (یا یک Framework) متد `GetCustomAttribute<T>()` را فراخوانی می‌کنید، CLR:
   * Metadata را می‌خواند.
   * یک **Instance جدید** از کلاس Attribute را با استفاده از Constructor آن می‌سازد (Instantiation).
   * مقادیر Named Arguments را روی Propertyها تنظیم می‌کند.
   * شیء ساخته شده را برمی‌گرداند.

**نتیجه:** هر بار که `GetCustomAttribute` را صدا بزنید، یک شیء جدید در حافظه (Heap) ساخته می‌شود (مگر اینکه Framework آن را Cache کرده باشد).

---

## 15. Roslyn و AttributeData

وقتی صحبت از **Compile-Time Analysis** (مثل Roslyn Analyzers یا Source Generators) می‌شود، ما به Reflection دسترسی نداریم، چون برنامه هنوز اجرا نشده است. در اینجا Roslyn از مفهوم **`AttributeData`** استفاده می‌کند.

### `AttributeData` چیست؟
یک ساختار داده‌ای در Roslyn است که نمایش‌دهنده یک Attribute در مرحله کامپایل است، **بدون اینکه Instanceای از آن کلاس ساخته شود**.

### تفاوت کلیدی با Reflection:
| ویژگی | Reflection (Runtime) | Roslyn `AttributeData` (Compile Time) |
| :--- | :--- | :--- |
| **ورودی** | Assembly بارگذاری شده | Syntax Tree یا Semantic Model |
| **خروجی** | Instance واقعی کلاس Attribute | شیء `AttributeData` (فقط داده) |
| **اجرای Constructor** | بله، هنگام `GetCustomAttribute` | خیر، هرگز اجرا نمی‌شود |
| **هدف** | تغییر رفتار در Runtime | تحلیل کد، تولید کد، هشدار |
| **دسترسی به مقادیر** | از طریق Propertyهای شیء | از طریق `ConstructorArguments` و `NamedArguments` |

### مثال دسترسی به مقادیر در Roslyn:
```csharp
// در یک Source Generator یا Analyzer
AttributeData? attr = symbol.GetAttributes().FirstOrDefault(a => a.AttributeClass?.Name == "DescriptionAttribute");
if (attr != null)
{
    // خواندن آرگومان Positional اول
    string desc = attr.ConstructorArguments[0].Value as string;
    
    // خواندن آرگومان Named
    TypedConstant authorArg = attr.NamedArguments.FirstOrDefault(kvp => kvp.Key == "Author").Value;
}
```

---

## 16. مقایسه Reflection و Roslyn

| ویژگی | Reflection | Roslyn (Analyzers / Source Generators) |
| :--- | :--- | :--- |
| **زمان اجرا** | Runtime | Compile Time |
| **هزینه Performance** | بالا (سربار Reflection و Allocation) | صفر در Runtime (کار در زمان کامپایل انجام شده) |
| **امنیت** | ممکن است با Trimming/AOT بشکند | کاملاً سازگار با Native AOT |
| **دسترسی به کد** | فقط Metadata و IL | دسترسی کامل به Source Code و Semantic Model |
| **کاربرد اصلی** | Dependency Injection, ORMs, Validation | تولید کد خودکار، اعمال قوانین کدنویسی |

---

## 17. Attribute در اکوسیستم مدرن .NET

### الف) ASP.NET Core
ستون فقرات پیکربندی MVC و Minimal APIs بر اساس Attribute است:
* `[ApiController]`: فعال‌سازی رفتارهای خاص API (مثل اعتبارسنجی خودکار).
* `[Route("api/[controller]")]`: مسیریابی.
* `[HttpGet]`, `[HttpPost]`: متدهای HTTP.
* `[Authorize]`, `[AllowAnonymous]`: امنیت.
* `[FromBody]`, `[FromQuery]`: مدل بایندینگ.

### ب) Serialization (System.Text.Json)
کنترل نحوه تبدیل اشیاء به JSON:
```csharp
public class User
{
    [JsonPropertyName("user_id")]
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public int Id { get; set; }
}
```

### ج) Validation (Data Annotations)
اعتبارسنجی Declarative در ASP.NET Core و EF Core:
```csharp
[Required(ErrorMessage = "ایمیل الزامی است")]
[EmailAddress]
[StringLength(100, MinimumLength = 5)]
public string Email { get; set; }
```
*نکته:* در سیستم‌های پیچیده، **Fluent Validation** به دلیل جداسازی منطق اعتبارسنجی از مدل، اغلب بر Attribute-based Validation ترجیح داده می‌شود.

---

## 18. معماری، الگوها و طراحی

### Attribute به عنوان Declarative Programming
به جای نوشتن کد دستوری (Imperative) برای پیکربندی:
```csharp
// Imperative
builder.Services.AddRouting(options => options.LowercaseUrls = true);
```
از کد اعلامی (Declarative) استفاده می‌کنیم:
```csharp
// Declarative
[Route("api/users", Name = "GetUsers")]
```
**مزیت:** کد خواناتر است و "چه کاری" باید انجام شود را می‌گوید، نه "چگونه" را.

### Domain Attribute vs Infrastructure Attribute
یک چالش معماری مهم: آیا باید Attributeهای فریم‌ورک (مثل `[JsonPropertyName]` یا `[Column]`) را در لایه Domain (موجودیت‌های خالص) قرار داد؟
* **دیدگاه خالص (DDD):** خیر. لایه Domain نباید به فریم‌ورک‌های زیرساختی (مثل EF Core یا System.Text.Json) وابسته باشد.
* **راه‌حل:** استفاده از الگوی جداگانه (Separate DTOs) یا پیکربندی Fluent (مثل `IEntityTypeConfiguration` در EF Core) به جای Attribute در لایه Domain.

---

## 19. Performance، AOT و Trimming

### هزینه‌های Performance
1. **اندازه Metadata:** استفاده بیش از حد از Attributeها حجم فایل DLL را کمی افزایش می‌دهد (معمولاً ناچیز).
2. **هزینه Reflection:** فراخوانی `GetCustomAttributes` کند است و باعث Allocation در Heap می‌شود.
3. **راه‌حل:** Frameworkهای حرفه‌ای (مثل ASP.NET Core) نتیجه Reflection را در یک `ConcurrentDictionary` **Cache** می‌کنند تا فقط بار اول هزینه پرداخت شود.

### Native AOT و IL Trimming
در .NET 8/9، وقتی از **Native AOT** یا **Trimming** استفاده می‌کنید، کامپایلر کدهایی را که به نظر می‌رسد استفاده نمی‌شوند حذف می‌کند.
* **مشکل:** Reflection به‌صورت پویا به دنبال Attributeها می‌گردد. Trimmer ممکن است کلاس Attribute یا Constructor آن را حذف کند، چون فراخوانی مستقیمی از آن در کد ندیده است.
* **راه‌حل:** استفاده از Attributeهای خاصی مثل `[DynamicallyAccessedMembers]` برای اطلاع دادن به Trimmer که این Metadata باید حفظ شود، یا ترجیحاً استفاده از **Source Generators** به جای Reflection برای کشف Attributeها.

---

## 20. اشتباهات رایج

| اشتباه | مشکل | راه‌حل بهتر |
| :--- | :--- | :--- |
| **منطق تجاری (Business Logic) در Constructor Attribute** | Constructor Attribute باید ساده و بدون عوارض جانبی (Side-effect) باشد. اجرای آن در زمان نامشخص (هنگام Reflection) باعث باگ‌های عجیب می‌شود. | Attribute فقط باید داده نگه دارد. منطق را در یک Handler یا Service جداگانه پیاده کنید. |
| **فراخوانی مکرر `GetCustomAttributes` در یک حلقه** | سربار Performance و Allocation شدید. | نتایج را یک‌بار در Startup Cache کنید. |
| **استفاده از Attribute برای پیکربندی پویا** | Attributeها در Compile-Time ثابت هستند. نمی‌توان آن‌ها را در Runtime تغییر داد. | برای پیکربندی پویا از `appsettings.json` و `IOptions<T>` استفاده کنید. |
| **فراموش کردن `[AttributeUsage]`** | ممکن است Attribute به اشتباه روی یک متد اعمال شود در حالی که فقط برای کلاس طراحی شده بود. | همیشه `[AttributeUsage]` را با `AttributeTargets` دقیق مشخص کنید. |
| **تصور اینکه Attribute یک مرز امنیتی است** | وجود `[Authorize]` به‌تنهایی کافی نیست؛ اگر Middleware به درستی پیکربندی نشده باشد، نادیده گرفته می‌شود. | Attributeها فقط Metadata هستند؛ مکانیزم اجرایی Framework باید به درستی پیکربندی شده باشد. |

---

## 21. مقایسه‌های کلیدی

### Attribute vs Interface
| ویژگی | Attribute | Interface |
| :--- | :--- | :--- |
| **هدف** | افزودن Metadata (توصیف) | تعریف Contract و رفتار (تعهد) |
| **اجرا** | غیرفعال (توسط دیگران خوانده می‌شود) | فعال (کلاس باید متدها را پیاده‌سازی کند) |
| **چندگانه** | می‌توان چندین Attribute متفاوت داشت | می‌توان چندین Interface را پیاده‌سازی کرد |
| **مثال** | `[Serializable]` | `IDisposable` |

### Attribute vs Configuration (Options Pattern)
* از **Attribute** استفاده کنید وقتی: پیکربندی ثابت است، به ساختار کد وابسته است و توسط توسعه‌دهنده تعیین می‌شود (مثل `[Route]`).
* از **Configuration (`appsettings.json`)** استفاده کنید وقتی: مقادیر بین محیط‌ها (Dev, Prod) تغییر می‌کنند یا توسط ادمین سیستم مدیریت می‌شوند (مثل Connection String).

---

## 22. پروژه عملی: Mini Framework مبتنی بر Attribute

بیایید یک سیستم ساده **Command Pattern** بسازیم که کلاس‌ها را بر اساس Attribute اسکن و ثبت می‌کند.

### 1. تعریف Attribute
```csharp
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false)]
public class CommandAttribute : Attribute
{
    public string Name { get; }
    public CommandAttribute(string name) => Name = name;
}
```

### 2. تعریف Commandها
```csharp
[Command("create-user")]
public class CreateUserCommand { public void Execute() => Console.WriteLine("User Created"); }

[Command("delete-user")]
public class DeleteUserCommand { public void Execute() => Console.WriteLine("User Deleted"); }
```

### 3. کشف و ثبت (Discovery Pattern) با Reflection
```csharp
using System.Reflection;

public class CommandRegistry
{
    private readonly Dictionary<string, Type> _commands = new();

    public void RegisterFromAssembly(Assembly assembly)
    {
        foreach (var type in assembly.GetTypes())
        {
            var attr = type.GetCustomAttribute<CommandAttribute>();
            if (attr != null)
            {
                _commands[attr.Name] = type;
            }
        }
    }

    public void Execute(string commandName)
    {
        if (_commands.TryGetValue(commandName, out var type))
        {
            var instance = Activator.CreateInstance(type);
            var method = type.GetMethod("Execute");
            method?.Invoke(instance, null);
        }
        else
        {
            Console.WriteLine("Command not found.");
        }
    }
}
```
*نسخه پیشرفته:* در یک پروژه واقعی، به جای `Activator.CreateInstance`، از **Dependency Injection** (`IServiceProvider`) برای ساخت Instance استفاده می‌شود. در مقیاس بزرگ‌تر، این کد کشف (Discovery) بهتر است توسط یک **Source Generator** در زمان کامپایل انجام شود تا هزینه Reflection در Startup حذف گردد.

---

## 23. تست و Benchmark

### تست Unit برای Custom Attribute (با xUnit)
```csharp
[Fact]
public void DescriptionAttribute_ShouldStoreDataCorrectly()
{
    // Arrange & Act
    var attr = new DescriptionAttribute("Test") { Author = "Ali" };

    // Assert
    Assert.Equal("Test", attr.Description);
    Assert.Equal("Ali", attr.Author);
}

[Fact]
public void Class_WithAttribute_ShouldBeDiscoverableViaReflection()
{
    // Act
    var attr = typeof(Customer).GetCustomAttribute<DescriptionAttribute>();

    // Assert
    Assert.NotNull(attr);
    Assert.Equal("Test", attr.Description);
}
```

### ملاحظات Benchmark
اگر از `BenchmarkDotNet` استفاده می‌کنید، همیشه سه حالت را مقایسه کنید:
1. دسترسی مستقیم (Direct Access) - پایه مقایسه.
2. دسترسی با Reflection خام (بدون Cache) - کندترین حالت.
3. دسترسی با Reflection Cache شده (یا Source Generator) - نزدیک به سرعت مستقیم.

---

## 24. Cheat Sheet و راهنمای تصمیم‌گیری

### Cheat Sheet سریع
```csharp
// 1. تعریف
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false, Inherited = true)]
public class MyAttribute : Attribute {
    public MyAttribute(string id) { Id = id; } // Positional
    public string Name { get; set; }           // Named
}

// 2. استفاده
[My("123", Name = "Test")]
public class MyClass { }

// 3. خواندن (Reflection)
var attr = typeof(MyClass).GetCustomAttribute<MyAttribute>();
```

### Decision Guide (راهنمای تصمیم‌گیری)
```text
آیا نیاز دارید رفتار یا Contract خاصی را تحمیل کنید؟
   ├── بله → از Interface استفاده کنید.
   └── خیر → ادامه دهید.

آیا اطلاعات فقط توصیفی (Metadata) هستند؟
   ├── بله → از Attribute استفاده کنید.
   └── خیر → ادامه دهید.

آیا این اطلاعات در زمان اجرا (Runtime) تغییر می‌کنند؟
   ├── بله → از Configuration (appsettings/Database) استفاده کنید.
   └── خیر → ادامه دهید.

آیا نیاز دارید این اطلاعات در زمان کامپایل (Compile Time) تحلیل یا تولید کد شوند؟
   ├── بله → از Roslyn Analyzer / Source Generator استفاده کنید.
   └── خیر → از Reflection در Runtime استفاده کنید (با در نظر گرفتن Cache و AOT).
```

---

## 25. جمع‌بندی

* **Attribute چیست؟** مکانیزمی برای افزودن Metadata به عناصر کد.
* **چرا ساخته شد؟** برای حذف وابستگی به فایل‌های کانفیگ خارجی و چسباندن اطلاعات توصیفی مستقیماً به کد.
* **چگونه کار می‌کند؟** کامپایلر آرگومان‌ها را در Metadata ذخیره می‌کند. در Runtime، Reflection این Metadata را می‌خواند و یک Instance از کلاس Attribute می‌سازد.
* **Roslyn:** در زمان کامپایل، بدون ساخت Instance، از طریق `AttributeData` به این اطلاعات دسترسی دارد (ایده‌آل برای Source Generators).
* **معماری:** Attributeها برای Declarative Programming عالی هستند، اما نباید برای نگهداری Business Logic یا پیکربندی پویا استفاده شوند. در لایه Domain با احتیاط مصرف شوند تا از Coupling به فریم‌ورک جلوگیری شود.
* **مدرن .NET:** در عصر Native AOT، وابستگی شدید به Reflection برای خواندن Attributeها باید با Source Generators یا `[DynamicallyAccessedMembers]` مدیریت شود.

---

## 26. منابع رسمی

برای مطالعه عمیق‌تر، همیشه به مستندات رسمی مایکروسافت مراجعه کنید:

1. **Attributes (C#)** - Microsoft Learn  
   [لینک مستقیم](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)
2. **Writing Custom Attributes** - Microsoft Learn  
   [لینک مستقیم](https://learn.microsoft.com/en-us/dotnet/standard/attributes/writing-custom-attributes)
3. **Reflection in .NET** - Microsoft Learn  
   [لینک مستقیم](https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection)
4. **Roslyn Source Generators** - GitHub dotnet/roslyn  
   [لینک مستقیم](https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.md)
5. **Native AOT and Trimming** - Microsoft Learn  
   [لینک مستقیم](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/)
6. **C# Language Specification (Attributes)** - ECMA / Microsoft  
   [لینک مستقیم](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/attributes)

---
*این سند بر اساس استانداردهای C# 12 و .NET 8/9 تدوین شده است و برای استفاده در Repositoryهای آموزشی و مرجع تیم‌های توسعه نرم‌افزار بهینه‌سازی شده است.*