# اشتباهات رایج در استفاده از Recordها در C#

از زمان معرفی `record` در C# 9 و گسترش آن با `record struct` در C# 10، این نوع‌ها به یکی از محبوب‌ترین ابزارها برای مدل‌سازی داده تبدیل شده‌اند. اما همین محبوبیت باعث شده تا بسیاری از برنامه‌نویسان بدون درک عمیق از رفتار آن‌ها، از آن‌ها استفاده کنند و با باگ‌های ظریفی مواجه شوند.

در این مقاله، ۱۲ اشتباه رایج را همراه با مثال کد و راه‌حل بررسی می‌کنیم.

---

## ۱. تصور اینکه Record همیشه Immutable است

### اشتباه چیست؟
بسیاری تصور می‌کنند چون `record` یک نوع «immutable» است، پس هیچ بخشی از آن قابل تغییر نیست.

### چرا اشتباه است؟
طبق مستندات Microsoft، `record` فقط propertyهای سطح اول را `init-only` می‌کند. اگر داخل record یک **reference type** (مثل `List<T>`، آرایه یا یک کلاس دیگر) داشته باشید، محتوای آن کاملاً قابل تغییر است. خودِ reference ثابت می‌ماند، اما state درونی آن نه.

### ❌ مثال بد
```csharp
public record ShoppingCart(List<Product> Items);

var cart = new ShoppingCart(new List<Product> { new Product("Laptop") });
cart.Items.Add(new Product("Mouse")); // کامپایل می‌شود و کار می‌کند!
// حالا دو reference از cart که فکر می‌کردید برابرند، محتوای متفاوت دارند.
```

### ✅ نسخه اصلاح‌شده
```csharp
public record ShoppingCart(IReadOnlyList<Product> Items);

// یا استفاده از ImmutableArray
public record ShoppingCart(ImmutableArray<Product> Items);
```

### 📌 قانون ساده
> **Record فقط reference را قفل می‌کند، نه محتوای آن را.** اگر داخلش reference type باشد، mutability نشت می‌کند.

---

## ۲. استفاده Record برای Entity

### اشتباه چیست؟
استفاده از `record` برای مدل‌هایی مثل `User`، `Order` یا `Customer` که هویت (Identity) دارند.

### چرا اشتباه است؟
Entityها بر اساس **شناسه** هویت می‌یابند، نه بر اساس مقدار همه فیلدها. دو `User` با `Id` یکسان ولی نام متفاوت، در واقع یک موجودیت هستند. اما `record` equality را بر اساس همه propertyها مقایسه می‌کند و این منطق را می‌شکند.

### ❌ مثال بد
```csharp
public record User(int Id, string Name, string Email);

var user1 = new User(1, "Ali", "ali@test.com");
var user2 = new User(1, "Ali Rezaei", "ali@test.com");

Console.WriteLine(user1 == user2); // False! در حالی که یک User هستند.
```

### ✅ نسخه اصلاح‌شده
```csharp
// برای Entity از class معمولی استفاده کنید
public class User
{
    public int Id { get; init; }
    public string Name { get; set; }
    public string Email { get; set; }
    
    public override bool Equals(object? obj) => 
        obj is User other && Id == other.Id;
    public override int GetHashCode() => Id.GetHashCode();
}
```

### 📌 قانون ساده
> **اگر `Id` دارد، `class` باشد. اگر فقط داده است، `record` باشد.**

---

## ۳. Mutable Collection داخل Record

### اشتباه چیست؟
قرار دادن `List<T>` یا آرایه به‌عنوان property در record.

### چرا اشتباه است؟
این مورد زیرمجموعه اشتباه اول است اما آن‌قدر رایج است که شایسته بحث جداگانه است. وقتی دو record برابر (`==`) هستند، انتظار داریم `GetHashCode` یکسان و رفتار یکسانی داشته باشند. اما اگر collection قابل تغییر باشد، این تضمین از بین می‌رود.

### ❌ مثال بد
```csharp
public record Order(List<int> ProductIds);

var order1 = new Order(new List<int> { 1, 2, 3 });
var order2 = new Order(new List<int> { 1, 2, 3 });

Console.WriteLine(order1 == order2); // True

order1.ProductIds.Add(4);
Console.WriteLine(order1 == order2); // حالا False شد!
// یک شیء که در Dictionary کلید بود، گم می‌شود.
```

### ✅ نسخه اصلاح‌شده
```csharp
public record Order(IReadOnlyList<int> ProductIds)
{
    // Constructor ایمن‌سازی می‌کند
    public Order(IEnumerable<int> ids) 
        : this(ids.ToImmutableList()) { }
}
```

