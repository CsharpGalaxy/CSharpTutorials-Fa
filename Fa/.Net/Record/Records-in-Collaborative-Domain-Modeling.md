# Recordها در Domain-Driven Design

## مقدمه

Domain-Driven Design (DDD) یک رویکرد در طراحی نرم‌افزار است که Eric Evans در سال ۲۰۰۳ با کتاب معروف خود *"Domain-Driven Design: Tackling Complexity in the Heart of Software"* معرفی کرد. در DDD، تمرکز اصلی روی مدل‌سازی دقیق دامنهٔ کسب‌وکار است. یکی از ابزارهای بسیار مفید در زبان C# که از نسخه ۹ معرفی شد و در نسخه‌های بعدی (۱۰، ۱۱، ۱۲ و ۱۳) تکامل یافت، **Record** است. در این مقاله می‌خواهیم بررسی کنیم که چگونه می‌توان از `record` برای پیاده‌سازی مفاهیم DDD به شکلی تمیز، ایمن و خوانا استفاده کرد.

---

## ۱. مفاهیم پایه در DDD

### ۱.۱ Domain Model

Domain Model یک نمایش مفهومی و انتزاعی از دامنهٔ کسب‌وکار است. این مدل شامل موجودیت‌ها (Entities)، اشیاء مقداری (Value Objects)، سرویس‌ها و رویدادهای دامنه می‌شود. هدف Domain Model این است که رفتار دامنه را به شکلی بازتاب دهد که برای کارشناسان کسب‌وکار (Domain Experts) قابل درک باشد.

### ۱.۲ Entity (موجودیت)

Entity شیئی است که **هویت (Identity)** مشخصی دارد. این هویت مستقل از مقادیر داخل آن است. دو Entity حتی اگر تمام خصوصیاتشان یکسان باشد، اگر هویت متفاوتی داشته باشند، دو شیء متفاوت محسوب می‌شوند.

مثال:
- یک `Customer` با شناسه `1234`
- یک `Order` با شماره سفارش `ORD-5678`
- یک `User` با نام کاربری `ali@example.com`

ویژگی‌های Entity:
- دارای هویت (معمولاً یک `Id` از نوع `Guid`، `int` یا یک Value Object خاص)
- دارای چرخهٔ حیات (Create, Update, Delete)
- قابل تغییر (Mutable)
- بر اساس **هویت** مقایسه می‌شود، نه محتوا

### ۱.۳ Value Object (شیء مقداری)

Value Object شیئی است که **مقدار** آن مهم است، نه هویت آن. دو Value Object اگر تمام خصوصیاتشان یکسان باشد، **کاملاً برابر** محسوب می‌شوند.

مثال:
- `Money` (مثلاً ۱۰۰ دلار)
- `Address` (تهران، خیابان ولیعصر، پلاک ۱۰)
- `Email` (user@example.com)
- `Coordinates` (عرض جغرافیایی و طول جغرافیایی)
- `DateRange` (تاریخ شروع و پایان)

ویژگی‌های Value Object:
- **بدون هویت**
- **تغییرناپذیر (Immutable)** — پس از ساخت، وضعیت آن قابل تغییر نیست
- بر اساس **مقدار** مقایسه می‌شود (Value-Based Equality)
- معمولاً برای توصیف ویژگی‌های Entity به کار می‌رود
- باید **Self-Validated** باشد (یعنی در هنگام ساخت، صحت مقادیر بررسی شود)

### ۱.۴ تفاوت Identity و Value

| ویژگی | Entity | Value Object |
|---|---|---|
| هویت | دارد | ندارد |
| مقایسه | بر اساس Id | بر اساس تمام فیلدها |
| تغییرپذیری | Mutable | Immutable |
| چرخه حیات | دارد | ندارد |
| نمونه | Customer, Order | Money, Address |

یک قانون طلایی در DDD می‌گوید:

> «اگر می‌خواهی دو شیء را بر اساس محتوایشان مقایسه کنی، آن شیء یک Value Object است. اگر بر اساس هویت مقایسه می‌کنی، یک Entity است.»

---

## ۲. چرا Record برای Value Object مناسب است؟

از نسخه ۹ زبان C#، نوع `record` به این زبان اضافه شد. ویژگی‌های ذاتی record دقیقاً با ویژگی‌های Value Object در DDD هم‌خوانی دارد:

### ۲.۱ Value-Based Equality (برابری بر اساس مقدار)

