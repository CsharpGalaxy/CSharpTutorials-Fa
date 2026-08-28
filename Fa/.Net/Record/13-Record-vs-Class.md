# آموزش کامل Positional Record و Parameter List در C#

این مستند به‌صورت جامع مفاهیم **Parameter List**، **Primary Constructor** و **Positional Record** را در زبان C# بررسی می‌کند. این قابلیت‌ها از نسخه **C# 9.0** (.NET 5) معرفی شده‌اند و برای ساخت انواع داده‌محور (Data-Centric Types) بسیار مفید هستند.

---

## ۱. Parameter List چیست؟

**Parameter List** فهرستی از پارامترهاست که در پرانتز بعد از نام کلاس یا متد قرار می‌گیرد. در کلاس‌های معمولی این پارامترها متعلق به constructor هستند:

```csharp
public class Person
{
    public string FirstName { get; }
    public string LastName { get; }

    public Person(string firstName, string lastName) // Parameter List
    {
        FirstName = firstName;
        LastName = lastName;
    }
}
```

اما در **Record**ها، Parameter List مستقیماً بعد از نام record می‌آید و رفتار متفاوتی دارد.

---

## ۲. Primary Constructor چیست؟

**Primary Constructor** یک constructor اصلی است که در خط تعریف کلاس یا record نوشته می‌شود. در C# 12 این قابلیت به کلاس‌ها و structها نیز اضافه شد، اما در **Record**ها از همان ابتدا (C# 9) وجود داشته است.

```csharp
public class Point(int x, int y)   // Primary Constructor
{
    public int X => x;
    public int Y => y;
}
```

در recordها، primary constructor نقش ویژه‌ای دارد: پارامترهای آن به‌صورت خودکار به property تبدیل می‌شوند.

---

## ۳. Positional Record چیست؟

**Positional Record** شکلی از record است که پارامترهای آن در یک **Parameter List** بعد از نام record نوشته می‌شوند. این پارامترها موقعیت (Position) دارند و ترتیب آن‌ها مهم است.

```csharp
public record Person(string FirstName, string LastName);
```

در اینجا `FirstName` و `LastName` پارامترهای positional هستند.

---

## ۴. ایجاد Property از روی پارامترها

وقتی از Positional Syntax استفاده می‌کنید، کامپایلر برای هر پارامتر یک **public property** با همان نام تولید می‌کند:

```csharp
public record Person(string FirstName, string LastName);
```

معادل است با:

```csharp
public record Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
}
```

---

## ۵. init-only Property

پارامترهای Positional Record به‌صورت پیش‌فرض propertyهایی با **init-only setter** تولید می‌کنند. یعنی فقط در زمان ساخت شیء (در constructor یا object initializer) قابل مقداردهی هستند و بعد از آن فقط **خواندنی** می‌باشند.

```csharp
var p = new Person("Ali", "Rezaei");
p.FirstName = "Reza"; // ❌ Compile Error: Init-only property
```

اما می‌توان با `with` expression یک کپی جدید با مقدار تغییر یافته ساخت:

```csharp
var p2 = p with { FirstName = "Reza" }; // ✅ OK
```

---

## ۶. Constructor تولیدشده

کامپایلر یک constructor با همان signature پارامترها تولید می‌کند:

```csharp
public Person(string FirstName, string LastName)
{
    this.FirstName = FirstName;
    this.LastName = LastName;
}
```

---

## ۷. Deconstructor تولیدشده

کامپایلر یک متد `Deconstruct` نیز تولید می‌کند که امکان **Pattern Matching** و **Tuple Deconstruction** را فراهم می‌کند:

```csharp
var (first, last) = new Person("Ali", "Rezaei");
Console.WriteLine(first); // Ali
```

معادل متد تولیدشده:

```csharp
public void Deconstruct(out string FirstName, out string LastName)
{
    FirstName = this.FirstName;
    LastName = this.LastName;
}
```

---

## ۸. تفاوت Positional Syntax و Property Syntax

