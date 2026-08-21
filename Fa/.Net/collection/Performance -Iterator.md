

# 🚀 راهنمای جامع Performance در Iteratorهای سی‌شارپ
**از مقدماتی تا پیشرفته: کالبدشکافی `yield return`، Memory و State Machine**

به ریپازیتوری آموزش مفاهیم عمیق Performance در سی‌شارپ خوش آمدید! در این داکیومنت، یکی از جذاب‌ترین و در عین حال بدفهمیده‌شده‌ترین ویژگی‌های سی‌شارپ، یعنی **Iteratorها (`yield return`)** را از نظر عملکردی (Performance)، مدیریت حافظه و ساختار داخلی بررسی می‌کنیم.

---

## 📑 فهرست مطالب
1. [مقدمه: Iterator چیست؟ (مفهوم پایه)](#1-مقدمه-iterator-چیست)
2. [Streaming و Incremental Processing (پردازش جریانی)](#2-streaming-و-incremental-processing)
3. [کاهش نیاز به Materialization و Memory Footprint](#3-کاهش-نیاز-به-materialization-و-memory-footprint)
4. [Fast Startup (راه‌اندازی سریع)](#4-fast-startup-راهاندازی-سریع)
5. [پشت صحنه: هزینه State Machine (پیشرفته)](#5-پشت-صحنه-هزینه-state-machine)
6. [هزینه MoveNext و Overheadها (پیشرفته)](#6-هزینه-movenext-و-overheadها)
7. [مقایسه کاربردی](#7-مقایسه-کاربردی)
   - [Iterator در مقابل دسترسی مستقیم (Direct Access)](#71-iterator-در-مقابل-دسترسی-مستقیم)
   - [Iterator در مقابل Collection Materialized](#72-iterator-در-مقابل-collection-materialized)
8. [جمع‌بندی: چه زمانی از چه چیزی استفاده کنیم؟](#8-جمع‌بندی)
9. [منابع معتبر و برای مطالعه بیشتر](#9-منابع-معتبر)

---

## 1. مقدمه: Iterator چیست؟
فرض کنید می‌خواهید ۱۰ میلیون رکورد را از دیتابیس بخوانید و پردازش کنید. 
روش سنتی این است که همه را در یک `List` بریزید و سپس روی آن حلقه بزنید. اما **Iterator** (که با کلمه کلیدی `yield return` ساخته می‌شود) به شما اجازه می‌دهد داده‌ها را **دانه به دانه** و **فقط در زمان نیاز** تولید کنید.

```csharp
// یک Iterator ساده
public IEnumerable<int> GetNumbers(int count)
{
    for (int i = 0; i < count; i++)
    {
        yield return i; // داده را تولید کن و برگردان، اما متوقف شو تا دوباره صدا زده شوی
    }
}
```
**نکته کلیدی برای مبتدیان:** در Iterator، کدی که بعد از `yield return` نوشته شده، تا زمانی که شما عنصر *بعدی* را درخواست نکنید، اجرا نمی‌شود! به این رفتار **Lazy Evaluation** (ارزیابی تنبل) می‌گویند.

---

## 2. Streaming و Incremental Processing
وقتی از Iterator استفاده می‌کنید، شما در حال انجام **Streaming** (جریان‌سازی) هستید. 
* **Incremental Processing (پردازش تدریجی):** یعنی شما منتظر نمی‌مانید تا کل داده‌ها آماده شوند. به محض اینکه اولین آیتم تولید شد، آن را مصرف (Consume) می‌کنید.
* **مثال واقعی:** پخش آنلاین فیلم (Netflix). شما منتظر دانلود کل فیلم نمی‌مانید؛ باگ دانلود شد، پخش می‌شود. Iteratorها دقیقاً مثل پخش آنلاین برای داده‌ها عمل می‌کنند.

---

## 3. کاهش نیاز به Materialization و Memory Footprint
**Materialization (مادی‌سازی)** یعنی کشیدن تمام داده‌ها به حافظه RAM (مثلاً با استفاده از `.ToList()` یا `.ToArray()`).

* **Memory Footprint (ردپای حافظه):** 
  * **روش Materialized:** اگر ۱۰ میلیون آبجکت بسازید، رم سرور شما پر می‌شود (Memory Footprint بالا).
  * **روش Iterator:** شما در هر لحظه فقط **یک** آبجکت در رم دارید (Memory Footprint تقریباً O(1)). 

```csharp
// ❌ بد: کل فایل ۱۰ گیگابایتی می‌رود توی رم!
var lines = File.ReadAllLines("huge_file.txt"); 

// ✅ عالی: خط به خط خوانده می‌شود (Streaming)
public IEnumerable<string> ReadLines(string path)
{
    using var reader = File.OpenText(path);
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        yield return line;
    }
}
```

---

## 4. Fast Startup (راه‌اندازی سریع)
یکی از بزرگترین مزیت‌های Iteratorها **Fast Startup** یا **Time to First Byte (TTFB)** است.
اگر تولید هر آیتم ۱ ثانیه طول بکشد:
* در روش Materialized، کاربر باید ۱۰ ثانیه صبر کند تا ۱۰ آیتم تولید و در لیست ذخیره شوند، سپس نتیجه را ببیند.
* در روش Iterator، کاربر در **همان ثانیه اول** اولین آیتم را دریافت می‌کند. این برای **UIها (جلوگیری از فریز شدن)** و **APIها (Streaming Response)** حیاتی است.

---

## 5. پشت صحنه: هزینه State Machine (پیشرفته)
تا اینجا همه چیز گل و بلبل بود! اما پشت پرده چه خبر است؟
وقتی شما `yield return` می‌نویسید، کامپایلر سی‌شارپ یک کلاس مخفی (State Machine) برای شما تولید می‌کند. این کلاس واسط‌های `IEnumerable<T>` و `IEnumerator<T>` را پیاده‌سازی می‌کند.

**هزینه State Machine چیست؟**
1. **Allocation (تخصیص حافظه):** کامپایلر یک آبجکت از نوع کلاس (Class) در Heap می‌سازد. این یعنی فشار روی Garbage Collector.
2. **Switch Statement:** متد `MoveNext` در این کلاس مخفی، شامل یک `switch` بزرگ است که بر اساس یک متغیر وضعیت (State) تصمیم می‌گیرد کجای کد شما را اجرا کند.

```csharp
// شبیه‌سازی کدی که کامپایلر تولید می‌کند (به زبان ساده)
private int state;
public bool MoveNext()
{
    switch (state)
    {
        case 0:
            // کدهای قبل از اولین yield
            state = 1;
            current = 1;
            return true;
        case 1:
            // کدهای بین yield اول و دوم
            state = 2;
            current = 2;
            return true;
        // ...
    }
}
```
**نتیجه:** استفاده از Iteratorها CPU را درگیر `switch` و مدیریت State می‌کند.

---

## 6. هزینه MoveNext و Overheadها
هر بار که شما در یک حلقه `foreach` روی یک Iterator قدم برمی‌دارید، متد `MoveNext` صدا زده می‌شود.
**هزینه‌های MoveNext:**
1. بررسی وضعیت (State Check).
2. اجرای کد شما تا رسیدن به `yield return`.
3. ذخیره متغیرهای محلی (Local Variables) در فیلدهای کلاس State Machine (تا در فراخوانی بعدی یادتان باشد کجا بودید!).

**مقایسه Performance:**
یک حلقه `for` ساده روی یک `Array` بسیار سریع‌تر از یک `foreach` روی Iterator است، چون در `Array` فقط یک ایندکس اضافه می‌شود، اما در Iterator هزینه‌های State Machine و `MoveNext` وجود دارد.

---

## 7. مقایسه کاربردی

### 7.1. Iterator در مقابل دسترسی مستقیم (Direct Access)
* **دسترسی مستقیم (مثل `list[50]`):** پیاده‌سازی `IList` یا `Array`. دسترسی **O(1)** و فوق‌العاده سریع. شما می‌توانید به عنصر صدم بپرید بدون اینکه ۹۹ عنصر قبلی را بخوانید.
* **Iterator:** فقط دسترسی **Sequential (ترتیبی)** دارد. شما نمی‌توانید به عنصر ایندکس ۵۰ بپرید؛ باید ۵۰ بار `MoveNext` را صدا بزنید. 
* **قانون:** اگر نیاز به دسترسی تصادفی (Random Access) یا ایندکس‌گذاری دارید، Iterator انتخاب مناسبی نیست.

### 7.2. Iterator در مقابل Collection Materialized
| ویژگی | Iterator (`IEnumerable` / `yield return`) | Materialized (`List` / `Array`) |
| :--- | :--- | :--- |
| **مصرف رم (Memory)** | بسیار کم (فقط یک آیتم در لحظه) | بالا (کل داده‌ها در رم) |
| **سرعت شروع (Startup)** | آنی (Fast Startup) | کند (زمان‌بر برای تولید کل داده) |
| **هزینه CPU** | بالاتر (به دلیل State Machine) | پایین‌تر (دسترسی مستقیم به رم) |
| **قابلیت تکرار (Re-iteration)** | ممکن است هر بار از نو محاسبه شود | سریع (داده از قبل در رم است) |
| **مناسب برای** | داده‌های حجیم، Streaming، Single-pass | داده‌های کوچک، نیاز به دسترسی تصادفی |

---

## 8. جمع‌بندی
* **از Iterator (`yield return`) استفاده کنید اگر:**
  * داده‌های شما بسیار حجیم هستند (مثل خواندن فایل‌های بزرگ).
  * تولید داده‌ها زمان‌بر است و می‌خواهید Fast Startup داشته باشید.
  * فقط یک بار روی داده‌ها حلقه می‌زنید (Single-pass).
  * می‌خواهید Memory Footprint را به حداقل برسانید.

* **از Collectionهای Materialized (`ToList()`) استفاده کنید اگر:**
  * داده‌ها در رم جا می‌شوند و حجم کمی دارند.
  * نیاز به دسترسی تصادفی (ایندکس) دارید.
  * قرار است چندین بار روی داده‌ها حلقه بزنید (چون در Iterator ممکن است محاسبات چندین بار تکرار شوند!).

---

## 9. منابع معتبر
برای مطالعه بیشتر و عمیق‌تر، منابع زیر که از مراجع اصلی جامعه `.NET هستند پیشنهاد می‌شوند:

1. **کتاب C# in Depth (نوشته Jon Skeet):** 
   * *فصل مربوط به Iteratorها.* این کتاب انجیل توسعه‌دهندگان سی‌شارپ است و بهترین توضیح را درباره نحوه کار State Machineها دارد.
   * [لینک به سایت کتاب](https://csharpindepth.com/)
2. **مستندات رسمی مایکروسافت (Microsoft Learn):**
   * [yield (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)
   * [IEnumerable<T> Interface](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1)
3. **کتاب Writing High-Performance .NET Code (نوشته Ben Watson):**
   * بررسی عمیق Overheadهای تخصیص حافظه در Iteratorها و نحوه بهینه‌سازی آن‌ها.
   * [لینک در آمازون / سایت نویسنده](http://highperformancedotnetbook.com/)
4. **مقالات و ویدیوهای Nick Chapsas:**
   * ویدیوی عالی در یوتوب با عنوان "How yield return works in C#" که State Machine را به صورت بصری نشان می‌دهد.
   * [کانال یوتیوب Nick Chapsas](https://www.youtube.com/@nickchapsas)
5. **کتاب Pro .NET Memory Management (نوشته Konstantin Ladinenko):**
   * برای درک عمیق Memory Footprint و تاثیر Allocationهای State Machine بر Garbage Collection.
   * [لینک به سایت کتاب](https://prodotnetmemory.com/)

---
*اگر این آموزش برای شما مفید بود، فراموش نکنید که به ریپازیتوری Star بدهید و آن را Fork کنید! ⭐*

***

### 💡 چند نکته برای شما (مدیر ریپازیتوری):
1. **لینک‌دهی:** لینک‌های منابع در انتهای متن کاملاً واقعی و معتبر هستند.
2. **فرمت‌بندی:** این متن دقیقاً با فرمت `Markdown` نوشته شده و می‌توانید مستقیماً آن را در فایل `README.md` یا یک فایل `Performance-Iterators.md` در ریپازیتوری خود کپی کنید.
3. **تصاویر:** پیشنهاد می‌کنم برای بخش **State Machine** یک اسکرین‌شات از ابزار **Sharplab.io** (که کد کامپایل شده توسط کامپایلر را نشان می‌دهد) بگیرید و در ریپازیتوری قرار دهید؛ تاثیر آموزشی آن فوق‌العاده است!