وقتی یک `record` تعریف می‌کنید، کامپایلر به‌صورت خودکار متدهای `Equals`، `GetHashCode` و عملگر `==` را بر اساس تمام فیلدهای آن پیاده‌سازی می‌کند. این دقیقاً همان چیزی است که Value Object نیاز دارد.

```csharp
public record Email(string Value)
{
    public Email : this(Value)
    {
        if (string.IsNullOrWhiteSpace(Value))
            throw new ArgumentException("Email cannot be empty.");
        if (!Value.Contains('@'))
            throw new ArgumentException("Invalid email format.");
    }
}

var email1 = new Email("ali@example.com");
var email2 = new Email("ali@example.com");

Console.WriteLine(email1 == email2); // True
Console.WriteLine(email1.Equals(email2)); // True
```

### ۲.۲ Immutability (تغییرناپذیری)

در `record`، پراپرتی‌ها به‌صورت پیش‌فرض `init-only` هستند. این یعنی پس از ساخت شیء، نمی‌توانید مقادیر را تغییر دهید. این ویژگی برای Value Object حیاتی است، چون:

- از تغییرات غیرمنتظره جلوگیری می‌کند
- شیء را Thread-Safe می‌کند
- استفاده از آن را به‌عنوان کلید در `Dictionary` یا عضو در `HashSet` امن می‌سازد

### ۲.۳ With Expression (کپی با تغییر)

یکی از قابلیت‌های زیبای `record`، عبارت `with` است که به شما اجازه می‌دهد یک کپی از شیء بسازید و فقط فیلدهای موردنظر را تغییر دهید:

```csharp
public record Address(string City, string Street, string PostalCode);

var address1 = new Address("Tehran", "Valiasr", "12345");
var address2 = address1 with { PostalCode = "54321" };

// address1 تغییر نکرده است
// address2 یک شیء جدید با PostalCode متفاوت است
```

این رفتار دقیقاً با اصل Immutability در DDD هم‌خوان است.

### ۲.۴ Positional Syntax و Compactness

Syntax موقعیتی (Positional) در record باعث می‌شود تعریف Value Objectها بسیار کوتاه و خوانا باشد:

```csharp
public record Money(decimal Amount, string Currency);
public record Coordinates(double Latitude, double Longitude);
public record DateRange(DateOnly Start, DateOnly End);
```

---

## ۳. بررسی مثال‌ها

### ۳.۱ Money

`Money` یک نمونهٔ کلاسیک از Value Object است. در بسیاری از دامنه‌ها (مالی، فروش، بانکداری)، مقدار پول و واحد آن با هم معنا پیدا می‌کنند.

```csharp
public record Money(decimal Amount, string Currency)
{
    public static Money operator +(Money left, Money right)
    {
        if (left.Currency != right.Currency)
            throw new InvalidOperationException(
                $"Cannot add {left.Currency} to {right.Currency}");
        return left with { Amount = left.Amount + right.Amount };
    }

    public static Money operator -(Money money) => money with { Amount = -money.Amount };
}
```

**چرا Record مناسب است؟**
- دو `Money` با مقدار و ارز یکسان، باید برابر باشند (Value-Based Equality)
- پول تغییرناپذیر است — شما ۱۰۰ دلار را «تغییر» نمی‌دهید، بلکه یک `Money` جدید می‌سازید
- عملگرها به‌راحتی پیاده می‌شوند

### ۳.۲ Address

```csharp
public record Address(
    string Country,
    string City,
    string Street,
    string PostalCode,
    string? Apartment = null);
```

**چرا Record مناسب است؟**
- آدرس یک توصیف است، نه یک موجودیت با هویت
- دو آدرس یکسان باید برابر باشند
- تغییرناپذیری باعث می‌شود بتوان آن را به‌راحتی در Entityهای مختلف به اشتراک گذاشت

### ۳.۳ Email

```csharp
public record Email(string Value)
{
    public Email : this(Value)
    {
        if (string.IsNullOrWhiteSpace(Value))
            throw new ArgumentException("Email cannot be empty.");
        if (!Value.Contains('@'))
            throw new ArgumentException("Invalid email format.");
    }
}
```

**چرا Record مناسب است؟**
- Email یک مقدار است، نه یک هویت (در بیشتر دامنه‌ها)
- تغییرناپذیری مهم است
- Self-Validation در constructor به‌راحتی انجام می‌شود

### ۳.۴ Coordinates

