# سفارشی‌سازی Equality در Recordها — راهنمای پیشرفته C#

---

## ۱. مقدمه: چرا Equality در Recordها متفاوت است؟

در C#، وقتی یک `record` تعریف می‌کنید، کامپایلر به‌صورت خودکار **ارزش‌سنجی مبتنی بر مقدار** (Value Equality) را پیاده‌سازی می‌کند. این برخلاف `class` معمولی است که از **ارزش‌سنجی مرجعی** (Reference Equality) استفاده می‌کند.

```csharp
public record Point(int X, int Y);

var p1 = new Point(1, 2);
var p2 = new Point(1, 2);

Console.WriteLine(p1 == p2);        // True  ← Value Equality
Console.WriteLine(p1.Equals(p2));   // True
```

اما گاهی این رفتار پیش‌فرض کافی نیست. شاید بخواهید:
- برخی فیلدها را از مقایسه **حذف** کنید (مثلاً `Id` یا `Timestamp`).
- مقایسه **Case-Insensitive** روی رشته‌ها داشته باشید.
- منطق **تقریبی** (Tolerance-based) برای اعداد اعشاری پیاده کنید.

---

## ۲. آیا می‌توان `Equals` را سفارشی کرد؟

**بله، اما با قواعد مشخص.**

کامپایلر برای هر Record چندین عضو Equality تولید می‌کند. شما **نمی‌توانید** همه آن‌ها را Override کنید، اما **می‌توانید** نسخه strongly-typed یعنی `Equals(R other)` را خودتان بنویسید.

### قاعده کلیدی:
> اگر `Equals(R other)` را به‌صورت explicit تعریف کنید، کامپایلر دیگر آن را تولید **نمی‌کند**، اما `Equals(object)` را همچنان تولید کرده و به نسخه شما **واگذار** می‌کند.

---

## ۳. `Equals(R other)` — نسخه Strongly-Typed

این مهم‌ترین متدی است که می‌توانید سفارشی کنید:

```csharp
public record Product(int Id, string Name, decimal Price, DateTime CreatedAt)
{
    // ✅ کامپایلر این متد را تولید نمی‌کند چون خودمان نوشته‌ایم
    public virtual bool Equals(Product? other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;

        // فقط Name و Price مهم هستند؛ Id و CreatedAt نادیده گرفته می‌شوند
        return EqualityContract == other.EqualityContract
            && string.Equals(Name, other.Name, StringComparison.OrdinalIgnoreCase)
            && Price == other.Price;
    }
}
```

### نکات حیاتی:
- متد باید `virtual` باشد تا در Recordهای مشتق‌شده درست کار کند.
- حتماً `EqualityContract` را بررسی کنید (توضیح در بخش ۸).
- `ReferenceEquals` را برای بهینه‌سازی بررسی کنید.

---

## ۴. `GetHashCode()` — همزاد جدانشدنی Equals

### ⚠️ قانون طلایی قرارداد Equals/GetHashCode:

> **اگر `a.Equals(b) == true`، آنگاه `a.GetHashCode() == b.GetHashCode()` الزامی است.**
>
> اما عکس آن الزامی نیست: دو شیء با HashCode یکسان می‌توانند نابرابر باشند (Collision).

**چرا؟** ساختارهایی مثل `Dictionary<TKey, TValue>` و `HashSet<T>` ابتدا HashCode را بررسی می‌کنند. اگر HashCode متفاوت باشد، اصلاً `Equals` را صدا **نمی‌زنند**. پس اگر دو شیء برابر باشند ولی HashCode متفاوت داشته باشند، در Dictionary پیدا نمی‌شوند!

```csharp
public record Product(int Id, string Name, decimal Price, DateTime CreatedAt)
{
    public virtual bool Equals(Product? other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;
        return EqualityContract == other.EqualityContract
            && string.Equals(Name, other.Name, StringComparison.OrdinalIgnoreCase)
            && Price == other.Price;
    }

    // ✅ MUST: فقط همان فیلدهایی که در Equals استفاده شده‌اند
    public override int GetHashCode()
    {
        return HashCode.Combine(
            EqualityContract,
            Name?.ToUpperInvariant(),  // هماهنگ با OrdinalIgnoreCase
            Price
        );
        // ❌ Id و CreatedAt اینجا نیامده‌اند چون در Equals هم نبودند
    }
}
```

### فاجعه‌ای که بدون هماهنگی رخ می‌دهد:

