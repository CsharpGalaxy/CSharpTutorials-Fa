# کالبدشکافی Record در C#: جادوی Compile-Time در برابر واقعیت Runtime

وقتی برای اولین بار با کلمه کلیدی `record` در C# (معرفی شده در نسخه 9.0) مواجه می‌شوید، ممکن است تصور کنید که مایکروسافت یک «نوع داده» کاملاً جدید به دنیای C# اضافه کرده است. اما واقعیت در پشت پرده بسیار جذاب‌تر و هوشمندانه‌تر از این حرف‌هاست. 

در این مقاله تخصصی اما قابل فهم، به کالبدشکافی `Record` می‌پردازیم و ثابت می‌کنیم که این ویژگی، بیش از آنکه یک «نوع» باشد، یک «شعبده‌بازی کامپایلر» است.

---

## ۱. آیا CLR نوعی به نام Record دارد؟
**پاسخ کوتاه: خیر، مطلقاً خیر.**

محیط اجرای مشترک (CLR - Common Language Runtime) و ماشین مجازی دات‌نت، هیچ درکی از کلمه کلیدی `record` ندارند. در سطح IL (زبان میانی دات‌نت)، هیچ Opcode یا Type جدیدی به نام Record وجود ندارد. 
شما در CLR فقط دو نوع داده اصلی برای تعریف ساختارها دارید:
1. **Reference Types** (که با کلاس‌ها ساخته می‌شوند و از `System.Object` ارث‌بری می‌کنند).
2. **Value Types** (که با استراکچرها ساخته می‌شوند و از `System.ValueType` ارث‌بری می‌کنند).

**تأکید مهم:** `Record` یک نوع مستقل در Runtime نیست؛ بلکه یک **Modifier (تغییردهنده) یا Contextual Keyword در سطح زبان C#** است که به کامپایلر (Roslyn) دستور می‌دهد کدهای تکراری (Boilerplate) را برای شما تولید کند.

---

## ۲. Record در سطح زبان C# چیست؟
در سطح زبان، `record` یک میانبر (Syntactic Sugar) برای ایجاد **انواع داده‌محور (Data-Centric)** است. وقتی شما از `record` استفاده می‌کنید، به کامپایلر می‌گویید:
> «من می‌خواهم یک کلاس (یا استراکچر) بسازم که هدف اصلی‌اش نگهداری داده است. لطفاً خودش برابری مقادیر (Value Equality)، قابلیت کپی کردن (Cloning)، و نمایش خوانا (ToString) را برایم پیاده‌سازی کند.»

---

## ۳. Record در زمان Compile به چه چیزی تبدیل می‌شود؟
بیایید یک Record ساده بنویسیم و ببینیم کامپایلر در پشت صحنه چه می‌کند.

### نمونه Record ساده:
```csharp
public record Person(string FirstName, string LastName);
```

### معادل مفهومی آن با Class (آنچه کامپایلر تولید می‌کند):
وقتی کد بالا را کامپایل می‌کنید، کامپایلر آن را به یک `class` معمولی تبدیل می‌کند که اعضای زیر به صورت خودکار به آن تزریق شده‌اند (Pseudo-code معادل):

```csharp
public class Person : IEquatable<Person>
{
    // 1. تولید Properties با دسترسی init-only (غیرقابل تغییر پس از ساخت)
    public string FirstName { get; init; }
    public string LastName { get; init; }

    // 2. Constructor اصلی
    public Person(string FirstName, string LastName) { ... }

    // 3. تولید متد ToString برای نمایش خوانا
    public override string ToString() 
    {
        return $"Person {{ FirstName = {FirstName}, LastName = {LastName} }}";
    }

    // 4. تولید متدهای Equality (برابری بر اساس مقدار، نه مرجع)
    public virtual bool Equals(Person other) { ... }
    public override int GetHashCode() { ... }
    
    // 5. اپراتورهای == و !=
    public static bool operator ==(Person left, Person right) { ... }
    public static bool operator !=(Person left, Person right) { ... }

    // 6. متد Clone و Copy Constructor (برای پشتیبانی از with)
    protected Person(Person original) { /* کپی کردن فیلدها */ }
    public Person <Clone>$() { return new Person(this); }
}
```
*نکته: در خروجی IL واقعی، کلاس شما مستقیماً از `System.Object` ارث‌بری می‌کند، نه هیچ کلاس پایه‌ای مرتبط با Record.*

