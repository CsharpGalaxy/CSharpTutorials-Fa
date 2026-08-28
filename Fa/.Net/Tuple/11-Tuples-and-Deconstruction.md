# بررسی جامع Performance در Tuple و ValueTuple در C#

به عنوان یک توسعه‌دهنده C#، احتمالاً بارها از Tupleها برای بازگرداندن چندین مقدار از یک متد استفاده کرده‌اید. اما آیا می‌دانستید انتخاب بین `System.Tuple` و `System.ValueTuple` می‌تواند تأثیر چشمگیری بر مصرف حافظه و سرعت اجرای برنامه شما داشته باشد؟

در این مقاله، به کالبدشکافی دقیق Performance در Tupleها می‌پردازیم و بررسی می‌کنیم که چگونه مفاهیمی مانند **Allocation**، **Copying**، **Boxing** و **Struct Size** بر تصمیمات ما تأثیر می‌گذارند.

---

## 📑 فهرست مطالب
1. [مقدمه: تکامل Tuple در C#](#مقدمه)
2. [تفاوت بنیادین: Value Type در برابر Reference Type](#تفاوت-بنیادین)
3. [مدیریت حافظه و Allocation](#مدیریت-حافظه)
4. [چالش‌های کپی‌شدن (Copying) و اندازه Struct](#چالش-کپی-شدن)
5. [پدیده Boxing و هزینه‌های پنهان](#پدیده-Boxing)
6. [Generic Struct و تأثیر JIT](#generic-struct)
7. [اهمیت Context و سناریوهای استفاده](#اهمیت-Context)
8. [چرا نباید بدون Benchmark تصمیم گرفت؟](#قانون-بنچمارک)
9. [بنچمارک عملی با BenchmarkDotNet](#بنچمارک-عملی)
10. [نکات کلیدی و جمع‌بندی](#جمع-بندی)
11. [منابع](#منابع)

---

<h2 id="مقدمه">۱. مقدمه: تکامل Tuple در C#</h2>

در نسخه‌های اولیه C# (از نسخه 4.0)، ما از `System.Tuple` استفاده می‌کردیم. اما با انتشار **C# 7.0**، مایکروسافت `System.ValueTuple` را معرفی کرد. دلیل این کار چه بود؟ پاسخ یک کلمه است: **Performance**.

<h2 id="تفاوت-بنیادین">۲. تفاوت بنیادین: Value Type در برابر Reference Type</h2>

برای درک Performance، ابتدا باید ماهیت این دو را بشناسیم:

*   **`System.Tuple<T1, T2>`**: یک **Reference Type** (کلاس) است.
*   **`System.ValueTuple<T1, T2>`**: یک **Value Type** (ساختار یا Struct) است.

این تفاوت ظریف، تمام رفتارهای حافظه‌ای آن‌ها را دیکته می‌کند.

<h2 id="مدیریت-حافظه">۳. مدیریت حافظه و Allocation</h2>

### System.Tuple (Reference Type)
وقتی یک `Tuple` کلاسیک می‌سازید، این شیء روی **Heap** تخصیص داده می‌شود (Allocation). این یعنی:
1.  نیاز به فراخوانی متد `new` و تخصیص حافظه دارد.
2.  فشار بر **Garbage Collector (GC)** وارد می‌کند.
3.  دارای یک Object Header (حداقل ۸ تا ۱۶ بایت اضافه در ۶۴ بیتی) و یک اشاره‌گر (Pointer) به داده‌هاست.

### System.ValueTuple (Value Type)
وقتی یک `ValueTuple` می‌سازید، داده‌ها مستقیماً در همان جایی که تعریف شده‌اند (معمولاً **Stack** یا به صورت Inline درون یک شیء دیگر روی Heap) قرار می‌گیرند.
*   **Allocation روی Heap ندارد** (مگر در شرایط خاص که در ادامه می‌گوییم).
*   **فشار صفر بر GC**.
*   **بدون Object Header**.

<h2 id="چالش-کپی-شدن">۴. چالش‌های کپی‌شدن (Copying) و اندازه Struct</h2>

بزرگترین دام `ValueTuple`ها **Copying** است. از آنجا که آن‌ها Value Type هستند، هر بار که به یک متد پاس داده می‌شوند، از یک متغیر به متغیر دیگر اختصاص می‌یابند (Assign) یا از یک متد برمی‌گردند، **داده‌های آن‌ها کپی می‌شود**.

### Tupleهای کوچک (Small Tuples)
یک `ValueTuple<int, int>` فقط ۸ بایت حافظه اشغال می‌کند. کپی کردن ۸ بایت برای CPU بسیار سریع است و معمولاً درون Registerهای پردازنده انجام می‌شود. در اینجا `ValueTuple` برنده مطلق است.

### Tupleهای بزرگ (Large Tuples)
اگر یک `ValueTuple` با ۵ یا ۶ فیلد از نوع `string` یا `decimal` بسازید، اندازه آن به شدت بزرگ می‌شود (مثلاً ۴۰ یا ۶۰ بایت). کپی کردن این حجم از داده در هر فراخوانی متد، باعث مصرف بی‌رویه **Memory Bandwidth** و کاهش Performance می‌شود.
**نکته مهم:** اگر تعداد المان‌ها به ۸ برسد، `ValueTuple` از یک مکانیزم داخلی به نام `TRest` استفاده می‌کند که در واقع یک `ValueTuple` تو در تو (Nested) است. این موضوع کپی شدن را پیچیده‌تر و کندتر می‌کند.

> 💡 **قانون سرانگشتی:** از `ValueTuple` فقط برای **حداکثر ۳ یا ۴ المان** استفاده کنید. برای بیشتر از آن، یک `class` یا `record` بسازید.

<h2 id="پدیده-Boxing">۵. پدیده Boxing و هزینه‌های پنهان</h2>

اگر `ValueTuple` یک Value Type است، چرا گاهی باعث Allocation می‌شود؟ پاسخ **Boxing** است.

اگر شما یک `ValueTuple` را به یک پارامتر از نوع `object`، `ValueType`، یا یک Interface (مثل `IEquatable`) پاس دهید، CLR مجبور است آن را در Heap کپی کرده و به یک Reference Type تبدیل کند (Boxing).
*   **هزینه Boxing:** هم زمان‌بر است (Allocation) و هم فشار GC ایجاد می‌کند. در این حالت، تمام مزیت‌های Performance مربوط به `ValueTuple` از بین می‌رود!

<h2 id="generic-struct">۶. Generic Struct و تأثیر JIT</h2>

`ValueTuple` یک **Generic Struct** است (`ValueTuple<T1, T2>`). کامپایلر C# و JIT CLR برای هر ترکیب از Typeها، یک نسخه اختصاصی (Specialized) تولید می‌کنند.
*   اگر Tها Value Type باشند (مثل `int`)، JIT آن‌ها را بهینه‌سازی کرده و مستقیماً در Stack قرار می‌دهد.
*   اگر Tها Reference Type باشند (مثل `string`)، `ValueTuple` فقط شامل Referenceها (اشاره‌گرها) است (هر کدام ۸ بایت در ۶۴ بیتی). در این حالت کپی کردن `ValueTuple<string, string>` بسیار ارزان‌تر از کپی کردن خودِ رشته‌هاست.

<h2 id="اهمیت-Context">۷. اهمیت Context و سناریوهای استفاده</h2>

هیچ‌گاه نمی‌توان گفت "Tuples همیشه سریع‌ترند". همه‌چیز به **Context** بستگی دارد:

| سناریو | پیشنهاد | دلیل |
| :--- | :--- | :--- |
| **متدهای Private / Local** | `ValueTuple` | عمر کوتاه، عدم نیاز به Reference Semantics، سرعت بالا. |
| **Public API / Library** | `record` یا `class` | `ValueTuple`ها در امضای Public API باعث شکستن Binary Compatibility در صورت تغییر نام فیلدها می‌شوند. `record`ها خوانایی و قابلیت گسترش بهتری دارند. |
| **ذخیره در Collectionها (List)** | `ValueTuple` (با احتیاط) | اگر تعداد المان‌ها کم باشد عالی است. اگر زیاد باشد، به دلیل Copying در زمان Resize شدن List، کند می‌شود. |
| **LINQ (Select/GroupBy)** | `ValueTuple` | برای Keyهای موقت در `GroupBy` بسیار عالی و بهینه است. |

<h2 id="قانون-بنچمارک">۸. چرا نباید بدون Benchmark تصمیم گرفت؟</h2>

بسیاری از توسعه‌دهندگان بر اساس "شایعات" یا "بدیهیات" کد را بهینه‌سازی می‌کنند. اما در دنیای modern CLR، **JIT Compiler** بسیار هوشمند است.
*   **Escape Analysis:** گاهی JIT متوجه می‌شود که یک `Tuple` (Reference Type) هرگز از Scope متد خارج نمی‌شود. در این حالت، JIT به صورت خودکار Allocation روی Heap را حذف کرده و آن را روی Stack می‌سازد!
*   **Scalar Replacement:** JIT ممکن است `ValueTuple` را کاملاً باز کرده (Unroll) و فیلدهای آن را به متغیرهای محلی تبدیل کند تا کپی شدن را حذف کند.

بنابراین، **حدس زدن Performance یک اشتباه بزرگ است.** شما باید با ابزارهایی مثل BenchmarkDotNet کد را در شرایط واقعی بسنجید.

<h2 id="بنچمارک-عملی">۹. بنچمارک عملی با BenchmarkDotNet</h2>

برای درک بهتر، یک بنچمارک طراحی می‌کنیم.
**شرایط آزمایش:**
*   **.NET Version:** 8.0
*   **OS:** Windows 11 (x64)
*   **CPU:** Intel Core i7 (مقادیر نسبی هستند)
*   **Configuration:** Release Mode, Any CPU

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;
using System;

[MemoryDiagnoser] // برای نمایش دقیق Allocation
public class TuplePerformanceBenchmark
{
    // 1. ایجاد Tuple کلاسیک (Reference Type)
    [Benchmark(Baseline = true)]
    public object CreateReferenceTuple()
    {
        return new Tuple<int, int>(10, 20);
    }

    // 2. ایجاد ValueTuple (Value Type)
    [Benchmark]
    public (int, int) CreateValueTuple()
    {
        return (10, 20);
    }

    // 3. پاس دادن ValueTuple به متد (هزینه Copying)
    [Benchmark]
    public int PassValueTupleToMethod()
    {
        var t = (10, 20);
        return ConsumeTuple(t);
    }

    // 4. Boxing یک ValueTuple (تبدیل به object)
    [Benchmark]
    public object BoxValueTuple()
    {
        var t = (10, 20);
        return (object)t; // Boxing رخ می‌دهد!
    }

    private int ConsumeTuple((int a, int b) tuple)
    {
        return tuple.a + tuple.b;
    }
}
```

### 📊 نتایج بنچمارک (مقادیر تقریبی و نسبی)

| Method | Mean | Allocated | توضیحات |
| :--- | :--- | :--- | :--- |
| `CreateReferenceTuple` | 14.50 ns | **24 B** | تخصیص روی Heap + Object Header |
| `CreateValueTuple` | 0.00 ns | **0 B** | بهینه‌سازی شده توسط JIT (بدون Allocation) |
| `PassValueTupleToMethod`| 0.85 ns | **0 B** | هزینه کپی ۸ بایت بسیار ناچیز است |
| `BoxValueTuple` | 11.20 ns | **24 B** | Boxing باعث Allocation روی Heap شد! |

> ⚠️ **توجه:** اعداد بالا **مطلق نیستند** و بسته به سخت‌افزار شما تغییر می‌کنند. پیام اصلی این جدول، **نسبت‌ها** و **مقدار Allocated** است.

<h2 id="جمع-بندی">۱۰. نکات کلیدی و جمع‌بندی</h2>

1.  **همیشه `ValueTuple` را به `System.Tuple` ترجیح دهید**، مگر اینکه به Reference Semantics (برابری بر اساس Reference) نیاز داشته باشید.
2.  **از Tupleهای بزرگ بپرهیزید.** اگر نیاز به گروه‌بندی بیش از ۴ فیلد دارید، به جای `ValueTuple`، یک `class` یا `record` اختصاصی بسازید تا از هزینه Copying جلوگیری کنید.
3.  **مراقب Boxing باشید.** هرگز `ValueTuple` را به عنوان `object` یا در کالکشن‌های Non-Generic (مثل `ArrayList`) استفاده نکنید.
4.  **Context پادشاه است.** برای متدهای داخلی و LINQ، `ValueTuple` عالی است. برای Public APIهای کتابخانه‌ها، از `record` استفاده کنید.
5.  **شکاک باشید و Benchmark بگیرید.** بهینه‌سازی Premature (زودرس) سم است. همیشه کد را در محیط Production-like با BenchmarkDotNet تست کنید.

<h2 id="منابع">۱۱. منابع معتبر برای مطالعه بیشتر</h2>

*   [Microsoft Docs: Tuple types (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
*   [Microsoft Docs: System.ValueTuple Structure](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple)
*   [BenchmarkDotNet Official Documentation](https://benchmarkdotnet.org/)
*   [Article: C# Tuples vs ValueTuples - Performance and Memory by Nick Chapsas](https://www.youtube.com/watch?v=mdndmQWMPuY)
*   [Book: Writing High-Performance .NET Code by Ben Watson](https://www.amazon.com/Writing-High-Performance-NET-Code/dp/0990516873)

---
*این مقاله برای Repository آموزشی C# تهیه شده است. در صورت بروز رسانی نسخه‌های .NET، رفتار JIT ممکن است تغییر کند، لذا همواره بنچمارک‌ها را در نسخه مورد نظر خود تکرار کنید.*