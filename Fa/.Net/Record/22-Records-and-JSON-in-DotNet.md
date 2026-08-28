# پیاده‌سازی Interface در Recordها: راهنمای جامع و کاربردی

رکوردها (Records) که از C# 9 معرفی شدند و در C# 10 با اضافه شدن `record struct` تکامل یافتند، انقلابی در نحوه مدیریت داده‌های تغییرناپذیر (Immutable) ایجاد کردند. از طرف دیگر، Interfaceها ستون فقرات طراحی مبتنی بر قرارداد (Contract-based Design) در .NET هستند.

در این مقاله آموزشی، به بررسی ترکیب این دو مفهوم قدرتمند می‌پردازیم و می‌بینیم چگونه استفاده از Interface در Recordها می‌تواند معماری نرم‌افزار شما را تمیزتر و منعطف‌تر کند.

---

## آیا Record می‌تواند Interface پیاده‌سازی کند؟

**پاسخ کوتاه: بله، قطعاً.**
رکوردها در نهایت زیر کاپوت، کلاس یا ساختار (Struct) هستند. بنابراین، تمام قوانینی که برای پیاده‌سازی Interface در کلاس‌ها و استراکت‌ها صدق می‌کند، برای رکوردها نیز کاملاً معتبر است.

---

## Record Class و Interface

رکوردهای کلاسی (Reference Type) برای مدل‌های داده‌ای پیچیده‌تر یا زمانی که می‌خواهیم از وراثت (Inheritance) استفاده کنیم، مناسب هستند.

```csharp
// تعریف Interface
public interface IAuditable
{
    DateTime CreatedAt { get; }
    string CreatedBy { get; }
}

// پیاده‌سازی در Record Class
public record UserRecord(string FirstName, string LastName, DateTime CreatedAt, string CreatedBy) 
    : IAuditable;
```
*نکته: در اینجا `CreatedAt` و `CreatedBy` هم به عنوان پارامترهای Positional رکورد شناخته می‌شوند و هم اعضای Interface را ارضا می‌کنند.*

---

## Record Struct و Interface

رکوردهای استراکتی (Value Type) برای داده‌های کوچک، سبک و با عمر کوتاه (مثل مختصات، پول، یا تنظیمات) عالی هستند.

```csharp
public interface ICoordinate
{
    double X { get; }
    double Y { get; }
}

// پیاده‌سازی در Record Struct (استفاده از readonly بهترین عملکرد را دارد)
public readonly record struct Point(double X, double Y) : ICoordinate;
```

---

## پیاده‌سازی چند Interface

رکوردها دقیقاً مانند کلاس‌های معمولی می‌توانند چندین Interface را به صورت همزمان پیاده‌سازی کنند:

```csharp
public interface ISoftDeletable
{
    bool IsDeleted { get; }
}

public interface ITenantAware
{
    Guid TenantId { get; }
}

// پیاده‌سازی چند Interface
public record DocumentRecord(
    string Title, 
    Guid TenantId, 
    bool IsDeleted = false) 
    : ITenantAware, ISoftDeletable;
```

---

## Interface به عنوان Contract، Polymorphism و Casting

استفاده از Interface به رکوردها اجازه می‌دهد تا در مفاهیم شی‌گرا مانند چندریختی (Polymorphism) شرکت کنند.

### 1. Interface به عنوان قرارداد (Contract)
Interface تضمین می‌کند که رکورد شما حداقل یک سری داده یا رفتار خاص را دارد، بدون اینکه به پیاده‌سازی داخلی آن کاری داشته باشد.

### 2. Polymorphism (چندریختی)
شما می‌توانید رکوردهای مختلف را در یک کالکشن از نوع Interface قرار دهید:

```csharp
public interface IPaymentEvent
{
    decimal Amount { get; }
}

public record SuccessfulPayment(decimal Amount, string TransactionId) : IPaymentEvent;
public record FailedPayment(decimal Amount, string ErrorMessage) : IPaymentEvent;

// استفاده از Polymorphism
List<IPaymentEvent> events = new()
{
    new SuccessfulPayment(100m, "TX-123"),
    new FailedPayment(50m, "Insufficient funds")
};

foreach (var evt in events)
{
    Console.WriteLine($"Amount: {evt.Amount}");
}
```

### 3. Casting (تبدیل نوع)
شما می‌توانید رکوردها را به Interface آن‌ها Cast کنید.
*   **برای Record Class:** یک تبدیل مرجع (Reference Cast) ساده و بدون هزینه است.
*   **برای Record Struct:** چون Struct یک Value Type است، Cast کردن آن به Interface باعث **Boxing** (انتقال به Heap) می‌شود.

```csharp
IPaymentEvent payment = new SuccessfulPayment(200m, "TX-999");

// Pattern Matching برای Casting و بررسی نوع
if (payment is SuccessfulPayment success)
{
    Console.WriteLine($"Success TX: {success.TransactionId}");
}

// استفاده از Switch Expression
string status = payment switch
{
    SuccessfulPayment s => $"Paid via {s.TransactionId}",
    FailedPayment f => $"Failed: {f.ErrorMessage}",
    _ => "Unknown"
};
```

---

## ارتباط Interface با Equality در Recordها (بسیار مهم)

یکی از سوالات رایج این است: *"آیا پیاده‌سازی Interface روی برابری (Equality) رکورد تأثیر می‌گذارد؟"*

### چه ارتباطی ندارد؟
پیاده‌سازی Interface **هیچ تأثیری** بر نحوه تولید خودکار `Equals`، `GetHashCode` و عملگر `==` توسط کامپایلر ندارد. برابری در رکوردها **صرفاً و صرفاً** بر اساس پارامترهای Positional (داده‌های تعریف شده در constructor رکورد) بررسی می‌شود.