```csharp
var p1 = new Product(1, "Laptop", 1000m, DateTime.Now);
var p2 = new Product(2, "laptop", 1000m, DateTime.Now.AddDays(-1));

Console.WriteLine(p1.Equals(p2));  // True  (چون case-insensitive)

var set = new HashSet<Product> { p1 };
Console.WriteLine(set.Contains(p2));
// ✅ True  — اگر GetHashCode درست باشد
// ❌ False — اگر GetHashCode فیلدهای Id/CreatedAt را هم لحاظ کند!
```

---

## ۵. `IEquatable<T>` و جایگاه آن در Recordها

هر Record به‌صورت خودکار `IEquatable<T>` را پیاده‌سازی می‌کند:

```csharp
// آنچه شما می‌نویسید:
public record Point(int X, int Y);

// آنچه کامپایلر تولید می‌کند (ساده‌شده):
public class Point : IEquatable<Point>
{
    public int X { get; init; }
    public int Y { get; init; }

    public virtual bool Equals(Point? other) { /* ... */ }
    public override bool Equals(object? obj) { /* ... */ }
    public override int GetHashCode() { /* ... */ }
    // ...
}
```

وقتی `Equals(R other)` را سفارشی می‌کنید، در واقع دارید پیاده‌سازی `IEquatable<R>` را جایگزین می‌کنید. نیازی به نوشتن `IEquatable<R>` به‌صورت جداگانه نیست.

---

## ۶. محدودیت Override کردن `Object.Equals(object)`

### ❌ این کار در Recordها **ممنوع** است:

```csharp
public record Point(int X, int Y)
{
    // ❌ CS8859: Cannot override 'object.Equals(object?)' in a record
    public override bool Equals(object? obj)
    {
        return base.Equals(obj);
    }
}
```

**دلیل:** کامپایلر `Equals(object)` را به‌صورت **`sealed override`** تولید می‌کند. این متد داخلی به‌صورت خودکار `Equals(R other)` را صدا می‌زند:

```csharp
// کد تولیدشده توسط کامپایلر (ساده‌شده):
public sealed override bool Equals(object? obj)
{
    return Equals(obj as Point);  // ← واگذاری به نسخه strongly-typed
}
```

> **نتیجه:** تنها دروازه سفارشی‌سازی، متد `Equals(R other)` است.

---

## ۷. عملگرهای `==` و `!=`

کامپایلر این عملگرها را نیز تولید می‌کند:

```csharp
// کد تولیدشده توسط کامپایلر:
public static bool operator ==(Point? left, Point? right)
{
    return (object)left == right
        || (left is not null && left.Equals(right));
}

public static bool operator !=(Point? left, Point? right)
{
    return !(left == right);
}
```

### آیا می‌توان `==` و `!=` را سفارشی کرد؟

**بله، اما توصیه نمی‌شود.** اگر خودتان تعریف کنید، کامپایلر نسخه خود را تولید نمی‌کند:

```csharp
public record Point(int X, int Y)
{
    // ⚠️ ممکن است، اما مراقب باشید
    public static bool operator ==(Point? left, Point? right)
    {
        // منطق سفارشی...
        return left?.Equals(right) ?? right is null;
    }

    public static bool operator !=(Point? left, Point? right) => !(left == right);
}
```

> **توصیه:** معمولاً نیازی به این کار نیست. سفارشی‌سازی `Equals(R other)` کافی است چون `==` به‌طور پیش‌فرض آن را صدا می‌زند.

---

## ۸. `EqualityContract` — نگهبان سلسله‌مراتب

`EqualityContract` یک `virtual property` است که نوع Record را برمی‌گرداند. هدف آن جلوگیری از برابری نادرست بین Record پایه و مشتق است:

```csharp
public record Base(int X);
public record Derived(int X, int Y) : Base(X);

var b = new Base(5);
var d = new Derived(5, 10);

Console.WriteLine(b == d);  // False ← حتی با X یکسان!
```

**چرا؟** چون `EqualityContract` برای `Base` برابر `typeof(Base)` و برای `Derived` برابر `typeof(Derived)` است.

### کد تولیدشده:

```csharp
// در Base:
protected virtual Type EqualityContract => typeof(Base);

// در Derived (کامپایلر خودکار override می‌کند):
protected override Type EqualityContract => typeof(Derived);
```

### سفارشی‌سازی EqualityContract:

در موارد بسیار نادر (مثل Recordهای `sealed`) ممکن است بخواهید آن را تغییر دهید:

```csharp
public sealed record Config(string Key, string Value)
{
    // در Recordهای sealed، EqualityContract عملاً بی‌تأثیر است
    // چون وراثتی وجود ندارد
    protected override Type EqualityContract => typeof(Config);
}
```

> **هشدار:** تغییر `EqualityContract` در Recordهای غیر-sealed می‌تواند قرارداد وراثت را بشکند.

