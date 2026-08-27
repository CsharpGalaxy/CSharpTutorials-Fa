# مقاله آموزشی: `TupleElementNamesAttribute` در C#

## مقدمه

از نسخه C# 7.0، زبان C# از **Tupleها** به‌عنوان یک ویژگی سطح زبان (Language-level feature) پشتیبانی می‌کند. وقتی شما یک Tuple می‌نویسید:

```csharp
var person = (Name: "Ali", Age: 30);
```

به نظر می‌رسد که شما یک نوع با فیلدهای `Name` و `Age` ساخته‌اید. اما در پشت صحنه، CLR چنین چیزی نمی‌شناسد. در این مقاله بررسی می‌کنیم که کامپایلر C# چگونه نام‌های منطقی اعضا را حفظ می‌کند و نقش `TupleElementNamesAttribute` در این فرآیند چیست.

---

## ۱. Tuple در C# چیست؟ (پایه)

هر Tuple که شما در C# می‌نویسید، در واقع یک نمونه از یکی از ساختارهای `System.ValueTuple<T1, T2, ...>` است:

```csharp
// چیزی که شما می‌نویسید:
var point = (X: 10, Y: 20);

// چیزی که کامپایلر تولید می‌کند:
ValueTuple<int, int> point = new ValueTuple<int, int>(10, 20);
```

ساختار `ValueTuple<T1, T2>` در .NET به این شکل تعریف شده است:

```csharp
public struct ValueTuple<T1, T2> : IEquatable<ValueTuple<T1, T2>>
{
    public T1 Item1;
    public F2 Item2;

    public ValueTuple(T1 item1, T2 item2)
    {
        Item1 = item1;
        Item2 = item2;
    }
}
```

> **نکته کلیدی:** فیلدهای این ساختار **همیشه** `Item1`، `Item2`، ... نام دارند. نه `X`، نه `Y`، نه `Name`، نه `Age`.

---

## ۲. چرا نام‌های Tuple به عنوان Field Name ذخیره نمی‌شوند؟

این سؤال مهمی است. چرا کامپایلر یک نوع جدید با فیلدهای `X` و `Y` تولید نمی‌کند؟

### دلایل اصلی:

**الف) انواع بی‌نهایت:**
اگر قرار بود برای هر ترکیب نام، یک نوع جدید ساخته شود، تعداد انواع مورد نیاز بی‌نهایت می‌شد. `(X: int, Y: int)` و `(Width: int, Height: int)` هر دو از نظر نوع یکسان هستند: `ValueTuple<int, int>`.

**ب) سازگاری با CLR:**
CLR فقط انواع تعریف‌شده در metadata را می‌شناسد. ساختن انواع پویا (Dynamic Types) در سطح CLR پیچیدگی زیادی دارد و با مدل نوعی (Type System) دات‌نت سازگار نیست.

**ج) Boxing و Performance:**
`ValueTuple` یک `struct` است (Value Type). اگر قرار بود انواع جدید ساخته شوند، باید برای هر کدام metadata جداگانه تعریف می‌شد و این باعث افزایش حجم Assembly و کاهش Performance می‌شد.

**د) Tuple Equality و Conversion:**
دو Tuple با نام‌های مختلف اما نوع‌های یکسان، از نظر ساختاری برابرند:

```csharp
var a = (X: 1, Y: 2);
var b = (Width: 1, Height: 2);
// a و b از نظر مقدار برابرند و می‌توانند به هم Assign شوند
```

---

## ۳. پس نام‌ها کجا ذخیره می‌شوند؟ (Compile Time)

نام‌هایی که شما به اعضای Tuple می‌دهید (مثل `Name` و `Age`) فقط در **Compile Time** وجود دارند. کامپایلر C# این نام‌ها را در **Metadata** به‌صورت یک Attribute ذخیره می‌کند.

وقتی شما می‌نویسید:

