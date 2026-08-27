# Deconstruction پیشرفته در C#

**مقاله آموزشی جامع — نسخه ۱.۰**
**تاریخ انتشار:** ۲۷ آگوست ۲۰۲۶
**سطح:** متوسط تا پیشرفته
**پیش‌نیاز:** آشنایی با Tuples، Records و متدهای پایه در C#

---

## 📑 فهرست مطالب

- [مقدمه](#مقدمه)
- [۱. Deconstruction در Assignment](#۱-deconstruction-در-assignment)
- [۲. Deconstruction در foreach](#۲-deconstruction-در-foreach)
- [۳. Deconstruction در متدها](#۳-deconstruction-در-متدها)
- [۴. Deconstruction مستقیم Return Value](#۴-deconstruction-مستقیم-return-value)
- [۵. استفاده از var](#۵-استفاده-از-var)
- [۶. استفاده از Typeهای مشخص](#۶-استفاده-از-typeهای-مشخص)
- [۷. استفاده از Discard](#۷-استفاده-از-discard)
- [۸. استفاده از _ (Underscore)](#۸-استفاده-از-_-underscore)
- [۹. چند Discard پشت سر هم](#۹-چند-discard-پشت-سر-هم)
- [۱۰. Deconstruction در Collectionها](#۱۰-deconstruction-در-collectionها)
- [۱۱. Deconstruction در حلقه‌ها](#۱۱-deconstruction-در-حلقهها)
- [۱۲. Deconstruction در Pattern Matching](#۱۲-deconstruction-در-pattern-matching)
- [۱۳. تفاوت Deconstruction با out](#۱۳-تفاوت-deconstruction-با-out)
- [۱۴. مثال‌های عملی](#۱۴-مثالهای-عملی)
- [نکات مهم](#نکات-مهم)
- [اشتباهات رایج](#اشتباهات-رایج)
- [جمع‌بندی](#جمع‌بندی)
- [منابع رسمی](#منابع-رسمی)

---

## مقدمه

**Deconstruction** (تجزیه) یکی از قابلیت‌های قدرتمند C# است که از نسخه ۷.۰ معرفی شد و در نسخه‌های بعدی گسترش یافت. این قابلیت به شما اجازه می‌دهد یک شیء را به اجزای تشکیل‌دهنده‌اش «باز کنید» و هر جزء را در یک متغیر جداگانه قرار دهید.

به زبان ساده:
> Deconstruction یعنی «باز کردن بسته‌بندی یک شیء و قرار دادن محتویات آن در متغیرهای مستقل».

این قابلیت به‌ویژه در کار با `Tuple`، `Record`، `Dictionary` و هر نوعی که متد `Deconstruct` را پیاده‌سازی کرده باشد، بسیار کاربرد دارد.

```csharp
// مثال ساده
var point = (X: 10, Y: 20);
var (x, y) = point;
Console.WriteLine($"X={x}, Y={y}"); // خروجی: X=10, Y=20
```

---

## ۱. Deconstruction در Assignment

ساده‌ترین شکل Deconstruction، قرار دادن یک Tuple یا شیء دارای متد `Deconstruct` در سمت راست یک انتساب است.

```csharp
(int x, int y) point = (5, 10);
var (a, b) = point;  // Deconstruction در assignment
Console.WriteLine($"{a}, {b}"); // 5, 10
```

**نکته:** در سمت چپ باید تعداد متغیرها با تعداد اجزای شیء تجزیه‌شونده برابر باشد.

```csharp
(string name, int age, string city) person = ("Ali", 30, "Tehran");
var (n, a, c) = person;
```

---

## ۲. Deconstruction در foreach

وقتی یک Collection از Tuple یا Record داشته باشید، می‌توانید در حلقه `foreach` به‌طور مستقیم آن را تجزیه کنید.

```csharp
var people = new List<(string Name, int Age)>
{
    ("Ali", 25),
    ("Sara", 30),
    ("Reza", 28)
};

foreach (var (name, age) in people)
{
    Console.WriteLine($"{name} is {age} years old.");
}
```

خروجی:
```
Ali is 25 years old.
Sara is 30 years old.
Reza is 28 years old.
```

---

## ۳. Deconstruction در متدها

می‌توانید متدی بنویسید که چند مقدار را به‌صورت Tuple برگرداند و در فراخوانی آن را تجزیه کنید.

```csharp
public (int Sum, int Product) Calculate(int a, int b)
{
    return (a + b, a * b);
}

// استفاده
var (sum, product) = Calculate(4, 5);
Console.WriteLine($"Sum={sum}, Product={product}"); // Sum=9, Product=20
```

---

## ۴. Deconstruction مستقیم Return Value

اگر متد شما یک Tuple یا Record برمی‌گرداند، می‌توانید نتیجه را مستقیماً در متغیرهای جداگانه دریافت کنید، بدون آنکه ابتدا یک متغیر میانی بسازید.

```csharp
public (double Latitude, double Longitude) GetLocation(string city)
{
    return city switch
    {
        "Tehran" => (35.6892, 51.3890),
        "Isfahan" => (32.6546, 51.6680),
        _ => (0, 0)
    };
}

// استفاده مستقیم
var (lat, lng) = GetLocation("Tehran");
Console.WriteLine($"Lat: {lat}, Lng: {lng}");
```

---

## ۵. استفاده از var

استفاده از `var` در Deconstruction بسیار رایج است. کامپایلر نوع هر متغیر را از روی نوع جزء مربوطه استنباط می‌کند.

```csharp
var person = (Name: "Mina", Age: 22, City: "Shiraz");

var (name, age, city) = person;
// name: string
// age: int
// city: string
```

> ⚠️ **قانون مهم:** اگر از `var` استفاده می‌کنید، باید برای **همه** متغیرها از `var` استفاده کنید. نمی‌توانید ترکیب کنید:
>
> ```csharp
> // ❌ اشتباه
> (var name, int age, var city) = person;
> ```

---

## ۶. استفاده از Typeهای مشخص

می‌توانید به‌جای `var`، نوع دقیق هر متغیر را مشخص کنید. این کار زمانی مفید است که می‌خواهید تبدیل ضمنی (implicit conversion) انجام شود یا نوع را صریحاً اعلام کنید.

```csharp
var data = (Id: 100L, Name: "Product", Price: 19.99);

// با نوع مشخص
(long id, string name, double price) = data;

// حتی می‌توانید نوع را به نوع سازگار تبدیل کنید
(int idAsInt, string n, decimal p) = data; // long -> int (با warning)
```

مثال با متغیرهای از پیش تعریف‌شده:

```csharp
string name;
int age;

(name, age) = ("Hamed", 40); // انتساب به متغیرهای موجود
```

---

## ۷. استفاده از Discard

گاهی اوقات به برخی از اجزا نیازی ندارید. به‌جای تعریف متغیرهای بی‌استفاده، از **Discard** با نماد `_` استفاده می‌کنید.

```csharp
var person = (Name: "Sara", Age: 30, City: "Mashhad", Job: "Engineer");

// فقط Name و Job را می‌خواهیم
var (name, _, _, job) = person;
Console.WriteLine($"{name} works as {job}");
```

Discard به کامپایلر می‌گوید «این مقدار را دور بریز و حافظه‌ای برای آن اختصاص نده».

---

## ۸. استفاده از _ (Underscore)

نماد `_` همان Discard است. در C# 7.0+ به‌عنوان یک نماد خاص شناخته می‌شود و متغیر واقعی نیست.

```csharp
var result = Divide(10, 3);
var (quotient, _) = result; // remainder را دور می‌اندازیم

public (int Quotient, int Remainder) Divide(int a, int b)
{
    return (a / b, a % b);
}
```

> 💡 `_` در Deconstruction، متغیر نیست؛ یک **Discard Pattern** است و نمی‌توانید بعداً به آن ارجاع دهید.

---

## ۹. چند Discard پشت سر هم

می‌توانید چند `_` پشت سر هم داشته باشید. هر کدام به‌طور مستقل عمل می‌کنند.

```csharp
var point3D = (X: 1, Y: 2, Z: 3, W: 4);

// فقط X را می‌خواهیم
var (x, _, _, _) = point3D;

// یا X و W
var (x2, _, _, w) = point3D;
```

توجه: هر `_` مستقل است و با دیگری تداخل ندارد.

---

## ۱۰. Deconstruction در Collectionها

وقتی Collection شما شامل اشیایی باشد که `Deconstruct` دارند، می‌توانید آن‌ها را تجزیه کنید.

### Dictionary

```csharp
var scores = new Dictionary<string, int>
{
    ["Ali"] = 95,
    ["Sara"] = 88,
    ["Reza"] = 76
};

foreach (var (student, score) in scores)
{
    Console.WriteLine($"{student}: {score}");
}
```

> 💡 `KeyValuePair<TKey, TValue>` از قبل دارای متد `Deconstruct` است.

### List از Record

```csharp
public record Product(string Name, decimal Price, int Stock);

var products = new List<Product>
{
    new("Laptop", 1200, 10),
    new("Mouse", 25, 50),
    new("Keyboard", 80, 30)
};

foreach (var (name, price, stock) in products)
{
    Console.WriteLine($"{name}: ${price} (Stock: {stock})");
}
```

---

## ۱۱. Deconstruction در حلقه‌ها

علاوه بر `foreach`، می‌توانید در حلقه‌های `for` یا LINQ نیز از Deconstruction استفاده کنید.

### با LINQ

```csharp
var users = new List<(string Name, int Age)>
{
    ("Ali", 20), ("Sara", 35), ("Reza", 17), ("Mina", 28)
};

var adults = users
    .Where(u => u.Age >= 18)
    .Select(u =>
    {
        var (name, age) = u;
        return $"{name} ({age})";
    });

foreach (var item in adults)
    Console.WriteLine(item);
```

### در for با Deconstruction

```csharp
var pairs = new List<(int Id, string Value)>
{
    (1, "A"), (2, "B"), (3, "C")
};

for (int i = 0; i < pairs.Count; i++)
{
    var (id, value) = pairs[i];
    Console.WriteLine($"{id} => {value}");
}
```

---

## ۱۲. Deconstruction در Pattern Matching

از C# 8.0 به بعد، Deconstruction با **Pattern Matching** ترکیب می‌شود و بسیار قدرتمند می‌شود.

### Positional Pattern

```csharp
public record Point(int X, int Y);

string Describe(object obj) => obj switch
{
    Point(0, 0) => "Origin",
    Point(var x, 0) => $"On X-axis at {x}",
    Point(0, var y) => $"On Y-axis at {y}",
    Point(var x, var y) when x == y => "On diagonal",
    Point(var x, var y) => $"Point at ({x}, {y})",
    _ => "Unknown"
};

Console.WriteLine(Describe(new Point(5, 5))); // On diagonal
```

### در if با Pattern Matching

```csharp
if (person is (var name, int age, _) and (not null, > 18, _))
{
    Console.WriteLine($"Adult: {name}, {age}");
}
```

### Recursive Pattern

```csharp
public record Employee(string Name, decimal Salary, Department Dept);
public record Department(string Name, string Location);

void Print(Employee emp)
{
    if (emp is (var name, var salary, ("IT", var loc)))
    {
        Console.WriteLine($"{name} in IT @ {loc} earns {salary}");
    }
}
```

---

## ۱۳. تفاوت Deconstruction با out

هر دو برای «برگرداندن چند مقدار» استفاده می‌شوند، اما تفاوت‌های اساسی دارند:

| ویژگی | Deconstruction | out |
|---|---|---|
| نوع خروجی | Tuple یا شیء با `Deconstruct` | پارامترهای جداگانه |
| محل استفاده | Assignment، foreach، pattern | فقط در فراخوانی متد |
| قابلیت استفاده در LINQ | ✅ بله | ❌ خیر |
| Discard | ✅ `_` | ✅ `out _` |
| خوانایی | بالا | متوسط |
| سازگاری با Pattern Matching | ✅ بله | ❌ خیر |

### مقایسه عملی

```csharp
// با out
public bool TryParse(string s, out int value)
{
    return int.TryParse(s, out value);
}

if (TryParse("42", out int result))
    Console.WriteLine(result);

// با Deconstruction (Tuple)
public (bool Success, int Value) ParseSafe(string s)
{
    return int.TryParse(s, out var v) ? (true, v) : (false, 0);
}

var (success, value) = ParseSafe("42");
if (success) Console.WriteLine(value);
```

> 💡 **قانون سرانگشتی:** اگر متد شما باید با الگوی `TryParse` کار کند (مثل APIهای قدیمی)، از `out` استفاده کنید. در غیر این صورت، Tuple + Deconstruction انتخاب مدرن‌تر و تمیزتری است.

---

## ۱۴. مثال‌های عملی

### مثال ۱: پیاده‌سازی Deconstruct سفارشی

```csharp
public class Rectangle
{
    public double Width { get; }
    public double Height { get; }

    public Rectangle(double w, double h) => (Width, Height) = (w, h);

    // متد Deconstruct سفارشی
    public void Deconstruct(out double width, out double height)
        => (width, height) = (Width, Height);

    public double Area() => Width * Height;
}

// استفاده
var rect = new Rectangle(10, 5);
var (w, h) = rect;
Console.WriteLine($"Area = {w * h}"); // 50
```

### مثال ۲: Deconstruct با Overload

```csharp
public class Circle
{
    public double Radius { get; }
    public (double X, double Y) Center { get; }

    public Circle(double r, double x, double y)
    {
        Radius = r;
        Center = (x, y);
    }

    // Overload 1
    public void Deconstruct(out double radius)
        => radius = Radius;

    // Overload 2
    public void Deconstruct(out double radius, out double x, out double y)
        => (radius, x, y) = (Radius, Center.X, Center.Y);
}

var c = new Circle(5, 3, 4);

var (r1) = c;                 // فقط radius
var (r2, x, y) = c;           // همه
Console.WriteLine($"{r2} at ({x},{y})");
```

### مثال ۳: پردازش فایل CSV

```csharp
public record CsvRow(string Name, string Email, int Age);

IEnumerable<CsvRow> ParseCsv(string[] lines)
{
    foreach (var line in lines.Skip(1)) // skip header
    {
        var parts = line.Split(',');
        yield return new CsvRow(parts[0], parts[1], int.Parse(parts[2]));
    }
}

var lines = new[]
{
    "Name,Email,Age",
    "Ali,ali@mail.com,30",
    "Sara,sara@mail.com,25"
};

foreach (var (name, email, age) in ParseCsv(lines))
{
    Console.WriteLine($"{name,-10} | {email,-20} | {age}");
}
```

### مثال ۴: Match روی ساختار تو‌در‌تو

```csharp
public record Address(string City, string Country);
public record User(string Name, Address Address);

string GetUserCountry(User user) => user switch
{
    (_, (_, "Iran")) => "🇮🇷 Iranian User",
    (_, (_, "USA"))  => "🇺🇸 American User",
    (_, (var city, _)) => $"User from {city}",
    _ => "Unknown"
};

var u = new User("Ali", new Address("Tehran", "Iran"));
Console.WriteLine(GetUserCountry(u)); // 🇮🇷 Iranian User
```

---

## نکات مهم

1. ✅ **تطابق تعداد:** تعداد متغیرهای سمت چپ باید با تعداد پارامترهای `out` در متد `Deconstruct` برابر باشد.
2. ✅ **نام متد:** متد تجزیه باید دقیقاً `Deconstruct` نام داشته باشد (case-sensitive).
3. ✅ **Extension Method:** می‌توانید برای کلاسی که به آن دسترسی ندارید، یک `Deconstruct` به‌صورت Extension Method بنویسید.
4. ✅ **Overload مجاز:** می‌توانید چند `Deconstruct` با امضاهای مختلف داشته باشید.
5. ✅ **Records به‌صورت پیش‌فرض:** همه `record`ها متد `Deconstruct` تولیدشده توسط کامپایلر دارند.
6. ✅ **Tupleها:** `System.ValueTuple` به‌طور خودکار از Deconstruction پشتیبانی می‌کند.

### Extension Method برای Deconstruct

```csharp
public static class DateTimeExtensions
{
    public static void Deconstruct(this DateTime dt,
        out int year, out int month, out int day)
        => (year, month, day) = (dt.Year, dt.Month, dt.Day);
}

// استفاده
var (y, m, d) = DateTime.Now;
Console.WriteLine($"{y}/{m}/{d}");
```

---

## اشتباهات رایج

### ❌ ۱. ترکیب var با نوع مشخص

```csharp
// اشتباه
(var name, int age) = person; // ❌ Compile Error

// درست
var (name, age) = person;
// یا
(string name, int age) = person;
```

### ❌ ۲. تعداد نامتناسب متغیرها

```csharp
var point = (1, 2, 3);
var (x, y) = point; // ❌ باید ۳ متغیر باشد
```

### ❌ ۳. استفاده از _ به‌عنوان متغیر واقعی

```csharp
var (name, _) = person;
Console.WriteLine(_); // ❌ _ متغیر نیست و قابل استفاده نیست
```

### ❌ ۴. فراموش کردن out در تعریف Deconstruct

```csharp
// اشتباه
public void Deconstruct(int x, int y) { ... } // ❌

// درست
public void Deconstruct(out int x, out int y) { ... } // ✅
```

### ❌ ۵. انتظار Deconstruction برای کلاس بدون متد

```csharp
public class Person { public string Name { get; set; } }
var p = new Person { Name = "Ali" };
var (name) = p; // ❌ Person متد Deconstruct ندارد
```

راه‌حل: یک `Deconstruct` اضافه کنید یا از Extension Method استفاده کنید.

### ❌ ۶. استفاده از Deconstruct در interface بدون پیاده‌سازی

```csharp
public interface IShape
{
    void Deconstruct(out double area); // ❌ interface نمی‌تواند implementation داشته باشد
}
```

---

## جمع‌بندی

Deconstruction یکی از زیباترین قابلیت‌های مدرن C# است که کد را:

- 📖 **خواناتر** می‌کند
- 🧹 **تمیزتر** می‌سازد
- ⚡ **مختصرتر** می‌نماید
- 🎯 **بی‌خطاتر** می‌کند

**خلاصه کاربردها:**

| کاربرد | مثال |
|---|---|
| Assignment | `var (x, y) = point;` |
| foreach | `foreach (var (k, v) in dict)` |
| Return Value | `var (s, p) = Calc(4,5);` |
| Pattern Matching | `if (obj is Point(var x, var y))` |
| Discard | `var (name, _, _) = person;` |

**توصیه نهایی:** هرگاه با Tuple، Record یا هر ساختاری کار می‌کنید که چند جزء مستقل دارد، به‌جای دسترسی جداگانه به خواص، از Deconstruction استفاده کنید. این کار کد شما را حرفه‌ای‌تر و قابل نگهداری‌تر می‌کند.

---

## منابع رسمی

1. 📘 [Microsoft Docs — Deconstructing tuples and other objects](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct)
2. 📘 [C# Language Reference — Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
3. 📘 [Pattern Matching (C# Documentation)](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)
4. 📘 [Records (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
5. 📘 [Discards (C# Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/discards)
6. 🎥 [What's new in C# 7.0 — Microsoft Channel 9](https://learn.microsoft.com/en-us/shows/csharp-conversations/)

---

**📌 نویسنده:** این مقاله برای Repository آموزشی C# تهیه شده است.
**🔄 آخرین به‌روزرسانی:** آگوست ۲۰۲۶ — منطبق با C# 12 / .NET 8+

اگر این مقاله برایتان مفید بود، با ⭐ Star دادن به Repository از ما حمایت کنید!