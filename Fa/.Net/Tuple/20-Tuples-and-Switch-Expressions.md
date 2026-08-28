# مقایسه جامع Tuple و DTO در #C
*(مقاله‌ای ساختاریافته برای مخزن آموزشی)*

---

## ۱. DTO چیست؟ (توضیحی برای مبتدیان)
**DTO** مخفف **Data Transfer Object** (شیء انتقال داده) است. به زبان ساده، DTO یک کلاس (یا `record`) ساده است که **فقط و فقط** برای حمل داده بین لایه‌های مختلف نرم‌افزار (مثلاً از سرور به کلاینت، یا از لایه Domain به لایه Presentation) استفاده می‌شود. 

یک DTO نباید هیچ‌گونه منطق تجاری (Business Logic)، متد پیچیده یا رفتار خاصی داشته باشد؛ بلکه صرفاً شامل Propertyهایی است که داده‌ها را در خود نگه می‌دارند.
> **تشبیه ساده:** فرض کنید می‌خواهید یک میز را اسباب‌کشی کنید. شما کل خانه (Domain Model) را منتقل نمی‌کنید، بلکه فقط قطعات لازم میز را در یک جعبه‌ی مشخص و برچسب‌خورده (DTO) بسته‌بندی کرده و ارسال می‌کنید. این کار هم امنیت را بالا می‌برد و هم حجم داده‌های ارسالی را بهینه می‌کند.

---

## ۲. Tuple چیست؟
در #C، یک Tuple (به‌ویژه `ValueTuple` که از #C 7.0 به‌طور بومی پشتیبانی می‌شود) یک ساختار سبک‌وزن (Value Type) است که اجازه می‌دهد چندین مقدار را بدون نیاز به تعریف یک کلاس یا `struct` جداگانه، گروه‌بندی و بازگردانی کنید .

---

## ۳. مقایسه جامع Tuple و DTO

