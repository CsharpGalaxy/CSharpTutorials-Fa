# آموزش کامل رفتار ToString در Recordهای سی‌شارپ

رکوردها (Records) که در سی‌شارپ ۹ معرفی شدند، یکی از بهترین ویژگی‌ها برای کار با داده‌های غیرقابل تغییر (Immutable) هستند. یکی از جذاب‌ترین و کاربردی‌ترین ویژگی‌های رکوردها، نحوه پیاده‌سازی خودکار متد `ToString` توسط کامپایلر است. 

در این آموزش، به بررسی عمیق رفتار `ToString` در رکوردها، تفاوت آن با کلاس‌ها، و مکانیزم داخلی آن از جمله متد `PrintMembers` می‌پردازیم.

---

## ۱. تفاوت ToString در Class و Record

### رفتار پیش‌فرض در Class
در کلاس‌های معمولی، اگر متد `ToString` را اورراید (Override) نکنید، خروجی آن صرفاً **نام کامل تایپ (Fully Qualified Name)** است:
```csharp
public class PersonClass {
    public string Name { get; set; }
    public int Age { get; set; }
}
// خروجی: MyNamespace.PersonClass
```

### رفتار پیش‌فرض در Record
اما در رکوردها، کامپایلر به صورت خودکار متد `ToString` را تولید می‌کند. خروجی شامل **نام رکورد و تمام پراپرتی‌های عمومی (Public) به همراه مقدار آن‌ها** است:
```csharp
public record PersonRecord(string Name, int Age);
// خروجی: PersonRecord { Name = Ali, Age = 30 }
```

---

## ۲. یک مثال عملی و خروجی ToString

بیایید یک رکورد تعریف کنیم و خروجی آن را ببینیم:

```csharp
public record User(string Username, string Email, bool IsActive);

public class Program
{
    public static void Main()
    {
        var user = new User("admin", "admin@example.com", true);
        Console.WriteLine(user.ToString());
    }
}
```

**خروجی در کنسول:**
```text
User { Username = admin, Email = admin@example.com, IsActive = True }
```
همانطور که می‌بینید، نیازی به نوشتن هیچ کدی برای فرمت‌دهی خروجی نیست.

---

## ۳. کامپایلر چگونه اطلاعات Record را برای نمایش تولید می‌کند؟

یک تصور اشتباه رایج این است که کامپایلر برای تولید `ToString` از **Reflection** استفاده می‌کند. اینطور نیست! استفاده از Reflection بسیار کند است و در زمان اجرا (Runtime) اتفاق می‌افتد.

در واقع، کامپایلر سی‌شارپ در زمان **کامپایل (Compile-time)**، کد معادل `ToString` را برای شما می‌نویسد. اگر کد بالا را دیکامپایل (Decompile) کنید، کدی شبیه به این می‌بینید:

```csharp
public override string ToString()
{
    StringBuilder sb = new StringBuilder();
    sb.Append("User");
    sb.Append(" { ");
    if (PrintMembers(sb))
    {
        sb.Append(" }");
    }
    else
    {
        sb.Append("}");
    }
    return sb.ToString();
}
```
کامپایلر دقیقاً می‌داند چه پراپرتی‌هایی وجود دارند و مستقیماً آن‌ها را به `StringBuilder` اضافه می‌کند. این کار باعث می‌شود سرعت اجرای آن دقیقاً هم‌اندازه با نوشتن دستی `ToString` باشد.

---

## ۴. بررسی عمیق PrintMembers (از ساده تا پیشرفته)

متد `PrintMembers` قلب تپنده `ToString` در رکوردهاست. این یک متد `protected virtual` است که وظیفه دارد پراپرتی‌ها را به `StringBuilder` اضافه کند.

### سطح ساده: چرا اصلاً PrintMembers وجود دارد؟
دلیل اصلی وجود `PrintMembers` **پشتیبانی از ارث‌بری (Inheritance)** است. اگر شما یک رکورد را از رکورد دیگری ارث‌بری کنید، `ToString` رکورد فرزند باید بتواند پراپرتی‌های رکورد والد را هم چاپ کند. `PrintMembers` این امکان را فراهم می‌کند.

### سطح پیشرفته: سفارشی‌سازی با PrintMembers
اگر بخواهید نحوه چاپ پراپرتی‌ها را تغییر دهید اما همچنان از مزایای ارث‌بری بهره ببرید، باید `PrintMembers` را اورراید کنید (نه `ToString` را).

**مثال پیشرفته:** فرض کنید می‌خواهید مقدار `Email` را در لاگ‌ها مخفی کنید (Masking):

```csharp
public record SecureUser(string Username, string Email, string Password)
{
    // اورراید کردن PrintMembers
    protected virtual bool PrintMembers(StringBuilder builder)
    {
        // فراخوانی متد والد (اگر رکورد ما خودش ارث‌بری داشت)
        if (base.PrintMembers(builder)) 
        {
            builder.Append(", ");
        }

        // اضافه کردن پراپرتی‌های دلخواه با فرمت دلخواه
        builder.Append($"Username = {Username}");
        builder.Append($", Email = {MaskEmail(Email)}");
        builder.Append(", Password = [PROTECTED]"); // رمز عبور چاپ نمی‌شود
        
        return true;
    }

    private string MaskEmail(string email) => "***@" + email.Split('@')[1];
}
```
**نکته مهم:** اگر `PrintMembers` را اورراید کنید، `ToString` تولید شده توسط کامپایلر به طور خودکار از `PrintMembers` جدید شما استفاده می‌کند.

