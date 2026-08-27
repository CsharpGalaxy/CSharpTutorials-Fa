# انواع Tuple در C# — راهنمای جامع از مقدماتی تا پیشرفته

---

## 📑 فهرست مطالب

1. [مقدمه](#مقدمه)
2. [Tuple چیست؟](#tuple-چیست)
3. [System.Tuple چیست؟](#systemtuple-چیست)
4. [System.ValueTuple چیست؟](#systemvaluetuple-چیست)
5. [تفاوت System.Tuple و System.ValueTuple](#تفاوت-systemtuple-و-systemvaluetuple)
   - 5.1 [Tuple به‌عنوان Reference Type](#51-tuple-به‌عنوان-reference-type)
   - 5.2 [ValueTuple به‌عنوان Value Type](#52-valuetuple-به‌عنوان-value-type)
   - 5.3 [تفاوت نحوه ایجاد](#53-تفاوت-نحوه-ایجاد)
   - 5.4 [تفاوت در Mutable و Immutable بودن](#54-تفاوت-در-mutable-و-immutable-بودن)
   - 5.5 [تفاوت در Deconstruction](#55-تفاوت-در-deconstruction)
   - 5.6 [تفاوت در Equality](#56-تفاوت-در-equality)
   - 5.7 [تفاوت در Performance](#57-تفاوت-در-performance)
6. [کاربرد امروزی System.Tuple](#کاربرد-امروزی-systemtuple)
7. [چه زمانی ValueTuple انتخاب بهتری است؟](#چه-زمانی-valuetuple-انتخاب-بهتری-است)
8. [مثال عملی کامل](#مثال-عملی-کامل)
9. [جدول مقایسه‌ای](#جدول-مقایسه‌ای)
10. [نکات مهم](#نکات-مهم)
11. [اشتباهات رایج](#اشتباهات-رایج)
12. [جمع‌بندی](#جمع‌بندی)
13. [منابع](#منابع)

---

## مقدمه

در برنامه‌نویسی C#، گاهی نیاز دارید چندین مقدار ناهمگن را در یک ساختار واحد بسته‌بندی کنید بدون آن‌که بخواهید یک کلاس یا رکورد مجزا تعریف کنید. **Tuple** دقیقاً برای همین سناریو طراحی شده است. در طول تاریخ C#، دو نسل از Tuple وجود داشته:

- **`System.Tuple`** (از .NET Framework 4.0 — سال ۲۰۱۰)
- **`System.ValueTuple`** (از C# 7.0 و .NET Framework 4.7 — سال ۲۰۱۷)

درک تفاوت‌های بنیادین این دو، هم از نظر معماری و هم از نظر عملکرد، برای نوشتن کد بهینه و مدرن ضروری است.

---

## Tuple چیست؟

> **تعریف:** Tuple یک ساختار داده‌ای است که تعداد مشخصی از عناصر (معمولاً ناهمگن از نظر نوع) را در کنار هم نگه می‌دارد.

در ریاضیات، Tuple یک دنبالهٔ متناهی و مرتب از عناصر است. در C#، این مفهوم به‌صورت تایپ‌های جنریک پیاده‌سازی شده که به شما اجازه می‌دهند ۱ تا ۸ (و با تکنیک‌های Nesting، بیش‌تر) مقدار را در یک شیء واحد نگهداری کنید.

**سناریوهای رایج استفاده:**
- بازگرداندن چندین مقدار از یک متد
- انتقال داده‌های موقت بین لایه‌ها
- کلیدهای مرکب در Dictionary
- نتایج کوئری‌های LINQ

---

## System.Tuple چیست؟

`System.Tuple` یک **کلاس جنریک** است که در فضای نام `System` قرار دارد و از .NET Framework 4.0 معرفی شد.

### ویژگی‌های کلیدی:

| ویژگی | توضیح |
|---|---|
| نوع | `class` (Reference Type) |
| تغییرپذیری | **Immutable** (غیرقابل‌تغییر) |
| محل ذخیره‌سازی | Heap |
| نسخه معرفی | .NET Framework 4.0 |

### مثال پایه:

```csharp
// ایجاد یک Tuple با دو عنصر
var person = new Tuple<string, int>("Ali", 30);

// دسترسی به عناصر — فقط از طریق Item1, Item2, ...
Console.WriteLine(person.Item1); // خروجی: Ali
Console.WriteLine(person.Item2); // خروجی: 30

// ❌ خطای کامپایل — Tuple غیرقابل‌تغییر است
// person.Item1 = "Reza";  // Error: Property or indexer 'Tuple<...>.Item1' cannot be assigned to
```

**توضیح خط‌به‌خط:**
- **خط ۲:** یک شیء `Tuple<string, int>` روی Heap ساخته می‌شود.
- **خط ۵ و ۶:** دسترسی فقط از طریق خواص `Item1` و `Item2` امکان‌پذیر است (نام‌گذاری معنادار وجود ندارد).
- **خط ۹:** تلاش برای تغییر مقدار با خطای کامپایل مواجه می‌شود زیرا خواص `Item` فقط `get` دارند.

### متد کمکی `Tuple.Create`:

```csharp
// استفاده از Type Inference — نیازی به ذکر صریح تایپ نیست
var point = Tuple.Create(10, 20);
// نوع point: Tuple<int, int>
```

---

## System.ValueTuple چیست؟

`System.ValueTuple` یک **ساختار (struct) جنریک** است که از C# 7.0 و .NET Framework 4.7 معرفی شد و همراه با **Syntactic Sugar** قدرتمندی ارائه گردید.

### ویژگی‌های کلیدی:

| ویژگی | توضیح |
|---|---|
| نوع | `struct` (Value Type) |
| تغییرپذیری | **Mutable** (قابل‌تغییر) |
| محل ذخیره‌سازی | Stack (در حالت عادی) |
| نسخه معرفی | C# 7.0 / .NET Framework 4.7 |

### مثال پایه:

```csharp
// روش ۱: استفاده از Literal Syntax (توصیه‌شده)
var person = (Name: "Ali", Age: 30);

// روش ۲: استفاده صریح از نوع
ValueTuple<string, int> person2 = ("Ali", 30);

// دسترسی با نام‌های معنادار
Console.WriteLine(person.Name); // خروجی: Ali
Console.WriteLine(person.Age);  // خروجی: 30

// ✅ قابل‌تغییر است
person.Name = "Reza";
Console.WriteLine(person.Name); // خروجی: Reza
```

**توضیح خط‌به‌خط:**
- **خط ۲:** از سینتکس `(Name: "Ali", Age: 30)` استفاده شده که در واقع `ValueTuple<string, int>` است. نام‌های `Name` و `Age` فقط در سطح کامپایلر وجود دارند (در IL به `Item1` و `Item2` تبدیل می‌شوند).
- **خط ۵:** ساختار صریح `ValueTuple<string, int>` بدون نام‌گذاری فیلدها.
- **خط ۸ و ۹:** دسترسی از طریق نام‌های معنادار که خوانایی کد را به‌شدت افزایش می‌دهد.
- **خط ۱۲:** برخلاف `System.Tuple`، فیلدهای `ValueTuple` قابل تغییر هستند.

---

## تفاوت System.Tuple و System.ValueTuple

### 5.1 Tuple به‌عنوان Reference Type

`System.Tuple` یک `class` است، بنابراین:

- هر بار که یک Tuple ایجاد می‌کنید، یک شیء جدید روی **Heap** تخصیص داده می‌شود.
- متغیر شما فقط یک **مرجع** (Reference) به آن شیء است.
- مقایسهٔ دو متغیر با `==` به‌صورت پیش‌فرض **مرجع** را مقایسه می‌کند (نه محتوا را).
- در سناریوهای پرفشار، ایجاد تعداد زیادی Tuple باعث **فشار بر Garbage Collector** می‌شود.

```csharp
var t1 = new Tuple<int, string>(1, "Hello");
var t2 = new Tuple<int, string>(1, "Hello");

Console.WriteLine(t1 == t2);       // False — مقایسه مرجع
Console.WriteLine(t1.Equals(t2));  // True  — مقایسه ساختاری
```

### 5.2 ValueTuple به‌عنوان Value Type

`System.ValueTuple` یک `struct` است، بنابراین:

- مقدار مستقیماً روی **Stack** ذخیره می‌شود (مگر این‌که داخل یک class یا boxing شود).
- تخصیص حافظه Heap و فشار بر GC **حداقل** است.
- کپی‌برداری به‌صورت **مقداری** (By Value) انجام می‌شود.

```csharp
var v1 = (1, "Hello");
var v2 = (1, "Hello");

Console.WriteLine(v1 == v2);  // True — مقایسه ساختاری (از C# 7.3)
```

> ⚠️ **نکته مهم:** اگر `ValueTuple` داخل یک `class` قرار بگیرد یا به `object` تبدیل شود (Boxing)، روی Heap ذخیره خواهد شد.

### 5.3 تفاوت نحوه ایجاد

| روش | System.Tuple | System.ValueTuple |
|---|---|---|
| Constructor | `new Tuple<int, string>(1, "A")` | `new ValueTuple<int, string>(1, "A")` |
| Factory Method | `Tuple.Create(1, "A")` | — |
| Literal Syntax | ❌ پشتیبانی نمی‌شود | `(1, "A")` یا `(Id: 1, Name: "A")` |
| Target-typed `new` (C# 9+) | `Tuple<int, string> t = new(1, "A")` | `(int, string) v = (1, "A")` |

```csharp
// ✅ ValueTuple — سینتکس مدرن و خوانا
(int Id, string Name) user = (1, "Sara");

// ❌ System.Tuple — سینتکس قدیمی و پُرنویس
Tuple<int, string> user2 = new Tuple<int, string>(1, "Sara");
```

### 5.4 تفاوت در Mutable و Immutable بودن

| | System.Tuple | System.ValueTuple |
|---|---|---|
| تغییرپذیری | **Immutable** (فقط خواندنی) | **Mutable** (قابل تغییر) |
| دلیل | خواص `Item` فقط `get` دارند | فیلدها `public` و قابل‌نوشتن هستند |

```csharp
// System.Tuple — Immutable
var t = Tuple.Create(10, 20);
// t.Item1 = 99;  // ❌ خطای کامپایل

// System.ValueTuple — Mutable
var v = (X: 10, Y: 20);
v.X = 99;  // ✅ بدون مشکل
Console.WriteLine(v.X);  // خروجی: 99
```

> ⚠️ **هشدار:** تغییرپذیری `ValueTuple` می‌تواند در سناریوهای همزمانی (Concurrency) مشکل‌ساز شود. اگر به تغییرناپذیری نیاز دارید، از `record` یا `System.Tuple` استفاده کنید.

### 5.5 تفاوت در Deconstruction

**Deconstruction** به فرآیند تجزیهٔ یک Tuple به متغیرهای مجزا گفته می‌شود.

#### ValueTuple — پشتیبانی کامل و طبیعی:

```csharp
var person = (Name: "Ali", Age: 30, City: "Tehran");

// Deconstruction کامل
var (name, age, city) = person;
Console.WriteLine($"{name}, {age}, {city}");  // Ali, 30, Tehran

// Deconstruction با Discard
var (n, _, c) = person;  // سن نادیده گرفته شد

// Deconstruction در حلقه foreach
var list = new List<(string Name, int Score)>
{
    ("Ali", 95),
    ("Sara", 88)
};

foreach (var (studentName, score) in list)
{
    Console.WriteLine($"{studentName}: {score}");
}
```

#### System.Tuple — پشتیبانی محدود (از C# 7.0+):

```csharp
var person = Tuple.Create("Ali", 30);

// Deconstruction از C# 7.0 به بعد ممکن است، اما...
// ❌ به‌صورت پیش‌فرض Deconstruct ندارد!
// var (name, age) = person;  // خطای کامپایل

// ✅ راه‌حل: تعریف Extension Method
public static class TupleExtensions
{
    public static void Deconstruct<T1, T2>(
        this Tuple<T1, T2> tuple,
        out T1 item1,
        out T2 item2)
    {
        item1 = tuple.Item1;
        item2 = tuple.Item2;
    }
}

// حالا کار می‌کند:
var (name, age) = person;  // ✅
```

> 💡 **نتیجه:** `ValueTuple` به‌صورت ذاتی از Deconstruction پشتیبانی می‌کند، اما `System.Tuple` نیاز به Extension Method دارد.

### 5.6 تفاوت در Equality

#### System.Tuple:

```csharp
var t1 = Tuple.Create(1, "A");
var t2 = Tuple.Create(1, "A");

// اپراتور == → مقایسه مرجع
Console.WriteLine(t1 == t2);       // False

// متد Equals → مقایسه ساختاری
Console.WriteLine(t1.Equals(t2));  // True

// IStructuralEquatable → مقایسه ساختاری با Comparer سفارشی
IStructuralEquatable se = t1;
Console.WriteLine(se.Equals(t2, EqualityComparer<object>.Default));  // True
```

#### System.ValueTuple:

```csharp
var v1 = (1, "A");
var v2 = (1, "A");

// اپراتور == → مقایسه ساختاری (از C# 7.3)
Console.WriteLine(v1 == v2);  // True

// متد Equals → مقایسه ساختاری
Console.WriteLine(v1.Equals(v2));  // True

// IEquatable<ValueTuple<T1, T2>>
Console.WriteLine(v1.Equals((1, "A")));  // True
```

| معیار | System.Tuple | System.ValueTuple |
|---|---|---|
| `==` | مقایسه **مرجع** | مقایسه **ساختاری** (C# 7.3+) |
| `Equals()` | مقایسه ساختاری | مقایسه ساختاری |
| `IStructuralEquatable` | ✅ پیاده‌سازی شده | ❌ پیاده‌سازی نشده |
| `IEquatable<T>` | ❌ | ✅ پیاده‌سازی شده |

### 5.7 تفاوت در Performance

```csharp
using System.Diagnostics;

const int iterations = 10_000_000;

// بنچمارک System.Tuple
var sw = Stopwatch.StartNew();
for (int i = 0; i < iterations; i++)
{
    var t = Tuple.Create(i, "test");
}
sw.Stop();
Console.WriteLine($"System.Tuple:      {sw.ElapsedMilliseconds} ms");

// بنچمارک System.ValueTuple
sw.Restart();
for (int i = 0; i < iterations; i++)
{
    var v = (i, "test");
}
sw.Stop();
Console.WriteLine($"System.ValueTuple:  {sw.ElapsedMilliseconds} ms");
```

**نتایج تقریبی (بسته به سخت‌افزار):**

| معیار | System.Tuple | System.ValueTuple |
|---|---|---|
| تخصیص Heap | ✅ هر بار یک شیء جدید | ❌ بدون تخصیص (روی Stack) |
| فشار بر GC | **بالا** | **حداقل** |
| سرعت ایجاد | کندتر | سریع‌تر |
| مصرف حافظه | بیشتر (Object Header + Reference) | کمتر (فقط داده‌ها) |

> 💡 **نکته:** در حلقه‌های پُرتکرار و مسیرهای بحرانی (Hot Paths)، `ValueTuple` می‌تواند **۲ تا ۱۰ برابر** سریع‌تر باشد.

---

## کاربرد امروزی System.Tuple

در عمل، `System.Tuple` امروزه **به‌ندرت** در کدهای جدید استفاده می‌شود. با این حال، هنوز در موارد زیر دیده می‌شود:

1. **کدهای قدیمی (Legacy):** پروژه‌هایی که قبل از C# 7.0 نوشته شده‌اند.
2. **سازگاری با کتابخانه‌های قدیمی:** برخی APIهای قدیمی .NET Framework هنوز `System.Tuple` برمی‌گردانند.
3. **نیاز به تغییرناپذیری:** اگر صراحتاً به یک ساختار Immutable نیاز دارید و نمی‌خواهید از `record` استفاده کنید.
4. **IStructuralEquatable:** اگر به مقایسه ساختاری با `Comparer` سفارشی از طریق اینترفیس `IStructuralEquatable` نیاز دارید.

> 🔴 **توصیه:** در پروژه‌های جدید، **همیشه** `ValueTuple` را ترجیح دهید مگر دلیل مشخصی برای استفاده از `System.Tuple` داشته باشید.

---

## چه زمانی ValueTuple انتخاب بهتری است؟

| سناریو | انتخاب | دلیل |
|---|---|---|
| بازگرداندن چند مقدار از متد | ✅ `ValueTuple` | سینتکس ساده، بدون تخصیص Heap |
| نتایج موقت LINQ | ✅ `ValueTuple` | عملکرد بالا در حلقه‌ها |
| کلید مرکب Dictionary | ✅ `ValueTuple` | `GetHashCode` بهینه |
| انتقال داده بین لایه‌ها | ⚠️ `record` بهتر است | خوانایی و تغییرناپذیری |
| API عمومی (Public API) | ⚠️ `record` یا `class` | نام‌های فیلد در مرز اسمبلی از بین می‌روند |
| کد Legacy | `System.Tuple` | سازگاری |

> ⚠️ **نکته مهم درباره نام فیلدها:** نام‌های فیلد `ValueTuple` (مثل `Name` و `Age`) فقط در **سطح کامپایلر** و با استفاده از `TupleElementNamesAttribute` حفظ می‌شوند. اگر `ValueTuple` را از مرز یک اسمبلی به اسمبلی دیگر (بدون ارجاع مستقیم) منتقل کنید، نام‌ها از بین می‌روند و فقط `Item1` و `Item2` باقی می‌مانند.

---

## مثال عملی کامل

### سناریو: سیستم مدیریت دانشجویان

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// ═══════════════════════════════════════════════════
// بخش ۱: استفاده از System.Tuple (روش قدیمی)
// ═══════════════════════════════════════════════════
public class LegacyStudentService
{
    // متد قدیمی که Tuple برمی‌گرداند
    public Tuple<string, double, string> GetStudentReport(int studentId)
    {
        // شبیه‌سازی دریافت از دیتابیس
        string name = "Ali Mohammadi";
        double gpa = 18.5;
        string status = "Excellent";

        // ایجاد Tuple — پُرنویس و بدون نام معنادار
        return Tuple.Create(name, gpa, status);
    }

    public void PrintReport(int studentId)
    {
        var report = GetStudentReport(studentId);

        // ❌ دسترسی فقط از طریق Item1, Item2, Item3
        // خوانایی بسیار پایین — کدام Item چیست؟
        Console.WriteLine($"Name:   {report.Item1}");
        Console.WriteLine($"GPA:    {report.Item2}");
        Console.WriteLine($"Status: {report.Item3}");
    }
}

// ═══════════════════════════════════════════════════
// بخش ۲: استفاده از System.ValueTuple (روش مدرن)
// ═══════════════════════════════════════════════════
public class ModernStudentService
{
    // متد مدرن که ValueTuple با نام‌های معنادار برمی‌گرداند
    public (string Name, double Gpa, string Status) GetStudentReport(int studentId)
    {
        string name = "Ali Mohammadi";
        double gpa = 18.5;
        string status = "Excellent";

        // ایجاد ValueTuple — ساده، خوانا، بدون تخصیص Heap
        return (name, gpa, status);
    }

    public void PrintReport(int studentId)
    {
        var report = GetStudentReport(studentId);

        // ✅ دسترسی با نام‌های معنادار
        Console.WriteLine($"Name:   {report.Name}");
        Console.WriteLine($"GPA:    {report.Gpa}");
        Console.WriteLine($"Status: {report.Status}");

        // ✅ Deconstruction
        var (name, gpa, status) = report;
        Console.WriteLine($"{name} has GPA {gpa} ({status})");
    }

    // مثال پیشرفته: استفاده در LINQ
    public List<(string Name, string Grade)> GetTopStudents(
        List<(string Name, double Gpa)> students)
    {
        return students
            .Where(s => s.Gpa >= 17.0)
            .OrderByDescending(s => s.Gpa)
            .Select(s => (
                s.Name,
                Grade: s.Gpa >= 19.0 ? "A+" :
                       s.Gpa >= 18.0 ? "A"  :
                       s.Gpa >= 17.0 ? "B+" : "B"
            ))
            .ToList();
    }

    // مثال پیشرفته: ValueTuple به‌عنوان کلید Dictionary
    public Dictionary<(int Year, int Semester), double> GetSemesterAverages()
    {
        return new Dictionary<(int Year, int Semester), double>
        {
            [(1402, 1)] = 17.5,
            [(1402, 2)] = 18.2,
            [(1403, 1)] = 19.0,
        };
    }
}

// ═══════════════════════════════════════════════════
// بخش ۳: اجرای برنامه
// ═══════════════════════════════════════════════════
class Program
{
    static void Main()
    {
        Console.WriteLine("=== Legacy (System.Tuple) ===");
        var legacy = new LegacyStudentService();
        legacy.PrintReport(1);

        Console.WriteLine("\n=== Modern (System.ValueTuple) ===");
        var modern = new ModernStudentService();
        modern.PrintReport(1);

        // LINQ Example
        var students = new List<(string Name, double Gpa)>
        {
            ("Ali", 19.2),
            ("Sara", 17.8),
            ("Reza", 15.5),
            ("Mina", 18.5)
        };

        Console.WriteLine("\n=== Top Students ===");
        foreach (var (name, grade) in modern.GetTopStudents(students))
        {
            Console.WriteLine($"{name}: {grade}");
        }

        // Dictionary Example
        Console.WriteLine("\n=== Semester Averages ===");
        var averages = modern.GetSemesterAverages();
        foreach (var ((year, semester), avg) in averages)
        {
            Console.WriteLine($"{year}-{semester}: {avg}");
        }
    }
}
```

**خروجی:**
```
=== Legacy (System.Tuple) ===
Name:   Ali Mohammadi
GPA:    18.5
Status: Excellent

=== Modern (System.ValueTuple) ===
Name:   Ali Mohammadi
GPA:    18.5
Status: Excellent
Ali Mohammadi has GPA 18.5 (Excellent)

=== Top Students ===
Ali: A+
Mina: A
Sara: B+

=== Semester Averages ===
1402-1: 17.5
1402-2: 18.2
1403-1: 19
```

---

## جدول مقایسه‌ای

| معیار | `System.Tuple` | `System.ValueTuple` |
|---|---|---|
| **نوع** | `class` (Reference Type) | `struct` (Value Type) |
| **معرفی** | .NET Framework 4.0 (2010) | C# 7.0 / .NET 4.7 (2017) |
| **تغییرپذیری** | Immutable | Mutable |
| **محل ذخیره‌سازی** | Heap | Stack (معمولاً) |
| **سینتکس Literal** | ❌ | ✅ `(x, y)` |
| **نام‌گذاری فیلدها** | ❌ فقط `Item1, Item2, ...` | ✅ `(Name: "A", Age: 1)` |
| **Deconstruction** | ❌ نیاز به Extension | ✅ ذاتی |
| **عملگر `==`** | مقایسه مرجع | مقایسه ساختاری (C# 7.3+) |
| **`IEquatable<T>`** | ❌ | ✅ |
| **`IStructuralEquatable`** | ✅ | ❌ |
| **عملکرد** | کندتر (Heap + GC) | سریع‌تر (Stack) |
| **فشار بر GC** | بالا | حداقل |
| **حداکثر عناصر مستقیم** | ۸ (با Rest برای بیشتر) | ۸ (با Rest برای بیشتر) |
| **Boxing** | نیازی نیست | در صورت cast به `object` |
| **استفاده در Public API** | قابل قبول | ⚠️ نام فیلدها ممکن است از بین برود |
| **توصیه برای کد جدید** | ❌ | ✅ |

---

## نکات مهم

> 💡 **۱.** از C# 7.3 به بعد، اپراتورهای `==` و `!=` برای `ValueTuple` پشتیبانی می‌شوند و مقایسه ساختاری انجام می‌دهند.

> 💡 **۲.** نام فیلدهای `ValueTuple` در IL به `TupleElementNamesAttribute` تبدیل می‌شوند. در Reflection، فیلدها همچنان `Item1` و `Item2` هستند.

> 💡 **۳.** برای Tuple با بیش از ۷ عنصر، از فیلد `Rest` استفاده می‌شود که خود یک Tuple تودرتو است:
> ```csharp
> var big = (1, 2, 3, 4, 5, 6, 7, 8, 9);
> Console.WriteLine(big.Rest.Item2);  // خروجی: 9
> ```

> 💡 **۴.** `ValueTuple` از `ITuple` (از .NET Core 2.0) پشتیبانی می‌کند که امکان دسترسی ایندکسی را فراهم می‌کند:
> ```csharp
> ITuple tuple = (10, 20, 30);
> Console.WriteLine(tuple[1]);  // خروجی: 20
> Console.WriteLine(tuple.Length);  // خروجی: 3
> ```

> 💡 **۵.** در C# 12+ و .NET 8+، اگر به ساختارهای داده‌ای پیچیده‌تر نیاز دارید، `record struct` را به‌عنوان جایگزین تغییرناپذیر `ValueTuple` در نظر بگیرید.

---

## اشتباهات رایج

### ❌ اشتباه ۱: استفاده از `System.Tuple` در کد جدید
```csharp
// ❌ بد
public Tuple<string, int> GetUser() => Tuple.Create("Ali", 30);

// ✅ خوب
public (string Name, int Age) GetUser() => ("Ali", 30);
```

### ❌ اشتباه ۲: فراموش کردن Boxing در ValueTuple
```csharp
var v = (1, 2);
object boxed = v;  // ⚠️ Boxing رخ داد — ValueTuple روی Heap رفت!
var unboxed = ((int, int))boxed;  // Unboxing
```

### ❌ اشتباه ۳: انتظار حفظ نام فیلدها در مرز اسمبلی‌ها
```csharp
// Assembly A:
public (string Name, int Age) GetData() => ("Ali", 30);

// Assembly B (بدون ارجاع مستقیم به A):
var data = GetData();
// data.Name  → ❌ ممکن است در دسترس نباشد
// data.Item1 → ✅ همیشه کار می‌کند
```

### ❌ اشتباه ۴: استفاده از ValueTuple Mutable در سناریوهای Concurrent
```csharp
// ❌ خطرناک — ValueTuple قابل‌تغییر است
var shared = (Count: 0, Total: 0.0);
Parallel.For(0, 1000, i =>
{
    shared.Count++;   // ⚠️ Race Condition!
    shared.Total += i;
});

// ✅ راه‌حل: از Interlocked یا ساختارهای Thread-Safe استفاده کنید
```

### ❌ اشتباه ۵: مقایسه System.Tuple با `==`
```csharp
var t1 = Tuple.Create(1, 2);
var t2 = Tuple.Create(1, 2);
if (t1 == t2)  // ❌ همیشه False! (مقایسه مرجع)
{
    Console.WriteLine("Equal");  // هرگز اجرا نمی‌شود
}

// ✅ راه‌حل:
if (t1.Equals(t2))  // True
```

---

## جمع‌بندی

| | System.Tuple | System.ValueTuple |
|---|---|---|
| **خلاصه** | نسل اول، Reference Type، Immutable | نسل دوم، Value Type، Mutable |
| **وضعیت فعلی** | Legacy — فقط برای سازگاری | **استاندارد فعلی** |

**توصیه نهایی:**

1. در **تمام پروژه‌های جدید**، از `ValueTuple` با سینتکس `(Type1 Name1, Type2 Name2)` استفاده کنید.
2. اگر به **تغییرناپذیری** نیاز دارید، `record` یا `record struct` را ترجیح دهید.
3. `System.Tuple` را فقط در **کدهای Legacy** یا زمانی که API قدیمی آن را برمی‌گرداند استفاده کنید.
4. از `ValueTuple` در **Public API** با احتیاط استفاده کنید (نام فیلدها در مرز اسمبلی‌ها حفظ نمی‌شوند).
5. در **حلقه‌های پُرتکرار**، `ValueTuple` به‌دلیل عدم تخصیص Heap، عملکرد بهتری دارد.

---

## منابع

1. **Microsoft Learn — Tuple types (C# reference)**
   <https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples>

2. **Microsoft Learn — System.Tuple Class**
   <https://learn.microsoft.com/en-us/dotnet/api/system.tuple>

3. **Microsoft Learn — System.ValueTuple Struct**
   <https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple>

4. **Microsoft Learn — C# 7.0 Release Notes (Tuples)**
   <https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7#tuples>

5. **C# Language Specification — Tuples**
   <https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types#tuple-types>

6. **Microsoft Learn — Deconstructing Tuples**
   <https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct>

7. **Microsoft Learn — ITuple Interface**
   <https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.ituple>

---

> 📝 **نویسنده:** این مقاله بخشی از Repository آموزشی C# است.
>
> 🔄 **آخرین به‌روزرسانی:** ۲۰۲۶
>
> 📌 **سطح:** مقدماتی تا پیشرفته
>
> 🏷️ **برچسب‌ها:** `C#`, `Tuple`, `ValueTuple`, `Performance`, `Data-Structures`