### 📌 قانون ساده
> **هر collection داخل record باید read-only باشد؛ ترجیحاً از `System.Collections.Immutable` استفاده کنید.**

---

## ۴. اشتباه گرفتن Value Equality با Reference Equality

### اشتباه چیست؟
فرض اینکه دو record برابر، در واقع یک شیء در حافظه هستند.

### چرا اشتباه است؟
`record` یک **reference type** است (مگر `record struct`). دو record با مقادیر یکسان، دو reference متفاوت دارند. `==` در recordها **value equality** را بررسی می‌کند، نه reference equality را.

### ❌ مثال بد
```csharp
public record Point(int X, int Y);

var p1 = new Point(1, 2);
var p2 = new Point(1, 2);

var set = new HashSet<Point> { p1 };
Console.WriteLine(set.Contains(p2)); // True ✓

// اما:
object o1 = p1;
object o2 = p2;
Console.WriteLine(ReferenceEquals(o1, o2)); // False
// اگر کد شما به reference identity وابسته باشد، باگ می‌دهد.
```

### ✅ نسخه اصلاح‌شده
```csharp
// اگر به reference identity نیاز دارید، از class معمولی استفاده کنید
// یا از ReferenceEquals برای بررسی آگاهی استفاده کنید
if (ReferenceEquals(p1, p2)) { /* واقعاً یک شیء */ }
else if (p1 == p2) { /* مقادیر برابر */ }
```

### 📌 قانون ساده
> **`==` در recordها مقدار را مقایسه می‌کند، نه آدرس حافظه را.**

---

## ۵. تصور Deep Copy بودن `with`

### اشتباه چیست؟
فرض اینکه `with` expression یک کپی عمیق (deep copy) می‌سازد.

### چرا اشتباه است؟
طبق مستندات Microsoft، `with` یک **shallow copy** می‌سازد. یعنی referenceهای داخلی بین کپی و اصلی مشترک‌اند.

### ❌ مثال بد
```csharp
public record Address(string City);
public record Person(string Name, Address Address);

var ali = new Person("Ali", new Address("Tehran"));
var aliCopy = ali with { Name = "Ali Rezaei" };

Console.WriteLine(ReferenceEquals(ali.Address, aliCopy.Address)); // True!
// تغییر Address در یکی، روی دیگری اثر می‌گذارد (اگر Address mutable باشد)
```

### ✅ نسخه اصلاح‌شده
```csharp
var aliCopy = ali with { Address = ali.Address with { City = "Shiraz" } };

// یا متد کمکی برای deep copy
public Person DeepCopy() => this with 
{ 
    Address = this.Address with { } 
};
```

### 📌 قانون ساده
> **`with` فقط یک لایه کپی می‌کند. برای لایه‌های داخلی باید خودتان `with` تو‌در‌تو بنویسید.**

---

## ۶. استفاده نادرست از `record struct`

### اشتباه چیست؟
استفاده از `record struct` برای داده‌های بزرگ یا پیچیده، یا تصور اینکه رفتار آن دقیقاً مثل `record` است.

### چرا اشتباه است؟
`record struct` یک **value type** است. این یعنی:
- روی stack کپی می‌شود (برای داده‌های بزرگ، پرفورمنس بد)
- `with` در آن **mutable** است (می‌توانید propertyها را ست کنید!)
- Boxing در سناریوهای خاص
- `Equals` در آن با `record` متفاوت عمل می‌کند

### ❌ مثال بد
```csharp
public record struct BigData(byte[] Payload, string[] Tags, Dictionary<int, string> Map);
// 1) struct بزرگ = کپی‌های پرهزینه
// 2) propertyها mutable هستند!

var d1 = new BigData(...);
var d2 = d1;
d2.Payload[0] = 0; // d1 هم تغییر می‌کند!
```

### ✅ نسخه اصلاح‌شده
```csharp
// فقط برای structهای کوچک و واقعاً value-based
public record struct Point(int X, int Y);

// برای داده‌های بزرگ، همان record (reference type)
public record BigData(ImmutableArray<byte> Payload, ...);
```

### 📌 قانون ساده
> **`record struct` فقط برای داده‌های کوچک (< 16-24 bytes) و بدون referenceهای تو‌در‌تو.**

---

## ۷. Equality سفارشی ناقص

### اشتباه چیست؟
override کردن `Equals` یا `PrintMembers` بدون در نظر گرفتن سایر اعضای equality chain.

