# مقاله جامع: کاربردهای Anonymous Type در C# (از مقدماتی تا پیشرفته)

این مقاله به‌عنوان یک مرجع آموزشی کامل برای مخزن (Repository) آموزشی شما طراحی شده است. با رعایت اصول آموزش شیءگرایی (OOP) و استانداردهای مدرن C#، تمامی جنبه‌های **Anonymous Type** (نوع ناشناس) از مفاهیم پایه تا مقایسه‌های پیشرفته و مثال‌های واقعی پوشش داده شده است.

---

## 📑 فهرست مطالب
1. [مقدمه: Anonymous Type چیست؟](#مقدمه)
2. [چرا از Anonymous Type استفاده می‌شود؟](#چرا-استفاده-می‌شود)
3. [مهم‌ترین کاربردها با مثال عملی](#کاربردها)
   - [۳.۱. داده‌های موقت و ساخت Object موقت](#داده‌های-موقت)
   - [۳.۲. LINQ Projection و Query Results](#linq-projection)
   - [۳.۳. انتخاب چند Property و ترکیب مقادیر مرتبط](#انتخاب-چند-پراپرتی)
   - [۳.۴. داده‌های تودرتو (Nested Data)](#داده‌های-تودرتو)
   - [۳.۵. استفاده در پردازش‌های داخلی](#پردازش‌های-داخلی)
4. [مثال‌های واقعی در پروژه‌ها](#مثال‌های-واقعی)
   - [۴.۱. پروژه عمومی C#](#پروژه-عمومی-c)
   - [۴.۲. پروژه ASP.NET Core](#پروژه-aspnet-core)
   - [۴.۳. LINQ و Entity Framework Core](#linq-و-entity-framework)
5. [مقایسه گزینه‌ها: چه زمانی کدام را انتخاب کنیم؟](#مقایسه-گزینه‌ها)
   - [چه زمانی Anonymous Type انتخاب مناسبی است؟](#چه-زمانی-anonymous-type)
   - [چه زمانی Class بهتر است؟](#چه-زمانی-class)
   - [چه زمانی Record بهتر است؟](#چه-زمانی-record)
   - [چه زمانی Tuple بهتر است؟](#چه-زمانی-tuple)
6. [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
7. [جمع‌بندی](#جمع‌بندی)
8. [منابع معتبر](#منابع-معتبر)

---

## <a id="مقدمه"></a>۱. مقدمه: Anonymous Type چیست؟
نوع ناشناس (Anonymous Type) در C# قابلیتی است که به شما اجازه می‌دهد بدون تعریف صریح یک کلاس (Class)، یک شیء با پراپرتی‌های خواندنی (Read-Only) ایجاد کنید. این قابلیت با استفاده از کلیدواژه `var` و سینتکس `new { ... }` پیاده‌سازی می‌شود و کامپایلر در زمان ساخت (Compile-time) یک نام منحصر‌به‌فرد و مخفی برای آن تولید می‌کند.

```csharp
var person = new { Name = "علی", Age = ۳۰ };
Console.WriteLine(person.Name); // خروجی: علی
```

---

## <a id="چرا-استفاده-می‌شود"></a>۲. چرا از Anonymous Type استفاده می‌شود؟
- **کاهش کدهای تکراری (Boilerplate)**: نیاز به تعریف کلاس‌های کوچک و یک‌بارمصرف (DTOهای موقت) را از بین می‌برد.
- **بهبود خوانایی**: در کوئری‌های LINQ، تمرکز را بر روی "داده‌های مورد نیاز" نگه می‌دارد، نه ساختار کلاس.
- **بهینه‌سازی حافظه و پرفورمنس**: فقط فیلدهای لازم را از پایگاه داده یا منابع دیگر واکشی می‌کند (به‌ویژه در EF Core).
- **ایمنی نوع (Type Safety)**: برخلاف `dynamic` یا `object`، کامپایلر نوع داده‌ها را بررسی می‌کند و از خطاهای زمان اجرا جلوگیری می‌شود.

---

## <a id="کاربردها"></a>۳. مهم‌ترین کاربردها با مثال عملی

### <a id="داده‌های-موقت"></a>۳.۱. داده‌های موقت و ساخت Object موقت
زمانی که نیاز به گروه‌بندی چند مقدار مرتبط در یک محدوده محلی (Local Scope) دارید و نمی‌خواهید یک کلاس جداگانه تعریف کنید.

```csharp
var tempConfig = new { Timeout = 5000, Retries = 3, UseCache = true };
// استفاده موقت در همان متد
if (tempConfig.UseCache) { /* ... */ }
```

### <a id="linq-projection"></a>۳.۲. LINQ Projection و Query Results
رایج‌ترین کاربرد Anonymous Type، شکل‌دهی (Shaping) نتایج کوئری‌هاست تا فقط داده‌های مورد نیاز بازگردانده شوند.

```csharp
var products = new List<Product>
{
    new Product { Id = 1, Name = "لپ‌تاپ", Price = 50000000, Category = "الکترونیک" }
};

var queryResult = products
    .Where(p => p.Price > 10000000)
    .Select(p => new { p.Name, p.Price }); // فقط نام و قیمت انتخاب می‌شود
```

### <a id="انتخاب-چند-پراپرتی"></a>۳.۳. انتخاب چند Property و ترکیب مقادیر مرتبط
می‌توانید نام پراپرتی‌ها را در نوع ناشناس تغییر دهید یا مقادیر محاسبه‌شده را ترکیب کنید.

```csharp
var userSummary = users.Select(u => new 
{
    FullName = $"{u.FirstName} {u.LastName}", // ترکیب مقادیر
    u.Email,
    IsPremium = u.SubscriptionLevel == "Premium" // مقدار محاسبه‌شده
});
```

### <a id="داده‌های-تودرتو"></a>۳.۴. داده‌های تودرتو (Nested Data)
انواع ناشناس می‌توانند درون یکدیگر قرار گیرند تا ساختارهای داده‌ای سلسله‌مراتبی موقت ایجاد کنند.

```csharp
var orderReport = orders.Select(o => new
{
    o.OrderId,
    Customer = new { o.Customer.Name, o.Customer.City }, // Nested Anonymous Type
    TotalItems = o.Items.Count
});
```

### <a id="پردازش‌های-داخلی"></a>۳.۵. استفاده در پردازش‌های داخلی
برای گروه‌بندی داده‌ها (`GroupBy`) یا مرتب‌سازی‌های پیچیده در داخل یک متد، بدون نشت این ساختار به بیرون از متد.

```csharp
var groupedData = transactions.GroupBy(t => t.Date.Date)
    .Select(g => new 
    {
        Date = g.Key,
        TotalAmount = g.Sum(t => t.Amount),
        Count = g.Count()
    });
```

---

## <a id="مثال‌های-واقعی"></a>۴. مثال‌های واقعی در پروژه‌ها

### <a id="پروژه-عمومی-c"></a>۴.۱. پروژه عمومی C# (گزارش‌گیری داخلی)
فرض کنید می‌خواهید یک گزارش موقت از وضعیت سیستم بسازید:

```csharp
public void GenerateSystemReport()
{
    var systemMetrics = new 
    {
        CpuUsage = GetCpuUsage(),
        MemoryAvailableMB = GetFreeMemory(),
        Uptime = DateTime.Now - Process.GetCurrentProcess().StartTime
    };

    Console.WriteLine($"CPU: {systemMetrics.CpuUsage}%, RAM: {systemMetrics.MemoryAvailableMB}MB");
}
```

### <a id="پروژه-aspnet-core"></a>۴.۲. پروژه ASP.NET Core (پاسخ API)
در کنترلرهای ASP.NET Core، بازگرداندن Anonymous Type به‌عنوان `IActionResult` بسیار رایج است. فریم‌ورک به‌طور خودکار آن را به JSON سریالایز می‌کند.

```csharp
[HttpGet("api/user/{id}")]
public IActionResult GetUserSummary(int id)
{
    var user = _dbContext.Users.Find(id);
    if (user is null) return NotFound();

    // بازگرداندن ساختار موقت بدون نیاز به ساخت UserSummaryDto
    return Ok(new 
    {
        user.Id,
        user.UserName,
        Roles = user.Roles.Select(r => r.Name),
        Message = "اطلاعات با موفقیت دریافت شد."
    });
}
```
*نکته:* اگر از Swagger/OpenAPI استفاده می‌کنید، برای مستندسازی دقیق‌تر، استفاده از `record` یا `class` ترجیح داده می‌شود، اما برای پاسخ‌های سریع و داخلی، Anonymous Type کاملاً معتبر است.

### <a id="linq-و-entity-framework"></a>۴.۳. LINQ و Entity Framework Core
استفاده از Anonymous Type در EF Core باعث می‌شود کوئری SQL فقط ستون‌های انتخاب‌شده را واکشی کند (بهینه‌سازی شدید پرفورمنس).

```csharp
var blogSummaries = await _context.Blogs
    .Where(b => b.IsActive)
    .Select(b => new 
    {
        b.BlogId,
        b.Title,
        PostCount = b.Posts.Count()
    })
    .ToListAsync();

// SQL تولیدشده فقط شامل ستون‌های BlogId, Title و یک COUNT برای Posts خواهد بود.
```

---

## <a id="مقایسه-گزینه‌ها"></a>۵. مقایسه گزینه‌ها: چه زمانی کدام را انتخاب کنیم؟

### <a id="چه-زمانی-anonymous-type"></a>✅ چه زمانی Anonymous Type انتخاب مناسبی است؟
- زمانی که داده فقط در **محدوده یک متد** (Local Scope) استفاده می‌شود.
- در کوئری‌های LINQ برای `Select` یا `GroupBy`.
- زمانی که نمی‌خواهید فایل/کلاس جدیدی به پروژه اضافه کنید.
- برای بازگرداندن پاسخ‌های سریع JSON در APIها (بدون نیاز به مستندسازی دقیق Swagger).

### <a id="چه-زمانی-class"></a>❌ چه زمانی Class بهتر است؟
- وقتی داده باید بین متدها، کلاس‌ها یا لایه‌های مختلف برنامه منتقل شود.
- وقتی نیاز به تغییر مقادیر (Mutability) دارید (پراپرتی‌های Anonymous Type فقط خواندنی هستند).
- وقتی نیاز به متدها، رویدادها یا ارث‌بری دارید.
- وقتی نیاز به مستندسازی دقیق API (مثل Swagger) دارید.

### <a id="چه-زمانی-record"></a>🛡️ چه زمانی Record بهتر است؟ (C# 9+)
- وقتی یک شیء **ایمن از نظر رشته‌ای (Immutable)** می‌خواهید که به‌عنوان DTO بین لایه‌ها منتقل شود.
- وقتی به **مقایسه مبتنی بر مقدار (Value-based Equality)** نیاز دارید (`recordA == recordB`).
- جایگزین مدرن و بهتر برای Anonymous Type در زمانی که داده از مرز متد خارج می‌شود.
```csharp
public record UserSummaryDto(int Id, string UserName, int PostCount);
```

### <a id="چه-زمانی-tuple"></a>🔄 چه زمانی Tuple بهتر است؟ (C# 7+)
- وقتی می‌خواهید یک متد **چندین مقدار** را بازگرداند (به‌عنوان جایگزین `out` parameters).
- وقتی نام‌گذاری پراپرتی‌ها اولویت دوم است و سرعت تایپ کد مهم‌تر است.
- Tupleها سربای کمتری دارند، اما در LINQ Projectionها، Anonymous Type خوانایی بهتری دارد.
```csharp
public (int Total, double Average) GetStats(List<int> numbers)
{
    return (numbers.Sum(), numbers.Average());
}
```

---

## <a id="نکات-مهم-و-اشتباهات-رایج"></a>۶. نکات مهم و اشتباهات رایج

| عنوان | توضیح |
| :--- | :--- |
| **فقط خواندنی (Read-Only)** | پراپرتی‌های Anonymous Type را نمی‌توان پس از مقداردهی اولیه تغییر داد. تلاش برای تغییر آن‌ها خطای کامپایل می‌دهد. |
| **محدوده دسترسی** | نوع ناشناس را نمی‌توان به‌عنوان نوع بازگشتی یک متد (به جز `object` یا `dynamic` که توصیه نمی‌شود) یا پارامتر متد استفاده کرد. |
| **اشتباه رایج ۱** | بازگرداندن Anonymous Type از یک متد عمومی با نوع `dynamic`. این کار بررسی نوع در زمان کامپایل را از بین می‌برد و خطر خطای زمان اجرا را افزایش می‌دهد. |
| **اشتباه رایج ۲** | استفاده از Anonymous Type برای داده‌هایی که باید سریالایز/دی‌سریالایز شوند و نیاز به نگاشت دقیق دارند (در این حالت `record` یا `class` بهتر است). |
| **برابری (Equality)** | دو نمونه Anonymous Type با پراپرتی‌های یکسان (همان نام، نوع و ترتیب)، توسط کامپایلر به‌عنوان یک نوع واحد در نظر گرفته می‌شوند و متد `Equals` مقدار `true` برمی‌گرداند. |

---

## <a id="جمع‌بندی"></a>۷. جمع‌بندی
انواع ناشناس (Anonymous Types) ابزاری قدرتمند در جعبه‌ابزار C# هستند که برای **شکل‌دهی موقت داده‌ها**، **بهینه‌سازی کوئری‌های LINQ** و **کاهش کدهای تکراری** طراحی شده‌اند. با این حال، با معرفی `record` در C# 9 و بهبود `Tuple`ها، مرزهای استفاده از آن‌ها مشخص‌تر شده است:
- برای داده‌های **محلی و کوئری‌ها**: Anonymous Type.
- برای داده‌های **قابل انتقال و ایمن (Immutable)**: `record`.
- برای **بازگرداندن چند مقدار ساده** از متد: `Tuple`.
- برای **مدل‌های پیچیده با رفتار**: `class`.

رعایت این اصول به شما کمک می‌کند تا کدی تمیز، بهینه و قابل نگهداری بنویسید که با استانداردهای مدرن توسعه نرم‌افزار همخوانی دارد.

---

## <a id="منابع-معتبر"></a>۸. منابع معتبر
برای مطالعه عمیق‌تر و ارجاع در مخزن آموزشی خود، از منابع رسمی زیر استفاده کنید:

1. **Microsoft Learn - Anonymous Types**:  
   [https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types)
2. **Microsoft Learn - LINQ Projection**:  
   [https://learn.microsoft.com/en-us/dotnet/csharp/linq/project-data](https://learn.microsoft.com/en-us/dotnet/csharp/linq/project-data)
3. **Microsoft Learn - Record Types (C# 9+)**:  
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
4. **Microsoft Learn - Value Tuples**:  
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
5. **Entity Framework Core - Querying Data (Projection)**:  
   [https://learn.microsoft.com/en-us/ef/core/querying/](https://learn.microsoft.com/en-us/ef/core/querying/)

---

💡 **پیشنهاد برای مخزن آموزشی شما**:  
با توجه به علاقه شما به ارائه مفاهیم به‌صورت شفاف و ساختاریافته (و سابقه شما در آموزش OOP)، پیشنهاد می‌شود این مقاله را در مخزن خود همراه با یک **پروژه نمونه کوچک (Sample Project)** شامل یک Console App و یک Minimal API در ASP.NET Core منتشر کنید تا خوانندگان بتوانند کدها را به‌صورت زنده اجرا و بررسی کنند. اگر نیاز به تولید کد آن پروژه نمونه داشتید، خوشحال می‌شوم کمک کنم!