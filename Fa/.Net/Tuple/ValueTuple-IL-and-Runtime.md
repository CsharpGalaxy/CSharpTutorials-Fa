این یک مقاله جامع و ساختاریافته برای Repository آموزشی شماست. مقاله به گونه‌ای طراحی شده که ابتدا با یک مثال ساده شروع شود، سپس لایه به لایه عمیق‌تر شده و در نهایت به مباحث پیشرفته (Advanced Internals) برسد.

***

# کالبدشکافی ValueTuple در سی‌شارپ: از سینتکس شیرین تا عمق IL و Runtime

## مقدمه
از زمانی که C# 7.0 معرفی شد، `Tuple`ها (تاپل‌ها) به یکی از پرکاربردترین ویژگی‌های زبان تبدیل شدند. آن‌ها به ما اجازه می‌دهند چندین مقدار را در یک بسته واحد برگردانیم یا متغیرهای موقت بسازیم. اما آیا تا به حال فکر کرده‌اید که وقتی می‌نویسید `(string Name, int Age) person = ("Ali", 30);`، کامپایلر و Runtime دقیقاً چه کاری انجام می‌دهند؟ 

در این مقاله، از سطح سینتکس C# شروع کرده و به اعماق IL، Metadata و ساختار حافظه در Runtime نفوذ می‌کنیم.

---

## ۱. سطح رویه: C# Tuple Syntax
بیایید با یک مثال ساده و آشنا شروع کنیم:

```csharp
// تعریف یک Tuple با نام‌گذاری فیلدها
var person = (Name: "Ali", Age: 30, City: "Tehran");

// دسترسی به فیلدها
Console.WriteLine(person.Name); 
Console.WriteLine(person.Age);  
```

تا اینجا همه چیز ساده و بصری به نظر می‌رسد. اما بیایید بپرسیم: **آیا در Runtime واقعاً یک کلاس یا استراکچر به نام `Person` با فیلدهای `Name` و `Age` ساخته شده است؟**
پاسخ **خیر** است.

---

## ۲. توهم کد منبع در برابر واقعیت Runtime
یکی از مهم‌ترین مفاهیمی که باید درک کنید، تفاوت بین **Source Code** (کدی که شما می‌نویسید) و **Runtime Representation** (کدی که در حافظه اجرا می‌شود) است.

وقتی شما از سینتکس Tuple استفاده می‌کنید، کامپایلر C# (Roslyn) کد شما را به یک **Generic Struct** به نام `System.ValueTuple` تبدیل می‌کند. 

### Type واقعی در Runtime
در Runtime، نوع متغیر `person` در مثال بالا هیچ نامی از `Name` یا `Age` در ساختار فیلدهای خود ندارد. نوع واقعی آن به شکل زیر است:
`System.ValueTuple<System.String, System.Int32, System.String>`

فیلدهای واقعی این استراکچر در Runtime به ترتیب `Item1`، `Item2` و `Item3` هستند.

---

## ۳. کالبدشکافی ValueTuple (Generic Struct)
کلاس `ValueTuple` در واقع مجموعه‌ای از `struct`های تو در تو (Nested) و Generic است که در Namespace سیستمی `System` تعریف شده‌اند.

### فیلدهای Item1 تا Item7
ساختار `ValueTuple` به گونه‌ای طراحی شده که برای هر تعداد آرگومان (از ۱ تا ۸)، یک استراکچر جداگانه دارد:
* `ValueTuple<T1>`
* `ValueTuple<T1, T2>`
* ...
* `ValueTuple<T1, T2, T3, T4, T5, T6, T7>`

همه این فیلدها (`Item1` تا `Item7`) به صورت **Public Field** (و نه Property) تعریف شده‌اند تا دسترسی به آن‌ها در Runtime سریع‌ترین حالت ممکن (Direct Memory Access) باشد.

### جادوی TRest (برای بیشتر از ۷ آیتم)
اگر دقت کنید، `ValueTuple` حداکثر ۷ فیلد Generic دارد (`T1` تا `T7`). اگر شما یک Tuple با ۸ آیتم بسازید چه اتفاقی می‌افتد؟ آیا کامپایلر خطا می‌دهد؟ خیر!

در اینجا از یک ترفند در C# Language Specification استفاده می‌شود. برای آیتم هشتم، از فیلدی به نام `TRest` استفاده می‌شود که خودش یک `ValueTuple` است!

```csharp
// کد شما:
var hugeTuple = (1, 2, 3, 4, 5, 6, 7, 8);

// معادل آن در Runtime:
// ValueTuple<int, int, int, int, int, int, int, ValueTuple<int>>
```
در این حالت، `Item1` تا `Item7` مقادیر ۱ تا ۷ را نگه می‌دارند و `Rest` (که فیلد هشتم استراکچر است)، خودش یک `ValueTuple<int>` است که عدد ۸ را درون `Item1` خود دارد. این یعنی Tupleهای بزرگ در Runtime به صورت **تاپل‌های تو در تو (Nested Tuples)** ذخیره می‌شوند.

---

## ۴. نام‌گذاری فیلدها: فقط در Compile Time!
اگر فیلدهای استراکچر فقط `Item1` و `Item2` هستند، پس چگونه `person.Name` کار می‌کند و IDE شما IntelliSense را نشان می‌دهد؟

پاسخ در **Metadata** و **Attribute**ها نهفته است.
وقتی شما نامی برای آیتم‌ها انتخاب می‌کنید، کامپایلر C# یک Attribute به نام `[TupleElementNames]` را در Metadata اسمبلی شما تزریق می‌کند.

```csharp
// چیزی که کامپایلر در Metadata تولید می‌کند (به صورت مفهومی):
[TupleElementNames(new string[] { "Name", "Age" })]
public ValueTuple<string, int> person;
```

**نکته کلیدی:**
* **CLR (Runtime)** هیچ اهمیتی به `TupleElementNames` نمی‌دهد. این Attribute صرفاً برای **Roslyn (کامپایلر)** و **IDE** است.
* وقتی شما `person.Name` را می‌نویسید، کامپایلر در زمان Compile، این عبارت را به `person.Item1` ترجمه می‌کند.
* اگر اسمبلی را در ابزارهایی مثل ILSpectrum یا dnSpy باز کنید، نام فیلدها را `Item1` می‌بینید، اما Attribute آن را در متادیتای متد/فیلد مشاهده خواهید کرد.

---

## ۵. نگاهی به IL (Intermediate Language)
بیایید یک متد ساده که یک Tuple برمی‌گرداند را به IL نگاه کنیم تا مطمئن شویم:

**کد C#:**
```csharp
public (string Name, int Age) GetPerson() 
{
    return ("Sara", 25);
}
```

