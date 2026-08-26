# نوع ناشناس (Anonymous Type) در C#

این راهنما مفهوم **نوع ناشناس** را در C# از سطح مقدماتی تا پیشرفته توضیح می‌دهد. ابتدا می‌بینیم نوع ناشناس چیست و چگونه ساخته می‌شود؛ سپس رفتار کامپایلر، کاربرد آن در LINQ، محدودیت‌ها و زمان مناسب برای استفاده از `class` یا `record` را بررسی می‌کنیم.

> مثال‌های این راهنما برای برنامه‌های C# مدرن نوشته شده‌اند. برای اجرای مثال‌های LINQ، فضای نام `System.Linq` را اضافه کنید.

## فهرست مطالب

- [نوع چیست؟](#what-is-a-type)
- [نوع ناشناس چیست؟](#definition)
- [اولین مثال](#first-example)
- [ساخت نوع ناشناس](#creating)
  - [نام‌گذاری صریح ویژگی‌ها](#explicit-property-names)
  - [استنباط نام ویژگی‌ها](#inferred-property-names)
  - [نوع ویژگی‌ها](#property-types)
  - [ویژگی‌های تو‌در‌تو](#nested-anonymous-types)
  - [آرایه‌ای از نوع‌های ناشناس](#anonymous-type-array)
- [کامپایلر چه کاری انجام می‌دهد؟](#compiler)
  - [تولید نوع در زمان کامپایل](#compile-time)
  - [مدل مفهومی نوع تولیدشده](#conceptual-model)
  - [چرا `var` استفاده می‌کنیم؟](#why-var)
- [ویژگی‌های مهم](#characteristics)
  - [فقط‌خواندنی بودن ویژگی‌ها](#read-only)
  - [نوع مرجع بودن](#reference-type)
  - [برابری مقدارمحور](#value-based-equality)
  - [متد `ToString`](#tostring)
  - [تغییرناپذیری سطحی](#shallow-immutability)
- [کاربرد در LINQ](#linq)
  - [Projection چیست؟](#projection)
  - [متد `Select`](#select)
  - [Query Syntax](#query-syntax)
  - [LINQ to Objects و Providerها](#linq-providers)
- [محدودیت‌ها و Scope](#limitations)
  - [نبودن نام قابل استفاده](#no-name)
  - [عدم استفاده در قرارداد متد](#method-contract)
  - [مرز لایه و اسمبلی](#assembly-boundary)
- [نوع ناشناس، `object` و `dynamic`](#object-dynamic)
- [عبارت `with`](#with)
- [مقایسه با `class`، `struct`، `record` و tuple](#comparison)
- [Best Practices](#best-practices)
- [اشتباهات رایج](#common-mistakes)
- [تمرین‌ها](#exercises)
- [جمع‌بندی](#summary)
- [منابع معتبر](#references)

<a id="what-is-a-type"></a>
## نوع چیست؟

در C# هر مقدار یک **نوع (Type)** دارد. نوع مشخص می‌کند:

- مقدار چه داده‌ای را نگه می‌دارد؛
- چه عملیاتی روی آن مجاز است؛
- چه اعضایی مانند property، field و method دارد.

```csharp
int age = 20;
string name = "Sara";
bool isActive = true;
```

وقتی به یک مدل در بخش‌های مختلف برنامه نیاز داریم، معمولاً نوعی نام‌گذاری‌شده تعریف می‌کنیم:

```csharp
public class Product
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}
```

اما گاهی چند مقدار مرتبط فقط در یک بخش کوتاه از برنامه لازم هستند. در چنین شرایطی ساختن یک کلاس جداگانه ممکن است بیش از نیاز باشد. **نوع ناشناس** برای همین داده‌های کوتاه‌عمر طراحی شده است.

<a id="definition"></a>
## نوع ناشناس چیست؟

نوع ناشناس (`Anonymous Type`) نوعی است که نام آن را در کد منبع نمی‌نویسیم و کامپایلر آن را از روی ویژگی‌هایی که در عبارت `new { ... }` قرار داده‌ایم تولید می‌کند.

به بیان ساده:

```csharp
var person = new
{
    Name = "Sara",
    Age = 28
};
```

در این مثال ما کلاسی به نامی مثل `Person` تعریف نکرده‌ایم، اما شیء ساخته‌شده همچنان یک **نوع واقعی** دارد. «ناشناس» یعنی نام نوع در اختیار برنامه‌نویس نیست، نه اینکه شیء بدون نوع یا کاملاً پویا باشد.

نکات کلیدی:

1. نوع ناشناس توسط کامپایلر تولید می‌شود.
2. تولید و تشخیص آن در زمان کامپایل انجام می‌شود.
3. ویژگی‌های نوع ناشناس فقط‌خواندنی هستند.
4. کاربرد اصلی آن نگهداری موقت داده و پروژه‌کردن نتیجه‌ی LINQ است.
5. معمولاً برای API عمومی، DTO یا داده‌ای که بین لایه‌ها جابه‌جا می‌شود مناسب نیست.

<a id="first-example"></a>
## اولین مثال

```csharp
var person = new
{
    Name = "Sara",
    Age = 28
};

Console.WriteLine(person.Name); // Sara
Console.WriteLine(person.Age);  // 28
```

اجزای مثال:

| بخش | توضیح |
|---|---|
| `new` | ساخت یک نمونه‌ی جدید |
| `{ ... }` | فهرست ویژگی‌های نوع ناشناس |
| `Name = "Sara"` | ویژگی‌ای از نوع `string` |
| `Age = 28` | ویژگی‌ای از نوع `int` |
| `var` | درخواست استنباط نوع از کامپایلر |

ساختار کلی:

```csharp
var variable = new
{
    Property1 = value1,
    Property2 = value2
};
```

<a id="creating"></a>
## ساخت نوع ناشناس

برای ساخت نوع ناشناس از `new` و initializer استفاده می‌کنیم:

```csharp
var book = new
{
    Title = "Clean Code",
    PageCount = 464,
    IsAvailable = true
};
```

نوع هر ویژگی از مقدار سمت راست آن استنباط می‌شود:

| ویژگی | مقدار | نوع |
|---|---|---|
| `Title` | `"Clean Code"` | `string` |
| `PageCount` | `464` | `int` |
| `IsAvailable` | `true` | `bool` |

مقدار ویژگی می‌تواند حاصل یک محاسبه یا فراخوانی متد باشد:

```csharp
int unitPrice = 100;
int count = 3;

var summary = new
{
    UnitPrice = unitPrice,
    TotalPrice = unitPrice * count,
    IsLargeOrder = count >= 3
};
```

<a id="explicit-property-names"></a>
### نام‌گذاری صریح ویژگی‌ها

در شکل کامل، نام ویژگی را خودمان مشخص می‌کنیم:

```csharp
var user = new
{
    UserName = "ali",
    LoginCount = 5
};

Console.WriteLine(user.UserName);
```

نام ویژگی باید یک نام معتبر در C# باشد و نمی‌تواند تکراری باشد:

```csharp
// خطا: دو ویژگی نام یکسان دارند.
var invalid = new
{
    Name = "Ali",
    // Name = "Reza"
};
```

<a id="inferred-property-names"></a>
### استنباط نام ویژگی‌ها

اگر مقدار از یک متغیر یا member access بیاید، می‌توانیم نام ویژگی را ننویسیم:

```csharp
string title = "C#";
int pages = 200;

var book = new { title, pages };

Console.WriteLine(book.title);
Console.WriteLine(book.pages);
```

در این حالت کامپایلر نام‌های `title` و `pages` را از همان عبارت‌ها برداشت می‌کند. این قابلیت **Projection Initializer** نام دارد.

برای تغییر نام، از شکل صریح استفاده کنید:

```csharp
string firstName = "Sara";
string lastName = "Ahmadi";

var person = new
{
    FirstName = firstName,
    LastName = lastName
};
```

<a id="property-types"></a>
### نوع ویژگی‌ها

نوع ناشناس می‌تواند ویژگی‌هایی از انواع مختلف داشته باشد:

```csharp
DateTime createdAt = DateTime.UtcNow;
decimal price = 125.50m;
string? description = null;

var product = new
{
    Id = 10,
    Name = "Keyboard",
    Price = price,
    CreatedAt = createdAt,
    Description = description
};
```

برای مقدار `null` خالی، کامپایلر نوعی برای استنباط ندارد. نوع را صریح مشخص کنید:

```csharp
var item = new
{
    Description = (string?)null
};
```

<a id="nested-anonymous-types"></a>
### ویژگی‌های تو‌در‌تو

مقدار یک ویژگی می‌تواند خودش نوع ناشناس دیگری باشد:

```csharp
var order = new
{
    Id = 101,
    Customer = new
    {
        Name = "Reza",
        Email = "reza@example.com"
    },
    Address = new
    {
        City = "Tehran",
        PostalCode = "1234567890"
    }
};

Console.WriteLine(order.Customer.Name);
Console.WriteLine(order.Address.City);
```

تودرتوکردن بیش از حد معمولاً خوانایی را کم می‌کند. اگر ساختار داده بزرگ یا قابل استفاده‌ی مجدد است، یک مدل نام‌گذاری‌شده تعریف کنید.

<a id="anonymous-type-array"></a>
### آرایه‌ای از نوع‌های ناشناس

برای ساخت آرایه، شکل و نوع ویژگی‌های تمام اعضا باید سازگار باشد:

```csharp
var products = new[]
{
    new { Id = 1, Name = "Keyboard", Price = 50m },
    new { Id = 2, Name = "Mouse",    Price = 25m }
};

foreach (var product in products)
{
    Console.WriteLine($"{product.Name}: {product.Price}");
}
```

این کد خطا می‌دهد، چون دو عضو شکل یکسانی ندارند:

```csharp
var invalid = new[]
{
    new { Id = 1, Name = "Keyboard" },
    new { Id = 2, Title = "Mouse" }
};
```

همچنین بهتر است نوع ناشناس همه‌ی اعضای آرایه دقیقاً با یک ترتیب و نوع یکسان نوشته شود:

```csharp
var valid = new[]
{
    new { Id = 1, Name = "A" },
    new { Id = 2, Name = "B" }
};

<a id="compiler"></a>
## کامپایلر چه کاری انجام می‌دهد؟

کد زیر را در نظر بگیرید:

```csharp
var product = new
{
    Id = 10,
    Name = "Monitor"
};
```

کامپایلر هنگام کامپایل، شکل این initializer را بررسی می‌کند و نوع مناسب را تولید می‌کند. این اتفاق در زمان اجرای برنامه و به‌صورت تصمیم‌گیری ناشناخته رخ نمی‌دهد.

<a id="compile-time"></a>
### تولید نوع در زمان کامپایل

کامپایلر به‌صورت کلی این مراحل را انجام می‌دهد:

1. تعداد و نام ویژگی‌ها را بررسی می‌کند.
2. نوع هر عبارت را استنباط می‌کند.
3. یک نوع ناشناس برای این شکل از ویژگی‌ها تولید می‌کند.
4. کدی برای ساخت نمونه و مقداردهی ویژگی‌ها تولید می‌کند.
5. دسترسی‌هایی مانند `product.Name` را در زمان کامپایل بررسی می‌کند.

پس نوع ناشناس با `dynamic` تفاوت اساسی دارد:

| مفهوم | زمان بررسی اعضا | توضیح |
|---|---|---|
| نوع ناشناس | زمان کامپایل | نوع واقعی را کامپایلر تولید می‌کند |
| `object` | نوع عمومی در کامپایل | برای دسترسی اختصاصی معمولاً به cast نیاز دارد |
| `dynamic` | زمان اجرا | خطاهای نامعتبر ممکن است تا runtime دیده نشوند |

<a id="conceptual-model"></a>
### مدل مفهومی نوع تولیدشده

کد تولیدشده دقیقاً با کد زیر یکی نیست، اما می‌توان آن را مفهومی شبیه این دانست:

```csharp
// نمایش مفهومی؛ این کلاس را خودمان تعریف نمی‌کنیم.
internal sealed class GeneratedAnonymousType
{
    public int Id { get; }
    public string Name { get; }

    public GeneratedAnonymousType(int id, string name)
    {
        Id = id;
        Name = name;
    }
}
```

نوع تولیدشده معمولاً:

- یک کلاس `internal` و `sealed` است؛
- مستقیماً از `object` ارث می‌برد؛
- propertyهای عمومی و فقط‌خواندنی دارد؛
- برای `Equals`، `GetHashCode` و `ToString` پیاده‌سازی تولیدشده دارد.

نام واقعی این نوع توسط کامپایلر ساخته می‌شود و قرارداد قابل اتکای برنامه نیست. بنابراین نباید بر اساس نام metadata آن منطق بنویسیم.

<a id="why-var"></a>
### چرا معمولاً از `var` استفاده می‌کنیم؟

نام نوع ناشناس در کد منبع قابل نوشتن نیست؛ به همین دلیل برای متغیر محلی از `var` استفاده می‌کنیم:

```csharp
var person = new { Name = "Sara", Age = 28 };
```

`var` به معنی «بدون نوع» یا «dynamic» نیست. نوع `person` در زمان کامپایل مشخص است و این دسترسی معتبر و type-safe است:

```csharp
Console.WriteLine(person.Name);
```

اما این دسترسی خطای کامپایل دارد:

```csharp
// Console.WriteLine(person.Unknown);
```

`var` باید مقدار اولیه داشته باشد:

```csharp
// var value; // خطا: نوع از روی هیچ مقداری قابل استنباط نیست
```

<a id="characteristics"></a>
## ویژگی‌های مهم

<a id="read-only"></a>
### فقط‌خواندنی بودن ویژگی‌ها

propertyهای نوع ناشناس setter عمومی ندارند:

```csharp
var item = new
{
    Name = "Pen",
    Price = 2m
};

// item.Price = 3m; // خطای کامپایل
```

این ویژگی نوع ناشناس را برای داده‌هایی مناسب می‌کند که بعد از ساخت نباید دوباره مقداردهی شوند.

اگر تغییر مقدار لازم است، نوع نام‌گذاری‌شده تعریف کنید:

```csharp
public class CartItem
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

var item = new CartItem
{
    Name = "Pen",
    Price = 2m
};

item.Price = 3m;
```

<a id="reference-type"></a>
### نوع مرجع بودن

نوع ناشناس از جنس class است، نه struct؛ بنابراین یک reference type محسوب می‌شود:

```csharp
var first = new { Name = "Sara" };
object value = first;

Console.WriteLine(value.GetType().IsClass); // True
```

تبدیل نمونه به `object` ممکن است، اما بعد از آن دسترسی مستقیم به propertyها را از دست می‌دهیم؛ چون نوع استاتیک متغیر `object` است.

<a id="value-based-equality"></a>
### برابری مقدارمحور

دو نمونه از یک شکل نوع ناشناس، در صورتی که مقدار ویژگی‌هایشان برابر باشد، برابر محسوب می‌شوند:

```csharp
var first = new { Id = 1, Name = "Book" };
var second = new { Id = 1, Name = "Book" };
var third = new { Id = 2, Name = "Book" };

Console.WriteLine(first.Equals(second)); // True
Console.WriteLine(first.Equals(third));  // False
```

برای اینکه دو initializer از یک نوع ناشناس مشترک استفاده کنند، نام، نوع کامپایل‌شده و ترتیب ویژگی‌ها باید یکسان باشد:

```csharp
var a = new { Id = 1, Name = "Book" };
var b = new { Id = 2, Name = "Pen" };

a = b; // معتبر: شکل هر دو یکسان است
```

تغییر ترتیب، شکل نوع را تغییر می‌دهد:

```csharp
var x = new { Id = 1, Name = "Book" };
var y = new { Name = "Book", Id = 1 };

Console.WriteLine(x.Equals(y)); // False
```

نکته: مقایسه‌ی مقدار ویژگی‌ها از `Equals` همان ویژگی‌ها استفاده می‌کند. برای مثال، اگر یک property به یک لیست قابل تغییر اشاره کند، رفتار برابری آن لیست نیز مهم است.

<a id="tostring"></a>
### متد `ToString`

نوع ناشناس نمایش متنی مفیدی برای مشاهده‌ی سریع مقدارها دارد:

```csharp
var product = new { Id = 10, Name = "Monitor", Price = 250m };

Console.WriteLine(product);
// نمونه خروجی: { Id = 10, Name = Monitor, Price = 250 }
```

این خروجی برای debug مناسب است، اما قرارداد پایدار برای log یا API محسوب نمی‌شود.

<a id="shallow-immutability"></a>
### تغییرناپذیری سطحی

فقط خود property قابل جایگزینی نیست؛ این به معنی تغییرناپذیری عمیق همه‌ی اشیای داخلی نیست:

```csharp
var data = new
{
    Items = new List<int> { 1, 2 }
};

// data.Items = new List<int>(); // خطا: property قابل انتساب نیست
data.Items.Add(3);                // معتبر: خود List قابل تغییر است
```

اگر تغییرناپذیری کامل می‌خواهید، باید نوع‌های داخلی را نیز تغییرناپذیر انتخاب کنید.

<a id="linq"></a>
## کاربرد در LINQ

یکی از رایج‌ترین کاربردهای نوع ناشناس، ساخت شکل جدیدی از داده در LINQ است. وقتی فقط بخشی از propertyهای یک مجموعه را لازم داریم، می‌توانیم در `Select` نوع ناشناس بسازیم.

<a id="projection"></a>
### Projection چیست؟

**Projection** یعنی تبدیل هر عضو به شکل یا مدل دیگری. برای نمونه، از یک دانش‌آموز با چندین ویژگی، فقط نام و نمره را انتخاب می‌کنیم:

```csharp
var students = new[]
{
    new { Id = 1, Name = "Sara", Score = 19 },
    new { Id = 2, Name = "Ali",  Score = 15 },
    new { Id = 3, Name = "Mina", Score = 18 }
};

var result = students.Select(student => new
{
    student.Name,
    student.Score
});
```

نوع نتیجه برای ما نام قابل نوشتنی ندارد، اما کامپایلر آن را می‌شناسد؛ بنابراین با `var` و `foreach` به‌درستی کار می‌کنیم:

```csharp
foreach (var student in result)
{
    Console.WriteLine($"{student.Name}: {student.Score}");
}
```

<a id="select"></a>
### متد `Select`

با `Select` می‌توانیم propertyهای جدید نیز محاسبه کنیم:

```csharp
var successfulStudents = students
    .Where(student => student.Score >= 18)
    .Select(student => new
    {
        student.Id,
        student.Name,
        Grade = student.Score >= 19
            ? "Excellent"
            : "Very Good"
    });

foreach (var student in successfulStudents)
{
    Console.WriteLine($"{student.Id} - {student.Name} - {student.Grade}");
}
```

مثال دیگری با سفارش‌ها:

```csharp
var orders = new[]
{
    new { Id = 1001, Customer = "Sara", Total = 1200m },
    new { Id = 1002, Customer = "Ali",  Total = 350m },
    new { Id = 1003, Customer = "Mina", Total = 890m }
};

var expensiveOrders = orders
    .Where(order => order.Total >= 800m)
    .Select(order => new
    {
        OrderId = order.Id,
        order.Customer,
        order.Total,
        HasDiscount = order.Total >= 1000m
    });
```

نوع ناشناس در اینجا ارزش دارد چون نتیجه فقط در همین جریان پردازش مصرف می‌شود و نیازی به ساخت مدل عمومی جداگانه نداریم.

<a id="query-syntax"></a>
### Query Syntax

همین کار را با syntax کوئری LINQ نیز می‌توان انجام داد:

```csharp
var result =
    from student in students
    where student.Score >= 18
    select new
    {
        student.Name,
        student.Score
    };
```

<a id="linq-providers"></a>
### LINQ to Objects و Providerها

در LINQ to Objects، عملیات روی داده‌های موجود در حافظه انجام می‌شود:

```csharp
var names = students
    .Where(student => student.Score >= 18)
    .Select(student => new { student.Name });
```

در ORMها یا providerهای دیگر، عبارت `Select` ممکن است به یک query منبع مانند SQL ترجمه شود:

```csharp
var query = dbContext.Products
    .Where(product => product.Price > 100)
    .Select(product => new
    {
        product.Id,
        product.Name
    });
```

انتخاب فقط propertyهای لازم می‌تواند داده‌ی کمتری از منبع بخواند؛ بااین‌حال قابلیت ترجمه و محدودیت‌ها به provider مورد استفاده بستگی دارد.

<a id="limitations"></a>
## محدودیت‌ها و Scope

<a id="no-name"></a>
### نام نوع در کد منبع در دسترس نیست

کامپایلر برای نوع ناشناس نامی تولید می‌کند، اما آن نام را نمی‌توان در کد خودمان بنویسیم:

```csharp
// نوعی مثل GeneratedAnonymousType در کد ما وجود ندارد.
// GeneratedAnonymousType value = new { Name = "Sara" };
```

نام‌هایی که ممکن است در reflection یا decompiler دیده شوند، جزئیات تولید کامپایلر هستند و نباید بخشی از قرارداد برنامه شوند.

<a id="method-contract"></a>
### عدم استفاده در قرارداد متد

چون نوع ناشناس نام قابل استفاده ندارد، نمی‌توان آن را به شکل مستقیم در return type، parameter یا field type نوشت:

```csharp
// قابل نوشتن نیست:
// public ??? CreateStudent() { ... }
// public void Print(??? student) { ... }
// private ??? cachedValue;
```

بازگرداندن آن به‌صورت `object` از نظر فنی ممکن است، اما مصرف‌کننده دسترسی type-safe به propertyها ندارد:

```csharp
public static object CreateData()
{
    return new
    {
        Name = "Sara",
        Score = 19
    };
}

object data = CreateData();
// data.Name; // خطا، چون نوع استاتیک data برابر object است
```

اگر داده از یک متد خارج می‌شود، معمولاً بهتر است مدل نام‌گذاری‌شده تعریف کنیم:

```csharp
public record StudentResult(string Name, int Score);

public static StudentResult CreateResult()
{
    return new("Sara", 19);
}
```

<a id="assembly-boundary"></a>
### مرز لایه و اسمبلی

نوع ناشناس برای استفاده‌ی محلی مناسب است. حتی اگر دو initializer در یک برنامه شکل یکسانی داشته باشند، نباید این ویژگی را به‌عنوان قرارداد بین پروژه‌ها یا اسمبلی‌ها طراحی کنیم.

برای داده‌ای که بین این بخش‌ها جابه‌جا می‌شود، از یک نوع عمومی و نام‌گذاری‌شده استفاده کنید:

```csharp
public sealed class ProductSummary
{
    public int Id { get; init; }
    public string Name { get; init; } = "";
}
```

این کار مستندسازی، refactoring، اعتبارسنجی و تغییرات آینده را ساده‌تر می‌کند.

<a id="object-dynamic"></a>
## نوع ناشناس، `object` و `dynamic`

### نوع ناشناس و `object`

می‌توان نمونه‌ی نوع ناشناس را در `object` قرار داد:

```csharp
object value = new
{
    Name = "Sara",
    Age = 28
};
```

اما نوع استاتیک `value` برابر `object` است:

```csharp
// Console.WriteLine(value.Name); // خطای کامپایل
```

با reflection می‌توان propertyها را خواند، اما این راه‌حل verbose و شکننده است:

```csharp
object value = new { Name = "Sara" };

var property = value.GetType().GetProperty("Name");
var name = property?.GetValue(value);

Console.WriteLine(name);
```

اگر برای دسترسی به propertyها دائماً به reflection نیاز دارید، احتمالاً نوع ناشناس انتخاب مناسبی نیست.

### نوع ناشناس و `dynamic`

با `dynamic` بررسی دسترسی به زمان اجرا منتقل می‌شود:

```csharp
dynamic value = new
{
    Name = "Sara"
};

Console.WriteLine(value.Name); // معتبر
```

اما این کد ممکن است کامپایل شود و در زمان اجرا خطا بدهد:

```csharp
// در runtime خطای Microsoft.CSharp.RuntimeBinder.RuntimeBinderException
// Console.WriteLine(value.Age);
```

`dynamic` جایگزین عمومی و type-safe برای نوع ناشناس نیست؛ فقط در سناریوهایی استفاده کنید که رفتار پویا واقعاً لازم است.

<a id="with"></a>
## عبارت `with`

در نسخه‌های جدید C#، نوع‌های ناشناس از عبارت `with` برای ساخت یک نمونه‌ی جدید با چند مقدار متفاوت پشتیبانی می‌کنند:

```csharp
var original = new
{
    Name = "Sara",
    Score = 18
};

var updated = original with
{
    Score = 20
};

Console.WriteLine(original.Score); // 18
Console.WriteLine(updated.Score);  // 20
```

`with` نمونه‌ی قبلی را تغییر نمی‌دهد و یک نمونه‌ی جدید می‌سازد. این کپی **سطحی (shallow)** است؛ اگر property به یک شیء قابل تغییر اشاره کند، مرجع داخلی ممکن است بین دو نمونه مشترک باشد.

اگر از `with` به‌صورت گسترده استفاده می‌کنید یا نمونه‌ها بین لایه‌ها جابه‌جا می‌شوند، `record` معمولاً قصد طراحی را بهتر نشان می‌دهد.

<a id="comparison"></a>
## مقایسه با `class`، `struct`، `record` و tuple

### مقایسه سریع

| ویژگی | Anonymous Type | `class` | `struct` / `ValueTuple` | `record` |
|---|---|---|---|---|
| نام قابل استفاده در کد | ندارد | دارد | دارد | دارد |
| نوع داده | reference type | reference type | value type | class یا struct |
| برابری مقدارمحور | بله | باید خودمان طراحی کنیم | معمولاً بله | بله |
| ویژگی‌های قابل تغییر | propertyها نه | با setter یا `init` | بسته به تعریف | بسته به تعریف |
| مناسب برای API عمومی | معمولاً نه | بله | گاهی | بله |
| deconstruction | ندارد | به‌صورت پیش‌فرض ندارد | بله | با الگوی مناسب |
| کاربرد رایج | داده موقت و LINQ | مدل و رفتار عمومی | چند مقدار کوتاه | DTO و مدل داده |

### نوع ناشناس در برابر `class`

نوع ناشناس:

```csharp
var product = new
{
    Name = "Keyboard",
    Price = 50m
};
```

کلاس نام‌گذاری‌شده:

```csharp
public class Product
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

var product = new Product
{
    Name = "Keyboard",
    Price = 50m
};
```

`class` زمانی انتخاب بهتری است که مدل در چند بخش برنامه استفاده می‌شود، متد یا اعتبارسنجی دارد، یا باید در public API قرار بگیرد.

### نوع ناشناس در برابر `record`

برای داده‌های مدل‌محور و قراردادهای مشخص، `record` اغلب خواناتر است:

```csharp
public record ProductSummary(string Name, decimal Price);

var product = new ProductSummary("Keyboard", 50m);
var discounted = product with { Price = 40m };
```

### نوع ناشناس در برابر tuple

```csharp
var anonymous = new
{
    Name = "Sara",
    Age = 28
};

var tuple = (Name: "Sara", Age: 28);
```

tuple (`ValueTuple`) برای چند مقدار ساده و کوتاه، مخصوصاً وقتی deconstruction لازم است، گزینه‌ی خوبی است:

```csharp
var person = (Name: "Sara", Age: 28);
var (name, age) = person;

Console.WriteLine($"{name} is {age}");
```

نوع ناشناس در سناریوهایی مثل projectionهای propertyمحور یا expression tree می‌تواند مناسب‌تر باشد. برای انتخاب بین این دو، علاوه بر خوانایی، نیاز به deconstruction، reference/value semantics و serialization را در نظر بگیرید.

<a id="best-practices"></a>
## Best Practices

### ۱. داده را تا حد امکان محلی نگه دارید

نوع ناشناس را داخل همان متد یا جریان LINQ مصرف کنید:

```csharp
var displayRows = products
    .Where(product => product.Price > 100)
    .Select(product => new
    {
        product.Name,
        PriceText = $"{product.Price:N0}"
    });

foreach (var row in displayRows)
{
    Console.WriteLine($"{row.Name}: {row.PriceText}");
}
```

### ۲. برای قرارداد، نام مشخص انتخاب کنید

اگر داده قرار است از متد برگردد، در چند پروژه استفاده شود، serialize شود یا بخشی از API باشد، `class` یا `record` تعریف کنید.

### ۳. از نام‌های روشن برای propertyها استفاده کنید

این کد خواناتر است:

```csharp
var result = new
{
    TotalPrice = price * quantity,
    IsAvailable = stock > 0
};
```

در برابر نام‌های مبهم:

```csharp
var result = new
{
    x = price * quantity,
    y = stock > 0
};
```

### ۴. تودرتویی را محدود کنید

اگر خواندن عبارت‌هایی مانند `data.A.B.C` به الگوی دائمی تبدیل شد، مدل را نام‌گذاری کنید.

### ۵. برای serialization محتاط باشید

برای JSON، ذخیره‌سازی و پیام بین سرویس‌ها، قرارداد صریح و پایدار بسازید. نوع ناشناس به‌دلیل نداشتن نام قابل استفاده، انتخاب مناسبی برای قراردادهای بلندمدت نیست.

### ۶. `var` را فقط وقتی استفاده کنید که نوع از کد واضح باشد

برای نوع ناشناس ناچاریم از `var` استفاده کنیم، اما در سایر کدها خوانایی را معیار قرار دهید:

```csharp
var person = new { Name = "Sara", Age = 28 }; // نوع از initializer واضح است
```

<a id="common-mistakes"></a>
## اشتباهات رایج

### اشتباه ۱: تصور اینکه `var` همان `dynamic` است

```csharp
var person = new { Name = "Ali" };

// person.Unknown; // خطای کامپایل
```

در `var`، نوع در زمان کامپایل مشخص است. در `dynamic`، بررسی عضو به زمان اجرا منتقل می‌شود.

### اشتباه ۲: تلاش برای تغییر property

```csharp
var config = new { RetryCount = 3 };

// config.RetryCount = 5; // خطای کامپایل
```

### اشتباه ۳: استفاده در return type یا parameter

اگر داده از مرز متد عبور می‌کند، نوع نام‌گذاری‌شده تعریف کنید:

```csharp
public record UserSummary(string UserName, int LoginCount);
```

### اشتباه ۴: تکیه بر نام تولیدشده توسط کامپایلر

نام‌هایی که در decompiler یا reflection مشاهده می‌شوند، بخشی از API زبان نیستند و ممکن است تغییر کنند.

### اشتباه ۵: یکی دانستن نوع ناشناس و tuple

نوع ناشناس و tuple هر دو می‌توانند چند مقدار را گروه‌بندی کنند، اما از نظر نوع، رفتار حافظه، deconstruction و کاربرد API یکسان نیستند.

### اشتباه ۶: انتظار تغییرناپذیری عمیق

خود property قابل جایگزینی نیست، اما شیء داخلی آن ممکن است قابل تغییر باشد:

```csharp
var data = new { Items = new List<int> { 1, 2 } };

data.Items.Add(3); // معتبر
```

<a id="exercises"></a>
## تمرین‌ها

### تمرین ۱: مشخصات محصول

نوع ناشناسی با propertyهای `Name`، `Price` و `IsAvailable` بسازید و اطلاعات آن را چاپ کنید.

### تمرین ۲: آرایه

آرایه‌ای از محصولات بسازید و با `foreach` فقط محصولاتی را چاپ کنید که قیمتشان بیشتر از ۱۰۰ است.

### تمرین ۳: Projection با LINQ

از فهرست سفارش‌ها فقط سفارش‌های بالاتر از ۸۰۰ را انتخاب کنید و با نوع ناشناس propertyهای `OrderId`، `Customer` و `Total` را بسازید.

### تمرین ۴: تبدیل به `record`

مدل زیر را ابتدا با نوع ناشناس بسازید:

```csharp
var result = new
{
    StudentName = "Mina",
    Passed = true
};
```

سپس آن را به `record` تبدیل کنید و از یک متد برگردانید. تفاوت خوانایی و قابلیت استفاده‌ی مجدد را بررسی کنید.

<a id="summary"></a>
## جمع‌بندی

- نوع ناشناس یک نوع واقعی است؛ فقط نامی ندارد که در کد منبع استفاده کنیم.
- با `new { ... }` ساخته می‌شود.
- کامپایلر نوع را در زمان کامپایل تولید و ویژگی‌ها را استنباط می‌کند.
- `var` نوع را حذف نمی‌کند و با `dynamic` تفاوت دارد.
- نوع ناشناس معمولاً یک `internal sealed class` با propertyهای عمومی و فقط‌خواندنی است.
- برابری آن مقدارمحور است؛ نام، نوع و ترتیب propertyها در شکل نوع اهمیت دارند.
- کاربرد اصلی آن نگهداری موقت داده و projection در LINQ است.
- ویژگی‌های نوع ناشناس سطحی فقط‌خواندنی هستند، نه الزاماً همه‌ی اشیای داخلی.
- برای API عمومی، serialization، قرارداد بین لایه‌ها و داده‌ی قابل استفاده‌ی مجدد از `class` یا `record` استفاده کنید.

<a id="references"></a>
## منابع معتبر

منابع زیر از مستندات رسمی Microsoft Learn و مشخصات زبان C# انتخاب شده‌اند:

1. [Anonymous types - C# Programming Guide | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types)
2. [Expressions - C# language specification | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions)
3. [Implicitly typed local variables - C# Programming Guide | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/implicitly-typed-local-variables)
4. [How to use implicitly typed local variables and arrays in a query expression | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-use-implicitly-typed-local-variables-and-arrays-in-a-query-expression)
5. [Object and collection initializers - C# Programming Guide | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers)
6. [The `with` expression - C# reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression)
7. [Choosing between anonymous and tuple types - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/base-types/choosing-between-anonymous-and-tuple)
8. [Tuple types - C# reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
9. [Language features that support LINQ - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/linq/get-started/features-that-support-linq)
10. [C# language specification repository](https://github.com/dotnet/csharpstandard)
