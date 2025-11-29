
## 📚 فهرست مطالب

1. [مقدمه](#مقدمه)
2. [اصل اول: Single Responsibility Principle (SRP)](#اصل-اول-single-responsibility-principle-srp)
3. [اصل دوم: Open/Closed Principle (OCP)](#اصل-دوم-openclosed-principle-ocp)
4. [اصل سوم: Liskov Substitution Principle (LSP)](#اصل-سوم-liskov-substitution-principle-lsp)
5. [اصل چهارم: Interface Segregation Principle (ISP)](#اصل-چهارم-interface-segregation-principle-isp)
6. [اصل پنجم: Dependency Inversion Principle (DIP)](#اصل-پنجم-dependency-inversion-principle-dip)
7. [جمع‌بندی](#جمع‌بندی)
8. [منابع معتبر](#منابع-معتبر)

---

## مقدمه

اصول **SOLID** مجموعه‌ای از پنج اصل طراحی (Design Principles) در برنامه‌نویسی شیء‌گرا (OOP) هستند که برای نوشتن کدِ **قابل نگهداری، انعطاف‌پذیر و قابل تست** طراحی شده‌اند. این اصول توسط **Robert C. Martin** (معروف به Uncle Bob) معرفی شده‌اند و در تمام زبان‌های شیء‌گرا — از جمله **C#** — کاربرد گسترده‌ای دارند.

در این راهنما، هر یک از اصول SOLID را به زبان ساده، همراه با مثال‌های عملی در C# و توضیحات مستند شده بررسی می‌کنیم.

---

## اصل اول: Single Responsibility Principle (SRP)

> **هر کلاس باید فقط یک دلیل برای تغییر داشته باشد.**

### توضیح:
کلاس‌ها نباید بیش از یک مسئولیت (Responsibility) داشته باشند. مسئولیت به معنای دلیلی است که باعث تغییر در کد شود.

### مثال نادرست (بدون رعایت SRP):

```csharp
public class Invoice
{
    public double CalculateTotal() { /* محاسبه مبلغ */ }
    public void SaveToFile(string filename) { /* ذخیره در فایل */ }
    public void PrintInvoice() { /* چاپ فاکتور */ }
}
```

این کلاس سه مسئولیت دارد:
- محاسبه مبلغ
- ذخیره‌سازی
- چاپ

### راه‌حل صحیح (با رعایت SRP):

```csharp
public class Invoice
{
    public double CalculateTotal() { /* محاسبه مبلغ */ }
}

public class InvoicePrinter
{
    public void Print(Invoice invoice) { /* چاپ */ }
}

public class InvoiceSaver
{
    public void SaveToFile(Invoice invoice, string filename) { /* ذخیره */ }
}
```

✅ **مزیت**: تغییر در یکی از بخش‌ها، روی بقیه تأثیر نمی‌گذارد.

---

## اصل دوم: Open/Closed Principle (OCP)

> **نرم‌افزار باید باز برای گسترش و بسته برای تغییر باشد.**

### توضیح:
شما باید بتوانید رفتار سیستم را **بدون تغییر در کد موجود** گسترش دهید (معمولاً با استفاده از ارث‌بری یا رابط‌ها).

### مثال نادرست:

```csharp
public class AreaCalculator
{
    public double CalculateArea(object shape)
    {
        if (shape is Rectangle r)
            return r.Width * r.Height;
        else if (shape is Circle c)
            return c.Radius * c.Radius * Math.PI;
        // اگر شکل جدیدی اضافه شود، باید کد را تغییر دهیم!
    }
}
```

### راه‌حل صحیح:

```csharp
public abstract class Shape
{
    public abstract double CalculateArea();
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    public override double CalculateArea() => Width * Height;
}

public class Circle : Shape
{
    public double Radius { get; set; }
    public override double CalculateArea() => Radius * Radius * Math.PI;
}

public class AreaCalculator
{
    public double CalculateArea(Shape shape) => shape.CalculateArea();
}
```

✅ **مزیت**: افزودن شکل جدید بدون تغییر در کلاس `AreaCalculator`.

---

## اصل سوم: Liskov Substitution Principle (LSP)

> **اگر S زیرمجموعه‌ی T باشد، آنگاه شیء‌های T باید با اشیاء S قابل جایگزینی باشند بدون اینکه رفتار برنامه تغییر کند.**

### توضیح:
زیرکلاس‌ها باید **رفتار منطقی** والد خود را حفظ کنند و نباید قراردادهای پایه را نقض کنند.

### مثال نادرست:

```csharp
public class Rectangle
{
    public virtual double Width { get; set; }
    public virtual double Height { get; set; }
    public double Area => Width * Height;
}

public class Square : Rectangle
{
    public override double Width
    {
        set { base.Width = base.Height = value; }
    }

    public override double Height
    {
        set { base.Width = base.Height = value; }
    }
}
```

اگر از `Square` به جای `Rectangle` استفاده کنیم، محاسبه‌ی مساحت به درستی کار نمی‌کند (چون تغییر عرض، ارتفاع را هم تغییر می‌دهد).

### راه‌حل صحیح:
از **ترکیب (Composition)** یا **رابط‌های جداگانه** استفاده کنید، نه ارث‌بری نادرست.

✅ **نکته**: اگر زیرکلاس شما باعث نقض منطق کلاس پایه شود، اصل LSP نقض شده است.

---

## اصل چهارم: Interface Segregation Principle (ISP)

> **مشتریان نباید وابسته به رابط‌هایی باشند که از آن‌ها استفاده نمی‌کنند.**

### توضیح:
به جای یک رابط بزرگ، چندین رابط کوچک و تخصصی تعریف کنید.

### مثال نادرست:

```csharp
public interface IMachine
{
    void Print();
    void Scan();
    void Fax();
}

public class OldPrinter : IMachine
{
    public void Print() { /* OK */ }
    public void Scan() { throw new NotSupportedException(); } // ❌
    public void Fax() { throw new NotSupportedException(); }   // ❌
}
```

### راه‌حل صحیح:

```csharp
public interface IPrinter
{
    void Print();
}

public interface IScanner
{
    void Scan();
}

public interface IFax
{
    void Fax();
}

public class OldPrinter : IPrinter
{
    public void Print() { /* OK */ }
}

public class ModernMachine : IPrinter, IScanner, IFax
{
    public void Print() { }
    public void Scan() { }
    public void Fax() { }
}
```

✅ **مزیت**: هر کلاس فقط رابط‌هایی را پیاده‌سازی می‌کند که نیاز دارد.

---

## اصل پنجم: Dependency Inversion Principle (DIP)

> **۱. ماژول‌های سطح بالا نباید به ماژول‌های سطح پایین وابسته باشند. هر دو باید به انتزاعات (Abstractions) وابسته باشند.**  
> **۲. انتزاعات نباید به جزئیات وابسته باشند. جزئیات باید به انتزاعات وابسته باشند.**

### توضیح:
به جای وابستگی مستقیم به کلاس‌های پیاده‌سازی، به **رابط‌ها یا کلاس‌های چکیده** وابسته شوید.

### مثال نادرست:

```csharp
public class EmailService
{
    public void SendEmail(string to, string message) { }
}

public class Notification
{
    private EmailService _emailService = new EmailService(); // ❌ وابستگی مستقیم

    public void Notify(string user, string message)
    {
        _emailService.SendEmail(user, message);
    }
}
```

### راه‌حل صحیح:

```csharp
public interface INotificationService
{
    void Send(string to, string message);
}

public class EmailService : INotificationService
{
    public void Send(string to, string message) { }
}

public class Notification
{
    private readonly INotificationService _service;

    public Notification(INotificationService service) // Dependency Injection
    {
        _service = service;
    }

    public void Notify(string user, string message)
    {
        _service.Send(user, message);
    }
}
```

✅ **مزیت**: قابلیت تست‌پذیری بالا، قابلیت جایگزینی سرویس (مثلاً با SMS)، و کاهش هم‌بستگی (Coupling).

---

## جمع‌بندی

| اصل | معنی | هدف |
|-----|------|------|
| **SRP** | یک مسئولیت در هر کلاس | کاهش پیچیدگی و افزایش قابلیت نگهداری |
| **OCP** | باز برای گسترش، بسته برای تغییر | انعطاف‌پذیری بدون تغییر کد موجود |
| **LSP** | قابل جایگزینی بودن زیرکلاس‌ها | حفظ رفتار منطقی و قابل پیش‌بینی |
| **ISP** | رابط‌های کوچک و تخصصی | کاهش وابستگی‌های غیرضروری |
| **DIP** | وابستگی به انتزاعات | کاهش هم‌بستگی و افزایش تست‌پذیری |

این اصول، پایه‌های **معماری نرم‌افزار** و **کدنویسی حرفه‌ای** در C# هستند و استفاده از آن‌ها، کدهای شما را برای پروژه‌های واقعی آماده می‌کند.

---

## منابع معتبر

1. **Agile Software Development, Principles, Patterns, and Practices** – Robert C. Martin (Uncle Bob)  
   📘 [https://www.oreilly.com/library/view/agile-software-development/0135974445/](https://www.oreilly.com/library/view/agile-software-development/0135974445/)

2. **The Principles of Object-Oriented JavaScript** – Nicholas C. Zakas (برای درک عمومی اصول)  
   *(با وجود نام، مفاهیم SOLID عمومی هستند)*

3. **Microsoft Learn – C# Programming Guide**  
   🌐 [https://learn.microsoft.com/en-us/dotnet/csharp/](https://learn.microsoft.com/en-us/dotnet/csharp/)

4. **Refactoring Guru – SOLID Principles**  
   🌐 [https://refactoring.guru/solid-principles](https://refactoring.guru/solid-principles)

5. **Clean Code: A Handbook of Agile Software Craftsmanship** – Robert C. Martin  
   📘 [https://www.oreilly.com/library/view/clean-code/9780136083238/](https://www.oreilly.com/library/view/clean-code/9780136083238/)

6. **C# in Depth** – Jon Skeet  
   📘 (برای درک عمیق‌تر از ویژگی‌های C# که پیاده‌سازی SOLID را تسهیل می‌کنند)

