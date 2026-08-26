# 📘 قوانین یکسان بودن Anonymous Typeها در C#

> یک راهنمای جامع از مفاهیم پایه تا جزئیات رفتار کامپایلر Roslyn

---

## 📑 فهرست مطالب

- [مقدمه](#مقدمه)
- [Anonymous Type چیست؟](#anonymous-type-چیست)
- [قانون طلایی یکسان بودن Type](#قانون-طلایی-یکسان-بودن-type)
- [نقش نام Propertyها](#نقش-نام-propertyها)
- [نقش نوع Propertyها](#نقش-نوع-propertyها)
- [نقش ترتیب Propertyها](#نقش-ترتیب-propertyها)
- [نقش Assembly](#نقش-assembly)
- [مثال‌های عملی](#مثالهای-عملی)
  - [دو Type کاملاً یکسان](#۱-دو-type-کاملاً-یکسان)
  - [نام Property متفاوت](#۲-نام-property-متفاوت)
  - [نوع Property متفاوت](#۳-نوع-property-متفاوت)
  - [ترتیب Property متفاوت](#۴-ترتیب-property-متفاوت)
- [آیا ترتیب Propertyها واقعاً اهمیت دارد؟](#آیا-ترتیب-propertyها-واقعاً-اهمیت-دارد)
- [رفتار کامپایلر Roslyn](#رفتار-کامپایلر-roslyn)
- [چرا کامپایلر یک Type مشترک تولید می‌کند؟](#چرا-کامپایلر-یک-type-مشترک-تولید-میکند)
- [Equality در Anonymous Typeها](#equality-در-anonymous-typeها)
- [تفاوت Type Identity با Value Equality](#تفاوت-type-identity-با-value-equality)
- [مقایسه ویژه: Ali و Reza](#مقایسه-ویژه-ali-و-reza)
- [نکات مهم](#نکات-مهم)
- [اشتباهات رایج](#اشتباهات-رایج)
- [جمع‌بندی](#جمع-بندی)
- [منابع](#منابع)

---

## مقدمه

هنگام کار با LINQ و ساختارهای داده موقت، احتمالاً بارها از `new { ... }` استفاده کرده‌اید. اما آیا تا به حال فکر کرده‌اید که آیا دو شیء زیر از **یک نوع (Type)** محسوب می‌شوند یا خیر؟

```csharp
var person1 = new { Name = "Ali", Age = 20 };
var person2 = new { Name = "Reza", Age = 20 };
```

پاسخ به این سؤال ساده، شما را وارد دنیای جذاب **Type Identity**، **Value Equality** و **رفتار کامپایلر Roslyn** می‌کند. در این مقاله، از سطح مقدماتی تا جزئیات عمیق کامپایلر، این موضوع را بررسی می‌کنیم.

---

## Anonymous Type چیست؟

یک **Anonymous Type** نوعی است که:

- توسط کامپایلر به‌صورت خودکار تولید می‌شود.
- نام مشخصی ندارد (نامی که کامپایلر تولید می‌کند قابل استفاده مستقیم نیست).
- فقط شامل **Propertyهای Read-Only** است.
- به‌طور خودکار متدهای `Equals`، `GetHashCode` و `ToString` را override می‌کند.
- از کلاس `object` ارث‌بری می‌کند.

```csharp
var product = new { Id = 1, Name = "Laptop", Price = 1200.50 };
// نوع product یک Anonymous Type است
Console.WriteLine(product.GetType().Name);
// خروجی: <>f__AnonymousType0`3
```

> 🔗 [بازگشت به فهرست](#فهرست-مطالب)

---

## قانون طلایی یکسان بودن Type

طبق **C# Language Specification (بخش 11.7.11.2)**، دو Anonymous Type تنها زمانی **یکسان (Identical)** محسوب می‌شوند که **هر چهار شرط زیر** برقرار باشد:

```
┌─────────────────────────────────────────────────────┐
│  1. در یک Assembly تعریف شده باشند                  │
│  2. نام Propertyها یکسان باشد                       │
│  3. نوع Propertyها یکسان باشد                       │
│  4. ترتیب Propertyها یکسان باشد                     │
└─────────────────────────────────────────────────────┘
```

اگر حتی یکی از این شرایط نقض شود، کامپایلر **دو Type کاملاً مجزا** تولید می‌کند.

---

## نقش نام Propertyها

نام Propertyها باید **دقیقاً یکسان** باشد (case-sensitive).

```csharp
var a = new { Name = "Ali" };
var b = new { name = "Ali" };  // ❌ Type متفاوت!
var c = new { Name = "Reza" }; // ✅ همان Type
```

> ⚠️ توجه: `Name` و `name` دو Property متفاوت هستند چون C# به بزرگی و کوچکی حروف حساس است.

---

## نقش نوع Propertyها

نوع Propertyها نیز باید **دقیقاً یکسان** باشد.

```csharp
var a = new { Age = 20 };       // int
var b = new { Age = 20L };      // long → Type متفاوت!
var c = new { Age = 20.0 };     // double → Type متفاوت!
var d = new { Age = (short)20 };// short → Type متفاوت!
```

حتی تفاوت بین `int` و `long` باعث تولید Type متفاوت می‌شود.

---

## نقش ترتیب Propertyها

ترتیب Propertyها **بسیار مهم** است.

```csharp
var a = new { Name = "Ali", Age = 20 };
var b = new { Age = 20, Name = "Ali" }; // ❌ Type متفاوت!
```

با وجود اینکه هر دو دقیقاً همان Propertyها را دارند، اما چون **ترتیب** متفاوت است، کامپایلر دو Type مجزا تولید می‌کند.

---

## نقش Assembly

این نکته‌ای است که بسیاری از برنامه‌نویسان از آن غافل‌اند:

> 📌 **دو Anonymous Type با مشخصات کاملاً یکسان، اگر در Assemblyهای متفاوت تعریف شوند، دو Type مجزا خواهند بود.**

```csharp
// در Assembly A
var x = new { Name = "Ali", Age = 20 };

// در Assembly B (مثلاً یک پروژه دیگر)
var y = new { Name = "Reza", Age = 20 };

// x و y از Type یکسان نیستند!
```

این محدودیت به این دلیل است که نوع تولیدشده توسط کامپایلر، **internal** است و در خارج از Assembly قابل دسترسی نیست.

---

## مثال‌های عملی

### ۱. دو Type کاملاً یکسان

```csharp
var p1 = new { Name = "Ali", Age = 20 };
var p2 = new { Name = "Reza", Age = 20 };

Console.WriteLine(p1.GetType() == p2.GetType()); // ✅ True
Console.WriteLine(object.ReferenceEquals(p1.GetType(), p2.GetType())); // ✅ True
```

چون نام، نوع و ترتیب Propertyها یکسان است، کامپایلر **یک Type واحد** تولید می‌کند.

### ۲. نام Property متفاوت

```csharp
var p1 = new { Name = "Ali", Age = 20 };
var p2 = new { FullName = "Ali", Age = 20 };

Console.WriteLine(p1.GetType() == p2.GetType()); // ❌ False
```

### ۳. نوع Property متفاوت

```csharp
var p1 = new { Name = "Ali", Age = 20 };
var p2 = new { Name = "Ali", Age = 20L };

Console.WriteLine(p1.GetType() == p2.GetType()); // ❌ False
```

### ۴. ترتیب Property متفاوت

```csharp
var p1 = new { Name = "Ali", Age = 20 };
var p2 = new { Age = 20, Name = "Ali" };

Console.WriteLine(p1.GetType() == p2.GetType()); // ❌ False
```

> 🔗 [بازگشت به فهرست](#فهرست-مطالب)

---

## آیا ترتیب Propertyها واقعاً اهمیت دارد؟

**بله، صد در صد!** این یکی از نکات کلیدی است که باید به خاطر بسپارید.

حتی اگر دو Anonymous Type از نظر منطقی یکسان به نظر برسند، اگر ترتیب Propertyها متفاوت باشد، کامپایلر آن‌ها را به‌عنوان دو Type مجزا در نظر می‌گیرد.

### چرا این‌گونه است؟

کامپایلر برای تولید نام داخلی Anonymous Type، از **ترکیب نام، نوع و ترتیب Propertyها** استفاده می‌کند. بنابراین ترتیب بخشی از **امضای Type** است.

```csharp
// این دو کاملاً متفاوت هستند:
var a = new { X = 1, Y = 2 };
var b = new { Y = 2, X = 1 };

// حتی نمی‌توانید آن‌ها را به یکدیگر assign کنید:
// a = b; // ❌ Compilation Error
```

---

## رفتار کامپایلر Roslyn

وقتی کامپایلر Roslyn با یک Anonymous Type مواجه می‌شود:

1. **امضای Type** را محاسبه می‌کند (شامل نام، نوع و ترتیب Propertyها).
2. بررسی می‌کند که آیا در Assembly جاری، Typeای با این امضا قبلاً تولید شده است یا خیر.
3. اگر وجود داشت، **همان Type قبلی** را مجدداً استفاده می‌کند.
4. در غیر این صورت، یک **Type جدید** با نامی مانند `<>f__AnonymousType0`3` تولید می‌کند.

### نام‌گذاری داخلی

```csharp
var x = new { Name = "Ali", Age = 20 };
Console.WriteLine(x.GetType().FullName);
// خروجی: <>f__AnonymousType0`2[[System.String, ...],[System.Int32, ...]]
```

عدد `0` نشان‌دهنده شماره Type و `2` نشان‌دهنده تعداد Propertyهاست.

---

## چرا کامپایلر یک Type مشترک تولید می‌کند؟

این کار به چند دلیل انجام می‌شود:

1. **بهینه‌سازی حافظه**: به جای تولید چندین کلاس مشابه، یک کلاس مشترک استفاده می‌شود.
2. **سازگاری با LINQ**: در عملیات LINQ مانند `Select` و `Join`، امکان مقایسه و ترکیب نتایج فراهم می‌شود.
3. **Value Equality**: امکان مقایسه منطقی بین دو شیء با مقادیر یکسان فراهم می‌شود.

```csharp
var list = new[]
{
    new { Name = "Ali", Age = 20 },
    new { Name = "Reza", Age = 25 },
    new { Name = "Sara", Age = 20 }
};

// همه این‌ها از یک Type هستند و می‌توانند در یک آرایه قرار گیرند
var filtered = list.Where(p => p.Age == 20);
```

> 🔗 [بازگشت به فهرست](#فهرست مطالب)

---

## Equality در Anonymous Typeها

Anonymous Typeها به‌صورت پیش‌فرض از **Value Equality** پشتیبانی می‌کنند، نه Reference Equality.

```csharp
var p1 = new { Name = "Ali", Age = 20 };
var p2 = new { Name = "Ali", Age = 20 };
var p3 = new { Name = "Reza", Age = 20 };

Console.WriteLine(p1.Equals(p2)); // ✅ True (Value Equality)
Console.WriteLine(p1.Equals(p3)); // ❌ False
Console.WriteLine(object.ReferenceEquals(p1, p2)); // ❌ False (Reference Equality)
```

### پیاده‌سازی داخلی `Equals`

کامپایلر متد `Equals` را به این صورت تولید می‌کند:

```csharp
public override bool Equals(object value)
{
    var v = value as AnonymousType;
    return v != null
        && EqualityComparer<string>.Default.Equals(this.Name, v.Name)
        && EqualityComparer<int>.Default.Equals(this.Age, v.Age);
}
```

و `GetHashCode` نیز بر اساس مقادیر Propertyها محاسبه می‌شود:

```csharp
public override int GetHashCode()
{
    int hash = -1784932;
    hash = hash * -1521134295 + EqualityComparer<string>.Default.GetHashCode(Name);
    hash = hash * -1521134295 + EqualityComparer<int>.Default.GetHashCode(Age);
    return hash;
}
```

---

## تفاوت Type Identity با Value Equality

این یکی از مهم‌ترین مفاهیم است که باید به‌خوبی درک شود:

| مفهوم | تعریف | مثال |
|-------|--------|------|
| **Type Identity** | آیا دو شیء از **یک Type** هستند؟ | `p1.GetType() == p2.GetType()` |
| **Value Equality** | آیا دو شیء **مقادیر یکسانی** دارند؟ | `p1.Equals(p2)` |
| **Reference Equality** | آیا دو شیء به **یک مکان در حافظه** اشاره می‌کنند؟ | `object.ReferenceEquals(p1, p2)` |

### مثال عملی

```csharp
var p1 = new { Name = "Ali", Age = 20 };
var p2 = new { Name = "Ali", Age = 20 };
var p3 = new { Name = "Ali", Age = 20L }; // long به جای int

// Type Identity
Console.WriteLine(p1.GetType() == p2.GetType()); // ✅ True
Console.WriteLine(p1.GetType() == p3.GetType()); // ❌ False (نوع Property متفاوت)

// Value Equality
Console.WriteLine(p1.Equals(p2)); // ✅ True
Console.WriteLine(p1.Equals(p3)); // ❌ False (حتی اگر Type یکسان بود، مقدار Age متفاوت است)

// Reference Equality
Console.WriteLine(object.ReferenceEquals(p1, p2)); // ❌ False (دو شیء مجزا)
```

> 🔗 [بازگشت به فهرست](#فهرست مطالب)

---

## مقایسه ویژه: Ali و Reza

بیایید این دو را به‌طور کامل بررسی کنیم:

```csharp
var ali = new { Name = "Ali", Age = 20 };
var reza = new { Name = "Reza", Age = 20 };
```

### تحلیل

| معیار | نتیجه |
|-------|-------|
| نام Propertyها | ✅ یکسان (`Name`, `Age`) |
| نوع Propertyها | ✅ یکسان (`string`, `int`) |
| ترتیب Propertyها | ✅ یکسان (`Name` اول، `Age` دوم) |
| Assembly | ✅ یکسان (در یک فایل کد) |

### نتیجه

```csharp
Console.WriteLine(ali.GetType() == reza.GetType()); // ✅ True
Console.WriteLine(ali.Equals(reza)); // ❌ False (مقادیر متفاوت)
Console.WriteLine(object.ReferenceEquals(ali, reza)); // ❌ False
```

**نکته کلیدی**: `ali` و `reza` از **یک Type** هستند، اما **مقادیر متفاوتی** دارند. این دقیقاً مانند دو شیء از یک کلاس `Person` است که نام‌های متفاوتی دارند.

### استفاده در LINQ

```csharp
var people = new[]
{
    new { Name = "Ali", Age = 20 },
    new { Name = "Reza", Age = 25 },
    new { Name = "Sara", Age = 22 }
};

// همه از یک Type هستند، پس می‌توانیم LINQ استفاده کنیم
var adults = people.Where(p => p.Age >= 20);
var names = people.Select(p => p.Name);
```

---

## نکات مهم

✅ **نکات کلیدی که باید به خاطر بسپارید:**

1. **چهار شرط یکسان بودن Type**: نام، نوع، ترتیب Propertyها و یکسان بودن Assembly.
2. **ترتیب مهم است**: `new { X = 1, Y = 2 }` با `new { Y = 2, X = 1 }` متفاوت است.
3. **نوع دقیق مهم است**: `int` با `long` متفاوت است.
4. **Case-sensitive**: `Name` با `name` متفاوت است.
5. **Value Equality پیش‌فرض**: `Equals` بر اساس مقادیر کار می‌کند.
6. **Typeهای تولیدشده internal هستند**: در خارج از Assembly قابل دسترسی نیستند.
7. **نام Type قابل پیش‌بینی نیست**: از نوشتن کد وابسته به نام Type خودداری کنید.

---

## اشتباهات رایج

❌ **اشتباه ۱: فرض یکسان بودن Type با ترتیب متفاوت**

```csharp
var a = new { Name = "Ali", Age = 20 };
var b = new { Age = 20, Name = "Ali" };
// ❌ این دو از یک Type نیستند!
// a = b; // Compilation Error
```

❌ **اشتباه ۲: استفاده از Anonymous Type به‌عنوان فیلد کلاس**

```csharp
public class Person
{
    // ❌ Compilation Error - نمی‌توان از Anonymous Type به‌عنوان نوع فیلد استفاده کرد
    public var Info = new { Name = "Ali", Age = 20 };
}
```

❌ **اشتباه ۳: بازگرداندن Anonymous Type از متد**

```csharp
// ❌ Compilation Error
public ??? GetPerson()
{
    return new { Name = "Ali", Age = 20 };
}

// ✅ راه‌حل: استفاده از var یا dynamic یا Tuple
public object GetPerson() => new { Name = "Ali", Age = 20 };
```

❌ **اشتباه ۴: انتظار Reference Equality**

```csharp
var p1 = new { Name = "Ali", Age = 20 };
var p2 = new { Name = "Ali", Age = 20 };
// ❌ ReferenceEquals همیشه false است (مگر اینکه همان شیء باشد)
Console.WriteLine(object.ReferenceEquals(p1, p2)); // False
```

❌ **اشتباه ۵: استفاده از Anonymous Type در Assemblyهای متفاوت**

```csharp
// در Assembly A
public static object GetPerson() => new { Name = "Ali", Age = 20 };

// در Assembly B
var person = AssemblyA.GetPerson();
// ❌ نمی‌توانید به Propertyها دسترسی مستقیم داشته باشید
// person.Name; // Compilation Error
```

> 🔗 [بازگشت به فهرست](#فهرست مطالب)

---

## جمع‌بندی

در این مقاله یاد گرفتیم که:

1. **Anonymous Typeها** توسط کامپایلر تولید می‌شوند و نام مشخصی ندارند.
2. **چهار شرط** برای یکسان بودن دو Anonymous Type وجود دارد: نام، نوع، ترتیب Propertyها و یکسان بودن Assembly.
3. **ترتیب Propertyها** بسیار مهم است و دو Type با ترتیب متفاوت، مجزا محسوب می‌شوند.
4. **کامپایلر Roslyn** برای بهینه‌سازی، Typeهای یکسان را به‌صورت مشترک استفاده می‌کند.
5. **Equality** در Anonymous Typeها بر اساس **Value Equality** است، نه Reference Equality.
6. **Type Identity** با **Value Equality** متفاوت است: دو شیء می‌توانند از یک Type باشند اما مقادیر متفاوتی داشته باشند.
7. **مثال Ali و Reza**: این دو از یک Type هستند اما مقادیر متفاوتی دارند.

### جدول خلاصه

| سناریو | Type یکسان؟ | Equals؟ |
|--------|--------------|---------|
| `new { Name = "Ali", Age = 20 }` و `new { Name = "Reza", Age = 20 }` | ✅ بله | ❌ خیر |
| `new { Name = "Ali", Age = 20 }` و `new { Name = "Ali", Age = 20 }` | ✅ بله | ✅ بله |
| `new { Name = "Ali", Age = 20 }` و `new { Age = 20, Name = "Ali" }` | ❌ خیر | ❌ خیر |
| `new { Name = "Ali", Age = 20 }` و `new { Name = "Ali", Age = 20L }` | ❌ خیر | ❌ خیر |
| `new { Name = "Ali", Age = 20 }` و `new { name = "Ali", Age = 20 }` | ❌ خیر | ❌ خیر |

---

## منابع

📚 **منابع رسمی و معتبر:**

1. **Microsoft Learn - Anonymous Types**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types)

2. **C# Language Specification - Anonymous Object Creation Expressions**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#117112-anonymous-object-creation-expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#117112-anonymous-object-creation-expressions)

3. **Microsoft Learn - Anonymous Types (C# Programming Guide)**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types)

4. **ECMA-334 C# Language Specification**  
   [https://www.ecma-international.org/publications-and-standards/standards/ecma-334/](https://www.ecma-international.org/publications-and-standards/standards/ecma-334/)

5. **Roslyn Source Code - Anonymous Type Generation**  
   [https://github.com/dotnet/roslyn](https://github.com/dotnet/roslyn)

---

> 💡 **نکته پایانی**: درک عمیق از رفتار کامپایلر در Anonymous Typeها، به شما کمک می‌کند کد بهینه‌تر بنویسید و از اشتباهات رایج در LINQ و کار با داده‌های موقت جلوگیری کنید.

---

**نویسنده**: [نام شما]  
**تاریخ**: August 27, 2026  
**نسخه C#**: 12.0+  
**سطح**: متوسط تا پیشرفته

---

*اگر این مقاله برای شما مفید بود، لطفاً آن را Star کنید و به اشتراک بگذارید!* ⭐