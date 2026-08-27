# اشتباهات رایج در استفاده از Tuple در C#

> **سطح:** متوسط تا پیشرفته  
> **زمان مطالعه:** ~۲۵ دقیقه  
> **نسخه C# مورد نیاز:** ۷.۰ به بالا (ValueTuple)

---

## فهرست مطالب

1. [مقدمه](#مقدمه)
2. [اشتباه ۱: استفاده بیش از حد از Tuple](#اشتباه-۱-استفاده-بیش-از-حد-از-tuple)
3. [اشتباه ۲: Tupleهای بسیار بزرگ](#اشتباه-۲-tupleهای-بسیار-بزرگ)
4. [اشتباه ۳: استفاده نامناسب در Domain Model](#اشتباه-۳-استفاده-نامناسب-در-domain-model)
5. [اشتباه ۴: استفاده نامناسب در Public API](#اشتباه-۴-استفاده-نامناسب-در-public-api)
6. [اشتباه ۵: استفاده از Item1 و Item2 به جای نام‌گذاری](#اشتباه-۵-استفاده-از-item1-و-item2-به-جای-نامگذاری)
7. [اشتباه ۶: اشتباه در Deconstruction](#اشتباه-۶-اشتباه-در-deconstruction)
8. [اشتباه ۷: اشتباه در Equality](#اشتباه-۷-اشتباه-در-equality)
9. [اشتباه ۸: استفاده Mutable از Tuple به عنوان Dictionary Key](#اشتباه-۸-استفاده-mutable-از-tuple-به-عنوان-dictionary-key)
10. [اشتباه ۹: اشتباه درباره Stack و Heap](#اشتباه-۹-اشتباه-درباره-stack-و-heap)
11. [اشتباه ۱۰: اشتباه درباره Performance](#اشتباه-۱۰-اشتباه-درباره-performance)
12. [اشتباه ۱۱: اشتباه گرفتن Tuple و Anonymous Type](#اشتباه-۱۱-اشتباه-گرفتن-tuple-و-anonymous-type)
13. [اشتباه ۱۲: اشتباه گرفتن Tuple و System.Tuple](#اشتباه-۱۲-اشتباه-گرفتن-tuple-و-systemtuple)
14. [اشتباه ۱۳: استفاده نامناسب در Serialization](#اشتباه-۱۳-استفاده-نامناسب-در-serialization)
15. [اشتباه ۱۴: کد بد و نسخه اصلاح‌شده](#اشتباه-۱۴-کد-بد-و-نسخه-اصلاحشده)
16. [جدول مقایسه سریع](#جدول-مقایسه-سریع)
17. [جمع‌بندی](#جمع‌بندی)
18. [منابع معتبر](#منابع-معتبر)

---

## مقدمه

از نسخه C# 7.0، `ValueTuple` به زبان اضافه شد و امکان ساخت ساختارهای سبک چندمقداری را فراهم کرد. این ویژگی برای بازگرداندن چند مقدار از یک متد، `Deconstruction` و کدهای کوتاه‌تر بسیار مفید است. اما استفاده نادرست از آن می‌تواند منجر به کدهای غیرخوانا، مشکلات Performance و باگ‌های سخت‌پیدا شود.

در این مقاله، ۱۴ اشتباه رایج را بررسی می‌کنیم.

---

## اشتباه ۱: استفاده بیش از حد از Tuple

### ❌ کد اشتباه

```csharp
public (int, string, bool, DateTime, double) ProcessUser()
{
    // ...
    return (userId, name, isActive, createdAt, balance);
}

// استفاده
var result = ProcessUser();
Console.WriteLine(result.Item1); // چیست؟
Console.WriteLine(result.Item3); // چه معنایی دارد؟
```

### 🔍 چرا اشتباه است؟

وقتی تعداد فیلدها زیاد باشد و معنای هرکدام مشخص نباشد، کد **غیرخوانا** می‌شود. خواننده باید به تعریف متد مراجعه کند تا بفهمد هر فیلد چیست. این کار **Cognitive Load** را افزایش می‌دهد.

### ✅ نسخه بهتر

```csharp
public record UserSummary(
    int UserId,
    string Name,
    bool IsActive,
    DateTime CreatedAt,
    double Balance);

public UserSummary ProcessUser()
{
    // ...
    return new UserSummary(userId, name, isActive, createdAt, balance);
}
```

### 💡 Best Practice

> اگر بیشتر از **۲ تا ۳ فیلد** دارید، یا فیلدها معنای واضحی ندارند، از `record` یا `class` استفاده کنید. Tuple برای موارد **موقت و داخلی** مناسب است.

---

## اشتباه ۲: Tupleهای بسیار بزرگ

### ❌ کد اشتباه

```csharp
public (int, string, string, DateTime, decimal, bool, int, string, Guid) GetOrderDetails()
{
    return (1, "Order-001", "Pending", DateTime.Now, 99.99m, true, 5, "Electronics", Guid.NewGuid());
}
```

### 🔍 چرا اشتباه است؟

- `ValueTuple` در C# حداکثر **۸ فیلد** مستقیم پشتیبانی می‌کند. بیشتر از آن، به صورت تو در تو (`Rest`) ذخیره می‌شود.
- خوانایی به شدت کاهش می‌یابد.
- نگهداری کد دشوار می‌شود.

### ✅ نسخه بهتر

```csharp
public record OrderInfo(
    int OrderId,
    string OrderNumber,
    string Status,
    DateTime OrderDate,
    decimal TotalAmount,
    bool IsPaid,
    int ItemCount,
    string Category,
    Guid TrackingId);

public OrderInfo GetOrderDetails()
{
    return new OrderInfo(1, "Order-001", "Pending", DateTime.Now, 
        99.99m, true, 5, "Electronics", Guid.NewGuid());
}
```

### 💡 Best Practice

> حداکثر **۳ تا ۴ فیلد** برای Tuple مناسب است. بیشتر از آن، از `record` یا `class` استفاده کنید.

---

## اشتباه ۳: استفاده نامناسب در Domain Model

### ❌ کد اشتباه

```csharp
public class User
{
    public (string FirstName, string LastName) Name { get; set; }
    public (string Street, string City, string ZipCode) Address { get; set; }
}
```

### 🔍 چرا اشتباه است؟

- Domain Model باید **معنای تجاری** داشته باشد. `Tuple` فاقد رفتار (Behavior) است.
- Validation، منطق تجاری و Invariantها نمی‌توانند در Tuple قرار بگیرند.
- در ORMها مثل Entity Framework، نگاشت Tuple مشکل‌ساز است.

### ✅ نسخه بهتر

```csharp
public record FullName(string FirstName, string LastName)
{
    public string GetDisplayName() => $"{FirstName} {LastName}";
}

public record Address(string Street, string City, string ZipCode)
{
    public bool IsValid() => !string.IsNullOrWhiteSpace(ZipCode);
}

public class User
{
    public FullName Name { get; set; }
    public Address Address { get; set; }
}
```

### 💡 Best Practice

> برای Domain Model همیشه از `record` یا `class` استفاده کنید تا بتوانید **رفتار و Validation** اضافه کنید.

---

## اشتباه ۴: استفاده نامناسب در Public API

### ❌ کد اشتباه

```csharp
public class UserService
{
    public (bool Success, string Error) CreateUser(string email)
    {
        // ...
    }
}
```

### 🔍 چرا اشتباه است؟

- Tupleها **نام‌گذاری‌شان در سطح باینری از بین می‌رود** (Attribute-based names).
- در زبان‌های دیگر (مثل F# یا VB.NET) نام فیلدها دیده نمی‌شود.
- تغییر نام فیلدها در آینده، **Breaking Change** نیست ولی باعث سردرگمی می‌شود.
- برای APIهای عمومی، `record` یا کلاس‌های Result مشخص بهترند.

### ✅ نسخه بهتر

```csharp
public record CreateUserResult(bool Success, string? Error);

public class UserService
{
    public CreateUserResult CreateUser(string email)
    {
        // ...
        return new CreateUserResult(true, null);
    }
}
```

### 💡 Best Practice

> در **Public API** و **Libraryها**، از `record` استفاده کنید. Tuple فقط برای استفاده **داخلی (Internal/Private)** مناسب است.

---

## اشتباه ۵: استفاده از Item1 و Item2 به جای نام‌گذاری

### ❌ کد اشتباه

```csharp
var result = GetCoordinates();
Console.WriteLine($"X: {result.Item1}, Y: {result.Item2}");
```

### 🔍 چرا اشتباه است؟

- `Item1` و `Item2` هیچ **معنای معنایی** ندارند.
- کد را غیرخوانا می‌کنند.
- احتمال اشتباه در ترتیب فیلدها زیاد است.

### ✅ نسخه بهتر

```csharp
// روش ۱: نام‌گذاری در تعریف
var (x, y) = GetCoordinates();
Console.WriteLine($"X: {x}, Y: {y}");

// روش ۲: نام‌گذاری در Tuple
(int X, int Y) result = GetCoordinates();
Console.WriteLine($"X: {result.X}, Y: {result.Y}");
```

### 💡 Best Practice

> **همیشه** از نام‌گذاری استفاده کنید. `Item1` و `Item2` فقط در موارد بسیار خاص و موقت قابل قبول هستند.

---

## اشتباه ۶: اشتباه در Deconstruction

### ❌ کد اشتباه

```csharp
public (int Id, string Name, string Email) GetUser()
{
    return (1, "Ali", "ali@example.com");
}

// اشتباه: ترتیب اشتباه
var (email, name, id) = GetUser();
Console.WriteLine($"ID: {id}"); // اشتباه! این در واقع email است
```

### 🔍 چرا اشتباه است؟

- در Deconstruction، **ترتیب متغیرها** مهم است، نه نام آنها.
- کامپایلر بر اساس **موقعیت** مقداردهی می‌کند.
- این باعث باگ‌های سخت‌پیدا می‌شود.

### ✅ نسخه بهتر

```csharp
// روش ۱: ترتیب صحیح
var (id, name, email) = GetUser();

// روش ۲: استفاده از discard برای فیلدهای غیرضروری
var (id, _, email) = GetUser();

// روش ۳: Deconstruction در متد
public void Deconstruct(out int id, out string name, out string email)
{
    id = Id;
    name = Name;
    email = Email;
}
```

### 💡 Best Practice

> همیشه **ترتیب متغیرها** را با ترتیب تعریف Tuple مطابقت دهید. از `discard (_)` برای فیلدهای غیرضروری استفاده کنید.

---

## اشتباه ۷: اشتباه در Equality

### ❌ کد اشتباه

```csharp
var tuple1 = (1, "Hello");
var tuple2 = (1, "Hello");

Console.WriteLine(tuple1 == tuple2); // True - درست است

// اما با reference types:
var tuple3 = (1, new StringBuilder("Hello"));
var tuple4 = (1, new StringBuilder("Hello"));

Console.WriteLine(tuple3 == tuple4); // False!
```

### 🔍 چرا اشتباه است؟

- `ValueTuple` از **Equality مقایسه‌ای** استفاده می‌کند.
- برای reference types، **Reference Equality** مقایسه می‌شود، نه مقدار.
- این باعث نتایج غیرمنتظره می‌شود.

### ✅ نسخه بهتر

```csharp
// استفاده از immutable types
var tuple3 = (1, "Hello");
var tuple4 = (1, "Hello");
Console.WriteLine(tuple3 == tuple4); // True

// یا پیاده‌سازی IEquatable
public record CustomTuple(int Id, string Name)
{
    public virtual bool Equals(CustomTuple? other)
    {
        return other != null && Id == other.Id && Name == other.Name;
    }
}
```

### 💡 Best Practice

> فقط از **immutable value types** (مثل `int`, `string`, `record`) در Tuple استفاده کنید. از reference typeهای mutable پرهیز کنید.

---

## اشتباه ۸: استفاده Mutable از Tuple به عنوان Dictionary Key

### ❌ کد اشتباه

```csharp
var dict = new Dictionary<(int X, int Y), string>();
var key = (1, 2);
dict[key] = "Point A";

// تغییر mutable field
key.X = 5; // ❌ این کار GetHashCode را خراب می‌کند!

Console.WriteLine(dict.ContainsKey((1, 2))); // False!
Console.WriteLine(dict.ContainsKey((5, 2))); // False! - Key گم شده!
```

### 🔍 چرا اشتباه است؟

- `ValueTuple` یک **struct** است، پس copy-by-value می‌شود.
- اما اگر فیلدهای reference type mutable داشته باشید، تغییر آنها **HashCode** را تغییر می‌دهد.
- این باعث می‌شود Key در Dictionary **گم** شود.

### ✅ نسخه بهتر

```csharp
// روش ۱: استفاده از immutable types
var dict = new Dictionary<(int X, int Y), string>();
var key = (1, 2);
dict[key] = "Point A";
// key را تغییر ندهید، یک key جدید بسازید

// روش ۲: استفاده از record
var dict2 = new Dictionary<Point, string>();
record Point(int X, int Y);
dict2[new Point(1, 2)] = "Point A";
```

### 💡 Best Practice

> **هرگز** Keyهای Dictionary را تغییر ندهید. از immutable types استفاده کنید.

---

## اشتباه ۹: اشتباه درباره Stack و Heap

### ❌ کد اشتباه

```csharp
// فرض اشتباه: ValueTuple همیشه در Stack است
public (int, int)[] GetPoints()
{
    var points = new (int, int)[1000];
    for (int i = 0; i < 1000; i++)
    {
        points[i] = (i, i * 2); // Boxing اتفاق می‌افتد!
    }
    return points;
}
```

### 🔍 چرا اشتباه است؟

- `ValueTuple` یک **struct** است، اما وقتی در **Array** یا به عنوان **field در class** قرار بگیرد، در **Heap** ذخیره می‌شود.
- فرض اینکه همیشه در Stack است، **اشتباه** است.
- در Array، structها به صورت **inline** ذخیره می‌شوند ولی در Heap هستند.

### ✅ نسخه بهتر

```csharp
// اگر واقعاً Performance مهم است:
public Span<(int X, int Y)> GetPoints(Span<(int X, int Y)> buffer)
{
    for (int i = 0; i < buffer.Length; i++)
    {
        buffer[i] = (i, i * 2);
    }
    return buffer;
}

// استفاده:
Span<(int, int)> points = stackalloc (int, int)[100]; // واقعاً در Stack
GetPoints(points);
```

### 💡 Best Practice

> `ValueTuple` در **Array** یا **class field** در **Heap** است. فقط در **local variables** و **stackalloc** در Stack قرار می‌گیرد.

---

## اشتباه ۱۰: اشتباه درباره Performance

### ❌ کد اشتباه

```csharp
// فرض اشتباه: Tuple همیشه سریع‌تر از class است
public class Point
{
    public int X { get; set; }
    public int Y { get; set; }
}

public (int, int) GetPoint() => (1, 2); // فرض: سریع‌تر

// اما در واقع:
public Point GetPointClass() => new Point { X = 1, Y = 2 };
```

### 🔍 چرا اشتباه است؟

- `ValueTuple` در **local variables** سریع‌تر است (_allocation کمتر_).
- اما در **Array** یا **field**، تفاوت Performance ناچیز است.
- در برخی موارد، **JIT Optimization** برای class بهتر کار می‌کند.
- **Benchmark** واقعی بهترین راه است.

### ✅ نسخه بهتر

```csharp
// Benchmark واقعی:
[MemoryDiagnoser]
public class TupleBenchmark
{
    [Benchmark]
    public (int, int) CreateTuple() => (1, 2);

    [Benchmark]
    public Point CreateClass() => new Point { X = 1, Y = 2 };

    [Benchmark]
    public PointRecord CreateRecord() => new PointRecord(1, 2);
}
```

### 💡 Best Practice

> **Benchmark** بگیرید. فرضیات بدون اندازه‌گیری، **اشتباه** هستند. در اکثر موارد، تفاوت ناچیز است.

---

## اشتباه ۱۱: اشتباه گرفتن Tuple و Anonymous Type

### ❌ کد اشتباه

```csharp
// Anonymous Type
var anon = new { Name = "Ali", Age = 30 };
Console.WriteLine(anon.Name);

// Tuple
var tuple = (Name: "Ali", Age: 30);
Console.WriteLine(tuple.Name);

// فرض اشتباه: اینها یکسان هستند
var result = GetData();
// اگر GetData() Anonymous Type برگرداند:
// var (name, age) = result; // ❌ کار نمی‌کند!
```

### 🔍 چرا اشتباه است؟

- **Anonymous Type**: class است، در **Heap**، فقط در **همان متد** قابل استفاده است.
- **Tuple**: struct است، در **Stack** (در local)، قابل **بازگشت از متد** است.
- **Deconstruction** فقط برای Tuple کار می‌کند، نه Anonymous Type.

### ✅ نسخه بهتر

```csharp
// اگر نیاز به بازگشت از متد دارید:
public (string Name, int Age) GetData() => ("Ali", 30);

// اگر فقط در همان متد استفاده می‌کنید:
public void ProcessData()
{
    var anon = new { Name = "Ali", Age = 30 };
    Console.WriteLine(anon.Name);
}
```

### 💡 Best Practice

> **Anonymous Type** فقط برای **LINQ queries** و استفاده **موقت در همان متد**. برای بازگشت از متد، از **Tuple** یا **record** استفاده کنید.

---

## اشتباه ۱۲: اشتباه گرفتن Tuple و System.Tuple

### ❌ کد اشتباه

```csharp
// System.Tuple (قدیمی، از C# 4.0)
System.Tuple<int, string> oldTuple = System.Tuple.Create(1, "Hello");
Console.WriteLine(oldTuple.Item1); // فقط Item1, Item2

// ValueTuple (جدید، از C# 7.0)
(int Id, string Name) newTuple = (1, "Hello");
Console.WriteLine(newTuple.Id); // نام‌گذاری شده

// فرض اشتباه: اینها یکسان هستند
```

### 🔍 چرا اشتباه است؟

- `System.Tuple`: **class** است، **Reference Type**، **Immutable**، نام‌گذاری **ندارد**.
- `ValueTuple`: **struct** است، **Value Type**، **Mutable**، نام‌گذاری **دارد**.
- Performance کاملاً متفاوت است.

### ✅ نسخه بهتر

```csharp
// همیشه از ValueTuple استفاده کنید (سینتکس جدید)
(int Id, string Name) tuple = (1, "Hello");

// یا با var:
var tuple = (Id: 1, Name: "Hello");

// هرگز از System.Tuple استفاده نکنید مگر برای سازگاری با کد قدیمی
```

### 💡 Best Practice

> **همیشه** از `ValueTuple` (سینتکس جدید) استفاده کنید. `System.Tuple` فقط برای **سازگاری با کد قدیمی** (.NET Framework قبل از 4.7) است.

---

## اشتباه ۱۳: استفاده نامناسب در Serialization

### ❌ کد اشتباه

```csharp
public record UserDto((int Id, string Name) UserInfo, string Email);

// Serialization با System.Text.Json
var user = new UserDto((1, "Ali"), "ali@example.com");
var json = JsonSerializer.Serialize(user);

// خروجی:
// {"UserInfo":{"Item1":1,"Item2":"Ali"},"Email":"ali@example.com"}
// ❌ نام‌ها از بین رفتند!
```

### 🔍 چرا اشتباه است؟

- `ValueTuple` نام فیلدها را در **Runtime** از دست می‌دهد (فقط در Compile-time با Attribute ذخیره می‌شود).
- Serializerها نام‌ها را نمی‌بینند و از `Item1`, `Item2` استفاده می‌کنند.
- در **APIها** و **Database** این مشکل‌ساز است.

### ✅ نسخه بهتر

```csharp
// روش ۱: استفاده از record
public record UserInfo(int Id, string Name);
public record UserDto(UserInfo UserInfo, string Email);

// روش ۲: اگر مجبور به استفاده از Tuple هستید
public record UserDto(
    [property: JsonPropertyName("UserId")] int Id,
    [property: JsonPropertyName("UserName")] string Name,
    string Email);
```

### 💡 Best Practice

> برای **Serialization** (JSON, XML, Database) هرگز از Tuple استفاده نکنید. از **record** یا **class** با نام‌گذاری واضح استفاده کنید.

---

## اشتباه ۱۴: کد بد و نسخه اصلاح‌شده

### ❌ کد بد (همه اشتباهات در یکجا)

```csharp
public class UserService
{
    public (int, string, string, DateTime, decimal, bool, int, string, Guid) GetUserDetails(int userId)
    {
        // ...
        return (userId, "Ali", "Ahmadi", DateTime.Now, 1000.50m, true, 5, "Active", Guid.NewGuid());
    }
}

// استفاده:
var service = new UserService();
var result = service.GetUserDetails(1);
Console.WriteLine(result.Item1); // چیست؟
Console.WriteLine(result.Item5); // چه معنایی دارد؟

// Serialization:
var json = JsonSerializer.Serialize(result);
// {"Item1":1,"Item2":"Ali","Item3":"Ahmadi",...}
```

### ✅ نسخه اصلاح‌شده

```csharp
public record UserSummary(
    int UserId,
    string FirstName,
    string LastName,
    DateTime CreatedAt,
    decimal Balance,
    bool IsActive,
    int LoginCount,
    string Status,
    Guid TrackingId);

public class UserService
{
    public UserSummary GetUserDetails(int userId)
    {
        // ...
        return new UserSummary(
            userId, "Ali", "Ahmadi", DateTime.Now, 
            1000.50m, true, 5, "Active", Guid.NewGuid());
    }
}

// استفاده:
var service = new UserService();
var user = service.GetUserDetails(1);
Console.WriteLine(user.UserId); // واضح
Console.WriteLine(user.Balance); // معنادار

// Serialization:
var json = JsonSerializer.Serialize(user);
// {"UserId":1,"FirstName":"Ali","LastName":"Ahmadi",...}
```

### 💡 Best Practice

> برای **بیشتر از ۳ فیلد**، **Public API**، **Serialization** و **Domain Model**، همیشه از `record` استفاده کنید.

---

## جدول مقایسه سریع

| ویژگی | ValueTuple (جدید) | System.Tuple (قدیمی) | Anonymous Type | Record |
|-------|-------------------|----------------------|----------------|--------|
| **نوع** | struct | class | class | class/struct |
| **مکان ذخیره** | Stack (local) | Heap | Heap | Heap |
| **نام‌گذاری** | ✅ دارد | ❌ ندارد | ✅ دارد | ✅ دارد |
| **Mutable** | ✅ بله | ❌ خیر | ❌ خیر | ❌ خیر (پیش‌فرض) |
| **Deconstruction** | ✅ دارد | ❌ ندارد | ❌ ندارد | ✅ دارد |
| **بازگشت از متد** | ✅ بله | ✅ بله | ❌ خیر | ✅ بله |
| **Serialization** | ⚠️ مشکل‌ساز | ⚠️ مشکل‌ساز | ❌ خیر | ✅ عالی |
| **Performance** | ⚡ عالی | 🐢 کندتر | 🐢 کندتر | 🐢 کندتر |
| **کاربرد** | Internal, موقت | Legacy code | LINQ | Public API, Domain |

---

## جمع‌بندی

### ✅ چه زمانی از Tuple استفاده کنیم؟

1. **بازگرداندن ۲-۳ مقدار** از یک متد **خصوصی/داخلی**
2. **Deconstruction** ساده
3. **کدهای موقت** و **LINQ queries**
4. وقتی **Performance** مهم است و **Allocation** باید حداقل باشد

### ❌ چه زمانی از Tuple استفاده نکنیم؟

1. **Public API** و **Libraryها**
2. **Domain Model** و **Business Logic**
3. **Serialization** (JSON, XML, Database)
4. **بیشتر از ۳-۴ فیلد**
5. **Dictionary Key** (مگر immutable)
6. وقتی **نام‌گذاری** مهم است

### 📋 چک‌لیست نهایی

- [ ] آیا تعداد فیلدها ≤ ۳ است؟
- [ ] آیا استفاده داخلی است (نه Public API)؟
- [ ] آیا نام‌گذاری کرده‌ام؟
- [ ] آیا از immutable types استفاده کرده‌ام؟
- [ ] آیا برای Serialization مناسب است؟
- [ ] آیا Performance واقعاً مهم است؟

---

## منابع معتبر

1. **Microsoft Docs - Tuples**  
   https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples

2. **C# in Depth (4th Edition) - Jon Skeet**  
   فصل ۶: Tuples and Deconstruction

3. **CLR via C# (4th Edition) - Jeffrey Richter**  
   فصل ۵: Value Types vs Reference Types

4. **BenchmarkDotNet Documentation**  
   https://benchmarkdotnet.org/

5. **System.Text.Json Serialization**  
   https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/

6. **C# Language Specification - Tuples**  
   https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/

---

> **نویسنده:** [نام شما]  
> **تاریخ:** August 27, 2026  
> **لایسنس:** MIT License

---

**اگر این مقاله مفید بود، لطفاً Star بدهید و برای دیگران به اشتراک بگذارید!** ⭐