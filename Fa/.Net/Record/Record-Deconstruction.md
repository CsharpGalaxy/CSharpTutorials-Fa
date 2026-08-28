# Deconstruction در Recordها در C#

این آموزش به صورت گام‌به‌گام از مفاهیم پایه تا نکات پیشرفته درباره **Deconstruction** در **Recordها** را پوشش می‌دهد.

---

## ۱. Deconstruction چیست؟

**Deconstruction** (تخریب / بازسازی معکوس) فرآیندی است که در آن یک شیء به اجزای سازنده‌اش تجزیه می‌شود. به عبارت ساده‌تر، شما می‌توانید یک شیء را به متغیرهای جداگانه‌ای که فیلدها یا Propertyهای آن را نگهداری می‌کنند، «بشکنید».

این ویژگی از C# 7.0 معرفی شد و با ورود **Recordها** در C# 9.0 کاربرد بسیار گسترده‌تری پیدا کرد.

```csharp
var point = new Point(10, 20);
var (x, y) = point;  // Deconstruction
```

---

## ۲. Deconstructor چیست؟

**Deconstructor** یک متد نمونه (instance method) با نام `Deconstruct` است که پارامترهای خروجی (`out`) دارد و وظیفه‌ی آن استخراج مقادیر از یک شیء است.

امضای کلی آن به این صورت است:

```csharp
public void Deconstruct(out T1 p1, out T2 p2, ...)
```

نکات کلیدی:
- نام متد باید دقیقاً `Deconstruct` باشد (حساس به بزرگی و کوچکی حروف).
- همه‌ی پارامترها باید `out` باشند.
- می‌تواند **overload** شود (چندین `Deconstruct` با تعداد پارامترهای متفاوت).
- مقدار بازگشتی آن `void` است.

---

## ۳. ارتباط Deconstruction با Positional Record

**Positional Record** (رکورد موقعیتی) رکوردی است که با **Primary Constructor** تعریف می‌شود:

```csharp
public record Person(string Name, int Age);
```

وقتی شما یک Positional Record تعریف می‌کنید، کامپایلر C# به صورت خودکار:
1. Propertyهای `init-only` برای هر پارامتر می‌سازد.
2. یک Constructor با همان پارامترها تولید می‌کند.
3. **یک متد `Deconstruct` با همان ترتیب و نوع پارامترها تولید می‌کند.**

پس بین Positional Record و Deconstruction یک ارتباط دوطرفه وجود دارد:
- **Constructor** → شیء را می‌سازد.
- **Deconstructor** → شیء را به اجزایش تجزیه می‌کند.

---

## ۴. Deconstructor تولیدشده توسط Compiler

برای رکورد زیر:

```csharp
public record Person(string Name, int Age);
```

کامپایلر به صورت خودکار چیزی شبیه کد زیر تولید می‌کند:

```csharp
public record Person(string Name, int Age)
{
    // Propertyها توسط کامپایلر ساخته می‌شوند
    public string Name { get; init; }
    public int Age { get; init; }

    // Deconstructor خودکار
    public void Deconstruct(out string name, out int age)
    {
        name = Name;
        age = Age;
    }
}
```

> 💡 ترتیب پارامترها در `Deconstruct` دقیقاً همان ترتیب پارامترهای Primary Constructor است.

---

## ۵. مثال ساده

```csharp
public record Person(string Name, int Age);

var person = new Person("Abolfazl", 25);

// Deconstruction
var (name, age) = person;

Console.WriteLine(name); // Abolfazl
Console.WriteLine(age);  // 25
```

### 🔍 پشت صحنه چه اتفاقی می‌افتد؟

کامپایلر کد بالا را به شکل زیر بازنویسی می‌کند:

```csharp
Person person = new Person("Abolfazl", 25);

string name;
int age;
person.Deconstruct(out name, out age);
```

یعنی:
1. متغیرهای `name` و `age` از روی نوع پارامترهای `Deconstruct` استنتاج می‌شوند (Type Inference).
2. متد `Deconstruct` روی شیء `person` فراخوانی می‌شود.
3. مقادیر از طریق پارامترهای `out` به متغیرها منتقل می‌شوند.

---

## ۶. چند Property

اگر رکورد شما Propertyهای بیشتری داشته باشد، `Deconstruct` همه‌ی آن‌ها را برمی‌گرداند:

```csharp
public record Employee(string Name, int Age, string Department, decimal Salary);

var emp = new Employee("Sara", 30, "IT", 2500);

var (name, age, dept, salary) = emp;

Console.WriteLine($"{name}, {age}, {dept}, {salary}");
// Sara, 30, IT, 2500
```

---

## ۷. استفاده از Discard (`_`)

گاهی اوقات به همه‌ی اجزا نیاز ندارید. می‌توانید با استفاده از **Discard** (`_`) بخش‌هایی را نادیده بگیرید:

```csharp
var (_, age) = person;          // فقط age
var (name, _) = person;         // فقط name
var (_, _, dept, _) = emp;      // فقط dept
```

> نکته: `_` در اینجا یک متغیر واقعی نیست؛ بلکه یک **Discard** است و کامپایلر برای آن فضای حافظه‌ای اختصاص نمی‌دهد.

---

## ۸. Deconstructor سفارشی

شما می‌توانید یک `Deconstruct` سفارشی به رکورد (یا هر کلاس دیگری) اضافه کنید. این کار در دو حالت کاربرد دارد:

### الف) افزودن `Deconstruct` به کلاسی که Record نیست

```csharp
public class Rectangle
{
    public int Width { get; init; }
    public int Height { get; init; }

    public void Deconstruct(out int width, out int height)
    {
        width = Width;
        height = Height;
    }
}

var rect = new Rectangle { Width = 10, Height = 20 };
var (w, h) = rect;
```

### ب) Overload کردن `Deconstruct` در Record

```csharp
public record Person(string FirstName, string LastName, int Age)
{
    // Deconstruct خودکار: (FirstName, LastName, Age)

    // Deconstruct سفارشی: فقط نام کامل و سن
    public void Deconstruct(out string fullName, out int age)
    {
        fullName = $"{FirstName} {LastName}";
        age = Age;
    }
}

var p = new Person("Ali", "Rezaei", 28);

var (first, last, age) = p;              // Deconstruct خودکار
var (fullName, age2) = p;                // Deconstruct سفارشی
```

کامپایلر بر اساس **تعداد و نوع متغیرهای هدف**، overload مناسب را انتخاب می‌کند.

---

## ۹. Record Class

`record` به صورت پیش‌فرض یک **Reference Type** است (معادل `record class`):

```csharp
public record Person(string Name, int Age);

var p = new Person("Nima", 35);
var (name, age) = p;
```

چون یک Reference Type است:
- روی Heap تخصیص می‌یابد.
- مقایسه بر اساس **مقدار** (value-based equality) انجام می‌شود، نه مرجع.
- `Deconstruct` به صورت خودکار تولید می‌شود.

---

## ۱۰. Record Struct

از C# 10 می‌توانید رکوردهای **Value Type** بسازید:

```csharp
public record struct Point(int X, int Y);

var pt = new Point(5, 10);
var (x, y) = pt;

Console.WriteLine($"X={x}, Y={y}"); // X=5, Y=10
```

### `readonly record struct`

اگر داده‌ها تغییرناپذیر باشند، بهتر است از `readonly record struct` استفاده کنید:

```csharp
public readonly record struct Point(int X, int Y);
```

در این حالت Propertyها به صورت `init` نخواهند بود، بلکه کاملاً **read-only** می‌شوند و `Deconstruct` نیز به همان شکل کار می‌کند.

---

## ۱۱. مثال کامل با توضیح پشت صحنه

```csharp
var person = new Person("Abolfazl", 25);
var (name, age) = person;
```

### مراحل اجرا:

| مرحله | عملیات |
|-------|--------|
| ۱ | `new Person("Abolfazl", 25)` → Constructor صدا زده می‌شود و Propertyهای `Name` و `Age` مقداردهی می‌شوند. |
| ۲ | کامپایلر نوع `name` و `age` را از امضای `Deconstruct(out string, out int)` استنتاج می‌کند. |
| ۳ | متغیرهای محلی `name` و `age` ساخته می‌شوند. |
| ۴ | `person.Deconstruct(out name, out age)` فراخوانی می‌شود. |
| ۵ | داخل `Deconstruct`: `name = this.Name; age = this.Age;` اجرا می‌شود. |

### کد معادل تولیدشده توسط کامپایلر:

```csharp
Person person = new Person("Abolfazl", 25);
string name;
int age;
person.Deconstruct(out name, out age);
```

---

## ۱۲. تفاوت Deconstruction با Constructor

| ویژگی | Constructor | Deconstructor |
|-------|-------------|---------------|
| **هدف** | ساختن یک شیء از مقادیر ورودی | استخراج مقادیر از یک شیء موجود |
| **نام** | هم‌نام با کلاس | همیشه `Deconstruct` |
| **پارامترها** | ورودی (معمولاً بدون `out`) | همه `out` |
| **مقدار بازگشتی** | خود شیء (از طریق `new`) | `void` |
| **زمان اجرا** | هنگام `new` | هنگام تجزیه به متغیرها |
| **تولید توسط کامپایلر** | برای Positional Record | برای Positional Record |
| **جهت جریان داده** | ورودی → شیء | شیء → خروجی |

### مثال مقایسه‌ای:

```csharp
// Constructor: مقادیر را می‌گیرد و شیء می‌سازد
var person = new Person("Abolfazl", 25);

// Deconstructor: شیء را می‌گیرد و مقادیر را بیرون می‌دهد
var (name, age) = person;
```

این دو عمل **معکوس یکدیگر** هستند؛ یکی شیء را می‌سازد و دیگری آن را باز می‌کند.

---

## ۱۳. نکات پیشرفته

### الف) Deconstruction در `foreach`

```csharp
var people = new[]
{
    new Person("Ali", 20),
    new Person("Sara", 25)
};

foreach (var (name, age) in people)
{
    Console.WriteLine($"{name} is {age}");
}
```

### ب) Deconstruction در `switch` expression

```csharp
var category = person switch
{
    (_, < 18) => "Minor",
    (_, >= 18) => "Adult"
};
```

### ج) Deconstruction در Tuple

```csharp
var tuple = (Name: "Reza", Age: 40);
var (n, a) = tuple; // Tuples هم از Deconstruction پشتیبانی می‌کنند
```

---

## ۱۴. منابع رسمی

برای مطالعه‌ی بیشتر می‌توانید به منابع رسمی زیر مراجعه کنید:

- **Microsoft Learn – Deconstructing tuples and types**  
  https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct

- **C# Language Reference – Deconstruct**  
  https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#how-to-use-records

- **C# Reference – Records**  
  https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

- **C# Reference – Record Struct**  
  https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record-struct

- **What's new in C# 9.0 – Record types**  
  https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#record-types

---

### خلاصه

- **Deconstruction** یعنی شکستن یک شیء به اجزای سازنده‌اش.
- این کار توسط متد `Deconstruct` با پارامترهای `out` انجام می‌شود.
- در **Positional Recordها**، کامپایلر به صورت خودکار `Deconstruct` تولید می‌کند.
- می‌توانید `Deconstruct` سفارشی یا چند overload تعریف کنید.
- این ویژگی هم در `record class` و هم در `record struct` کار می‌کند.
- Deconstructor **معکوس Constructor** است: یکی می‌سازد، دیگری تجزیه می‌کند.