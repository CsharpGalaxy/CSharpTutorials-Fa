# بررسی عمیق Copy Constructor و Clone در Recordهای سی‌شارپ

رکوردها (Records) که در C# 9 معرفی شدند، انقلابی در نحوه مدیریت داده‌ها و اشیاء غیرقابل تغییر (Immutable) ایجاد کردند. یکی از جذاب‌ترین ویژگی‌های رکوردها، قابلیت کپی کردن و تغییر آسان آن‌هاست. اما در پشت صحنه این سادگی، مکانیزم‌های عمیقی نهفته است. در این مقاله به کالبدشکافی **Copy Constructor** و **Clone** در رکوردها می‌پردازیم.

---

## ۱. Copy Constructor چیست؟

در برنامه‌نویسی شی‌گرا، **Copy Constructor** (سازنده کپی) یک الگوی طراحی است که در آن یک سازنده (Constructor)، یک نمونه از همان کلاس را به عنوان پارامتر دریافت کرده و یک شیء جدید با مقادیر مشابه ایجاد می‌کند.

```csharp
public class Person
{
    public string Name { get; set; }
    
    // Copy Constructor
    public Person(Person original)
    {
        this.Name = original.Name;
    }
}
```

## ۲. چرا Record به Copy Constructor نیاز دارد؟

رکوردها ذاتاً برای **Immutability** (تغییرناپذیری) طراحی شده‌اند. وقتی یک پراپرتی در رکورد تعریف می‌کنید، به صورت پیش‌فرض `init` یا `readonly` است. بنابراین، شما نمی‌توانید پس از ایجاد شیء، مقدار آن را تغییر دهید.

اگر بخواهید یک پراپرتی را تغییر دهید، چاره‌ای جز ایجاد یک **شیء جدید** ندارید. برای اینکه مجبور نباشید تمام پراپرتی‌های دیگر را دوباره مقداردهی کنید، به یک مکانیزم نیاز دارید که تمام State (وضعیت) شیء قبلی را در شیء جدید کپی کند و سپس تغییرات جدید را اعمال نماید. اینجاست که Copy Constructor وارد عمل می‌شود.

## ۳. Copy Constructor تولیدشده توسط Compiler

وقتی شما یک `record class` تعریف می‌کنید، کامپایلر سی‌شارپ به صورت خودکار یک Copy Constructor **Protected** برای آن تولید می‌کند.

```csharp
public record Person(string Name, int Age);
```

کد معادل تولید شده توسط کامپایلر (به زبان ساده‌شده):

```csharp
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }

    // سازنده اصلی
    public Person(string name, int age) { ... }

    // Copy Constructor تولید شده توسط کامپایلر
    protected Person(Person original)
    {
        this.Name = original.Name;
        this.Age = original.Age;
    }
}
```
**نکته مهم:** این سازنده همیشه `protected` است، یعنی شما نمی‌توانید آن را مستقیماً از بیرون کلاس فراخوانی کنید، اما کلاس‌های مشتق‌شده (Derived) برای کپی کردن State کلاس پایه به آن دسترسی دارند.

---

## ۴. Clone چیست و چه رابطه‌ای با `with` دارد؟

### Clone (کلون)
کلون کردن به معنای ایجاد یک کپی دقیق از یک شیء است. در سی‌شارپ، این مفهوم معمولاً با اینترفیس `ICloneable` و متد `Clone()` شناخته می‌شود، اما در رکوردها کامپایلر یک متد خاص به نام `Clone` تولید می‌کند.

### رابطه Clone و `with`
وقتی شما از عبارت `with` استفاده می‌کنید، در واقع در حال فراخوانی غیرمستقیم متد `Clone` هستید. عبارت `with` فقط یک **Syntactic Sugar** (شکر نحوی) برای کلون کردن و تغییر همزمان است.

---

## ۵. پشت صحنه عبارت `with` چه اتفاقی می‌افتد؟

وقتی کد زیر را می‌نویسید:
```csharp
var newPerson = oldPerson with { Name = "Ali" };
```

کامپایلر این کد را به شکل زیر بازنویسی (Desugar) می‌کند:

1. **فراخوانی Clone:** ابتدا متد `Clone()` روی شیء `oldPerson` فراخوانی می‌شود.
2. **اجرای Copy Constructor:** متد `Clone()` به نوبه خود، Copy Constructor را صدا می‌زند تا یک کپی از شیء ایجاد کند.
3. **اعمال تغییرات:** در نهایت، مقدار جدید (`Name = "Ali"`) روی شیء کپی شده اعمال می‌شود.

کد معادل در سطح IL (زبان واسط):
```csharp
// 1. ایجاد کلون با استفاده از متد تولید شده توسط کامپایلر
Person <clone> = oldPerson.Clone(); 

// 2. اعمال تغییرات (کامپایلر اجازه می‌دهد پراپرتی‌های init را اینجا ست کنید)
<clone>.Name = "Ali"; 

// 3. مقداردهی به متغیر جدید
var newPerson = <clone>;
```

---

## ۶. تفاوت Copy Constructor و Clone

بسیاری از توسعه‌دهندگان این دو مفهوم را با هم اشتباه می‌گیرند. تفاوت آن‌ها به شرح زیر است:

| ویژگی | Copy Constructor | Clone (متد Clone) |
| :--- | :--- | :--- |
| **ماهیت** | یک **سازنده** (Constructor) است. | یک **متد** (Method) است. |
| **هدف** | مقداردهی اولیه یک شیء جدید بر اساس یک شیء موجود. | بازگرداندن یک کپی از شیء فعلی. |
| **دسترسی در رکورد** | همیشه `protected` است. | همیشه `protected virtual` (یا `override`) است. |
| **نحوه استفاده** | توسط متد `Clone` یا کلاس‌های فرزند فراخوانی می‌شود. | توسط عبارت `with` فراخوانی می‌شود. |
| **رابطه** | **ابزار** انجام کلون کردن است. | **مفهوم** و نقطه ورود برای کلون کردن است. |

به زبان ساده: متد `Clone` از Copy Constructor استفاده می‌کند تا کارش را انجام دهد.

---

## ۷. Shallow Copy و Deep Copy

### Shallow Copy (کپی سطحی - پیش‌فرض رکوردها)
به صورت پیش‌فرض، رکوردها **Shallow Copy** انجام می‌دهند. یعنی اگر رکورد شما دارای پراپرتی‌هایی از نوع Reference (مثل کلاس‌ها، آرایه‌ها یا لیست‌ها) باشد، فقط **آدرس مرجع (Reference)** کپی می‌شود، نه خودِ شیء.

```csharp
public record Address(string City);
public record Person(string Name, Address Address);

var oldAddress = new Address("Tehran");
var oldPerson = new Person("Reza", oldAddress);

var newPerson = oldPerson with { Name = "Ali" };

// تغییر در Address شیء جدید، روی شیء قدیم هم تاثیر می‌گذارد!
// چون هر دو به یک آبجکت Address در حافظه اشاره می‌کنند.
```

### Deep Copy (کپی عمیق)
اگر نیاز به Deep Copy دارید (یعنی کپی کامل و مستقل از تمام اشیاء مرجع)، باید **Copy Constructor سفارشی** بنویسید.

---

## ۸. Copy Constructor سفارشی (برای Deep Copy)

شما می‌توانید Copy Constructor پیش‌فرض را بازنویسی کنید تا کپی عمیق انجام دهد:

```csharp
public record Person
{
    public string Name { get; init; }
    public Address Address { get; init; }

    public Person(string name, Address address) { ... }

    // Copy Constructor سفارشی برای Deep Copy
    protected Person(Person original)
    {
        this.Name = original.Name;
        // کپی عمیق: ایجاد یک شیء Address جدید
        this.Address = new Address(original.Address.City); 
    }
}
```
حالا وقتی از `with` استفاده کنید، `Address` نیز به صورت کامل کپی می‌شود و تغییرات آن روی شیء اصلی اثر نمی‌گذارد.

---

## ۹. Copy Constructor در Record Inheritance (ارث‌بری)

ارث‌بری در رکوردها چالش‌برانگیز است، زیرا باید مطمئن شویم که وقتی یک شیء از نوع Derived را از طریق یک متغیر Base کپی می‌کنیم، نوع واقعی (Runtime Type) حفظ شود.

### کپی کردن State کلاس Base و Derived
برای حل این مشکل، کامپایلر یک متد `Clone` مجازی (Virtual) تولید می‌کند و Copy Constructor کلاس Derived باید حتماً Copy Constructor کلاس Base را فراخوانی کند.

#### مثال ارث‌بری:

