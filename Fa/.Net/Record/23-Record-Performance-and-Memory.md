# آموزش عملی: استفاده از `Record` و JSON Serialization در .NET

در این آموزش، نحوه استفاده از انواع `Record` در #C را همراه با `System.Text.Json` برای ساخت قراردادهای API (API Contracts) در ASP.NET Core بررسی می‌کنیم. این رویکرد کدی تمیز، ایمن و غیرقابل تغییر (Immutable) را برای لایه انتقال داده (DTO) فراهم می‌کند.

---

## ۱. مفاهیم پایه
- **Serialization (سریال‌سازی)**: فرآیند تبدیل یک شیء .NET (مانند یک `Record`) به یک فرمت قابل انتقال و ذخیره‌سازی مانند رشته JSON .
- **Deserialization (بازسازی/دی‌سریال‌سازی)**: فرآیند معکوس؛ تبدیل داده‌های JSON دریافتی (مثلاً از بدنه درخواست HTTP) به یک شیء .NET با نوع مشخص .

---

## ۲. آشنایی با انواع `Record` در #C
کلمه کلیدی `record` قابلیت‌های داخلی برای کپسوله‌سازی داده و برابری مبتنی بر مقدار (Value Equality) را فراهم می‌کند .

- **Record Class**: نوع مرجع (Reference Type) است. نوشتن `record` به‌تنهایی، مخفف `record class` است . این نوع برای مدل‌های داده و DTOها که رفتار Reference Type با آن‌ها همخوانی بیشتری دارد، توصیه می‌شود .
- **Record Struct**: نوع مقداری (Value Type) است که با `record struct` تعریف می‌شود و برای داده‌های کوچک با کارایی بالا و کپی‌شدن بر اساس مقدار مناسب است .
- **Positional Record**: روشی مختصر برای تعریف رکورد با پارامترهای سازنده (Constructor Parameters). کامپایلر به‌طور خودکار این پارامترها را به Propertyهای عمومی با دسترسی `init` تبدیل می‌کند .
- **Constructor Parameters**: پارامترهایی که در امضای اصلی رکورد تعریف می‌شوند و مقداردهی اولیه را اجباری می‌کنند.
- **init Properties**: ویژگی‌هایی که فقط در لحظه ساخت شیء (Object Initializer) یا فرآیند Deserialization قابل تنظیم هستند و پس از آن غیرقابل تغییر (Immutable) باقی می‌مانند.
- **Required Properties**: با استفاده از کلمه کلیدی `required`، تضمین می‌کنیم که یک Property حتماً در زمان ساخت شیء یا در فرآیند Deserialization مقداردهی شود .

---

## ۳. کار با `System.Text.Json` و `Record`ها
برای کنترل نام Propertyها در JSON، از ویژگی `[JsonPropertyName]` استفاده می‌شود . در `Positional Record`ها، برای اعمال این ویژگی روی Property تولیدشده، باید از سینتکس `[property: ...]` استفاده کرد.

### مثال واقعی: تعریف Request و Response DTO
```csharp
using System.Text.Json.Serialization;

// ۱. Request DTO (Positional Record)
public record CreateUserRequest(
    [property: JsonPropertyName("full_name")] string Name,
    [property: JsonPropertyName("email_address")] string Email
);

// ۲. Response DTO (Record با بدنه صریح و Required Properties)
public record UserResponse
{
    [JsonPropertyName("id")]
    public required Guid Id { get; init; }
    
    [JsonPropertyName("status")]
    public string Status { get; init; } = "Active";
}
```

### نحوه تبدیل JSON به Record (Deserialization)
```csharp
using System.Text.Json;

string jsonPayload = @"{
    ""full_name"": ""علی احمدی"",
    ""email_address"": ""ali@example.com""
}";

// تنظیمات برای نادیده گرفتن حساسیت به حروف بزرگ/کوچک (اختیاری اما توصیه‌شده)
var options = new JsonSerializerOptions 
{ 
    PropertyNameCaseInsensitive = true 
};

// تبدیل JSON به Record
CreateUserRequest request = JsonSerializer.Deserialize<CreateUserRequest>(jsonPayload, options);

Console.WriteLine(request.Name); // خروجی: علی احمدی
```

---