| ویژگی | Positional Syntax | Property Syntax |
|-------|-------------------|-----------------|
| تعریف پارامتر | `record P(string A, string B);` | `record P { public string A { get; init; } ... }` |
| Constructor خودکار | ✅ تولید می‌شود | ❌ باید دستی نوشته شود |
| Deconstruct خودکار | ✅ تولید می‌شود | ❌ باید دستی نوشته شود |
| کنترل روی setter | ❌ همیشه init-only | ✅ قابل تنظیم (init, set, readonly) |
| مقداردهی پیش‌فرض | ✅ `record P(string A = "x");` | ✅ در initializer |
| Validation در constructor | ❌ نیاز به body جداگانه | ✅ در constructor دستی |

### مثال Property Syntax:

```csharp
public record Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }

    public Person(string firstName, string lastName)
    {
        if (string.IsNullOrWhiteSpace(firstName))
            throw new ArgumentException(nameof(firstName));
        FirstName = firstName;
        LastName = lastName;
    }

    public void Deconstruct(out string firstName, out string lastName)
    {
        firstName = FirstName;
        lastName = LastName;
    }
}
```

---

## ۹. ترکیب دو روش (Positional + Body)

می‌توانید پارامترها را به‌صورت positional تعریف کنید و در body کارهای اضافی انجام دهید:

```csharp
public record Person(string FirstName, string LastName)
{
    // Validation در initializer
    public string FirstName { get; init; } = 
        string.IsNullOrWhiteSpace(FirstName) 
            ? throw new ArgumentException() 
            : FirstName;

    // Property اضافی
    public string FullName => $"{FirstName} {LastName}";
}
```

---

## ۱۰. Modifierهای مجاز روی پارامترها

### Modifierهای مجاز:

| Modifier | وضعیت | توضیح |
|----------|--------|-------|
| `in` | ✅ مجاز | برای جلوگیری از کپی شدن value typeهای بزرگ |
| `params` | ✅ مجاز | برای آرایه‌های متغیر |
| `ref` | ❌ غیرمجاز | خطای کامپایل |
| `out` | ❌ غیرمجاز | خطای کامپایل |
| `this` | ❌ غیرمجاز | فقط برای extension method |

### استفاده از `in`:

```csharp
public record LargeData(in decimal Value1, in decimal Value2);

var d = new LargeData(10.5m, 20.3m);
```

`in` باعث می‌شود مقدار به‌صورت **read-only reference** پاس داده شود (مناسب برای structهای بزرگ).

### استفاده از `params`:

```csharp
public record Numbers(params int[] Values);

var n = new Numbers(1, 2, 3, 4, 5);
Console.WriteLine(n.Values.Length); // 5
```

---

## ۱۱. محدودیت `ref` و `out`

استفاده از `ref` و `out` در Parameter List یک record **ممنوع** است:

```csharp
public record BadRecord(ref int x);     // ❌ CS8908
public record BadRecord(out int x);     // ❌ CS8908
```

**دلیل:** Recordها باید **immutable** و **value-based** باشند. پارامترهای `ref` و `out` semantics متفاوتی دارند و با مفهوم equality بر اساس مقدار (value equality) سازگار نیستند.

---

## ۱۲. مقایسه کامل دو روش

### روش اول: Positional Syntax

```csharp
public record Person(string FirstName, string LastName);
```

### روش دوم: Property Syntax

```csharp
public record Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
}
```

### تفاوت‌های کلیدی:

| ویژگی | Positional | Property |
|-------|------------|----------|
| تعداد خط کد | ۱ خط | چندین خط |
| Constructor خودکار | ✅ | ❌ |
| Deconstruct خودکار | ✅ | ❌ |
| `PrintMembers` خودکار | ✅ | ❌ |
| `Equals` بر اساس property | ✅ | ✅ |
| `with` expression | ✅ | ✅ |
| امکان validation | محدود | کامل |
| امکان property فقط خواندنی | ❌ | ✅ |

### نکته مهم:

در روش Property Syntax، چون constructor و Deconstruct خودکار تولید نمی‌شوند، باید آن‌ها را دستی بنویسید تا بتوانید از `with` و deconstruction به‌طور کامل استفاده کنید.

---

## ۱۳. Compiler چه چیزهایی برای Positional Record تولید می‌کند؟

برای این تعریف:

```csharp
public record Person(string FirstName, string LastName);
```

کامپایلر موارد زیر را تولید می‌کند:

### ۱. Propertyهای init-only:

```csharp
public string FirstName { get; init; }
public string LastName { get; init; }
```