---

## ۵. سفارشی‌سازی کامل ToString

گاهی اوقات می‌خواهید فرمت خروجی را کاملاً تغییر دهید و نیازی به ارث‌بری پراپرتی‌ها ندارید. در این حالت می‌توانید مستقیماً `ToString` را اورراید کنید:

```csharp
public record Point(int X, int Y)
{
    public override string ToString()
    {
        return $"[{X}, {Y}]";
    }
}
// خروجی: [10, 20]
```
**هشدار:** اگر `ToString` را مستقیماً اورراید کنید، متد `PrintMembers` نادیده گرفته می‌شود و اگر رکورد دیگری از این رکورد ارث‌بری کند، پراپرتی‌های آن به صورت خودکار چاپ نخواهند شد.

---

## ۶. کاربرد ToString در Debugging و Logging

تولید خودکار `ToString` در رکوردها دو کاربرد حیاتی دارد:

1. **دیباگر ویژوال استودیو:** وقتی در حال دیباگ هستید و موس را روی یک متغیر از نوع Record می‌برید، یا آن را در پنجره‌های *Locals* یا *Watch* می‌بینید، ویژوال استودیو به صورت خودکار `ToString` را فراخوانی کرده و تمام مقادیر را به شما نشان می‌دهد.
2. **فریم‌ورک‌های لاگ (Serilog, NLog):** وقتی یک رکورد را لاگ می‌کنید، فریم‌ورک لاگ نیازی به تنظیمات اضافی ندارد و تمام پراپرتی‌ها را به صورت ساختاریافته در خروجی ثبت می‌کند.
   ```csharp
   logger.LogInformation("User logged in: {User}", user);
   ```

---

## ۷. تفاوت‌های کلیدی ToString در Record و Class

| ویژگی | Class (پیش‌فرض) | Record (تولید شده توسط کامپایلر) |
| :--- | :--- | :--- |
| **خروجی پیش‌فرض** | نام کامل Namespace و Class | نام Record + لیست تمام پراپرتی‌ها |
| **نیاز به کدنویسی** | نیاز به اورراید دستی دارد | بدون نیاز به کدنویسی |
| **عملکرد (Performance)** | سریع (چون فقط نام را برمی‌گرداند) | بسیار سریع (بدون Reflection) |
| **پشتیبانی از ارث‌بری** | دستی باید مدیریت شود | خودکار از طریق `PrintMembers` |

---

## ۸. نکات مهم و اشتباهات رایج

### ❌ اشتباه رایج ۱: نشت اطلاعات حساس (Data Leakage)
چون `ToString` تمام پراپرتی‌های `public` را چاپ می‌کند، اگر پراپرتی‌هایی مثل `Password`، `CreditCard` یا `SSN` در رکورد داشته باشید، به صورت خودکار در لاگ‌ها و دیباگر چاپ می‌شوند.
**راه‌حل:** پراپرتی‌های حساس را `private` یا `init`-only نگه دارید، یا `PrintMembers` را اورراید کنید تا آن‌ها را چاپ نکنید.

### ❌ اشتباه رایج ۲: چاپ کالکشن‌های بزرگ
اگر رکورد شما شامل یک `List` با هزاران آیتم باشد، `ToString` تمام آن‌ها را چاپ می‌کند که باعث پر شدن حافظه (OOM) و کند شدن شدید برنامه در زمان لاگ‌گیری می‌شود.
**راه‌حل:** برای کالکشن‌های بزرگ، `ToString` یا `PrintMembers` را اورراید کنید و فقط `Count` را چاپ کنید.

### ❌ اشتباه رایج ۳: فراموش کردن `base.PrintMembers`
اگر رکورد شما از یک رکورد دیگر ارث‌بری کرده است و `PrintMembers` را اورراید می‌کنید، حتماً باید `base.PrintMembers(builder)` را فراخوانی کنید، در غیر این صورت پراپرتی‌های رکورد والد چاپ نخواهند شد.

### 💡 نکته طلایی: رکوردهای `struct`
در سی‌شارپ ۱۰، `record struct` معرفی شد. رفتار `ToString` در آن‌ها دقیقاً مشابه `record`های کلاسی است، با این تفاوت که از Boxing جلوگیری می‌کنند و برای سناریوهای با کارایی بالا (High-performance) مناسب‌ترند.

---

## ۹. منابع رسمی

برای مطالعه بیشتر و بررسی دقیق مشخصات زبان، می‌توانید به منابع رسمی مایکروسافت مراجعه کنید:

1. **مستندات رسمی Record Types در Microsoft Learn:**
   [Records - C# reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
   *(بخش `String representation` را مطالعه کنید)*

2. **مشخصات زبان سی‌شارپ (C# Language Specification) - بخش Records:**
   [Records - C# Language Specification | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1811-records)
   *(برای درک عمیق از نحوه تولید `PrintMembers` و `ToString` توسط کامپایلر در سطح Spec)*