# بررسی عمیق Record در C#: از Source Code تا IL و Compiler

رکوردها (Records) در C# 9 معرفی شدند و به سرعت به یکی از محبوب‌ترین ویژگی‌ها برای کار با داده‌ها (Data-centric) تبدیل شدند. در نگاه اول، به نظر می‌رسد که رکوردها یک "جادو" (Magic) هستند؛ شما یک خط کد می‌نویسید و کامپایلر ده‌ها متد و قابلیت مثل Equality، Immutability و `with` expression را برای شما تولید می‌کند.

اما در دنیای مهندسی نرم‌افزار، جادویی وجود ندارد! همه چیز **Syntactic Sugar** و تولید کد در سطح **Compiler** است. در این مقاله Advanced، کالبدشکافی دقیقی از یک Record انجام می‌دهیم تا ببینیم کامپایلر دقیقاً چه کدهایی را در سطح IL و Metadata تولید می‌کند.

---

## ابزارهای مورد استفاده برای کالبدشکافی

برای بررسی کدهای تولید شده، ما به ابزارهایی نیاز داریم که بتوانند کد C# را کامپایل کرده و خروجی آن را در سطح IL و Metadata نشان دهند:

1. **SharpLab:** یک ابزار تحت وب فوق‌العاده که به شما اجازه می‌دهد کد C# را بنویسید و همزمان خروجی آن را به صورت IL، C# دیکامپایل شده (توسط ILSpy) و VB ببینید. برای بررسی‌های سریع و آموزش، بهترین گزینه است.
2. **ILSpy:** یک دیکامپایلر قدرتمند دسکتاپ (و افزونه Visual Studio) که فایل‌های DLL را باز کرده و ساختار Metadata، IL و کد دیکامپایل شده را نمایش می‌دهد.
3. **dotnet ildasm:** یک ابزار خط فرمان (CLI) از مجموعه .NET SDK که فایل‌های DLL را به فایل‌های متنی IL تبدیل می‌کند. برای بررسی‌های دقیق و اسکریپت‌نویسی روی خروجی کامپایلر عالی است.

---

## 1. Source Code (کد مبدأ)

بیایید با یک رکورد موقعیتی (Positional Record) بسیار ساده شروع کنیم:

```csharp
public record Person(string FirstName, string LastName);
```

این تمام چیزی است که ما می‌نویسیم. اما بیایید ببینیم کامپایلر (Roslyn) چه بلایی سر این یک خط می‌آورد.

---

## 2. Compilation و Generated Members (اعضای تولید شده)

وقتی شما کد بالا را کامپایل می‌کنید، کامپایلر آن را به یک **کلاس** (یا استراکچر، اگر از `record struct` استفاده کنید) تبدیل می‌کند. کامپایلر به طور خودکار اعضای زیر را به کلاس اضافه می‌کند:

1. **Primary Constructor & Properties:** پارامترهای رکورد به پراپرتی‌های `init` تبدیل می‌شوند.
2. **EqualityContract:** یک پراپرتی محافظت شده برای بررسی تطابق نوع در ارث‌بری.
3. **Equals & PrintMembers:** برای پیاده‌سازی Value Equality.
4. **GetHashCode:** برای تولید هش بر اساس مقادیر.
5. **Clone & Copy Constructor:** برای پشتیبانی از اکسپرشن `with`.
6. **ToString:** برای نمایش خوانای رکورد.

---

## 3. بررسی عمیق اعضا در سطح IL و کد تولید شده