---

## ۴. Compiler چه اعضایی را برای Record تولید می‌کند؟

کامپایلر Roslyn وظایف زیر را بر عهده دارد:

### الف) نقش Compiler در ایجاد Equality (برابری)
برخلاف کلاس‌های معمولی که برابری آن‌ها بر اساس **Reference** (آدرس حافظه) است، کامپایلر برای Record برابری را بر اساس **Value** (مقدار فیلدها) بازنویسی می‌کند. 
کامپایلر به جای استفاده از Reflection (که کند است)، کدی بهینه تولید می‌کند که تک‌تک فیلدها را با استفاده از `EqualityComparer<T>.Default` مقایسه می‌کند.

### ب) نقش Compiler در ToString
کامپایلر متد `ToString` را Override می‌کند تا خروجی را به فرمت `RecordName { Property1 = Value1, Property2 = Value2 }` برگرداند. این کار دیباگ کردن را بسیار لذت‌بخش می‌کند.

### ج) نقش Compiler در `with` و Copy Constructor
وقتی شما از اکسپرشن `with` استفاده می‌کنید:
```csharp
var person2 = person1 with { LastName = "Smith" };
```
کامپایلر این کد را به شکل زیر باز نویسی می‌کند:
1. فراخوانی متد `<Clone>$` (که کامپایلر ساخته است).
2. استفاده از Object Initializer برای تغییر فیلد مورد نظر.
متد `<Clone>$` در واقع یک **Copy Constructor** را صدا می‌زند که تمام مقادیر را از آبجکت اصلی کپی می‌کند. (نام `<Clone>$` عمداً دارای کاراکترهای غیرمجاز C# انتخاب شده تا برنامه‌نویس نتواند مستقیماً آن را در کد صدا بزند).

---

## ۵. تفاوت Record Class با Class معمولی

| ویژگی | Class معمولی | Record Class |
| :--- | :--- | :--- |
| **نوع در CLR** | Reference Type | Reference Type |
| **برابری (Equality)** | Reference Equality (پیش‌فرض) | Value Equality (تولید شده توسط کامپایلر) |
| **تغییرپذیری (Mutability)** | معمولاً Mutable (با `set`) | پیش‌فرض Immutable (با `init`) |
| **متد ToString** | نام کامل کلاس (Namespace.ClassName) | فرمت خوانا شامل نام پراپرتی‌ها و مقادیر |
| **ارث‌بری** | می‌تواند از کلاس‌های دیگر ارث ببرد | فقط می‌تواند از یک Record دیگر ارث ببرد |

---

## ۶. تفاوت Record Struct با Struct معمولی (معرفی شده در C# 10)
در C# 10، امکان استفاده از `record struct` فراهم شد.

| ویژگی | Struct معمولی | Record Struct |
| :--- | :--- | :--- |
| **نوع در CLR** | Value Type | Value Type |
| **برابری (Equality)** | Value Equality (اما با استفاده از Reflection که **کند** است) | Value Equality (تولید کد بهینه و **سریع** توسط کامپایلر) |
| **تغییرپذیری** | Mutable | پیش‌فرض Mutable (برخلاف Record Class!) |
| **Immutable کردن** | نیاز به تعریف دستی فیلدهای `readonly` | استفاده از `readonly record struct` |

*نکته مهم:* برخلاف `record class` که پیش‌فرض آن `init` است، در `record struct` پراپرتی‌ها پیش‌فرض `set` هستند (مگر اینکه `readonly` را اضافه کنید).

---

## ۷. رفتار Record در Runtime و تفاوت Compile-Time Abstraction با Runtime Type

این بخش مهم‌ترین مفهوم برای درک عمیق C# است.

### انتزاع در زمان Compile (Compile-Time Abstraction)
شما به عنوان برنامه‌نویس، `record` را می‌نویسید. این یک **انتزاع** است. کامپایلر این انتزاع را می‌گیرد و آن را به مفاهیم پایه‌ای CLR (کلاس‌ها و استراکچرها با متدهای اضافی) ترجمه می‌کند.

### واقعیت در زمان Runtime
وقتی کد شما اجرا می‌شود، CLR فقط یک `Class` یا `Struct` می‌بیند. 
اگر از Reflection استفاده کنید:
```csharp
Person p = new Person("Ali", "Rezaei");
Console.WriteLine(p.GetType().BaseType); 
// خروجی: System.Object (نه System.Record!)
```

**چگونه در Runtime بفهمیم یک کلاس، Record بوده است؟**
از آنجا که CLR نوعی به نام Record ندارد، در Runtime هیچ راه رسمی و مستقیمی (مثل یک `IsRecord` property) وجود ندارد. اما کامپایلر برای کمک به ابزارهایی مثل Reflection یا Serializerها، سرنخ‌هایی باقی می‌گذارد:
1. متدهای تولید شده توسط کامپایلر (مثل `<Clone>$` و `PrintMembers`) دارای Attribute `[CompilerGenerated]` هستند.
2. حضور متد `<Clone>$` (که نام آن در C# قابل نوشتن نیست) قوی‌ترین نشانه این است که این کلاس در سطح زبان، یک Record بوده است.

---

## ۸. نگاهی به IL (خروجی کامپایلر)
اگر کد `public record Person(string Name);` را کامپایل کرده و با ابزارهایی مثل ILSpy باز کنید، بخش تعریف کلاس در IL به این شکل است:

```il
.class public auto ansi serializable beforefieldinit Person
    extends [System.Runtime]System.Object
    implements class [System.Runtime]System.IEquatable`1<class Person>
{
    // توجه کنید که هیچ اشاره‌ای به Record نشده است!
    // این دقیقاً همان تعریف یک کلاس معمولی در IL است.
}
```
این خروجی IL، سند قطعی این ادعاست که **Record فقط یک ویژگی زبان (Language Feature) است و در سطح ماشین مجازی دات‌نت وجود خارجی ندارد.**

---

## جمع‌بندی
استفاده از `Record` در C# یک شاهکار مهندسی در کامپایلر Roslyn است. این ویژگی به شما اجازه می‌دهد تا با نوشتن **یک خط کد**، از شر صدها خط کد تکراری برای پیاده‌سازی `Equals`، `GetHashCode`، `ToString` و متدهای Clone راحت شوید. 
اما به عنوان یک توسعه‌دهنده حرفه‌ای، همیشه به یاد داشته باشید که در نهایت، شما در حال کار با همان `Class` و `Struct` همیشگی در Runtime هستید، فقط با لباسی جدید و امکاناتی بیشتر که کامپایلر برایتان دوخته است.

---

## منابع معتبر

1. **Microsoft Docs - Records (C# Reference)**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
2. **Microsoft Docs - What's new in C# 9.0 (Introduction of Records)**
   [https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#record-types](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#record-types)
3. **Microsoft Docs - What's new in C# 10.0 (Record Structs)**
   [https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#record-structs](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#record-structs)
4. **GitHub - C# Language Design Notes (Records Proposal)**
   [https://github.com/dotnet/csharplang/blob/main/proposals/csharp-9.0/records.md](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-9.0/records.md)
5. **Microsoft Docs - How to determine whether an object is a record (Reflection approach)**
   [https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records#how-to-determine-whether-an-object-is-a-record](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records#how-to-determine-whether-an-object-is-a-record)