

# 🚀 راهنمای جامع Extension Methods در سی‌شارپ: از مقدماتی تا پیشرفته

به مستند جامع **Extension Methods** خوش آمدید! این فایل برای درک عمیق این قابلیت جذاب سی‌شارپ طراحی شده است. ما در این آموزش از مفاهیم پایه شروع کرده و تا زیر کاپوت کامپایلر و زمان اجرا (Runtime) پیش می‌رویم.

## 📑 فهرست مطالب
1. [مقدمه: Extension Method چیست؟](#1-مقدمه-extension-method-چیست)
2. [سطح مقدماتی: نوشتن اولین Extension Method](#2-سطح-مقدماتی-نوشتن-اولین-extension-method)
3. [سطح متوسط: جادوی زمان کامپایل (Compile Time)](#3-سطح-متوسط-جادوی-زمان-کامپایل-compile-time)
4. [سطح پیشرفته: دنیای Runtime و Reflection](#4-سطح-پیشرفته-دنیای-runtime-و-reflection)
5. [حل تعارض و اولویت‌ها (Resolution & Priorities)](#5-حل-تعارض-و-اولویت‌ها-resolution--priorities)
   - [الف) اولویت Instance Methodها](#الف-اولویت-instance-methodها)
   - [ب) اولویت بین Extension Methodها](#ب-اولویت-بین-extension-methodها)
   - [ج) Overload Resolution](#ج-overload-resolution)
6. [منابع معتبر و برای مطالعه بیشتر](#6-منابع-معتبر-و-برای-مطالعه-بیشتر)

---

## 1. مقدمه: Extension Method چیست؟
فرض کنید از یک کلاس آماده (مثل `string` یا `DateTime`) استفاده می‌کنید و احساس می‌کنید یک متد خاص در آن وجود ندارد. در گذشته، شما دو راه داشتید:
1. کلاس را ارث‌بری کنید (که برای کلاس‌های `sealed` مثل `string` ممکن نیست).
2. یک کلاس `Utility` بسازید و متدهای `static` در آن بنویسید.

**Extension Method** (متد توسعه‌دهنده) به شما اجازه می‌دهد بدون تغییر کد اصلی کلاس و بدون ارث‌بری، متدهای جدیدی به آن "تزریق" کنید تا دقیقاً مثل متدهای اصلی آن کلاس صدا زده شوند.
> 💡 **مثال واقعی:** تمام متدهای معروف **LINQ** (مثل `Where`, `Select`, `FirstOrDefault`) در واقع Extension Methodهایی هستند که به کلاس `IEnumerable<T>` اضافه شده‌اند!

---

## 2. سطح مقدماتی: نوشتن اولین Extension Method

برای نوشتن یک Extension Method، باید **۳ قانون طلایی** را رعایت کنید:
1. متد باید در یک **کلاس استاتیک** (`static class`) تعریف شود.
2. خود متد باید **استاتیک** (`static`) باشد.
3. اولین پارامتر متد باید با کلمه کلیدی **`this`** شروع شود (نوع داده‌ای که می‌خواهید متد به آن اضافه شود).

### 🛠️ مثال عملی:
می‌خواهیم متدی به `string` اضافه کنیم که تعداد کلمات یک متن را بشمارد.

```csharp
// 1. کلاس باید static باشد
public static class StringExtensions
{
    // 2. متد باید static باشد
    // 3. پارامتر اول با this شروع می‌شود
    public static int WordCount(this string str)
    {
        if (string.IsNullOrWhiteSpace(str)) return 0;
        return str.Split(new char[] { ' ', '.', ',' }, StringSplitOptions.RemoveEmptyEntries).Length;
    }
}
```

**نحوه استفاده:**
```csharp
string myText = "Hello world, this is C#.";
// حالا myText متد WordCount را دارد!
int count = myText.WordCount(); 
Console.WriteLine(count); // خروجی: 5
```

---

## 3. سطح متوسط: جادوی زمان کامپایل (Compile Time)

بسیاری از مبتدیان فکر می‌کنند Extension Methodها ساختار جدیدی در سی‌شارپ هستند. اما واقعیت این است که آن‌ها فقط یک **Syntactic Sugar** (شیرین‌کننده سینتکس) هستند.

### تبدیل Extension Method به Static Method
کامپایلر سی‌شارپ در زمان کامپایل (Compile Time)، کد شما را فریب می‌دهد! وقتی شما متد را به صورت شی‌گرا صدا می‌زنید، کامپایلر آن را به یک **فراخوانی متد استاتیک معمولی** تبدیل می‌کند.

**کدی که شما می‌نویسید:**
```csharp
string text = "Hello";
text.WordCount();
```

**کدی که کامپایلر در خروجی (IL) تولید می‌کند:**
```csharp
string text = "Hello";
StringExtensions.WordCount(text); // کامپایلر دقیقاً همین کار را می‌کند!
```

> 🔍 **نتیجه‌گیری مهم:** Extension Methodها هیچ رفتار جادویی در زمان اجرا ندارند. آن‌ها فقط یک میانبر نوشتاری (Syntax Shortcut) هستند که کامپایلر آن را به فراخوانی یک متد استاتیک معمولی ترجمه می‌کند.

---

## 4. سطح پیشرفته: دنیای Runtime و Reflection

حالا که می‌دانیم کامپایلر این متدها را به متدهای استاتیک معمولی تبدیل می‌کند، یک سوال پیشرفته مطرح می‌شود:
**آیا Extension Methodها در زمان اجرا (Runtime) وجود دارند؟**

**پاسخ: خیر!** 
در فایل DLL کامپایل شده، هیچ ردی از اینکه `WordCount` به کلاس `string` تعلق دارد وجود ندارد. این متد صرفاً یک متد استاتیک در کلاس `StringExtensions` است.

### بررسی با Reflection
اگر سعی کنید با استفاده از `Reflection` این متد را روی کلاس `string` پیدا کنید، با خطا یا مقدار `null` مواجه می‌شوید:

```csharp
// این کد null برمی‌گرداند!
var method = typeof(string).GetMethod("WordCount"); 
```

**چرا؟** چون Reflection فقط ساختار واقعی تایپ‌ها در زمان اجرا را می‌بیند و از "قصد و نیت" کامپایلر بی‌خبر است.
*(نکته: کامپایلر یک Attribute به نام `ExtensionAttribute` به متدهای استاتیکِ دارای `this` اضافه می‌کند تا ابزارهای دیگر مثل IDEها متوجه وجود آن‌ها شوند، اما این Attribute در زمان اجرا به معنای الحاق متد به تایپ مقصد نیست).*

---

## 5. حل تعارض و اولویت‌ها (Resolution & Priorities)

وقتی شما یک Extension Method می‌نویسید، کامپایلر باید تصمیم بگیرد کدام متد را صدا بزند. این فرآیند **Method Resolution** نام دارد. قوانین اولویت به شدت مهم هستند.

### الف) اولویت Instance Methodها
**قانون طلایی:** متدهای اصلی (Instance Methods) که خودِ کلاس دارد، **همیشه** بر Extension Methodها ارجحیت دارند.

```csharp
public class MyClass
{
    public void DoSomething() => Console.WriteLine("Instance Method");
}

public static class MyExtensions
{
    public static void DoSomething(this MyClass obj) => Console.WriteLine("Extension Method");
}

// استفاده:
MyClass obj = new MyClass();
obj.DoSomething(); 
// خروجی: Instance Method
```
> ⚠️ **هشدار:** اگر در آینده مایکروسافت یا نویسنده کلاس، متدی هم‌نام با Extension Method شما به کلاس اصلی اضافه کند، کد شما بدون هیچ خطای کامپایلی، رفتار متفاوتی (صدا زدن متد اصلی) پیدا خواهد کرد!

### ب) اولویت بین Extension Methodها
اگر دو Extension Method با امضای یکسان (نام و پارامترهای مشابه) در دو کلاس مختلف داشته باشیم، کامپایلر بر اساس **Namespace** تصمیم می‌گیرد.

**قانون جستجوی کامپایلر:**
1. ابتدا در **Namespace فعلی** (جایی که کد در حال نوشتن است) جستجو می‌کند.
2. سپس به **Namespaceهای والد** (Outer namespaces) می‌رود.
3. در نهایت **Namespaceهای import شده** (با دستور `using`) را بررسی می‌کند.

```csharp
namespace App.Utilities
{
    public static class ExtA { public static void Print(this string s) => Console.WriteLine("A"); }
}

namespace App.Core
{
    public static class ExtB { public static void Print(this string s) => Console.WriteLine("B"); }
}

namespace App.Core // Namespace فعلی
{
    using App.Utilities; // Import کردن

    class Program
    {
        static void Main()
        {
            "Hello".Print(); 
            // خروجی: B
            // چون ExtB در Namespace فعلی (App.Core) است و به ExtA (که در using است) اولویت دارد.
        }
    }
}
```
*نکته: اگر دو متد در یک سطح از اولویت Namespace باشند، کامپایلر خطای **Ambiguous Invocation** می‌دهد و شما باید متد را به صورت استاتیک (مثل `ExtA.Print("Hello")`) صدا بزنید.*

### ج) Overload Resolution
اگر Extension Methodهای شما دارای Overload (بارگذاری) باشند، کامپایلر دقیقاً مثل متدهای استاتیک معمولی، بر اساس **تطابق دقیق پارامترها** و **تبدیل‌های ضمنی (Implicit Conversions)** بهترین متد را انتخاب می‌کند.

```csharp
public static class MathExtensions
{
    public static double Add(this int a, int b) => a + b;
    public static double Add(this double a, double b) => a + b;
}

int x = 5;
x.Add(10); // متد اول (int, int) صدا زده می‌شود.

double y = 5.5;
y.Add(10); // متد دوم (double, double) صدا زده می‌شود (چون 10 به double تبدیل می‌شود).
```

---

## 6. منابع معتبر و برای مطالعه بیشتر

برای اطمینان از صحت مطالب و مطالعه عمیق‌تر، این مستند بر اساس منابع زیر تدوین شده است:

1. **[Microsoft Learn - Extension Methods (Official C# Docs)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)**
   * مرجع اصلی و رسمی مایکروسافت برای سینتکس و قوانین پایه.
2. **[C# Language Specification - Extension Methods (ECMA-334)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128103-extension-methods)**
   * مشخصات رسمی زبان سی‌شارپ (بخش 12.8.10.3) که الگوریتم دقیق Method Resolution و Overload Resolution را توضیح می‌دهد.
3. **[CLR via C# by Jeffrey Richter](https://www.microsoftpressstore.com/store/clr-via-c-9780735667457)**
   * فصل مربوط به متدها و درک عمیق از تفاوت Compile Time و Runtime و نحوه کار Reflection.
4. **[C# in Depth by Jon Skeet](https://csharpindepth.com/)**
   * بررسی عالی از نحوه کار کامپایلر سی‌شارپ و تبدیل Extension Methods به Static Calls.
5. **[SharpLab.io](https://sharplab.io/)**
   * ابزار آنلاین و معتبر برای مشاهده کد IL و اثبات اینکه Extension Methodها در زمان اجرا وجود ندارند (کدهای این مقاله را می‌توانید در این سایت تست کنید).

---
<div align="center">
  <i>اگر این آموزش برای شما مفید بود، لطفاً به ریپازیتوری Star بدهید و آن را با دیگر برنامه‌نویسان به اشتراک بگذارید! ⭐</i>
</div>