### ۲. Constructor:

```csharp
public Person(string FirstName, string LastName)
{
    this.FirstName = FirstName;
    this.LastName = LastName;
}
```

### ۳. متد `Deconstruct`:

```csharp
public void Deconstruct(out string FirstName, out string LastName)
{
    FirstName = this.FirstName;
    LastName = this.LastName;
}
```

### ۴. متد `PrintMembers` (برای ToString):

```csharp
protected virtual bool PrintMembers(StringBuilder builder)
{
    builder.Append("FirstName = ");
    builder.Append(FirstName);
    builder.Append(", LastName = ");
    builder.Append(LastName);
    return true;
}
```

### ۵. متد `ToString` (override):

```csharp
public override string ToString()
{
    StringBuilder builder = new StringBuilder();
    builder.Append("Person");
    builder.Append(" { ");
    if (PrintMembers(builder))
        builder.Append(' ');
    builder.Append('}');
    return builder.ToString();
}
```

### ۶. متد `Equals` (value-based equality):

```csharp
public virtual bool Equals(Person? other)
{
    if (other is null) return false;
    if (ReferenceEquals(this, other)) return true;
    if (GetType() != other.GetType()) return false;
    return FirstName == other.FirstName && LastName == other.LastName;
}
```

### ۷. متد `GetHashCode`:

```csharp
public override int GetHashCode()
{
    return HashCode.Combine(FirstName, LastName);
}
```

### ۸. `EqualityContract` (protected):

```csharp
protected virtual Type EqualityContract => typeof(Person);
```

### ۹. متد `Clone` (برای `with` expression):

```csharp
protected virtual Person <Clone>$()
{
    return new Person(this);
}
```

### ۱۰. Copy Constructor (protected):

```csharp
protected Person(Person original)
{
    FirstName = original.FirstName;
    LastName = original.LastName;
}
```

---

## ۱۴. مثال‌های ساده

### مثال ۱: Record ساده

```csharp
public record Point(int X, int Y);

var p1 = new Point(10, 20);
var p2 = new Point(10, 20);

Console.WriteLine(p1 == p2); // True (value equality)
Console.WriteLine(p1);       // Point { X = 10, Y = 20 }
```

### مثال ۲: استفاده از `with`

```csharp
var p1 = new Point(10, 20);
var p2 = p1 with { X = 30 };

Console.WriteLine(p1); // Point { X = 10, Y = 20 }
Console.WriteLine(p2); // Point { X = 30, Y = 20 }
```

### مثال ۳: Deconstruction

```csharp
var point = new Point(5, 10);
var (x, y) = point;
Console.WriteLine($"X={x}, Y={y}"); // X=5, Y=10
```

---

## ۱۵. مثال‌های پیشرفته

### مثال ۱: Record با validation

```csharp
public record Email(string Value)
{
    public string Value { get; init; } = 
        IsValidEmail(Value) ? Value : throw new ArgumentException("Invalid email");

    private static bool IsValidEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

var email = new Email("test@example.com"); // ✅
var badEmail = new Email("invalid");       // ❌ Throws
```

### مثال ۲: Record با property محاسبه‌شده

```csharp
public record Rectangle(double Width, double Height)
{
    public double Area => Width * Height;
    public double Perimeter => 2 * (Width + Height);
}

var rect = new Rectangle(5, 10);
Console.WriteLine(rect.Area);      // 50
Console.WriteLine(rect.Perimeter); // 30
```

### مثال ۳: Record ارث‌بری

```csharp
public record Shape(string Color);
public record Circle(string Color, double Radius) : Shape(Color);
public record Rectangle(string Color, double Width, double Height) : Shape(Color);

Shape[] shapes =
{
    new Circle("Red", 5),
    new Rectangle("Blue", 10, 20)
};

foreach (var shape in shapes)
{
    Console.WriteLine(shape);
}
```

### مثال ۴: Pattern Matching با Record

```csharp
public record Point(int X, int Y);

public string DescribePoint(Point p) => p switch
{
    (0, 0) => "Origin",
    (0, _) => "On Y-axis",
    (_, 0) => "On X-axis",
    (var x, var y) when x == y => "On diagonal",
    _ => "Somewhere else"
};

Console.WriteLine(DescribePoint(new Point(0, 0))); // Origin
Console.WriteLine(DescribePoint(new Point(5, 5))); // On diagonal
```