```csharp
// کلاس پایه
public record Person(string Name, int Age)
{
    // کامپایلر این را تولید می‌کند:
    // protected virtual Person Clone() => new Person(this);
}

// کلاس مشتق شده
public record Student(string Name, int Age, string University) : Person(Name, Age)
{
    // 1. Copy Constructor سفارشی در Derived
    protected Student(Student original) : base(original) // فراخوانی Copy Constructor کلاس Base
    {
        this.University = original.University;
    }

    // 2. بازنویسی متد Clone برای حفظ نوع Derived
    protected override Student Clone() => new Student(this);
}
```

### چرا این معماری حیاتی است؟
فرض کنید کد زیر را اجرا کنید:

```csharp
Person person = new Student("Ali", 20, "Sharif University");
Person newPerson = person with { Age = 21 };

Console.WriteLine(newPerson.GetType().Name); 
// خروجی: Student (نه Person!)
```

**چه اتفاقی افتاد؟**
1. عبارت `with` متد `Clone()` را روی `person` صدا می‌زند.
2. چون `person` در واقع یک `Student` است و متد `Clone` در کلاس `Student` با `override` بازنویسی شده است، متد `Clone` کلاس `Student` اجرا می‌شود.
3. متد `Clone` کلاس `Student`، Copy Constructor کلاس `Student` را صدا می‌زند.
4. Copy Constructor کلاس `Student`، با استفاده از `: base(original)`، Copy Constructor کلاس `Person` را صدا می‌زند تا State کلاس Base کپی شود.
5. در نهایت، State کلاس Derived (مثل `University`) کپی شده و یک شیء جدید از نوع `Student` با سن 21 سال برگردانده می‌شود.

---

## ۱۰. مثال کامل و یکپارچه

```csharp
using System;

// رکورد پایه
public record Animal(string Name, int Legs)
{
    // اگر بخواهیم Deep Copy کنیم، Copy Constructor را می‌نویسیم
    // در اینجا Shallow Copy است چون پراپرتی‌ها Value Type هستند.
}

// رکورد مشتق شده
public record Dog(string Name, int Legs, string Breed) : Animal(Name, Legs)
{
    // Copy Constructor برای کپی کردن State کلاس Derived
    protected Dog(Dog original) : base(original)
    {
        this.Breed = original.Breed;
    }

    // بازنویسی Clone برای حفظ نوع Dog هنگام استفاده از with
    protected override Dog Clone() => new Dog(this);
}

public class Program
{
    public static void Main()
    {
        Animal myDog = new Dog("Rex", 4, "German Shepherd");
        
        // استفاده از with expression
        Animal updatedDog = myDog with { Legs = 3 };

        Console.WriteLine($"Type: {updatedDog.GetType().Name}"); // خروجی: Dog
        Console.WriteLine($"Name: {updatedDog.Name}, Legs: {updatedDog.Legs}, Breed: {((Dog)updatedDog).Breed}");
        // خروجی: Name: Rex, Legs: 3, Breed: German Shepherd
    }
}
```

---

## خلاصه و نتیجه‌گیری

*   **Copy Constructor** مکانیزمی است که State یک شیء را در یک شیء جدید کپی می‌کند و در رکوردها به صورت `protected` تولید می‌شود.
*   **Clone** یک متد مجازی است که کامپایلر تولید می‌کند و وظیفه دارد با فراخوانی Copy Constructor، یک کپی از شیء برگرداند.
*   عبارت **`with`** در واقع ترکیبی از فراخوانی `Clone` و اعمال Object Initializer است.
*   رکوردها به صورت پیش‌فرض **Shallow Copy** انجام می‌دهند. برای Deep Copy باید Copy Constructor را دستی بازنویسی کنید.
*   در **ارث‌بری**، بازنویسی متد `Clone` و فراخوانی `base(original)` در Copy Constructor کلاس Derived، تضمین می‌کند که نوع واقعی شیء در حین کپی شدن حفظ شود (Polymorphic Copy).

---

## منابع رسمی

برای مطالعه بیشتر و اطمینان از صحت مفاهیم، می‌توانید به منابع رسمی زیر مراجعه کنید:

1. **Microsoft Docs - Record Types:**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
   *(بخش‌های Copy and Clone Record Types و Inheritance)*

2. **C# Language Specification - Records:**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-14.0/records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-14.0/records) 
   *(یا نسخه‌های C# 9.0 / 10.0 در مخزن GitHub زبان سی‌شارپ: [csharplang/spec/records.md](https://github.com/dotnet/csharplang/blob/main/spec/records.md))*

3. **Microsoft Docs - with expression:**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression)