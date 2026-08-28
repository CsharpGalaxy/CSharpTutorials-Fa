# آموزش جامع Inheritance در Recordهای C# (از مقدماتی تا پیشرفته)

فیلد `record` در سی‌شارپ (از نسخه 9) برای ایجاد اشیایی طراحی شده که رفتار آن‌ها بیشتر شبیه به **مقادیر (Values)** است تا اشیاء کلاسیک. با این حال، `record class` از **وراثت (Inheritance)** به طور کامل پشتیبانی می‌کند. 

در این آموزش، از مفاهیم پایه شروع کرده و به بررسی عمیق (Under the hood) کامپایلر و مدیریت State در وراثت چندسطحی می‌پردازیم.

---

## ۱. مفاهیم پایه: Base و Derived Record

در سی‌شارپ، کلمه کلیدی `record` به صورت پیش‌فرض یک `record class` است. برای ایجاد وراثت، Derived Record باید پارامترهای Positional خود را به Base Record پاس دهد.

### مثال پایه
```csharp
// Base Record
public record Person(string FirstName, string LastName);

// Derived Record
public record Student(string FirstName, string LastName, int Grade) 
    : Person(FirstName, LastName);
```

**نکته مهم در Positional Parameters:**
در Derived Record، شما **مجبورید** پارامترهایی که مربوط به Base هستند را در لیست پارامترهای Derived بیاورید و آن‌ها را به Constructor کلاس Base ارسال کنید.

---

## ۲. جادوی کامپایلر: اعضای تولید شده (Compiler-Generated Members)

وقتی یک Derived Record می‌سازید، کامپایلر سی‌شارپ اعضای خاصی را برای مدیریت وراثت، برابری و کپی کردن تولید می‌کند. بیایید آن‌ها را بررسی کنیم:

### الف) Copy Constructor و نقش Base
برای هر Record، کامپایلر یک **Copy Constructor** محافظت‌شده (Protected) تولید می‌کند. در Derived Record، این Constructor باید حتماً Copy Constructor کلاس Base را صدا بزند.

```csharp
// معادل کدی که کامپایلر برای Student تولید می‌کند:
protected Student(Student original) : base(original)
{
    // کپی کردن State مخصوص به Student
    this.Grade = original.Grade;
}
```
**نقش Base:** صدا زدن `base(original)` تضمین می‌کند که تمام State (داده‌های) مربوط به `Person` نیز به درستی در شیء جدید کپی شود.

### ب) EqualityContract
این یک Property مجازی (Virtual) است که `Type` خود کلاس را برمی‌گرداند. 
**چرا مهم است؟** برای اینکه یک `Student` با یک `Person` (حتی اگر نام و فامیل یکسانی داشته باشند) برابر (`==`) در نظر گرفته **نشود**.
```csharp
protected virtual Type EqualityContract => typeof(Student);
```

### ج) متدهای Equals و GetHashCode
کامپایلر متد `Equals(T)` را Override می‌کند. الگوریتم آن به این شکل است:
1. بررسی می‌کند که آیا `EqualityContract` هر دو شیء یکسان است؟ (جلوگیری از برابری Base و Derived).
2. متد `Equals` کلاس **Base** را صدا می‌زند (برای بررسی State پایه).
3. State مخصوص به **Derived** را بررسی می‌کند.

```csharp
// الگوریتم تقریبی Equals در Derived:
public virtual bool Equals(Student? other)
{
    return other is not null 
        && this.EqualityContract == other.EqualityContract 
        && base.Equals(other) // بررسی State کلاس Person
        && this.Grade == other.Grade; // بررسی State کلاس Student
}
```
متد `GetHashCode` نیز ترکیبی از HashCode کلاس Base و Derived را محاسبه می‌کند.

### د) متد ToString
متد `ToString` در Derived Record، به صورت خودکار تمام Propertyهای Base و Derived را به صورت `Key-Value` چاپ می‌کند.
```csharp
// خروجی: Student { FirstName = Ali, LastName = Ahmadi, Grade = 18 }
```

---

## ۳. عبارت `with` و نقش Base در Copy

عبارت `with` برای ایجاد یک کپی از Record با تغییر یک یا چند Property استفاده می‌شود.

```csharp
var student1 = new Student("Ali", "Ahmadi", 18);
var student2 = student1 with { Grade = 20 };
```

**زیر کاپوت (Under the hood) چه اتفاقی می‌افتد؟**
1. کامپایلر متد مجازی `Clone()` را صدا می‌زند.
2. متد `Clone()` یک کپی سطحی (Shallow Copy) از شیء می‌سازد.
3. سپس مقدار جدید (`Grade = 20`) روی کپی اعمال می‌شود.

**نقش Base در اینجا:** چون `Clone()` در Derived Record پیاده‌سازی شده و از Copy Constructor استفاده می‌کند، و Copy Constructor نیز `base(original)` را صدا می‌زند، **تمام State کلاس‌های والد به درستی و بدون از دست رفتن داده در شیء جدید کپی می‌شود.**

---

## ۴. مثال چندسطحی (Multi-Level Inheritance) و مدیریت State

بیایید یک سناریوی سه سطحی را بررسی کنیم تا ببینیم هر سطح چگونه State خود را مدیریت می‌کند.

```csharp
// سطح ۱: Base Record
public record Person(string Name, int Age);

// سطح ۲: Derived Record
public record Employee(string Name, int Age, string Department, decimal Salary) 
    : Person(Name, Age);

// سطح ۳: More Derived Record
public record Manager(string Name, int Age, string Department, decimal Salary, int TeamSize) 
    : Employee(Name, Age, Department, Salary);
```

### مدیریت State در هر سطح:
1. **سطح Person:** فقط مسئول `Name` و `Age` است.
2. **سطح Employee:** مسئول `Department` و `Salary` است. اما چون از Person ارث می‌برد، **مجبور است** `Name` و `Age` را در پارامترهای خود دریافت کرده و به `base` پاس دهد.
3. **سطح Manager:** مسئول `TeamSize` است. تمام پارامترهای قبلی را دریافت و به `Employee` پاس می‌دهد.

### زنجیره Copy Constructor در وراثت چندسطحی:
وقتی از `Manager` یک کپی با `with` می‌گیریم، زنجیره به این شکل اجرا می‌شود:
```text
Manager Copy Ctor 
   -> صدا زدن Employee Copy Ctor 
      -> صدا زدن Person Copy Ctor
```
در هر مرحله، State مخصوص به همان سطح کپی می‌شود. این طراحی تضمین می‌کند که **هیچ داده‌ای در هیچ سطحی از وراثت گم نمی‌شود.**

### تست برابری در وراثت چندسطحی:
```csharp
var emp = new Employee("Sara", 30, "IT", 5000);
var mgr = new Manager("Sara", 30, "IT", 5000, 5);

Console.WriteLine(emp == mgr); // خروجی: False
```
حتی اگر تمام داده‌های مشترک یکسان باشند، به دلیل تفاوت در `EqualityContract` (یکی `typeof(Employee)` و دیگری `typeof(Manager)` است)، این دو رکورد **هرگز برابر نیستند**.

---

## ۵. محدودیت Record Struct در زمینه Inheritance

در سی‌شارپ ۱۰، `record struct` معرفی شد. اما یک محدودیت بزرگ وجود دارد:
**`record struct` نمی‌تواند از هیچ کلاس یا رکورد دیگری ارث‌بری کند.**

```csharp
public record struct Point2D(int X, int Y);

// خطای کامپایلر:
public record struct Point3D(int X, int Y, int Z) : Point2D(X, Y); 
// Error: Structs cannot inherit from other structs or classes.
```

### چرا این محدودیت وجود دارد؟
1. **ماهیت Value Type:** استراکت‌ها Value Type هستند. وراثت در Value Typeها باعث مشکلات جدی در مدیریت حافظه (Boxing و Allocation) می‌شود.
2. **اصل Liskov Substitution:** اگر استراکت‌ها وراثت داشته باشند، یک `Point3D` باید بتواند جایگزین `Point2D` شود، اما چون سایز حافظه آن‌ها متفاوت است، در آرایه‌ها و ساختارهای Value Type دچار شکست می‌شویم.
3. **EqualityContract در Struct:** در `record struct`، `EqualityContract` وجود ندارد (چون sealed هستند و وراثتی در کار نیست). برابری صرفاً بر اساس Type و فیلدها بررسی می‌شود.

*نکته: `record struct` به صورت پیش‌فرض `sealed` است و حتی نمی‌توان آن را `abstract` یا `virtual` کرد.*

---

## ۶. منابع رسمی و مراجع

برای مطالعه بیشتر و اطمینان از صحت مطالب، می‌توانید به منابع رسمی زیر مراجعه کنید:

1. **Microsoft C# Documentation - Records:**
   * [Record types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
   * [How to use records for immutable data](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)
   * *بخش Inheritance in records* در داکیومنت مایکروسافت به طور خاص به نحوه پاس دادن پارامترها به Base و رفتار `with` اشاره کرده است.

2. **C# Language Specification (ECMA-334 / GitHub Roslyn):**
   * [C# Language Specification - Records Chapter](https://github.com/dotnet/csharplang/blob/main/spec/records.md)
   * در این داکیومنت، بخش‌های **Copy constructor** و **Equality members** دقیقاً توضیح می‌دهند که کامپایلر چگونه متدهای `Equals`، `GetHashCode` و `EqualityContract` را برای سلسله مراتب وراثت تولید می‌کند.

3. **Source Code کامپایلر (Roslyn):**
   * اگر به کد دقیق تولید شده توسط کامپایلر علاقه‌مندید، می‌توانید از ابزار [SharpLab](https://sharplab.io/) استفاده کنید و کد Record خود را به IL یا C# دیکامپایل شده تبدیل کنید تا جادوی `EqualityContract` و `Clone` را با چشم ببینید.