
## 📚 فهرست مطالب

1. [مقدمه: Sealing چیست؟](#مقدمه-sealing-چیست)  
2. [چرا Sealing برای امنیت مهم است؟](#چرا-sealing-برای-امنیت-مهم-است)  
3. [مکانیزم `sealed` در C#](#مکانیزم-sealed-در-c)  
4. [Sealing در کلاس‌ها](#sealing-در-کلاس‌ها)  
5. [Sealing در متدها و Properties](#sealing-در-متدها-و-properties)  
6. [Sealing به عنوان یک Design Principle](#sealing-به-عنوان-یک-design-principle)  
7. [مثال‌های عملی در C#](#مثال‌های-عملی-در-c)  
8. [پیشنهادات بهترین روش‌ها (Best Practices)](#پیشنهادات-بهترین-روش‌ها-best-practices)  
9. [جمع‌بندی](#جمع‌بندی)  
10. [منابع معتبر](#منابع-معتبر)

---

## مقدمه: Sealing چیست؟

در C#، کلمه کلیدی `sealed` به شما این امکان را می‌دهد که از **وراثت دادن** یک کلاس یا **overriding کردن** یک متد جلوگیری کنید. این مفهوم، به عنوان یک ابزار برای کنترل چگونگی توسعه و بازنویسی کدها در زیرکلاس‌ها به کار می‌رود.

اما فراتر از محدودیت فنی، **Sealing** می‌تواند به عنوان یک **اصل طراحی امنیتی** (Security-by-Design) در OOP مورد استفاده قرار گیرد.

---

## چرا Sealing برای امنیت مهم است؟

وقتی یک کلاس یا متد قابل ارث‌بری یا override نباشد:

- **عدم قابلیت پیش‌بینی رفتار**: دیگران نمی‌توانند رفتار داخلی کلاس را تغییر دهند → کاهش حملات از نوع **inheritance-based exploitation**.
- **کاهش سطح حمله (Attack Surface)**: کدهایی که قابل override نیستند، خطر کمتری برای تزریق منطق مخرب دارند.
- **پیاده‌سازی قابل اعتماد**: امنیت کتابخانه‌های عمومی (public APIs) با sealing تقویت می‌شود.

> ⚠️ نکته: Sealing به‌خودی‌خود یک مکانیسم "امنیت مطلق" نیست، اما یکی از اصول **Secure by Default** در طراحی نرم‌افزار است.

---

## مکانیزم `sealed` در C#

کلمه کلیدی `sealed` در C# دو کاربرد اصلی دارد:

1. **در کلاس‌ها**: جلوگیری از ارث‌بری.
2. **در متدها/properties**: جلوگیری از override شدن در زیرکلاس‌ها (حتی اگر متد `virtual` یا `override` باشد).

### ساختار عمومی:

```csharp
sealed class MyClass { } // ارث‌بری ممنوع

class Base
{
    public virtual void DoWork() { }
}

class Derived : Base
{
    public sealed override void DoWork() { } // override شد ولی دیگر قابل override نیست
}
```

---

## Sealing در کلاس‌ها

```csharp
sealed class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount) => Balance += amount;
    public void Withdraw(decimal amount) => Balance -= amount;
}
```

> در اینجا، هیچ کلاس دیگری نمی‌تواند از `BankAccount` ارث‌بری کند — مثلاً نمی‌توان یک `EvilBankAccount` ساخت که منطق withdraw را دستکاری کند.

---

## Sealing در متدها و Properties

```csharp
class SecureService
{
    public virtual void ProcessData() 
    {
        // logic
    }
}

class FinalService : SecureService
{
    public sealed override void ProcessData()
    {
        // final implementation
    }
}
```

در این مثال، `ProcessData` دیگر در هیچ کلاسی که از `FinalService` ارث‌بری می‌کند قابل override نیست.

---

## Sealing به عنوان یک Design Principle

در اصول طراحی مدرن نرم‌افزار (مانند SOLID)، sealing مستقیماً ذکر نمی‌شود، اما با اصول زیر همسو است:

- **Principle of Least Privilege (POLP)**: فقط حداقل دسترسی لازم را فراهم کن.
- **Fail Securely**: اگر یک کلاس برای تغییر رفتار طراحی نشده، نباید اجازهٔ تغییر داده شود.
- **Defensive Design**: انتظار نداشته باشید که تمام توسعه‌دهندگان از قوانین شما پیروی کنند — محدودیت‌ها را در کد اعمال کنید.

> 📌 Microsoft در [راهنمای امنیتی .NET](https://learn.microsoft.com/en-us/dotnet/standard/security/secure-coding) توصیه می‌کند که کلاس‌هایی که برای ارث‌بری طراحی نشده‌اند، **باید sealed باشند**.

---

## مثال‌های عملی در C#

### 🔐 مثال ۱: محافظت از اعتبارسنجی

```csharp
public sealed class AuthToken
{
    private readonly string _token;

    public AuthToken(string token)
    {
        if (string.IsNullOrEmpty(token)) throw new ArgumentException("Invalid token");
        _token = token;
    }

    public string Value => _token;
}
```

- بدون sealing، یک کلاس مثل `FakeAuthToken : AuthToken` می‌تواند منطق سازنده را دور بزند.
- با sealing، این خطر رفع می‌شود.

### 🔐 مثال ۲: جلوگیری از تغییر رفتار امنیتی

```csharp
public class SecureLogger
{
    public virtual void Log(string message)
    {
        // Write to secure log file
        File.AppendAllText("secure.log", $"[SECURE] {message}{Environment.NewLine}");
    }
}

public sealed class ProductionLogger : SecureLogger
{
    public sealed override void Log(string message)
    {
        if (string.IsNullOrWhiteSpace(message))
            throw new ArgumentException("Log message cannot be empty");
        base.Log(message);
    }
}
```

- هیچ کلاسی نمی‌تواند رفتار `Log` را در `ProductionLogger` تغییر دهد.

---

## پیشنهادات بهترین روش‌ها (Best Practices)

✅ **هر کلاسی که برای ارث‌بری طراحی نشده، sealed باشد.**  
✅ از sealing برای محافظت از منطق حساس (مانند اعتبارسنجی، لاگینگ، رمزنگاری) استفاده کنید.  
✅ در کتابخانه‌های عمومی (NuGet packages)، sealed کردن کلاس‌ها امنیت و پایداری را افزایش می‌دهد.  
❌ از sealed کردن بی‌رویه در کلاس‌های آزمایشی یا داخلی (internal) خودداری کنید — ممکن است تست‌پذیری را کاهش دهد.

---

## جمع‌بندی

- `sealed` یک مکانیسم ساده اما قدرتمند برای کنترل وراثت در C# است.
- از دید امنیتی، sealed کردن کلاس‌ها و متدها می‌تواند از سوءاستفاده و تغییرات ناخواسته جلوگیری کند.
- استفاده از sealing به عنوان یک **اصل طراحی دفاعی** (Defensive Design Principle) در توسعه نرم‌افزارهای امن توصیه می‌شود.

---

## منابع معتبر

1. **Microsoft Learn – Sealed Classes and Methods**  
   🔗 [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed)

2. **Microsoft – Secure Coding Guidelines for .NET**  
   🔗 [https://learn.microsoft.com/en-us/dotnet/standard/security/secure-coding](https://learn.microsoft.com/en-us/dotnet/standard/security/secure-coding)

3. **C# in Depth – Jon Skeet (4th Edition), Chapter on Inheritance**  
   📘 ISBN: 978-1617294531

4. **OWASP – Secure Coding Practices**  
   🔗 [https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)

5. **Framework Design Guidelines – Microsoft (2nd Edition), Section on Sealing**  
   📘 ISBN: 978-0321545619  
   (فصل 5: "Member Design Guidelines" – بخش sealed members)

اگر نیاز به نسخهٔ PDF یا قالب‌بندی برای وب دارید، خوشحال می‌شوم کمک کنم.
