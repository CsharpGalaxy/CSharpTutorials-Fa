

## 📚 فهرست مطالب

1. [مقدمه](#1-مقدمه)  
2. [Primary Constructor چیست؟](#2-primary-constructor-چیست)  
3. [تفاوت Primary Constructor با Constructor سنتی](#3-تفاوت-primary-constructor-با-constructor-سنتی)  
4. [نحوه تعریف Primary Constructor](#4-نحوه-تعریف-primary-constructor)  
5. [استفاده در کلاس‌ها و رکوردها (Records)](#5-استفاده-در-کلاس‌ها-و-رکوردها-records)  
6. [محدودیت‌ها و نکات مهم](#6-محدودیت‌ها-و-نکات-مهم)  
7. [چرا Primary Constructor مفید است؟](#7-چرا-primary-constructor-مفید-است)  
8. [مثال‌های کاربردی](#8-مثال‌های-کاربردی)  
9. [منابع معتبر](#9-منابع-معتبر)

---

## 1. مقدمه

در نسخه‌های جدید C# (به‌ویژه از C# 12 و .NET 8 به بعد)، مایکروسافت یک ویژگی جدید و قدرتمند به نام **Primary Constructor** را معرفی کرده است. این ویژگی ابتدا فقط برای `record`ها قابل استفاده بود، اما اکنون به **کلاس‌ها** و **ساختارها (structs)** نیز گسترش یافته است.

هدف اصلی Primary Constructor، **کاهش کد تکراری** و **افزایش خوانایی** در تعریف کلاس‌هایی است که نیاز به dependency injection یا مقداردهی اولیه دارند.

---

## 2. Primary Constructor چیست؟

Primary Constructor نوعی سینتکس فشرده در C# است که به شما اجازه می‌دهد **پارامترهای سازنده (constructor) را مستقیماً در تعریف کلاس** بنویسید، بدون نیاز به نوشتن یک متد سازنده جداگانه.

این پارامترها در سراسر بدنه کلاس قابل استفاده هستند و می‌توانند برای مقداردهی فیلدها، پراپرتی‌ها، یا حتی استفاده در متد‌های کلاس به کار روند.

---

## 3. تفاوت Primary Constructor با Constructor سنتی

| ویژگی | Constructor سنتی | Primary Constructor |
|--------|------------------|----------------------|
| نحوه تعریف | در بدنه کلاس به صورت متد | در هدر کلاس (همان‌جا که نام کلاس تعریف می‌شود) |
| حجم کد | بیشتر (مخصوصاً برای کلاس‌های ساده) | بسیار کمتر |
| خوانایی | متوسط | بسیار بالا |
| امکان ترکیب با constructorهای دیگر | بله | **خیر** (اگر Primary Constructor داشته باشید، نمی‌توانید constructor دیگری بنویسید — [محدودیت مهم]) |

---

## 4. نحوه تعریف Primary Constructor

### سینتکس کلی:

```csharp
public class MyClass(params);
```

### مثال ساده:

```csharp
public class Person(string firstName, string lastName)
{
    public string FirstName => firstName;
    public string LastName => lastName;

    public string FullName => $"{firstName} {lastName}";
}
```

در این مثال:
- `firstName` و `lastName` پارامترهای Primary Constructor هستند.
- در بدنه کلاس، مستقیماً به این پارامترها دسترسی داریم.
- نیازی به تعریف فیلد یا constructor جداگانه نیست.

---

## 5. استفاده در کلاس‌ها و رکوردها (Records)

### در Recordها (از C# 9):

Primary Constructor از ابتدا برای `record`ها طراحی شده بود:

```csharp
public record Person(string FirstName, string LastName);
```

این کد به‌صورت خودکار دو پراپرتی فقط-خواندنی (`init-only`) ایجاد می‌کند.

### در کلاس‌ها (C# 12+):

حالا می‌توانید همین سبک را برای کلاس‌های معمولی نیز استفاده کنید:

```csharp
public class BankAccount(string accountNumber, decimal initialBalance)
{
    private decimal balance = initialBalance;

    public void Deposit(decimal amount) => balance += amount;
    public decimal GetBalance() => balance;
}
```

> 💡 نکته: پارامترهای Primary Constructor در کلاس‌ها **فقط در بدنه کلاس قابل دسترسی هستند** و به‌صورت خودکار به پراپرتی تبدیل نمی‌شوند (برخلاف record).

---

## 6. محدودیت‌ها و نکات مهم

1. **عدم امکان تعریف constructor اضافی**:  
   اگر یک Primary Constructor دارید، **نمی‌توانید constructor دیگری بنویسید**. این یک محدودیت عمده است.

2. **پارامترها فقط در بدنه کلاس قابل دسترسی‌اند**:  
   نمی‌توانید به‌راحتی آن‌ها را به‌عنوان فیلد یا پراپرتی اکسپوز کنید (مگر آنکه خودتان دستی تعریف کنید).

3. **عدم پشتیبانی از initializer در پارامترها**:  
   نمی‌توانید مقدار پیش‌فرض برای پارامترها بگذارید (مثل `(string name = "Guest")`) — چون این کار نیاز به constructor دیگری دارد که مجاز نیست.

4. **استفاده در inheritance**:  
   کلاس فرزند می‌تواند Primary Constructor خود را داشته باشد، اما باید constructor والد را فراخوانی کند:

   ```csharp
   public class Employee(string name, int id) : Person(name)
   {
       public int Id => id;
   }
   ```

---

## 7. چرا Primary Constructor مفید است؟

- ✅ **کاهش boilerplate code** (کد تکراری)
- ✅ **ایده‌آل برای dependency injection** در برنامه‌های مدرن
- ✅ **خوانایی بالا** برای کلاس‌های داده‌محور (Data Classes)
- ✅ **هماهنگی با سبک‌های مدرن برنامه‌نویسی** (مثل functional-first در recordها)

مثال در dependency injection:

```csharp
public class OrderService(IOrderRepository repo, ILogger logger)
{
    public async Task ProcessOrder(Order order)
    {
        logger.LogInformation("Processing order...");
        await repo.SaveAsync(order);
    }
}
```

این سبک در ASP.NET Core بسیار رایج است و Primary Constructor آن را بسیار تمیزتر می‌کند.

---

## 8. مثال‌های کاربردی

### مثال 1: کلاس ساده با منطق داخلی

```csharp
public class Circle(double radius)
{
    public double Radius => radius;
    public double Area => Math.PI * radius * radius;
    public double Circumference => 2 * Math.PI * radius;
}
```

### مثال 2: استفاده در سرویس با DI

```csharp
public class EmailService(ISmtpClient client, IConfiguration config)
{
    public async Task SendEmail(string to, string subject, string body)
    {
        var from = config["Email:From"];
        await client.SendAsync(from, to, subject, body);
    }
}
```

### مثال 3: ترکیب با record برای مدل‌های داده

```csharp
public record LoginRequest(string Username, string Password);

// یا به صورت کلاس اگر نیاز به رفتار دارید:
public class LoginRequest(string username, string password)
{
    public bool IsValid() => !string.IsNullOrWhiteSpace(username) && password.Length >= 8;
}
```

---

## 9. منابع معتبر

1. **مستندات رسمی مایکروسافت (Microsoft Learn)**  
   🔗 [Primary Constructors in C# 12](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/primary-constructors)  
   🔗 [Records in C#](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)

2. **C# Language Proposal (GitHub - dotnet/csharplang)**  
   🔗 [Primary Constructors Specification](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-12-primary-constructors.md)

3. **.NET 8 Release Notes**  
   🔗 [.NET 8 What's New](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-8)

4. **Pluralsight / Microsoft Virtual Training**  
   دوره‌های رسمی C# 12 و .NET 8 در پلتفرم‌های آموزشی معتبر

5. **کتاب‌های معتبر**  
   - *C# 12 and .NET 8 – Modern Cross-Platform Development* (Mark J. Price)  
   - *Effective C#* (Bill Wagner) — نسخه‌های به‌روزشده درباره primary constructors

---

## ✅ جمع‌بندی

Primary Constructor یکی از هوشمندانه‌ترین ویژگی‌های اضافه‌شده به C# در سال‌های اخیر است. این ویژگی به‌ویژه برای توسعه‌دهندگانی که از **الگوهای مدرن** مانند **dependency injection** و **data modeling** استفاده می‌کنند، بسیار کاربردی است.  
با این حال، باید از **محدودیت‌های آن** (مثل عدم امکان داشتن چند constructor) آگاه بود و با هوشمندی استفاده کرد.
.
