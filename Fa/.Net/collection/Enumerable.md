

# 📘 راهنمای جامع مفهوم Enumerable در #C
**از مقدماتی تا پیشرفته | مناسب برای توسعه‌دهندگان .NET**

به این آموزش خوش آمدید! در این مستندات، یکی از مهم‌ترین و در عین حال درک‌نشده‌ترین مفاهیم در #C یعنی **Enumerable** را کالبدشکافی می‌کنیم. اگر می‌خواهید بدانید پشت پرده حلقه `foreach` چه می‌گذرد یا LINQ چگونه کار می‌کند، این راهنما برای شماست.

---

## 📑 فهرست مطالب
1. [مفهوم Enumerable (مجموعه‌های قابل پیمایش)](#1-مفهوم-enumerable)
2. [اینترفیس‌های `IEnumerable` و `IEnumerable<T>`](#2-اینترفیسهای-ienumerable-و-ienumerablet)
3. [متد `GetEnumerator()` و نقش آن](#3-متد-getenumerator)
4. [Enumerable به‌عنوان تولیدکننده Enumerator](#4-enumerable-بعنوان-تولیدکننده-enumerator)
5. [پشت پرده `foreach` (کشف یک راز!)](#5-پشت-پرده-foreach)
6. [Enumeration مجدد (آیا می‌توانیم دوباره پیمایش کنیم؟)](#6-enumeration-مجدد)
7. [تفاوت Enumerable با Cursor در پایگاه داده](#7-تفاوت-enumerable-با-cursor)
8. [مباحث پیشرفته: `yield return` و اجرای تاخیری](#8-مباحث-پیشرفته)
9. [منابع معتبر برای مطالعه بیشتر](#9-منابع-معتبر)

---

## 1. مفهوم Enumerable
کلمه **Enumerable** به معنای «قابل شمارش» یا «قابل پیمایش» است. در #C، هرگاه شما مجموعه‌ای از داده‌ها داشته باشید که بتوانید یکی‌یکی روی عناصر آن راه بروید (مثل آرایه‌ها، لیست‌ها، دیکشنری‌ها و...)، با یک Enumerable طرف هستید.

💡 **یک مثال ساده برای درک بهتر:**
فرض کنید یک **کتاب** (Enumerable) در دست دارید. کتاب به خودی خود مجموعه‌ای از صفحات است. برای خواندن کتاب، شما به یک **انگشت** (Enumerator) نیاز دارید تا خط به خط روی کلمات حرکت کند. 
* **Enumerable:** خودِ منبع داده (کتاب)
* **Enumerator:** اشاره‌گری که داده‌ها را یکی‌یکی می‌خواند (انگشت شما)

---

## 2. اینترفیس‌های `IEnumerable` و `IEnumerable<T>`
در #C، برای اینکه یک کلاس بتواند پیمایش شود، باید اینترفیس `IEnumerable` را پیاده‌سازی کند. ما دو نسخه از این اینترفیس داریم:

### الف) `IEnumerable` (نسخه غیر جنریک - قدیمی)
این نسخه از زمان .NET 1.0 وجود داشته و عناصر را به صورت `object` برمی‌گرداند.
```csharp
// پیاده‌سازی غیر جنریک
public class MyCollection : IEnumerable
{
    public IEnumerator GetEnumerator() { ... }
}
```
⚠️ **مشکل:** چون خروجی `object` است، برای انواع داده‌ای Value Type (مثل `int`) باعث **Boxing/Unboxing** و افت شدید عملکرد می‌شود.

### ب) `IEnumerable<T>` (نسخه جنریک - مدرن و پیشنهادی)
این نسخه در .NET 2.0 معرفی شد و **Type-Safe** (امن از نظر نوع داده) است.
```csharp
// پیاده‌سازی جنریک (استاندارد امروزی)
public class MyCollection : IEnumerable<int>
{
    public IEnumerator<int> GetEnumerator() { ... }
}
```
✅ **مزیت:** بدون نیاز به Boxing، دارای IntelliSense قوی و جلوگیری از خطاهای زمان اجرا.

---

## 3. متد `GetEnumerator()`
این متد، قلب تپنده پیمایش است! وقتی شما `IEnumerable` را پیاده‌سازی می‌کنید، **مجبورید** متد `GetEnumerator()` را بنویسید. 
این متد وظیفه دارد یک شیء از نوع `IEnumerator` را بسازد و برگرداند.

```csharp
public IEnumerator<int> GetEnumerator()
{
    // اینجا باید یک Enumerator جدید بسازیم و برگردانیم
    return new MyCustomEnumerator(); 
}
```

---

## 4. Enumerable به‌عنوان تولیدکننده Enumerator
رابط بین این دو، یک رابطه **کارخانه و محصول** است:
* **Enumerable (کارخانه):** منبع داده است. هر بار که بخواهید داده‌ها را بخوانید، از کارخانه درخواست یک خواننده جدید می‌کنید.
* **Enumerator (محصول):** اشاره‌گری است که وضعیت فعلی پیمایش (اینکه کجای داده‌ها هستیم) را در حافظه نگه می‌دارد.

**نکته کلیدی:** یک Enumerable می‌تواند **چندین Enumerator** همزمان تولید کند. یعنی دو نفر می‌توانند همزمان در حال خواندن یک لیست باشند، بدون اینکه روی خوانش یکدیگر تاثیری بگذارند (چون هر کدام Enumerator جداگانه‌ای دارند).

---

## 5. پشت پرده `foreach`
بسیاری از مبتدیان فکر می‌کنند `foreach` یک حلقه جادویی است. اما در واقع، `foreach` فقط یک **Syntactic Sugar** (شیرین‌کننده سینتکس) برای `GetEnumerator` است!

وقتی شما این کد را می‌نویسید:
```csharp
List<int> numbers = new List<int> { 1, 2, 3 };
foreach (var num in numbers)
{
    Console.WriteLine(num);
}
```
کامپایلر #C آن را در پشت صحنه دقیقاً به شکل زیر تبدیل می‌کند:
```csharp
IEnumerator<int> enumerator = numbers.GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
        int num = enumerator.Current;
        Console.WriteLine(num);
    }
}
finally
{
    // آزادسازی منابع اگر Enumerator نیاز به Dispose داشته باشد
    (enumerator as IDisposable)?.Dispose(); 
}
```
*(این بخش را حتماً در آموزش خود برجسته کنید، چون دید برنامه‌نویس را کاملاً تغییر می‌دهد!)*

---

## 6. Enumeration مجدد
**سوال مهم:** آیا می‌توانیم یک Enumerable را دو بار پیمایش کنیم؟
**پاسخ:** بستگی به نوع Enumerable دارد!

### حالت اول: مجموعه‌های در حافظه (Materialized)
مثل `List<T>` یا `Array`. این‌ها داده‌ها را در رم نگه می‌دارند. شما می‌توانید هزار بار `GetEnumerator()` بگیرید و آن‌ها را پیمایش کنید. هر بار یک Enumerator **جدید و مستقل** ساخته می‌شود.
```csharp
var list = new List<int> { 1, 2, 3 };
foreach(var i in list) { } // بار اول
foreach(var i in list) { } // بار دوم - کاملاً اوکی است!
```

### حالت دوم: جریان‌های داده (Streams / Iterators)
مثل خواندن یک فایل بزرگ از روی هارد، یا دریافت دیتا از شبکه. در این حالت، Enumeration مجدد یا **باعث خطا (Exception)** می‌شود، یا **بسیار کند** خواهد بود چون باید کل عملیات خواندن از اول انجام شود.
```csharp
// مثال: خواندن خطوط یک فایل
var lines = File.ReadLines("data.txt"); 
foreach(var line in lines) { } // بار اول خوانده شد
// foreach(var line in lines) { } // بار دوم ممکن است فایل دوباره از اول خوانده شود که بسیار کند است!
```

---

## 7. تفاوت Enumerable با Cursor
اگر با پایگاه داده (SQL) کار کرده باشید، با مفهوم **Cursor** آشنا هستید. تفاوت‌های اصلی به شرح زیر است:

| ویژگی | C# Enumerable / Enumerator | Database Cursor |
| :--- | :--- | :--- |
| **جهت حرکت** | فقط رو به جلو (Forward-Only) | معمولاً دوطرفه (Forward/Backward) |
| **دسترسی** | فقط خواندنی (Read-Only) | قابلیت خواندن و ویرایش (Updateable) |
| **محل پردازش** | درون حافظه (In-Memory) | روی سرور دیتابیس |
| **وضعیت** | وضعیت درون Enumerator ذخیره می‌شود | وضعیت روی سرور DB ذخیره می‌شود |
| **کاربرد** | پیمایش اشیاء و کالکشن‌ها در کد | پیمایش ردیف‌های جدول در دیتابیس |

---

## 8. مباحث پیشرفته
برای اینکه از سطح مقدماتی فراتر بروید، باید با دو مفهوم زیر آشنا شوید:

### الف) اجرای تاخیری (Deferred Execution)
در LINQ، وقتی یک کوئری می‌نویسید، **هیچ داده‌ای در آن لحظه تولید یا فیلتر نمی‌شود!** کوئری فقط یک Enumerable برمی‌گرداند. داده‌ها دقیقاً در لحظه‌ای که `foreach` را اجرا می‌کنید (یا `ToList()` می‌گیرید) تولید می‌شوند.

### ب) جادوی `yield return`
نوشتن کلاس `IEnumerator` به صورت دستی بسیار سخت و طولانی است. #C کلمه کلیدی `yield return` را معرفی کرد تا کامپایلر به صورت خودکار یک State Machine بسازد و کار تولید Enumerator را انجام دهد.

```csharp
// یک Enumerable سفارشی که اعداد زوج را تولید می‌کند
public IEnumerable<int> GetEvenNumbers(int max)
{
    for (int i = 0; i < max; i++)
    {
        if (i % 2 == 0)
        {
            yield return i; // کامپایلر خودش GetEnumerator را می‌سازد!
        }
    }
}
```

---

## 9. منابع معتبر
برای مطالعه عمیق‌تر و ارجاع به منابع اصلی، لینک‌ها و کتاب‌های زیر پیشنهاد می‌شوند:

### 📚 کتاب‌های مرجع:
1. **[C# in Depth](https://www.manning.com/books/c-sharp-in-depth-fourth-edition)** اثر *Jon Skeet* (فصل مربوط به Iterator و Delegateها فوق‌العاده است).
2. **[CLR via C#](https://www.microsoftpressstore.com/store/clr-via-c-4th-edition-9780735667457)** اثر *Jeffrey Richter* (برای درک عمیق مدیریت حافظه و Boxing/Unboxing).

### 🌐 مستندات و مقالات آنلاین:
1. **[Microsoft Docs: IEnumerable<T> Interface](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1)** - مستندات رسمی مایکروسافت.
2. **[Microsoft Docs: yield (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)** - راهنمای رسمی yield return.
3. **[Understanding yield return in C#](https://www.csharpindepth.com/Articles/IteratorBlockImplementation)** - مقاله‌ای از Jon Skeet در وب‌سایت شخصی‌اش.
4. **[Enumerable vs Enumerator - StackOverflow](https://stackoverflow.com/questions/6199765/enumerable-vs-enumerator)** - یک تاپیک بسیار خواندنی برای رفع شبهات.

---
> 💡 **نکته پایانی برای خواننده:** 
> درک مفهوم Enumerable و Enumerator، مرز بین یک برنامه‌نویس معمولی و یک برنامه‌نویس حرفه‌ای #C است. هر زمان که از LINQ استفاده می‌کنید، به یاد بیاورید که در حال کار با یک کارخانه تولید Enumerator هستید!

***

### 📝 راهنمای استفاده برای شما (صاحب ریپازیتوری):
* این متن را در یک فایل به نام `Enumerable_Guide.md` یا در `README.md` ریپازیتوری خود کپی کنید.
* اگر ریپازیتوری شما شامل کدهای تمرینی است، می‌توانید برای بخش ۵ و ۸، پوشه‌هایی به نام `01_UnderTheHood_Foreach` و `02_YieldReturn` ایجاد کنید و لینک آن‌ها را در همین فایل Markdown قرار دهید.
* ساختار Markdown به گونه‌ای تنظیم شده که در گیت‌هاب به زیبایی رندر (Render) می‌شود و جدول‌ها و بولت‌پوینت‌ها خوانایی بالایی دارند.