---

## ۹. هماهنگی Equals و GetHashCode — چک‌لیست کامل

| قانون | توضیح |
|---|---|
| **فیلدهای یکسان** | دقیقاً همان فیلدهایی که در `Equals` مقایسه می‌شوند باید در `GetHashCode` هم باشند |
| **تبدیل یکسان** | اگر در `Equals` از `ToUpperInvariant` استفاده کردید، در `GetHashCode` هم باید `ToUpperInvariant` بزنید |
| **EqualityComparer یکسان** | اگر `StringComparer.OrdinalIgnoreCase` استفاده کردید، از `GetHashCode` همان comparer استفاده کنید |
| **ثبات** | HashCode یک شیء نباید در طول عمرش تغییر کند (فیلدهای mutable خطرناک‌اند) |
| **EqualityContract** | همیشه `EqualityContract` را در هر دو متد لحاظ کنید |

### مثال هماهنگی کامل با `StringComparer`:

```csharp
public record User(string Username, string Email, int LoginCount)
{
    private static readonly StringComparer Comparer = StringComparer.OrdinalIgnoreCase;

    public virtual bool Equals(User? other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;
        return EqualityContract == other.EqualityContract
            && Comparer.Equals(Username, other.Username)
            && Comparer.Equals(Email, other.Email);
        // LoginCount عمداً حذف شده
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(
            EqualityContract,
            Comparer.GetHashCode(Username),
            Comparer.GetHashCode(Email)
        );
    }
}
```

---

## ۱۰. خطرات Equality سفارشی

### ⚠️ خطر ۱: شکستن قرارداد HashCode
```csharp
// ❌ فاجعه: Equals فقط Name را چک می‌کند ولی GetHashCode همه فیلدها را
public override int GetHashCode() => HashCode.Combine(Name, Age, Email);
```

### ⚠️ خطر ۲: فیلدهای Mutable
```csharp
public record CacheEntry(string Key)
{
    public int HitCount { get; set; }  // ❌ mutable!

    public virtual bool Equals(CacheEntry? other)
    {
        // اگر HitCount در Equals باشد، تغییر آن بعد از اضافه‌شدن
        // به Dictionary باعث گم‌شدن شیء می‌شود
        return other is not null && Key == other.Key && HitCount == other.HitCount;
    }
}
```

### ⚠️ خطر ۳: عدم تقارن (Asymmetry)
```csharp
// ❌ اگر a.Equals(b) == true ولی b.Equals(a) == false
// این معمولاً وقتی رخ می‌دهد که EqualityContract درست چک نشود
```

### ⚠️ خطر ۴: شکستن `with` Expression
```csharp
var p1 = new Product(1, "Laptop", 1000m, DateTime.Now);
var p2 = p1 with { Id = 2 };  // Id تغییر کرده ولی Equals هنوز true است!
Console.WriteLine(p1 == p2);  // True — آیا این رفتار مورد انتظار شماست؟
```

### ⚠️ خطر ۵: تأثیر بر `ToString()` و `PrintMembers`
سفارشی‌سازی Equality روی `ToString()` تأثیری ندارد، اما ممکن است باعث سردرگمی شود: دو شیء با `ToString()` متفاوت می‌توانند برابر باشند.

---

## ۱۱. جدول جامع: اعضای تولیدشده vs قابل تعریف

| عضو | تولید کامپایلر | قابل تعریف صریح | توضیح |
|---|:---:|:---:|---|
| `Equals(R other)` | ✅ | ✅ | **اصلی‌ترین نقطه سفارشی‌سازی.** اگر تعریف کنید، کامپایلر تولید نمی‌کند |
| `Equals(object obj)` | ✅ `sealed` | ❌ | **غیرقابل override.** همیشه توسط کامپایلر تولید و sealed می‌شود |
| `GetHashCode()` | ✅ | ✅ | اگر تعریف کنید، کامپایلر تولید نمی‌کند |
| `operator ==` | ✅ | ✅ | معمولاً نیازی به تعریف نیست |
| `operator !=` | ✅ | ✅ | معمولاً نیازی به تعریف نیست |
| `EqualityContract` | ✅ `virtual` | ✅ `override` | در Recordهای مشتق‌شده خودکار override می‌شود |
| `PrintMembers(StringBuilder)` | ✅ `virtual` | ✅ `override` | برای سفارشی‌سازی `ToString()` |
| `ToString()` | ✅ `sealed` | ❌ | غیرقابل override؛ از `PrintMembers` استفاده کنید |
| `IEquatable<R>` | ✅ | — | خودکار پیاده‌سازی می‌شود |
| Copy Constructor | ✅ | ✅ | اگر تعریف کنید، کامپایلر تولید نمی‌کند |
| `Deconstruct(...)` | ✅ (positional) | ✅ | برای Recordهای positional |

