این یک آموزش عملی و جامع درباره کاربرد `record` و `record struct` در معماری نرم‌افزار با تمرکز بر C# و ASP.NET Core است. با توجه به سطح متوسط و رویکرد آموزشی شما، این راهنما به گونه‌ای ساختار یافته که هم مبانی نظری (چرایی) و هم پیاده‌سازی عملی (چگونگی) را پوشش دهد.

---

# کاربرد Record در معماری نرم‌افزار: یک راهنمای عملی

در C# (از نسخه ۹ به بعد)، `record` به عنوان یک نوع مرجعی (Reference Type) معرفی شد که **تساوی مبتنی بر مقدار (Value-based Equality)** و **تغییرناپذیری (Immutability)** را به صورت پیش‌فرض ارائه می‌دهد. در C# 10، `record struct` نیز به عنوان یک نوع مقداری (Value Type) با همان ویژگی‌ها اضافه شد. این ویژگی‌ها آن‌ها را به ابزاری ایده‌آل برای مدل‌سازی داده‌ها در معماری‌های مدرن تبدیل کرده است.

در ادامه، ۱۱ کاربرد رایج را بررسی می‌کنیم:

---

### ۱. DTO (Data Transfer Object)
- **آیا Record مناسب است؟** بله، بسیار مناسب.
- **چرا؟** DTOها فقط داده را حمل می‌کنند. تغییرناپذیری آن‌ها از تغییر تصادفی داده‌ها در لایه‌های مختلف جلوگیری می‌کند و تساوی مبتنی بر مقدار، تست‌نویسی (Unit Testing) را بسیار ساده‌تر می‌کند.
- **مثال C#:**
  ```csharp
  public record UserDto(int Id, string FirstName, string LastName);
  ```
- **چه زمانی Class بهتر است؟** زمانی که DTO نیاز به تغییرپذیری (Mutability) گسترده دارد یا از سلسله‌مراتب ارث‌بری پیچیده‌ای استفاده می‌کند که با محدودیت‌های Record سازگار نیست.
- **چه زمانی Record Struct بهتر است؟** زمانی که حجم عظیمی از DTOها به صورت آرایه‌ای پردازش می‌شوند و سربای تخصیص حافظه (Heap Allocation) گلوگاه عملکردی ایجاد کرده است.

---

### ۲. Request Model (الگوی درخواست)
- **آیا Record مناسب است؟** بله.
- **چرا؟** درخواست‌های ورودی باید پس از دریافت و اعتبارسنجی، تغییر نکنند. ASP.NET Core Model Binding به‌طور کامل از سازنده‌های اصلی (Primary Constructors) در Recordها پشتیبانی می‌کند.
- **مثال C# (ASP.NET Core):**
  ```csharp
  [HttpPost("register")]
  public async Task<IActionResult> Register([FromBody] RegisterUserRequest request)
  {
      // request.Immutable و ایمن است
      return Ok();
  }

  public record RegisterUserRequest(string Username, string Email, string Password);
  ```
- **چه زمانی Class بهتر است؟** زمانی که فریم‌ورک‌های قدیمی Serialization نیاز به سازنده بدون پارامتر (Parameterless) و Propertyهای Setter دارند (هرچند System.Text.Json در .NET مدرن این مشکل را حل کرده است).
- **چه زمانی Record Struct بهتر است؟** برای درخواست‌های بسیار کوچک و ساده (مثلاً فقط یک `Id`) که می‌خواهید از تخصیص حافظه در هر درخواست HTTP جلوگیری کنید.

---

### ۳. Response Model (الگوی پاسخ)
- **آیا Record مناسب است؟** بله.
- **چرا؟** پاسخ‌ها پس از ساخت نباید تغییر کنند. این موضوع از تغییرات جانبی (Side Effects) قبل از Serialization جلوگیری می‌کند.
- **مثال C#:**
  ```csharp
  public record ApiResponse<T>(T Data, bool IsSuccess, string? Message = null);
  // استفاده: return new ApiResponse<UserDto>(user, true);
  ```
- **چه زمانی Class بهتر است؟** زمانی که پاسخ نیاز به محاسبات تنبل (Lazy Loading) یا منطق پیچیده درون Propertyها دارد.
- **چه زمانی Record Struct بهتر است؟** به ندرت. سربای شبکه (Network I/O) بسیار بیشتر از سربای تخصیص یک Reference Type است، بنابراین استفاده از `record` معمولی کافی و خواناتر است.

---

### ۴. API Contract (قرارداد API)
- **آیا Record مناسب است؟** بله.
- **چرا؟** قراردادها باید شکل داده‌ها را به وضوح بیان کنند. `record` به طور ضمنی به سایر توسعه‌دهندگان می‌گوید: "این فقط داده است، نه رفتار".
- **مثال C#:**
  ```csharp
  public record ApiErrorContract(int StatusCode, string Title, string Detail);
  ```
