

# 🚀 آموزش جامع Enumeration و Enumerator در سی‌شارپ: از مقدماتی تا پیشرفته

به بخش جذاب و زیرساختی سی‌شارپ خوش آمدید! اگر تا به حال از حلقه `foreach` استفاده کرده‌اید، یعنی از مفهوم **Enumeration** بهره برده‌اید. اما در پشت پرده این حلقه ساده، چه اتفاقاتی می‌افتد؟ 

در این آموزش، ما از مفاهیم پایه شروع کرده و تا عمق کامپایلر سی‌شارپ و مبحث پیشرفته **Duck Typing** پیش می‌رویم.

---

## 📑 فهرست مطالب
1. [مفهوم Enumeration و Enumerator](#concepts)
2. [تفاوت Enumerable و Enumerator](#differences)
3. [آشنایی با IEnumerator و IEnumerator<T>](#interfaces)
4. [متدها و پراپرتی‌های کلیدی: MoveNext و Current](#members)
5. [متد GetEnumerator و شرایط قابل پیمایش بودن یک Type](#getenumerator)
6. [جادوی Duck Typing در حلقه foreach (مبحث پیشرفته)](#duck-typing)
7. [پیاده‌سازی عملی یک Enumerator سفارشی](#implementation)
8. [منابع معتبر](#references)

---

<h2 id="concepts">۱. مفهوم Enumeration و Enumerator</h2>

برای درک بهتر، بیایید از یک مثال دنیای واقعی استفاده کنیم: **یک کتاب و یک خواننده.**

* **مفهوم Enumeration (پیمایش/شمارش):** 
  به **فرآیند** خواندن کتاب صفحه به صفحه، از ابتدا تا انتها، Enumeration گفته می‌شود. در برنامه‌نویسی، Enumeration یعنی فرآیند پیمایش تک‌تک عناصر یک مجموعه (Collection) به ترتیب، بدون اینکه بدانیم آن مجموعه در حافظه چگونه پیاده‌سازی شده است.

* **مفهوم Enumerator (پیمایش‌گر/شمارنده):** 
  اگر Enumeration "فرآیند خواندن" باشد، Enumerator خودِ **"خواننده"** یا **"نشانگر کتاب"** است. Enumerator یک شیء (Object) است که وظیفه دارد وضعیت فعلی پیمایش را در حافظه نگه دارد (مثلاً بداند الان روی کدام عنصر است) و عنصر بعدی را به ما تحویل دهد.

---

<h2 id="differences">۲. تفاوت Enumerable و Enumerator</h2>

این دو کلمه بسیار به هم شبیه‌اند اما تفاوت ظریف و مهمی دارند:

| ویژگی | Enumerable (پیمایش‌پذیر) | Enumerator (پیمایش‌گر) |
| :--- | :--- | :--- |
| **نقش** | خودِ **مجموعه** (مثل کتاب) | **ابزار پیمایش** مجموعه (مثل نشانگر صفحه) |
| **وظیفه** | تولید کردن یک Enumerator | حرکت روی عناصر و برگرداندن آن‌ها |
| **اینترفیس** | `IEnumerable` یا `IEnumerable<T>` | `IEnumerator` یا `IEnumerator<T>` |
| **مثال** | `List<T>`, `Array`, `string` | شیء‌ای که از `GetEnumerator()` برگردانده می‌شود |

**نکته کلیدی:** یک شیء `Enumerable` می‌تواند چندین `Enumerator` مختلف را همزمان تولید کند (مثلاً دو حلقه foreach تو در تو روی یک لیست).

---

<h2 id="interfaces">۳. آشنایی با IEnumerator و IEnumerator<T></h2>

در سی‌شارپ، رفتار یک Enumerator توسط اینترفیس‌ها تعریف می‌شود:

### الف) `IEnumerator` (نسخه غیر جنریک)
این اینترفیس در نسخه‌های اولیه .NET معرفی شد و از نوع `object` استفاده می‌کند.
```csharp
public interface IEnumerator
{
    object Current { get; }
    bool MoveNext();
    void Reset();
}
```
*مشکل:* چون از `object` استفاده می‌کند، برای انواع مقداری (Value Types مثل `int`) باعث ایجاد پدیده **Boxing/Unboxing** و افت شدید عملکرد (Performance) می‌شود.

### ب) `IEnumerator<T>` (نسخه جنریک)
برای حل مشکل عملکرد، این اینترفیس معرفی شد.
```csharp
public interface IEnumerator<T> : IEnumerator, IDisposable
{
    new T Current { get; } // پراپرتی Current را با نوع T بازنویسی می‌کند
}
```
*مزایا:* 
۱. **Type-Safe** است (نیازی به Cast کردن نیست).
۲. **بدون Boxing/Unboxing** است (سرعت بسیار بالا).
۳. از `IDisposable` ارث‌بری می‌کند تا در صورت نیاز به منابعی مثل دیتابیس یا فایل، آن‌ها را آزاد کند.

---

<h2 id="members">۴. متدها و پراپرتی‌های کلیدی: MoveNext و Current</h2>

یک Enumerator در واقع یک **ماشین حالت (State Machine)** ساده است. بیایید ببینیم چگونه کار می‌کند:

1. **حالت اولیه:** وقتی Enumerator تازه ساخته می‌شود، نشانگر آن **قبل از اولین عنصر** قرار دارد.
2. **`MoveNext()`:** این متد نشانگر را یک قدم به جلو می‌برد. 
   * اگر عنصری وجود داشته باشد، مقدار `true` برمی‌گرداند.
   * اگر به انتهای مجموعه رسیده باشیم، مقدار `false` برمی‌گرداند.
3. **`Current`:** عنصری که نشانگر الان روی آن قرار دارد را برمی‌گرداند.

**⚠️ هشدار مهم:** اگر قبل از فراخوانی `MoveNext()` یا بعد از اینکه `MoveNext()` مقدار `false` برگرداند، از `Current` استفاده کنید، برنامه **Exception** پرتاب می‌کند!

**شکل عملکرد در کد:**
```csharp
IEnumerator<int> enumerator = myList.GetEnumerator();

// حالت اولیه: نشانگر قبل از عنصر اول است
while (enumerator.MoveNext()) // نشانگر را جلو می‌برد
{
    int item = enumerator.Current; // عنصر فعلی را می‌خواند
    Console.WriteLine(item);
}
// در اینجا MoveNext مقدار false برگردانده و پیمایش تمام است
```

---

<h2 id="getenumerator">۵. متد GetEnumerator و شرایط قابل پیمایش بودن یک Type</h2>

وقتی شما می‌نویسید `foreach (var item in collection)`، کامپایلر سی‌شارپ در پشت صحنه این کد را به شکل زیر بازنویسی می‌کند:

```csharp
var enumerator = collection.GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
        var item = enumerator.Current;
        // بدنه حلقه foreach
    }
}
finally
{
    // اگر Enumerator از IDisposable پیروی کند، آن را Dispose می‌کند
    (enumerator as IDisposable)?.Dispose(); 
}
```

### شرایط یک Type برای قابل پیمایش (Iterable) بودن:
برای اینکه بتوانید روی یک کلاس یا ساختار حلقه `foreach` بزنید، آن Type باید شرایط زیر را داشته باشد:
1. باید یک متد عمومی (Public) به نام **`GetEnumerator()`** داشته باشد (بدون پارامتر).
2. شیئی که `GetEnumerator()` برمی‌گرداند باید دارای متد **`MoveNext()`** باشد که یک `bool` برگرداند.
3. شیئی که `GetEnumerator()` برمی‌گرداند باید دارای پراپرتی **`Current`** باشد.

---

<h2 id="duck-typing">۶. جادوی Duck Typing در حلقه foreach (مبحث پیشرفته)</h2>

تا اینجا فکر می‌کردیم که برای استفاده از `foreach`، کلاس ما **حتماً** باید اینترفیس `IEnumerable` را پیاده‌سازی کند. **اما این تصور اشتباه است!**

در سی‌شارپ، کامپایلر برای `foreach` از الگویی به نام **Duck Typing** (نوع‌سنجی اردکی) استفاده می‌کند. 
شعار Duck Typing این است: *"اگر مثل اردک راه می‌رود و مثل اردک صدا در می‌آورد، پس اردک است!"*

یعنی کامپایلر سی‌شارپ **چک نمی‌کند** که آیا کلاس شما `IEnumerable` را پیاده‌سازی کرده است یا خیر؛ بلکه فقط **شکل (Shape)** و **امضای متدها** را بررسی می‌کند.

### چرا سی‌شارپ این کار را می‌کند؟ (دلیل عملکردی)
اگر کامپایلر سخت‌گیری می‌کرد و فقط `IEnumerable` را قبول می‌کرد، ما مجبور بودیم `GetEnumerator` را در استراکچرها (Structs) به صورت اینترفیس برگردانیم. این کار باعث **Boxing** (تبدیل استراکچر به object در هپ) می‌شد و **عملکرد را به شدت افت می‌داد**.

**مثال عملی Duck Typing:**
در این مثال، کلاس ما `IEnumerable` را پیاده‌سازی **نکرده**، اما `foreach` روی آن کار می‌کند!

```csharp
// این کلاس IEnumerable را پیاده‌سازی نکرده است!
public class CustomRange 
{
    private int _start;
    private int _end;

    public CustomRange(int start, int end) { _start = start; _end = end; }

    // کامپایلر فقط نام متد و خروجی آن را چک می‌کند
    public CustomRangeEnumerator GetEnumerator() 
    {
        return new CustomRangeEnumerator(_start, _end);
    }
}

// این استراکچر هم IEnumerator را پیاده‌سازی نکرده است!
public struct CustomRangeEnumerator
{
    private int _current;
    private int _end;

    public CustomRangeEnumerator(int start, int end)
    {
        _current = start - 1; // حالت اولیه
        _end = end;
    }

    // شرط ۱: متد MoveNext
    public bool MoveNext()
    {
        _current++;
        return _current <= _end;
    }

    // شرط ۲: پراپرتی Current
    public int Current => _current;
}

// تست در برنامه اصلی:
class Program
{
    static void Main()
    {
        var range = new CustomRange(1, 5);
        
        // این کد بدون هیچ خطایی کامپایل و اجرا می‌شود! (به لطف Duck Typing)
        foreach (var num in range)
        {
            Console.WriteLine(num); // چاپ 1 تا 5
        }
    }
}
```
*نکته حرفه‌ای:* دقیقاً به همین دلیل است که `List<T>.Enumerator` یک **Struct** است و `IEnumerable` را پیاده‌سازی نمی‌کند تا از Boxing جلوگیری کند و سرعت `foreach` روی لیست‌ها فوق‌العاده بالا باشد.

---

<h2 id="implementation">۷. پیاده‌سازی عملی یک Enumerator سفارشی</h2>

اگر بخواهیم یک کلاس بنویسیم که هم `IEnumerable` را پیاده‌سازی کند و هم یک Enumerator سفارشی داشته باشد:

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

public class StudentCollection : IEnumerable<string>
{
    private string[] _students = { "Ali", "Sara", "Reza" };

    // پیاده‌سازی اینترفیس جنریک
    public IEnumerator<string> GetEnumerator()
    {
        return new StudentEnumerator(_students);
    }

    // پیاده‌سازی اینترفیس غیر جنریک (الزامی برای تطابق با IEnumerable)
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    // کلاس Enumerator سفارشی ما
    private class StudentEnumerator : IEnumerator<string>
    {
        private string[] _data;
        private int _position = -1; // حالت اولیه: قبل از عنصر اول

        public StudentEnumerator(string[] data)
        {
            _data = data;
        }

        public string Current
        {
            get
            {
                if (_position == -1 || _position >= _data.Length)
                    throw new InvalidOperationException();
                return _data[_position];
            }
        }

        object IEnumerator.Current => Current;

        public bool MoveNext()
        {
            _position++;
            return (_position < _data.Length);
        }

        public void Reset() => _position = -1;

        public void Dispose()
        {
            // اگر منبعی باز شده، اینجا آزاد می‌شود
        }
    }
}
```

---

<h2 id="references">۸. منابع معتبر</h2>

این مقاله بر اساس منابع مرجع و معتبر زیر در زمینه زبان سی‌شارپ و معماری .NET گردآوری شده است:

1. **مستندات رسمی مایکروسافت (Microsoft Learn / Docs)**
   * [IEnumerable<T> Interface](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1)
   * [IEnumerator<T> Interface](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerator-1)
   * [How to: Implement IEnumerable(T)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-implement-ienumerable-of-t)
2. **کتاب C# in Depth (نوشته Jon Skeet)**
   * *فصل مربوط به Iterator Blocks و Duck Typing در حلقه‌های foreach.* (این کتاب مرجع اصلی درک عمیق از Duck Typing در سی‌شارپ است).
3. **کتاب CLR via C# (نوشته Jeffrey Richter)**
   * *بخش مربوط به Collections و بررسی عملکرد (Performance) و Boxing/Unboxing در Enumeratorها.*
4. **کتاب Pro C# with .NET (نوشته Andrew Troelsen)**
   * *فصل Custom Indexers and Iterators برای درک نحوه پیاده‌سازی سفارشی Enumerator.*

---
> 💡 **نکته برای خوانندگان:** اگر این آموزش برای شما مفید بود، فراموش نکنید که به این ریپازیتوری ستاره (⭐) بدهید و آن را با دیگر برنامه‌نویسان به اشتراک بگذارید! برای آموزش‌های بیشتر، سایر پوشه‌های ریپازیتوری را بررسی کنید.