```csharp
public (string Name, int Age) GetPerson()
{
    return ("Ali", 30);
}
```

کامپایلر این کد را به شکل زیر تبدیل می‌کند:

```csharp
[return: TupleElementNames(new string[] { "Name", "Age" })]
public ValueTuple<string, int> GetPerson()
{
    return new ValueTuple<string, int>("Ali", 30);
}
```

---

## ۴. `TupleElementNamesAttribute` چیست؟

این Attribute در namespace `System.Runtime.CompilerServices` تعریف شده است:

```csharp
namespace System.Runtime.CompilerServices
{
    [AttributeUsage(
        AttributeTargets.Field | AttributeTargets.Parameter | 
        AttributeTargets.Property | AttributeTargets.ReturnValue | 
        AttributeTargets.Class | AttributeTargets.Method | 
        AttributeTargets.Event,
        AllowMultiple = false, 
        Inherited = false)]
    public sealed class TupleElementNamesAttribute : Attribute
    {
        public TupleElementNamesAttribute(string[] transformNames)
        {
            TransformNames = transformNames;
        }

        public IList<string> TransformNames { get; }
    }
}
```

### ویژگی‌های مهم:

| ویژگی | توضیح |
|--------|--------|
| **`TransformNames`** | آرایه‌ای از نام‌های منطقی اعضای Tuple به ترتیب |
| **`AllowMultiple = false`** | فقط یک بار روی هر عنصر اعمال می‌شود |
| **Target‌ها** | روی Field، Parameter، Property، ReturnValue، Method و... قابل اعمال است |

---

## ۵. نقش Attribute در Metadata

وقتی کامپایلر Roslyn کد شما را کامپایل می‌کند، اطلاعات زیر در Metadata Assembly ذخیره می‌شود:

```
┌─────────────────────────────────────────────────────────┐
│  Method: GetPerson                                      │
│  ┌───────────────────────────────────────────────────┐  │
│  │ Return Type: ValueTuple<String, Int32>            │  │
│  │ Custom Attribute: TupleElementNamesAttribute      │  │
│  │   TransformNames: ["Name", "Age"]                 │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### چه چیزی در IL باقی می‌ماند؟

```il
.method public hidebysig 
    instance valuetype [System.Runtime]System.ValueTuple`2<string, int32> 
    GetPerson() cil managed
{
    .custom instance void [System.Runtime]
        System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[]) = (
            01 00 08 4E 61 6D 65 03 41 67 65 00 00
        )
    // ... IL code
}
```

> **نکته:** نام‌های `Name` و `Age` به‌صورت رشته در Custom Attribute ذخیره شده‌اند (در hex: `4E 61 6D 65` = "Name" و `41 67 65` = "Age").

---

## ۶. تفاوت نام منطقی عضو با `Item1` و `Item2`

```csharp
var person = (Name: "Ali", Age: 30);

// دسترسی با نام منطقی (Compile Time):
Console.WriteLine(person.Name);  // ✅ کامپایلر می‌داند Name = Item1

// دسترسی با نام واقعی فیلد (همیشه کار می‌کند):
Console.WriteLine(person.Item1); // ✅ "Ali"

// هر دو یک مقدار را برمی‌گردانند
```

### در سطح Reflection:

```csharp
var person = (Name: "Ali", Age: 30);
var type = person.GetType(); // ValueTuple<string, int>

foreach (var field in type.GetFields())
{
    Console.WriteLine(field.Name); 
    // خروجی: Item1, Item2
    // نه Name و Age!
}
```

> **نتیجه مهم:** Reflection نام‌های منطقی را نمی‌بیند. برای دسترسی به نام‌های منطقی باید `TupleElementNamesAttribute` را از Metadata بخوانید.

---

## ۷. Tuple در Return Type

وقتی Tuple به‌عنوان Return Type یک متد استفاده می‌شود، Attribute روی **ReturnValue** اعمال می‌شود:

```csharp
// کد شما:
public (string Name, int Age) GetPerson() => ("Ali", 30);

// معادل IL:
[return: TupleElementNames(new[] { "Name", "Age" })]
public ValueTuple<string, int> GetPerson() => new ValueTuple<string, int>("Ali", 30);
```

### در Parameter:

```csharp
// کد شما:
public void Print((string Name, int Age) person) { }

// معادل IL:
public void Print(
    [TupleElementNames(new[] { "Name", "Age" })] 
    ValueTuple<string, int> person) { }
```

### در Field و Property:

```csharp
// کد شما:
public (int X, int Y) Position;

// معادل IL:
[TupleElementNames(new[] { "X", "Y" })]
public ValueTuple<int, int> Position;
```

---

## ۸. Tuple بین Assemblyها (Cross-Assembly)

این یکی از مهم‌ترین کاربردهای `TupleElementNamesAttribute` است. فرض کنید شما یک Library ساخته‌اید:

### Library (MyLibrary.dll):

```csharp
namespace MyLibrary
{
    public class DataService
    {
        public (string Name, int Age) GetPerson()
        {
            return ("Ali", 30);
        }
    }
}
```

### Application (MyApp.exe):

```csharp
using MyLibrary;

var service = new DataService();
var person = service.GetPerson();

Console.WriteLine(person.Name); // ✅ کار می‌کند!
Console.WriteLine(person.Age);  // ✅ کار می‌کند!
```

### چه اتفاقی می‌افتد؟

1. **در Library:** کامپایلر `TupleElementNamesAttribute` را با نام‌های `["Name", "Age"]` در Metadata متد `GetPerson` ذخیره می‌کند.

2. **در Application:** وقتی کامپایلر MyApp کد `person.Name` را می‌بیند:
   - ابتدا Return Type متد `GetPerson` را بررسی می‌کند → `ValueTuple<string, int>`
   - سپس `TupleElementNamesAttribute` را از Metadata می‌خواند → `["Name", "Age"]`
   - نتیجه می‌گیرد: `Name` = `Item1`
   - کد را به `person.Item1` تبدیل می‌کند

> **بدون این Attribute،** کامپایلر Application نمی‌توانست بداند که `Name` مربوط به کدام `Item` است و خطای Compile می‌داد.

---

## ۹. مثال واقعی: Library و Application

### سناریو:

شما یک NuGet Package ساخته‌اید که یک API برای دریافت اطلاعات کاربر دارد.

**Library (UserApi.dll):**

```csharp
namespace UserApi
{
    public class UserRepository
    {
        public (int Id, string Username, string Email, bool IsActive) 
            GetUser(int userId)
        {
            // شبیه‌سازی دریافت از دیتابیس
            return (userId, $"user_{userId}", $"user{userId}@example.com", true);
        }

        public List<(string ProductName, decimal Price, int Stock)> 
            GetProducts()
        {
            return new List<(string, decimal, int)>
            {
                ("Laptop", 1200.50m, 10),
                ("Mouse", 25.99m, 150),
                ("Keyboard", 75.00m, 80)
            };
        }
    }
}
```

**Metadata تولیدشده در Library (بررسی با ILDasm):**

```
.method public hidebysig 
    instance valuetype [System.Runtime]System.ValueTuple`4<
        int32, string, string, bool> 
    GetUser(int32 userId) cil managed
{
    .custom instance void [System.Runtime]
        System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[]) = (
            01 00 06 49 64 08 55 73 65 72 6E 61 6D 65 
            05 45 6D 61 69 6C 08 49 73 41 63 74 69 76 65 00 00
        )
    // Id, Username, Email, IsActive
}
```

**Application (ConsoleApp.exe):**

```csharp
using UserApi;

var repo = new UserRepository();