- **چه زمانی Class بهتر است؟** زمانی که قرارداد نیاز به متدهای کمکی یا رفتارهای پیچیده دارد (که البته نقض اصل Single Responsibility محسوب می‌شود).
- **چه زمانی Record Struct بهتر است؟** معمولاً توصیه نمی‌شود، مگر در میکروسرویس‌هایی با کارایی بسیار بالا (High-Performance) که میلیون‌ها قرارداد در ثانیه پردازش می‌شوند.

---

### ۵. Command (در الگوی CQRS)
- **آیا Record مناسب است؟** بله، انتخاب اول.
- **چرا؟** یک Command نشان‌دهنده "قصد" (Intent) است. این قصد نباید در حین عبور از Pipelineها (مثلاً در MediatR) تغییر کند.
- **مثال C#:**
  ```csharp
  public record CreateOrderCommand(Guid CustomerId, List<OrderItemDto> Items) : IRequest<Result>;
  ```
- **چه زمانی Class بهتر است؟** زمانی که Command نیاز به ساخت تدریجی (Incremental Building) دارد و از الگوی Builder با حالت تغییرپذیر استفاده می‌کنید.
- **چه زمانی Record Struct بهتر است؟** برای Commandهای ساده با فیلدهای کم (مثلاً `DeleteOrderCommand(Guid Id)`) تا فشار روی Garbage Collector کاهش یابد.

---

### ۶. Query (در الگوی CQRS)
- **آیا Record مناسب است؟** بله.
- **چرا؟** مشابه Command، Queryها درخواست‌های تغییرناپذیر برای دریافت داده هستند.
- **مثال C#:**
  ```csharp
  public record GetOrderByIdQuery(Guid OrderId) : IRequest<OrderDto>;
  ```
- **چه زمانی Class بهتر است؟** به ندرت. شاید زمانی که Query دارای فیلترهای داینامیک پیچیده‌ای است که به صورت حالت‌دار (Stateful) ساخته می‌شوند.
- **چه زمانی Record Struct بهتر است؟** برای Queryهای بسیار ساده (مانند جستجو بر اساس یک کلید اولیه).

---

### ۷. Event (Integration Event / پیام‌های سیستمی)
- **آیا Record مناسب است؟** بله، بسیار مناسب.
- **چرا؟** رویدادها "حقایقی" هستند که در گذشته رخ داده‌اند. یک واقعیت هرگز تغییر نمی‌کند. تغییرناپذیری در اینجا یک الزام معماری است.
- **مثال C#:**
  ```csharp
  public record OrderShippedEvent(Guid OrderId, DateTime ShippedAt, string TrackingCode);
  ```
- **چه زمانی Class بهتر است؟** تقریباً هرگز برای خودِ Payload رویداد.
- **چه زمانی Record Struct بهتر است؟** در سیستم‌های با رویدادهای بسیار پرتعداد (High-Frequency) مانند تله‌متری یا پردازش جریان داده (Stream Processing) که کاهش تخصیص حافظه حیاتی است.

---

### ۸. Message (پیام‌های صف پیام / Message Queue)
- **آیا Record مناسب است؟** بله.
- **چرا؟** پیام‌ها در RabbitMQ، Azure Service Bus یا Kafka باید قراردادهای داده‌ای تغییرناپذیر باشند.
- **مثال C#:**
  ```csharp
  public record SendEmailMessage(string To, string Subject, string Body);
  ```
- **چه زمانی Class بهتر است؟** اگر از کتابخانه‌های قدیمی Serialization استفاده می‌کنید که از سازنده‌های اصلی Record پشتیبانی نمی‌کنند.
- **چه زمانی Record Struct بهتر است؟** هنگام ارسال حجم عظیمی از پیام‌های کوچک و بهینه‌سازی حافظه در Producer/Consumer.

---

### ۹. Configuration (پیکربندی)
- **آیا Record مناسب است؟** بله.
- **چرا؟** تنظیمات برنامه باید پس از راه‌اندازی (Startup) تغییرناپذیر باشند تا از تغییرات ناگهانی و Race Condition جلوگیری شود. .NET 8+ به‌طور بومی از Binding به Recordها پشتیبانی می‌کند.
- **مثال C# (ASP.NET Core):**
  ```csharp
  var settings = builder.Configuration.GetSection("AppSettings").Get<AppSettings>();
  
  public record AppSettings(string ApiKey, int MaxRetries, bool EnableLogging);
  ```
- **چه زمانی Class بهتر است؟** در پروژه‌های قدیمی‌تر (.NET Framework یا .NET Core 3.1) که مکانیزم Configuration Binding با سازنده‌های Record مشکل داشت.
- **چه زمانی Record Struct بهتر است؟** توصیه نمی‌شود. تنظیمات یک‌بار خوانده و در حافظه کش می‌شوند، بنابراین سربای Heap Allocation ناچیز است.

---

### ۱۰. Domain Event (رویداد دامنه در DDD)
- **آیا Record مناسب است؟** بله، بهترین انتخاب.
- **چرا؟** در Domain-Driven Design، رویدادهای دامنه باید تغییرناپذیر باشند و تساوی مبتنی بر مقدار برای تست کردن رفتارهای دامنه (Domain Behavior) ضروری است.
- **مثال C#:**
  ```csharp
  public record UserRegisteredDomainEvent(Guid UserId, string Email, DateTime OccurredOn);
  ```
