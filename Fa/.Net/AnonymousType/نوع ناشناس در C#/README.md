# مفهوم نوع ناشناس (Anonymous Type) در C#

یک راهنمای کامل از مقدماتی تا پیشرفته

---

## فهرست مطالب

- [مقدمه](#مقدمه)
- [Anonymous Type چیست؟](#anonymous-type-چیست)
- [چرا Anonymous Type به وجود آمده است؟](#چرا-anonymous-type-به-وجود-آمده-است)
- [چه مشکلی را حل می‌کند؟](#چه-مشکلی-را-حل-میکند)
- [ساختار کلی Anonymous Type](#ساختار-کلی-anonymous-type)
- [کامپایلر چگونه Anonymous Type را ایجاد می‌کند؟](#کامپایلر-چگونه-anonymous-type-را-ایجاد-میکند)
- [نوع Anonymous Type از دید CLR](#نوع-anonymous-type-از-دید-clr)
- [رابطه Anonymous Type با object](#رابطه-anonymous-type-با-object)
- [تفاوت Anonymous Type با Class معمولی](#تفاوت-anonymous-type-با-class-معمولی)
- [مثال‌های ساده](#مثالهای-ساده)
- [مثال‌های واقعی‌تر](#مثالهای-واقعیتر)
- [اشتباهات رایج مبتدیان](#اشتباهات-رایج-مبتدیان)
- [نکات مهم برای برنامه‌نویسی حرفه‌ای](#نکات-مهم-برای-برنامهنویسی-حرفهای)
- [چه زمانی استفاده کنیم و چه زمانی نکنیم؟](#چه-زمانی-استفاده-کنیم-و-چه-زمانی-نکنیم)
- [جمع‌بندی](#جمع‌بندی)
- [منابع و مطالعه بیشتر](#منابع-و-مطالعه-بیشتر)

---

## مقدمه

در برنامه‌نویسی C#، گاهی اوقات نیاز داریم داده‌های موقتی را در قالب یک شیء ذخیره کنیم، اما نمی‌خواهیم یا نیازی نیست که یک کلاس جداگانه برای آن تعریف کنیم. در این شرایط، **Anonymous Type** (نوع ناشناس) یک راه‌حل عالی و تمیز ارائه می‌دهد.

این مقاله شما را از مفاهیم پایه‌ای تا نکات پیشرفته در مورد Anonymous Type راهنمایی می‌کند.

---

## Anonymous Type چیست؟

**Anonymous Type** یک نوع داده‌ای است که:

- **بدون نام** است (شما نامی برای آن انتخاب نمی‌کنید)
- **توسط کامپایلر** به‌صورت خودکار ایجاد می‌شود
- **فقط خواندنی (Read-only)** است - پس از ایجاد، نمی‌توانید مقادیر آن را تغییر دهید
- **معمولاً برای گروه‌بندی موقت داده‌ها** استفاده می‌شود
- **در سطح کامپایلر** به یک کلاس واقعی تبدیل می‌شود

به زبان ساده: شما فقط ساختار داده را تعریف می‌کنید و کامپایلر خودش یک کلاس کامل برای شما می‌سازد.

---

## چرا Anonymous Type به وجود آمده است؟

Anonymous Type در **C# 3.0** (همراه با .NET Framework 3.5 در سال 2008) معرفی شد. دلیل اصلی پیدایش آن، نیاز به پشتیبانی از **LINQ** (Language Integrated Query) بود.

### مشکلات قبل از Anonymous Type:

1. **نیاز به تعریف کلاس‌های متعدد**: برای هر ساختار داده موقت، باید یک کلاس جداگانه تعریف می‌کردید
2. **کد اضافی و تکراری**: کلاس‌های کوچک فقط برای نگهداری چند فیلد
3. **عدم انعطاف‌پذیری**: تغییر ساختار داده نیازمند تغییر کلاس بود
4. **پیچیدگی در LINQ**: وقتی می‌خواستید چند فیلد را از منابع مختلف ترکیب کنید، نیاز به کلاس‌های واسط داشتید

---

## چه مشکلی را حل می‌کند؟

### مثال: ترکیب داده‌ها بدون Anonymous Type

فرض کنید می‌خواهید نام و سن افراد را از یک لیست استخراج کنید:

```csharp
// قبل از Anonymous Type - نیاز به تعریف کلاس
class PersonInfo
{
    public string Name { get; set; }
    public int Age { get; set; }
}

var people = new List<Person> { /* ... */ };
var personInfos = new List<PersonInfo>();

foreach (var person in people)
{
    personInfos.Add(new PersonInfo 
    { 
        Name = person.Name, 
        Age = person.Age 
    });
}
```

### همان مثال با Anonymous Type:

```csharp
// با Anonymous Type - بدون نیاز به تعریف کلاس
var people = new List<Person> { /* ... */ };

var personInfos = people.Select(p => new 
{ 
    Name = p.Name, 
    Age = p.Age 
}).ToList();
```

**مزایا:**
- ✅ کد کوتاه‌تر و تمیزتر
- ✅ بدون نیاز به تعریف کلاس اضافی
- ✅ انعطاف‌پذیری بیشتر
- ✅ مناسب برای عملیات‌های موقت

---

## ساختار کلی Anonymous Type

### سینتکس پایه:

```csharp
var anonymousObject = new 
{ 
    Property1 = value1, 
    Property2 = value2,
    Property3 = value3
};
```

### نکات مهم در سینتکس:

1. **استفاده از `var`**: چون نام نوع را نمی‌دانید، باید از `var` استفاده کنید
2. **کلمه کلیدی `new`**: برای ایجاد نمونه جدید
3. **Initializers**: مقادیر را درون `{ }` تعریف می‌کنید
4. **نام‌گذاری خودکار**: اگر نام property را مشخص نکنید، از نام متغیر استفاده می‌شود

### مثال با نام‌گذاری خودکار:

```csharp
string name = "Ali";
int age = 25;

// نام propertyها همان نام متغیرها خواهد بود
var person = new { name, age };

// معادل با:
// var person = new { name = name, age = age };

Console.WriteLine(person.name); // خروجی: Ali
Console.WriteLine(person.age);  // خروجی: 25
```

---

## کامپایلر چگونه Anonymous Type را ایجاد می‌کند؟

وقتی شما یک Anonymous Type تعریف می‌کنید، کامپایلر C# در پشت صحنه کارهای زیر را انجام می‌دهد:

### ۱. ایجاد یک کلاس واقعی

کامپایلر یک کلاس با نامی تولید‌شده (مانند `<>f__AnonymousType0`) ایجاد می‌کند.

### ۲. تعریف Propertyهای فقط خواندنی

تمام propertyها به‌صورت `get-only` تعریف می‌شوند:

```csharp
// کدی که شما می‌نویسید:
var person = new { Name = "Ali", Age = 25 };

// کدی که کامپایلر تولید می‌کند (تقریبی):
[CompilerGenerated]
internal sealed class <>f__AnonymousType0<string Name, int Age>
{
    private readonly string <Name>k__BackingField;
    private readonly int <Age>k__BackingField;

    public string Name 
    { 
        get { return <Name>k__BackingField; } 
    }

    public int Age 
    { 
        get { return <Age>k__BackingField; } 
    }

    public <>f__AnonymousType0(string name, int age)
    {
        <Name>k__BackingField = name;
        <Age>k__BackingField = age;
    }
}
```

### ۳. پیاده‌سازی متدهای مهم

کامپایلر به‌صورت خودکار متدهای زیر را پیاده‌سازی می‌کند:

- **`Equals()`**: برای مقایسه دو شیء
- **`GetHashCode()`**: برای استفاده در HashSet و Dictionary
- **`ToString()`**: برای نمایش رشته‌ای

```csharp
var person1 = new { Name = "Ali", Age = 25 };
var person2 = new { Name = "Ali", Age = 25 };

Console.WriteLine(person1.Equals(person2)); // True
Console.WriteLine(person1.GetHashCode() == person2.GetHashCode()); // True
Console.WriteLine(person1.ToString()); // { Name = Ali, Age = 25 }
```

### ۴. استفاده از Generics

کامپایلر از **Type Inference** استفاده می‌کند تا نوع propertyها را تشخیص دهد.

---

## نوع Anonymous Type از دید CLR

از دید **CLR** (Common Language Runtime)، Anonymous Type هیچ تفاوتی با یک کلاس معمولی ندارد:

### ویژگی‌ها از دید CLR:

1. **یک کلاس واقعی است**: با تمام ویژگی‌های یک کلاس
2. **Internal است**: فقط در assembly فعلی قابل دسترسی است
3. **Sealed است**: نمی‌توان از آن ارث‌بری کرد
4. **وراثت از `object`**: مستقیماً از `System.Object` ارث‌بری می‌کند
5. **Propertyهای فقط خواندنی**: هیچ setter ندارد

### بررسی با Reflection:

```csharp
var person = new { Name = "Ali", Age = 25 };
Type type = person.GetType();

Console.WriteLine($"نام نوع: {type.Name}");
Console.WriteLine($"Assembly: {type.Assembly}");
Console.WriteLine($"IsClass: {type.IsClass}");
Console.WriteLine($"IsSealed: {type.IsSealed}");
Console.WriteLine($"IsPublic: {type.IsPublic}");
Console.WriteLine($"BaseType: {type.BaseType.Name}");

// خروجی:
// نام نوع: <>f__AnonymousType0`2
// Assembly: [نام assembly شما]
// IsClass: True
// IsSealed: True
// IsPublic: False
// BaseType: Object
```

---

## رابطه Anonymous Type با object

چون Anonymous Type مستقیماً از `object` ارث‌بری می‌کند، می‌توانید آن را به `object` تبدیل کنید:

```csharp
var person = new { Name = "Ali", Age = 25 };
object obj = person; // تبدیل مجاز

// اما برای دسترسی به propertyها باید cast کنید
// var name = obj.Name; // ❌ خطا!
var name = ((dynamic)obj).Name; // ✅ با dynamic
```

### محدودیت مهم:

شما **نمی‌توانید** Anonymous Type را به‌صورت مستقیم به عنوان نوع بازگشتی یک متد استفاده کنید:

```csharp
// ❌ خطا - نمی‌توانید Anonymous Type را به‌صورت مستقیم برگردانید
public ??? GetPerson()
{
    return new { Name = "Ali", Age = 25 };
}

// ✅ راه‌حل 1: استفاده از object
public object GetPerson()
{
    return new { Name = "Ali", Age = 25 };
}

// ✅ راه‌حل 2: استفاده از dynamic
public dynamic GetPerson()
{
    return new { Name = "Ali", Age = 25 };
}

// ✅ راه‌حل 3: استفاده از Tuple (C# 7.0+)
public (string Name, int Age) GetPerson()
{
    return ("Ali", 25);
}
```

---

## تفاوت Anonymous Type با Class معمولی

| ویژگی | Anonymous Type | Class معمولی |
|--------|----------------|--------------|
| **نام** | ندارد (توسط کامپایلر نام‌گذاری می‌شود) | توسط برنامه‌نویس انتخاب می‌شود |
| **تعریف** | در محل استفاده | در جای جداگانه |
| **Propertyها** | فقط خواندنی (Read-only) | خواندنی/نوشتنی |
| **متدها** | ندارد | می‌تواند داشته باشد |
| **Constructor** | فقط constructor با پارامتر | می‌تواند چندین constructor داشته باشد |
| **ارث‌بری** | نمی‌توان ارث‌بری کرد | می‌توان ارث‌بری کرد |
| **Interface** | پیاده‌سازی نمی‌کند | می‌تواند پیاده‌سازی کند |
| **دسترسی** | Internal | Public/Internal/Private |
| **کاربرد** | موقت و محلی | دائمی و قابل استفاده مجدد |
| **Performance** | کمی سریع‌تر (بهینه‌سازی شده) | معمولی |

### مثال مقایسه:

```csharp
// Anonymous Type
var person1 = new { Name = "Ali", Age = 25 };
// person1.Name = "Reza"; // ❌ خطا - فقط خواندنی

// Class معمولی
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

var person2 = new Person { Name = "Ali", Age = 25 };
person2.Name = "Reza"; // ✅ مجاز
```

---

## مثال‌های ساده

### مثال ۱: ایجاد یک شیء ساده

```csharp
var product = new 
{ 
    Name = "Laptop", 
    Price = 1200.50, 
    InStock = true 
};

Console.WriteLine($"Product: {product.Name}");
Console.WriteLine($"Price: ${product.Price}");
Console.WriteLine($"Available: {product.InStock}");
```

### مثال ۲: استفاده در LINQ - انتخاب فیلدهای خاص

```csharp
var students = new List<Student>
{
    new Student { Id = 1, Name = "Ali", Grade = 18, City = "Tehran" },
    new Student { Id = 2, Name = "Sara", Grade = 19, City = "Isfahan" },
    new Student { Id = 3, Name = "Reza", Grade = 17, City = "Tehran" }
};

// فقط نام و نمره را انتخاب می‌کنیم
var studentNames = students.Select(s => new 
{ 
    s.Name, 
    s.Grade 
});

foreach (var student in studentNames)
{
    Console.WriteLine($"{student.Name}: {student.Grade}");
}
```

### مثال ۳: گروه‌بندی داده‌ها

```csharp
var orders = new List<Order>
{
    new Order { CustomerId = 1, Product = "Laptop", Amount = 1200 },
    new Order { CustomerId = 2, Product = "Mouse", Amount = 25 },
    new Order { CustomerId = 1, Product = "Keyboard", Amount = 50 }
};

// محاسبه مجموع خرید هر مشتری
var customerTotals = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new 
    { 
        CustomerId = g.Key, 
        TotalAmount = g.Sum(o => o.Amount),
        OrderCount = g.Count()
    });

foreach (var customer in customerTotals)
{
    Console.WriteLine($"Customer {customer.CustomerId}: " +
                     $"${customer.TotalAmount} ({customer.OrderCount} orders)");
}
```

---

## مثال‌های واقعی‌تر

### مثال ۱: ترکیب داده‌ها از چند منبع

```csharp
var employees = new List<Employee>
{
    new Employee { Id = 1, Name = "Ali", DepartmentId = 101 },
    new Employee { Id = 2, Name = "Sara", DepartmentId = 102 }
};

var departments = new List<Department>
{
    new Department { Id = 101, Name = "IT" },
    new Department { Id = 102, Name = "HR" }
};

// ترکیب اطلاعات کارمند و دپارتمان
var employeeDetails = employees
    .Join(departments, 
          e => e.DepartmentId, 
          d => d.Id, 
          (e, d) => new 
          { 
              EmployeeName = e.Name, 
              DepartmentName = d.Name,
              EmployeeId = e.Id
          });

foreach (var detail in employeeDetails)
{
    Console.WriteLine($"{detail.EmployeeName} works in {detail.DepartmentName}");
}
```

### مثال ۲: ساخت DTO موقت برای API Response

```csharp
public async Task<IActionResult> GetUsers()
{
    var users = await _userService.GetAllUsersAsync();
    
    // ایجاد ساختار موقت برای response
    var response = users.Select(u => new 
    { 
        u.Id, 
        FullName = $"{u.FirstName} {u.LastName}",
        u.Email,
        IsActive = u.Status == UserStatus.Active,
        RegisteredDays = (DateTime.Now - u.RegisteredDate).Days
    });
    
    return Ok(response);
}
```

### مثال ۳: مرتب‌سازی چند سطحی

```csharp
var products = new List<Product>
{
    new Product { Name = "Laptop", Category = "Electronics", Price = 1200 },
    new Product { Name = "Phone", Category = "Electronics", Price = 800 },
    new Product { Name = "Shirt", Category = "Clothing", Price = 50 }
};

var sortedProducts = products
    .OrderBy(p => p.Category)
    .ThenByDescending(p => p.Price)
    .Select(p => new 
    { 
        p.Name, 
        p.Category, 
        p.Price,
        PriceRange = p.Price > 1000 ? "High" : p.Price > 100 ? "Medium" : "Low"
    });

foreach (var product in sortedProducts)
{
    Console.WriteLine($"{product.Name} ({product.Category}) - " +
                     $"${product.Price} [{product.PriceRange}]");
}
```

### مثال ۴: استفاده در Unit Testing

```csharp
[Fact]
public void CalculateDiscount_ShouldReturnCorrectValues()
{
    // Arrange
    var testCases = new[]
    {
        new { OriginalPrice = 100m, DiscountPercent = 10, ExpectedFinal = 90m },
        new { OriginalPrice = 200m, DiscountPercent = 20, ExpectedFinal = 160m },
        new { OriginalPrice = 50m, DiscountPercent = 5, ExpectedFinal = 47.5m }
    };

    // Act & Assert
    foreach (var testCase in testCases)
    {
        var result = _calculator.CalculateDiscount(
            testCase.OriginalPrice, 
            testCase.DiscountPercent);
        
        Assert.Equal(testCase.ExpectedFinal, result);
    }
}
```

---

## اشتباهات رایج مبتدیان

### ❌ اشتباه ۱: تلاش برای تغییر مقدار property

```csharp
var person = new { Name = "Ali", Age = 25 };
person.Name = "Reza"; // ❌ خطای کامپایل!
// Error: Property or indexer 'AnonymousType.Name' cannot be assigned to -- it is read only
```

**راه‌حل:** Anonymous Type فقط خواندنی است. اگر نیاز به تغییر دارید، از کلاس معمولی یا record استفاده کنید.

### ❌ اشتباه ۲: تلاش برای بازگشت Anonymous Type از متد

```csharp
public var GetPerson() // ❌ خطا!
{
    return new { Name = "Ali", Age = 25 };
}
```

**راه‌حل:** از `object`، `dynamic`، یا Tuple استفاده کنید:

```csharp
public object GetPerson()
{
    return new { Name = "Ali", Age = 25 };
}
```

### ❌ اشتباه ۳: مقایسه دو Anonymous Type مختلف

```csharp
var person1 = new { Name = "Ali", Age = 25 };
var person2 = new { Name = "Ali", Age = 25, City = "Tehran" };

Console.WriteLine(person1.Equals(person2)); // False!
```

**توضیح:** دو Anonymous Type فقط در صورتی برابر هستند که **نام، نوع و ترتیب propertyها** یکسان باشد.

### ❌ اشتباه ۴: استفاده از Anonymous Type در سطح کلاس

```csharp
class MyClass
{
    // ❌ خطا - Anonymous Type فقط در scope محلی قابل استفاده است
    private var _person = new { Name = "Ali", Age = 25 };
}
```

**توضیح:** Anonymous Type فقط در scope محلی (داخل متد) قابل استفاده است.

### ❌ اشتباه ۵: تلاش برای پیاده‌سازی Interface

```csharp
var person = new { Name = "Ali", Age = 25 };
IPerson iPerson = person; // ❌ خطا!
```

**توضیح:** Anonymous Type نمی‌تواند Interface پیاده‌سازی کند.

### ❌ اشتباه ۶: استفاده نادرست در LINQ to Entities

```csharp
// ❌ ممکن است مشکل ایجاد کند
var result = context.Users
    .Select(u => new { u.Name, u.Age })
    .ToList()
    .Where(x => x.Age > 18); // فیلتر بعد از ToList - performance issue!

// ✅ صحیح
var result = context.Users
    .Where(u => u.Age > 18)
    .Select(u => new { u.Name, u.Age })
    .ToList();
```

---

## نکات مهم برای برنامه‌نویسی حرفه‌ای

### ✅ نکته ۱: استفاده از Type Inference

```csharp
// ✅ خوب - استفاده از نام property خودکار
string name = "Ali";
int age = 25;
var person = new { name, age };

// ❌ غیرضروری - تکرار نام
var person = new { name = name, age = age };
```

### ✅ نکته ۲: استفاده در Projectionهای پیچیده

```csharp
var report = salesData
    .GroupBy(s => s.Region)
    .Select(g => new 
    { 
        Region = g.Key,
        TotalSales = g.Sum(s => s.Amount),
        AverageSale = g.Average(s => s.Amount),
        TopProduct = g.OrderByDescending(s => s.Amount)
                      .Select(s => s.Product)
                      .FirstOrDefault(),
        SalesCount = g.Count()
    })
    .OrderByDescending(r => r.TotalSales);
```

### ✅ نکته ۳: استفاده از Tuple به جای Anonymous Type در C# 7.0+

```csharp
// C# 7.0+ - Tuple می‌تواند جایگزین مناسبی باشد
var person1 = new { Name = "Ali", Age = 25 }; // Anonymous Type
var person2 = (Name: "Ali", Age: 25); // Tuple

// Tuple مزایا:
// - می‌تواند به عنوان نوع بازگشتی استفاده شود
// - قابل تغییر است (اگر نیاز باشد)
// - syntax ساده‌تر

// Anonymous Type مزایا:
// - پیاده‌سازی بهتر Equals و GetHashCode
// - مناسب‌تر برای LINQ
// - سازگاری با کدهای قدیمی‌تر
```

### ✅ نکته ۴: استفاده از `var` برای Anonymous Type

```csharp
// ✅ همیشه از var استفاده کنید
var person = new { Name = "Ali", Age = 25 };

// ❌ هرگز سعی نکنید نوع را مشخص کنید
// <>f__AnonymousType0<string, int> person = ... // غیرممکن!
```

### ✅ نکته ۵: توجه به Performance

```csharp
// ✅ خوب - Anonymous Type در LINQ بهینه است
var result = data.Select(x => new { x.Name, x.Age });

// ⚠️ توجه - در حلقه‌های تکراری، Anonymous Type ممکن است overhead داشته باشد
for (int i = 0; i < 1000000; i++)
{
    var temp = new { Index = i, Value = data[i] }; // ایجاد شیء جدید در هر تکرار
}
```

### ✅ نکته ۶: استفاده در Pattern Matching (C# 9.0+)

```csharp
var person = new { Name = "Ali", Age = 25 };

if (person is { Name: "Ali", Age: > 18 })
{
    Console.WriteLine("Adult Ali");
}

// C# 9.0+ - استفاده از target-typed new
PersonInfo info = new("Ali", 25); // اگر constructor داشته باشد
```

### ✅ نکته ۷: مستندسازی کد

```csharp
// ✅ کامنت‌گذاری برای clarity
var userStats = users
    .GroupBy(u => u.Status)
    .Select(g => new 
    { 
        Status = g.Key,
        Count = g.Count(),
        Percentage = (double)g.Count() / users.Count * 100
    })
    .ToList(); // لیست آمار کاربران بر اساس وضعیت
```

---

## چه زمانی استفاده کنیم و چه زمانی نکنیم؟

### ✅ زمان‌های مناسب برای استفاده:

1. **LINQ Queries**: وقتی نیاز به projection یا transformation داده‌ها دارید
   ```csharp
   var result = data.Select(x => new { x.Name, x.Value });
   ```

2. **گروه‌بندی داده‌ها**: برای ایجاد ساختارهای موقت
   ```csharp
   var grouped = data.GroupBy(x => x.Category)
                     .Select(g => new { Category = g.Key, Items = g });
   ```

3. **Unit Testing**: برای ایجاد test data
   ```csharp
   var testCases = new[] { new { Input = 5, Expected = 25 } };
   ```

4. **DTOهای موقت**: برای انتقال داده در scope محلی
   ```csharp
   var tempData = new { UserId = 1, Action = "Login" };
   ```

5. **API Responses**: برای ساخت response ساختاریافته
   ```csharp
   return Ok(new { Success = true, Data = result });
   ```

### ❌ زمان‌های نامناسب برای استفاده:

1. **وقتی نیاز به تغییر داده‌ها دارید**
   ```csharp
   // ❌ Anonymous Type فقط خواندنی است
   var person = new { Name = "Ali" };
   person.Name = "Reza"; // خطا!
   ```

2. **وقتی نیاز به بازگشت از متد دارید**
   ```csharp
   // ❌ نمی‌توان Anonymous Type را به‌صورت مستقیم برگرداند
   public ??? GetPerson() { return new { Name = "Ali" }; }
   ```

3. **وقتی نیاز به ارث‌بری یا Interface دارید**
   ```csharp
   // ❌ Anonymous Type نمی‌تواند Interface پیاده‌سازی کند
   IPerson person = new { Name = "Ali" }; // خطا!
   ```

4. **وقتی داده‌ها باید در سطح کلاس ذخیره شوند**
   ```csharp
   // ❌ Anonymous Type فقط در scope محلی است
   class MyClass { private var _data = new { Value = 1 }; } // خطا!
   ```

5. **وقتی نیاز به استفاده مجدد از نوع دارید**
   ```csharp
   // ❌ اگر چندین جا نیاز به همان ساختار دارید، کلاس تعریف کنید
   var person1 = new { Name = "Ali", Age = 25 };
   var person2 = new { Name = "Sara", Age = 30 }; // نوع متفاوت!
   ```

6. **در Performance-Critical Code**
   ```csharp
   // ❌ در حلقه‌های با تکرار بالا، overhead ایجاد شیء
   for (int i = 0; i < 1000000; i++)
   {
       var temp = new { Index = i }; // overhead
   }
   ```

---

## جمع‌بندی

### نکات کلیدی:

1. **Anonymous Type** یک نوع داده‌ای موقت و فقط خواندنی است که توسط کامپایلر ایجاد می‌شود
2. **کاربرد اصلی**: LINQ queries، projection داده‌ها، و ساختارهای موقت
3. **از دید CLR**: یک کلاس واقعی با propertyهای فقط خواندنی
4. **محدودیت‌ها**: فقط خواندنی، عدم امکان بازگشت مستقیم، عدم ارث‌بری
5. **جایگزین‌ها**: در C# 7.0+ می‌توانید از Tuple یا Record استفاده کنید

### بهترین شیوه‌ها:

- ✅ از Anonymous Type برای LINQ و عملیات‌های موقت استفاده کنید
- ✅ همیشه از `var` برای تعریف Anonymous Type استفاده کنید
- ✅ برای داده‌های دائمی یا قابل تغییر، از کلاس یا record استفاده کنید
- ✅ به performance توجه کنید - در حلقه‌های تکراری ممکن است overhead داشته باشد
- ✅ کد را مستند کنید تا هدف از استفاده Anonymous Type واضح باشد

### مقایسه سریع با جایگزین‌ها:

| ویژگی | Anonymous Type | Tuple | Record | Class |
|--------|----------------|-------|--------|-------|
| **فقط خواندنی** | ✅ | ❌ | ✅ | ❌ |
| **بازگشت از متد** | ❌ | ✅ | ✅ | ✅ |
| **LINQ-friendly** | ✅ | ⚠️ | ✅ | ✅ |
| **Equals/GetHashCode** | ✅ خودکار | ✅ خودکار | ✅ خودکار | ❌ دستی |
| **کاربرد** | موقت/محلی | ساده/موقت | DTO/Value | عمومی |

---

## منابع و مطالعه بیشتر

### منابع رسمی مایکروسافت:

**منبع ۱: Microsoft Learn - Anonymous Types**
- **توضیح**: مستندات رسمی مایکروسافت درباره Anonymous Types در C#
- **لینک**: https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types

**منبع ۲: C# Programming Guide - Anonymous Types**
- **توضیح**: راهنمای جامع برنامه‌نویسی C# با تمرکز بر Anonymous Types
- **لینک**: https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types

**منبع ۳: C# Language Specification**
- **توضیح**: مشخصات رسمی زبان C# شامل بخش مربوط به Anonymous Types
- **لینک**: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/introduction

**منبع ۴: LINQ and Anonymous Types**
- **توضیح**: نحوه استفاده از Anonymous Types در LINQ queries
- **لینک**: https://learn.microsoft.com/en-us/dotnet/csharp/linq/query-expression-basics

### منابع تکمیلی:

**منبع ۵: .NET API Browser**
- **توضیح**: مستندات APIهای .NET مرتبط با Anonymous Types
- **لینک**: https://learn.microsoft.com/en-us/dotnet/api/

**منبع ۶: C# 9.0 Records**
- **توضیح**: معرفی Record types به عنوان جایگزین مدرن برای Anonymous Types
- **لینک**: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

---

**نویسنده**: [نام شما]  
**تاریخ**: August 27, 2026  
**نسخه C#**: C# 3.0 تا C# 12.0  
**سطح**: مقدماتی تا پیشرفته

---

*این مقاله بخشی از یک Repository آموزشی درباره C# است. برای مشاهده سایر مباحث، به صفحه اصلی Repository مراجعه کنید.*