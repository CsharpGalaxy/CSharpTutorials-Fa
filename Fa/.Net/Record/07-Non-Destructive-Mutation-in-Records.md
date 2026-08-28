# 📘 آموزش کامل تعریف Record در C#

**Record**ها در C# 9 معرفی شدند و نوعی reference type (یا value type در C# 10 با `record struct`) هستند که برای مدل‌سازی **داده‌محور (data-centric)** طراحی شده‌اند. ویژگی کلیدی آن‌ها این است که کامپایلر به صورت خودکار **value equality**، `ToString()`، `with` expression و `Deconstruct` را برایشان تولید می‌کند.

---

## ۱. سه روش اصلی تعریف Record

### 🔹 روش اول: تعریف ساده (بدون Member)

```csharp
public record Person;
```

این کوتاه‌ترین شکل است. کامپایلر یک record خالی تولید می‌کند که هیچ property یا field ندارد.

**کاربرد:** به عنوان پایه‌ای برای ارث‌بری یا marker type.

---

### 🔹 روش دوم: تعریف با Body (سبک کلاسیک)

```csharp
public record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}
```

در این روش، propertyها را داخل بدنه تعریف می‌کنید. می‌توانید از `init` (immutable پس از ساخت) یا `set` استفاده کنید.

**کاربرد:** زمانی که می‌خواهید کنترل کامل روی propertyها داشته باشید (مثلاً validation).

---

### 🔹 روش سوم: Primary Constructor (سبک مدرن و رایج)

```csharp
public record Person(string Name, int Age);
```

کامپایلر به صورت خودکار:
- یک constructor با پارامترهای `Name` و `Age` می‌سازد.
- برای هر پارامتر، یک **public init-only property** هم‌نام تولید می‌کند.
- `Deconstruct` و `Equals` را بر اساس این propertyها پیاده می‌کند.

**کاربرد:** DTOها، ViewModelها، و هر جایی که داده‌ها immutable هستند. این محبوب‌ترین روش است.

---

## ۲. تفاوت سه Syntax در یک نگاه

| ویژگی | `record Person;` | `record Person { ... }` | `record Person(string Name);` |
|---|---|---|---|
| Property خودکار | ❌ | ❌ | ✅ (برای هر پارامتر) |
| Constructor خودکار | ✅ (بدون پارامتر) | ❌ (باید دستی بنویسید) | ✅ |
| Deconstruct خودکار | ❌ | ❌ | ✅ |
| کنترل روی property | — | کامل | فقط با `init`/`set` |
| مقداردهی اولیه | — | ✅ در initializer | ✅ در initializer |

### مثال کامل مقایسه‌ای:

```csharp
// روش 1: خالی
public record EmptyRecord;

// روش 2: با body
public record PersonClassic
{
    public string Name { get; init; }
    public int Age { get; init; }
}

// روش 3: Primary Constructor
public record PersonModern(string Name, int Age);
```

استفاده:

```csharp
var p1 = new EmptyRecord();
var p2 = new PersonClassic { Name = "Ali", Age = 30 };
var p3 = new PersonModern("Ali", 30);
```

---

## ۳. افزودن Property، Field، Method و Constructor

حتی با Primary Constructor هم می‌توانید body اضافه کنید:

```csharp
public record Person(string FirstName, string LastName)
{
    // Property محاسبه‌شده
    public string FullName => $"{FirstName} {LastName}";

    // Property با مقدار اولیه
    public string Country { get; init; } = "Iran";

    // Field (private)
    private int _internalCode = 100;

    // Method
    public string GetGreeting() => $"Hello, {FirstName}!";

    // Constructor اضافی (باید primary را صدا بزند)
    public Person(string fullName) 
        : this(
            fullName.Split(' ')[0], 
            fullName.Split(' ')[1])
    { }
}
```

> ⚠️ **نکته مهم:** در Primary Constructor، نمی‌توانید constructor بدون پارامتر (parameterless) تعریف کنید، چون کامپایلر خودش یکی می‌سازد.

---

## ۴. پیاده‌سازی Interface

```csharp
public interface IAuditable
{
    DateTime CreatedAt { get; }
}

public record AuditLog(string Action, DateTime CreatedAt) 
    : IAuditable;
```

یا چندین interface:

```csharp
public record User(string Name, string Email) 
    : IAuditable, IValidatable
{
    public bool Validate() => Email.Contains("@");
}
```

---

## ۵. ارث‌بری (Inheritance)

Recordها می‌توانند از یک record دیگر ارث ببرند (نه از کلاس معمولی):

```csharp
public record Animal(string Name, int Legs);

public record Dog(string Name, int Legs, string Breed) 
    : Animal(Name, Legs);
```

استفاده:

```csharp
var dog = new Dog("Rex", 4, "Labrador");
Console.WriteLine(dog); 
// Dog { Name = Rex, Legs = 4, Breed = Labrador }
```

> ⚠️ **محدودیت:** یک record نمی‌تواند هم‌زمان از یک کلاس و یک record ارث ببرد. فقط **یک record base** مجاز است.

### ارث‌بری با sealed:

```csharp
public sealed record Point(int X, int Y); // دیگر نمی‌تواند base باشد
```

---

## ۶. Modifierهای قابل استفاده

### Modifierهای سطح Type:

| Modifier | توضیح |
|---|---|
| `public` | دسترسی از همه جا |
| `internal` | فقط در همان assembly (پیش‌فرض) |
| `private` | فقط داخل کلاس enclosing |
| `protected` | فقط در کلاس و derivedها |
| `sealed` | جلوگیری از ارث‌بری بیشتر |
| `abstract` | نمی‌توان نمونه ساخت |
| `partial` | تقسیم تعریف در چند فایل |
| `static` | ❌ **مجاز نیست** برای record |

### مثال‌ها:

```csharp
// Abstract record
public abstract record Shape(string Color);

// Sealed record
public sealed record Circle(double Radius) : Shape("Red");

// Partial record (معمولاً برای source generatorها)
public partial record LogEntry(string Message);

// Nested private record
public class OrderService
{
    private record OrderItem(string Product, int Qty);
}
```

---

## ۷. Access Modifierها برای Memberها

درون record می‌توانید از تمام access modifierهای معمول استفاده کنید:

```csharp
public record Customer(string Id)
{
    // Public
    public string PublicInfo => $"Customer {Id}";
    
    // Private field
    private string _secret = "xyz";
    
    // Protected method (برای derivedها)
    protected string GetSecret() => _secret;
    
    // Internal
    internal void Log() => Console.WriteLine("Logged");
    
    // Init-only property
    public string Email { get; init; } = "";
    
    // Required property (C# 11+)
    public required string Name { get; init; }
}
```

---

## ۸. جدول مقایسه نهایی سه روش

### ✅ روش ۱: `public record Person;`
- **مزایا:** کوتاه، سریع
- **معایب:** هیچ داده‌ای ندارد
- **کاربرد:** Marker type، base برای ارث‌بری، placeholder

### ✅ روش ۲: `public record Person { ... }`
- **مزایا:** کنترل کامل روی propertyها، validation، مقدار اولیه
- **معایب:** پرحجم‌تر، باید constructor را دستی نوشت
- **کاربرد:** زمانی که propertyها نیاز به logic دارند

### ✅ روش ۳: `public record Person(string Name);`
- **مزایا:** بسیار مختصر، Deconstruct خودکار، equality خودکار
- **معایب:** همه propertyها init-only هستند (مگر اینکه دوباره تعریف کنید)
- **کاربرد:** DTO، modelهای immutable، ۹۰٪ موارد واقعی

---

## ۹. تمرین‌های عملی 🏋️

### تمرین ۱: ساخت DTO ساده
یک record به نام `Product` با propertyهای `Id` (int)، `Name` (string)، `Price` (decimal) با Primary Constructor بسازید.

<details><summary>💡 پاسخ</summary>

```csharp
public record Product(int Id, string Name, decimal Price);
```
</details>

---

### تمرین ۲: ارث‌بری
یک record abstract به نام `Vehicle` بسازید با propertyهای `Brand` و `Year`. سپس دو record `Car` و `Motorcycle` از آن ارث ببرند.

<details><summary>💡 پاسخ</summary>

```csharp
public abstract record Vehicle(string Brand, int Year);
public record Car(string Brand, int Year, int Doors) : Vehicle(Brand, Year);
public record Motorcycle(string Brand, int Year, bool HasSidecar) : Vehicle(Brand, Year);
```
</details>

---

### تمرین ۳: پیاده‌سازی Interface + Method
رکورد `BankAccount` با propertyهای `AccountNumber` و `Balance`. یک interface `IWithdrawable` تعریف کنید و متد `Withdraw(decimal amount)` را در record پیاده کنید که اگر موجودی کافی نبود، exception بزند.

<details><summary>💡 پاسخ</summary>

```csharp
public interface IWithdrawable
{
    decimal Withdraw(decimal amount);
}

public record BankAccount(string AccountNumber, decimal Balance) 
    : IWithdrawable
{
    public decimal Withdraw(decimal amount)
    {
        if (amount > Balance)
            throw new InvalidOperationException("Insufficient funds");
        return Balance - amount;
    }
}
```
</details>

---

### تمرین ۴: استفاده از `with` expression
رکورد `User` را بسازید و با استفاده از `with` یک کپی با `Email` تغییر یافته ایجاد کنید.

<details><summary>💡 پاسخ</summary>

```csharp
public record User(string Name, string Email);

var user1 = new User("Ali", "ali@test.com");
var user2 = user1 with { Email = "ali.new@test.com" };
```
</details>

---

### تمرین ۵ (پیشرفته): ترکیب همه
یک سیستم لاگ بسازید:
- Interface به نام `ILog` با متد `Format()`
- Record abstract به نام `LogEntry` که `ILog` را پیاده کند
- دو record sealed به نام `InfoLog` و `ErrorLog` که از `LogEntry` ارث ببرند
- `ErrorLog` یک property اضافی `Exception` داشته باشد

<details><summary>💡 پاسخ</summary>

```csharp
public interface ILog
{
    string Format();
}

public abstract record LogEntry(DateTime Timestamp, string Message) : ILog
{
    public virtual string Format() => $"[{Timestamp}] {Message}";
}

public sealed record InfoLog(DateTime Timestamp, string Message) 
    : LogEntry(Timestamp, Message);

public sealed record ErrorLog(DateTime Timestamp, string Message, Exception Error) 
    : LogEntry(Timestamp, Message)
{
    public override string Format() => 
        $"[ERROR {Timestamp}] {Message} | {Error.Message}";
}
```
</details>

---

## 📚 منابع رسمی

1. **Microsoft Learn — Records (C# Reference)**  
   🔗 https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

2. **Microsoft Learn — How to define records (Tutorial)**  
   🔗 https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records

3. **C# Language Specification — Records (ECMA/ISO)**  
   🔗 https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#155-record-declarations

4. **C# 9 Feature Specification — Records (GitHub dotnet/roslyn)**  
   🔗 https://github.com/dotnet/csharplang/blob/main/proposals/csharp-9.0/records.md

5. **Microsoft Learn — with expression**  
   🔗 https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression

---

## 🎯 خلاصه سریع

| هدف | روش پیشنهادی |
|---|---|
| DTO ساده | `record Person(string Name, int Age);` |
| نیاز به validation | `record Person { ... }` با body |
| Marker / Base type | `record Base;` یا `abstract record Base;` |
| جلوگیری از ارث‌بری | `sealed record Point(int X, int Y);` |
| Immutable data | Primary Constructor + `init` |

موفق باشید! 🚀