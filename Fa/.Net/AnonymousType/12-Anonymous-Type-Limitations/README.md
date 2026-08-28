# 📘 مقاله جامع: مقایسه Anonymous Type با Class در C#

> **تاریخ نگارش:** ۵ شهریور ۱۴۰۵ (۲۷ آگوست ۲۰۲۶)
> **مخاطب:** توسعه‌دهندگان C# از سطح مبتدی تا پیشرفته
> **پیش‌نیاز:** آشنایی مقدماتی با C# و مفاهیم OOP

---

## 📑 فهرست مطالب

1. [مقدمه](#مقدمه)
2. [Anonymous Type چیست؟](#anonymous-type-چیست)
3. [Class چیست؟](#class-چیست)
4. [مقایسه تفصیلی Anonymous Type و Class](#مقایسه-تفصیلی)
5. [مقایسه Anonymous Type با Record، Struct و Tuple](#مقایسه-با-سایر-انواع)
6. [جدول مقایسه جامع](#جدول-مقایسه-جامع)
7. [سناریوهای واقعی و انتخاب مناسب](#سناریوهای-واقعی)
8. [نکات مهم](#نکات-مهم)
9. [اشتباهات رایج](#اشتباهات-رایج)
10. [جمع‌بندی](#جمع-بندی)
11. [منابع معتبر](#منابع-معتبر)

---

## مقدمه

در زبان #C، ما با انواع مختلفی از داده‌ها سروکار داریم. گاهی نیاز داریم یک شیء موقت بسازیم که فقط در محدوده‌ای خاص از کد استفاده شود و گاهی به یک ساختار کامل با قابلیت‌های وراثت، متد و … نیاز داریم. در این میان، **Anonymous Type** و **Class** دو ابزار قدرتمند اما با کاربردهای کاملاً متفاوت هستند.

انتخاب اشتباه بین این دو می‌تواند منجر به کدهای غیرقابل نگهداری، کاهش کارایی و نقض اصول طراحی شود. در این مقاله، این دو مفهوم را از پایه تا سطح پیشرفته بررسی و مقایسه می‌کنیم.

---

## Anonymous Type چیست؟

**Anonymous Type** (نوع بی‌نام) یک نوع مرجع (Reference Type) است که توسط کامپایلر #C به‌صورت خودکار تولید می‌شود و **نامی توسط توسعه‌دهنده برای آن انتخاب نمی‌شود**. این نوع معمولاً برای گروه‌بندی موقت چند Property در یک شیء استفاده می‌شود.

### ویژگی‌های کلیدی

- توسط کامپایلر تولید می‌شود (Compiler-generated)
- نام نوع توسط کامپایلر تعیین می‌شود (مثلاً `<>f__AnonymousType0`)
- فقط می‌تواند شامل **Read-only Property** باشد
- **Immutable** (تغییرناپذیر) است
- نمی‌تواند Memberهای دیگری مانند Method، Event، Field داشته باشد
- از `object` ارث‌بری می‌کند و `Equals`، `GetHashCode` و `ToString` را Override می‌کند

### نحوه تعریف

```csharp
// تعریف یک Anonymous Type
var person = new 
{ 
    Name = "Ali", 
    Age = 30, 
    City = "Tehran" 
};

Console.WriteLine(person.Name); // Ali
Console.WriteLine(person.GetType().Name); // <>f__AnonymousType0`3
```

> ⚠️ **نکته مهم:** چون نام نوع مشخص نیست، حتماً باید از `var` استفاده شود.

### مثال پیشرفته: استفاده در LINQ

```csharp
var users = new List<User>
{
    new User { Name = "Sara", Age = 25, Department = "IT" },
    new User { Name = "Reza", Age = 30, Department = "HR" }
};

var result = users.Select(u => new 
{ 
    FullName = u.Name.ToUpper(), 
    u.Age, 
    IsYoung = u.Age < 30 
});

foreach (var item in result)
{
    Console.WriteLine($"{item.FullName}, Age: {item.Age}, Young: {item.IsYoung}");
}
```

---

## Class چیست؟

**Class** (کلاس) یک نوع مرجع (Reference Type) است که توسط توسعه‌دهنده تعریف می‌شود و می‌تواند شامل Property، Field، Method، Event، Constructor و … باشد. کلاس‌ها ستون فقرات برنامه‌نویسی شیءگرا (OOP) در #C هستند.

### ویژگی‌های کلیدی

- توسط توسعه‌دهنده نام‌گذاری می‌شود
- می‌تواند هر نوع Member را داشته باشد
- به‌صورت پیش‌فرض **Mutable** (تغییرپذیر) است
- از **Inheritance** پشتیبانی می‌کند
- می‌تواند **Interface** پیاده‌سازی کند
- می‌تواند در هر Scope و به‌صورت Public/Internal/Private تعریف شود

### نحوه تعریف

```csharp
public class Person
{
    // Field
    private string _name;

    // Constructor
    public Person(string name, int age, string city)
    {
        _name = name;
        Age = age;
        City = city;
    }

    // Properties
    public string Name 
    { 
        get => _name; 
        set => _name = value; 
    }
    public int Age { get; set; }
    public string City { get; set; }

    // Method
    public void Introduce()
    {
        Console.WriteLine($"Hi, I'm {Name}, {Age} years old from {City}.");
    }

    // Event
    public event EventHandler NameChanged;
}
```

### ایجاد Object

```csharp
var person = new Person("Ali", 30, "Tehran");
person.Age = 31; // Mutable - قابل تغییر
person.Introduce();
```

---

## مقایسه تفصیلی Anonymous Type و Class

در ادامه، ۲۴ معیار مختلف را به‌صورت دقیق مقایسه می‌کنیم:

### 1. نحوه تعریف (Definition)

**Anonymous Type:**
```csharp
var obj = new { Name = "Ali", Age = 30 };
```

**Class:**
```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
var obj = new Person { Name = "Ali", Age = 30 };
```

### 2. نام Type (Type Name)

- **Anonymous Type:** نام توسط کامپایلر تولید می‌شود و در دسترس نیست.
- **Class:** توسعه‌دهنده نام را مشخص می‌کند.

### 3. ایجاد Object (Object Creation)

هر دو با کلمه کلیدی `new` ایجاد می‌شوند، اما Anonymous Type حتماً نیاز به `var` دارد.

### 4. Propertyها

- **Anonymous Type:** فقط Read-only (فقط `get`).
- **Class:** می‌تواند `get`، `set`، یا هر دو را داشته باشد.

### 5. Immutable بودن

- **Anonymous Type:** کاملاً Immutable.
- **Class:** به‌صورت پیش‌فرض Mutable (مگر اینکه خودتان Immutable طراحی کنید).

### 6. قابلیت تغییر (Mutability)

```csharp
// Anonymous Type - ❌ خطا
var a = new { Name = "Ali" };
a.Name = "Reza"; // Compile Error!

// Class - ✅ مجاز
var p = new Person { Name = "Ali" };
p.Name = "Reza"; // OK
```

### 7. Methodها

- **Anonymous Type:** ❌ نمی‌تواند Method داشته باشد.
- **Class:** ✅ می‌تواند هر تعداد Method داشته باشد.

### 8. Constructor

- **Anonymous Type:** ❌ Constructor سفارشی ندارد.
- **Class:** ✅ می‌تواند چندین Constructor داشته باشد.

### 9. Field

- **Anonymous Type:** ❌ Field ندارد.
- **Class:** ✅ می‌تواند Field داشته باشد.

### 10. Event

- **Anonymous Type:** ❌ Event ندارد.
- **Class:** ✅ می‌تواند Event تعریف کند.

### 11. Inheritance (وراثت)

- **Anonymous Type:** ❌ فقط از `object` ارث‌بری می‌کند و نمی‌توان آن را تغییر داد.
- **Class:** ✅ می‌تواند از کلاس‌های دیگر ارث‌بری کند.

### 12. Interface

- **Anonymous Type:** ❌ نمی‌تواند Interface پیاده‌سازی کند.
- **Class:** ✅ می‌تواند هر تعداد Interface را پیاده‌سازی کند.

### 13. Accessibility

- **Anonymous Type:** همیشه در محدوده متد جاری است.
- **Class:** می‌تواند `public`, `internal`, `private`, `protected` باشد.

### 14. Scope

- **Anonymous Type:** فقط در محدوده متدی که تعریف شده قابل استفاده است.
- **Class:** در کل Namespace یا Assembly قابل دسترسی است.

### 15. استفاده در Method

```csharp
// Anonymous Type - فقط محلی
public void ShowUser()
{
    var user = new { Name = "Ali", Age = 30 };
    Console.WriteLine(user.Name);
}

// Class - در هر جای کلاس قابل استفاده
public class UserService
{
    public Person GetUser() => new Person();
}
```

### 16. استفاده به عنوان Return Type

- **Anonymous Type:** ❌ نمی‌توان مستقیماً به‌عنوان Return Type استفاده کرد (مگر با `object` یا `dynamic` که توصیه نمی‌شود).
- **Class:** ✅ به‌راحتی به‌عنوان Return Type استفاده می‌شود.

```csharp
// ❌ این کار ممکن نیست
public ??? GetUser() => new { Name = "Ali" };

// ✅ راه‌حل: استفاده از Tuple یا Record یا Class
public (string Name, int Age) GetUserTuple() => ("Ali", 30);
```

### 17. استفاده در API

- **Anonymous Type:** ❌ برای APIهای عمومی مناسب نیست (چون قابل نام‌گذاری نیست).
- **Class:** ✅ گزینه اصلی برای DTO در APIها.

### 18. استفاده در LINQ

- **Anonymous Type:** ✅ **بهترین گزینه** برای `Select` و `GroupBy` در LINQ.
- **Class:** ✅ قابل استفاده است ولی معمولاً verboseتر.

```csharp
var result = products
    .GroupBy(p => p.Category)
    .Select(g => new 
    { 
        Category = g.Key, 
        TotalPrice = g.Sum(p => p.Price) 
    });
```

### 19. استفاده در Domain Model

- **Anonymous Type:** ❌ اصلاً مناسب نیست.
- **Class:** ✅ گزینه اصلی برای Domain Model.

### 20. قابلیت استفاده مجدد (Reusability)

- **Anonymous Type:** ❌ قابل استفاده مجدد نیست.
- **Class:** ✅ در کل پروژه قابل استفاده مجدد است.

### 21. Maintainability (نگهداری‌پذیری)

- **Anonymous Type:** در استفاده‌های کوتاه خوب است، اما برای ساختارهای پیچیده ضعیف.
- **Class:** نگهداری‌پذیری بالا، به‌خصوص در پروژه‌های بزرگ.

### 22. خوانایی (Readability)

- **Anonymous Type:** برای داده‌های ساده، خوانایی بالایی دارد.
- **Class:** با نام‌گذاری مناسب، خوانایی معنایی بهتری دارد.

### 23. زمان مناسب استفاده

- **Anonymous Type:** برای داده‌های موقت، LINQ Projections، Shape-changing در کوئری‌ها.
- **Class:** برای Entityها، DTOها، Serviceها و هر ساختار پایدار.

### 24. Equality Comparison

```csharp
var a1 = new { Name = "Ali", Age = 30 };
var a2 = new { Name = "Ali", Age = 30 };
Console.WriteLine(a1.Equals(a2)); // ✅ True (Value-based equality)

var p1 = new Person { Name = "Ali", Age = 30 };
var p2 = new Person { Name = "Ali", Age = 30 };
Console.WriteLine(p1.Equals(p2)); // ❌ False (Reference-based, unless overridden)
```

---

## مقایسه Anonymous Type با سایر انواع

### 🔹 Anonymous Type در مقابل Record (C# 9+)

**Record** نوعی Reference Type است که مانند Anonymous Type Immutable و دارای Value-based Equality است، اما **نام‌گذاری شده** و قابلیت‌های بیشتری دارد.

```csharp
// Anonymous Type
var person1 = new { Name = "Ali", Age = 30 };

// Record
public record Person(string Name, int Age);
var person2 = new Person("Ali", 30);
```

| ویژگی | Anonymous Type | Record |
|---|---|---|
| نام Type | ندارد | دارد |
| Immutable | ✅ | ✅ |
| Value Equality | ✅ | ✅ |
| Return Type | ❌ | ✅ |
| Inheritance | ❌ | ✅ (محدود) |
| Interface | ❌ | ✅ |
| `with` expression | ❌ | ✅ |

> 💡 **توصیه:** در #C 9 به بعد، برای بسیاری از مواردی که قبلاً از Anonymous Type استفاده می‌شد، **Record** جایگزین بهتری است.

### 🔹 Anonymous Type در مقابل Struct

**Struct** یک Value Type است که برای داده‌های کوچک و با کارایی بالا استفاده می‌شود.

| ویژگی | Anonymous Type | Struct |
|---|---|---|
| نوع | Reference Type | Value Type |
| نام | ندارد | دارد |
| Mutable | ❌ | ✅ (معمولاً) |
| Allocation | Heap | Stack (معمولاً) |
| مناسب برای | داده‌های موقت | داده‌های کوچک و پرکاربرد |

### 🔹 Anonymous Type در مقابل Tuple

**Tuple** (ValueTuple) برای گروه‌بندی سریع چند مقدار بدون نام Property استفاده می‌شود.

```csharp
// Tuple
var t = ("Ali", 30);
Console.WriteLine(t.Item1); // بدون نام معنادار

// Named Tuple
var nt = (Name: "Ali", Age: 30);
Console.WriteLine(nt.Name);

// Anonymous Type
var a = new { Name = "Ali", Age = 30 };
Console.WriteLine(a.Name);
```

| ویژگی | Anonymous Type | Tuple |
|---|---|---|
| نوع | Reference Type | Value Type |
| نام Property | ✅ (اجباری) | ❌ (اختیاری) |
| کارایی | کمتر | بیشتر |
| LINQ Projection | ✅ عالی | ✅ خوب |
| Return Type | ❌ | ✅ |

---

## جدول مقایسه جامع

| معیار | Anonymous Type | Class | Record | Struct | Tuple |
|---|:---:|:---:|:---:|:---:|:---:|
| نوع | Reference | Reference | Reference | Value | Value |
| نام Type | ❌ | ✅ | ✅ | ✅ | ❌ |
| Immutable | ✅ | ❌ (پیش‌فرض) | ✅ | ❌ | ❌ |
| Property | ✅ فقط get | ✅ get/set | ✅ init | ✅ | ✅ |
| Method | ❌ | ✅ | ✅ | ✅ | ❌ |
| Constructor | ❌ | ✅ | ✅ | ✅ | ❌ |
| Field | ❌ | ✅ | ✅ | ✅ | ❌ |
| Event | ❌ | ✅ | ✅ | ✅ | ❌ |
| Inheritance | ❌ | ✅ | ✅ (محدود) | ❌ | ❌ |
| Interface | ❌ | ✅ | ✅ | ✅ | ❌ |
| Return Type | ❌ | ✅ | ✅ | ✅ | ✅ |
| Value Equality | ✅ | ❌ | ✅ | ❌ | ✅ |
| LINQ Projection | ✅ عالی | ✅ | ✅ | ✅ | ✅ |
| استفاده در API | ❌ | ✅ | ✅ | ✅ | ⚠️ محدود |
| Domain Model | ❌ | ✅ | ✅ | ⚠️ | ❌ |
| Reusability | ❌ | ✅ | ✅ | ✅ | ❌ |
| کارایی | متوسط | متوسط | متوسط | بالا | بالا |

---

## سناریوهای واقعی

### 🎯 سناریو 1: Projection در LINQ

```csharp
var orders = dbContext.Orders
    .Where(o => o.Status == "Pending")
    .Select(o => new 
    { 
        o.OrderId, 
        CustomerName = o.Customer.Name, 
        Total = o.Items.Sum(i => i.Price * i.Quantity) 
    })
    .ToList();
```

✅ **انتخاب:** Anonymous Type
**دلیل:** داده موقت است، فقط برای نمایش استفاده می‌شود و نیازی به بازگشت از متد ندارد.

---

### 🎯 سناریو 2: DTO برای Web API

```csharp
public class UserDto
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string Email { get; set; }
}
```

✅ **انتخاب:** Class (یا Record)
**دلیل:** باید به‌عنوان Return Type استفاده شود، در Swagger نمایش داده شود و قابل استفاده مجدد باشد.

---

### 🎯 سناریو 3: Domain Entity

```csharp
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Order> Orders { get; set; }
    
    public void PlaceOrder(Order order) { /* ... */ }
}
```

✅ **انتخاب:** Class
**دلیل:** نیاز به Method، Mutable بودن، و ارتباط با سایر Entityها دارد.

---

### 🎯 سناریو 4: بازگشت چند مقدار از متد

```csharp
// گزینه 1: Tuple
public (int Id, string Name) GetBasicInfo() => (1, "Ali");

// گزینه 2: Record (بهتر برای ساختارهای پیچیده‌تر)
public record BasicInfo(int Id, string Name);
public BasicInfo GetBasicInfo() => new(1, "Ali");
```

✅ **انتخاب:** Tuple برای ساده، Record برای پیچیده
**دلیل:** Anonymous Type قابل بازگشت نیست.

---

### 🎯 سناریو 5: Config یا Options Immutable

```csharp
public record DatabaseConfig(
    string ConnectionString, 
    int Timeout, 
    bool EnableLogging);
```

✅ **انتخاب:** Record
**دلیل:** Immutable است، نام‌گذاری شده و قابل استفاده مجدد است.

---

### 🎯 سناریو 6: ساختار کوچک با کارایی بالا (مثلاً Point)

```csharp
public readonly struct Point
{
    public double X { get; }
    public double Y { get; }
    public Point(double x, double y) { X = x; Y = y; }
}
```

✅ **انتخاب:** Struct
**دلیل:** داده کوچک، Immutable، و در حلقه‌های زیاد استفاده می‌شود (کاهش GC Pressure).

---

## نکات مهم

1. ⚡ **Anonymous Type فقط در محدوده متد** قابل استفاده است و نمی‌توان آن را از متد خارج کرد.
2. ⚡ برای عبور دادن Anonymous Type بین متدها، **مجبور به استفاده از `dynamic` یا `object`** هستید که **توصیه نمی‌شود** (از بین رفتن Type Safety و کارایی).
3. ⚡ دو Anonymous Type با Propertyهای یکسان (از نظر نام، نوع و ترتیب) در یک Assembly، **یک نوع واحد** در نظر گرفته می‌شوند.
4. ⚡ در #C 9 به بعد، برای بیشتر کاربردهای Anonymous Type، **Record** جایگزین بهتری است.
5. ⚡ Anonymous Type از `Equals` و `GetHashCode` بر اساس **Value** استفاده می‌کند (نه Reference).
6. ⚡ برای Serializing کردن (مثلاً به JSON)، Anonymous Type کار می‌کند اما **توصیه نمی‌شود**؛ بهتر است از Class یا Record استفاده شود.

---

## اشتباهات رایج

### ❌ اشتباه 1: تلاش برای Return کردن Anonymous Type

```csharp
// ❌ غلط
public var GetUser() => new { Name = "Ali" }; // Compile Error!

// ✅ درست
public object GetUser() => new { Name = "Ali" }; // نیاز به Cast دارد
public (string Name, int Age) GetUser() => ("Ali", 30); // Tuple
public UserDto GetUser() => new UserDto { Name = "Ali" }; // Class/Record
```

### ❌ اشتباه 2: استفاده از Anonymous Type در Domain Model

Anonymous Type برای Domain Model مناسب نیست چون:
- قابل نام‌گذاری نیست
- Method ندارد
- نمی‌تواند Validation داشته باشد

### ❌ اشتباه 3: استفاده از Class برای LINQ Projection ساده

```csharp
// ❌ verbose و غیرضروری
public class TempUser { public string Name { get; set; } }
var users = db.Users.Select(u => new TempUser { Name = u.Name });

// ✅ تمیز و مختصر
var users = db.Users.Select(u => new { u.Name });
```

### ❌ اشتباه 4: فکر کردن اینکه Anonymous Type Mutable است

```csharp
var x = new { Name = "Ali" };
x.Name = "Reza"; // ❌ Compile Error
```

### ❌ اشتباه 5: استفاده از Anonymous Type در API Response

```csharp
// ❌ مشکل در Swagger و Document Generation
return Ok(new { Message = "Success" });

// ✅ بهتر
return Ok(new ApiResponse { Message = "Success" });
```

---

## جمع‌بندی

| نوع | بهترین کاربرد |
|---|---|
| **Anonymous Type** | LINQ Projection، داده‌های موقت محلی، Shape-changing |
| **Class** | Domain Model، Entity، Service، هر ساختار با رفتار (Method) |
| **Record** | DTO، Config، داده‌های Immutable با نام، جایگزین مدرن Anonymous Type |
| **Struct** | داده‌های کوچک، Immutable، با کارایی بالا (Point، Vector، …) |
| **Tuple** | بازگشت چند مقدار از متد، داده‌های موقت بسیار ساده |

### 🎯 قانون طلایی

> اگر داده‌ای **موقت** است، **فقط در یک متد** استفاده می‌شود و **نیازی به رفتار (Method)** ندارد → **Anonymous Type**
> 
> اگر داده‌ای **پایدار** است، **در چندین جا** استفاده می‌شود یا **نیاز به رفتار** دارد → **Class** یا **Record**
> 
> اگر در #C 9+ هستید و داده Immutable است → ترجیحاً **Record** به جای Anonymous Type

---

## منابع معتبر

1. 📚 [Microsoft Learn - Anonymous Types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types)
2. 📚 [Microsoft Learn - Classes](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes)
3. 📚 [Microsoft Learn - Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
4. 📚 [Microsoft Learn - Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
5. 📚 [Microsoft Learn - Structs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct)
6. 📚 [C# Language Reference - Official Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/)
7. 📚 Jon Skeet, *C# in Depth*, 4th Edition
8. 📚 Andrew Lock, *Learning C# by Developing Games with Unity*

---

> 💬 **سوال یا پیشنهادی دارید؟** در بخش Issues مخزن مطرح کنید تا با هم یاد بگیریم!
>
> 📌 **مقالات مرتبط این Repository:**
> - [مقایسه Class و Struct](./class-vs-struct.md)
> - [مقایسه Record و Class](./record-vs-class.md)
> - [Tupleها در #C](./tuples-in-csharp.md)
> - [LINQ و Projection](./linq-projection.md)