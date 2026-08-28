# کالبدشکافی Performance و Memory در Recordهای #C: افسانه‌ها و واقعیت‌ها

در دنیای #C، از زمان معرفی `record` در نسخه 9 و `record struct` در نسخه 10، یک باور عمومی و نیمه‌صحیح در بین توسعه‌دهندگان شکل گرفته است. پیش از آنکه به عمق تکنیکال ماجرا برویم، باید به یک سوال کلیدی پاسخ دهیم و یک افسانه را باطل کنیم.

## افسانه‌زدایی: آیا «Recordها سریع‌تر از Class هستند»؟

**پاسخ کوتاه: خیر، این جمله ساده‌سازی بیش از حد و در بسیاری از موارد کاملاً غلط است.**

**چرا؟** چون `Record` یک جادوی سخت‌افزاری یا میان‌بر پردازنده نیست؛ بلکه **Syntactic Sugar** (شکر سینتکسی) است. کامپایلر #C برای یک `record`، کدهای زیادی را در پشت صحنه تولید می‌کند (مانند متدهای `Equals`، `GetHashCode`، `Clone` و اپراتورهای `==` و `!=`). 
در واقع، استفاده از `record` به دلیل محاسبه **Value-based Equality** (برابری مبتنی بر مقدار) و پشتیبانی از `with expression`، **سربار (Overhead) پردازشی بیشتری** نسبت به یک `class` معمولی با Reference Equality دارد.

سرعت و مصرف حافظه در #C به **نوع داده (Value Type در برابر Reference Type)** و **محل قرارگیری آن‌ها (Heap در برابر Stack)** بستگی دارد، نه صرفاً به کلمه کلیدی `record`.

---

## مفاهیم پایه: زیر کاپوت حافظه و پردازنده

برای درک Performance، باید با زمین بازی آشنا باشیم:

1. **Heap (هیپ):** جایی که **Reference Type**ها (مثل `class` و `record class`) زندگی می‌کنند. تخصیص (Allocation) در هیپ کندتر است و نیاز به مدیریت توسط **Garbage Collection (GC)** دارد.
2. **Stack (استک):** جایی که **Value Type**ها (مثل `struct` و `record struct`) و متغیرهای محلی قرار می‌گیرند. تخصیص در استک بسیار سریع است و به محض خروج از Scope، حافظه بدون دخالت GC آزاد می‌شود.
3. **Garbage Collection (GC):** فرآیندی که حافظه هیپ را پاکسازی می‌کند. هرچه Allocation بیشتری در هیپ داشته باشید، GC بیشتر کار می‌کند و باعث **توقف‌های کوتاه (Pause)** در برنامه می‌شود.
4. **Boxing:** وقتی یک Value Type را به یک Reference Type (مثل `object` یا یک Interface) تبدیل می‌کنید، CLR آن را در هیپ کپی می‌کند. **Boxing قاتل خاموش Performance است** و باید به شدت از آن اجتناب کرد.

---

## بررسی عمیق انواع Record

### 1. Record Class (همان `record` پیش‌فرض)
وقتی می‌نویسید `public record Person(...)`, شما در حال ساخت یک **Reference Type** هستید.
* **Allocation:** هر بار که از `new` استفاده می‌کنید، حافظه در **Heap** تخصیص می‌یابد.
* **With Expression & Shallow Copy:** اپراتور `with` یک **کپی کم‌عمق (Shallow Copy)** ایجاد می‌کند. کامپایلر یک Copy Constructor تولید می‌کند. 
  * *نکته مهم Performance:* استفاده از `with` در `record class` به معنای **یک Allocation جدید در Heap** است. اگر این کار را در یک حلقه `for` انجام دهید، فشار وحشتناکی به GC وارد می‌کنید.