### چه ارتباطی دارد؟ (یک تله رایج!)
اگر Interface شما پراپرتی‌ای را تعریف کند که در پارامترهای Positional رکورد **نباشد**، آن پراپرتی در محاسبه برابری **نادیده گرفته می‌شود**.

```csharp
public interface IEntity
{
    Guid Id { get; }
    string Name { get; }
}

// دقت کنید: Id در پارامترهای رکورد نیست، بلکه یک Property جداگانه است!
public record UserEntity(string Name) : IEntity
{
    public Guid Id { get; init; } = Guid.NewGuid();
}

var user1 = new UserEntity("Ali") { Id = Guid.Parse("11111111-1111-1111-1111-111111111111") };
var user2 = new UserEntity("Ali") { Id = Guid.Parse("22222222-2222-2222-2222-222222222222") };

Console.WriteLine(user1 == user2); // خروجی: True !!!
```
**چرا؟** چون کامپایلر فقط `Name` را در برابری در نظر می‌گیرد و `Id` (که از Interface آمده اما Positional نیست) را نادیده می‌گیرد.
**راه‌حل:** اگر می‌خواهید تمام اعضای Interface در برابری دخیل باشند، باید آن‌ها را به عنوان پارامترهای Positional رکورد تعریف کنید:
`public record UserEntity(Guid Id, string Name) : IEntity;`

---

## استفاده Record در معماری (Architecture)

ترکیب Record و Interface در معماری‌های مدرن مثل **Clean Architecture**، **DDD (طراحی دامنه محور)** و **CQRS** بی‌نظیر عمل می‌کند.

1.  **لایه Domain:** استفاده از Recordها به عنوان **Value Object**ها که Interfaceهای دامنه را پیاده‌سازی می‌کنند.
2.  **لایه Application:** استفاده از Recordها برای **DTO**ها و **Commands/Queries** که Interfaceهای MediatR را پیاده‌سازی می‌کنند.
3.  **لایه Infrastructure:** استفاده از Interfaceها برای تزریق وابستگی‌ها، در حالی که رکوردها داده‌های تغییرناپذیر را بین لایه‌ها جابجا می‌کنند.

---

## سناریوهای واقعی برای استفاده Record + Interface

### سناریوی 1: الگوی CQRS با MediatR
در MediatR، دستورات (Commands) و پرس‌وجوها (Queries) باید یک Interface خاص را پیاده‌سازی کنند. رکوردها بهترین گزینه برای این کار هستند.

```csharp
// Interface از کتابخانه MediatR
// public interface IRequest<out TResponse> { }

public record GetProductByIdQuery(Guid ProductId) : IRequest<ProductDto>;

public record ProductDto(string Name, decimal Price);
```

### سناریوی 2: رویدادهای دامنه (Domain Events)
در Event Sourcing یا ارتباطات Pub/Sub، رویدادها باید تغییرناپذیر باشند.

```csharp
public interface IDomainEvent
{
    DateTime OccurredOn { get; }
}

public record OrderPlacedEvent(
    Guid OrderId, 
    Guid CustomerId, 
    decimal TotalAmount, 
    DateTime OccurredOn) : IDomainEvent;
```

### سناریوی 3: یکپارچه‌سازی پاسخ‌های API (Standardized API Responses)
برای اینکه تمام APIهای شما ساختار یکسانی داشته باشند، از یک Interface استفاده کرده و رکوردهای مختلف را برای انواع پاسخ‌ها برمی‌گردانید.

```csharp
public interface IApiResponse
{
    bool IsSuccess { get; }
    string Message { get; }
}

public record SuccessResponse<T>(T Data, string Message = "Success") : IApiResponse
{
    public bool IsSuccess => true;
}

public record ErrorResponse(string Message, List<string> Errors) : IApiResponse
{
    public bool IsSuccess => false;
}

// در Controller:
public IApiResponse GetUser(int id)
{
    if (id < 0) return new ErrorResponse("Invalid ID", new List<string> { "ID must be positive" });
    return new SuccessResponse<UserDto>(new UserDto("John"));
}
```

### سناریوی 4: Value Objectها در DDD
مفاهیمی مثل پول، آدرس یا مختصات که هویت مستقل ندارند و بر اساس مقدارشان مقایسه می‌شوند.

```csharp
public interface IValueObject; // یک Marker Interface برای شناسایی در لایه زیرساخت

public readonly record struct Money(decimal Amount, string Currency) : IValueObject
{
    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Cannot add different currencies.");
        return new Money(a.Amount + b.Amount, a.Currency);
    }
}
```

---

## جمع‌بندی

استفاده از Interface در Recordها به شما اجازه می‌دهد تا **مزایای داده‌های تغییرناپذیر و برابری مبتنی بر مقدار (Records)** را با **انعطاف‌پذیری، انتزاع و چندریختی (Interfaces)** ترکیب کنید. 
فقط به یاد داشته باشید که برابری در رکوردها فقط به پارامترهای Positional وابسته است و پراپرتی‌های تعریف شده در Interface اگر در constructor رکورد نباشند، در `Equals` نادیده گرفته می‌شوند.

---

## منابع معتبر

1.  **Microsoft Learn - Records (C# Reference)**
    *   [Records - C# Reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
2.  **Microsoft Learn - Interfaces**
    *   [Interfaces - C# Programming Guide | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces)
3.  **C# Language Specification (ECMA-334 / GitHub)**
    *   [Record Classes Specification](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#153-record-declarations)
    *   [Record Structs Specification](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/structs#162-record-struct-declarations)
4.  **مقالات مرتبط با طراحی معماری:**
    *   [Value Objects in DDD using Records](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects)