# Recordها در Domain-Driven Design

## مقدمه

Domain-Driven Design (DDD) یک رویکرد برای طراحی نرم‌افزار است که بر مدل‌سازی دامنه کسب‌وکار تمرکز دارد. در این مقاله، به بررسی کاربرد `record` در C# برای پیاده‌سازی مفاهیم DDD می‌پردازیم.

## مفاهیم پایه DDD

### Entity چیست؟

**Entity** یک شیء است که هویت (Identity) مشخصی دارد. هویت Entity مستقل از ویژگی‌های آن است و حتی اگر تمام ویژگی‌های آن تغییر کند، همچنان همان Entity باقی می‌ماند.

**ویژگی‌های Entity:**
- دارای هویت (Id) منحصر به فرد
- هویت مستقل از مقادیر ویژگی‌ها
- معمولاً Mutable (قابل تغییر) است
- مقایسه بر اساس Identity

```csharp
public class Customer
{
    public Guid Id { get; private set; }
    public string Name { get; private set; }
    public string Email { get; private set; }
    
    public Customer(Guid id, string name, string email)
    {
        Id = id;
        Name = name;
        Email = email;
    }
    
    public void ChangeEmail(string newEmail)
    {
        Email = newEmail;
    }
}
```

### Value Object چیست؟

**Value Object** یک شیء است که هویت ندارد و فقط بر اساس مقادیر ویژگی‌هایش تعریف می‌شود. دو Value Object با مقادیر یکسان، برابر در نظر گرفته می‌شوند.

**ویژگی‌های Value Object:**
- بدون هویت (Identity)
- مقایسه بر اساس Value (مقدار)
- Immutable (غیرقابل تغییر)
- معمولاً برای توصیف ویژگی‌های Entity استفاده می‌شود

### تفاوت Identity و Value

| ویژگی | Entity | Value Object |
|-------|--------|--------------|
| **هویت** | دارد (Id) | ندارد |
| **مقایسه** | بر اساس Identity | بر اساس Value |
| **تغییرپذیری** | معمولاً Mutable | Immutable |
| **چرخه حیات** | مستقل | وابسته به Entity |
| **مثال** | Customer, Order | Money, Address |

## چرا Record برای Value Object مناسب است؟

### Value-Based Equality

`record` در C# به صورت خودکار **Value-Based Equality** را پیاده‌سازی می‌کند. یعنی دو record با مقادیر یکسان، برابر در نظر گرفته می‌شوند:

```csharp
public record Money(decimal Amount, string Currency);

var money1 = new Money(100, "USD");
var money2 = new Money(100, "USD");

Console.WriteLine(money1 == money2); // True
Console.WriteLine(money1.Equals(money2)); // True
```

این رفتار دقیقاً همان چیزی است که از Value Object انتظار داریم.

### Immutability

`record` به صورت پیش‌فرض **Immutable** است. پس از ساخت یک record، نمی‌توان مقادیر آن را تغییر داد (مگر با `with` expression):

```csharp
public record Address(string Street, string City, string ZipCode);

var address = new Address("123 Main St", "New York", "10001");
// address.Street = "456 Oak Ave"; // Error: Cannot modify

var newAddress = address with { Street = "456 Oak Ave" };
```

این immutability برای Value Object‌ها بسیار مهم است زیرا:
- از تغییرات ناخواسته جلوگیری می‌کند
- Thread-safe است
- استفاده از آن‌ها در Collection‌ها (مثل HashSet, Dictionary) امن است

### Syntax Concise

`record` syntax مختصری دارد و نیاز به نوشتن کد boilerplate کمتری دارد:

```csharp
// بدون record (کد زیاد)
public class Email
{
    public string Value { get; }
    
    public Email(string value)
    {
        Value = value;
    }
    
    public override bool Equals(object obj)
    {
        return obj is Email other && Value == other.Value;
    }
    
    public override int GetHashCode()
    {
        return Value.GetHashCode();
    }
}

// با record (کد کم)
public record Email(string Value);
```

## مثال‌های عملی

### 1. Money

```csharp
public record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
        
        return new Money(Amount + other.Amount, Currency);
    }
}

// استفاده
var price = new Money(100, "USD");
var tax = new Money(10, "USD");
var total = price.Add(tax); // Money(110, "USD")
```

**چرا Record مناسب است؟**
- Money یک Value Object کلاسیک است
- مقایسه بر اساس مقدار (Amount و Currency) مهم است
- Immutable بودن از تغییرات ناخواسته جلوگیری می‌کند
- عملیات‌های ریاضی می‌توانند Money جدید برگردانند

### 2. Address

```csharp
public record Address(
    string Street,
    string City,
    string State,
    string ZipCode,
    string Country);

// استفاده
var homeAddress = new Address("123 Main St", "New York", "NY", "10001", "USA");
var workAddress = homeAddress with { Street = "456 Office Blvd" };
```

**چرا Record مناسب است؟**
- Address یک Value Object است که مکان فیزیکی را توصیف می‌کند
- دو Address با مقادیر یکسان، یکسان هستند
- معمولاً Immutable است (آدرس تغییر نمی‌کند، بلکه آدرس جدید ایجاد می‌شود)
- `with` expression برای ایجاد تغییرات جزئی عالی است

### 3. Email

```csharp
public record Email(string Value)
{
    public Email : this(Value.Trim().ToLower())
    {
        if (!IsValidEmail(Value))
            throw new ArgumentException("Invalid email format");
    }
    
    private static bool IsValidEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

// استفاده
var email = new Email("User@Example.com");
Console.WriteLine(email.Value); // "user@example.com"
```

**چرا Record مناسب است؟**
- Email یک Value Object ساده است
- مقایسه بر اساس مقدار مهم است
- Immutable بودن از تغییر ناخواسته جلوگیری می‌کند
- Validation در constructor انجام می‌شود

### 4. Coordinates

```csharp
public record Coordinates(double Latitude, double Longitude)
{
    public double DistanceTo(Coordinates other)
    {
        // محاسبه فاصله بین دو نقطه
        const double R = 6371; // شعاع زمین به کیلومتر
        var dLat = ToRadians(other.Latitude - Latitude);
        var dLon = ToRadians(other.Longitude - Longitude);
        
        var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                Math.Cos(ToRadians(Latitude)) * Math.Cos(ToRadians(other.Latitude)) *
                Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        
        var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return R * c;
    }
    
    private static double ToRadians(double degrees) => degrees * Math.PI / 180;
}

// استفاده
var tehran = new Coordinates(35.6892, 51.3890);
var isfahan = new Coordinates(32.6546, 51.6680);
var distance = tehran.DistanceTo(isfahan); // ~340 km
```

**چرا Record مناسب است؟**
- Coordinates یک Value Object است که موقعیت جغرافیایی را توصیف می‌کند
- دو Coordinates با مقادیر یکسان، یکسان هستند
- Immutable بودن مهم است (مختصات تغییر نمی‌کند)
- عملیات‌های محاسباتی می‌توانند Coordinates جدید برگردانند

### 5. DateRange

```csharp
public record DateRange(DateOnly Start, DateOnly End)
{
    public DateRange
    {
        if (End < Start)
            throw new ArgumentException("End date must be after start date");
    }
    
    public int Days => End.DayNumber - Start.DayNumber + 1;
    
    public bool Overlaps(DateRange other)
    {
        return Start <= other.End && End >= other.Start;
    }
    
    public bool Contains(DateOnly date)
    {
        return date >= Start && date <= End;
    }
}

// استفاده
var vacation = new DateRange(
    new DateOnly(2026, 8, 1),
    new DateOnly(2026, 8, 15)
);

Console.WriteLine(vacation.Days); // 15
Console.WriteLine(vacation.Contains(new DateOnly(2026, 8, 10))); // True
```

**چرا Record مناسب است؟**
- DateRange یک Value Object است که یک بازه زمانی را توصیف می‌کند
- مقایسه بر اساس مقدار مهم است
- Immutable بودن مهم است (بازه زمانی تغییر نمی‌کند)
- Validation در constructor انجام می‌شود

## چرا Entity را نباید به Record تبدیل کرد؟

### مشکل 1: Identity vs Value Equality

Entity‌ها بر اساس **Identity** مقایسه می‌شوند، نه Value:

```csharp
// ❌ اشتباه
public record Customer(Guid Id, string Name, string Email);

var customer1 = new Customer(Guid.NewGuid(), "John", "john@example.com");
var customer2 = new Customer(customer1.Id, "John", "john@example.com");

Console.WriteLine(customer1 == customer2); // False! (چون Id متفاوت است)
```

در DDD، دو Entity با Id یکسان باید برابر باشند، حتی اگر Name یا Email متفاوت باشد. اما `record` بر اساس تمام ویژگی‌ها مقایسه می‌کند.

### مشکل 2: Immutability نامناسب

Entity‌ها معمولاً **Mutable** هستند و تغییر می‌کنند:

```csharp
// ❌ مشکل
public record Order(Guid Id, DateTime OrderDate, string Status);

var order = new Order(Guid.NewGuid(), DateTime.Now, "Pending");
// order = order with { Status = "Shipped" }; // نیاز به ایجاد instance جدید
```

با `record`، برای تغییر Status باید یک instance جدید ایجاد کنید که:
- پیچیدگی را افزایش می‌دهد
- با مفهوم Change Tracking در ORM‌ها (مثل Entity Framework) سازگار نیست
- باعث مشکل در Persistence می‌شود

### مشکل 3: Navigation Properties

Entity‌ها معمولاً **Navigation Properties** به Entity‌های دیگر دارند:

```csharp
// ❌ مشکل
public record Order(Guid Id, Customer Customer, List<OrderItem> Items);
```

این باعث:
- Circular Reference در Equality comparison
- مشکل در Serialization
- مشکل در Lazy Loading

### مثال صحیح: Entity با Class

```csharp
public class Customer
{
    public Guid Id { get; private set; }
    public string Name { get; private set; }
    public Email Email { get; private set; } // Value Object
    public Address Address { get; private set; } // Value Object
    
    public Customer(Guid id, string name, Email email, Address address)
    {
        Id = id;
        Name = name;
        Email = email;
        Address = address;
    }
    
    public void ChangeName(string newName)
    {
        Name = newName;
    }
    
    public void ChangeEmail(Email newEmail)
    {
        Email = newEmail;
    }
    
    // Equality بر اساس Identity
    public override bool Equals(object obj)
    {
        return obj is Customer other && Id == other.Id;
    }
    
    public override int GetHashCode()
    {
        return Id.GetHashCode();
    }
}
```

## Domain Event و Record

`record` برای **Domain Event** نیز مناسب است:

```csharp
public record OrderPlacedEvent(
    Guid OrderId,
    Guid CustomerId,
    DateTime OccurredOn,
    decimal TotalAmount);

public record OrderShippedEvent(
    Guid OrderId,
    DateTime ShippedDate,
    string TrackingNumber);
```

**چرا مناسب است؟**
- Domain Event‌ها Immutable هستند (اتفاقی که افتاده تغییر نمی‌کند)
- مقایسه بر اساس Value مفید است
- Syntax مختصر
- مناسب برای Event Sourcing

## Aggregate و Aggregate Root

### Aggregate

**Aggregate** یک گروه از Entity‌ها و Value Object‌ها است که به عنوان یک واحد تغییر می‌کنند.

### Aggregate Root

**Aggregate Root** یک Entity خاص است که ورودی به Aggregate را کنترل می‌کند.

```csharp
// Aggregate Root (Entity)
public class Order
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public Money TotalAmount { get; private set; } // Value Object
    public Address ShippingAddress { get; private set; } // Value Object
    private List<OrderItem> _items;
    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();
    
    public Order(Guid id, Guid customerId, Address shippingAddress)
    {
        Id = id;
        CustomerId = customerId;
        ShippingAddress = shippingAddress;
        TotalAmount = new Money(0, "USD");
        _items = new List<OrderItem>();
    }
    
    public void AddItem(Product product, int quantity)
    {
        var item = new OrderItem(product.Id, product.Price, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        TotalAmount = _items.Aggregate(
            new Money(0, "USD"),
            (total, item) => total.Add(item.Subtotal)
        );
    }
}

// Entity داخل Aggregate
public class OrderItem
{
    public Guid ProductId { get; private set; }
    public Money UnitPrice { get; private set; }
    public int Quantity { get; private set; }
    public Money Subtotal => UnitPrice with { Amount = UnitPrice.Amount * Quantity };
}

// Value Objects
public record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException();
        return new Money(Amount + other.Amount, Currency);
    }
}

public record Address(string Street, string City, string ZipCode, string Country);
```

## DTO و Record

`record` برای **DTO (Data Transfer Object)** نیز عالی است:

```csharp
public record CustomerDto(
    Guid Id,
    string Name,
    string Email,
    AddressDto Address);

public record AddressDto(
    string Street,
    string City,
    string ZipCode,
    string Country);

// در API Controller
[HttpGet("{id}")]
public async Task<ActionResult<CustomerDto>> GetCustomer(Guid id)
{
    var customer = await _repository.GetById(id);
    
    var dto = new CustomerDto(
        customer.Id,
        customer.Name,
        customer.Email.Value,
        new AddressDto(
            customer.Address.Street,
            customer.Address.City,
            customer.Address.ZipCode,
            customer.Address.Country
        )
    );
    
    return Ok(dto);
}
```

**چرا مناسب است؟**
- DTO‌ها Immutable هستند
- مقایسه بر اساس Value مفید است
- Syntax مختصر
- مناسب برای Serialization/Deserialization

## Domain Model

**Domain Model** قلب DDD است و شامل:
- **Entities**: با هویت و چرخه حیات
- **Value Objects**: بدون هویت، Immutable
- **Aggregates**: گروه‌بندی Entity‌ها
- **Domain Events**: اتفاقات مهم در دامنه
- **Domain Services**: منطق که متعلق به هیچ Entity نیست

### مثال کامل Domain Model