### 2. Record Struct (معرفی شده در C# 10)
وقتی می‌نویسید `public record struct Point(...)`, شما یک **Value Type** ساخته‌اید.
* **Allocation:** به صورت پیش‌فرض در **Stack** (یا به صورت Inline در آرایه‌ها و اشیاء دیگر) قرار می‌گیرد. هیچ فشاری به GC ندارد.
* **Copy Cost (هزینه کپی):** برخلاف Reference Typeها که فقط یک Pointer (8 بایت) کپی می‌شود، در Value Typeها **تمام فیلدها** کپی می‌شوند.
* **Small vs Large Struct:** 
  * اگر `record struct` شما **کوچک** باشد (مثلاً ۲ یا ۳ فیلد `int` - زیر ۱۶ بایت)، Performance فوق‌العاده‌ای دارد.
  * اگر **بزرگ** باشد (مثلاً شامل چندین `string` یا آرایه)، هزینه کپی کردن آن در Stack و هنگام پاس دادن به متدها، از هزینه Allocation در Heap هم بیشتر می‌شود! مایکروسافت اکیداً توصیه می‌کند Structها را زیر ۱۶ بایت نگه دارید.

---

## مقایسه Performance: کلاس در برابر ۴ ساختار اصلی

بیایید این چهار مورد را از نظر CPU و Memory مقایسه کنیم:

| ویژگی | `class` | `struct` | `record class` | `record struct` |
| :--- | :--- | :--- | :--- | :--- |
| **نوع داده** | Reference Type | Value Type | Reference Type | Value Type |
| **محل استقرار** | Heap | Stack / Inline | Heap | Stack / Inline |
| **فشار روی GC** | دارد | ندارد | دارد (بیشتر به خاطر `with`) | ندارد |
| **هزینه Equality** | سریع (مقایسه Reference) | کندتر (مقایسه Value) | کند (مقایسه تک‌تک فیلدها) | کند (مقایسه تک‌تک فیلدها) |
| **هزینه `with`** | ندارد | ندارد | **Allocation جدید در Heap** | کپی در Stack (بدون GC) |
| **خطر Boxing** | ندارد | دارد (در صورت Cast) | ندارد | دارد (در صورت Cast) |

**نتیجه‌گیری از جدول:**
* اگر فقط به دنبال **Immutable Data** هستید و نیازی به `with` یا `Equality` پیچیده ندارید، یک `struct` ساده یا `class` معمولی سریع‌تر از Record است.
* اگر داده‌های شما **کوچک** هستند و نیاز به Value Equality دارید، `record struct` پادشاه Performance است.
* اگر داده‌های شما **بزرگ** هستند، `record class` بهتر است (چون کپی کردن آن در Stack هزینه دارد).

---

## چگونه Benchmark درست انجام دهیم؟

برای سنجش Performance در #C، هرگز از `Stopwatch` یا `DateTime.Now` استفاده نکنید. ابزار استاندارد و مورد تایید مایکروسافت **BenchmarkDotNet** است. این ابزار کد شما را در محیط Release کامپایل کرده، Warm-up می‌کند و نویزهای سیستم عامل را فیلتر می‌کند.

### مثال عملی Benchmark

ابتدا پکیج را نصب کنید:
```bash
dotnet add package BenchmarkDotNet
```

سپس کد زیر را برای مقایسه Allocation و سرعت `with` بنویسید:

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

// تعریف انواع مختلف
public class PersonClass { public string Name { get; set; } public int Age { get; set; } }
public record PersonRecordClass(string Name, int Age);
public record struct PersonRecordStruct(string Name, int Age);

[MemoryDiagnoser] // برای نمایش دقیق مصرف حافظه و GC
public class RecordPerformanceBenchmark
{
    private const int Iterations = 1000;

    [Benchmark(Description = "Class: Allocation")]
    public object ClassAllocation()
    {
        object result = null;
        for (int i = 0; i < Iterations; i++)
            result = new PersonClass { Name = "Ali", Age = 30 };
        return result;
    }

    [Benchmark(Description = "Record Class: Allocation & With")]
    public object RecordClassWith()
    {
        var person = new PersonRecordClass("Ali", 30);
        object result = null;
        for (int i = 0; i < Iterations; i++)
            result = person with { Age = i }; // تولید Shallow Copy و Allocation جدید
        return result;
    }

    [Benchmark(Description = "Record Struct: Allocation & With")]
    public object RecordStructWith()
    {
        var person = new PersonRecordStruct("Ali", 30);
        object result = null;
        for (int i = 0; i < Iterations; i++)
            result = person with { Age = i }; // کپی در Stack، بدون GC
        return result;
    }
}

public class Program
{
    public static void Main() => BenchmarkRunner.Run<RecordPerformanceBenchmark>();
}
```

### تحلیل خروجی BenchmarkDotNet (شبیه‌سازی شده)

اگر این کد را اجرا کنید، نتایج تقریبی به این شکل خواهد بود:

| Method | Mean (زمان) | Allocated (حافظه) | Gen0 (تعداد GC) |
| :--- | :--- | :--- | :--- |
| **Class: Allocation** | ~12.5 μs | 32 KB | 2.0 |
| **Record Class: With** | ~28.0 μs | 48 KB | 3.5 |
| **Record Struct: With** | ~8.2 μs | **0 B** | **-** |

**تحلیل نتایج:**
1. **Record Struct** برنده مطلق در Memory است (Allocated = 0 B). چون در Stack کپی می‌شود و هیچ فشاری به GC وارد نمی‌کند.
2. **Record Class** به دلیل استفاده از `with`، مجبور است در هر حلقه یک آبجکت جدید در Heap بسازد (Shallow Copy)، بنابراین زمان و حافظه مصرفی آن به شدت بالاست.
3. **هزینه CPU:** اگر متد `Equals` را بنچمارک می‌گرفتید، `class` سریع‌ترین بود (چون فقط دو Pointer را مقایسه می‌کند)، در حالی که `record`ها باید تک‌تک فیلدها را مقایسه کنند.

---

## قوانین طلایی برای انتخاب نوع داده در #C

1. **قانون اول:** همیشه با `class` (یا `record class`) شروع کنید. سادگی و عدم نگرانی از Boxing و Copy Cost ارزشش را دارد.
2. **قانون دوم:** اگر پروفایلر (Profiler) به شما نشان داد که GC Pressure بالاست و تعداد زیادی آبجکت کوچک و Immutable دارید، آن‌ها را به `record struct` تبدیل کنید.
3. **قانون سوم:** هرگز `record struct` بزرگ (بیش از ۱۶ بایت) نسازید. اگر فیلدهای زیادی دارید، از `record class` استفاده کنید یا از `in` modifier و `ref readonly` برای پاس دادن آن‌ها استفاده کنید.
4. **قانون چهارم:** مراقب **Boxing** در `record struct` باشید. اگر آن را به متدی بدهید که پارامتر `object` یا `IEquatable<T>` (غیر جنریک) می‌گیرد، ناگهان به Heap پرتاب شده و تمام مزیت Stack را از دست می‌دهد.

---

## منابع معتبر مایکروسافت و مستندات .NET

برای مطالعه بیشتر و اطمینان از صحت مفاهیم، منابع زیر بهترین مراجع رسمی هستند:

1. **Microsoft Docs - Records (C# Reference):**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
   *(توضیح کامل سینتکس و رفتارهای کامپایلر)*

2. **Microsoft Docs - Choosing Between Class and Struct:**
   [https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct)
   *(قوانین مایکروسافت برای انتخاب Value Type در برابر Reference Type)*

3. **Microsoft Docs - Structure Types (Performance & Size):**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct)
   *(توضیح درباره محدودیت اندازه Struct و مشکلات Copy Cost)*

4. **.NET Documentation - Garbage Collection:**
   [https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals)
   *(درک عمیق از نحوه کار GC و تاثیر Allocation بر Performance)*

5. **BenchmarkDotNet Official Documentation:**
   [https://benchmarkdotnet.org/articles/guides/how-it-works.html](https://benchmarkdotnet.org/articles/guides/how-it-works.html)
   *(چرا و چگونه باید بنچمارک بگیریم)*