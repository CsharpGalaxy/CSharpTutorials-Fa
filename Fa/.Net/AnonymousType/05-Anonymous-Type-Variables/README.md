# ویژگی‌های Anonymous Type در C#

> 📘 **مقاله آموزشی جامع** — از مفاهیم پایه تا جزئیات فنی پیشرفته

---

## فهرست مطالب

- [مقدمه](#مقدمه)
- [Anonymous Type چیست؟](#anonymous-type-چیست)
- [ساختار و سینتکس پایه](#ساختار-و-سینتکس-پایه)
- [ویژگی‌های کلیدی Anonymous Type](#ویژگیهای-کلیدی-anonymous-type)
  - [Immutable بودن](#immutable-بودن)
  - [Read-only بودن Propertyها](#read-only-بودن-propertyها)
  - [تفاوت Immutable با Read-only](#تفاوت-immutable-با-read-only)
- [Anonymous Type از دید کامپایلر](#anonymous-type-از-دید-کامپایلر)
  - [internal بودن Type](#internal-بودن-type)
  - [sealed بودن Type](#sealed-بودن-type)
  - [Reference Type بودن](#reference-type-بودن)
  - [رابطه با object](#رابطه-با-object)
- [متدهای Overridden](#متدهای-overridden)
  - [Equals و Value-based Equality](#equals-و-value-based-equality)
  - [GetHashCode](#gethashcode)
  - [ToString](#tostring)
- [Nested Anonymous Types](#nested-anonymous-types)
- [with Expression در Anonymous Types](#with-expression-در-anonymous-types)
- [بررسی رفتار در Runtime](#بررسی-رفتار-در-runtime)
- [مثال‌های کاربردی](#مثالهای-کاربردی)
- [اشتباهات رایج](#اشتباهات-رایج)
- [نکات مهم](#نکات-مهم)
- [جمع‌بندی](#جمع‌بندی)
- [منابع معتبر](#منابع-معتبر)

---

## مقدمه

در زبان C#، گاهی اوقات نیاز داریم یک شیء موقت بسازیم که فقط برای انتقال داده استفاده شود و نیازی به تعریف یک کلاس اختصاصی برای آن نداشته باشیم. در این شرایط، **Anonymous Type** یک راه‌حل قدرتمند و تمیز ارائه می‌دهد.

در این مقاله، از مفاهیم پایه شروع می‌کنیم و سپس به بررسی دقیق ویژگی‌های فنی آن از دید کامپایلر و Runtime می‌پردازیم.

---

## Anonymous Type چیست؟

**Anonymous Type** (نوع ناشناس) یک نوع کلاس است که:

- توسط **کامپایلر** به‌صورت خودکار ساخته می‌شود
- **نام مشخصی** ندارد (نام آن توسط کامپایلر تولید می‌شود)
- فقط شامل **Propertyهای Read-only** است
- برای **گروه‌بندی داده‌های موقت** استفاده می‌شود
- معمولاً در کوئری‌های **LINQ** کاربرد دارد

> 💡 **نکته کلیدی:** شما به‌عنوان برنامه‌نویس، کلاس را نمی‌نویسید؛ کامپایلر این کار را برای شما انجام می‌دهد.

---

## ساختار و سینتکس پایه

برای ساخت یک Anonymous Type از کلمه کلیدی `var` به همراه `new` و یک **Object Initializer** استفاده می‌کنیم:

```csharp
var person = new { Name = "Ali", Age = 30 };

Console.WriteLine(person.Name); // خروجی: Ali
Console.WriteLine(person.Age);  // خروجی: 30
```

### سینتکس‌های مختلف

```csharp
// 1. مقداردهی مستقیم
var product = new { Id = 1, Title = "Laptop", Price = 1200.50 };

// 2. استفاده از متغیرهای موجود (نام Property = نام متغیر)
string name = "Sara";
int age = 25;
var user = new { name, age };
// معادل است با: new { name = "Sara", age = 25 }

// 3. نام‌گذاری صریح Property
var point = new { X = 10, Y = 20 };

// 4. استفاده از Expression
var square = new { Side = 5, Area = 5 * 5 };
```

> ⚠️ **توجه:** حتماً باید از `var` استفاده کنید، چون نام نوع توسط کامپایلر تولید شده و شما آن را نمی‌دانید.

---

## ویژگی‌های کلیدی Anonymous Type

### Immutable بودن

Anonymous Type کاملاً **Immutable** (غیرقابل تغییر) است. این یعنی:

- پس از ساخت یک نمونه، **نمی‌توانید** هیچ Property آن را تغییر دهید
- نمی‌توانید Property جدید اضافه یا حذف کنید
- ساختار شیء ثابت باقی می‌ماند

```csharp
var person = new { Name = "Ali", Age = 30 };

// ❌ خطای کامپایل - Propertyها فقط خواندنی هستند
person.Name = "Reza";  // CS0200: Property or indexer cannot be assigned to
person.Age = 35;       // CS0200
```

### Read-only بودن Propertyها

تمام Propertyهای یک Anonymous Type به‌صورت **Read-only** تولید می‌شوند. در کد تولیدشده توسط کامپایلر، این Propertyها فقط **getter** دارند و **setter** ندارند:

```csharp
// کد معادل تولیدشده توسط کامپایلر (به‌صورت ساده‌شده):
internal sealed class <>f__AnonymousType0<string Name, int Age>
{
    private readonly string _Name;
    private readonly int _Age;

    public string Name => _Name;  // فقط getter
    public int Age => _Age;       // فقط getter

    public <>f__AnonymousType0(string Name, int Age)
    {
        _Name = Name;
        _Age = Age;
    }
}
```

### تفاوت Immutable با Read-only

این دو مفهوم اغلب اشتباه گرفته می‌شوند:

| ویژگی | Read-only | Immutable |
|-------|-----------|-----------|
| **تعریف** | مقدار قابل تغییر نیست | کل شیء قابل تغییر نیست |
| **سطح** | فقط Property | کل ساختار |
| **Anonymous Type** | ✅ Propertyها read-only هستند | ✅ کل شیء immutable است |

> 🔍 **توضیح:** در Anonymous Type، Propertyها **Read-only** هستند (setter ندارند) و چون هیچ راهی برای تغییر مقادیر وجود ندارد، کل شیء **Immutable** محسوب می‌شود.

**مثال برای درک تفاوت:**

```csharp
// یک کلاس با Propertyهای read-only ولی نه لزوماً immutable
public class Person
{
    public string Name { get; }  // read-only
    public List<string> Hobbies { get; }  // read-only

    public Person(string name)
    {
        Name = name;
        Hobbies = new List<string>();
    }
}

var p = new Person("Ali");
// p.Name = "Reza";  // ❌ خطا - Property read-only است
p.Hobbies.Add("Football");  // ✅ مجاز! - محتوای لیست قابل تغییر است

// پس این کلاس read-only است ولی immutable نیست!
```

در Anonymous Type، چون Propertyها از انواع **Value Type** یا **Immutable Reference Type** (مثل string) هستند، کل شیء **Immutable** واقعی است.

---

## Anonymous Type از دید کامپایلر

وقتی شما یک Anonymous Type می‌نویسید، کامپایلر C# یک کلاس کامل برای شما تولید می‌کند. بیایید این کلاس را بررسی کنیم:

### internal بودن Type

کلاس تولیدشده **internal** است، یعنی:

- فقط در **همان Assembly** قابل دسترسی است
- نمی‌تواند از یک Assembly به Assembly دیگر منتقل شود
- به‌عنوان نوع بازگشتی متدهای public قابل استفاده نیست

```csharp
// در Assembly A:
public object GetPerson()
{
    return new { Name = "Ali", Age = 30 };
}

// در Assembly B:
var person = GetPerson();
// ❌ نمی‌توانید به Propertyها دسترسی مستقیم داشته باشید
// چون نوع Anonymous Type در Assembly B وجود ندارد
```

> 💡 **راه‌حل:** اگر نیاز به عبور از مرز Assembly دارید، از **Record** یا **کلاس معمولی** استفاده کنید.

### sealed بودن Type

کلاس تولیدشده **sealed** است:

- نمی‌توان از آن **ارث‌بری** کرد
- برای بهینه‌سازی عملکرد و جلوگیری از تغییر ساختار

```csharp
// ❌ این کار ممکن نیست
class MyType : /* Anonymous Type */  // کامپایلر اجازه نمی‌دهد
{
}
```

### Reference Type بودن

Anonymous Type یک **Reference Type** است (نه Value Type):

- روی **Heap** ذخیره می‌شود
- متغیرها **Reference** به شیء را نگه می‌دارند
- مقایسه با `==` به‌صورت **Reference Equality** انجام می‌شود (مگر اینکه `Equals` را override کند)

```csharp
var p1 = new { Name = "Ali", Age = 30 };
var p2 = new { Name = "Ali", Age = 30 };

Console.WriteLine(p1 == p2);        // False - Reference متفاوت
Console.WriteLine(p1.Equals(p2));   // True - Value-based equality
Console.WriteLine(object.ReferenceEquals(p1, p2)); // False
```

### رابطه با object

هر Anonymous Type به‌صورت مستقیم از **`object`** ارث‌بری می‌کند:

```csharp
// کد تولیدشده توسط کامپایلر:
internal sealed class <>f__AnonymousType0<T1, T2> : object
{
    // ...
}
```

این یعنی:
- می‌توانید آن را به `object` تبدیل کنید
- متدهای `Equals`، `GetHashCode` و `ToString` را **Override** می‌کند
- هیچ Interface خاصی را پیاده‌سازی نمی‌کند

```csharp
var person = new { Name = "Ali", Age = 30 };
object obj = person;  // ✅ مجاز - Implicit casting به object

// ❌ نمی‌توانید به Propertyها دسترسی داشته باشید
// Console.WriteLine(obj.Name);  // خطای کامپایل
```

---

## متدهای Overridden

کامپایلر سه متد مهم را برای Anonymous Type Override می‌کند:

### Equals و Value-based Equality

متد `Equals` بر اساس **مقدار Propertyها** مقایسه می‌کند (نه Reference):

```csharp
var p1 = new { Name = "Ali", Age = 30 };
var p2 = new { Name = "Ali", Age = 30 };
var p3 = new { Name = "Sara", Age = 25 };

Console.WriteLine(p1.Equals(p2));  // True - مقادیر یکسان
Console.WriteLine(p1.Equals(p3));  // False - مقادیر متفاوت
```

**قوانین Value-based Equality:**

1. **تعداد Propertyها** باید یکسان باشد
2. **نام Propertyها** باید یکسان باشد
3. **نوع Propertyها** باید یکسان باشد
4. **ترتیب Propertyها** مهم است
5. **مقدار Propertyها** باید برابر باشد

```csharp
var a = new { X = 1, Y = 2 };
var b = new { X = 1, Y = 2 };
var c = new { Y = 2, X = 1 };  // ❌ ترتیب متفاوت
var d = new { X = 1, Y = 2, Z = 3 };  // ❌ تعداد متفاوت

Console.WriteLine(a.Equals(b));  // True
Console.WriteLine(a.Equals(c));  // False - نوع متفاوت تولید می‌شود
Console.WriteLine(a.Equals(d));  // False
```

### GetHashCode

متد `GetHashCode` بر اساس **مقدار Propertyها** محاسبه می‌شود:

```csharp
var p1 = new { Name = "Ali", Age = 30 };
var p2 = new { Name = "Ali", Age = 30 };

Console.WriteLine(p1.GetHashCode());  // مثلاً: 12345678
Console.WriteLine(p2.GetHashCode());  // 12345678 - یکسان!

// استفاده در Dictionary و HashSet
var dict = new Dictionary<object, string>();
dict[p1] = "First";
dict[p2] = "Second";  // مقدار قبلی را Overwrite می‌کند

Console.WriteLine(dict.Count);  // 1 - چون Hash Code یکسان است
```

**پیاده‌سازی GetHashCode توسط کامپایلر:**

```csharp
// کد معادل (ساده‌شده):
public override int GetHashCode()
{
    int hash = 17;
    hash = hash * 31 + (Name?.GetHashCode() ?? 0);
    hash = hash * 31 + Age.GetHashCode();
    return hash;
}
```

### ToString

متد `ToString` یک **نمایش متنی** از Propertyها برمی‌گرداند:

```csharp
var person = new { Name = "Ali", Age = 30 };
Console.WriteLine(person.ToString());
// خروجی: { Name = Ali, Age = 30 }

var point = new { X = 10, Y = 20 };
Console.WriteLine(point);  // ToString به‌صورت خودکار فراخوانی می‌شود
// خروجی: { X = 10, Y = 20 }
```

---

## Nested Anonymous Types

می‌توانید Anonymous Typeهای **تودرتو** بسازید:

```csharp
var employee = new
{
    Name = "Ali",
    Department = new
    {
        Title = "IT",
        Floor = 3
    },
    Skills = new[] { "C#", "SQL", "Azure" }
};

Console.WriteLine(employee.Name);              // Ali
Console.WriteLine(employee.Department.Title);  // IT
Console.WriteLine(employee.Skills[0]);         // C#
```

**نکته مهم:** هر سطح از تودرتویی، یک Anonymous Type جداگانه تولید می‌کند.

---

## with Expression در Anonymous Types

از **C# 14** (منتشرشده با .NET 9 در نوامبر 2024)، می‌توانید از **`with` expression** برای Anonymous Typeها استفاده کنید. این ویژگی امکان ساخت یک کپی با تغییرات را فراهم می‌کند:

```csharp
var original = new { Name = "Ali", Age = 30 };

// ساخت یک کپی با تغییر Age
var modified = original with { Age = 35 };

Console.WriteLine(original.Name);  // Ali
Console.WriteLine(original.Age);   // 30
Console.WriteLine(modified.Name);  // Ali
Console.WriteLine(modified.Age);   // 35 - تغییر کرده
```

### چندین تغییر همزمان

```csharp
var point1 = new { X = 10, Y = 20 };
var point2 = point1 with { X = 100, Y = 200 };

Console.WriteLine(point1);  // { X = 10, Y = 20 }
Console.WriteLine(point2);  // { X = 100, Y = 200 }
```

> 💡 **نکته:** `with` expression یک **شیء جدید** می‌سازد و شیء اصلی را تغییر نمی‌دهد (Immutable باقی می‌ماند).

---

## بررسی رفتار در Runtime

بیایید با استفاده از **Reflection** رفتار Anonymous Type را در Runtime بررسی کنیم:

```csharp
using System;
using System.Reflection;

var person = new { Name = "Ali", Age = 30 };
Type type = person.GetType();

Console.WriteLine($"Type Name: {type.Name}");
Console.WriteLine($"Full Name: {type.FullName}");
Console.WriteLine($"Is Class: {type.IsClass}");
Console.WriteLine($"Is Sealed: {type.IsSealed}");
Console.WriteLine($"Is Public: {type.IsPublic}");
Console.WriteLine($"Is Nested: {type.IsNested}");
Console.WriteLine($"Base Type: {type.BaseType?.Name}");

Console.WriteLine("\nProperties:");
foreach (var prop in type.GetProperties())
{
    Console.WriteLine($"  {prop.PropertyType.Name} {prop.Name} " +
                      $"(CanRead: {prop.CanRead}, CanWrite: {prop.CanWrite})");
}
```

**خروجی نمونه:**

```
Type Name: <>f__AnonymousType3`2
Full Name: <>f__AnonymousType3`2[[System.String, ...],[System.Int32, ...]]
Is Class: True
Is Sealed: True
Is Public: False
Is Nested: True
Base Type: Object

Properties:
  String Name (CanRead: True, CanWrite: False)
  Int32 Age (CanRead: True, CanWrite: False)
```

### بررسی متدها

```csharp
var person = new { Name = "Ali", Age = 30 };
Type type = person.GetType();

var equalsMethod = type.GetMethod("Equals", new[] { typeof(object) });
var getHashCodeMethod = type.GetMethod("GetHashCode");
var toStringMethod = type.GetMethod("ToString");

Console.WriteLine($"Equals declared: {equalsMethod.DeclaringType == type}");
Console.WriteLine($"GetHashCode declared: {getHashCodeMethod.DeclaringType == type}");
Console.WriteLine($"ToString declared: {toStringMethod.DeclaringType == type}");

// خروجی: همه True - یعنی Override شده‌اند
```

---

## مثال‌های کاربردی

### مثال 1: استفاده در LINQ

```csharp
var students = new[]
{
    new { Name = "Ali", Grade = 18 },
    new { Name = "Sara", Grade = 19 },
    new { Name = "Reza", Grade = 17 }
};

var topStudents = students
    .Where(s => s.Grade >= 18)
    .Select(s => new { s.Name, Status = "Excellent" });

foreach (var student in topStudents)
{
    Console.WriteLine($"{student.Name}: {student.Status}");
}
// خروجی:
// Ali: Excellent
// Sara: Excellent
```

### مثال 2: گروه‌بندی داده‌ها

```csharp
var orders = new[]
{
    new { Product = "Laptop", Category = "Electronics", Price = 1200 },
    new { Product = "Phone", Category = "Electronics", Price = 800 },
    new { Product = "Shirt", Category = "Clothing", Price = 50 }
};

var summary = orders
    .GroupBy(o => o.Category)
    .Select(g => new
    {
        Category = g.Key,
        Count = g.Count(),
        TotalPrice = g.Sum(o => o.Price)
    });

foreach (var item in summary)
{
    Console.WriteLine($"{item.Category}: {item.Count} items, ${item.TotalPrice}");
}
```

### مثال 3: بازگرداندن چندین مقدار

```csharp
public (int, int) GetMinMax(int[] numbers)
{
    // روش قدیمی با Anonymous Type (قبل از Tuple)
    var result = new { Min = numbers.Min(), Max = numbers.Max() };
    return (result.Min, result.Max);
}

// روش مدرن با Tuple (توصیه‌شده)
public (int Min, int Max) GetMinMaxModern(int[] numbers)
{
    return (numbers.Min(), numbers.Max());
}
```

### مثال 4: ساخت DTO موقت

```csharp
var users = GetUsersFromDatabase();

var userDtos = users.Select(u => new
{
    u.Id,
    FullName = $"{u.FirstName} {u.LastName}",
    u.Email,
    IsActive = u.Status == "Active"
});

// ارسال به View یا API Response
return Ok(userDtos);
```

---

## اشتباهات رایج

### ❌ اشتباه 1: تلاش برای تغییر Property

```csharp
var person = new { Name = "Ali", Age = 30 };
person.Age = 35;  // ❌ خطای کامپایل

// ✅ راه‌حل: استفاده از with expression (C# 14+)
var updatedPerson = person with { Age = 35 };
```

### ❌ اشتباه 2: استفاده به‌عنوان نوع بازگشتی public

```csharp
public class UserService
{
    // ❌ مشکل: نوع بازگشتی object است
    public object GetUser()
    {
        return new { Name = "Ali", Age = 30 };
    }
}

// ✅ راه‌حل 1: استفاده از Record
public record UserDto(string Name, int Age);
public UserDto GetUser() => new("Ali", 30);

// ✅ راه‌حل 2: استفاده در همان Scope
public void ProcessUser()
{
    var user = new { Name = "Ali", Age = 30 };
    // استفاده محلی
}
```

### ❌ اشتباه 3: مقایسه با ==

```csharp
var p1 = new { Name = "Ali", Age = 30 };
var p2 = new { Name = "Ali", Age = 30 };

if (p1 == p2)  // ❌ مقایسه Reference
{
    // این بلوک اجرا نمی‌شود!
}

// ✅ راه‌حل: استفاده از Equals
if (p1.Equals(p2))  // ✅ مقایسه Value-based
{
    Console.WriteLine("Equal!");
}
```

### ❌ اشتباه 4: استفاده در Cross-Assembly

```csharp
// Assembly A
public class DataProvider
{
    public object GetData() => new { Id = 1, Value = "Test" };
}

// Assembly B
var data = dataProvider.GetData();
// ❌ نمی‌توانید به data.Id دسترسی داشته باشید
```

### ❌ اشتباه 5: ذخیره در Collection با نوع مشخص

```csharp
var list = new List</* چه نوعی؟ */>();
// ❌ نمی‌توانید Anonymous Type را مستقیماً در List ذخیره کنید

// ✅ راه‌حل: استفاده از var و LINQ
var items = Enumerable.Range(1, 5)
    .Select(i => new { Id = i, Value = $"Item {i}" })
    .ToList();  // ✅ List<AnonymousType>
```

---

## نکات مهم

### ✅ نکات کلیدی

1. **Immutable بودن:** Anonymous Type کاملاً Immutable است و نمی‌توان مقادیر آن را تغییر داد
2. **Read-only Properties:** تمام Propertyها فقط getter دارند
3. **Internal و Sealed:** کلاس تولیدشده internal و sealed است
4. **Reference Type:** روی Heap ذخیره می‌شود
5. **Value-based Equality:** متد `Equals` بر اساس مقادیر مقایسه می‌کند
6. **نام تولیدشده:** نام نوع توسط کامپایلر تولید می‌شود (مثل `<>f__AnonymousType0`)
7. **Type Inference:** کامپایلر بر اساس ساختار، نوع را تعیین می‌کند
8. **Scope محدود:** فقط در همان Assembly قابل استفاده است

### 🎯 چه زمانی از Anonymous Type استفاده کنیم؟

- ✅ کوئری‌های LINQ
- ✅ گروه‌بندی داده‌ها
- ✅ ساخت DTO موقت در Scope محلی
- ✅ انتقال داده در یک متد

### 🚫 چه زمانی از Anonymous Type استفاده نکنیم؟

- ❌ عبور از مرز Assembly
- ❌ نیاز به تغییر مقادیر
- ❌ ذخیره‌سازی طولانی‌مدت
- ❌ استفاده در API عمومی
- ❌ نیاز به Serialization پیچیده

### 🔄 جایگزین‌های مدرن

```csharp
// 1. Record (C# 9+) - توصیه‌شده
public record Person(string Name, int Age);
var p = new Person("Ali", 30);

// 2. Tuple (C# 7+)
var point = (X: 10, Y: 20);

// 3. Class/Struct معمولی
public class Product
{
    public int Id { get; init; }
    public string Name { get; init; }
}
```

---

## جمع‌بندی

**Anonymous Type** یک ویژگی قدرتمند در C# است که:

- توسط **کامپایلر** به‌صورت خودکار ساخته می‌شود
- **Immutable** و **Read-only** است
- یک **Reference Type** داخلی و **sealed** است
- متدهای `Equals`، `GetHashCode` و `ToString` را **Override** می‌کند
- از **Value-based Equality** پشتیبانی می‌کند
- در **LINQ** و کوئری‌های موقت بسیار مفید است
- از **C# 14** با `with` expression قابل کپی و تغییر است

**نکات پایانی:**

- برای استفاده‌های محلی و موقت عالی است
- برای عبور از مرز Assembly مناسب نیست
- در پروژه‌های مدرن، **Record** جایگزین بهتری است
- همیشه به **Immutable بودن** آن توجه کنید

---

## منابع معتبر

1. **Microsoft Learn - Anonymous Types**  
   https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types

2. **C# Language Specification - Anonymous Object Creation Expressions**  
   https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128155-anonymous-object-creation-expressions

3. **C# Programming Guide - Anonymous Types**  
   https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types

4. **ECMA-334 C# Language Specification**  
   Section 12.8.15.5 - Anonymous object creation expressions

5. **Roslyn Source Code - Anonymous Type Generation**  
   https://github.com/dotnet/roslyn

6. **C# 14 Features - with expression for anonymous types**  
   https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-14

---

> 📝 **نویسنده:** این مقاله برای Repository آموزشی C# تهیه شده است.  
> 📅 **آخرین به‌روزرسانی:** August 2026  
> 🎯 **سطح:** مبتدی تا پیشرفته

---

**🔗 مقالات مرتبط:**
- [Record Types در C#](./record-types.md)
- [LINQ Query Syntax](./linq-basics.md)
- [Value Types vs Reference Types](./type-system.md)