در ادامه، کدی که کامپایلر تولید می‌کند (معادل C# دیکامپایل شده) و نکات IL آن را بررسی می‌کنیم.

### الف) Constructor و Properties (پراپرتی‌ها و سازنده)

کامپایلر یک سازنده اصلی (Primary Constructor) و پراپرتی‌هایی با setter از نوع `init` تولید می‌کند.

```csharp
// کد معادل تولید شده توسط کامپایلر
public class Person : IEquatable<Person>
{
    public string FirstName { get; init; }
    public string LastName { get; init; }

    public Person(string FirstName, string LastName)
    {
        this.FirstName = FirstName;
        this.LastName = LastName;
    }
}
```

**نکته IL و Metadata:**
در سطح IL، پراپرتی‌های `init` در واقع همان پراپرتی‌های معمولی با یک `set` method هستند، با این تفاوت که متادیتای آن‌ها دارای یک `modreq` (Modifier Required) به نام `System.Runtime.CompilerServices.IsExternalInit` است. این به Runtime و Compiler می‌فهماند که این setter فقط در زمان مقداردهی اولیه (Constructor یا Object Initializer) قابل صدا زدن است.

```il
// بخش IL برای setter پراپرتی FirstName
.property instance string FirstName()
{
    .custom instance void System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
    .get instance string Person::get_FirstName()
    .set instance void modreq([System.Runtime]System.Runtime.CompilerServices.IsExternalInit) 
          Person::set_FirstName(string)
}
```

### ب) EqualityContract (قرارداد برابری)

این یکی از هوشمندانه‌ترین بخش‌های رکوردهاست. یک پراپرتی `protected virtual` که `Type` کلاس را برمی‌گرداند.

```csharp
protected virtual Type EqualityContract => typeof(Person);
```

**چرا این لازم است؟**
اگر شما از یک رکورد ارث‌بری کنید (Inheritance)، این پراپرتی در کلاس فرزند Override می‌شود تا `typeof(DerivedPerson)` را برگرداند. متد `Equals` ابتدا این قرارداد را چک می‌کند تا مطمئن شود دو آبجکت دقیقاً از یک نوع هستند. این کار از باگ‌های مربوط به Covariance در Equality جلوگیری می‌کند.

### ج) Equals و PrintMembers (بررسی برابری)

کامپایلر متد `Equals` را برای بررسی برابری مقادیر (Value Equality) پیاده‌سازی می‌کند.

```csharp
public virtual bool Equals(Person? other)
{
    return other is not null 
        && EqualityContract == other.EqualityContract 
        && FirstName == other.FirstName 
        && LastName == other.LastName;
}

// متد کمکی برای ToString و ارث‌بری
protected virtual bool PrintMembers(StringBuilder builder)
{
    builder.Append("FirstName = ");
    builder.Append(FirstName?.ToString());
    builder.Append(", LastName = ");
    builder.Append(LastName?.ToString());
    return true;
}
```

**نکته IL:**
در سطح IL، کامپایلر برای مقایسه پراپرتی‌ها از `System.Collections.Generic.EqualityComparer<T>.Default.Equals` استفاده می‌کند تا از باکسینگ (Boxing) برای تایپ‌های Value جلوگیری کند و همچنین `null` را به درستی مدیریت نماید.
متد `PrintMembers` به صورت `virtual` تولید می‌شود تا اگر کلاسی از این رکورد ارث‌بری کرد، بتواند فیلدهای جدید خود را به خروجی `ToString` اضافه کند.

### د) GetHashCode

تولید هش کد بر اساس تمام فیلدهای رکورد:

```csharp
public override int GetHashCode()
{
    // استفاده از HashCode.Combine برای ترکیب هش‌ها
    return HashCode.Combine(EqualityComparer<string>.Default.GetHashCode(FirstName), 
                            EqualityComparer<string>.Default.GetHashCode(LastName));
}
```

### هـ) Clone و Copy Constructor (هسته اصلی with)

برای اینکه اکسپرشن `with` کار کند، کامپایلر دو عضو حیاتی تولید می‌کند:

```csharp
// 1. متد Clone (با نام خاص <Clone>$)
protected virtual Person <Clone>$()
{
    return new Person(this); // صدا زدن Copy Constructor
}

// 2. Copy Constructor
protected Person(Person original)
{
    this.FirstName = original.FirstName;
    this.LastName = original.LastName;
}
```

**نکته Metadata:**
نام متد Clone در IL به صورت `<Clone>$` تولید می‌شود. این نام‌گذاری خاص (با کاراکترهای `<` و `>`) باعث می‌شود که این متد از دید برنامه‌نویس در IntelliSense مخفی بماند، زیرا کامپایلر به طور صریح آن را برای استفاده داخلی خود تولید کرده است.

### و) ToString

تولید یک خروجی خوانا با استفاده از `PrintMembers`:

```csharp
public override string ToString()
{
    StringBuilder builder = new StringBuilder();
    builder.Append("Person");
    builder.Append(" { ");
    if (PrintMembers(builder))
    {
        builder.Append(' ');
    }
    builder.Append('}');
    return builder.ToString();
}
```

---

## 4. بررسی اکسپرشن `with` در سطح IL

حالا بیایید جادوی `with` را بررسی کنیم. فرض کنید کد زیر را نوشته‌اید:

```csharp
var p1 = new Person("Ali", "Rezaei");
var p2 = p1 with { LastName = "Smith" };
```

شما ممکن است فکر کنید کامپایلر یک آبجکت جدید می‌سازد و مقادیر را کپی می‌کند. اما در سطح IL، کامپایلر دقیقاً از همان متدهایی که در بخش قبل دیدیم استفاده می‌کند!

**کد معادل تولید شده توسط کامپایلر برای `with`:**

```csharp
// کامپایلر این کد را برای p2 تولید می‌کند:
Person person = p1.<Clone>$(); // 1. کپی کردن کل آبجکت
person.LastName = "Smith";     // 2. تغییر فقط فیلد مورد نظر
Person p2 = person;
```

**بررسی IL تولید شده برای `with`:**

```il
IL_0000: ldarg.0      // لود کردن p1 روی استک
IL_0001: callvirt instance Person Person::<Clone>$() // صدا زدن متد Clone
IL_0006: ldstr "Smith" // لود کردن مقدار جدید
IL_0007: callvirt instance void Person::set_LastName(string) // ست کردن پراپرتی
IL_000c: stloc.0      // ذخیره در متغیر محلی (p2)
```

**نتیجه‌گیری از IL:**
همانطور که در IL می‌بینید، هیچ جادویی وجود ندارد! کامپایلر فقط یک `callvirt` به متد `<Clone>$` انجام می‌دهد (که خودش از Copy Constructor استفاده می‌کند) و سپس setter پراپرتی مورد نظر را صدا می‌زند.

---

## 5. Metadata: آیا Record یک نوع جدید در CLR است؟

یکی از مهم‌ترین سوالات این است: آیا CLR (Common Language Runtime) می‌داند که این کلاس یک Record است؟
**پاسخ: خیر.**

اگر فایل DLL را با ILSpy باز کنید، در سطح Metadata، این کلاس هیچ تفاوتی با یک کلاس معمولی ندارد. هیچ `[RecordAttribute]` خاصی در Metadata آن وجود ندارد (مگر در برخی نسخه‌های میانی کامپایلر که بعداً حذف شد).

**چرا؟**
چون تیم طراحی C# تصمیم گرفت که رکوردها صرفاً یک ویژگی در سطح **Compiler (Roslyn)** باشند و نیازی به تغییر CLR یا اضافه کردن تایپ‌های جدید به .NET Runtime نباشد. این کار باعث می‌شود رکوردها با تمام کتابخانه‌های قدیمی، Reflection و فریم‌ورک‌ها کاملاً سازگار (Compatible) باشند.

کامپایلر Roslyn برای تشخیص اینکه یک کلاس "رکورد" است (مثلاً برای اجازه دادن به استفاده از `with` روی آن)، به دنبال "شکل" (Shape) خاصی از کلاس می‌گردد: وجود متد `<Clone>$`، `PrintMembers` و `EqualityContract`.

---

## خلاصه و نتیجه‌گیری

همانطور که در این بررسی Advanced دیدیم، **Record یک ویژگی جادویی نیست**. 
کامپایلر C# (Roslyn) تنها کاری که می‌کند این است که کد کوتاه و تمیز شما را می‌گیرد و **Boilerplate Code** (کدهای تکراری و خسته‌کننده) مثل پیاده‌سازی `Equals`, `GetHashCode`, `Clone` و `ToString` را به صورت خودکار تولید می‌کند.

درک این موضوع به شما کمک می‌کند تا:
1. بدانید که رکوردها دقیقاً همان کلاس‌ها (یا استراکچرها) هستند و هیچ سربار Runtime اضافی ندارند.
2. در صورت نیاز، بتوانید متدهایی مثل `PrintMembers` یا `Equals` را Override کنید تا رفتار دلخواه خود را پیاده‌سازی کنید.
3. مشکلات احتمالی در ارث‌بری (Inheritance) رکوردها را به دلیل وجود `EqualityContract` بهتر درک و دیباگ کنید.

---

## منابع رسمی برای مطالعه بیشتر

برای عمیق‌تر شدن در این مبحث، منابع زیر پیشنهاد می‌شوند:

1. **Microsoft Docs - Records (C# Reference):**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
2. **C# Language Specification (Records Chapter):**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15313-records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15313-records)
3. **Roslyn Source Code (Implementation of Records):**
   برای دیدن کد دقیق کامپایلر که این اعضا را تولید می‌کند، می‌توانید به ریپازیتوری گیت‌هاب Roslyn مراجعه کنید:
   [https://github.com/dotnet/roslyn](https://github.com/dotnet/roslyn) (جستجو برای `RecordBinder` یا `SynthesizedRecordMembers`).