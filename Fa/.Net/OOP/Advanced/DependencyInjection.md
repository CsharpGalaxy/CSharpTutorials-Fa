
# 📚 Dependency Injection (DI) در C#  
## (از مقدماتی تا سطح متوسط)

> 📌 **سطح مخاطب**: برنامه‌نویسان مبتدی تا متوسط در سی‌شارپ  
> 📅 **بروزرسانی**: ۶ دسامبر ۲۰۲۵  
> 📂 **بخش از**: مجموعه آموزش اصول طراحی (Design Principles) در OOP

---

## فهرست مطالب

1. [مقدمه‌ای بر Dependency Injection](#مقدمه‌ای-بر-dependency-injection)
2. [چرا به DI نیاز داریم؟](#چرا-به-di-نیاز-داریم)
3. [انواع Dependency Injection](#انواع-dependency-injection)
4. [پیاده‌سازی ساده DI بدون فریم‌ورک](#پیاده‌سازی-ساده-di-بدون-فریم‌ورک)
5. [استفاده از DI در ASP.NET Core](#استفاده-از-di-در-aspnet-core)
6. [مزایا و معایب DI](#مزایا-و-معایب-di)
7. [نکات کلیدی برای استفاده درست از DI](#نکات-کلیدی-برای-استفاده-درست-از-di)
8. [منابع معتبر](#منابع-معتبر)

---

## مقدمه‌ای بر Dependency Injection

**Dependency Injection (DI)** — یا "تزریق وابستگی" — یکی از **الگوهای طراحی (Design Patterns)** و در عین حال یکی از **اصول کلیدی SOLID** (به‌ویژه **اصل وارونگی وابستگی — Dependency Inversion Principle**) است که به ما کمک می‌کند کدهای **انعطاف‌پذیر، قابل تست و قابل نگهداری** بنویسیم.

به زبان ساده:  
> به جای اینکه یک کلاس **خودش** شیء مورد نیازش را ایجاد کند، آن **وابستگی** را از بیرون دریافت می‌کند — یعنی **تزریق** می‌شود.

---

## چرا به DI نیاز داریم؟

بدون DI، کلاس‌ها به‌صورت **مستقیم** به پیاده‌سازی‌های خاصی وابسته می‌شوند. این امر باعث می‌شود:

- کد **قابل تست نباشد** (چون نمی‌توانی Mock کنی)
- تغییر پیاده‌سازی دشوار باشد
- کوپلینگ (Coupling) بین کلاس‌ها **زیاد** شود

### مثال بدون DI (بد):

```csharp
public class EmailService
{
    public void Send(string to, string message)
    {
        Console.WriteLine($"Email sent to {to}: {message}");
    }
}

public class NotificationService
{
    private EmailService _emailService = new EmailService(); // وابستگی سخت‌کد شده!

    public void Notify(string user, string message)
    {
        _emailService.Send(user, message);
    }
}
```

در اینجا، `NotificationService` به‌طور مستقیم به `EmailService` وابسته است. اگر بخواهیم از SMS یا Push Notification استفاده کنیم، باید کد را تغییر دهیم!

---

## انواع Dependency Injection

در C#، سه روش اصلی برای تزریق وابستگی وجود دارد:

| روش | توضیح |
|------|--------|
| **Constructor Injection** | وابستگی از طریق سازنده (Constructor) تزریق می‌شود — **رایج‌ترین و توصیه‌شده‌ترین روش** |
| **Property Injection** | وابستگی از طریق یک Property قابل نوشتن (Setter) |
| **Method Injection** | وابستگی به عنوان پارامتر یک متد خاص ارسال می‌شود (کمتر رایج) |

> ✅ **توصیه**: همیشه از **Constructor Injection** استفاده کنید مگر اینکه دلیل خاصی برای غیر آن داشته باشید.

---

## پیاده‌سازی ساده DI بدون فریم‌ورک

در C# می‌توانیم بدون هیچ فریم‌ورک DI (مثل Unity، Autofac یا حتی built-in container) از DI استفاده کنیم — فقط با **اساس OOP**!

### گام ۱: تعریف رابط (Interface)

```csharp
public interface INotificationService
{
    void Notify(string user, string message);
}
```

### گام ۲: پیاده‌سازی‌های مختلف

```csharp
public class EmailNotificationService : INotificationService
{
    public void Notify(string user, string message)
    {
        Console.WriteLine($"[Email] Sent to {user}: {message}");
    }
}

public class SmsNotificationService : INotificationService
{
    public void Notify(string user, string message)
    {
        Console.WriteLine($"[SMS] Sent to {user}: {message}");
    }
}
```

### گام ۳: کلاس مصرف‌کننده با Constructor Injection

```csharp
public class UserManager
{
    private readonly INotificationService _notificationService;

    public UserManager(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }

    public void RegisterUser(string username)
    {
        // منطق ثبت‌نام...
        _notificationService.Notify(username, "Welcome!");
    }
}
```

### گام ۴: استفاده در برنامه

```csharp
class Program
{
    static void Main()
    {
        // می‌توانیم هر پیاده‌سازی را جایگزین کنیم!
        var notificationService = new SmsNotificationService();
        var userManager = new UserManager(notificationService);
        userManager.RegisterUser("Ali");
    }
}
```

> 🔑 **نکته**: این کد کاملاً **بدون فریم‌ورک DI** است، ولی اصول DI را رعایت کرده است!

---

## استفاده از DI در ASP.NET Core

در **ASP.NET Core**، یک **سیستم DI داخلی (Built-in IoC Container)** وجود دارد که به راحتی قابل استفاده است.

### ۱. ثبت سرویس‌ها در `Program.cs` (یا `Startup.cs` در نسخه‌های قدیمی)

```csharp
var builder = WebApplication.CreateBuilder(args);

// ثبت سرویس به صورت Scoped
builder.Services.AddScoped<INotificationService, EmailNotificationService>();

var app = builder.Build();
// ...
```

### ۲. استفاده در کنترلر یا سرویس

```csharp
public class UserController : ControllerBase
{
    private readonly INotificationService _notificationService;

    public UserController(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }

    [HttpPost("register")]
    public IActionResult Register(string username)
    {
        // منطق ثبت‌نام...
        _notificationService.Notify(username, "Welcome!");
        return Ok();
    }
}
```

### انواع Lifetime در ASP.NET Core DI

| نوع | توضیح |
|------|--------|
| **Transient** | هر بار که درخواست شد، یک نمونه جدید ایجاد می‌شود |
| **Scoped** | یک نمونه در هر درخواست HTTP (در وب) یا scope |
| **Singleton** | یک نمونه در کل برنامه |

```csharp
services.AddTransient<IService, Service>();
services.AddScoped<IService, Service>();
services.AddSingleton<IService, Service>();
```

---

## مزایا و معایب DI

### ✅ مزایا
- کاهش **وابستگی سخت‌کد شده** (Tight Coupling)
- افزایش **قابلیت تست‌پذیری** (Unit Testing با Mock)
- قابلیت **تعویض پیاده‌سازی** بدون تغییر کد
- رعایت **اصل وارونگی وابستگی (DIP)** از SOLID

### ❌ معایب (در صورت سوءاستفاده)
- پیچیدگی اولیه برای مبتدی‌ها
- ممکن است **کد را طولانی‌تر** کند (اگر بیش از حد abstraction شود)
- نیاز به درک صحیح از **Lifetime** سرویس‌ها

> ⚠️ **هشدار**: DI یک ابزار است، نه هدف! همیشه ساده‌ترین راه را انتخاب کن.

---

## نکات کلیدی برای استفاده درست از DI

1. **همیشه از رابط (Interface) استفاده کن** — نه کلاس ملموس.
2. **Constructor Injection را ترجیح بده**.
3. **Lifetime سرویس‌ها را با دقت انتخاب کن** — مخصوصاً از `Singleton` با دقت استفاده کن.
4. **از Service Locator الگو استفاده نکن** — این ضد الگو (Anti-Pattern) است.
5. **در تست‌ها، Mock کن** — با استفاده از Moq یا NSubstitute.

---

## منابع معتبر

1. **Microsoft Learn – Dependency Injection in .NET**  
   👉 [https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)

2. **Martin Fowler – Inversion of Control Containers and the Dependency Injection pattern**  
   👉 [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html)

3. **Microsoft – ASP.NET Core Dependency Injection**  
   👉 [https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)

4. **Clean Code by Robert C. Martin** – Chapter on Dependency Management  
   (ISBN: 978-0132350884)

5. **C# in Depth by Jon Skeet** – Section on DI and Design Principles  
   (ISBN: 978-1617294536)