```csharp
// Value Objects
public record Email(string Value)
{
    public Email : this(Value.Trim().ToLower())
    {
        if (!IsValid(Value))
            throw new ArgumentException("Invalid email");
    }
    
    private static bool IsValid(string email) =>
        email.Contains("@") && email.Contains(".");
}

public record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Currency mismatch");
        return new Money(Amount + other.Amount, Currency);
    }
    
    public Money Subtract(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Currency mismatch");
        return new Money(Amount - other.Amount, Currency);
    }
}

public record Address(
    string Street,
    string City,
    string State,
    string ZipCode,
    string Country);

// Domain Event
public record CustomerRegisteredEvent(
    Guid CustomerId,
    Email Email,
    DateTime OccurredOn);

// Entity (Aggregate Root)
public class Customer
{
    private readonly List<Order> _orders = new();
    
    public Guid Id { get; private set; }
    public string Name { get; private set; }
    public Email Email { get; private set; }
    public Address Address { get; private set; }
    public IReadOnlyList<Order> Orders => _orders.AsReadOnly();
    
    private readonly List<DomainEvent> _domainEvents = new();
    public IReadOnlyList<DomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    public Customer(Guid id, string name, Email email, Address address)
    {
        Id = id;
        Name = name;
        Email = email;
        Address = address;
        
        _domainEvents.Add(new CustomerRegisteredEvent(id, email, DateTime.UtcNow));
    }
    
    public void ChangeName(string newName)
    {
        Name = newName;
    }
    
    public void ChangeEmail(Email newEmail)
    {
        Email = newEmail;
    }
    
    public void ChangeAddress(Address newAddress)
    {
        Address = newAddress;
    }
    
    public Order PlaceOrder(Address shippingAddress)
    {
        var order = new Order(Guid.NewGuid(), Id, shippingAddress);
        _orders.Add(order);
        return order;
    }
    
    // Equality بر اساس Identity
    public override bool Equals(object obj)
    {
        return obj is Customer other && Id == other.Id;
    }
    
    public override int GetHashCode()
    {
        return Id.GetHashCode();
    }
}

// Entity داخل Aggregate
public class Order
{
    private readonly List<OrderItem> _items = new();
    
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public Address ShippingAddress { get; private set; }
    public Money TotalAmount { get; private set; }
    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();
    
    public Order(Guid id, Guid customerId, Address shippingAddress)
    {
        Id = id;
        CustomerId = customerId;
        ShippingAddress = shippingAddress;
        TotalAmount = new Money(0, "USD");
    }
    
    public void AddItem(Guid productId, Money unitPrice, int quantity)
    {
        var item = new OrderItem(productId, unitPrice, quantity);
        _items.Add(item);
        RecalculateTotal();
    }
    
    private void RecalculateTotal()
    {
        TotalAmount = _items.Aggregate(
            new Money(0, "USD"),
            (total, item) => total.Add(item.Subtotal)
        );
    }
}

// Entity
public class OrderItem
{
    public Guid ProductId { get; private set; }
    public Money UnitPrice { get; private set; }
    public int Quantity { get; private set; }
    public Money Subtotal => UnitPrice with { Amount = UnitPrice.Amount * Quantity };
    
    public OrderItem(Guid productId, Money unitPrice, int quantity)
    {
        ProductId = productId;
        UnitPrice = unitPrice;
        Quantity = quantity;
    }
}

// Domain Event base class
public abstract record DomainEvent(DateTime OccurredOn);
```

## جمع‌بندی

### چه زمانی از Record استفاده کنیم؟

✅ **Value Objects**: Money, Address, Email, Coordinates, DateRange
✅ **Domain Events**: OrderPlacedEvent, CustomerRegisteredEvent
✅ **DTOs**: CustomerDto, OrderDto
✅ **Query Results**: وقتی فقط برای خواندن استفاده می‌شود

### چه زمانی از Record استفاده نکنیم؟

❌ **Entities**: چون Identity-based Equality نیاز دارند
❌ **Aggregate Roots**: چون Mutable هستند و تغییر می‌کنند
❌ **Objects با Navigation Properties**: چون Circular Reference ایجاد می‌کند

### قوانین کلی

1. **Value Object → Record**: همیشه مناسب است
2. **Entity → Class**: همیشه مناسب‌تر است
3. **Domain Event → Record**: معمولاً مناسب است
4. **DTO → Record**: معمولاً مناسب است

## منابع

### Microsoft Documentation
- [Records (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- [Record types in C#](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)

### DDD Resources
- [Domain-Driven Design Reference by Eric Evans](https://www.domainlanguage.com/ddd/reference/)
- [Implementing Domain-Driven Design by Vaughn Vernon](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577)
- [Microsoft DDD Architecture](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/)
- [eShopOnContainers DDD Implementation](https://github.com/dotnet-architecture/eShopOnContainers)

### مقالات مرتبط
- [Value Objects in DDD](https://martinfowler.com/bliki/ValueObject.html)
- [Entity vs Value Object](https://enterprisecraftsmanship.com/posts/entity-vs-value-object-the-whole-story/)
- [C# Records for Value Objects](https://enterprisecraftsmanship.com/posts/record-types-in-csharp-for-value-objects/)

---

**نویسنده**: دستیار هوش مصنوعی  
**تاریخ**: August 28, 2026  
**نسخه**: 1.0