### چرا اشتباه است؟
`record` یک زنجیره کامل از متدها را تولید می‌کند: `Equals(T)`, `Equals(object)`, `GetHashCode`, `==`, `!=`, `PrintMembers`, `ToString`. اگر یکی را ناقص override کنید، رفتار ناسازگار می‌شود.

### ❌ مثال بد
```csharp
public record User(string Name, string Email)
{
    // فقط Name را در نظر می‌گیریم
    public virtual bool Equals(User? other) => 
        other is not null && Name == other.Name;
    // GetHashCode را override نکردیم!
}

var u1 = new User("Ali", "a@x.com");
var u2 = new User("Ali", "b@x.com");
Console.WriteLine(u1 == u2); // True
Console.WriteLine(u1.GetHashCode() == u2.GetHashCode()); // False!
// Dictionary و HashSet خراب می‌شوند.
```

### ✅ نسخه اصلاح‌شده
```csharp
public record User(string Name, string Email)
{
    public virtual bool Equals(User? other) =>
        other is not null && Name == other.Name;
    
    public override int GetHashCode() => 
        HashCode.Combine(Name);
}
```

### 📌 قانون ساده
> **هر وقت `Equals` را override کردی، `GetHashCode` را هم با همان منطق override کن.**

---

## ۸. نادیده گرفتن `GetHashCode`

### اشتباه چیست؟
استفاده از record در `Dictionary` یا `HashSet` بدون درک اینکه `GetHashCode` چگونه کار می‌کند.

### چرا اشتباه است؟
`record` به‌طور خودکار `GetHashCode` را بر اساس همه propertyها تولید می‌کند. اگر هر کدام از این propertyها mutable باشد (اشتباه شماره ۳)، hash code بعد از insertion تغییر می‌کند و شیء در collection گم می‌شود.

### ❌ مثال بد
```csharp
public record Config(Dictionary<string, string> Settings);

var cfg = new Config(new Dictionary<string, string> { ["key"] = "value" });
var dict = new HashSet<Config> { cfg };

cfg.Settings["key"] = "new"; // hash code تغییر کرد!
Console.WriteLine(dict.Contains(cfg)); // ممکن است False شود!
```

### ✅ نسخه اصلاح‌شده
```csharp
public record Config(IImmutableDictionary<string, string> Settings);

// یا اگر مجبور به mutable بودن هستید، equality را فقط روی immutable parts بسازید
public record Config(Dictionary<string, string> Settings)
{
    public override int GetHashCode() => 0; // فقط اگر واقعاً نمی‌شود hash کرد
}
```

### 📌 قانون ساده
> **اگر چیزی که در hash شرکت می‌کند قابل تغییر است، record را در Dictionary نگذار.**

---

## ۹. استفاده Record برای Objectهای دارای Lifecycle

### اشتباه چیست؟
استفاده از `record` برای کلاس‌هایی که در طول عمرشان state تغییر می‌کند، مثل `DbContext`، `HttpClient`، یا `ViewModel`ها.

### چرا اشتباه است؟
Recordها برای **snapshotهای immutable** از داده طراحی شده‌اند. اشیاء دارای lifecycle معمولاً نیاز به mutation، event raising، و مدیریت resource دارند که با فلسفه record در تضاد است.

### ❌ مثال بد
```csharp
public record OrderProcessor(IRepository Repo, ILogger Logger)
{
    public int ProcessedCount { get; set; } // mutable state!
    
    public void Process(Order o)
    {
        Repo.Save(o);
        ProcessedCount++;
        Logger.Log("done");
    }
}
```

### ✅ نسخه اصلاح‌شده
```csharp
// برای serviceها و اشیاء stateful از class معمولی استفاده کنید
public class OrderProcessor
{
    private readonly IRepository _repo;
    private readonly ILogger _logger;
    public int ProcessedCount { get; private set; }
    // ...
}
```

### 📌 قانون ساده
> **اگر behavior دارد یا lifecycle دارد، `class` باشد. اگر فقط data است، `record` باشد.**

---

## ۱۰. استفاده Record در جایی که Identity اهمیت دارد

### اشتباه چیست؟
مدل‌سازی مفاهیمی که «همان بودن» آن‌ها مهم‌تر از «برابر بودن» مقادیرشان است.

### چرا اشتباه است؟
مثال‌های واقعی: `Session`، `Connection`، `Transaction`. دو session با مقادیر یکسان، دو session جدا هستند.

### ❌ مثال بد
```csharp
public record Session(string UserId, DateTime StartTime);

var s1 = new Session("user1", now);
var s2 = new Session("user1", now);

// اگر این‌ها را در یک Set نگه دارید، با هم merge می‌شوند!
// در حالی که دو session واقعی و جدا هستند.
```

### ✅ نسخه اصلاح‌شده
```csharp
public class Session
{
    public Guid Id { get; } = Guid.NewGuid();
    public string UserId { get; init; }
    public DateTime StartTime { get; init; }
}
```

### 📌 قانون ساده
> **اگر سوال «آیا این‌ها یکی هستند؟» مهم‌تر از «آیا مقادیرشان برابر است؟» است، از `class` با Id استفاده کن.**

---

## ۱۱. استفاده بیش از حد از Record

### اشتباه چیست؟
تبدیل همه DTOها، configها، و حتی کلاس‌های داخلی به `record` بدون نیاز واقعی.

### چرا اشتباه است؟
هر `record` هزینه‌هایی دارد:
- تولید `Equals`, `GetHashCode`, `ToString`, `Clone` در compile time
- حجم بیشتر IL
- رفتار equality که ممکن است برای استفاده‌های داخلی مناسب نباشد
- عدم امکان inheritance آسان (sealed هستند به‌صورت پیش‌فرض در equality)

### ❌ مثال بد
```csharp
// یک کلاس داخلی ساده که فقط یک بار استفاده می‌شود
public record InternalCacheEntry(string Key, byte[] Value, DateTime Expires);
// Equality سفارشی لازم نیست، ToString لازم نیست، با این حال همه تولید می‌شوند
```

### ✅ نسخه اصلاح‌شده
```csharp
// اگر به equality value-based نیاز ندارید، class ساده کافی است
internal class InternalCacheEntry
{
    public string Key { get; init; }
    public byte[] Value { get; init; }
    public DateTime Expires { get; init; }
}
```

### 📌 قانون ساده
> **فقط زمانی `record` استفاده کن که واقعاً به value equality و immutability نیاز داری.**

---

## ۱۲. تصور اینکه Record همیشه Performance بهتری دارد

### اشتباه چیست؟
فرض اینکه `record` به‌خاطر «بهینه بودن» جایگزین class شده است.

### چرا اشتباه است؟
`record` یک reference type است (مگر `record struct`). رفتار equality آن بر اساس همه propertyها، برای objectهای بزرگ **پرهزینه** است. `GetHashCode` و `Equals` تولیدشده، همه فیلدها را بررسی می‌کنند.

### ❌ مثال بد
```csharp
public record LargePayload(
    byte[] Data, 
    string[] Tags, 
    Dictionary<int, string> Metadata,
    // ... 20 فیلد دیگر
);

// استفاده در HashSet با هزاران آیتم
var set = new HashSet<LargePayload>();
// هر Lookup باید همه 20 فیلد را مقایسه کند → کند
```

### ✅ نسخه اصلاح‌شده
```csharp
// 1. استفاده از record struct برای داده‌های کوچک
public record struct SmallPayload(int A, int B);

// 2. override دستی Equals/GetHashCode برای فیلدهای کلیدی
public record LargePayload(byte[] Data, string[] Tags)
{
    public virtual bool Equals(LargePayload? other) =>
        other is not null && Data.AsSpan().SequenceEqual(other.Data);
    
    public override int GetHashCode()
    {
        var hash = new HashCode();
        foreach (var b in Data.Take(16)) hash.Add(b); // فقط 16 byte اول
        return hash.ToHashCode();
    }
}
```

### 📌 قانون ساده
> **`record` برای راحتی است، نه سرعت. برای داده‌های بزرگ، equality دستی بنویس.**

---

## جمع‌بندی: چک‌لیست استفاده صحیح از Record

| سوال | پاسخ درست |
|------|-----------|
| آیا `Id` دارد؟ | `class` |
| آیا mutable state دارد؟ | `class` |
| آیا lifecycle دارد؟ | `class` |
| آیا فقط داده است و immutable؟ | `record` |
| آیا collection داخلش هست؟ | `IReadOnlyList` / Immutable |
| آیا در Dictionary استفاده می‌شود؟ | مطمئن شو hash code پایدار است |
| آیا داده بزرگ است؟ | `record struct` با احتیاط، یا equality دستی |
| آیا `with` استفاده می‌کنی؟ | یادت باشد shallow copy است |

---

**قانون طلایی نهایی:**
> `record` یک ابزار عالی برای **DTOهای immutable**، **پیام‌های CQRS**، **configuration** و **value objectها** است. اما برای **Entityها**، **Serviceها** و **اشیاء stateful**، همان `class` قدیمی بهتر است.

### منابع برای مطالعه بیشتر
- [Microsoft Learn: Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- [Microsoft Learn: Record Structs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#record-struct)
- [C# Language Reference: with expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression)