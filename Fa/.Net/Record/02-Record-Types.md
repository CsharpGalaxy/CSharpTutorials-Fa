# راهنمای جامع و پیشرفته: الگوهای طراحی و استفاده از `record` در C#

معرفی `record` در C# 9 (و تکامل آن در نسخه‌های 10 تا 12) تنها یک ویژگی زبانی جدید نبود، بلکه یک تغییر پارادایم در نحوه طراحی مدل‌های دامنه، انتقال داده و مدیریت حالت بود. `record`ها ذاتاً **تغییرناپذیر (Immutable)** هستند و **برابری مبتنی بر مقدار (Value-based Equality)** را ارائه می‌دهند.

در این مقاله پیشرفته، بررسی می‌کنیم که چگونه می‌توان از `record` در الگوهای مختلف معماری نرم‌افزار (DDD، CQRS، Clean Architecture) استفاده کرد.

---

## 1. Value Object (شیء مقدار)
**مشکل:** در DDD، اشیایی داریم که هویت (Identity) ندارند و تنها بر اساس ویژگی‌هایشان تعریف می‌شوند (مثل پول، مختصات، بازه زمانی). پیاده‌سازی `Equals`، `GetHashCode` و تغییرناپذیری در `class`های معمولی بسیار پرخطا و زمان‌بر است.
**چرا Record؟** `record`ها به صورت پیش‌فرض برابری مبتنی بر مقدار دارند و تغییرناپذیرند.
**مثال C#:**
```csharp
public record Money(decimal Amount, string Currency)
{
    public static Money operator +(Money a, Money b) => 
        a.Currency == b.Currency ? new Money(a.Amount + b.Amount, a.Currency) 
        : throw new CurrencyMismatchException();
}
```
*   **مزایا:** حذف Boilerplate code، ایمنی در برابر تغییرات ناخواسته، پشتیبانی از `with` expression برای ایجاد کپی با تغییرات جزئی.
*   **معایب:** در صورت نیاز به تغییرات مکرر (Mutation)، ایجاد آبجکت جدید هزینه سربار GC دارد (برای حل این مشکل در سناریوهای High-Performance از `record struct` استفاده کنید).
*   **چه زمانی استفاده نکنیم؟** وقتی شیء شما هویت دارد (Entity است) یا نیاز به تغییرات درجا (Mutation) بدون ایجاد کپی دارد.

## 2. DTO (Data Transfer Object)
**مشکل:** انتقال داده بین لایه‌ها یا فرآیندها. کلاس‌های DTO معمولاً پراپرتی‌های زیادی دارند و نوشتن Constructor برای آن‌ها خسته‌کننده است.
**چرا Record؟** سینتکس فشرده (Positional records) و ایمنی در برابر تغییر داده در حین انتقال بین لایه‌ها.
**مثال C#:**
```csharp
public record UserDto(Guid Id, string Username, string Email);
```
*   **مزایا:** خوانایی بالا، جلوگیری از باگ‌های ناشی از تغییر تصادفی داده در لایه‌های میانی.
*   **معایب:** برخی از Serializers قدیمی یا خاص ممکن است با `record`ها مشکل داشته باشند (هرچند System.Text.Json کاملاً سازگار است).
*   **چه زمانی استفاده نکنیم؟** وقتی فریم‌ورک یا کتابخانه‌ای نیاز به DTO با `setter`های عمومی و قابل تغییر (Mutable) دارد.

## 3. Command (در الگوی CQRS)
**مشکل:** Command نشان‌دهنده یک "قصد" برای تغییر state است. یک Command نباید در حین پردازش تغییر کند.
**چرا Record؟** تضمین می‌کند که پس از ایجاد Command توسط کلاینت، هیچ Handler یا Middlewareای نمی‌تواند مقادیر آن را دستکاری کند.
**مثال C#:**
```csharp
public record CreateOrderCommand(Guid CustomerId, List<OrderItemDto> Items) : IRequest<OrderResult>;
```
*   **مزایا:** ایمنی Thread-safe، پیش‌بینی‌پذیری در Pipeline های MediatR.
*   **معایب:** اگر نیاز به Validation در حین پردازش دارید که state را تغییر دهد، باید از Mutable class استفاده کنید یا Validation را قبل از ایجاد Command انجام دهید.
*   **چه زمانی استفاده نکنیم؟** در Commandهای بسیار پیچیده که نیاز به متدهای داخلی برای محاسبات حین پردازش دارند (هرچند بهتر است این محاسبات در Handler باشد).

## 4. Query (در الگوی CQRS)
**مشکل:** درخواست برای خواندن داده. همچنین، مدل‌های بازگشتی (Read Models) نیاز به ساختار مشخصی دارند.
**چرا Record؟** هم برای خودِ Query و هم برای Read Model (خروجی) عالی است.
**مثال C#:**
```csharp
// Query
public record GetOrderByIdQuery(Guid OrderId) : IRequest<OrderReadModel>;

// Read Model (خروجی)
public record OrderReadModel(Guid Id, decimal TotalPrice, string Status);
```
*   **مزایا:** تطبیق عالی با ASP.NET Core Model Binding و Swagger.
*   **معایب:** مشابه Command.
*   **چه زمانی استفاده نکنیم؟** وقتی خروجی Query نیاز به Lazy Loading یا ارتباطات پیچیده گراف آبجکت دارد.

## 5. Domain Event (رویداد دامنه)
**مشکل:** ثبت این واقعیت که "چیزی در دامنه اتفاق افتاده است". رویدادهای دامنه تاریخچه هستند و تاریخچه نباید تغییر کند.
**چرا Record؟** یک رویداد دامنه یک "فکت" است. `record` این فکت را تغییرناپذیر و قابل مقایسه می‌کند.
**مثال C#:**
```csharp
public record OrderShippedDomainEvent(Guid OrderId, DateTime ShippedAt, string TrackingCode) : INotification;
```
*   **مزایا:** تست‌نویسی آسان (به دلیل Value Equality)، جلوگیری از دستکاری رویداد قبل از Publish شدن.
*   **معایب:** اگر سایز رویداد خیلی بزرگ شود، مصرف حافظه بالا می‌رود.
*   **چه زمانی استفاده نکنیم؟** وقتی رویداد نیاز به ارجاع به Entityهای دامنه دارد (رویداد باید فقط شامل ID و داده‌های خام باشد).

## 6. Integration Event (رویداد یکپارچه‌سازی)
**مشکل:** ارتباط بین Bounded Contextها یا میکروسرویس‌ها از طریق Message Broker. داده‌ها باید Serializable و Flat باشند.
**چرا Record؟** ساختار مسطح و Immutable آن برای Serializing/Deserializing در RabbitMQ/Kafka ایده‌آل است.
**مثال C#:**
```csharp
public record OrderShippedIntegrationEvent(Guid EventId, Guid OrderId, DateTime OccurredOn, string TrackingCode);
```
*   **مزایا:** سازگاری عالی با `System.Text.Json`، جلوگیری از تغییر داده در حین انتقال بین سرویس‌ها.
*   **معایب:** در صورت نیاز به Versioning پیچیده (مثلاً اضافه کردن فیلد اختیاری در نسخه 2)، `record`ها به تنهایی کافی نیستند و نیاز به Custom Converters دارید.
*   **چه زمانی استفاده نکنیم؟** وقتی ساختار داده سلسله‌مراتبی (Hierarchical) و پیچیده است و Polymorphism نیاز دارید (مگر اینکه از `JsonDerivedType` استفاده کنید).

## 7. Message (پیام در صف)
**مشکل:** بسته‌بندی داده‌ها به همراه Metadata (مثل شناسه پیام، زمان ارسال، نوع پیام) برای ارسال به صف.
**چرا Record؟** استفاده از الگوی Envelope با `record`ها بسیار تمیز است.
**مثال C#:**
```csharp
public record MessageEnvelope<T>(Guid MessageId, string MessageType, DateTime SentAt, T Payload);
```
*   **مزایا:** تایپ‌سیفتی (Type-safety) برای Payload.
*   **معایب:** سربار حافظه برای Wrapper.
*   **چه زمانی استفاده نکنیم؟** وقتی از فریم‌ورک‌های Message Broker خاصی (مثل MassTransit) استفاده می‌کنید که خودشان Envelope را مدیریت می‌کنند.

## 8. API Contract (Request/Response)
**مشکل:** تعریف مرزهای API. Requestها و Responseها باید ساختار مشخصی داشته باشند.
**چرا Record؟** در ASP.NET Core Minimal APIs و Controllerها، `record`ها بهترین گزینه برای Model Binding هستند.
**مثال C#:**
```csharp
public record CreateUserRequest(string Name, string Email);
public record CreateUserResponse(Guid Id, DateTime CreatedAt);
```
*   **مزایا:** تولید خودکار و دقیق Swagger/OpenAPI Schema.
*   **معایب:** اگر نیاز به Default Valueهای پیچیده یا Logic در Constructor داشته باشید، سینتکس Positional کمی محدود می‌شود.
*   **چه زمانی استفاده نکنیم؟** در APIهای قدیمی که از Model Binding با فرم‌های HTML و Mutable objects استفاده می‌کنند.

