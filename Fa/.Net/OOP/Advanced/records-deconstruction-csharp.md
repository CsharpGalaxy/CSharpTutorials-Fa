

## فهرست مطالب

1. [مقدمه: Records چیست؟](#1-مقدمه-records-چیست)  
2. [تفاوت Records با Classes و Structs](#2-تفاوت-records-با-classes-و-structs)  
3. [Deconstruction چیست؟](#3-deconstruction-چیست)  
4. [ترکیب Records و Deconstruction](#4-ترکیب-records-و-deconstruction)  
5. [نحوه پیاده‌سازی Deconstruction در Records](#5-نحوه-پیادهسازی-deconstruction-در-records)  
6. [کاربردهای عملی](#6-کاربردهای-عملی)  
7. [نکات پیشرفته و بهترین روش‌ها (Best Practices)](#7-نکات-پیشرفته-و-بهترین-روشها-best-practices)  
8. [جمع‌بندی](#8-جمعبندی)  
9. [منابع معتبر](#9-منابع-معتبر)

---

## 1. مقدمه: Records چیست؟

در C# 9.0 (انتشار یافته در سال 2020)، مایکروسافت یک نوع داده جدید به نام **`record`** معرفی کرد. هدف اصلی این نوع داده، تسهیل کار با **داده‌های غیرقابل تغییر (immutable data)** و **مقایسه مبتنی بر محتوا (value-based equality)** است.

Records به‌طور خودکار:
- متد‌های `Equals()`, `GetHashCode()`, و `ToString()` را بر اساس مقادیر فیلدها پیاده‌سازی می‌کنند.
- از **value-based equality** پشتیبانی می‌کنند (برخلاف کلاس‌های معمولی که reference-based هستند).
- از **with-expressions** برای ایجاد کپی‌های اصلاح‌شده پشتیبانی می‌کنند.

### مثال ساده:

```csharp
public record Person(string FirstName, string LastName);
```

این یک **record با سینتکس primary constructor** است که در C# 9+ معرفی شد. کامپایلر به‌طور خودکار پروپرتی‌ها، متد‌های مقایسه، و سازنده را تولید می‌کند.

---

## 2. تفاوت Records با Classes و Structs

| ویژگی | Class | Struct | Record |
|--------|-------|--------|--------|
| نوع داده | Reference | Value | Reference (پیش‌فرض) |
| Equality | Reference-based | Value-based (با override) | **Value-based** (به‌صورت پیش‌فرض) |
| Immutability | نیاز به پیاده‌سازی دستی | نیاز به پیاده‌سازی دستی | **پشتیبانی خودکار** (با init-only یا readonly) |
| With-expression | ❌ | ❌ | ✅ |
| Deconstruction | نیاز به پیاده‌سازی دستی | نیاز به پیاده‌سازی دستی | **قابل پیاده‌سازی آسان** |

> 💡 نکته: از C# 10 به بعد، می‌توانید از `record struct` نیز استفاده کنید که ترکیبی از ویژگی‌های record و struct است.

---

## 3. Deconstruction چیست؟

**Deconstruction** یعنی شکستن یک شیء به چندین متغیر مستقل. این ویژگی از C# 7.0 وجود داشته، اما با records ترکیب بسیار زیبایی پیدا کرده است.

### مثال ساده با Tuple:

```csharp
var (name, age) = ("Ali", 30);
```

اما برای کلاس‌ها یا records، باید یک متد `Deconstruct` تعریف کنید.

---

## 4. ترکیب Records و Deconstruction

Records به‌طور طبیعی برای **داده‌محور بودن** طراحی شده‌اند، پس Deconstruction برای آن‌ها بسیار منطقی است. مثلاً وقتی یک record دارای چند فیلد است، می‌توانید مستقیماً آن را به متغیرهای جداگانه "بُریزید".

---

## 5. نحوه پیاده‌سازی Deconstruction در Records

### روش ۱: استفاده از Primary Constructor (ساده‌ترین روش)

```csharp
public record Point(int X, int Y);

// استفاده:
var point = new Point(10, 20);
var (x, y) = point; // ✅ کار می‌کند!
Console.WriteLine($"X: {x}, Y: {y}");
```

> 🔥 **نکته مهم**: وقتی از سینتکس primary constructor استفاده می‌کنید (`record Person(string Name, int Age)`), کامپایلر **به‌صورت خودکار** یک متد `Deconstruct` برای شما تولید می‌کند!

### روش ۲: پیاده‌سازی دستی متد `Deconstruct`

اگر نیاز به ترتیب خاص یا فیلتر کردن دارید:

```csharp
public record Employee(string FirstName, string LastName, int Salary)
{
    public void Deconstruct(out string fullName, out int salary)
    {
        fullName = $"{FirstName} {LastName}";
        salary = Salary;
    }
}

// استفاده:
var emp = new Employee("Reza", "Mohammadi", 5000);
var (name, salary) = emp;
Console.WriteLine($"{name} earns {salary}");
```

> ⚠️ توجه: متد `Deconstruct` باید **void** باشد و پارامترهایش **out** باشند.

---

## 6. کاربردهای عملی

### ✅ بازگرداندن چند مقدار از متد

```csharp
public static Person GetPerson() => new("Sara", "Ahmadi");

// در جای دیگر:
var (first, last) = GetPerson();
```

### ✅ پردازش لیست از Records

```csharp
var people = new List<Person>
{
    new("Ali", "Rahimi"),
    new("Narges", "Kazemi")
};

foreach (var (first, last) in people)
{
    Console.WriteLine($"{first} {last}");
}
```

### ✅ تبدیل به Tuple یا ساختارهای دیگر

```csharp
var points = new[] { new Point(1,2), new Point(3,4) };
var tuples = points.Select(p => (p.X, p.Y)).ToArray();
```

---

## 7. نکات پیشرفته و بهترین روش‌ها (Best Practices)

### ✅ از Primary Constructor برای Records ساده استفاده کنید  
→ کد کمتر، خوانا‌تر، و Deconstruction خودکار.

### ✅ برای Records پیچیده، `Deconstruct` را دستی پیاده‌سازی کنید  
→ مثلاً وقتی می‌خواهید فقط بخشی از داده را برگردانید.

### ❌ از Deconstruction برای Records با فیلدهای زیاد خودداری کنید  
→ خوانایی کد کاهش می‌یابد. مثلاً اگر ۱۰ فیلد دارید، بهتر است از نام‌گذاری صریح استفاده کنید.

### ✅ از `record struct` در C# 10+ برای داده‌های کوچک و مکرر استفاده کنید  
→ کاهش فشار روی Garbage Collector.

### ✅ همیشه ترتیب پارامترهای `Deconstruct` را مستند کنید  
→ چون ترتیب مهم است و تغییر آن می‌تواند باعث باگ شود.

---

## 8. جمع‌بندی

- **Records** در C# 9+ ابزاری قدرتمند برای مدل‌سازی داده‌های غیرقابل تغییر هستند.
- **Deconstruction** امکان استخراج راحت مقادیر از یک record را فراهم می‌کند.
- با **Primary Constructor**، Deconstruction به‌صورت خودکار فعال می‌شود.
- این ترکیب، کد را **خوانا‌تر، کوتاه‌تر و ایمن‌تر** می‌کند — ویژگی‌هایی که در برنامه‌نویسی مدرن OOP بسیار ارزشمند هستند.

---

## 9. منابع معتبر

1. **مستندات رسمی مایکروسافت (Microsoft Learn)**  
   - [Records in C#](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)  
   - [Deconstructing tuples and other types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct)

2. **C# Language Proposal (GitHub - dotnet/csharplang)**  
   - [Records Proposal](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-9.0/records.md)  
   - [Deconstruction Proposal](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-7.0/deconstruction.md)

3. **کتاب‌های معتبر**  
   - *C# 10 and .NET 6* – Mark J. Price (فصل 7: Working with Data)  
   - *Effective C#* – Bill Wagner (ویرایش 3rd، بخش Records و Immutability)

4. **وبلاگ‌های معتبر**  
   - [Stephen Toub – Records in C# 9](https://devblogs.microsoft.com/dotnet/welcome-to-c-9-0/)  
   - [Jon Skeet – C# in Depth (Records section)](https://csharpindepth.com/)
!