```csharp
public record Coordinates(double Latitude, double Longitude)
{
    public double DistanceTo(Coordinates other)
    {
        // محاسبه فاصله با فرمول Haversine
        const double R = 6371; // شعاع زمین به کیلومتر
        var dLat = ToRad(other.Latitude - Latitude);
        var dLon = ToRad(other.Longitude - Longitude);
        var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                Math.Cos(ToRad(Latitude)) * Math.Cos(ToRad(other.Latitude)) *
                Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return R * c;
    }

    private static double ToRad(double deg) => deg * Math.PI / 180;
}
```

**چرا Record مناسب است؟**
- مختصات جغرافیایی یک مقدار است
- رفتار دامنه‌ای (مثل محاسبه فاصله) می‌تواند به‌عنوان متد در record قرار گیرد
- تغییرناپذیری مهم است

### ۳.۵ DateRange

```csharp
public record DateRange(DateOnly Start, DateOnly End)
{
    public DateRange : this(Start, End)
    {
        if (End < Start)
            throw new ArgumentException("End date cannot be before start date.");
    }

    public int Days => End.DayNumber - Start.DayNumber + 1;

    public bool Overlaps(DateRange other) =>
        Start <= other.End && other.Start <= End;

    public bool Contains(DateOnly date) =>
        date >= Start && date <= End;
}
```

**چرا Record مناسب است؟**
- یک بازهٔ زمانی یک مقدار است
- Self-Validation در constructor
- رفتار دامنه‌ای (Overlap, Contains) در خود Value Object قرار می‌گیرد که اصل **Rich Domain Model** را تقویت می‌کند

---

## ۴. چرا Entity را نباید به Record تبدیل کرد؟

این یکی از مهم‌ترین نکاتی است که تازه‌کاران DDD باید بدانند. با اینکه `record` ویژگی‌های جذابی دارد، اما برای Entity مناسب نیست. دلایل:

### ۴.۱ Entityها Mutable هستند

Entityها در طول چرخهٔ حیات خود تغییر می‌کنند. یک `Order` ممکن است وضعیتش از `Pending` به `Shipped` تغییر کند. `record`ها به‌صورت پیش‌فرض تغییرناپذیرند و این با ماهیت Entity در تضاد است.

### ۴.۲ Entityها بر اساس هویت مقایسه می‌شوند

وقتی دو `Order` با `Id` یکسان از دیتابیس بارگذاری می‌شوند، حتی اگر داده‌هایشان کمی متفاوت باشد (مثلاً به دلیل caching)، باید برابر در نظر گرفته شوند. `record` بر اساس **مقدار** مقایسه می‌کند، نه هویت.

### ۴.۳ پیاده‌سازی نادرست Equality

برخی برنامه‌نویسان فکر می‌کنند می‌توانند با override کردن `Equals` در یک `record`، آن را بر اساس `Id` مقایسه کنند:

```csharp
// ❌ این کار اشتباه است
public record Customer(Guid Id, string Name)
{
    public virtual bool Equals(Customer? other)
    {
        if (other is null) return false;
        return Id == other.Id;
    }
    // GetHashCode هم باید override شود
}
```

این کار چند مشکل دارد:
- با `with` expression، اگر `Id` را تغییر دهید، `Equals` همچنان `true` برمی‌گرداند که گیج‌کننده است
- رفتار `record` را می‌شکنید
- کد را برای دیگران غیرمنتظره می‌کنید
- `GetHashCode` باید بر اساس همان فیلدهایی باشد که در `Equals` استفاده شده، وگرنه در `Dictionary` و `HashSet` مشکل ایجاد می‌شود

### ۴.۴ راه‌حل صحیح برای Entity

Entityها باید به‌صورت `class` پیاده‌سازی شوند:

```csharp
public class Customer
{
    public Guid Id { get; private set; }
    public string Name { get; private set; }
    public Email Email { get; private set; } // Value Object به‌عنوان record
    public Address Address { get; private set; } // Value Object به‌عنوان record

    private Customer() { } // برای EF Core

    public Customer(Guid id, string name, Email email, Address address)
    {
        Id = id;
        ChangeName(name);
        Email = email;
        Address = address;
    }

    public void ChangeName(string newName)
    {
        if (string.IsNullOrWhiteSpace(newName))
            throw new ArgumentException("Name cannot be empty.");
        Name = newName;
    }
}
```

در این حالت:
- Entity به‌صورت `class` تعریف می‌شود
- `Equals` و `GetHashCode` بر اساس `Id` پیاده می‌شوند
- Value Objectهای داخل آن (`Email`, `Address`) به‌صورت `record` هستند