- **چه زمانی Class بهتر است؟** هرگز. تغییرپذیری در رویداد دامنه یک ضدالگو (Anti-Pattern) است.
- **چه زمانی Record Struct بهتر است؟** فقط اگر پروفایلینگ (Profiling) نشان دهد که تخصیص رویدادهای دامنه گلوگاه عملکردی شده است (که به ندرت پیش می‌آید).

---

### ۱۱. Value Object (شیء مقداری در DDD)
- **آیا Record مناسب است؟** بله، اما `record struct` اغلب **بهترین** انتخاب است.
- **چرا؟** اشیاء مقداری بر اساس ویژگی‌هایشان تعریف می‌شوند، نه هویتشان (مانند `Money` یا `EmailAddress`). آن‌ها باید تغییرناپذیر باشند و تساوی مبتنی بر مقدار داشته باشند. `record struct` این ویژگی‌ها را بدون سربای تخصیص Heap ارائه می‌دهد.
- **مثال C#:**
  ```csharp
  public readonly record struct Money(decimal Amount, string Currency);
  // یا در C# 12 با سازنده اصلی:
  public readonly record struct EmailAddress(string Value);
  ```
- **چه زمانی Class بهتر است؟** زمانی که شیء مقداری حاوی مجموعه‌های بزرگ (مثل لیست‌های تو در تو) است که کپی کردن آن‌ها در Value Typeها پرهزینه است.
- **چه زمانی Record Struct بهتر است؟** **این مورد استفاده اصلی و ایده‌آل برای `record struct` است.** همیشه برای Value Objectهای ساده از `readonly record struct` استفاده کنید.

---

## جدول ماتریس تصمیم‌گیری (Decision Matrix)

| کاربرد | نوع پیشنهادی | دلیل اصلی | نیاز به تغییرناپذیری |
| :--- | :--- | :--- | :--- |
| **DTO** | `record` | تساوی مبتنی بر مقدار، خوانایی، ایمنی در انتقال | بالا |
| **Request Model** | `record` | سازگاری عالی با Model Binding در ASP.NET Core | بالا |
| **Response Model** | `record` | جلوگیری از تغییر تصادفی قبل از Serialization | بالا |
| **API Contract** | `record` | بیان واضح "این فقط داده است" | بالا |
| **Command (CQRS)** | `record` | حفظ "قصد" تغییرناپذیر در طول Pipeline | بسیار بالا |
| **Query (CQRS)** | `record` یا `record struct` | سادگی و ایمنی؛ `struct` برای کوئری‌های تک‌فیلدی | بسیار بالا |
| **Integration Event** | `record` | رویدادها واقعیت‌های تغییرناپذیر گذشته هستند | مطلق |
| **Message Queue** | `record` | قرارداد داده‌ای استاندارد و ایمن | بسیار بالا |
| **Configuration** | `record` | ایمنی پس از راه‌اندازی برنامه | بالا |
| **Domain Event** | `record` | الزام معماری DDD برای تغییرناپذیری | مطلق |
| **Value Object** | `readonly record struct` | تساوی مقداری + کارایی بالا (بدون سربای Heap) | مطلق |

---

## نکات کلیدی برای آموزش (OOP)
1. **تساوی مقداری (Value Equality)**: دو `record` با مقادیر یکسان، برابر (`==`) در نظر گرفته می‌شوند، حتی اگر مراجع (References) متفاوتی داشته باشند. این موضوع تست‌نویسی را متحول می‌کند.
2. **تغییرناپذیری (Immutability)**: پراپرتی‌های تعریف‌شده در سازنده اصلی (Primary Constructor) به‌طور پیش‌فرض `init` هستند. برای تغییر یک کپی از رکورد، از عبارت `with` استفاده کنید:  
   `var updatedUser = user with { LastName = "NewName" };`
3. **کارایی**: اگر شیء شما کوچک است (زیر ۱۶ بایت) و فقط داده را نگه می‌دارد (مثل Value Object)، `readonly record struct` را به `record` ترجیح دهید تا از فشار روی Garbage Collector بکاهید.

---

## منابع معتبر و مستندات رسمی
برای مطالعه عمیق‌تر و ارجاع در آموزش‌های خود، این منابع رسمی مایکروسافت توصیه می‌شوند:

1. **مرجع انواع Record در C#**:  
   [Record types - C# Reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
2. **تفاوت Record و Record Struct**:  
   [Record structs - C# Reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record-struct)
3. **تغییرناپذیری (Immutability) در C#**:  
   [What's new in C# 9.0: Record types | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#record-types)
4. **Model Binding در ASP.NET Core با Recordها**:  
   [Model Binding in ASP.NET Core | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding)

اگر نیاز به گسترش هر یک از این بخش‌ها با مثال‌های پیچیده‌تر (مثل استفاده از Recordها در Entity Framework Core یا الگوی Builder) دارید، خوشحال می‌شوم کمک کنم.