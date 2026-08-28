# ویژگی‌های اصلی Recordها در C#

> **مخاطب هدف:** توسعه‌دهندگان C# از سطح مبتدی تا پیشرفته
> **نسخه‌ی زبان:** C# 9.0 به بعد (با اشاره به قابلیت‌های C# 10)
> **پیش‌نیاز:** آشنایی مقدماتی با کلاس‌ها، ساختارها و مفاهیم OOP در C#

---

## مقدمه

در نسخه‌ی C# 9.0 (منتشر شده در نوامبر ۲۰۲۰ همراه با .NET 5)، مایکروسافت نوع جدیدی به نام **Record** را به زبان اضافه کرد. هدف اصلی این نوع، ساده‌سازی نوشتن کلاس‌هایی است که نقش اصلی آن‌ها **ذخیره‌ی داده** است — چیزی که در ادبیات طراحی نرم‌افزار به آن **Data-Centric Type** یا **DTO (Data Transfer Object)** یا **Immutable Model** گفته می‌شود.

قبل از معرفی Record، برای ساخت چنین کلاس‌هایی باید کد زیادی می‌نوشتیم: پیاده‌سازی `Equals`، `GetHashCode`، `==` operator، `ToString`، و... . Record تمام این کارها را به صورت خودکار برای ما انجام می‌دهد.

ساختار کلی یک Record ساده:

```csharp
public record Person(string FirstName, string LastName, int Age);
```

در ادامه، ۹ ویژگی اصلی Record را با جزئیات بررسی می‌کنیم.

---

## ۱. Value-Based Equality (برابری مبتنی بر مقدار)

### مفهوم برای مبتدی

در C#، وقتی دو شیء از یک کلاس معمولی را با `==` یا `Equals` مقایسه می‌کنیم، CLR **آدرس حافظه** (Reference) آن‌ها را مقایسه می‌کند. یعنی حتی اگر دو شیء دقیقاً محتوای یکسانی داشته باشند، چون در دو جای مختلف حافظه قرار دارند، **برابر نیستند**.

اما Recordها به صورت پیش‌فرض **برابری مبتنی بر مقدار** دارند؛ یعنی اگر تمام فیلدهای دو Record مقدار یکسانی داشته باشند، آن دو Record **برابر** در نظر گرفته می‌شوند.

### مقایسه قبل و بعد از Record

**قبل از Record (با کلاس معمولی):**

```csharp
public class PersonClass
{
    public string FirstName { get; init; }
    public string LastName { get; init; }

    // باید خودمان این‌ها را بنویسیم:
    public override bool Equals(object obj)
    {
        return obj is PersonClass other &&
               FirstName == other.FirstName &&
               LastName == other.LastName;
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(FirstName, LastName);
    }

    public static bool operator ==(PersonClass left, PersonClass right)
        => Equals(left, right);

    public static bool operator !=(PersonClass left, PersonClass right)
        => !Equals(left, right);
}
```

**بعد از Record:**

```csharp
public record PersonRecord(string FirstName, string LastName);
```

### مثال کامل

```csharp
var p1 = new PersonRecord("Ali", "Rezaei");
var p2 = new PersonRecord("Ali", "Rezaei");
var p3 = new PersonRecord("Sara", "Ahmadi");

Console.WriteLine($"p1 == p2 : {p1 == p2}");  // True
Console.WriteLine($"p1 == p3 : {p1 == p3}");  // False
Console.WriteLine($"Equals : {p1.Equals(p2)}"); // True
```

**خروجی:**

```
p1 == p2 : True
p1 == p3 : False
Equals : True
```

### نکات مهم و محدودیت‌ها

- مقایسه به صورت **عمیق (Deep)** انجام می‌شود؛ یعنی اگر یکی از فیلدها خودش یک Reference Type باشد، محتوای آن نیز مقایسه می‌شود.
- اگر به برابری سفارشی نیاز دارید، می‌توانید `Equals` را override کنید.
- برای Reference Equality (مقایسه آدرس)، می‌توان از `ReferenceEquals` استفاده کرد.

---

## ۲. Immutability (تغییرناپذیری)

### مفهوم برای مبتدی