> **قانون طلایی:** «از `record` برای Value Objectها استفاده کن، از `class` برای Entityها.»

---

## ۵. Domain Event

Domain Event رویدادی است که نشان می‌دهد چیزی مهم در دامنه اتفاق افتاده است. مثال: `OrderPlaced`, `CustomerRegistered`, `PaymentReceived`.

### آیا Domain Event را باید به‌صورت record پیاده کرد؟

**بله، معمولاً record بهترین انتخاب است.**

دلایل:
- Domain Eventها تغییرناپذیرند — یک رویداد در گذشته اتفاق افتاده و قابل تغییر نیست
- بر اساس مقدار مقایسه می‌شوند (برای تست و serialization)
- حمل داده (Payload) هستند، نه رفتار

```csharp
public record OrderPlacedEvent(
    Guid OrderId,
    Guid CustomerId,
    Money TotalAmount,
    DateTime OccurredOn);

public record CustomerRegisteredEvent(
    Guid CustomerId,
    Email Email,
    DateTime OccurredOn);
```

### استثنا: Domain Event با رفتار

اگر Domain Event نیاز به رفتار پیچیده داشته باشد (که نادر است)، می‌تواند `class` باشد. اما در ۹۹٪ موارد، `record` انتخاب بهتری است.

---

## ۶. Aggregate و Aggregate Root

### ۶.۱ Aggregate

Aggregate مجموعه‌ای از Entityها و Value Objectهاست که از نظر منطقی به هم مرتبط هستند و یک واحد از تغییرات را تشکیل می‌دهند.

### ۶.۲ Aggregate Root

Aggregate Root یک Entity خاص است که ورودی اصلی به Aggregate محسوب می‌شود. تمام تغییرات در Aggregate از طریق Aggregate Root انجام می‌شود.

مثال: `Order` یک Aggregate Root است که شامل `OrderLine`ها (Entity) و `Address` (Value Object) می‌شود.

```csharp
public class Order : AggregateRoot // class، نه record
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public Address ShippingAddress { get; private set; } // Value Object (record)
    private readonly List<OrderLine> _lines = new();
    public IReadOnlyCollection<OrderLine> Lines => _lines.AsReadOnly();
    public Money TotalAmount => _lines.Aggregate(Money.Zero, (sum, line) => sum + line.Subtotal);

    private readonly List<DomainEvent> _domainEvents = new();
    public IReadOnlyCollection<DomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    private Order() { } // برای EF Core

    public Order(Guid id, Guid customerId, Address shippingAddress)
    {
        Id = id;
        CustomerId = customerId;
        ShippingAddress = shippingAddress;
        _domainEvents.Add(new OrderPlacedEvent(id, customerId, Money.Zero, DateTime.UtcNow));
    }

    public void AddLine(Product product, int quantity, Money unitPrice)
    {
        var line = new OrderLine(Guid.NewGuid(), product.Id, quantity, unitPrice);
        _lines.Add(line);
    }
}

public class OrderLine : Entity // class، نه record
{
    public Guid Id { get; private set; }
    public Guid ProductId { get; private set; }
    public int Quantity { get; private set; }
    public Money UnitPrice { get; private set; } // Value Object (record)
    public Money Subtotal => UnitPrice * Quantity;
}
```

### چرا Aggregate Root باید class باشد؟

- Aggregate Root یک Entity است (هویت دارد، Mutable است)
- چرخهٔ حیات دارد
- Domain Eventها را جمع‌آوری می‌کند
- رفتار دامنه‌ای پیچیده دارد

### چه چیزهایی در Aggregate می‌توانند record باشند؟

- Value Objectها (مثل `Address`, `Money`)
- Domain Eventها
- برخی Snapshotها یا DTOهای داخلی

---

## ۷. DTO (Data Transfer Object)

DTO شیئی است که برای انتقال داده بین لایه‌ها (مثلاً بین Application Layer و UI یا بین سرویس‌ها) استفاده می‌شود.

### آیا DTO را باید به‌صورت record پیاده کرد؟

**بله، معمولاً record انتخاب عالی است.**

دلایل:
- DTOها معمولاً تغییرناپذیرند
- بر اساس مقدار مقایسه می‌شوند (مفید برای تست)
- Syntax کوتاه و خوانا
- سازگاری عالی با System.Text.Json برای serialization

