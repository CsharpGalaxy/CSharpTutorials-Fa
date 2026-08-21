

# 📘 راهنمای جامع `IDisposable` و `IEnumerator` در #C
**از مدیریت منابع تا جادوی پشت پرده `foreach`**

## 📑 فهرست مطالب
1. [مقدمه: چرا این مبحث مهم است؟](#مقدمه)
2. [بخش اول: مفاهیم پایه](#بخش-اول-مفاهیم-پایه)
   - [مدیریت منابع و `IDisposable`](#1-مدیریت-منابع-و-idisposable)
   - [پیمایش‌گر یا `IEnumerator` چیست؟](#2-پیمایش‌گر-ienumerator-چیست)
3. [بخش دوم: ارتباط پنهان (رابطه این دو)](#بخش-دوم-ارتباط-پنهان)
   - [چرا `IEnumerator<T>` ارث‌بری از `IDisposable` دارد؟](#رابطه-ienumerable-و-idisposable)
4. [بخش سوم: حلقه `foreach` و جادوی `Dispose`](#بخش-سوم-حلقه-foreach-و-جادوی-dispose)
   - [پشت پرده `foreach` چه می‌گذرد؟](#پشت-پرده-foreach)
   - [نقش حیاتی `finally`](#نقش-finally)
   - [تفاوت پایان طبیعی Sequence و فراخوانی Dispose](#پایان-طبیعی-در-برابر-dispose)
5. [بخش چهارم: پیمایش دستی (Manual Enumeration)](#بخش-چهارم-پیمایش-دستی)
6. [بخش پنجم: مفاهیم پیشرفته](#بخش-پنجم-مفاهیم-پیشرفته)
   - [تاثیر `yield return` بر Dispose](#جادوی-yield-return)
   - [دنیای Async و `IAsyncDisposable`](#دنیای-async)
7. [نتیجه‌گیری](#نتیجهگیری)
8. [منابع و مراجع معتبر](#منابع-و-مراجع)

---

<a name="مقدمه"></a>
## 🎯 مقدمه: چرا این مبحث مهم است؟
تصور کنید در حال خواندن یک کتاب هستید (پیمایش در یک مجموعه داده). وقتی کتاب تمام شد یا تصمیم گرفتید آن را نیمه‌کاره رها کنید، آیا آن را روی میز رها می‌کنید یا در قفسه می‌گذارید؟ 
در برنامه‌نویسی، مجموعه‌های داده (مثل لیست‌ها، یا کوئری‌های دیتابیس) منابعی هستند که ممکن است نیاز به "بسته شدن" یا "آزادسازی" داشته باشند. در این مقاله یاد می‌گیریم چگونه زبان C# با ترکیب `IDisposable` و `IEnumerator` تضمین می‌کند که هیچ منبعی هرگز باز جا نمی‌ماند، حتی اگر شما حلقه را با `break` نیمه‌کاره رها کنید!

---

<a name="بخش-اول-مفاهیم-پایه"></a>
## 🟢 بخش اول: مفاهیم پایه

### 1. مدیریت منابع و `IDisposable`
رابط `IDisposable` تنها یک متد دارد: `Dispose()`. 
**کاربرد:** وقتی کلاسی منبعی را در اختیار می‌گیرد که **خارج از کنترل Garbage Collector (GC)** است (مثل فایل‌ها، ارتباط با شبکه، یا Cursor دیتابیس)، باید این اینترفیس را پیاده‌سازی کند تا توسعه‌دهنده بتواند منبع را *سریعاً* آزاد کند.

```csharp
public class DatabaseConnection : IDisposable
{
    public void Dispose()
    {
        // بستن اتصال به دیتابیس و آزادسازی منابع
    }
}
```

### 2. پیمایش‌گر (`IEnumerator`) چیست؟
وقتی شما از یک آرایه یا لیست `foreach` می‌زنید، در واقع در حال استفاده از یک `IEnumerator` هستید. اینترفیس `IEnumerator` مثل یک **نشانگر (Bookmark)** در کتاب عمل می‌کند.
سه عضو اصلی دارد:
*   `MoveNext()`: به صفحه بعد می‌رود (اگر صفحه‌ای باشد `true` برمی‌گرداند).
*   `Current`: آیتم صفحه فعلی را برمی‌گرداند.
*   `Reset()`: به اول کتاب برمی‌گردد (در C# مدرن کمتر استفاده می‌شود).

---

<a name="بخش-دوم-ارتباط-پنهان"></a>
## 🟡 بخش دوم: ارتباط پنهان

### رابطه `IEnumerable` و `IDisposable`
اینجا یک نکته **بسیار مهم و کنکوری** وجود دارد که تفاوت برنامه‌نویسان Junior و Senior را مشخص می‌کند:

1. **`IEnumerator` (غیر جنریک):** از `IDisposable` ارث‌بری **نمی‌کند**. (چون آرایه‌های ساده نیازی به Dispose ندارند).
2. **`IEnumerator<T>` (جنریک):** مستقیماً از `IDisposable` **ارث‌بری می‌کند**.

```csharp
// تعریف در سورس کات #C
public interface IEnumerator<T> : IDisposable, IEnumerator { ... }
```
**چرا؟** چون در دنیای جنریک، ما نمی‌دانیم زیرِ یک `IEnumerable<T>` چه منبعی خوابیده است. ممکن است یک آرایه ساده باشد، یا یک Stream باز از شبکه، یا یک کوئری LINQ to SQL که یک Connection دیتابیس را درگیر کرده است. زبان C# برای اطمینان از عدم نشت منابع (Resource Leak)، فرض را بر این می‌گذارد که *هر* `IEnumerator<T>` باید قابلیت Dispose شدن داشته باشد.

---

<a name="بخش-سوم-حلقه-foreach-و-جادوی-dispose"></a>
## 🟠 بخش سوم: حلقه `foreach` و جادوی `Dispose`

### پشت پرده `foreach`
وقتی شما می‌نویسید:
```csharp
foreach (var item in myCollection)
{
    Console.WriteLine(item);
}
```
کامپایلر C# این کد را به شکل زیر (معادل آن) تبدیل می‌کند:

```csharp
var enumerator = myCollection.GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
        var item = enumerator.Current;
        Console.WriteLine(item);
    }
}
finally
{
    // جادوی اینجاست!
    if (enumerator is IDisposable disposable)
    {
        disposable.Dispose();
    }
}
```

### نقش `finally`
بلوک `finally` تضمین می‌کند که **در هر شرایطی** (چه حلقه تمام شود، چه `break` بخورید، چه `return` کنید و چه خطا/Exception رخ دهد)، متد `Dispose` فراخوانی خواهد شد. این یعنی خداحافظی با نشت منابع در حلقه‌ها!

### تفاوت پایان طبیعی Sequence و فراخوانی Dispose
*   **پایان طبیعی (Natural End):** زمانی که `MoveNext()` مقدار `false` برمی‌گرداند (یعنی به انتهای لیست رسیده‌ایم). در این حالت هم بلوک `finally` اجرا می‌شود و `Dispose` صدا زده می‌شود. *چرا؟* چون کامپایلر از قبل نمی‌داند آیا این Enumerator نیاز به پاکسازی دارد یا خیر، پس برای امنیت بیشتر، همیشه آن را صدا می‌زند.
*   **خروج زودهنگام (Early Exit):** اگر شما وسط حلقه `break` بزنید یا Exception پرتاب شود، بلوک `finally` به عنوان یک **تور ایمنی** وارد عمل شده و منابع باز (مثل دیتابیس) را می‌بندد.

---

<a name="بخش-چهارم-پیمایش-دستی"></a>
## 🔴 بخش چهارم: پیمایش دستی (Manual Enumeration)

گاهی اوقات (مثلاً در الگوریتم‌های پیچیده مثل Merge کردن دو لیست مرتب) نمی‌توانیم از `foreach` استفاده کنیم و باید دستی پیمایش کنیم. **وظیفه شماست که نقش `finally` را بازی کنید!**

❌ **روش غلط (باعث نشت منابع می‌شود):**
```csharp
var enumerator = myCollection.GetEnumerator();
while (enumerator.MoveNext())
{
    if (someCondition) break; // اگر بریک بخورد، Dispose هرگز صدا زده نمی‌شود!
}
```

✅ **روش صحیح (الگوی استاندارد):**
```csharp
var enumerator = myCollection.GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
        var item = enumerator.Current;
        if (someCondition) break; 
    }
}
finally
{
    // بررسی می‌کنیم که آیا اصلا قابلیت Dispose دارد یا خیر
    (enumerator as IDisposable)?.Dispose();
}
```
*نکته پیشرفته:* در پیمایش دستی، چون ممکن است `IEnumerator` غیر جنریک باشد (که `IDisposable` ندارد)، باید با `as IDisposable` یا Pattern Matching چک کنیم که آیا آبجکت قابلیت Dispose دارد یا خیر.

---

<a name="بخش-پنجم-مفاهیم-پیشرفته"></a>
## 🟣 بخش پنجم: مفاهیم پیشرفته

### جادوی `yield return`
وقتی شما از `yield return` برای ساخت یک Sequence سفارشی استفاده می‌کنید، کامپایلر C# یک **State Machine** (کلاس مخفی) می‌سازد. این کلاس مخفی به صورت خودکار `IDisposable` را پیاده‌سازی می‌کند!

```csharp
public IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2; // اگر اینجا break بخوریم، متغیرهای محلی و منابع Dispose می‌شوند
    yield return 3;
}
```
اگر مصرف‌کننده این متد، حلقه را با `break` تمام کند، کامپایلر تضمین می‌کند که State Machine متوقف شده و متد `Dispose` آن (که توسط کامپایلر تولید شده) فراخوانی می‌شود تا وضعیت (State) سرریست شود.

### دنیای Async و `IAsyncDisposable`
در C# 8.0 به بعد، برای پیمایش‌های ناهمگام (مثل خواندن استریم‌های شبکه)، ما `IAsyncEnumerable<T>` داریم که از `IAsyncDisposable` ارث‌بری می‌کند.
در حلقه `await foreach`، کامپایلر به جای `finally` معمولی، از `await using` و `DisposeAsync()` استفاده می‌کند تا منابع ناهمگام به درستی آزاد شوند.

---

<a name="نتیجهگیری"></a>
## 💡 نتیجه‌گیری
1. رابط `IEnumerator<T>` برای جلوگیری از نشت منابع، `IDisposable` را ارث‌بری می‌کند.
2. حلقه `foreach` در C# یک حلقه ساده نیست؛ بلکه یک بلوک `try...finally` است که تضمین می‌کند `Dispose` همیشه (چه در پایان طبیعی و چه در `break`) صدا زده شود.
3. اگر در حال نوشتن پیمایش دستی (`while` + `MoveNext`) هستید، **همیشه** از بلوک `try...finally` استفاده کنید و `Dispose` را فراخوانی نمایید.
4. زبان C# با کامپایلر هوشمند خود، بارِ مدیریت منابع را از دوش برنامه‌نویس برمی‌دارد، اما درک این مکانیزم برای نوشتن کدهای بهینه و بدون باگ (به ویژه در کار با دیتابیس و فایل) الزامی است.

---

<a name="منابع-و-مراجع"></a>
## 📚 منابع و مراجع معتبر
برای مطالعه بیشتر و اطمینان از صحت مطالب، می‌توانید به منابع رسمی زیر مراجعه کنید:

1. **[Microsoft Docs: IDisposable Interface](https://learn.microsoft.com/en-us/dotnet/api/system.idisposable)**
   *مستندات رسمی مایکروسافت درباره نحوه مدیریت منابع و الگوی Dispose.*
2. **[Microsoft Docs: IEnumerator<T> Interface](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerator-1)**
   *بررسی سورس کات و ارث‌بری `IDisposable` در اینترفیس جنریک.*
3. **[C# Language Specification: The foreach statement](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1395-the-foreach-statement)**
   *مستندات رسمی نحوه ترجمه `foreach` به `try...finally` توسط کامپایلر.*
4. **[Microsoft Docs: yield (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)**
   *نحوه عملکرد State Machine و مدیریت Dispose در متدهای Iterator.*
5. **کتاب C# in Depth (نوشته Jon Skeet)**
   *فصل مربوط به Iterator Blocks و Delegates، بهترین مرجع برای درک عمیق State Machineهای تولید شده توسط `yield`.*

---
> 📝 **توسعه‌دهنده:** [نام شما / نام ریپازیتوری شما]
> 📅 **آخرین بروزرسانی:** August 2026
> 💡 *اگر این آموزش برای شما مفید بود، فراموش نکنید که به این ریپازیتوری ستاره (⭐) بدهید!*