// نام‌های منطقی کار می‌کنند!
var user = repo.GetUser(1);
Console.WriteLine($"ID: {user.Id}");           // ✅
Console.WriteLine($"Name: {user.Username}");   // ✅
Console.WriteLine($"Email: {user.Email}");     // ✅
Console.WriteLine($"Active: {user.IsActive}"); // ✅

// نام‌های Item هم کار می‌کنند
Console.WriteLine($"Same as Id: {user.Item1}"); // ✅

// لیست Tuple
var products = repo.GetProducts();
foreach (var product in products)
{
    Console.WriteLine($"{product.ProductName}: ${product.Price}");
    // ✅ نام‌های منطقی در Tupleهای تودرتو هم کار می‌کنند
}
```

---

## ۱۰. Tupleهای تودرتو (Nested Tuples)

وقتی Tupleها تودرتو هستند (بیش از ۷ عضو یا Tuple در Tuple)، `TupleElementNamesAttribute` همه نام‌ها را به‌صورت flat ذخیره می‌کند:

```csharp
// کد شما:
public (int A, int B, int C, int D, int E, int F, int G, int H, int I) 
    GetMany()
{
    return (1, 2, 3, 4, 5, 6, 7, 8, 9);
}
```

### معادل تولیدشده:

```csharp
[return: TupleElementNames(new[] { 
    "A", "B", "C", "D", "E", "F", "G", "H", "I" 
})]
public ValueTuple<int, int, int, int, int, int, int, 
    ValueTuple<int, int>> GetMany()
{
    return new ValueTuple<int, int, int, int, int, int, int, 
        ValueTuple<int, int>>(1, 2, 3, 4, 5, 6, 7, 
            new ValueTuple<int, int>(8, 9));
}
```

> **نکته:** `ValueTuple` فقط تا ۷ عضو مستقیم دارد (`TRest`). برای بیش از ۷ عضو، عضو هشتم به‌عنوان یک `ValueTuple` تودرتو در `TRest` ذخیره می‌شود. اما `TupleElementNamesAttribute` همه ۹ نام را به‌صورت یک آرایه flat نگه می‌دارد.

---

## ۱۱. بررسی نام‌های خالی و پیش‌فرض

اگر نامی برای عضوی مشخص نکنید، کامپایلر آن را خالی (`""`) در Attribute ذخیره می‌کند:

```csharp
public (string Name, int, string Email) GetData()
{
    return ("Ali", 30, "ali@example.com");
}
```

### معادل:

```csharp
[return: TupleElementNames(new[] { "Name", "", "Email" })]
public ValueTuple<string, int, string> GetData()
{
    return new ValueTuple<string, int, string>("Ali", 30, "ali@example.com");
}
```

در این حالت، عضو دوم فقط با `Item2` قابل دسترسی است (نام منطقی ندارد).

---

## ۱۲. خواندن نام‌های منطقی در Runtime با Reflection

اگر بخواهید در Runtime نام‌های منطقی اعضای Tuple را بخوانید:

```csharp
using System;
using System.Reflection;
using System.Runtime.CompilerServices;

public static class TupleHelper
{
    public static string[] GetTupleElementNames(MemberInfo member)
    {
        var attr = member.GetCustomAttribute<TupleElementNamesAttribute>();
        return attr?.TransformNames.ToArray();
    }

    public static string[] GetTupleElementNames(ParameterInfo param)
    {
        var attr = param.GetCustomAttribute<TupleElementNamesAttribute>();
        return attr?.TransformNames.ToArray();
    }
}

// استفاده:
var method = typeof(DataService).GetMethod("GetPerson");
var names = TupleHelper.GetTupleElementNames(method.ReturnParameter);
// names: ["Name", "Age"]