```csharp
// Request DTO
public record CreateOrderRequest(
    Guid CustomerId,
    AddressDto ShippingAddress,
    List<OrderLineDto> Lines);

public record AddressDto(
    string Country,
    string City,
    string Street,
    string PostalCode);

public record OrderLineDto(Guid ProductId, int Quantity);

// Response DTO
public record OrderResponse(
    Guid Id,
    Guid CustomerId,
    AddressDto ShippingAddress,
    MoneyDto TotalAmount,
    string Status,
    DateTime CreatedAt);

public record MoneyDto(decimal Amount, string Currency);
```

### تفاوت DTO با Domain Model

یک نکتهٔ مهم: **DTO نباید با Domain Model اشتباه گرفته شود.**

- DTOها در لایهٔ Application یا Presentation قرار دارند
- Domain Model (شامل Entity و Value Object) در لایهٔ Domain قرار دارد
- DTOها معمولاً ساختار تخت (Flat) دارند، در حالی که Domain Model رفتار دامنه‌ای دارد

```csharp
// ❌ اشتباه: استفاده از DTO در Domain Layer
public class Order
{
    public OrderDto Data { get; set; } // این کار اشتباه است
}

// ✅ صحیح: تبدیل DTO به Domain Model در Application Layer
public class OrderService
{
    public Order CreateOrder(CreateOrderRequest request)
    {
        var address = new Address(
            request.ShippingAddress.Country,
            request.ShippingAddress.City,
            request.ShippingAddress.Street,
            request.ShippingAddress.PostalCode);

        var order = new Order(Guid.NewGuid(), request.CustomerId, address);
        foreach (var line in request.Lines)
        {
            var product = _productRepository.GetById(line.ProductId);
            order.AddLine(product, line.Quantity, product.Price);
        }
        return order;
    }
}
```

---

## ۸. Domain Model و ساختار کلی

در یک Domain Model خوب، ساختار به این شکل است:

```
Domain Layer
├── Entities (class)
│   ├── Customer : Entity, AggregateRoot
│   ├── Order : Entity, AggregateRoot
│   └── OrderLine : Entity
├── Value Objects (record)
│   ├── Money
│   ├── Address
│   ├── Email
│   ├── Coordinates
│   └── DateRange
├── Domain Events (record)
│   ├── OrderPlacedEvent
│   ├── CustomerRegisteredEvent
│   └── PaymentReceivedEvent
└── Interfaces
    ├── IRepository<T>
    └── IDomainEvent
```

### اصول طراحی

1. **Value Objectها را `record` تعریف کن** — به دلیل Immutability و Value-Based Equality
2. **Entityها را `class` تعریف کن** — به دلیل Identity و Mutability
3. **Domain Eventها را `record` تعریف کن** — به دلیل Immutability
4. **DTOها را `record` تعریف کن** — به دلیل سادگی و Immutability
5. **Aggregate Rootها را `class` تعریف کن** — چون Entity هستند
6. **رفتار دامنه‌ای را در Entity و Value Object قرار بده** — Rich Domain Model
7. **از Anemic Domain Model پرهیز کن** — جایی که Entityها فقط Getter/Setter دارند

---

## ۹. نکات پیشرفته

### ۹.۱ Record Struct

از C# ۱۰ می‌توان `record struct` تعریف کرد:

```csharp
public record struct Coordinates(double Latitude, double Longitude);
```

این نوع Value Type است و برای Value Objectهای کوچک که نیاز به allocation روی Stack دارند، مفید است. اما توجه داشته باشید:
- `record struct` به‌صورت پیش‌فرض Mutable است (مگر اینکه از `readonly record struct` استفاده کنید)
- برای Domain Model، معمولاً `record class` (همان `record` معمولی) انتخاب بهتری است

```csharp
// ✅ بهتر برای DDD
public readonly record struct Coordinates(double Latitude, double Longitude);
```

### ۹.۲ Non-Destructive Mutation

عبارت `with` در record یک قابلیت قدرتمند است:

```csharp
var address = new Address("Tehran", "Valiasr", "12345");
var newAddress = address with { PostalCode = "54321" };
```

این عبارت:
- یک شیء جدید می‌سازد
- تمام فیلدها را از شیء قبلی کپی می‌کند
- فقط فیلدهای مشخص‌شده را تغییر می‌دهد
- شیء قبلی را دست‌نخورده باقی می‌گذارد

### ۹.۳ Equality در Recordهای تو در تو