**خروجی IL (توسط ILSpy/dnSpy):**
```il
.method public hidebysig 
    instance valuetype [System.Runtime]System.ValueTuple`2<string, int32> GetPerson () cil managed 
{
    // متادیتا برای نام‌گذاری فیلدها (فقط در Compile Time کاربرد دارد)
    .custom instance void [System.Runtime]System.TupleElementNamesAttribute::.ctor(string[]) = (
        01 00 08 4e 61 6d 65 03 41 67 65 00 00 // "Name", "Age"
    )
    
    // بدنه متد
    IL_0000: nop
    IL_0001: ldstr "Sara"
    IL_0006: ldc.i4.s 25
    IL_0008: newobj instance void valuetype [System.Runtime]System.ValueTuple`2<string, int32>::.ctor(!0, !1)
    IL_000d: ret
}
```

**تحلیل IL:**
1. می‌بینید که Return Type متد، `ValueTuple'2` است.
2. نام‌های `Name` و `Age` فقط در `.custom` (همان Attribute) ذخیره شده‌اند.
3. در خط `newobj`، سازنده `ValueTuple` با دو آرگومان فراخوانی شده است. هیچ اشاره‌ای به نام‌های سفارشی شما در دستورالعمل‌های اجرایی IL وجود ندارد.

---

## ۶. بخش پیشرفته: Advanced Internals و ملاحظات پرفورمنس

برای توسعه‌دهندگانی که می‌خواهند از حداکثر پرفورمنس در .NET استفاده کنند، درک نکات زیر درباره `ValueTuple` حیاتی است:

### الف) Value Type بودن و تخصیص حافظه
از آنجا که `ValueTuple` یک `struct` است، روی **Stack** (یا درون آبجکت والد اگر فیلد یک کلاس باشد) تخصیص می‌یابد. این یعنی:
* **بدون سربار GC:** فشاری روی Garbage Collector وارد نمی‌کند.
* **کپی شدن (Copying):** هر بار که Tuple را پاس می‌دهید یا برمی‌گردانید، تمام بایت‌های آن کپی می‌شوند. برای Tupleهای کوچک (۱ تا ۳ آیتم) عالی است، اما برای Tupleهای ۷ آیتمی بزرگ، سربار کپی کردن حافظه (Memory Copy) می‌تواند پرفورمنس را در حلقه‌هایtight کاهش دهد.

### ب) مشکل Mutability (تغییرپذیری)
در طراحی اولیه C# 7.0، فیلدهای `ValueTuple` (مثل `Item1`) به عنوان `public T1 Item1;` تعریف شدند (بدون کلمه `readonly`). 
این یک تصمیم بحث‌برانگیز بود. اگر یک Tuple را از یک Property برگردانید و سعی کنید `Item1` آن را تغییر دهید، شما در واقع یک **کپی** از استراکچر را تغییر داده‌اید و مقدار اصلی تغییر نمی‌کند!
*توصیه:* همیشه با Tupleها به چشم **Immutable** (غیرقابل تغییر) رفتار کنید.

### ج) پیاده‌سازی Equals و GetHashCode
وقتی دو Tuple را با `==` یا `Equals` مقایسه می‌کنید، `ValueTuple` از `EqualityComparer<T>.Default` برای هر آیتم استفاده می‌کند. 
* **نکته پرفورمنسی:** اگر نوع‌های استفاده شده در Tuple (مثل `T1`) اینترفیس `IEquatable<T>` را پیاده‌سازی نکرده باشند، Runtime مجبور به **Boxing** (تبدیل Value Type به Reference Type) می‌شود که پرفورمنس را به شدت کاهش می‌دهد. همیشه از تایپ‌های استاندارد یا پیاده‌سازی `IEquatable` استفاده کنید.

### د) Deconstruction در سطح IL
وقتی می‌نویسید `var (name, age) = GetPerson();`، کامپایلر این را به فراخوانی متد `Deconstruct` روی `ValueTuple` ترجمه می‌کند. این متد پارامترهای `out` دارد و مقادیر `Item1` و `Item2` را در متغیرهای محلی شما کپی می‌کند.

---

## خلاصه و نتیجه‌گیری

| ویژگی | در Source Code (C#) | در Runtime / IL |
| :--- | :--- | :--- |
| **نام فیلدها** | `Name`, `Age` | `Item1`, `Item2` |
| **نوع داده** | `(string, int)` | `System.ValueTuple<String, Int32>` |
| **نام‌گذاری سفارشی** | بخشی از سینتکس | فقط در Metadata (`TupleElementNamesAttribute`) |
| **تعداد آیتم‌ها** | نامحدود (تئوری) | حداکثر ۷ فیلد + `TRest` (تو در تو) |
| **نوع ساختار** | - | `Generic Struct` (Value Type) |

**پیام نهایی:**
سینتکس Tuple در سی‌شارپ یک شاهکار از **Compiler Sugar** است. این ویژگی به ما اجازه می‌دهد کدی تمیز و خوانا بنویسیم، در حالی که در پشت صحنه، Runtime با ساختارهای بهینه‌سازی شده، Genericها و Metadata، بهترین پرفورمنس ممکن را بدون درگیر کردن Garbage Collector ارائه می‌دهد. درک این تفاوت بین "آنچه می‌نویسیم" و "آنچه اجرا می‌شود"، مرز بین یک برنامه‌نویس معمولی و یک مهندس نرم‌افزار مسلط به پلتفرم .NET است.

***
*منابع برای مطالعه بیشتر:*
* *C# Language Specification - Tuples*
* *Microsoft Docs: System.ValueTuple Struct*
* *CLR via C# (Jeffrey Richter) - بخش Value Types و Metadata*