foreach (var name in names)
{
    Console.WriteLine(name);
}
```

---

## ۱۳. مقایسه Tupleهای نام‌دار و بی‌نام

| ویژگی | Tuple با نام | Tuple بی‌نام |
|--------|--------------|---------------|
| **دسترسی** | `person.Name` | `person.Item1` |
| **Attribute** | `TupleElementNames` دارد | ندارد یا `["", ""]` |
| **نوع Runtime** | `ValueTuple<string, int>` | `ValueTuple<string, int>` |
| **سازگاری** | ✅ قابل تبدیل به هم | ✅ قابل تبدیل به هم |
| **خوانایی کد** | بالا | پایین |

```csharp
// این دو از نظر نوع یکسان هستند:
(string Name, int Age) a = ("Ali", 30);
(string, int) b = ("Ali", 30);

a = b; // ✅ مجاز
b = a; // ✅ مجاز
```

---

## ۱۴. Deconstruction و نام‌های منطقی

نام‌های منطقی در Deconstruction هم استفاده می‌شوند:

```csharp
public (string Name, int Age) GetPerson() => ("Ali", 30);

// Deconstruction با نام‌های دلخواه:
var (myName, myAge) = GetPerson();
Console.WriteLine(myName); // "Ali"

// Deconstruction با var
var (name, age) = GetPerson();
```

---

## ۱۵. نکات مهم و Best Practices

### ✅ انجام دهید:

1. **همیشه نام‌های معنادار بدهید:**
   ```csharp
   // خوب ✅
   (string FirstName, string LastName) GetFullName()
   
   // بد ❌
   (string, string) GetFullName()
   ```

2. **در APIهای عمومی از Tuple استفاده کنید** (برای Return Typeهای ساده):
   ```csharp
   public (bool Success, string ErrorMessage) Validate(string input)
   ```

3. **برای موارد پیچیده‌تر، Record یا Class بسازید:**
   ```csharp
   // اگر بیش از ۴-۵ عضو دارد یا رفتار نیاز دارد:
   public record Person(string Name, int Age, string Email);
   ```

### ❌ انجام ندهید:

1. **نام‌های متناقض نگذارید:**
   ```csharp
   // بد ❌ - نام‌ها با نوع همخوانی ندارند
   (string Number, int Name) GetData()
   ```

2. **به TupleElementNamesAttribute در کد عادی依赖 نکنید:**
   ```csharp
   // بد ❌ - نباید مستقیماً با Reflection بخوانید مگر ضرورت باشد
   ```

---

## ۱۶. جمع‌ بندی

| مفهوم | توضیح |
|--------|--------|
| **Tuple در Runtime** | همیشه `ValueTuple<T1, T2, ...>` با فیلدهای `Item1`, `Item2` |
| **نام‌های منطقی** | فقط در Compile Time وجود دارند |
| **TupleElementNamesAttribute** | Attributeای که نام‌های منطقی را در Metadata ذخیره می‌کند |
| **نقش Attribute** | ارتباط بین نام منطقی و `ItemN` در Compile Time |
| **Cross-Assembly** | بدون این Attribute، نام‌های منطقی بین Library و Application کار نمی‌کردند |
| **در IL** | نام‌ها به‌صورت رشته در Custom Attribute ذخیره می‌شوند |
| **Reflection** | نام‌های منطقی را مستقیماً نمی‌بیند، باید Attribute را بخواند |

---

## منابع

1. **Microsoft Docs - Tuple types:**  
   https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples

2. **C# Language Specification - Tuples:**  
   https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/

3. **TupleElementNamesAttribute Class:**  
   https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerproperties.tupleelementnamesattribute

4. **Roslyn Source Code - TupleElementNamesAttribute:**  
   https://github.com/dotnet/roslyn

5. **ECMA-335 (CLI Specification) - Metadata:**  
   https://www.ecma-international.org/publications-and-standards/standards/ecma-335/

---

این مقاله نشان داد که چگونه C# با استفاده از `TupleElementNamesAttribute` و Metadata، تجربه‌ای یکپارچه از Tupleهای نام‌دار را فراهم می‌کند، در حالی که در سطح CLR همه چیز ساده و کارآمد باقی می‌ماند.