## 9. Configuration (تنظیمات)
**مشکل:** تنظیمات برنامه (AppSettings) معمولاً در طول اجرای برنامه خوانده می‌شوند. تغییر تصادفی آن‌ها در Runtime می‌تواند فاجعه‌بار باشد.
**چرا Record؟** ترکیب `record` با `IOptions<T>` در .NET تضمین می‌کند که تنظیمات فقط یکبار بارگذاری و تغییرناپذیر می‌مانند.
**مثال C#:**
```csharp
public record DatabaseSettings(string ConnectionString, int TimeoutSeconds, bool EnableRetry);
```
*   **مزایا:** جلوگیری از باگ‌های ناشی از تغییر تنظیمات در Runtime.
*   **معایب:** اگر نیاز به Hot-Reloading تنظیمات (تغییر در appsettings.json و اعمال بلافاصله) دارید، `record`های Immutable این کار را سخت می‌کنند (باید از `IOptionsSnapshot` یا `IOptionsMonitor` با دقت استفاده کنید).
*   **چه زمانی استفاده نکنیم؟** وقتی تنظیمات شما به شدت پویا هستند و در Runtime توسط کاربر نهایی تغییر می‌کنند.

## 10. Immutable State (حالت تغییرناپذیر)
**مشکل:** مدیریت State در UI (مثل Blazor) یا State Machineها. تغییرات جزئی در State نباید State قبلی را خراب کند.
**چرا Record؟** عبارت `with` در `record`ها برای ایجاد State جدید بر اساس State قبلی معجزه می‌کند.
**مثال C#:**
```csharp
public record ShoppingCartState(decimal Total, int ItemCount, bool IsCheckedOut)
{
    public ShoppingCartState AddItem(decimal price) => this with { Total += price, ItemCount++ };
}
```
*   **مزایا:** مدیریت State بسیار تمیز، مناسب برای Time-travel debugging و Undo/Redo.
*   **معایب:** برای Stateهای بسیار بزرگ و عمیق، کپی کردن (Shallow copy در `with`) ممکن است نیاز به Deep Clone دستی داشته باشد.
*   **چه زمانی استفاده نکنیم؟** در سناریوهایی که State باید بین چندین Thread به اشتراک گذاشته شود و نیاز به Lock کردن دارد (هرچند Immutable بودن خود نوعی Thread-safety است).

## 11. Result Type (Railway Oriented Programming)
**مشکل:** مدیریت خطاها بدون استفاده از Exception (که پرهزینه است). بازگرداندن either Success or Failure.
**چرا Record؟** C# از Discriminated Unions پشتیبانی نمی‌کند، اما می‌توان با استفاده از `abstract record` و `sealed record` آن را شبیه‌سازی کرد.
**مثال C#:**
```csharp
public abstract record Result
{
    public sealed record Success<T>(T Value) : Result;
    public sealed record Failure(string Error) : Result;
}

// استفاده:
public Result<User> GetUser(int id) => 
    id > 0 ? new Result.Success<User>(new User(id)) : new Result.Failure("Invalid ID");
```
*   **مزایا:** اجبار کامپایلر به بررسی تمام حالت‌ها (با استفاده از Pattern Matching)، حذف Null Reference Exceptions.
*   **معایب:** پیچیدگی بیشتر در سینتکس نسبت به try/catch ساده.
*   **چه زمانی استفاده نکنیم؟** در خطاهای غیرمنتظره (Unexpected Errors) که باید برنامه متوقف شود (برای آن‌ها Exception بهتر است).

---

## تفاوت Domain Event و Integration Event و نقش Record

درک تفاوت این دو در معماری میکروسرویس و DDD حیاتی است:

| ویژگی | Domain Event (رویداد دامنه) | Integration Event (رویداد یکپارچه‌سازی) |
| :--- | :--- | :--- |
| **محدوده (Scope)** | داخل یک Bounded Context (یک میکروسرویس). | بین Bounded Contextها یا میکروسرویس‌های مختلف. |
| **حامل داده** | In-Memory (مثل MediatR). | Message Broker (مثل RabbitMQ, Kafka). |
| **ساختار داده** | می‌تواند شامل اشیاء غنی دامنه (Value Objects) باشد. | باید **Flat** و فقط شامل Primitive Types یا DTOهای ساده باشد. |
| **زبان** | زبان غنی دامنه (Ubiquitous Language). | زبان مشترک و قراردادی بین سرویس‌ها (API Contract). |
| **نقش `record`** | تضمین می‌کند که "فکت" رخ داده در حافظه دستکاری نمی‌شود. از `with` برای enrich کردن رویداد قبل از publish استفاده می‌شود. | تضمین می‌کند که ساختار داده برای Serialization/Deserialization ایمن است و در حین انتقال در شبکه تغییر نمی‌کند. |

