# کالبدشکافی Async/Await در سی‌شارپ: از State Machine تا ValueTuple

به مخزن آموزشی سی‌شارپ خوش آمدید. در این مقاله، یکی از جذاب‌ترین و در عین حال پیچیده‌ترین مباحث سی‌شارپ، یعنی **مکانیزم داخلی Async/Await** و نحوه تعامل آن با **ValueTuple** را بررسی می‌کنیم. هدف ما عبور از سطح ظاهری و درک اتفاقاتی است که در سطح Compiler و Runtime رخ می‌دهد.

---

## 📑 فهرست مطالب
1. [Async/Await به زبان ساده](#1-asyncawait-به-زبان-ساده)
2. [Async State Machine چیست؟](#2-async-state-machine-چیست)
3. [Compiler چه جادویی انجام می‌دهد؟](#3-compiler-چه-جادویی-انجام-می‌دهد)
4. [محل نگهداری State (متغیرهای محلی)](#4-محل-نگهداری-state-متغیرهای-محلی)
5. [تفاوت اجرای کد قبل و بعد از await](#5-تفاوت-اجرای-کد-قبل-و-بعد-از-await)
6. [ValueTuple در متدهای Async](#6-valuetuple-در-متدهای-async)
7. [مفهوم Boxing در سناریوهای Async](#7-مفهوم-boxing-در-سناریوهای-async)
8. [مثال عملی و بررسی خروجی](#8-مثال-عملی-و-بررسی-خروجی)
9. [نکات Performance و بهینه‌سازی](#9-نکات-performance-و-بهینهسازی)
10. [اشتباهات رایج](#10-اشتباهات-رایج)
11. [جمع‌بندی](#11-جمع‌بندی)
12. [منابع معتبر](#12-منابع-معتبر)

---

## 1. Async/Await به زبان ساده

فرض کنید در حال آشپزی هستید و می‌خواهید ماکارونی بپزید. آب را روی گاز می‌گذارید.
* **حالت Sync (همگام):** شما خیره به قابلمه آب نگاه می‌کنید تا بجوشد. در این مدت هیچ کار دیگری نمی‌کنید (Thread مسدود یا Block می‌شود).
* **حالت Async (ناهمگام):** آب را روی گاز می‌گذارید و به کارهای دیگر (مثل خرد کردن سبزیجات) می‌پردازید. هر از گاهی چک می‌کنید که آب جوشیده است یا نه. وقتی جوشید، ماکارونی را داخل آن می‌ریزید.

در سی‌شارپ، `await` یعنی: *"Thread را آزاد کن تا کار دیگری انجام دهد. وقتی این عملیات طولانی (مثل I/O یا Network) تمام شد، بقیه کد من را ادامه بده."*

---

## 2. Async State Machine چیست؟

کلمه کلیدی `async` و `await` هیچ رفتار جادویی در Runtime ندارند. آن‌ها صرفاً دستوراتی برای **Compiler** هستند.
وقتی شما یک متد را `async` می‌کنید، Compiler کل بدنه متد شما را به یک **ساختار (Struct)** تبدیل می‌کند که اینترفیس `IAsyncStateMachine` را پیاده‌سازی کرده است. به این ساختار، **Async State Machine** می‌گویند.

این ماشین حالت، وظیفه دارد متد شما را به چندین "تکه" (Continuation) تقسیم کند و بر اساس وضعیت فعلی (State)، بداند که باید کدام تکه را در ادامه اجرا کند.

---

## 3. Compiler چه جادویی انجام می‌دهد؟

وقتی Compiler به کلمه `async` برمی‌خورد، مراحل زیر را طی می‌کند:
1. یک Struct مخفی می‌سازد که `IAsyncStateMachine` را پیاده‌سازی می‌کند.
2. تمام متغیرهای محلی (Local Variables) متد شما را به عنوان **Field** درون این Struct قرار می‌دهد.
3. یک `AsyncMethodBuilder` (مثل `AsyncTaskMethodBuilder<T>`) ایجاد می‌کند تا `Task` خروجی را مدیریت کند.
4. متد اصلی شما را به یک کد بسیار ساده تبدیل می‌کند که فقط `Builder.Start()` را صدا می‌زند.

> **منبع:** [C# Language Specification - Async Functions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/async-functions)

---

## 4. محل نگهداری State (متغیرهای محلی)

یکی از مهم‌ترین سوالات این است: *"وقتی Thread در خط `await` متد را ترک می‌کند، متغیرهای محلی من کجا می‌روند؟"*

* **قبل از Suspend (تعلیق):** متغیرها روی **Stack** (درون Struct ماشین حالت) قرار دارند.
* **بعد از Suspend:** چون Stack Frame فعلی از بین می‌رود، Compiler ماشین حالت (که یک Struct است) را به **Heap** منتقل می‌کند (Boxing می‌کند) تا متغیرها زنده بمانند.
* **هنگام Resume (ادامه):** Runtime ماشین حالت را از Heap می‌خواند، State را به‌روزرسانی می‌کند و متد `MoveNext()` را برای اجرای ادامه کد فراخوانی می‌کند.

---

## 5. تفاوت اجرای کد قبل و بعد از await

* **قبل از اولین await:** کد شما به‌صورت **Sync** و روی Thread فراخوانی‌کننده (Caller) اجرا می‌شود. این بخش بسیار سریع است (Fast Path).
* **بعد از await:** کد شما به عنوان یک **Continuation** اجرا می‌شود.
  * اگر در محیط UI (مثل WinForms/WPF) باشید، روی Thread اصلی (UI Thread) رزومه می‌شود.
  * اگر در محیط Console یا ASP.NET Core باشید (یا از `ConfigureAwait(false)` استفاده کنید)، روی یکی از Threadهای **ThreadPool** ادامه می‌یابد.

---

## 6. ValueTuple در متدهای Async

از سی‌شارپ 7.0، `ValueTuple` (که یک Struct است) معرفی شد. استفاده از آن در متدهای Async بسیار کارآمد است.

```csharp
public async Task<(int Id, string Name)> GetUserAsync()
{
    await Task.Delay(100);
    return (1, "Ali");
}
```

**چرا این عالی است؟**
چون `ValueTuple` یک Struct است، وقتی آن را در `Task<T>` برمی‌گردانید، **هیچ Boxing برای خود Tuple رخ نمی‌دهد**. فقط یک آبجکت `Task` روی Heap ساخته می‌شود که داده‌های Tuple را درون خود (یا در State Machine) نگه می‌دارد. این در مقایسه با ساخت یک `Class` سفارشی برای بازگشت چند مقدار، Allocation بسیار کمتری دارد.

---

## 7. مفهوم Boxing در سناریوهای Async

Boxing (تبدیل Value Type به Reference Type) در Async چند کاربرد و هزینه دارد:

1. **Boxing خود State Machine:** همان‌طور که در بخش ۴ گفتیم، اگر متد شما suspend شود (یعنی `await` روی یک Task ناتمام قرار گیرد)، Struct ماشین حالت برای ماندگاری روی Heap **Boxing** می‌شود. این یعنی **یک Allocation** در ازای هر بار Suspend.
2. **Boxing ValueTuple:** اگر شما `ValueTuple` را به `object` یا یک Interface (مثل `ITuple`) کست کنید، Boxing رخ می‌دهد. اما در بازگشت طبیعی `Task<(T1, T2)>` این اتفاق نمی‌افتد.
3. **آرایه‌های State:** در برخی پیاده‌سازی‌های قدیمی‌تر یا سناریوهای خاص (مثل `async void` یا حلقه‌های پیچیده)، ممکن است Compiler برای ذخیره Stateها از آرایه `object[]` استفاده کند که باعث Boxing متغیرهای Value Type می‌شود.

> **نکته مهم:** در .NET Core 2.1 به بعد، بهینه‌سازی‌هایی انجام شد تا در صورت امکان از Boxing State Machine جلوگیری شود (مثلاً با استفاده از `IValueTaskSource`)، اما در `Task`های استاندارد، Suspend همیشه یک Allocation دارد.

---

## 8. مثال عملی و بررسی خروجی

کد زیر را در نظر بگیرید:

```csharp
public async Task<(int Sum, string Message)> CalculateAsync(int a, int b)
{
    int localVariable = a + b; // State 0
    
    await Task.Delay(100); // نقطه Suspend
    
    string message = $"Sum is {localVariable}"; // State 1
    return (localVariable, message);
}
```

**Compiler این کد را به چیزی شبیه به این تبدیل می‌کند (به زبان ساده‌شده):**

```csharp
[CompilerGenerated]
private struct <CalculateAsync>d__0 : IAsyncStateMachine
{
    public int <>1__state;
    public AsyncTaskMethodBuilder<(int, string)> <>t__builder;
    public int a;
    public int b;
    private int <localVariable>5__1;
    private TaskAwaiter <>u__1;

    public void MoveNext()
    {
        int num = <>1__state;
        try
        {
            TaskAwaiter awaiter;
            if (num != 0)
            {
                // State 0: قبل از await
                <localVariable>5__1 = a + b;
                awaiter = Task.Delay(100).GetAwaiter();
                if (!awaiter.IsCompleted)
                {
                    <>1__state = 0;
                    <>u__1 = awaiter;
                    // اینجا State Machine Boxing شده و روی Heap می‌رود
                    <>t__builder.AwaitUnsafeOnCompleted(ref awaiter, ref this);
                    return;
                }
            }
            else
            {
                // State 1: بعد از await (Resume)
                awaiter = <>u__1;
                <>u__1 = default;
                <>1__state = -1;
            }
            awaiter.GetResult();
            
            string message = $"Sum is {<localVariable>5__1}";
            <>t__builder.SetResult((<localVariable>5__1, message));
        }
        catch (Exception ex)
        {
            <>t__builder.SetException(ex);
        }
    }
    // ... SetStateMachine implementation
}
```

---

## 9. نکات Performance و بهینه‌سازی

1. **استفاده از `ValueTask`:** اگر متد شما در بیشتر مواقع به‌صورت Sync کامل می‌شود (مثلاً داده از Cache خوانده می‌شود)، به جای `Task<T>` از `ValueTask<T>` استفاده کنید. `ValueTask` یک Struct است و از ساخت Task و Boxing State Machine در مسیرهای سریع (Fast Path) جلوگیری می‌کند.
   * *منبع:* [Stephen Toub - Understanding the Whys, Whats, and Whens of ValueTask](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/)
2. **`ConfigureAwait(false)`:** در کتابخانه‌ها (Library Code) حتماً از این متد استفاده کنید تا از تلاش Runtime برای بازگشت به SynchronizationContext جلوگیری شود. این کار Performance را بهبود داده و از Deadlock جلوگیری می‌کند.
3. **کوچک نگه داشتن State Machine:** هرچه متغیرهای محلی (Local Variables) کمتری قبل از `await` تعریف کنید، اندازه Struct ماشین حالت کوچک‌تر بوده و کپی/Boxing آن سریع‌تر است.
4. **استفاده از ValueTuple:** برای بازگشت چند مقدار، به جای ساخت Class، از `ValueTuple` استفاده کنید تا از Allocation اضافی روی Heap جلوگیری شود.

---

## 10. اشتباهات رایج

* ❌ **استفاده از `async void`:** به جز برای Event Handlerها، هرگز از `async void` استفاده نکنید. در این حالت Exceptionها قابل catch نیستند و ماشین حالت رفتار متفاوتی دارد. همیشه `async Task` برگردانید.
* ❌ **Block کردن روی Async:** استفاده از `.Result` یا `.Wait()` روی Taskها. این کار Thread را مسدود کرده و در محیط‌های دارای SynchronizationContext (مثل UI) باعث **Deadlock** می‌شود.
* ❌ **استفاده نابجا از `ValueTask`:** یک `ValueTask` را نمی‌توانید چند بار `await` کنید یا همزمان روی آن `await` و `.Result` بگیرید. اگر نیاز به این کارها دارید، باید آن را به `Task` تبدیل کنید (`.AsTask()`).

---

## 11. جمع‌بندی

کلمات `async` و `await` در سی‌شارپ صرفاً یک لطف از سمت Compiler هستند تا کدنویسی عملیات ناهمگام را برای ما ساده کنند. در پشت پرده، Compiler متد شما را به یک **State Machine** تبدیل می‌کند.
استفاده از **ValueTuple** در این متدها، به دلیل ماهیت Struct بودن آن، راهکاری فوق‌العاده برای بازگشت چند مقدار بدون ایجاد Allocation اضافی است. با این حال، باید مراقب **Boxing** خود State Machine در هنگام Suspend باشید و در سناریوهای با Performance بالا، از **ValueTask** استفاده کنید.

---

## 12. منابع معتبر

برای مطالعه عمیق‌تر، منابع زیر به‌شدت پیشنهاد می‌شوند:

1. **مستندات مایکروسافت:** [C# Language Specification - Async Functions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/async-functions)
2. **مقاله مرجع Stephen Toub:** [Understanding the Whys, Whats, and Whens of ValueTask](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/) (بسیار مهم برای درک Performance)
3. **سورس کاد .NET Runtime:** بررسی کلاس [`AsyncTaskMethodBuilder`](https://source.dot.net/System.Runtime.CompilerServices/AsyncTaskMethodBuilder.cs.html) و [`IAsyncStateMachine`](https://source.dot.net/System.Runtime.CompilerServices/IAsyncStateMachine.cs.html)
4. **کتاب C# in Depth (نوشته Jon Skeet):** فصل مربوط به Async/Await برای درک مفاهیم پایه‌ای و Continuationها.
5. **مقالات Stephen Cleary:** [There Is No Thread](https://blog.stephencleary.com/2013/11/there-is-no-thread.html) (برای درک اینکه Async به معنای استفاده از Thread اضافی نیست).

---
*اگر این مقاله برای شما مفید بود، لطفاً به Repository ما Star بدهید و برای توسعه‌دهندگان دیگر ارسال کنید. نظرات و Pull Requestهای شما برای بهبود این محتوای آموزشی باعث افتخار ماست.*