وقتی یک record، record دیگری را به‌عنوان فیلد دارد، equality به‌صورت عمیق (Deep Equality) مقایسه می‌شود:

```csharp
public record Address(string City, string Street);
public record Customer(string Name, Address Address);

var c1 = new Customer("Ali", new Address("Tehran", "Valiasr"));
var c2 = new Customer("Ali", new Address("Tehran", "Valiasr"));

Console.WriteLine(c1 == c2); // True - Deep Equality
```

این ویژگی برای Value Objectهای ترکیبی بسیار مفید است.

### ۹.۴ چاپ خوانا

Recordها به‌صورت خودکار `ToString` را override می‌کنند:

```csharp
var money = new Money(100, "USD");
Console.WriteLine(money); 
// Money { Amount = 100, Currency = USD }
```

این ویژگی برای Debugging و Logging بسیار مفید است.

---

## ۱۰. جمع‌بندی

| مفهوم DDD | نوع پیشنهادی در C# | دلیل |
|---|---|---|
| Value Object | `record` | Immutability, Value-Based Equality |
| Entity | `class` | Identity, Mutability, Lifecycle |
| Aggregate Root | `class` | چون Entity است |
| Domain Event | `record` | Immutability, Payload carrier |
| DTO | `record` | سادگی, Immutability, Serialization |
| Repository | `interface` + `class` | Separation of Concerns |

### قوانین طلایی

1. **هر شیئی که هویت دارد → `class`**
2. **هر شیئی که فقط مقدار دارد → `record`**
3. **هرگز Entity را به `record` تبدیل نکن، حتی اگر بخواهی Equality را بر اساس Id پیاده کنی**
4. **Value Objectها باید Self-Validated باشند**
5. **Domain Eventها و DTOها معمولاً `record` هستند**

با رعایت این اصول، کد شما:
- خواناتر می‌شود
- ایمن‌تر می‌شود (به‌خصوص در محیط‌های Concurrent)
- با اصول DDD هم‌خوان می‌شود
- نگهداری آن آسان‌تر می‌شود

---

## منابع

### منابع Microsoft

1. **Microsoft Docs - Records (C# Reference)**
   https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

2. **Microsoft Docs - Records and Immutability**
   https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records#immutability

3. **Microsoft Architecture Guides - Domain-Driven Design**
   https://learn.microsoft.com/en-us/dotnet/architecture/domain-driven-design/

4. **eShopOnContainers - Reference Architecture (Microsoft)**
   https://github.com/dotnet-architecture/eShopOnContainers

5. **Microsoft Docs - Value Objects in DDD**
   https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects

6. **Microsoft Docs - Entity vs Value Object**
   https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/design-domain-model

### منابع معتبر DDD

7. **Eric Evans - Domain-Driven Design: Tackling Complexity in the Heart of Software (2003)**
   کتاب اصلی که DDD را معرفی کرد. فصل ۶ آن به‌طور خاص به Value Objectها می‌پردازد.

8. **Vaughn Vernon - Implementing Domain-Driven Design (2013)**
   کتابی که پیاده‌سازی عملی DDD را توضیح می‌دهد. فصل‌های مربوط به Value Object و Entity بسیار مفیدند.

9. **Vaughn Vernon - Domain-Driven Design Distilled (2016)**
   نسخهٔ خلاصه و کاربردی DDD برای شروع سریع.

10. **Martin Fowler - Domain Model**
    https://martinfowler.com/eaaCatalog/domainModel.html

11. **Martin Fowler - Anemic Domain Model**
    https://martinfowler.com/bliki/AnemicDomainModel.html

12. **Martin Fowler - Value Object**
    https://martinfowler.com/bliki/ValueObject.html

13. **Jimmy Bogard - DDD in .NET**
    https://jimmybogard.com/primer-on-domain-driven-design/

14. **CQRS and DDD Patterns (Microsoft)**
    https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs

15. **Julie Lerman & Steve Smith - Data Points - DDD and EF Core**
    https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/brownfield/domain-driven-design

---

### یادداشت پایانی

استفاده از `record` در DDD یک ابزار قدرتمند است، اما نباید به‌صورت کورکورانه برای همهٔ مفاهیم به کار رود. درک تفاوت بین Entity و Value Object، کلید استفادهٔ صحیح از `record` است. با تمرین و مطالعهٔ بیشتر، می‌توانید Domain Modelهایی بسازید که هم زیبا، هم ایمن و هم نزدیک به زبان کسب‌وکار باشند.

موفق باشید! 🚀