### مثال ۵: Record با `params`

```csharp
public record Tags(params string[] Values);

var tags = new Tags("csharp", "dotnet", "record");
Console.WriteLine(string.Join(", ", tags.Values));
// csharp, dotnet, record
```

### مثال ۶: Record با `in` modifier

```csharp
public record Matrix(in decimal[,] Data)
{
    public int Rows => Data.GetLength(0);
    public int Cols => Data.GetLength(1);
}

var data = new decimal[,] { { 1, 2 }, { 3, 4 } };
var matrix = new Matrix(in data);
Console.WriteLine($"{matrix.Rows}x{matrix.Cols}"); // 2x2
```

### مثال ۷: Positional Record با مقدار پیش‌فرض

```csharp
public record Config(string Host, int Port = 8080, bool UseSsl = true);

var config1 = new Config("localhost");
var config2 = new Config("api.example.com", 443, false);

Console.WriteLine(config1); // Config { Host = localhost, Port = 8080, UseSsl = True }
Console.WriteLine(config2); // Config { Host = api.example.com, Port = 443, UseSsl = False }
```

### مثال ۸: ترکیب Positional و Property

```csharp
public record User(string Username, string Email)
{
    // Property اضافی
    public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
    
    // Property محاسبه‌شده
    public string DisplayName => Username.Split('@')[0];
    
    // متد سفارشی
    public bool IsGmail => Email.EndsWith("@gmail.com");
}

var user = new User("john@gmail.com", "john@gmail.com");
Console.WriteLine(user.DisplayName); // john
Console.WriteLine(user.IsGmail);     // True
```

---

## ۱۶. نکات مهم و Best Practices

### ✅ توصیه‌ها:

1. **برای DTOها و Data Transfer Objects** از Positional Record استفاده کنید.
2. **برای Domain Models** که نیاز به validation دارند، از Property Syntax یا ترکیب هر دو استفاده کنید.
3. **از `with` expression** برای ایجاد تغییرات immutable استفاده کنید.
4. **از Pattern Matching** برای کار با recordها بهره ببرید.
5. **Recordها را برای انواع Immutable** طراحی کنید.

### ❌ اشتباهات رایج:

1. استفاده از `ref` یا `out` در parameter list (خطای کامپایل).
2. انتظار mutable property از positional record (همه init-only هستند).
3. فراموش کردن نوشتن constructor دستی در Property Syntax.
4. استفاده از record برای انواعی که نیاز به reference equality دارند (از class معمولی استفاده کنید).

---

## ۱۷. منابع رسمی

برای مطالعه بیشتر و جزئیات دقیق، به منابع رسمی مایکروسافت مراجعه کنید:

### مستندات رسمی Microsoft Learn:

1. **Records (C# Reference)**
   - [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)

2. **Record Types (C# Programming Guide)**
   - [https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)

3. **Primary Constructors (C# Programming Guide)**
   - [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/primary-constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/primary-constructors)

4. **What's New in C# 9.0**
   - [https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9)

5. **What's New in C# 12.0**
   - [https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12)

### مشخصات زبان (Language Specification):

6. **C# Language Specification - Records**
   - [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-record-types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-record-types)

### مقالات آموزشی:

7. **Record Types in C# (Microsoft Docs)**
   - [https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/records](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/records)

---

## خلاصه

| مفهوم | توضیح |
|-------|-------|
| **Parameter List** | فهرست پارامترها در تعریف type |
| **Primary Constructor** | Constructor اصلی در خط تعریف |
| **Positional Record** | Record با پارامترهای موقعیتی |
| **init-only Property** | Property فقط قابل مقداردهی در ساخت |
| **Deconstruct** | متد تولیدشده برای tuple deconstruction |
| **Value Equality** | مقایسه بر اساس مقدار، نه reference |

Positional Recordها یکی از قدرتمندترین ویژگی‌های مدرن C# برای ساخت انواع داده‌محور، immutable و با value equality هستند. با درک صحیح از Parameter List و Primary Constructor، می‌توانید کدهای تمیزتر، کوتاه‌تر و قابل‌نگهداری‌تری بنویسید.