## ۴. استفاده در ASP.NET Core API
استفاده از `Record` به عنوان DTO در کنترلرها، کد را خوانا و از تغییرات ناخواسته داده‌ها در طول پردازش جلوگیری می‌کند.

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult CreateUser([FromBody] CreateUserRequest request)
    {
        // اعتبارسنجی خودکار توسط مدل‌بایندر انجام می‌شود
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        // شبیه‌سازی ذخیره در پایگاه داده
        var response = new UserResponse 
        { 
            Id = Guid.NewGuid(), 
            Status = "Created" 
        };

        // بازگرداندن Response DTO (به‌طور خودکار توسط ASP.NET Core به JSON سریال می‌شود)
        return CreatedAtAction(nameof(GetUser), new { id = response.Id }, response);
    }

    [HttpGet("{id}")]
    public IActionResult GetUser(Guid id)
    {
        var response = new UserResponse { Id = id, Status = "Active" };
        return Ok(response);
    }
}
```

---

## ۵. مشکلات احتمالی Deserialization و راه‌حل‌ها

1. **حساسیت به حروف (Case Sensitivity)**:  
   `System.Text.Json` به‌طور پیش‌فرض به حروف بزرگ و کوچک حساس است. اگر JSON دارای `full_name` باشد اما پارامتر رکورد `FullName` نامیده شود و `[JsonPropertyName]` استفاده نشده باشد، مقداردهی انجام نمی‌شود.  
   ✅ *راه‌حل*: استفاده از `[property: JsonPropertyName("full_name")]` یا تنظیم سراسری `PropertyNameCaseInsensitive = true`.

2. **عدم تطابق نام پارامترهای سازنده (Constructor)**:  
   در `Positional Record`ها، `System.Text.Json` ابتدا سعی می‌کند نام فیلدهای JSON را با نام پارامترهای سازنده (نه Propertyها) تطبیق دهد .  
   ✅ *راه‌حل*: همان‌طور که در مثال بالا نشان داده شد، از `[property: JsonPropertyName("...")]` روی پارامترهای Positional استفاده کنید تا هم نام Property و هم رفتار Deserialization کنترل شود.

3. **فیلدهای اضافی یا ناشناخته در JSON**:  
   به‌طور پیش‌فرض، `System.Text.Json` فیلدهای اضافی را نادیده می‌گیرد که ممکن است باعث پنهان‌ماندن حملات یا خطاهای خاموش شود.  
   ✅ *راه‌حل*: در .NET 8+ می‌توانید از حالت Strict Mode استفاده کنید یا `UnknownTypeHandling` را پیکربندی نمایید تا در صورت وجود فیلد ناشناخته، خطا صادر شود.

4. **مقدار `null` برای نوع‌های غیرقابل‌قبول (Non-nullable)**:  
   اگر JSON مقدار `null` بفرستد اما Property از نوع `string` (بدون `?`) باشد، ممکن است باعث رفتار ناخواسته شود.  
   ✅ *راه‌حل*: ترکیب `required` با Nullable Reference Types (مثل `string?` برای فیلدهای اختیاری) تا کامپایلر و `System.Text.Json` وجود و صحت داده را تضمین کنند .

---

## ۶. بهترین روش‌ها (Best Practices) برای استفاده از Record به عنوان API Contract

1. **اولویت با `record class` برای DTOها**: برای مدل‌های درخواست و پاسخ، `record` (که همان `record class` است) ترجیح داده می‌شود، زیرا رفتار Reference Type با نحوه استفاده معمول از مدل‌های داده در وب هماهنگ‌تر است .
2. **استفاده از `required` برای فیلدهای اجباری**: به جای نوشتن منطق اعتبارسنجی دستی، از `required` استفاده کنید. این کار باعث می‌شود اگر فیلد در JSON payload وجود نداشته باشد، `System.Text.Json` به‌طور خودکار خطا صادر کند .
3. **جداسازی نام‌گذاری #C از استاندارد JSON**: همیشه از `[JsonPropertyName]` استفاده کنید تا بتوانید از استاندارد PascalCase در #C و camelCase یا snake_case در JSON به‌طور همزمان پشتیبانی کنید .
4. **حفظ سادگی (KISS)**: رکوردها باید حامل داده (Data Carriers) باشند. از افزودن منطق تجاری پیچیده، متدهای بزرگ یا Stateهای متغیر به آن‌ها خودداری کنید.
5. **پیکربندی سراسری در `Program.cs`**: برای یکپارچگی، تنظیمات JSON را در سطح برنامه اعمال کنید:
   ```csharp
   builder.Services.Configure<Microsoft.AspNetCore.Http.Json.JsonOptions>(options =>
   {
       options.JsonSerializerOptions.PropertyNameCaseInsensitive = true;
       options.JsonSerializerOptions.WriteIndented = true; // برای خوانایی در محیط توسعه
   });
   ```

---

## ۷. منابع رسمی Microsoft Learn
برای مطالعه عمیق‌تر و به‌روز، می‌توانید به مستندات رسمی مایکروسافت مراجعه کنید:

1. [مرجع کامل انواع Record در #C](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) 
2. [آموزش عملی استفاده از Recordها](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/records) 
3. [بررسی کلی سریال‌سازی و دی‌سریال‌سازی با System.Text.Json](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/overview) 
4. [نحوه شخصی‌سازی نام Propertyها با JsonPropertyName](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/customize-properties) 
5. [الزامی کردن Propertyها برای Deserialization (ویژگی required)](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/required-properties) 
6. [نحوه دی‌سریال‌سازی JSON در #C](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/deserialization) 

این ساختار به شما کمک می‌کند تا APIهایی مقاوم، خوانا و منطبق با استانداردهای مدرن .NET طراحی کنید. اگر نیاز به مثال خاصی در مورد سناریوی پیچیده‌تری (مانند رکوردهای تودرتو یا ارث‌بری) دارید، بفرمایید!