**نکته معماری:** معمولاً یک Domain Event در یک Integration Event "ترجمه" (Map) می‌شود. هر دو می‌توانند `record` باشند، اما Integration Event باید از پیچیدگی‌های دامنه دوری کند.

---

## Architecture Examples (مثال‌های معماری)

### مثال 1: جریان داده در Clean Architecture + CQRS
فرض کنید کاربر دکمه "ثبت سفارش" را می‌زند:

1.  **API Contract (Record):**
    ```csharp
    public record CreateOrderRequest(Guid CustomerId, List<Guid> ProductIds);
    ```
2.  **Command (Record):** در لایه Application ساخته می‌شود.
    ```csharp
    public record CreateOrderCommand(Guid CustomerId, List<Guid> ProductIds) : IRequest<Guid>;
    ```
3.  **Domain Event (Record):** پس از موفقیت‌آمیز بودن ایجاد Order در لایه Domain.
    ```csharp
    public record OrderCreatedDomainEvent(Guid OrderId, Guid CustomerId, DateTime OccurredOn) : INotification;
    ```
4.  **Integration Event (Record):** توسط یک Event Handler از Domain Event ساخته شده و به صف ارسال می‌شود.
    ```csharp
    public record OrderCreatedIntegrationEvent(Guid EventId, Guid OrderId, DateTime OccurredOn);
    ```

### مثال 2: الگوی Outbox برای تضمین تحویل پیام (Reliable Messaging)
برای جلوگیری از از دست رفتن Integration Eventها، آن‌ها را در دیتابیس ذخیره می‌کنیم. `record`ها برای ذخیره در Outbox Table عالی هستند:

```csharp
// Outbox Message Entity
public record OutboxMessage(
    Guid Id, 
    string Type, 
    string Content, // JSON serialized Integration Event
    DateTime OccurredOn, 
    DateTime? ProcessedOn = null);
```
چون `record` است، تضمین می‌شود که پس از ذخیره در دیتابیس، هیچ پراستس دیگری نمی‌تواند فیلد `Content` یا `OccurredOn` آن را تغییر دهد.

---

## منابع معتبر برای مطالعه بیشتر

### Microsoft Learn & .NET Documentation
1.  **Records (C# Reference):** [Microsoft Learn - Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) (بررسی دقیق تفاوت `record class` و `record struct`).
2.  **CQRS Pattern:** [Microsoft Learn - CQRS](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs) (نحوه استفاده از Commandها و Queryها).
3.  **Outbox Pattern:** [Microsoft Learn - Outbox Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/transaction-outbox) (ترکیب Domain و Integration Eventها).

### کتاب‌ها و منابع DDD / Software Architecture
1.  **Domain-Driven Design Distilled** - *Vaughn Vernon* (برای درک عمیق Value Objects و Domain Events).
2.  **Implementing Domain-Driven Design** - *Vaughn Vernon* (معروف به کتاب قرمز DDD؛ فصل‌های مربوط به Events و Integration عالی هستند).
3.  **Clean Architecture** - *Robert C. Martin* (برای درک مرزها و نحوه استفاده از DTOها و API Contracts).
4.  **Patterns, Principles, and Practices of Domain-Driven Design** - *Scott Millett & Nick Tune* (برای تطبیق DDD با C# و الگوهای CQRS).
5.  **MediatR Documentation & Jimmy Bogard's Blog:** برای درک نحوه پیاده‌سازی CQRS و Domain Events در دنیای واقعی .NET.

### نتیجه‌گیری
استفاده از `record` در C# دیگر فقط یک "ترفند سینتکسی" نیست، بلکه یک **انتخاب معماری** است. هر جا که با **داده‌های تغییرناپذیر (Immutable Data)**، **فکت‌های تاریخی (Events)**، **قراردادهای مرزی (Contracts)** و **اشیاء مبتنی بر مقدار (Value Objects)** سر و کار دارید، `record` باید انتخاب اول شما باشد. این کار باعث می‌شود کد شما ایمن‌تر، خواناتر و بسیار نزدیک‌تر به مفاهیم DDD و Clean Architecture باشد.