| معیار مقایسه | Tuple (مقدار چندگانه) | DTO (کلاس / Record) |
| :--- | :--- | :--- |
| **Named Type** | خیر. یک نوع عمومی (`ValueTuple<T1, T2>`) است و نام معنایی واقعی در سطح سیستم ندارد. | بله. یک نوع با نام مشخص و معنادار (مثل `UserDto`) است که هدف آن کاملاً واضح است. |
| **Maintainability** | ضعیف. در پروژه‌های بزرگ، ردیابی `Item1` یا حتی نام‌های سفارشی Tuple در متدهای مختلف دشوار و خطاپذیر است. | عالی. به دلیل نام‌گذاری واضح و جداسازی مسئولیت‌ها، نگهداری و بازخوانی کد آسان است. |
| **Serialization** | ضعیف. در JSON به‌صورت پیش‌فرض با نام‌های `item1`, `item2` سریالایز می‌شوند و نام‌های معنایی Named Tuple در زمان اجرا (Runtime) از دست می‌روند . | عالی. به‌طور کامل و با نام Propertyهای صحیح توسط کتابخانه‌هایی مثل `System.Text.Json` سریالایز می‌شود. |
| **API Contract** | نامناسب. ابزارهایی مثل Swagger/OpenAPI نمی‌توانند ساختار Tuple را به‌درستی مستند و نمایش دهند. | عالی. پایه‌ی اصلی تعریف قراردادهای API و تولید خودکار مستندات Swagger است. |
| **Versioning** | بسیار دشوار. تغییر تعداد یا نوع پارامترها باعث شکستن قرارداد (Breaking Change) در کل فراخوانی‌ها می‌شود. | آسان. می‌توان نسخه‌های جدید (مثل `ProductDtoV2`) ساخت یا Propertyهای جدید را با احتیاط اضافه کرد. |
| **Validation** | پشتیبانی نمی‌کند. نمی‌توان روی اجزای Tuple از Data Annotations (مثل `[Required]`) استفاده کرد. | پشتیبانی کامل. به‌راحتی با Data Annotations یا FluentValidation اعتبارسنجی می‌شود . |
| **Documentation** | ضعیف. امکان افزودن XML Comments معنادار به اجزای داخلی Tuple وجود ندارد. | عالی. هر Property می‌تواند دارای توضیحات XML و مستندات Swagger باشد. |
| **Domain Model** | خطرناک. استفاده از آن ممکن است منجر به نشت جزئیات Domain به لایه‌های بیرونی شود. | ایده‌آل. به‌طور خاص برای جداسازی Domain Model از لایه نمایش طراحی شده است. |
| **Application Layer** | محدود. فقط برای ارتباطات داخلی و کوتاه‌مدت بین متدهای خصوصی مناسب است. | استاندارد. بهترین انتخاب برای Commandها و Queryها در الگوهایی مثل CQRS. |
| **Return Type** | مناسب برای متدهای **خصوصی (Private)** که نیاز به بازگرداندن سریع ۲ یا ۳ مقدار دارند. | مناسب برای متدهای **عمومی (Public)** و سرویس‌هایی که خروجی آن‌ها توسط سایر بخش‌ها مصرف می‌شود. |
| **Internal Method** | **عالی**. سربار تعریف کلاس را حذف می‌کند و کد را تمیز و مختصر نگه می‌دارد. | خوب، اما ممکن است برای یک استفاده‌ی بسیار کوچک، تعریف یک کلاس اضافه (Over-engineering) باشد. |
| **Public API** | **ممنوع**. هرگز نباید در امضای متدهای Public یک کتابخانه یا API استفاده شود. | **الزامی**. استاندارد طلایی برای طراحی Public APIها. |
| **قابلیت توسعه** | ندارد. نمی‌توان به یک Tuple موجود Property اضافه کرد یا از Interface ارث‌بری کرد. | عالی. می‌تواند Interface پیاده‌سازی کند، از `record` استفاده کند و به‌راحتی گسترش یابد. |
| **Performance** | **بالاتر** (در موارد خاص). چون `ValueTuple` یک `struct` (نوع مقداری) است، در Heap تخصیص داده نمی‌شود و فشار کمتری به GC وارد می‌کند . | کمی پایین‌تر. کلاس‌ها Reference Type هستند و سربار تخصیص حافظه دارند (هرچند استفاده از `record` این فاصله را کم کرده است). |

---

## ۴. مثال کاربردی در ASP.NET Core

### ❌ سناریوی نامناسب (استفاده از Tuple در Public API)
```csharp
[HttpGet("{id}")]
public async Task<ActionResult<(string UserName, string Email)>> GetUser(int id)
{
    var user = await _dbContext.Users.FindAsync(id);
    
    // مشکل: Swagger نمی‌تواند این ساختار را به‌درستی مستند کند
    // و در JSON خروجی به‌جای UserName و Email، مقادیر item1 و item2 دیده می‌شود.
    return Ok((user.Name, user.Email)); 
}
```

### ✅ سناریوی مناسب (استفاده از DTO)
```csharp
// ۱. تعریف DTO با قابلیت اعتبارسنجی
public class UserResponseDto
{
    [Required]
    public string UserName { get; set; }
    
    [EmailAddress]
    public string Email { get; set; }
}

// ۲. استفاده در Controller
[HttpGet("{id}")]
public async Task<ActionResult<UserResponseDto>> GetUser(int id)
{
    var user = await _dbContext.Users.FindAsync(id);
    
    // نگاشت به DTO (می‌توان از AutoMapper یا Manual Mapping استفاده کرد)
    var dto = new UserResponseDto 
    { 
        UserName = user.Name, 
        Email = user.Email 
    };
    
    return Ok(dto); // خروجی JSON کاملاً معنادار، قابل مستندسازی و اعتبارسنجی
}
```

---

## ۵. راهنمای تصمیم‌گیری (Decision Guide)

