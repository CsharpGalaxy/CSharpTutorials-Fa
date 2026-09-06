
# آموزش جامع و صفر تا صد `PeriodicTimer` در .NET و C#
**(از مقدماتی تا پیشرفته | ویژه .NET 8, .NET 9 و .NET 10)**

---

## فهرست مطالب (Table of Contents)
1. [بخش 1: آشنایی با Timer](#بخش-1-آشنایی-با-timer)
2. [بخش 2: PeriodicTimer از پایه](#بخش-2-periodictimer-از-پایه)
3. [بخش 3: مدل اجرای PeriodicTimer](#بخش-3-مدل-اجرای-periodictimer)
4. [بخش 4: حلقه‌های دوره‌ای](#بخش-4-حلقه‌های-دوره‌ای)
5. [بخش 5: Cancellation](#بخش-5-cancellation)
6. [بخش 6: Dispose و Lifetime](#بخش-6-dispose-و-lifetime)
7. [بخش 7: Concurrency](#بخش-7-concurrency)
8. [بخش 8: Exception Handling](#بخش-8-exception-handling)
9. [بخش 9: استفاده در ASP.NET Core](#بخش-9-استفاده-در-aspnet-core)
10. [بخش 10: کاربردهای واقعی](#بخش-10-کاربردهای-واقعی)
11. [بخش 11: Performance](#بخش-11-performance)
12. [بخش 12: مقایسه‌های مهم](#بخش-12-مقایسه‌های-مهم)
13. [بخش 13: اشتباهات رایج](#بخش-13-اشتباهات-رایج)
14. [بخش 14: مباحث پیشرفته و پروژه واقعی](#بخش-14-مباحث-پیشرفته-و-پروژه-واقعی)
15. [بخش‌های تکمیلی الزامی](#بخش‌های-تکمیلی-الزامی)

---

## بخش 1: آشنایی با Timer

### 1. Timer چیست؟
**توضیح ساده**: تایمر مانند یک ساعت زنگ‌دار است که به برنامه می‌گوید: «هر X ثانیه یک‌بار، این کار خاص را انجام بده».
**توضیح فنی**: تایمر یک مکانیزم زمان‌بندی (Scheduling Mechanism) است که اجازه می‌دهد یک Delegate یا Task به‌صورت دوره‌ای (Periodic) یا یک‌باره (One-shot) در زمان مشخصی در آینده اجرا شود.

**چرا به Timer نیاز داریم؟**
اجرای یک عملیات به‌صورت دوره‌ای یعنی تکرار خودکار یک منطق بدون دخالت کاربر یا تریگر خارجی.
*مثال‌های واقعی*:
- اجرای Job تمیزکاری (Cleanup) هر 5 ثانیه.
- بررسی وضعیت سفارش‌های در انتظار (Polling).
- ارسال Heartbeat یا Health Check به یک سرویس مانیتورینگ.
- دریافت اطلاعات به‌روزرسانی‌شده از یک API خارجی هر 1 دقیقه.

**تفاوت اجرای یک‌باره با دوره‌ای**: یک‌باره (One-shot) فقط یک‌بار پس از گذشت زمان مشخص اجرا می‌شود (مثل `Task.Delay` ساده)، اما دوره‌ای (Periodic) تا زمانی که متوقف نشود، به تکرار ادامه می‌دهد.

### 2. انواع Timer در .NET
در .NET سه پیاده‌سازی اصلی برای Timer وجود دارد:

1. **`System.Threading.Timer`**:
   - **هدف**: قدیمی‌ترین و سبک‌ترین تایمر پایه‌ای.
   - **مدل اجرا**: Callback-based (بر اساس متد بازگشتی).
   - **Threading**: از ThreadPool Thread استفاده می‌کند.
   - **Async/Await**: پشتیبانی مستقیم و تمیزی ندارد (باید `async void` یا `async Task` بدون await در callback استفاده شود که خطرناک است).
   - **کاربرد**: زمانی که به کمترین سربار ممکن نیاز دارید و منطق شما Sync است.

2. **`System.Timers.Timer`**:
   - **هدف**: یک Wrapper حول `System.Threading.Timer` با امکانات بیشتر.
   - **مدل اجرا**: Event-based (رویداد `Elapsed`).
   - **Threading**: به‌طور پیش‌فرض از ThreadPool استفاده می‌کند، اما خاصیت `SynchronizingObject` برای هماهنگی با UI Thread دارد.
   - **Async/Await**: مشابه مورد قبل، مدیریت async در Event Handler پیچیده و مستعد خطا است.
   - **کاربرد**: برنامه‌های قدیمی Windows Forms/WPF یا زمانی که مدل Event-driven الزامی است.

3. **`System.Threading.PeriodicTimer`** (معرفی شده در .NET 6):
   - **هدف**: ارائه یک مدل مدرن، کاملاً Async-first و قابل کنترل برای اجرای دوره‌ای.
   - **مدل اجرا**: Polling-based با استفاده از `async/await` (`WaitForNextTickAsync`).
   - **Threading**: به‌طور کامل با ThreadPool و `async/await` هماهنگ است (Thread را Block نمی‌کند).
   - **Cancellation**: پشتیبانی ذاتی و عالی از `CancellationToken`.
   - **کاربرد**: استاندارد طلایی برای تمام کارهای پس‌زمینه (Background Tasks) در .NET مدرن (.NET 6 به بعد).

#### جدول مقایسه کلی
| ویژگی | `System.Threading.Timer` | `System.Timers.Timer` | `System.Threading.PeriodicTimer` |
| :--- | :--- | :--- | :--- |
| **مدل برنامه‌نویسی** | Callback | Event (`Elapsed`) | Async/Await (`while` loop) |
| **پشتیبانی از Async** | ضعیف (خطرناک) | متوسط (پیچیده) | **عالی (طراحی شده برای آن)** |
| **Cancellation** | دستی و پیچیده | دستی | **ذاتی و ساده (`CancellationToken`)** |
| **مدیریت منابع (Dispose)** | لازم است | لازم است | **لازم است (با `using` ساده می‌شود)** |
| **خطر Reentrancy** | بالا (اگر کار طول بکشد) | بالا | **قابل کنترل توسط توسعه‌دهنده** |
| **مناسب برای .NET مدرن** | خیر | خیر | **بله (انتخاب اول)** |

> **خلاصه این بخش**: تایمرها برای اجرای خودکار کارها در بازه‌های زمانی مشخص استفاده می‌شوند. در حالی که تایمرهای قدیمی بر اساس Callback یا Event هستند و مدیریت Async در آن‌ها دشوار است، `PeriodicTimer` به‌طور خاص برای ادغام روان با `async/await` در .NET 6 و بالاتر طراحی شده است.

---

## بخش 2: PeriodicTimer از پایه

### 3. PeriodicTimer چیست؟
`PeriodicTimer` یک کلاس در Namespace `System.Threading` است که از .NET 6 به بعد در دسترس است. این کلاس برای حل مشکل "Callback Hell" و مدیریت ضعیف `async/await` در تایمرهای قدیمی به وجود آمده است. به جای اینکه منتظر بمانید تا تایمر یک متد را صدا بزند، شما به‌صورت فعالانه (اما بدون اشغال Thread) منتظر "تیک" بعدی تایمر می‌مانید.

**مثال ساده و خط‌به‌خط**:
```csharp
using var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));

while (await timer.WaitForNextTickAsync())
{
    Console.WriteLine("Tick");
}
```
- `using var`: تضمین می‌کند که منابع تایمر در پایان بلاک آزاد (Dispose) می‌شوند.
- `new PeriodicTimer(...)`: یک نمونه با فاصله زمانی 5 ثانیه می‌سازد.
- `while`: حلقه‌ای که تا زمانی که شرط برقرار است ادامه می‌یابد.
- `await timer.WaitForNextTickAsync()`: به‌صورت غیرهمزمان (Async) منتظر می‌ماند تا 5 ثانیه بگذرد. اگر تایمر فعال باشد `true` برمی‌گرداند و اگر Dispose شده باشد `false` برمی‌گرداند.
- `Console.WriteLine`: عملیاتی که در هر تیک اجرا می‌شود.

### 4. ساخت PeriodicTimer
سازنده (Constructor) این کلاس تنها یک پارامتر از نوع `TimeSpan` می‌پذیرد.
```csharp
new PeriodicTimer(TimeSpan.FromSeconds(1));  // هر 1 ثانیه
new PeriodicTimer(TimeSpan.FromMinutes(1));  // هر 1 دقیقه
new PeriodicTimer(TimeSpan.FromHours(1));    // هر 1 ساعت
```
**محدودیت‌ها**:
- مقدار `TimeSpan` باید بزرگتر از صفر باشد (`> TimeSpan.Zero`).
- مقدار آن نباید از `Int32.MaxValue` میلی‌ثانیه (حدود 24.8 روز) بیشتر باشد. در غیر این صورت `ArgumentOutOfRangeException` پرتاب می‌شود.
- **نکته .NET 8+**: اگرچه خود `PeriodicTimer` مستقیماً `TimeProvider` نمی‌گیرد، اما برای تست‌پذیری در .NET 8 به بعد، بهتر است منطق وابسته به زمان را در سرویسی جداگانه قرار دهید که `TimeProvider` دریافت می‌کند.

### 5. WaitForNextTickAsync
این متد قلب تپنده `PeriodicTimer` است.
```csharp
public ValueTask<bool> WaitForNextTickAsync(CancellationToken cancellationToken = default);
```
- **چه کاری انجام می‌دهد؟**: به‌صورت غیرهمزمان منتظر می‌ماند تا بازه زمانی (Interval) بعدی فرا برسد.
- **چرا Async است؟**: تا Thread فعلی (معمولاً ThreadPool Thread) را در زمان انتظار مسدود (Block) نکند و آن را برای پردازش درخواست‌های دیگر آزاد بگذارد.
- **چه زمانی `true` برمی‌گرداند؟**: وقتی بازه زمانی با موفقیت سپری شود.
- **چه زمانی `false` برمی‌گرداند؟**: وقتی تایمر `Dispose` شده باشد.
- **رابطه با CancellationToken**: اگر توکن کنسل شود، متد `OperationCanceledException` پرتاب می‌کند (به جای برگرداندن `false`).
- **مصرف CPU**: در زمان انتظار، **صفر** است. این یک Wait Handle بهینه‌شده در سطح سیستم‌عامل است.

> **خلاصه این بخش**: `PeriodicTimer` با دریافت یک `TimeSpan` معتبر ساخته می‌شود و با متد `WaitForNextTickAsync` به شکلی کاملاً Async و بدون اشغال Thread، انتظار پایان بازه زمانی را می‌کشد.

---

## بخش 3: مدل اجرای PeriodicTimer

### 6. نحوه اجرای Tick
جریان اجرا به این صورت است:

```text
PeriodicTimer Start
      ↓
await WaitForNextTickAsync()  ---> (آزادسازی Thread به ThreadPool)
      ↓
[ انتظار به مدت Interval ]    ---> (مصرف CPU = 0)
      ↓
Tick فرا می‌رسد
      ↓
متد true برمی‌گرداند و Thread از ThreadPool گرفته می‌شود
      ↓
اجرای عملیات (DoWorkAsync)
      ↓
بازگشت به ابتدای حلقه و await WaitForNextTickAsync()
```

### 7. PeriodicTimer و async/await
چرا این مدل برای Async مناسب است؟ زیرا کنترل کامل جریان اجرا (Flow Control) را در اختیار شما قرار می‌دهد.
- `Thread.Sleep(5000)`: Thread را به‌طور کامل فریز می‌کند (بدترین حالت).
- `await Task.Delay(5000)`: یک تایمر جدید یک‌باره می‌سازد، منتظر می‌ماند و سپس کار را انجام می‌دهد. (سربار ساخت تایمر در هر تکرار).
- `await timer.WaitForNextTickAsync()`: از یک تایمر واحد و پایدار استفاده می‌کند که برای اجرای دوره‌ای بهینه‌سازی شده است.

### 8. PeriodicTimer و Thread
- **آیا Thread اختصاصی ایجاد می‌کند؟**: خیر. به‌طور کامل بر روی ThreadPool اجرا می‌شود.
- **Context ادامه اجرا**: در یک Console App یا ASP.NET Core، ادامه اجرا (Continuation) روی یک Thread آزاد از ThreadPool انجام می‌شود. در برنامه‌های UI (مثل WPF/MAUI)، اگر `ConfigureAwait(false)` استفاده نشود، سعی می‌کند به UI Thread برگردد (که برای Background Work اشتباه است؛ همیشه در Background Service از `ConfigureAwait(false)` استفاده کنید یا فرض کنید Context وجود ندارد).

> **خلاصه این بخش**: `PeriodicTimer` با آزاد کردن Thread در زمان انتظار، مقیاس‌پذیری (Scalability) بالایی ایجاد می‌کند. این کلاس Thread جدید نمی‌سازد و کاملاً با مکانیزم ThreadPool و async/await هماهنگ است.

---

## بخش 4: حلقه‌های دوره‌ای

### 9. الگوی استاندارد استفاده
```csharp
using var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));

while (await timer.WaitForNextTickAsync())
{
    await DoWorkAsync();
}
```
- **چرا `using`؟**: برای اطمینان از فراخوانی `Dispose` و جلوگیری از Memory Leak.
- **چرا `while`؟**: چون می‌خواهیم عملیات به‌صورت نامحدود (تا زمان کنسل یا Dispose) تکرار شود.
- **چرا `await` در شرط؟**: برای اینکه اجرای کد تا فرا رسیدن تیک بعدی متوقف شود، بدون بلاک کردن Thread.
- **چه زمانی حلقه تمام می‌شود؟**: وقتی `WaitForNextTickAsync` مقدار `false` برگرداند (یعنی تایمر Dispose شده باشد) یا یک Exception (مثل Cancellation) پرتاب شود.

### 10. PeriodicTimer در برابر Task.Delay
```csharp
// روش 1: Task.Delay
while (true)
{
    await Task.Delay(TimeSpan.FromSeconds(5));
    await DoWorkAsync();
}
```
```csharp
// روش 2: PeriodicTimer
using var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));
while (await timer.WaitForNextTickAsync())
{
    await DoWorkAsync();
}
```
**تفاوت‌های کلیدی**:
1. **Drift (انحراف زمانی)**: در `Task.Delay`، اگر `DoWorkAsync` سه ثانیه طول بکشد، فاصله واقعی بین شروع دو اجرا 8 ثانیه خواهد بود (5 ثانیه تاخیر + 3 ثانیه کار). در `PeriodicTimer`، اگر کار کمتر از 5 ثانیه طول بکشد، تیک بعدی دقیقاً سر 5 ثانیه رخ می‌دهد. اگر کار بیشتر طول بکشد، تیک بعدی *بلافاصله* پس از اتمام کار برگردانده می‌شود (که نیاز به مدیریت Overlap دارد).
2. **Allocation**: `PeriodicTimer` یک شیء واحد می‌سازد. `Task.Delay` در هر تکرار یک شیء `Task` جدید تخصیص می‌دهد (سربار Garbage Collection بیشتر در فرکانس بالا).
3. **Cancellation**: کنسل کردن `PeriodicTimer` تمیزتر و متمرکزتر است.

> **خلاصه این بخش**: الگوی `while (await timer.WaitForNextTickAsync())` استانداردترین روش است. `PeriodicTimer` نسبت به `Task.Delay` در حلقه، سربار کمتر و زمان‌بندی دقیق‌تری (Fixed Period) ارائه می‌دهد.

---

## بخش 5: Cancellation

### 11. CancellationToken
برای توقف ایمن، باید `CancellationToken` را به متد پاس دهید:
```csharp
await timer.WaitForNextTickAsync(cancellationToken);
```
- **چرا مهم است؟**: بدون آن، برنامه ممکن است هرگز به‌طور طبیعی بسته نشود (زیرا حلقه بی‌نهایت است).
- **نحوه اتفاق افتادن**: به محض اینکه `cancellationToken.Cancel()` صدا زده شود، متد `WaitForNextTickAsync` اجرای خود را قطع کرده و `OperationCanceledException` پرتاب می‌کند.
- **توقف صحیح**: این Exception باید در سطح `BackgroundService` مدیریت شود تا حلقه به‌طور تمیز خارج شود.

### 12. Graceful Shutdown
جریان توقف ایمن:
```text
Application Shutdown Triggered
     ↓
CancellationToken.Cancel() صدا زده می‌شود
     ↓
WaitForNextTickAsync استثنا پرتاب می‌کند
     ↓
حلقه while شکسته می‌شود
     ↓
بلوک using به‌طور خودکار Dispose را صدا می‌زند
     ↓
برنامه بدون قطع ناگهانی عملیات بسته می‌شود
```

> **خلاصه این بخش**: همیشه یک `CancellationToken` (معمولاً `stoppingToken` در `BackgroundService`) را به `WaitForNextTickAsync` پاس دهید تا برنامه بتواند به‌صورت ایمن و تمیز (Graceful) خاموش شود.

---

## بخش 6: Dispose و Lifetime

### 13. Dispose در PeriodicTimer
- **چرا قابل Dispose است؟**: زیرا منابع سیستم‌عامل (مثل Timer Queue) را اشغال می‌کند.
- **اتفاق پس از Dispose**: هر فراخوانی در حال انتظار از `WaitForNextTickAsync` فوراً با مقدار `false` (اگر کنسل نشده باشد) یا `OperationCanceledException` (اگر کنسل شده باشد) پایان می‌یابد.
- **چرا `using`؟**: استفاده از `using` یا `await using` تضمین می‌کند که حتی در صورت وقوع Exception، منابع آزاد می‌شوند.

### 14. مدیریت Lifetime
- **داخل Method**: خیر، زیرا با پایان متد، تایمر Dispose شده و کار نمی‌کند.
- **داخل Service (Singleton/BackgroundService)**: **بله**. این بهترین مکان است. تایمر باید هم‌عمر برنامه یا سرویس پس‌زمینه باشد.
- **Scoped**: خیر. در ASP.NET Core، سرویس‌های Scoped با پایان HTTP Request نابود می‌شوند، اما تایمر پس‌زمینه باید فراتر از Request زنده بماند.

> **خلاصه این بخش**: `PeriodicTimer` باید در طول عمر طولانی (مثل `BackgroundService` یا Singleton) ایجاد و با الگوی `using` مدیریت شود تا از نشت منابع جلوگیری شود.

---

## بخش 7: Concurrency

### 15. آیا چند WaitForNextTickAsync مجاز است؟
**خیر، مطلقاً خیر.**
مستندات رسمی مایکروسافت صریحاً بیان می‌کند که `WaitForNextTickAsync` فقط توسط **یک Consumer** در هر لحظه قابل فراخوانی است.
```csharp
// این کد باعث پرتاب InvalidOperationException می‌شود!
Task.WhenAll(timer.WaitForNextTickAsync(), timer.WaitForNextTickAsync());
```
اگر نیاز به چندین Consumer دارید، باید به ازای هر کدام یک نمونه جداگانه از `PeriodicTimer` بسازید.

### 16. Overlapping Work
اگر `DoWorkAsync` بیشتر از `Interval` طول بکشد، چه می‌شود؟
```text
Interval = 5s
Tick 1: شروع کار (مدت زمان 7 ثانیه)
Tick 2: در ثانیه 5 فرا می‌رسد، اما کار هنوز در حال اجراست.
```
در این حالت، به محض اینکه `DoWorkAsync` تمام شود، `WaitForNextTickAsync` **بلافاصله** مقدار `true` برمی‌گرداند (چون زمان تیک دوم قبلاً گذشته است). این باعث می‌شود کارها روی هم بیفتند (Overlap) و ممکن است به منابع فشار وارد شود.

**راهکارهای جلوگیری از Overlap**:
1. **استفاده از SemaphoreSlim**:
   ```csharp
   if (await _semaphore.WaitAsync(0)) // 0 یعنی اگر قفل است، منتظر نمان
   {
       try { await DoWorkAsync(); }
       finally { _semaphore.Release(); }
   }
   ```
2. **پرچم (Flag) در حال اجرا**: بررسی یک متغیر `bool` (با رعایت Thread-Safety مثل `Interlocked`) قبل از شروع کار.

> **خلاصه این بخش**: `PeriodicTimer` به‌طور ذاتی از اجرای همزمان چند `WaitForNextTickAsync` جلوگیری می‌کند (با پرتاب خطا). همچنین توسعه‌دهنده مسئول مدیریت Overlap کارها در صورت طولانی‌شدن زمان اجرا نسبت به Interval است.

---

## بخش 8: Exception Handling

### 17. Exception در عملیات Periodic
اگر `DoWorkAsync` خطا دهد و `try/catch` نداشته باشید:
```csharp
while (await timer.WaitForNextTickAsync())
{
    await DoWorkAsync(); // اگر اینجا Exception بدهد چه می‌شود؟
}
```
**اتفاق**: Exception به بالای حلقه Propagate می‌شود، حلقه شکسته می‌شود و `BackgroundService` کاملاً از کار می‌افتد (Crash می‌کند) و دیگر هیچ‌گاه اجرا نخواهد شد.

**راهکار صحیح**: `try/catch` باید **داخل** حلقه باشد.
```csharp
while (await timer.WaitForNextTickAsync(cancellationToken))
{
    try
    {
        await DoWorkAsync(cancellationToken);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "خطا در اجرای کار دوره‌ای");
        // تصمیم‌گیری: آیا باید Loop ادامه یابد یا خیر؟ معمولاً بله.
    }
}
```

> **خلاصه این بخش**: برای جلوگیری از توقف کامل سرویس پس‌زمینه، عملیات داخل حلقه `PeriodicTimer` باید همیشه در یک بلوک `try/catch` محصور شود.

---

## بخش 9: استفاده در ASP.NET Core

### 18. PeriodicTimer در ASP.NET Core
استفاده مستقیم از `PeriodicTimer` در یک **Controller** اشتباه است. درخواست‌های HTTP کوتاه‌مدت هستند و ایجاد یک حلقه بی‌نهایت در آن‌ها باعث Hang شدن Request و مصرف منابع می‌شود. عملیات دوره‌ای باید در **Hosted Service** یا **BackgroundService** اجرا شود.

### 19. PeriodicTimer در BackgroundService
```csharp
public class MyBackgroundService : BackgroundService
{
    private readonly ILogger<MyBackgroundService> _logger;

    public MyBackgroundService(ILogger<MyBackgroundService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("سرویس پس‌زمینه شروع به کار کرد.");
        
        // 1. ساخت تایمر
        using var timer = new PeriodicTimer(TimeSpan.FromSeconds(10));

        // 2. حلقه با پشتیبانی از Cancellation
        while (await timer.WaitForNextTickAsync(stoppingToken))
        {
            try
            {
                _logger.LogInformation("شروع اجرای کار...");
                await DoWorkAsync(stoppingToken);
            }
            catch (OperationCanceledException)
            {
                // این خط معمولاً اجرا نمی‌شود چون در شرط while مدیریت می‌شود،
                // اما برای اطمینان از خروج تمیز خوب است.
                break;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "خطا در DoWorkAsync");
            }
        }
        
        _logger.LogInformation("سرویس پس‌زمینه به‌طور ایمن متوقف شد.");
    }

    private async Task DoWorkAsync(CancellationToken token)
    {
        await Task.Delay(1000, token); // شبیه‌سازی کار
    }
}
```

### 20. PeriodicTimer و Dependency Injection
- **آیا خود Timer را Register کنیم؟**: خیر. `PeriodicTimer` یک شیء ساده است، آن را مستقیماً در متد `ExecuteAsync` بسازید (`new`).
- **Scoped Service در BackgroundService**: `BackgroundService` یک Singleton است. اگر نیاز به استفاده از سرویس‌های Scoped (مثل `DbContext`) دارید، باید در هر تکرار حلقه، یک Scope جدید بسازید:
  ```csharp
  while (await timer.WaitForNextTickAsync(stoppingToken))
  {
      using (var scope = _serviceProvider.CreateScope())
      {
          var dbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();
          await DoWorkWithDbAsync(dbContext, stoppingToken);
      }
  }
  ```

> **خلاصه این بخش**: در ASP.NET Core، `PeriodicTimer` باید درون `BackgroundService` استفاده شود. برای دسترسی به سرویس‌های Scoped، باید در هر تیک یک Scope جدید ایجاد کنید تا از خطاهای Lifetime جلوگیری شود.

---

## بخش 10: کاربردهای واقعی

### 21. Polling
برای بررسی دوره‌ای یک منبع بدون استفاده از Webhook.
- **API Polling**: بررسی وضعیت یک پرداخت هر 30 ثانیه.
- **Database Polling**: بررسی رکوردهای جدید با `IsProcessed = false`.
- **نکته**: Interval نباید خیلی کوتاه باشد (مثلاً زیر 5 ثانیه) تا به منبع خارجی فشار وارد نشود.

### 22. Cleanup و Maintenance
- **Cache Cleanup**: حذف آیتم‌های منقضی‌شده از حافظه (Interval: 5 دقیقه).
- **Temporary File Cleanup**: پاک کردن فایل‌های آپلودشده موقت قدیمی‌تر از 24 ساعت (Interval: 1 ساعت).
- **Log Maintenance**: فشرده‌سازی یا انتقال لاگ‌های قدیمی (Interval: روزانه).

> **خلاصه این بخش**: `PeriodicTimer` برای کارهای نگهداری (Maintenance) و نظرسنجی (Polling) ایده‌آل است، به شرطی که Interval منطقی انتخاب شود تا بار اضافی بر سیستم وارد نکند.

---

## بخش 11: Performance

### 23. Performance
- **Allocation**: بسیار پایین. تنها یک شیء `PeriodicTimer` ساخته می‌شود. در مقایسه، `Task.Delay` در هر تکرار یک `Task` جدید Allocates می‌کند.
- **CPU Usage**: در زمان انتظار، 0٪. فقط در لحظه Tick و اجرای کار، CPU مصرف می‌شود.
- **Frequency بالا**: برای فواصل زیر 10 میلی‌ثانیه، `PeriodicTimer` ممکن است به دلیل سربار ThreadPool و Scheduling، دقت زمانی (Timing Accuracy) خود را از دست بدهد. برای کارهای Real-time با تاخیر میکروثانیه، مناسب نیست.

### 24. Drift و Timing
آیا اجرا همیشه دقیق است؟ خیر.
```text
Interval = 5s
Tick 1: 00:00
Work: 3s (پایان در 00:03)
Tick 2: 00:05 (دقیق)

اما اگر Work = 6s باشد:
Tick 1: 00:00
Work: 6s (پایان در 00:06)
Tick 2: 00:06 (بلافاصله، چون زمان 00:05 گذشته است) -> Drift رخ داده است.
```
**عوامل Drift**:
1. مدت زمان اجرای `DoWorkAsync`.
2. اشباع ThreadPool (ThreadPool Saturation) و تاخیر در اختصاص Thread.
3. بار کلی سیستم (System Load) و Garbage Collection pauses.

> **خلاصه این بخش**: `PeriodicTimer` از نظر تخصیص حافظه بهینه است، اما تضمین‌کننده دقت زمانی مطلق (Hard Real-time) نیست و تحت تأثیر مدت زمان کار و شلوغی ThreadPool قرار می‌گیرد.

---

## بخش 12: مقایسه‌های مهم

### 25. PeriodicTimer vs System.Threading.Timer
| ویژگی | PeriodicTimer | System.Threading.Timer |
| :--- | :--- | :--- |
| Async/Await | عالی (طراحی شده برای آن) | ضعیف (نیاز به async void یا Fire-and-forget) |
| Cancellation | ذاتی و ساده | دستی و پیچیده |
| Dispose | ساده با `using` | نیاز به مدیریت دستی |
| Callback | خیر (Polling با `await`) | بله (Delegate) |
| Concurrency | تک‌مصرف‌کننده (ایمن) | خطر Reentrancy بالا |
| مناسب BackgroundService | **بله (انتخاب اول)** | خیر (منسوخ شده برای کارهای جدید) |

### 26. PeriodicTimer vs System.Timers.Timer
- **مدل**: `PeriodicTimer` از `async/await` استفاده می‌کند، در حالی که `Timers.Timer` مبتنی بر رویداد (`Elapsed`) است که مدیریت خطا و async در آن دشوار است.
- **Overlap**: در `Timers.Timer` اگر `AutoReset=true` باشد و کار طول بکشد، تیک‌های جدید روی هم انباشته می‌شوند (Reentrancy). در `PeriodicTimer` شما کنترل کامل دارید که آیا منتظر بمانید یا تیک را رد کنید.

### 27. PeriodicTimer vs Task.Delay
| ویژگی | PeriodicTimer | Task.Delay (در حلقه) |
| :--- | :--- | :--- |
| **نحوه زمان‌بندی** | Fixed Period (بر اساس زمان مطلق) | Relative Delay (تاخیر پس از اتمام کار قبلی) |
| **سربار حافظه** | بسیار کم (یک شیء) | متوسط (ساخت شیء Task در هر تکرار) |
| **مدیریت Dispose** | آسان (توقف فوری حلقه) | سخت‌تر (باید توکن را به Delay و Work پاس داد) |
| **کاربرد اصلی** | کارهای پس‌زمینه بلندمدت و منظم | تاخیرهای ساده و یک‌باره یا حلقه‌های ساده |

> **خلاصه این بخش**: `PeriodicTimer` در تقریباً تمام سناریوهای مدرن Background Processing بر رقبای قدیمی و `Task.Delay` ارجحیت دارد، مگر در موارد بسیار خاص Real-time یا تاخیرهای یک‌باره.

---

## بخش 13: اشتباهات رایج

### 28. Common Mistakes (15 مورد)
1. **فراموش کردن `CancellationToken`**: باعث می‌شود برنامه هنگام Shutdown به‌درستی متوقف نشود.
2. **عدم استفاده از `using`**: منجر به نشت منابع (Memory/Handle Leak) می‌شود.
3. **استفاده در Controller**: باعث Hang شدن درخواست HTTP می‌شود.
4. **قرار دادن `try/catch` خارج از حلقه**: با اولین خطا، کل سرویس پس‌زمینه برای همیشه متوقف می‌شود.
5. **نادیده گرفتن Overlapping Work**: اگر کار طول بکشد، تیک‌ها انباشته شده و به سیستم فشار می‌آورند.
6. **فراخوانی همزمان `WaitForNextTickAsync`**: باعث پرتاب `InvalidOperationException` می‌شود.
7. **استفاده از Interval بسیار کوتاه (مثلاً 1ms)**: ThreadPool را اشباع کرده و دقت زمانی را از بین می‌برد.
8. **اشتباه گرفتن Timer با Scheduler**: `PeriodicTimer` قابلیت Cron Expression (مثل "هر دوشنبه ساعت 2") را ندارد.
9. **فرض کردن Exactly-Once Execution**: در صورت Crash و Restart، ممکن است یک تیک از قلم بیفتد یا دوبار اجرا شود.
10. **عدم ایجاد Scope برای سرویس‌های Scoped**: باعث پرتاب خطای "Cannot consume scoped service from singleton" می‌شود.
11. **استفاده از `Task.Run` بدون نیاز**: `WaitForNextTickAsync` خودش غیرهمزمان است؛ نیازی به `Task.Run` برای فراخوانی آن نیست.
12. **نادیده گرفتن Shutdown در محیط‌های Distributed**: باعث می‌شود چندین Instance همزمان یک کار را انجام دهند.
13. **بلاک کردن Thread داخل حلقه**: استفاده از متدهای Sync (مثل `.Result` یا `.Wait()`) داخل حلقه Async.
14. **عدم لاگ کردن خطاها**: باعث می‌شود شکست‌های دوره‌ای نامرئی بمانند.
15. **انتظار Timing کاملاً دقیق**: نادیده گرفتن تاثیر Garbage Collection و سیستم‌عامل بر دقت میلی‌ثانیه‌ای.

> **خلاصه این بخش**: اجتناب از این 15 اشتباه رایج، پایداری و قابلیت اطمینان (Reliability) سرویس‌های پس‌زمینه شما را به‌طور چشمگیری افزایش می‌دهد.

---

## بخش 14: مباحث پیشرفته و پروژه واقعی

### 29. PeriodicTimer در معماری واقعی (Distributed)
در یک محیط Distributed با چندین Instance:
```text
Load Balancer
      ↓
Instance A → PeriodicTimer (هر 30 ثانیه اجرا می‌شود)
Instance B → PeriodicTimer (هر 30 ثانیه اجرا می‌شود)
Instance C → PeriodicTimer (هر 30 ثانیه اجرا می‌شود)
```
**مشکل**: کار مورد نظر 3 بار اجرا می‌شود! `PeriodicTimer` فقط فرآیند محلی (Local Process) را مدیریت می‌کند و از وجود سایر Instanceها بی‌خبر است.

**راهکارها**:
1. **Distributed Lock**: استفاده از Redis یا SQL Server برای اطمینان از اینکه فقط یک Instance در هر لحظه کار را انجام می‌دهد.
2. **Leader Election**: فقط Instanceای که "Leader" است تایمر را فعال کند.
3. **External Scheduler**: استفاده از ابزارهایی مثل **Hangfire**، **Quartz.NET** یا **Temporal** که ذاتاً برای اجرای توزیع‌شده (Distributed) طراحی شده‌اند.
*نکته*: `PeriodicTimer` جایگزین Schedulerهای Distributed نیست.

### 30. جمع‌بندی نهایی و طراحی یک Background Worker واقعی
سناریو: پردازش سفارش‌های در انتظار از دیتابیس هر 30 ثانیه.

```csharp
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

public class OrderProcessingBackgroundService : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<OrderProcessingBackgroundService> _logger;

    public OrderProcessingBackgroundService(
        IServiceProvider serviceProvider,
        ILogger<OrderProcessingBackgroundService> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("سرویس پردازش سفارشات شروع به کار کرد.");

        // 1. ساخت PeriodicTimer
        using var timer = new PeriodicTimer(TimeSpan.FromSeconds(30));

        // 2. حلقه اصلی با پشتیبانی از Cancellation
        while (await timer.WaitForNextTickAsync(stoppingToken))
        {
            // 3. ایجاد Scope برای دسترسی به سرویس‌های Scoped (مثل DbContext)
            using var scope = _serviceProvider.CreateScope();
            var dbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();

            try
            {
                _logger.LogInformation("شروع بررسی سفارش‌های جدید...");
                
                // 4. اجرای منطق کاری
                var pendingOrders = await dbContext.Orders
                    .Where(o => o.Status == OrderStatus.Pending)
                    .Take(100)
                    .ToListAsync(stoppingToken);

                foreach (var order in pendingOrders)
                {
                    // شبیه‌سازی پردازش
                    order.Status = OrderStatus.Processed;
                    _logger.LogInformation("سفارش {OrderId} پردازش شد.", order.Id);
                }

                await dbContext.SaveChangesAsync(stoppingToken);
                _logger.LogInformation("پردازش دسته‌ای با موفقیت انجام شد.");
            }
            catch (OperationCanceledException)
            {
                _logger.LogInformation("پردازش سفارشات به‌درستی متوقف شد.");
                break; // خروج از حلقه
            }
            catch (Exception ex)
            {
                // 5. مدیریت خطا برای جلوگیری از کرش کردن سرویس
                _logger.LogError(ex, "خطای غیرمنتظره در پردازش سفارشات. حلقه به کار خود ادامه می‌دهد.");
            }
        }

        _logger.LogInformation("سرویس پردازش سفارشات به‌طور کامل خاموش شد.");
    }
}
```
**توضیح اجزا**:
- `IServiceProvider`: برای ساخت Scope در هر تکرار.
- `using var timer`: مدیریت خودکار منابع.
- `while (await ...)`: الگوی استاندارد با `stoppingToken`.
- `try/catch` داخلی: تضمین می‌کند که یک خطای دیتابیس، کل چرخه حیات برنامه را متوقف نمی‌کند.

> **خلاصه این بخش**: در محیط‌های توزیع‌شده، `PeriodicTimer` به تنهایی کافی نیست و نیاز به مکانیزم‌های قفل توزیع‌شده دارد. کد ارائه‌شده، الگوی طلایی و ایمن برای پیاده‌سازی Workerها در .NET مدرن است.

---

## بخش‌های تکمیلی الزامی

### Cheat Sheet
| عملیات | کد نمونه |
| :--- | :--- |
| **ساخت تایمر** | `using var timer = new PeriodicTimer(TimeSpan.FromSeconds(5));` |
| **انتظار برای تیک** | `await timer.WaitForNextTickAsync(cancellationToken);` |
| **توقف ایمن** | پاس دادن `CancellationToken` به `WaitForNextTickAsync` |
| **جلوگیری از Overlap** | استفاده از `SemaphoreSlim.WaitAsync(0)` قبل از کار |
| **مدیریت خطا** | قرار دادن `try/catch` **داخل** حلقه `while` |
| **استفاده از Scoped DI** | `using var scope = _serviceProvider.CreateScope();` |

### Mental Model
`PeriodicTimer` را مانند یک **مترونوم (Metronome)** در موسیقی در نظر بگیرید. مترونوم به‌طور منظم ضربه می‌زند. شما (برنامه‌نویس) منتظر ضربه می‌مانید (`await`)، کار خود را انجام می‌دهید، و دوباره منتظر ضربه بعدی می‌شوید. اگر کار شما طول بکشد، ضربه بعدی را از دست می‌دهید یا بلافاصله پس از اتمام کار، صدای ضربه‌ای که قبلاً رخ داده را می‌شنوید (که باید آن را مدیریت کنید).

### Decision Guide
| سناریو | ابزار پیشنهادی |
| :--- | :--- |
| کار پس‌زمینه ساده و دوره‌ای در یک Instance | **`PeriodicTimer`** |
| نیاز به تاخیر ساده و یک‌باره در یک متد | **`Task.Delay`** |
| نیاز به زمان‌بندی پیچیده (Cron: "هر دوشنبه ساعت 2") | **Quartz.NET** یا **Cronos** |
| نیاز به تضمین اجرا، Retry، و داشبورد مدیریت | **Hangfire** |
| کارهای پس‌زمینه در محیط Distributed با حجم بالا | **Queue-based Processing** (RabbitMQ, Azure Service Bus) |
| کدبیس قدیمی (.NET Framework) | `System.Threading.Timer` |

### Interview Questions (20 سؤال)
1. **س**: `PeriodicTimer` در کدام نسخه .NET معرفی شد؟ **ج**: .NET 6.
2. **س**: تفاوت اصلی `PeriodicTimer` با `System.Threading.Timer` چیست؟ **ج**: پشتیبانی ذاتی و تمیز از `async/await` و الگوی Polling به جای Callback.
3. **س**: آیا `WaitForNextTickAsync` Thread را Block می‌کند؟ **ج**: خیر، آن را به ThreadPool بازمی‌گرداند.
4. **س**: اگر `PeriodicTimer` Dispose شود، `WaitForNextTickAsync` چه برمی‌گرداند؟ **ج**: مقدار `false` (یا `OperationCanceledException` اگر توکن کنسل شده باشد).
5. **س**: آیا می‌توان دو بار همزمان `WaitForNextTickAsync` را روی یک نمونه صدا زد؟ **ج**: خیر، `InvalidOperationException` پرتاب می‌کند.
6. **س**: چگونه از Overlapping Work در `PeriodicTimer` جلوگیری می‌کنید؟ **ج**: با استفاده از `SemaphoreSlim` یا یک پرچم (Flag) Thread-Safe.
7. **س**: چرا `try/catch` باید داخل حلقه `while` باشد؟ **ج**: تا از توقف کامل و کرش کردن `BackgroundService` جلوگیری شود.
8. **س**: چگونه یک سرویس Scoped را در `BackgroundService` استفاده می‌کنید؟ **ج**: با ایجاد `IServiceScope` در هر تکرار حلقه.
9. **س**: آیا `PeriodicTimer` دقت زمانی Hard Real-time دارد؟ **ج**: خیر، تحت تأثیر تاخیر ThreadPool و مدت زمان اجرای کار قرار می‌گیرد.
10. **س**: مقدار Interval در `PeriodicTimer` چه محدودیتی دارد؟ **ج**: باید بزرگتر از صفر و حداکثر `Int32.MaxValue` میلی‌ثانیه باشد.
11. **س**: تفاوت رفتار `PeriodicTimer` و `Task.Delay` در حلقه از نظر Drift چیست؟ **ج**: `Task.Delay` همیشه بعد از کار صبر می‌کند (Drift تجمعی)، اما `PeriodicTimer` سعی می‌کند به زمان‌بندی مطلق پایبند باشد.
12. **س**: آیا `PeriodicTimer` برای محیط‌های Distributed مناسب است؟ **ج**: به تنهایی خیر، زیرا هر Instance تایمر مستقل خود را دارد. نیاز به Distributed Lock است.
13. **س**: نقش `CancellationToken` در `WaitForNextTickAsync` چیست؟ **ج**: اجازه می‌دهد عملیات انتظار به‌صورت ایمن و فوری قطع شود.
14. **س**: چرا استفاده از `PeriodicTimer` در Controller اشتباه است؟ **ج**: زیرا درخواست‌های HTTP کوتاه‌مدت هستند و حلقه بی‌نهایت باعث Hang شدن Request می‌شود.
15. **س**: آیا `PeriodicTimer` یک Thread اختصاصی ایجاد می‌کند؟ **ج**: خیر، کاملاً بر پایه ThreadPool است.
16. **س**: چگونه می‌توان یک تیک را در صورت مشغول بودن سیستم Skip کرد؟ **ج**: با بررسی وضعیت (مثلاً `SemaphoreSlim.WaitAsync(0)`) و انجام ندادن کار در صورت عدم موفقیت.
17. **س**: آیا `PeriodicTimer` قابلیت تنظیم زمان شروع اولیه (Due Time) دارد؟ **ج**: خیر، بلافاصله پس از ساخت، شروع به شمارش می‌کند. (برخلاف `System.Threading.Timer`).
18. **س**: بهترین مکان برای Instantiate کردن `PeriodicTimer` کجاست؟ **ج**: داخل متد `ExecuteAsync` یک `BackgroundService`.
19. **س**: اگر `DoWorkAsync` استثنا پرتاب کند و Catch نشود، چه اتفاقی برای Host می‌افتد؟ **ج**: Host ممکن است سرویس را متوقف کرده و در برخی تنظیمات، کل برنامه را ری‌استارت کند.
20. **س**: چگونه `PeriodicTimer` را در Unit Test شبیه‌سازی (Mock) می‌کنید؟ **ج**: با تزریق `TimeProvider` (در .NET 8+) به سرویسی که منطق زمان‌بندی را مدیریت می‌کند، یا با استفاده از یک Interface Wrapper حول `PeriodicTimer`.

### Exercises (10 تمرین عملی)
1. **Beginner**: یک Console App بنویسید که هر 2 ثانیه ساعت فعلی را چاپ کند و با فشار دادن کلید Enter متوقف شود.
2. **Beginner**: کدی بنویسید که نشان دهد پس از `Dispose` کردن `PeriodicTimer`، متد `WaitForNextTickAsync` مقدار `false` برمی‌گرداند.
3. **Intermediate**: یک `BackgroundService` بسازید که هر 10 ثانیه یک فایل متنی ایجاد کرده و تاریخ در آن بنویسد. از `CancellationToken` برای توقف تمیز استفاده کنید.
4. **Intermediate**: سناریوی Overlap را شبیه‌سازی کنید: Interval را 2 ثانیه و `Task.Delay` داخل کار را 5 ثانیه تنظیم کنید. مشاهده کنید که چگونه تیک‌ها بلافاصله پشت سر هم اجرا می‌شوند.
5. **Intermediate**: تمرین 4 را با استفاده از `SemaphoreSlim` اصلاح کنید تا از اجرای همزمان جلوگیری شود.
6. **Advanced**: یک `PeriodicTimer` بسازید که در صورت وقوع Exception در `DoWorkAsync`، با استفاده از یک الگوی Simple Retry (مثلاً 3 بار تلاش) مجدداً کار را انجام دهد.
7. **Advanced**: یک Wrapper حول `PeriodicTimer` بسازید که قابلیت `Cron Expression` (با استفاده از کتابخانه‌ای مثل `NCrontab`) را شبیه‌سازی کند.
8. **Advanced**: سرویسی بنویسید که در هر تیک، یک Scope جدید بسازد، یک سرویس Scoped را Resolve کند، کار را انجام دهد و Scope را Dispose کند.
9. **Expert**: یک Benchmark با `BenchmarkDotNet` بنویسید که سربار حافظه (Allocation) `PeriodicTimer` را در 1000 تکرار با `Task.Delay` مقایسه کند.
10. **Expert**: یک مکانیزم "Skip if busy" پیاده‌سازی کنید: اگر تیک قبلی هنوز در حال اجراست، تیک جدید نادیده گرفته شود (بدون صف‌بندی).

### Real-World Examples
1. **API Polling**: بررسی وضعیت یک تراکنش بانکی هر 30 ثانیه تا زمانی که وضعیت به "موفق" تغییر کند.
2. **Cache Cleanup**: یک سرویس که هر 5 دقیقه، آیتم‌های `IMemoryCache` که منقضی شده‌اند اما به‌طور خودکار پاک نشده‌اند (Scavenging) را بررسی می‌کند.
3. **Database Maintenance**: پاک کردن رکوردهای لاگ قدیمی‌تر از 30 روز در دیتابیس، هر شب ساعت 2 بامداد (با ترکیب `PeriodicTimer` و محاسبه زمان باقی‌مانده تا ساعت 2).
4. **Background Processing**: خواندن پیام‌ها از یک Queue داخلی (مثل `Channel<T>`) هر 1 ثانیه و پردازش دسته‌ای آن‌ها.
5. **External Service Monitoring**: ارسال یک درخواست Ping به یک سرویس وابسته هر 1 دقیقه و ثبت وضعیت آن در سیستم مانیتورینگ (Health Check فعال).

### Common Pitfalls Checklist
- [ ] آیا `CancellationToken` را به `WaitForNextTickAsync` پاس داده‌ام؟
- [ ] آیا از `using` برای `PeriodicTimer` استفاده کرده‌ام؟
- [ ] آیا `try/catch` داخل حلقه `while` قرار دارد؟
- [ ] آیا از فراخوانی همزمان (Concurrent) `WaitForNextTickAsync` خودداری کرده‌ام؟
- [ ] آیا مکانیزمی برای جلوگیری از Overlap کارها در نظر گرفته‌ام؟
- [ ] آیا برای سرویس‌های Scoped از `CreateScope()` استفاده می‌کنم؟
- [ ] آیا از قرار دادن این منطق در Controller خودداری کرده‌ام؟

### منابع معتبر
1. **Microsoft Learn: PeriodicTimer Class**
   - سازمان: Microsoft
   - لینک: [https://learn.microsoft.com/en-us/dotnet/api/system.threading.periodictimer](https://learn.microsoft.com/en-us/dotnet/api/system.threading.periodictimer)
2. **Microsoft Learn: Background tasks with hosted services in ASP.NET Core**
   - سازمان: Microsoft
   - لینک: [https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services)
3. **.NET Blog: Introducing PeriodicTimer in .NET 6**
   - نویسنده: Stephen Toub (تیم .NET)
   - لینک: [https://devblogs.microsoft.com/dotnet/announcing-net-6/#system-threading-periodictimer](https://devblogs.microsoft.com/dotnet/announcing-net-6/)
4. **Microsoft Learn: TimeProvider in .NET 8**
   - سازمان: Microsoft
   - لینک: [https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider](https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider)

---
*این آموزش بر اساس مستندات رسمی .NET 8/9/10 تدوین شده و برای استفاده مستقیم به‌عنوان یک فایل Markdown در GitHub بهینه‌سازی شده است.*