**Immutability** یعنی پس از ساخت یک شیء، **نمی‌توان** مقادیر فیلدهای آن را تغییر داد. این ویژگی باعث می‌شود کد **قابل پیش‌بینی‌تر** و **ایمن‌تر در محیط‌های چند نخی (Thread-Safe)** باشد.

در Record، به صورت پیش‌فرض تمام Propertyها از نوع **`init`** هستند، نه `set`. یعنی فقط در زمان ساخت (Constructor) می‌توان مقدار داد.

### مقایسه قبل و بعد از Record

**قبل از Record (با کلاس معمولی):**

```csharp
public class PointClass
{
    public int X { get; set; }  // قابل تغییر!
    public int Y { get; set; }
}

var p = new PointClass { X = 5, Y = 10 };
p.X = 20; // مجاز است!
```

**بعد از Record:**

```csharp
public record PointRecord(int X, int Y);

var p = new PointRecord(5, 10);
// p.X = 20;  // ❌ خطای کامپایلر: Property or indexer 'PointRecord.X' cannot be assigned to -- it is read only
```

### مثال کامل

```csharp
public record User(string Username, string Email);

var user = new User("ali_dev", "ali@example.com");
Console.WriteLine($"Username: {user.Username}"); // ali_dev

// user.Username = "sara_dev"; // ❌ خطای کامپایل
```

**خروجی:**

```
Username: ali_dev
```

### نکات مهم و محدودیت‌ها

- اگر واقعاً به Property قابل تغییر نیاز دارید، می‌توانید از `set` استفاده کنید:
  ```csharp
  public record MutableRecord { public int Value { get; set; } }
  ```
  اما این کار فلسفه‌ی Record را زیر سوال می‌برد.
- برای Reference Typeها (مثل `List<T>`)، خودِ Reference تغییرناپذیر است اما محتوای داخلی آن قابل تغییر است.
- در C# 10، **Record Struct** معرفی شد که به صورت پیش‌فرض `set` دارد (چون structها کپی می‌شوند). برای داشتن struct تغییرناپذیر باید از `readonly record struct` استفاده کرد.

---

## ۳. Non-Destructive Mutation (تغییر غیرمخرب)

### مفهوم برای مبتدی

حالا که Recordها تغییرناپذیرند، چطور یک نسخه‌ی جدید با یک تغییر کوچک بسازیم؟ به جای نوشتن یک Constructor جدید یا یک متد `WithX()`، از **`with` expression** استفاده می‌کنیم. این عبارت یک **کپی** از Record اصلی می‌سازد و فقط فیلدهای مشخص‌شده را تغییر می‌دهد.

### مقایسه قبل و بعد از Record

**قبل از Record (با کلاس معمولی):**

```csharp
public class UserClass
{
    public string Name { get; init; }
    public int Age { get; init; }
    public string Email { get; init; }
}

var user = new UserClass { Name = "Ali", Age = 30, Email = "ali@x.com" };

// برای تغییر یک فیلد باید همه چیز را دوباره بنویسیم:
var updated = new UserClass
{
    Name = user.Name,
    Age = user.Age + 1,
    Email = user.Email
};
```

**بعد از Record:**

```csharp
public record UserRecord(string Name, int Age, string Email);

var user = new UserRecord("Ali", 30, "ali@x.com");
var updated = user with { Age = 31 }; // فقط Age تغییر می‌کند
```

### مثال کامل

```csharp
public record Address(string City, string Street, string PostalCode);

var addr1 = new Address("Tehran", "Valiasr", "12345");
var addr2 = addr1 with { PostalCode = "67890" };

Console.WriteLine(addr1);
Console.WriteLine(addr2);
Console.WriteLine($"Are they same reference? {ReferenceEquals(addr1, addr2)}");
```

**خروجی:**

```
Address { City = Tehran, Street = Valiasr, PostalCode = 12345 }
Address { City = Tehran, Street = Valiasr, PostalCode = 67890 }
Are they same reference? False
```

### نکات مهم و محدودیت‌ها

- `with` یک **کپی سطحی (Shallow Copy)** می‌سازد. اگر فیلدی Reference Type باشد، فقط Reference کپی می‌شود.
- می‌توان چند فیلد را همزمان تغییر داد:
  ```csharp
  var newAddr = addr1 with { City = "Isfahan", PostalCode = "99999" };
  ```
- اگر هیچ تغییری ندهیم، یک کپی دقیق ساخته می‌شود:
  ```csharp
  var clone = addr1 with { };
  ```

---

## ۴. کاهش Boilerplate Code (کدهای تکراری)

### مفهوم برای مبتدی

**Boilerplate Code** به کدهای تکراری و کلیشه‌ای گفته می‌شود که باید بارها و بارها بنویسیم اما منطق خاصی ندارند. در کلاس‌های DTO، باید Propertyها، Constructor، `Equals`، `GetHashCode`، `ToString`، و... را پیاده‌سازی کنیم.

Record تمام این‌ها را در **یک خط** انجام می‌دهد.

### مقایسه قبل و بعد از Record

**قبل از Record (حدود ۴۰ خط کد):**

```csharp
public class ProductClass : IEquatable<ProductClass>
{
    public string Name { get; init; }
    public decimal Price { get; init; }
    public string Category { get; init; }

    public ProductClass(string name, decimal price, string category)
    {
        Name = name;
        Price = price;
        Category = category;
    }

    public bool Equals(ProductClass other)
    {
        if (other is null) return false;
        return Name == other.Name &&
               Price == other.Price &&
               Category == other.Category;
    }

    public override bool Equals(object obj) => Equals(obj as ProductClass);

    public override int GetHashCode()
        => HashCode.Combine(Name, Price, Category);

    public override string ToString()
        => $"Product {{ Name = {Name}, Price = {Price}, Category = {Category} }}";

    public static bool operator ==(ProductClass left, ProductClass right)
        => Equals(left, right);

    public static bool operator !=(ProductClass left, ProductClass right)
        => !Equals(left, right);
}
```

**بعد از Record (۱ خط!):**

```csharp
public record ProductRecord(string Name, decimal Price, string Category);
```

### مثال کامل

```csharp
var p1 = new ProductRecord("Laptop", 1500.00m, "Electronics");
var p2 = new ProductRecord("Laptop", 1500.00m, "Electronics");

Console.WriteLine(p1);
Console.WriteLine($"p1 == p2: {p1 == p2}");
```

**خروجی:**

```
ProductRecord { Name = Laptop, Price = 1500.00, Category = Electronics }
p1 == p2: True
```

### نکات مهم و محدودیت‌ها

- کاهش چشمگیر کد = کاهش احتمال خطا.
- اگر نیاز به منطق سفارشی در `Equals` یا `ToString` دارید، می‌توانید آن‌ها را override کنید.

---

## ۵. Compiler-Generated Members (اعضای تولیدشده توسط کامپایلر)

### مفهوم برای مبتدی

وقتی شما یک Record می‌نویسید، کامپایلر C# به صورت خودکار اعضای زیر را تولید می‌کند:

| عضو | توضیح |
|------|--------|
| Propertyهای `init` | برای هر پارامتر positional |
| Constructor | برای مقداردهی اولیه |
| `Equals(T)` | پیاده‌سازی برابری مبتنی بر مقدار |
| `Equals(object)` | Override برای `object` |
| `GetHashCode()` | تولید کد هش بر اساس تمام فیلدها |
| `operator ==` و `!=` | اپراتورهای برابری |
| `ToString()` | نمایش فرمت‌شده از همه‌ی فیلدها |
| `Clone()` (protected) | برای پشتیبانی از `with` expression |
| `PrintMembers` | متد کمکی برای `ToString` |
| `Deconstruct` | برای positional records |

### مثال: دیدن خروجی کامپایلر

کد زیر را در نظر بگیرید:

```csharp
public record Book(string Title, string Author, int Year);
```

کد معادل تولیدشده توسط کامپایلر (به صورت تقریبی):

```csharp
public class Book : IEquatable<Book>
{
    public string Title { get; init; }
    public string Author { get; init; }
    public int Year { get; init; }

    public Book(string title, string author, int year)
    {
        Title = title;
        Author = author;
        Year = year;
    }

    // تمام متدهای Equals, GetHashCode, ToString, operator== و...
    // توسط کامپایلر تولید می‌شوند
}
```

