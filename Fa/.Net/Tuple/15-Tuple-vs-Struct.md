# مقدمه و مفهوم Tuple در C#

> یک راهنمای کامل، مرحله‌به‌مرحله و استاندارد برای درک عمیق مفهوم Tuple در زبان C#، از سطح مقدماتی تا پیشرفته.

---

## 📑 فهرست مطالب

- [مقدمه](#مقدمه)
- [Tuple چیست؟](#tuple-چیست)
- [چرا Tuple به وجود آمد؟](#چرا-tuple-به-وجود-آمد)
- [Tuple چه مشکلی را حل می‌کند؟](#tuple-چه-مشکلی-را-حل-میکند)
- [هدف اصلی Tuple](#هدف-اصلی-tuple)
- [تاریخچه: دو نسل از Tuple در C#](#تاریخچه-دو-نسل-از-tuple-در-c)
- [چگونه چند مقدار را در یک ساختار قرار می‌دهد؟](#چگونه-چند-مقدار-را-در-یک-ساختار-قرار-میدهد)
- [مثال‌های بسیار ساده](#مثالهای-بسیار-ساده)
- [تفاوت Tuple با یک متغیر معمولی](#تفاوت-tuple-با-یک-متغیر-معمولی)
- [تفاوت Tuple با Array](#تفاوت-tuple-با-array)
- [تفاوت Tuple با Class](#تفاوت-tuple-با-class)
- [تفاوت Tuple با Anonymous Type](#تفاوت-tuple-با-anonymous-type)
- [مثال‌های واقعی‌تر](#مثالهای-واقعیتر)
- [کاربردهای رایج Tuple در C#](#کاربردهای-رایج-tuple-در-c)
- [نکات مهم](#نکات-مهم)
- [اشتباهات رایج](#اشتباهات-رایج)
- [جمع‌بندی](#جمعبندی)
- [منابع و مطالعه بیشتر](#منابع-و-مطالعه-بیشتر)

---

## مقدمه

در برنامه‌نویسی، گاهی اوقات نیاز داریم چند مقدار مرتبط را در کنار یکدیگر نگهداری کنیم، بدون اینکه بخواهیم برای آن‌ها یک کلاس یا ساختار جداگانه تعریف کنیم. زبان C# برای پاسخ به این نیاز، ساختاری به نام **Tuple** را ارائه می‌دهد.

این مقاله شما را قدم‌به‌قدم با مفهوم Tuple آشنا می‌کند، از تعریف پایه شروع می‌کند و تا کاربردهای پیشرفته و تفاوت آن با سایر ساختارها پیش می‌رود.

---

## Tuple چیست؟

به زبان ساده، **Tuple** یک ساختار داده‌ای است که اجازه می‌دهد چند مقدار با **نوع‌های مختلف** را در یک واحد (یک شیء) کنار هم قرار دهیم.

به عبارت دیگر، Tuple مانند یک «بسته» یا «جعبه» است که چند آیتم را در خود نگه می‌دارد. هر آیتم می‌تواند نوع متفاوتی داشته باشد؛ مثلاً یک `string`، یک `int` و یک `bool` همه در یک Tuple.

**تعریف فنی:**
Tuple یک ساختار داده‌ای **تعداد ثابت** (fixed-size) و **ترتیب‌دار** (ordered) از عناصر است که هر عنصر می‌تواند نوع متفاوتی داشته باشد.

> 💡 نکته: کلمه Tuple از ریاضیات می‌آید و به معنای دنباله‌ای مرتب از عناصر است. مثلاً به Tuple دو عضوی «دوتایی» (pair)، سه عضوی «سه‌تایی» (triple) و چهار عضوی «چهارتایی» (quadruple) می‌گویند.

---

## چرا Tuple به وجود آمد؟

قبل از ورود Tuple به زبان C#، برنامه‌نویسان برای بازگرداندن چند مقدار از یک متد، مجبور بودند یکی از راه‌حل‌های زیر را انتخاب کنند:

1. **استفاده از پارامترهای `out` یا `ref`** — که کد را ناخوانا و پیچیده می‌کرد.
2. **تعریف یک کلاس یا ساختار مخصوص** — که برای استفاده‌های موقت و کوچک، سربار زیادی داشت.
3. **استفاده از `Dictionary` یا `List<object>`** — که type-safe نبود و نیاز به casting داشت.
4. **استفاده از Anonymous Type** — که فقط در محدوده همان متد قابل استفاده بود و نمی‌شد آن را از متد خارج کرد.

Tuple به دنیا آمد تا یک راه‌حل **سبک، استاندارد و type-safe** برای این موقعیت‌ها ارائه دهد.

---

## Tuple چه مشکلی را حل می‌کند؟

Tuple سه مشکل اصلی را حل می‌کند:

### ۱. بازگرداندن چند مقدار از یک متد
به‌جای استفاده از پارامترهای `out` یا تعریف کلاس، می‌توان چند مقدار را مستقیماً به‌صورت یک Tuple برگرداند.

### ۲. گروه‌بندی موقت داده‌های مرتبط
گاهی می‌خواهیم چند داده مرتبط را موقتاً در کنار هم نگه داریم، بدون اینکه ساختار دائمی برای آن‌ها تعریف کنیم.

### ۳. جایگزینی سبک برای کلاس‌های کوچک
وقتی نیاز به یک کلاس با چند فیلد ساده داریم و نمی‌خواهیم کد را شلوغ کنیم، Tuple گزینه مناسبی است.

---

## هدف اصلی Tuple

هدف اصلی Tuple ارائه یک ساختار **سبک، سریع و موقت** برای نگهداری چند مقدار مرتبط است، بدون اینکه نیاز به تعریف یک نوع (type) جدید باشد.

به بیان دیگر: **Tuple برای زمانی است که داده مهم‌تر از نام نوع است.**

---

## تاریخچه: دو نسل از Tuple در C#

برای درک درست Tuple، باید بدانیم که در C# **دو نوع Tuple** وجود دارد:

| ویژگی | `System.Tuple` (نسل قدیم) | `ValueTuple` (نسل جدید) |
|---|---|---|
| **معرفی شده در** | C# 4.0 (.NET Framework 4.0) | C# 7.0 (.NET Framework 4.7 / .NET Core) |
| **نوع** | Reference Type (کلاس) | Value Type (struct) |
| **تغییرپذیری** | Immutable (غیرقابل تغییر) | Mutable (قابل تغییر) |
| **نام‌گذاری فیلدها** | ندارد (Item1, Item2, ...) | دارد (نام دلخواه) |
| **عملکرد** | کندتر (به دلیل heap allocation) | سریع‌تر (روی stack) |
| **Deconstruction** | ندارد | دارد |
| **تعداد عناصر** | حداکثر 8 (با تو‌رتو) | حداکثر 7 (با تو‌رتو بیشتر) |

> ⚠️ **نکته مهم:** در C# مدرن (نسخه 7.0 به بعد)، تقریباً همیشه از `ValueTuple` استفاده می‌کنیم، حتی اگر کلمه `Tuple` را تایپ کنیم. سینتکس جدید `(int, string)` در واقع `ValueTuple<int, string>` است.

در این مقاله تمرکز اصلی ما روی **ValueTuple (C# 7.0 به بعد)** است، چون استاندارد امروزی C# محسوب می‌شود.

---

## چگونه چند مقدار را در یک ساختار قرار می‌دهد؟

Tuple از یک **ساختار (struct)** به نام `ValueTuple<T1, T2, ...>` استفاده می‌کند. وقتی شما یک Tuple می‌سازید، کامپایلر C# یک نمونه از این struct ایجاد می‌کند و مقادیر را در فیلدهای آن قرار می‌دهد.

### ساختار داخلی (به‌صورت ساده‌شده):

```csharp
public struct ValueTuple<T1, T2>
{
    public T1 Item1;
    public T2 Item2;
    
    public ValueTuple(T1 item1, T2 item2)
    {
        Item1 = item1;
        Item2 = item2;
    }
}
```

در عمل، شما نیازی به نوشتن این ساختار ندارید، چون کامپایلر C# با سینتکس ساده `(value1, value2)` این کار را برای شما انجام می‌دهد.

---

## مثال‌های بسیار ساده

### مثال ۱: ساخت یک Tuple دو عضوی

```csharp
// ساخت یک Tuple با سینتکس ساده (C# 7.0 به بعد)
var person = ("Ali", 25);

// دسترسی به مقادیر با نام‌های پیش‌فرض
Console.WriteLine(person.Item1); // خروجی: Ali
Console.WriteLine(person.Item2); // خروجی: 25
```

**توضیح خط‌به‌خط:**
- خط ۲: یک Tuple با دو عضو ساخته می‌شود: یک `string` و یک `int`.
- خط ۵ و ۶: چون نامی برای اعضا تعیین نکرده‌ایم، به‌صورت پیش‌فرض `Item1` و `Item2` هستند.

### مثال ۲: Tuple با نام‌گذاری اعضا

```csharp
// نام‌گذاری اعضا هنگام ساخت
var person = (Name: "Ali", Age: 25);

// دسترسی با نام‌های دلخواه
Console.WriteLine(person.Name); // خروجی: Ali
Console.WriteLine(person.Age);  // خروجی: 25
```

**توضیح خط‌به‌خط:**
- خط ۲: هنگام ساخت Tuple، برای هر عضو یک نام تعیین می‌کنیم.
- خط ۵ و ۶: حالا می‌توانیم با این نام‌ها به مقادیر دسترسی داشته باشیم که کد را خواناتر می‌کند.

### مثال ۳: تعیین نوع صریح Tuple

```csharp
// تعیین نوع به‌صورت صریح
(string Name, int Age) person = ("Sara", 30);

Console.WriteLine(person.Name); // خروجی: Sara
Console.WriteLine(person.Age);  // خروجی: 30
```

### مثال ۴: Tuple با بیش از دو عضو

```csharp
// Tuple چهار عضوی
var point4D = (X: 10, Y: 20, Z: 30, W: 40);

Console.WriteLine($"{point4D.X}, {point4D.Y}, {point4D.Z}, {point4D.W}");
// خروجی: 10, 20, 30, 40
```

### مثال ۵: تغییر مقادیر Tuple (چون ValueTuple یک struct است)

```csharp
var person = (Name: "Ali", Age: 25);

// تغییر مقدار
person.Age = 26;

Console.WriteLine(person.Age); // خروجی: 26
```

> 💡 توجه: این ویژگی فقط در `ValueTuple` (C# 7.0 به بعد) وجود دارد. در `System.Tuple` قدیمی، اعضا غیرقابل تغییر بودند.

---

## تفاوت Tuple با یک متغیر معمولی

| ویژگی | متغیر معمولی | Tuple |
|---|---|---|
| **تعداد مقادیر** | فقط یک مقدار | چندین مقدار |
| **نوع داده** | یک نوع مشخص | چند نوع مختلف |
| **کاربرد** | نگهداری یک داده | گروه‌بندی داده‌های مرتبط |
| **مثال** | `int x = 5;` | `var t = (5, "hello");` |

### مثال مقایسه:

```csharp
// متغیرهای معمولی - جدا از هم
string name = "Ali";
int age = 25;
bool isActive = true;

// Tuple - همه در یک ساختار
var person = (Name: "Ali", Age: 25, IsActive: true);
```

در مثال Tuple، سه داده مرتبط در یک واحد نگهداری می‌شوند که انتقال آن‌ها را به‌عنوان یک بسته آسان می‌کند.

---

## تفاوت Tuple با Array

| ویژگی | Array | Tuple |
|---|---|---|
| **تعداد اعضا** | متغیر (پویا در اندازه) | ثابت (از پیش تعیین‌شده) |
| **نوع اعضا** | همه یک نوع | هر عضو می‌تواند نوع متفاوتی داشته باشد |
| **دسترسی** | با ایندکس عددی | با نام یا Item1, Item2, ... |
| **Type Safety** | نوع یکسان برای همه | نوع مشخص برای هر عضو |
| **کاربرد** | مجموعه‌ای از داده‌های هم‌نوع | گروهی از داده‌های ناهم‌گون |

### مثال مقایسه:

```csharp
// Array - همه اعضا باید از یک نوع باشند
int[] numbers = { 1, 2, 3, 4, 5 };

// Tuple - هر عضو می‌تواند نوع متفاوتی داشته باشد
var mixed = (1, "hello", true, 3.14);
```

> 💡 **قانون کلی:** اگر همه داده‌های شما از یک نوع هستند → از **Array** استفاده کنید. اگر داده‌ها از نوع‌های مختلف و مرتبط هستند → از **Tuple** استفاده کنید.

---

## تفاوت Tuple با Class

| ویژگی | Class | Tuple |
|---|---|---|
| **تعریف** | نیاز به تعریف جداگانه دارد | بدون تعریف جداگانه |
| **نوع** | Reference Type | Value Type (در ValueTuple) |
| **سربار** | نیاز به نوشتن کلاس و فیلدها | سینتکس کوتاه |
| **نام نوع** | دارای نام معنادار | معمولاً بدون نام معنادار |
| **متد و رفتار** | می‌تواند متد داشته باشد | فقط داده، بدون متد |
| **کاربرد** | مدل‌های پیچیده و ماندگار | داده‌های موقت و ساده |

### مثال مقایسه:

```csharp
// روش کلاسیک: تعریف یک کلاس
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

var p1 = new Person { Name = "Ali", Age = 25 };

// روش Tuple: بدون تعریف کلاس
var p2 = (Name: "Ali", Age: 25);
```

> 💡 **قانون کلی:** اگر داده‌ها **ماندگار، پیچیده یا دارای رفتار** هستند → از **Class** استفاده کنید. اگر داده‌ها **موقت، ساده و فقط برای انتقال** هستند → از **Tuple** استفاده کنید.

---

## تفاوت Tuple با Anonymous Type

این یکی از مهم‌ترین مقایسه‌هاست، چون هر دو سینتکس مشابهی دارند.

| ویژگی | Anonymous Type | Tuple |
|---|---|---|
| **سینتکس** | `new { Name = "Ali", Age = 25 }` | `(Name: "Ali", Age: 25)` |
| **نوع** | Reference Type (کلاس) | Value Type (struct) |
| **تغییرپذیری** | Immutable (غیرقابل تغییر) | Mutable (قابل تغییر در ValueTuple) |
| **خروج از متد** | ❌ نمی‌تواند از متد خارج شود | ✅ می‌تواند از متد خارج شود |
| **بازگشت از متد** | ❌ فقط با `object` یا `dynamic` | ✅ به‌صورت type-safe |
| **Deconstruction** | ❌ ندارد | ✅ دارد |
| **عملکرد** | کندتر (heap allocation) | سریع‌تر (stack allocation) |

### مثال مقایسه:

```csharp
// Anonymous Type - فقط در محدوده محلی
var anon = new { Name = "Ali", Age = 25 };
// نمی‌توانیم این را به‌عنوان نوع بازگشتی متد استفاده کنیم

// Tuple - می‌تواند از متد خارج شود
(string Name, int Age) GetPerson()
{
    return ("Ali", 25);
}

var tuple = GetPerson(); // کاملاً type-safe
```

> 💡 **قانون کلی:** اگر می‌خواهید داده را **فقط در یک متد** استفاده کنید → **Anonymous Type**. اگر می‌خواهید داده را **بین متدها انتقال دهید** → **Tuple**.

---

## مثال‌های واقعی‌تر

### مثال ۱: بازگرداندن چند مقدار از یک متد

```csharp
using System;

class Program
{
    // متدی که چند مقدار برمی‌گرداند
    static (int Sum, int Product) Calculate(int a, int b)
    {
        int sum = a + b;
        int product = a * b;
        return (sum, product);
    }

    static void Main()
    {
        var result = Calculate(5, 3);
        
        Console.WriteLine($"Sum: {result.Sum}");       // خروجی: Sum: 8
        Console.WriteLine($"Product: {result.Product}"); // خروجی: Product: 15
    }
}
```

**توضیح:**
- متد `Calculate` دو مقدار را به‌صورت یک Tuple برمی‌گرداند.
- نوع بازگشت `(int Sum, int Product)` است که کاملاً type-safe و خوانا است.
- در `Main` با نام‌های `Sum` و `Product` به مقادیر دسترسی داریم.

### مثال ۲: استفاده در LINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        string[] names = { "Ali", "Sara", "Reza", "Mina" };

        // ایجاد Tuple در Select
        var nameLengths = names.Select(name => (Name: name, Length: name.Length));

        foreach (var item in nameLengths)
        {
            Console.WriteLine($"{item.Name}: {item.Length} characters");
        }
        // خروجی:
        // Ali: 3 characters
        // Sara: 4 characters
        // Reza: 4 characters
        // Mina: 4 characters
    }
}
```

### مثال ۳: Deconstruction (تجزیه Tuple)

یکی از قابلیت‌های قدرتمند C# 7.0، امکان **تجزیه** Tuple به متغیرهای جداگانه است:

```csharp
using System;

class Program
{
    static void Main()
    {
        var person = (Name: "Ali", Age: 25, City: "Tehran");

        // Deconstruction - تجزیه به متغیرهای جداگانه
        var (name, age, city) = person;

        Console.WriteLine(name); // خروجی: Ali
        Console.WriteLine(age);  // خروجی: 25
        Console.WriteLine(city); // خروجی: Tehran

        // استفاده از var برای برخی اعضا و discard (_) برای بقیه
        var (n, _, c) = person;
        Console.WriteLine($"{n} from {c}"); // خروجی: Ali from Tehran
    }
}
```

**توضیح:**
- خط ۱۰: Tuple به سه متغیر جداگانه `name`, `age`, `city` تجزیه می‌شود.
- خط ۱۶: با استفاده از `_` (discard) می‌توانیم از برخی اعضا صرف‌نظر کنیم.

### مثال ۴: استفاده در Dictionary برای کلید مرکب

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        // استفاده از Tuple به‌عنوان کلید Dictionary
        var grid = new Dictionary<(int X, int Y), string>
        {
            { (0, 0), "Start" },
            { (1, 0), "Path" },
            { (2, 0), "Goal" }
        };

        Console.WriteLine(grid[(0, 0)]); // خروجی: Start
        Console.WriteLine(grid[(2, 0)]); // خروجی: Goal
    }
}
```

### مثال ۵: مقایسه و تساوی Tupleها

```csharp
using System;

class Program
{
    static void Main()
    {
        var t1 = (1, "hello");
        var t2 = (1, "hello");
        var t3 = (1, "world");

        Console.WriteLine(t1 == t2); // خروجی: True
        Console.WriteLine(t1 == t3); // خروجی: False
        Console.WriteLine(t1.Equals(t2)); // خروجی: True
    }
}
```

> 💡 `ValueTuple` بر اساس **مقدار** مقایسه می‌شود (value-based equality)، نه بر اساس مرجع.

---

## کاربردهای رایج Tuple در C#

### ۱. بازگرداندن چند مقدار از متد
رایج‌ترین کاربرد Tuple.

```csharp
(bool Success, string Message, int ErrorCode) TryProcess(string input)
{
    if (string.IsNullOrEmpty(input))
        return (false, "Input is empty", 1001);
    
    return (true, "Processed successfully", 0);
}
```

### ۲. جایگزین `out` parameters

```csharp
// روش قدیمی با out
bool TryParse(string s, out int result) { ... }

// روش مدرن با Tuple
(int Value, bool Success) ParseNumber(string s)
{
    if (int.TryParse(s, out int value))
        return (value, true);
    return (0, false);
}
```

### ۳. گروه‌بندی داده‌های موقت در LINQ

```csharp
var query = students.Select(s => (s.Name, Average: s.Grades.Average()));
```

### ۴. کلید مرکب در Dictionary

```csharp
var cache = new Dictionary<(string UserId, string ResourceId), DateTime>();
```

### ۵. Swapping دو متغیر

```csharp
int a = 5, b = 10;
(a, b) = (b, a); // جابجایی ساده بدون متغیر موقت
```

### ۶. بازگرداندن موقعیت یا مختصات

```csharp
(int X, int Y) GetCursorPosition() => (100, 200);
```

### ۷. استفاده در Pattern Matching (C# 8.0 به بعد)

```csharp
string Describe((int X, int Y) point) => point switch
{
    (0, 0) => "Origin",
    (_, 0) => "On X-axis",
    (0, _) => "On Y-axis",
    _ => "Somewhere else"
};
```

---

## نکات مهم

✅ **نکته ۱:** در C# 7.0 به بعد، سینتکس `(T1, T2, ...)` در واقع `ValueTuple<T1, T2, ...>` است، نه `System.Tuple`.

✅ **نکته ۲:** `ValueTuple` یک **struct** است و روی **stack** قرار می‌گیرد، بنابراین عملکرد بهتری نسبت به `System.Tuple` دارد.

✅ **نکته ۳:** نام‌های اعضا در Tuple **فقط در زمان کامپایل** وجود دارند و در زمان اجرا از بین می‌روند. این یعنی در Reflection دیده نمی‌شوند.

✅ **نکته ۴:** حداکثر تعداد اعضا در یک Tuple بدون nesting، **۷ عضو** است. برای بیشتر از آن، باید از Tuple تودرتو استفاده کنید.

✅ **نکته ۵:** Tupleها قابل مقایسه با `==` هستند و بر اساس مقدار مقایسه می‌شوند.

✅ **نکته ۶:** می‌توانید از **Deconstruction** برای تجزیه Tuple به متغیرهای جداگانه استفاده کنید.

✅ **نکته ۷:** Tupleها **nullable** هستند. می‌توانید `(int, string)?` داشته باشید.

✅ **نکته ۸:** در **نام‌گذاری اعضا**، حروف بزرگ و کوچک مهم نیستند (case-insensitive) اما باید با قوانین نام‌گذاری C# مطابقت داشته باشند.

✅ **نکته ۹:** `ValueTuple` در **NuGet Package** به نام `System.ValueTuple` برای نسخه‌های قدیمی‌تر .NET Framework در دسترس است. در .NET Core 2.0+ و .NET 5+ به‌صورت پیش‌فرض موجود است.

✅ **نکته ۱۰:** اگر نیاز به **نام معنادار برای نوع** دارید (مثلاً برای API عمومی)، به‌جای Tuple از یک **record** یا **class** استفاده کنید.

---

## اشتباهات رایج

❌ **اشتباه ۱: استفاده از Tuple برای مدل‌های پیچیده و ماندگار**

```csharp
// ❌ اشتباه
public List<(string, int, string, bool, DateTime)> GetUsers() { ... }

// ✅ درست: استفاده از class یا record
public record User(string Name, int Age, string Email, bool IsActive, DateTime CreatedAt);
public List<User> GetUsers() { ... }
```

❌ **اشتباه ۲: فراموش کردن نام‌گذاری اعضا**

```csharp
// ❌ نامفهوم
var result = GetData();
Console.WriteLine(result.Item1); // Item1 چیست؟

// ✅ با نام‌گذاری
var result = GetData();
Console.WriteLine(result.UserName); // واضح و خوانا
```

❌ **اشتباه ۳: استفاده از Tuple به‌جای Anonymous Type در LINQ محلی**

```csharp
// ❌ غیرضروری
var query = names.Select(n => (Name: n, Length: n.Length));

// ✅ Anonymous Type برای استفاده محلی
var query = names.Select(n => new { n, Length = n.Length });
```

❌ **اشتباه ۴: تلاش برای استفاده از Tuple به‌عنوان نوع بازگشتی Anonymous Type**

```csharp
// ❌ این کار نمی‌کند
public ??? GetAnonymous() 
{
    return new { Name = "Ali", Age = 25 };
}

// ✅ از Tuple استفاده کنید
public (string Name, int Age) GetPerson() 
{
    return ("Ali", 25);
}
```

❌ **اشتباه ۵: انتظار داشتن نام اعضا در زمان اجرا**

```csharp
var person = (Name: "Ali", Age: 25);

// ❌ این کار نمی‌کند - نام‌ها در زمان اجرا وجود ندارند
var type = person.GetType();
var properties = type.GetProperties(); // خالی است!
```

❌ **اشتباه ۶: استفاده بیش از حد از Tuple در API عمومی**

Tuple برای **استفاده داخلی** عالی است، اما برای **API عمومی** بهتر است از انواع مشخص (class, record, struct) استفاده کنید تا معنای داده‌ها واضح باشد.

❌ **اشتباه ۷: نادیده گرفتن محدودیت ۷ عضوی**

```csharp
// ❌ خطای کامپایل - بیش از 7 عضو
var bigTuple = (1, 2, 3, 4, 5, 6, 7, 8);

// ✅ استفاده از Tuple تودرتو
var bigTuple = (1, 2, 3, 4, 5, 6, 7, Rest: (8, 9));
```

❌ **اشتباه ۸: عدم توجه به تفاوت `System.Tuple` و `ValueTuple`**

```csharp
// ❌ استفاده از System.Tuple قدیمی در کد جدید
System.Tuple<int, string> oldTuple = new System.Tuple<int, string>(1, "hello");

// ✅ استفاده از ValueTuple (سینتکس مدرن)
var newTuple = (1, "hello");
```

---

## جمع‌بندی

در این مقاله با مفهوم **Tuple** در C# به‌صورت کامل آشنا شدیم:

🔹 **Tuple** یک ساختار داده‌ای برای نگهداری چند مقدار با نوع‌های مختلف در یک واحد است.

🔹 در C# **دو نسل Tuple** وجود دارد: `System.Tuple` (قدیمی، C# 4.0) و `ValueTuple` (جدید، C# 7.0 به بعد).

🔹 **ValueTuple** یک struct است، سریع‌تر است، نام‌گذاری اعضا را پشتیبانی می‌کند و از Deconstruction برخوردار است.

🔹 Tuple برای **بازگرداندن چند مقدار از متد**، **گروه‌بندی موقت داده‌ها** و **جایگزینی کلاس‌های کوچک** مناسب است.

🔹 تفاوت‌های کلیدی با Array، Class و Anonymous Type را یاد گرفتیم و می‌دانیم هر کدام در چه موقعیتی استفاده می‌شوند.

🔹 کاربردهای رایج شامل LINQ، Dictionary با کلید مرکب، جایگزینی `out parameters` و Pattern Matching است.

🔹 Tuple برای **داده‌های موقت و ساده** عالی است، اما برای **مدل‌های پیچیده و ماندگار** باید از class یا record استفاده کرد.

> 📌 **قانون طلایی:** اگر داده‌های شما **موقت، ساده و برای انتقال بین متدها** هستند → **Tuple**. اگر **ماندگار، پیچیده یا دارای رفتار** هستند → **Class/Record**.

---

## منابع و مطالعه بیشتر

### منبع ۱: Microsoft Learn - Tuples
**توضیح:** مستندات رسمی مایکروسافت درباره Tuple در C#، شامل سینتکس، مثال‌ها و کاربردها.
🔗 [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)

### منبع ۲: Microsoft Learn - Tuple Types (C# Programming Guide)
**توضیح:** راهنمای برنامه‌نویسی C# درباره انواع Tuple، شامل تاریخچه و مقایسه با System.Tuple.
🔗 [https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/tuples](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/tuples)

### منبع ۳: C# Language Specification - Tuple Types
**توضیح:** مشخصات رسمی زبان C# درباره Tuple Types، شامل قوانین زبان و رفتار کامپایلر.
🔗 [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#11716-tuple-expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#11716-tuple-expressions)

### منبع ۴: Microsoft Learn - ValueTuple Struct
**توضیح:** مستندات API برای ساختار `ValueTuple<T1, T2, ...>` در .NET.
🔗 [https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple)

### منبع ۵: Microsoft Learn - Deconstruct
**توضیح:** آموزش Deconstruction در C# که برای تجزیه Tupleها استفاده می‌شود.
🔗 [https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct)

### منبع ۶: C# in Depth (Jon Skeet) - Chapter on Tuples
**توضیح:** کتاب معتبر Jon Skeet درباره تاریخچه و طراحی Tuple در C#.
🔗 [http://csharpindepth.com/](http://csharpindepth.com/)

### منبع ۷: .NET Blog - Discover the tuples
**توضیح:** مقاله رسمی تیم .NET درباره معرفی ValueTuple در C# 7.0.
🔗 [https://devblogs.microsoft.com/dotnet/2017/03/09/discover-the-tuples/](https://devblogs.microsoft.com/dotnet/2017/03/09/discover-the-tuples/)

---

> 📝 **نویسنده:** این مقاله برای Repository آموزشی C# تهیه شده است.
> 📅 **آخرین به‌روزرسانی:** August 2026
> 🎯 **سطح:** مقدماتی تا پیشرفته
> 🛠️ **نسخه C# مورد نیاز:** C# 7.0 به بعد (برای ValueTuple)