برای اینکه همیشه بهترین انتخاب را داشته باشید، از این چک‌لیست سریع استفاده کنید:

✅ **از Tuple استفاده کنید زمانی که:**
- در حال نوشتن یک متد `private` یا `internal` هستید و می‌خواهید ۲ یا ۳ مقدار مرتبط را به‌سرعت برگردانید.
- داده‌ها فقط در محدوده‌ی همان متد یا کلاس مصرف می‌شوند و از مرز لایه‌ها (Layer Boundaries) عبور نمی‌کنند.
- عملکرد (Performance) و اجتناب از تخصیص حافظه در Heap (به دلیل Value Type بودن) در یک حلقه‌ی پرتکرار حیاتی است.
- در حال استفاده از قابلیت‌های مدرن #C مانند Pattern Matching یا Deconstruction هستید.

✅ **از DTO استفاده کنید زمانی که:**
- داده‌ها از مرز یک لایه به لایه‌ی دیگر عبور می‌کنند (مثلاً از Controller به Client، یا از Application به Domain).
- نیاز به اعتبارسنجی (Validation) روی داده‌های ورودی/خروجی دارید.
- می‌خواهید API شما مستندات شفاف و خودکار (Swagger/OpenAPI) داشته باشد.
- داده‌ها نیاز به نسخه‌بندی (Versioning) یا توسعه در آینده دارند.
- می‌خواهید از بروز خطرات امنیتی مانند **Overposting** (دستکاری داده‌های ارسالی توسط کاربر و تزریق به Model اصلی) جلوگیری کنید .

---

## ۶. منابع معتبر Microsoft و .NET

برای مطالعه‌ی عمیق‌تر و اطمینان از رعایت بهترین روش‌ها (Best Practices)، منابع رسمی زیر پیشنهاد می‌شوند:

1. **مستندات رسمی مایکروسافت درباره Tupleها در #C**: توضیح می‌دهد که Tupleها انواع مقداری (Value Types) هستند و برای گروه‌بندی سبک‌وزن مقادیر بدون تعریف نوع نام‌دار استفاده می‌شوند .
   - 🔗 [Tuples and deconstruction - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/tuples)

2. **مستندات رسمی مایکروسافت درباره DTOها**: توضیح می‌دهد که چگونه DTOها لایه سرویس را از لایه پایگاه داده جدا می‌کنند و چگونه داده‌ها باید از طریق شبکه ارسال شوند .
   - 🔗 [Create Data Transfer Objects (DTOs) | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5)

3. **بهترین روش‌های جلوگیری از Overposting با استفاده از DTO**: مایکروسافت به‌طور صریح توصیه می‌کند برای عملیات ایجاد (Insert) یا به‌روزرسانی، از یک ViewModel یا DTO جداگانه استفاده کنید تا از تزریق داده‌های ناخواسته جلوگیری شود .
   - 🔗 [ASP.NET Core forms overview (Overposting mitigation) | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/blazor/forms/?view=aspnetcore-8.0)

4. **محدودیت‌های سریالایزیشن Tuple**: بحث‌های رسمی در مورد اینکه چرا نام‌های معنایی (Semantic names) در `ValueTuple` در زمان اجرا (Runtime) و در فرمت‌هایی مثل JSON حفظ نمی‌شوند و نیاز به راهکارهای جایگزین دارند .
   - 🔗 [Make Value Tuple property names resolvable at runtime | GitHub dotnet/csharplang](https://github.com/dotnet/csharplang/discussions/1906)

---
*نکته برای مخزن آموزشی:* این مقاله با در نظر گرفتن اصول شیءگرایی (OOP) مانند «پنهان‌سازی اطلاعات» (Encapsulation) و «اصل مسئولیت واحد» (SRP) تدوین شده است تا برای برنامه‌نویسان مبتدی تا متوسط، درکی عمیق و کاربردی از معماری تمیز در #C فراهم کند.