### نکته‌ی مهم: مشاهده‌ی کد تولیدشده

می‌توانید با ابزارهایی مثل **ILSpy** یا **SharpLab** (https://sharplab.io) کد واقعی تولیدشده را ببینید. این کار برای درک عمیق‌تر بسیار مفید است.

---

## ۶. ToString (نمایش فرمت‌شده)

### مفهوم برای مبتدی

در کلاس‌های معمولی، `ToString()` به صورت پیش‌فرض فقط **نام کامل نوع** را برمی‌گرداند (مثلاً `MyNamespace.PersonClass`). این برای دیباگ کردن چندان مفید نیست.

اما Recordها به صورت پیش‌فرض تمام Propertyها را به صورت فرمت‌شده نمایش می‌دهند.

### مقایسه قبل و بعد از Record

**قبل از Record:**

```csharp
public class PersonClass
{
    public string Name { get; init; }
    public int Age { get; init; }
}

var p = new PersonClass { Name = "Ali", Age = 30 };
Console.WriteLine(p); // PersonClass  (نام نوع به تنهایی!)
```

**بعد از Record:**

```csharp
public record PersonRecord(string Name, int Age);

var p = new PersonRecord("Ali", 30);
Console.WriteLine(p);
```

**خروجی:**

```
PersonRecord { Name = Ali, Age = 30 }
```

### مثال با Override سفارشی

```csharp
public record User(string Username, string Email)
{
    // می‌توان ToString را override کرد
    public virtual bool PrintMembers(StringBuilder builder)
    {
        builder.Append($"User: {Username} <{Email}>");
        return true;
    }
}

var u = new User("ali_dev", "ali@example.com");
Console.WriteLine(u);
```

**خروجی:**

```
User: ali_dev <ali@example.com>
```

### نکات مهم و محدودیت‌ها

- برای override کردن `ToString` در Record، باید متد `PrintMembers` را override کنید، نه خود `ToString` را. این کار باعث می‌شود فرمت استاندارد Record حفظ شود.
- اگر `ToString` را مستقیماً override کنید، فرمت `{ ... }` از بین می‌رود.

---

## ۷. Deconstruction (تجزیه)

### مفهوم برای مبتدی

**Deconstruction** یعنی بتوانید یک Record را به اجزای سازنده‌اش تجزیه کنید و هر جزء را در یک متغیر جداگانه قرار دهید. این ویژگی مخصوصاً برای **Positional Records** (Recordهایی که با پارامتر در constructor تعریف می‌شوند) در دسترس است.

### مثال

```csharp
public record Point(int X, int Y);

var point = new Point(10, 20);

// Deconstruction
var (x, y) = point;

Console.WriteLine($"X = {x}, Y = {y}");
```

**خروجی:**

```
X = 10, Y = 20
```

### Deconstruction در عمل (مثال واقعی)

```csharp
public record Customer(string Name, string Email, string City);

var customers = new List<Customer>
{
    new("Ali", "ali@x.com", "Tehran"),
    new("Sara", "sara@x.com", "Isfahan"),
    new("Reza", "reza@x.com", "Shiraz")
};

// استفاده در LINQ
foreach (var (name, email, city) in customers)
{
    Console.WriteLine($"{name} from {city}");
}
```

**خروجی:**

```
Ali from Tehran
Sara from Isfahan
Reza from Shiraz
```

### نکات مهم و محدودیت‌ها

- فقط برای **Positional Records** به صورت خودکار تولید می‌شود.
- اگر Record را با syntax معمولی Property تعریف کنید، باید خودتان `Deconstruct` را بنویسید:
  ```csharp
  public record Product
  {
      public string Name { get; init; }
      public decimal Price { get; init; }
      
      public void Deconstruct(out string name, out decimal price)
      {
          name = Name;
          price = Price;
      }
  }
  ```
- می‌توانید فقط بخشی از اجزا را دریافت کنید:
  ```csharp
  var (name, _, _) = customer; // فقط name
  ```

---

## ۸. Copy Semantics (مفهوم کپی)

### مفهوم برای مبتدی

وقتی از `with` expression استفاده می‌کنید، کامپایلر یک **کپی سطحی (Shallow Copy)** از Record می‌سازد. این کار از طریق یک متد `protected` به نام `<Clone>$` انجام می‌شود.

### مثال

```csharp
public record Address(string City, string Street);
public record Person(string Name, Address Address);

var originalAddress = new Address("Tehran", "Valiasr");
var person1 = new Person("Ali", originalAddress);

// کپی با تغییر Name
var person2 = person1 with { Name = "Sara" };

Console.WriteLine($"person1: {person1}");
Console.WriteLine($"person2: {person2}");

// توجه: Address یک Reference است و کپی سطحی انجام شده
Console.WriteLine($"Same Address reference? {ReferenceEquals(person1.Address, person2.Address)}");
```

**خروجی:**

```
person1: Person { Name = Ali, Address = Address { City = Tehran, Street = Valiasr } }
person2: Person { Name = Sara, Address = Address { City = Tehran, Street = Valiasr } }
Same Address reference? True
```

### مقایسه با کلاس معمولی

در کلاس معمولی، برای پیاده‌سازی کپی باید:
- `ICloneable` را پیاده‌سازی کنیم
- یا یک Constructor کپی بنویسیم
- یا از `MemberwiseClone` استفاده کنیم

```csharp
// کلاس معمولی - نیاز به کد زیاد
public class PersonClass : ICloneable
{
    public string Name { get; init; }
    public object Clone() => MemberwiseClone();
}
```

### نکات مهم و محدودیت‌ها

- **کپی سطحی** یعنی Reference Typeها به اشتراک گذاشته می‌شوند.
- برای **کپی عمیق (Deep Copy)**، باید خودتان پیاده‌سازی کنید:
  ```csharp
  var person3 = person1 with { Address = person1.Address with { City = "Shiraz" } };
  ```
- متد `<Clone>$` یک متد `protected` است و فقط برای استفاده‌ی داخلی `with` طراحی شده.

---

## ۹. پشتیبانی از Inheritance در Record Class

### مفهوم برای مبتدی

Recordها می‌توانند از یکدیگر ارث‌بری کنند. این ویژگی فقط برای **Record Class** در دسترس است (نه Record Struct).

### قوانین ارث‌بری

1. یک Record Class می‌تواند از یک Record Class دیگر ارث ببرد.
2. یک Record Class می‌تواند از یک کلاس معمولی ارث ببرد.
3. **یک کلاس معمولی نمی‌تواند از یک Record ارث ببرد.**
4. Recordها نمی‌توانند از `struct` ارث ببرند (چون structها sealed هستند).

### مثال: ارث‌بری بین Recordها

```csharp
public record Vehicle(string Brand, int Year);
public record Car(string Brand, int Year, int Doors) : Vehicle(Brand, Year);
public record ElectricCar(string Brand, int Year, int Doors, int BatteryCapacity)
    : Car(Brand, Year, Doors);

var car = new ElectricCar("Tesla", 2024, 4, 100);
Console.WriteLine(car);
```

**خروجی:**

```
ElectricCar { Brand = Tesla, Year = 2024, Doors = 4, BatteryCapacity = 100 }
```

### مثال: ارث‌بری از کلاس معمولی

```csharp
public abstract class BaseEntity
{
    public Guid Id { get; init; } = Guid.NewGuid();
}

public record User(string Name, string Email) : BaseEntity;

var user = new User("Ali", "ali@x.com");
Console.WriteLine($"Id: {user.Id}, Name: {user.Name}");
```

**خروجی:**

```
Id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx, Name: Ali
```

### برابری در ارث‌بری

کامپایلر به صورت خودکار برابری را برای تمام فیلدها (حتی فیلدهای کلاس پایه) بررسی می‌کند:

```csharp
var car1 = new ElectricCar("Tesla", 2024, 4, 100);
var car2 = new ElectricCar("Tesla", 2024, 4, 100);

Console.WriteLine(car1 == car2); // True
```

### نکات مهم و محدودیت‌ها

- **کلاس معمولی نمی‌تواند از Record ارث ببرد.** این یک محدودیت طراحی است.
- اگر کلاس پایه `sealed` باشد، Record نمی‌تواند از آن ارث ببرد.
- متد `PrintMembers` به صورت `virtual` تولید می‌شود تا کلاس‌های مشتق‌شده بتوانند فیلدهای خود را به `ToString` اضافه کنند.
- در ارث‌بری، نوع پایه باید در constructor با `base(...)` صدا زده شود.

---

## چرا Record بیشتر برای Data-Centric Types طراحی شده است؟

### مفهوم Data-Centric Type

**Data-Centric Type** به نوعی گفته می‌شود که **نقش اصلی آن نگهداری داده است**، نه رفتار. این نوع‌ها معمولاً:

- تعداد زیادی Property دارند
- رفتار (Method) کمی دارند یا اصلاً ندارند
- به عنوان DTO، ViewModel، Configuration، یا Model در لایه‌های مختلف استفاده می‌شوند
- معمولاً تغییرناپذیر (Immutable) هستند

### مثال‌های کاربردی

```csharp
// DTO برای انتقال داده بین لایه‌ها
public record CreateUserRequest(string Username, string Email, string Password);

// Configuration
public record DatabaseConfig(string ConnectionString, int Timeout, bool UseSsl);

// Domain Model
public record Money(decimal Amount, string Currency);

// API Response
public record ApiResponse<T>(bool Success, T Data, string ErrorMessage);
```

### چرا Record برای این کار مناسب است؟

| ویژگی | مزیت برای Data-Centric Types |
|--------|-------------------------------|
| **Value-Based Equality** | مقایسه‌ی دو DTO بر اساس محتوا |
| **Immutability** | جلوگیری از تغییر تصادفی داده‌ها |
| **with Expression** | ساخت نسخه‌های جدید با تغییرات جزئی |
| **ToString فرمت‌شده** | دیباگ آسان‌تر |
| **کاهش Boilerplate** | تمرکز بر منطق به جای کد تکراری |
| **Thread-Safety** | ایمن در محیط‌های چند نخی |

### چه زمانی از Record استفاده نکنیم؟

- وقتی نوع شما **رفتار زیادی** دارد (مثل Serviceها، Controllerها)
- وقتی نیاز به **تغییرپذیری** دارید (Mutable State)
- وقتی نیاز به **ارث‌بری معکوس** دارید (کلاس معمولی از Record ارث ببرد)
- وقتی نیاز به **سازگاری با ORM**هایی دارید که به Propertyهای `set` نیاز دارند (مثل Entity Framework در برخی سناریوها)

---

## تفاوت «Record به عنوان یک زبان‌فیچر» و «Class/Struct به عنوان نوع زیرین»

### این بخش بسیار مهم است!

یکی از مفاهیمی که اغلب باعث سردرگمی می‌شود، این است که **Record یک نوع جدید در CLR نیست**. Record فقط یک **زبان‌فیچر (Language Feature)** است که کامپایلر C# را مجبور می‌کند اعضای خاصی را تولید کند.

### توضیح دقیق

وقتی می‌نویسید:

```csharp
public record Person(string Name, int Age);
```

کامپایلر این را به یک **کلاس معمولی** تبدیل می‌کند که اعضای خاصی دارد:

```csharp
// کد معادل تولیدشده (تقریبی)
public class Person : IEquatable<Person>
{
    public string Name { get; init; }
    public int Age { get; init; }
    
    // + Constructor, Equals, GetHashCode, ToString, operator==, Clone, ...
}
```

### جدول مقایسه

| مفهوم | توضیح |
|--------|--------|
| **Record** | یک **کلمه‌کلیدی** در زبان C# که به کامپایلر می‌گوید اعضای خاصی تولید کند |
| **Record Class** | یک کلاس معمولی با قابلیت‌های اضافی (پیش‌فرض از C# 9) |
| **Record Struct** | یک struct معمولی با قابلیت‌های اضافی (از C# 10) |
| **CLR Type** | در سطح CLR، فقط `class` یا `struct` وجود دارد، نه `record` |

### چرا این تمایز مهم است؟

1. **سازگاری با کد قدیمی:** یک Record می‌تواند در کدی که انتظار کلاس معمولی دارد استفاده شود:
   ```csharp
   public record Person(string Name);
   
   // این کار می‌کند!
   object obj = new Person("Ali");
   ```

2. **Reflection:** در Reflection، یک Record دقیقاً مانند یک کلاس معمولی به نظر می‌رسد:
   ```csharp
   var type = typeof(Person);
   Console.WriteLine(type.IsClass); // True
   ```

3. **IL Code:** در کد IL تولیدشده، هیچ نشانه‌ای از `record` وجود ندارد (فقط یک attribute کوچک به نام `System.Runtime.CompilerServices.CompilerFeatureRequired` اضافه می‌شود).

4. **محدودیت‌ها:** چون Record در واقع یک کلاس است، محدودیت‌های کلاس را هم دارد:
   - نمی‌تواند از struct ارث ببرد
   - نمی‌تواند در stackalloc استفاده شود
   - به صورت Reference Type روی Heap قرار می‌گیرد

### مثال عملی

```csharp
public record Point(int X, int Y);

var p = new Point(5, 10);

// این‌ها همه کار می‌کنند چون Record در واقع یک کلاس است:
object obj = p;
IEquatable<Point> equatable = p;
Type type = p.GetType();

Console.WriteLine($"Is Class: {type.IsClass}");
Console.WriteLine($"Type Name: {type.Name}");
Console.WriteLine($"Base Type: {type.BaseType?.Name}");
```

**خروجی:**

```
Is Class: True
Type Name: Point
Base Type: Object
```

### جمع‌بندی این بخش

> **Record یک کلاس (یا struct) است که کامپایلر به صورت خودکار اعضای خاصی را برای آن تولید می‌کند.** این یعنی شما تمام مزایای OOP را دارید، به علاوه‌ی قابلیت‌های اضافی که Record ارائه می‌دهد.

---

## جمع‌بندی نهایی

| ویژگی | وضعیت در Record |
|--------|------------------|
| Value-Based Equality | ✅ پیش‌فرض |
| Immutability | ✅ پیش‌فرض (با `init`) |
| Non-Destructive Mutation | ✅ با `with` expression |
| کاهش Boilerplate | ✅ چشمگیر |
| Compiler-Generated Members | ✅ خودکار |
| ToString فرمت‌شده | ✅ خودکار |
| Deconstruction | ✅ برای Positional Records |
| Copy Semantics | ✅ Shallow Copy با `with` |
| Inheritance | ✅ فقط برای Record Class |

### توصیه‌های نهایی

1. **از Record برای DTOها، Configurationها، و Domain Models استفاده کنید.**
2. **برای Serviceها و Controllerها از کلاس معمولی استفاده کنید.**
3. **اگر نیاز به تغییرپذیری دارید، از `record struct` یا کلاس معمولی استفاده کنید.**
4. **همیشه به Deep Copy بودن یا نبودن توجه کنید.**
5. **از ابزارهایی مثل SharpLab برای دیدن کد تولیدشده استفاده کنید.**

---

## منابع رسمی

برای مطالعه‌ی بیشتر، منابع زیر توصیه می‌شوند:

### Microsoft Learn
- [Record Types - C# Guide](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)
- [Records - C# Reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- [What's new in C# 9.0](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#record-types)
- [What's new in C# 10.0 - Record Structs](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#record-structs)
- [with expression - C# Reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression)

### C# Language Specification
- [C# Language Specification - Records (PDF)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/specifications/)
- [Records Proposal (GitHub - dotnet/csharplang)](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-9.0/records.md)
- [Record Structs Proposal](https://github.com/dotnet/csharplang/blob/main/proposals/csharp-10.0/record-structs.md)

### ابزارهای مفید
- [SharpLab - C# Compiler Explorer](https://sharplab.io/)
- [.NET Fiddle](https://dotnetfiddle.net/)

---

> **نویسنده:** این مقاله بخشی از Repository آموزشی جامع C# است.
> **تاریخ بازبینی:** August 2026
> **نسخه‌ی C# مورد بررسی:** C# 9.0 تا C# 12

اگر سوال یا پیشنهادی دارید، لطفاً Issue باز کنید یا Pull Request بفرستید. 🚀