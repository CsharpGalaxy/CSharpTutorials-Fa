
# 🚀 راهنمای جامع Iterator و Exception در سی‌شارپ: از مقدماتی تا پیشرفته

به ریپازیتوری آموزشی ما خوش آمدید! در این مقاله، یکی از جذاب‌ترین و در عین حال چالش‌برانگیزترین مباحث سی‌شارپ، یعنی ترکیب **Iteratorها (کلمات کلیدی `yield`)** و **مدیریت خطا (Exception)** را بررسی می‌کنیم. 

این راهنما به گونه‌ای طراحی شده که اگر مبتدی هستید، با مفاهیم پایه شروع کنید و اگر توسعه‌دهنده میان‌رده یا پیشرفته هستید، از نکات عمیق مربوط به ماشین حالت (State Machine) و مدیریت حافظه لذت ببرید.

---

## 📑 فهرست مطالب
1. [مفهوم پایه‌ای Iterator و Deferred Execution (اجرای تأخیری)](#1-deferred-execution)
2. [Exception در Iterator چه زمانی رخ می‌دهد؟](#2-when-exceptions-occur)
3. [محدودیت catch در Iterator (یک قانون مهم کامپایلر)](#3-try-catch-limitations)
4. [نقش finally در Iterator و ارتباط آن با Dispose](#4-finally-and-dispose)
5. [اجرای finally در چه شرایطی تضمین می‌شود؟](#5-finally-execution)
6. [خطر استفاده دستی از Enumerator (چرا foreach بهتر است؟)](#6-manual-enumerator-danger)
7. [جمع‌بندی و بهترین روش‌ها (Best Practices)](#7-best-practices)
8. [منابع معتبر برای مطالعه بیشتر](#8-references)

---

<h2 id="1-deferred-execution">۱. مفهوم پایه‌ای Iterator و Deferred Execution (اجرای تأخیری)</h2>

قبل از ورود به مبحث خطاها، باید یک مفهوم طلایی را درک کنید: **اجرای تأخیری (Deferred Execution)**.
وقتی شما یک متد را می‌نویسید که از `yield return` استفاده می‌کند، کامپایلر سی‌شارپ کد شما را به یک **ماشین حالت (State Machine)** تبدیل می‌کند. 

**قانون طلایی:** کد داخل یک متد Iterator، تا زمانی که شما شروع به پیمایش (Iterate) آن نکنید، **اجرا نمی‌شود**.

```csharp
public IEnumerable<int> GetNumbers()
{
    Console.WriteLine("این کد الان اجرا نمی‌شود!");
    yield return 1;
    Console.WriteLine("این کد فقط وقتی اجرا می‌شود که عدد دوم درخواست شود.");
    yield return 2;
}

// در اینجا هیچ پیامی در کنسول چاپ نمی‌شود و هیچ خطایی رخ نمی‌دهد.
var numbers = GetNumbers(); 

// اینجا کد اجرا می‌شود (فقط عدد اول)
foreach(var num in numbers) { } 
```

---

<h2 id="2-when-exceptions-occur">۲. Exception در Iterator چه زمانی رخ می‌دهد؟</h2>

بسیاری از مبتدیان تصور می‌کنند اگر اعتبارسنجی (Validation) را در ابتدای متد Iterator بنویسند، خطا بلافاصله پس از فراخوانی متد پرتاب می‌شود. **اما اینطور نیست!**

### ❌ حالت اشتباه (خطای تأخیری)
```csharp
public IEnumerable<int> ProcessData(List<int> data)
{
    if (data == null) throw new ArgumentNullException(nameof(data)); // ❌
    
    foreach(var item in data)
        yield return item * 2;
}

// فراخوانی متد هیچ خطایی نمی‌دهد!
var result = ProcessData(null); 

// خطا اینجا (در زمان foreach) رخ می‌دهد!
foreach(var r in result) { } 
```

### ✅ حالت درست (الگوی متد تفکیک‌شده - Split Method Pattern)
اگر می‌خواهید خطا **بلافاصله** (Immediate) رخ دهد، باید متد را به دو بخش تقسیم کنید:

```csharp
// ۱. متد اصلی (بدون yield) برای اعتبارسنجی آنی
public IEnumerable<int> ProcessDataSafe(List<int> data)
{
    if (data == null) throw new ArgumentNullException(nameof(data)); // ✅ خطای آنی
    return ProcessDataIterator(data);
}

// ۲. متد داخلی (با yield) برای اجرای تأخیری
private IEnumerable<int> ProcessDataIterator(List<int> data)
{
    foreach(var item in data)
        yield return item * 2;
}
```

---

<h2 id="3-try-catch-limitations">۳. محدودیت catch در Iterator (یک قانون مهم کامپایلر)</h2>

اگر سعی کنید داخل یک متد Iterator از بلوک `catch` استفاده کنید، کامپایلر سی‌شارپ به شما **خطای کامپایل (CS1626)** می‌دهد.

```csharp
public IEnumerable<int> BadIterator()
{
    try 
    {
        yield return 1;
    }
    catch (Exception ex) // ❌ خطای کامپایلر: Cannot yield a value in the body of a try block with a catch clause
    {
        Console.WriteLine(ex.Message);
    }
}
```

### 🤔 چرا کامپایلر اجازه استفاده از catch را نمی‌دهد؟
دلیل این امر به نحوه کار **ماشین حالت (State Machine)** برمی‌گردد. وقتی شما `yield return` می‌کنید، اجرای متد متوقف شده و وضعیت (State) آن ذخیره می‌شود. اگر خطایی رخ دهد و کامپایلر بخواهد به بلوک `catch` بپرد، بازگرداندن وضعیت (State) به حالت قبل از خطا برای ادامه پیمایش در دفعات بعد، از نظر معماری ماشین حالت سی‌شارپ غیرممکن و بسیار پیچیده است.

**نتیجه:** شما در Iterator فقط می‌توانید از `try...finally` استفاده کنید، نه `try...catch`.

---

<h2 id="4-finally-and-dispose">۴. نقش finally در Iterator و ارتباط آن با Dispose</h2>

از آنجا که نمی‌توانیم خطا را `catch` کنیم، پس چگونه منابع (مثل فایل، دیتابیس یا Network) را آزاد کنیم؟ جواب: **بلوک `finally`**.

وقتی کامپایلر متد Iterator شما را به ماشین حالت تبدیل می‌کند، کلاسی می‌سازد که اینترفیس `IDisposable` را پیاده‌سازی می‌کند. کد داخل `finally` دقیقاً درون متد `Dispose()` این کلاسِ تولید شده قرار می‌گیرد.

```csharp
public IEnumerable<string> ReadLines(string filePath)
{
    StreamReader reader = new StreamReader(filePath);
    try
    {
        string line;
        while ((line = reader.ReadLine()) != null)
        {
            yield return line; // اجرای متد اینجا متوقف می‌شود
        }
    }
    finally
    {
        // این کد زمانی اجرا می‌شود که حلقه تمام شود یا Dispose صدا زده شود
        reader.Dispose(); 
        Console.WriteLine("منابع آزاد شدند.");
    }
}
```

---

<h2 id="5-finally-execution">۵. اجرای finally در چه شرایطی تضمین می‌شود؟</h2>

بلوک `finally` در Iterator در **سه حالت** اجرا می‌شود:
1. **پیمایش کامل:** حلقه `foreach` به طور طبیعی به پایان برسد.
2. **خروج زودهنگام (Break):** شما در وسط حلقه `foreach` از دستور `break` یا `return` استفاده کنید.
3. **بروز Exception:** اگر در حین پیمایش خطایی رخ دهد.

**نکته پیشرفته:** در حلقه `foreach`، کامپایلر سی‌شارپ به طور خودکار یک بلوک `try...finally` در اطراف کد شما می‌سازد و در بخش `finally`، متد `Dispose()` را روی Enumerator صدا می‌زند. این کار باعث می‌شود `finally`ِ داخل متد Iterator شما اجرا شود.

---

<h2 id="6-manual-enumerator-danger">۶. خطر استفاده دستی از Enumerator (چرا foreach بهتر است؟)</h2>

حلقه `foreach` فقط یک شکر‌نوشته (Syntactic Sugar) است. در پشت صحنه، سی‌شارپ از `GetEnumerator()`, `MoveNext()`, `Current` و `Dispose()` استفاده می‌کند.

اگر بخواهید **به صورت دستی** از Enumerator استفاده کنید و حواستان به `Dispose` نباشد، دچار **نشت منابع (Resource Leak)** می‌شوید!

### ❌ کد خطرناک و اشتباه (نشت منابع)
```csharp
var enumerator = ReadLines("data.txt").GetEnumerator();
while (enumerator.MoveNext())
{
    Console.WriteLine(enumerator.Current);
    if (enumerator.Current == "STOP") 
        break; // ❌ اگر اینجا خارج شوید، Dispose صدا زده نمی‌شود و فایل باز می‌ماند!
}
// فراموش کردن Dispose باعث باز ماندن فایل در سطح سیستم عامل می‌شود.
```

### ✅ کد ایمن (معادل دقیق foreach)
اگر مجبور به استفاده دستی هستید، **باید** از `try...finally` و `Dispose` استفاده کنید:

```csharp
var enumerator = ReadLines("data.txt").GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
        Console.WriteLine(enumerator.Current);
        if (enumerator.Current == "STOP") break;
    }
}
finally
{
    // ✅ تضمین آزادسازی منابع حتی در صورت بروز خطا یا break
    enumerator.Dispose(); 
}
```
*به همین دلیل است که در ۹۹٪ مواقع، استفاده از `foreach` یا LINQ ایمن‌تر و خواناتر است.*

---

<h2 id="7-best-practices">۷. جمع‌بندی و بهترین روش‌ها (Best Practices)</h2>

1. **اعتبارسنجی آنی (Eager Validation):** همیشه آرگومان‌ها را قبل از ورود به بخش `yield` چک کنید (با استفاده از Split Method Pattern) تا خطا در زمان فراخوانی متد پرتاب شود، نه در زمان پیمایش.
2. **مدیریت منابع:** اگر در Iterator از منابعی مثل `Stream`, `SqlConnection` یا `File` استفاده می‌کنید، حتماً آن‌ها را در بلوک `try...finally` آزاد کنید.
3. **پرهیز از catch:** بپذیرید که `catch` در Iterator ممنوع است. اگر نیاز به مدیریت خطا دارید، آن را در لایه بالاتر (جایی که `foreach` را صدا می‌زنید) انجام دهید.
4. **استفاده از foreach:** تا حد امکان از پیمایش دستی (`GetEnumerator`) خودداری کنید تا درگیر پیچیدگی‌های `Dispose` نشوید.

---

<h2 id="8-references">۸. منابع معتبر برای مطالعه بیشتر</h2>

این مقاله بر اساس منابع رسمی و کتاب‌های مرجع زیر تدوین شده است:

1. **Microsoft Learn - Iterators (C#)**
   * [مستندات رسمی مبحث yield return و Iteratorها](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/yield)
   * [مستندات مدیریت خطا در Iterators](https://learn.microsoft.com/en-us/dotnet/standard/linq/deferred-execution)
2. **کتاب C# in Depth (نوشته Jon Skeet)**
   * فصل مربوط به Delegates و Iterators (توضیح عالی درباره نحوه کار State Machine و دلیل ممنوعیت catch).
3. **کتاب CLR via C# (نوشته Jeffrey Richter)**
   * فصل مربوط به `IDisposable` و `foreach` (توضیح عمیق درباره ارتباط `Dispose` و بلوک `finally` در سطح IL).
4. **مقاله Jon Skeet در مورد Iterator Exceptions**
   * [بررسی دقیق زمان وقوع خطا در Deferred Execution](https://codeblog.jonskeet.uk/)

---
💡 **اگر این آموزش برای شما مفید بود، لطفاً به ریپازیتوری ما Star بدهید و برای توسعه‌دهندگان دیگر ارسال کنید!** 
*تاریخ آخرین بروزرسانی: ۳۰ مرداد ۱۴۰۵ (August 21, 2026)*