---

## ۱۲. مثال عملی کامل: Record با Equality سفارشی‌شده

سناریو: یک سیستم مدیریت مقالات که دو مقاله با `Id` متفاوت اما `Title` و `Content` یکسان را **مساوی** در نظر می‌گیرد (برای تشخیص تکراری‌ها).

```csharp
using System.Text;

public record Article(
    Guid Id,
    string Title,
    string Content,
    DateTime PublishedAt,
    int ViewCount)
{
    // ─── سفارشی‌سازی Equality ───────────────────────────
    // دو مقاله فقط بر اساس Title و Content مقایسه می‌شوند
    public virtual bool Equals(Article? other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;

        return EqualityContract == other.EqualityContract
            && string.Equals(Title, other.Title, StringComparison.Ordinal)
            && string.Equals(Content, other.Content, StringComparison.Ordinal);
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(
            EqualityContract,
            Title,
            Content
        );
    }

    // ─── سفارشی‌سازی ToString ──────────────────────────
    protected override bool PrintMembers(StringBuilder builder)
    {
        builder.Append($"Id = {Id}, Title = \"{Title}\"");
        return true;
    }
}

// ─── تست ────────────────────────────────────────────────
public class Program
{
    public static void Main()
    {
        var a1 = new Article(
            Guid.NewGuid(), "C# Records", "Records are great...", 
            DateTime.Now, 100);

        var a2 = new Article(
            Guid.NewGuid(), "C# Records", "Records are great...", 
            DateTime.Now.AddDays(-5), 500);

        var a3 = new Article(
            Guid.NewGuid(), "C# Classes", "Classes are classic...", 
            DateTime.Now, 200);

        Console.WriteLine($"a1 == a2: {a1 == a2}");  // True  (Title+Content یکسان)
        Console.WriteLine($"a1 == a3: {a1 == a3}");  // False

        // تست HashSet — تشخیص تکراری
        var articles = new HashSet<Article> { a1 };
        Console.WriteLine($"Contains a2: {articles.Contains(a2)}");  // True ✅
        Console.WriteLine($"Contains a3: {articles.Contains(a3)}");  // False

        // تست with expression
        var a4 = a1 with { ViewCount = 9999 };
        Console.WriteLine($"a1 == a4: {a1 == a4}");  // True (ViewCount در Equals نیست)

        Console.WriteLine(a1);  // Article { Id = ..., Title = "C# Records" }
    }
}
```

---

## ۱۳. الگوی تصمیم‌گیری

```
آیا رفتار پیش‌فرض Equality کافی است؟
├── بله → هیچ کاری نکنید ✅
└── خیر →
    ├── آیا فقط می‌خواهید فیلدهایی را حذف/اضافه کنید؟
    │   └── Equals(R) + GetHashCode() را override کنید
    ├── آیا مقایسه Case-Insensitive یا Tolerance-based نیاز دارید؟
    │   └── Equals(R) + GetHashCode() با Comparer مناسب
    └── آیا رفتار == و != را هم تغییر می‌دهید؟
        └── operator == و != را هم تعریف کنید (نادر)
```

---

## ۱۴. منابع و مراجع رسمی

| منبع | لینک |
|---|---|
| **C# Record Types — Microsoft Learn** | [learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) |
| **Value Equality in Records** | [learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records#value-equality](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records#value-equality) |
| **C# Language Specification (Records)** | [learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types#1628-record-types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types#1628-record-types) |
| **IEquatable\<T\> Interface** | [learn.microsoft.com/en-us/dotnet/api/system.iequatable-1](https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1) |
| **Guidelines for Equals and GetHashCode** | [learn.microsoft.com/en-us/dotnet/api/system.object.gethashcode#notes-to-inheritors](https://learn.microsoft.com/en-us/dotnet/api/system.object.gethashcode#notes-to-inheritors) |
| **Equality Operators (==, !=)** | [learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators) |

---

## خلاصه نهایی

> **در Recordها، سفارشی‌سازی Equality از طریق `Equals(R other)` و `GetHashCode()` انجام می‌شود. `Equals(object)` توسط کامپایلر sealed شده و غیرقابل تغییر است. همیشه مطمئن شوید فیلدهای استفاده‌شده در `Equals` دقیقاً همان فیلدهای `GetHashCode` هستند — در غیر این صورت، ساختارهای داده‌ای مبتنی بر Hash به‌شدت شکست می‌خورند.**