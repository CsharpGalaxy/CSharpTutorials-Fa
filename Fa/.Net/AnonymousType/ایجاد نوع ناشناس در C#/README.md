# ایجاد نوع ناشناس در سی‌شارپ (Creating Anonymous Types)

به Repository آموزشی سی‌شارپ خوش آمدید! در این مقاله، به بررسی یکی از ویژگی‌های جذاب و پرکاربرد سی‌شارپ یعنی **انواع ناشناس (Anonymous Types)** می‌پردازیم. این مقاله از مفاهیم پایه شروع شده و تا نکات پیشرفته کامپایلر را پوشش می‌دهد.

---

## 📑 فهرست مطالب
- [مقدمه: نوع ناشناس چیست؟](#مقدمه-نوع-ناشناس-چیست)
- [Object Initializer (مقداردهی اولیه شی) چیست؟](#object-initializer-مقداردهی-اولیه-شی-چیست)
- [نحوه ایجاد Anonymous Type و Syntax کامل](#نحوه-ایجاد-anonymous-type-و-syntax-کامل)
- [تعریف Propertyها (صریح و استنباط‌شده)](#تعریف-propertyها-صریح-و-استنباط-شده)
- [استفاده از متغیرها و Member Access Expression](#استفاده-از-متغیرها-و-member-access-expression)
- [ایجاد Anonymous Type تو در تو (Nested)](#ایجاد-anonymous-type-تو-در-تو-nested)
- [نحوه دسترسی به Propertyها](#نحوه-دسترسی-به-propertyها)
- [نکات مهم و رفتار کامپایلر (سطح پیشرفته)](#نکات-مهم-و-رفتار-کامپایلر-سطح-پیشرفته)
- [اشتباهات رایج در Syntax](#اشتباهات-رایج-در-syntax)
- [جمع‌بندی](#جمع‌بندی)
- [منابع معتبر](#منابع-معتبر)

---

## مقدمه: نوع ناشناس چیست؟
نوع ناشناس (Anonymous Type) کلاسی است که **نام ندارد** و در همان لحظه‌ای که به آن نیاز دارید (معمولاً برای ذخیره موقت چند داده مرتبط با هم) ایجاد می‌شود. شما نیازی به تعریف کلاس در سطح Namespace ندارید؛ کامپایلر سی‌شارپ به‌صورت خودکار نام کلاس را تولید کرده و آن را کامپایل می‌کند.

> 💡 **کاربرد اصلی:** بیشترین کاربرد Anonymous Types در کوئری‌های **LINQ** برای انتخاب (Select) و گروه‌بندی (Group By) داده‌هاست.

---

## Object Initializer (مقداردهی اولیه شی) چیست؟
قبل از درک Anonymous Type، باید با **Object Initializer** آشنا شوید. این سینتکس به شما اجازه می‌دهد هنگام ساخت یک شیء، مقادیر اولیه را برای Propertyهای آن تعیین کنید، بدون اینکه نیاز به فراخوانی Constructor خاصی باشد.

```csharp
// یک کلاس معمولی
public class Person 
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// استفاده از Object Initializer
var p = new Person { Name = "Ali", Age = 30 };
```
در Anonymous Types، ما دقیقاً از همین سینتکس `{}` استفاده می‌کنیم، اما به جای مشخص کردن نام کلاس، فقط `new` را می‌نویسیم.

---

## نحوه ایجاد Anonymous Type و Syntax کامل

برای ایجاد یک نوع ناشناس، باید از کلمه کلیدی `new` به همراه یک بلوک `{}` استفاده کنید.

### Syntax پایه:
```csharp
var variableName = new { Property1 = Value1, Property2 = Value2 };
```

### مثال ساده:
```csharp
var student = new { FirstName = "Sara", StudentId = 101, Grade = 18.5 };

Console.WriteLine(student.FirstName); // خروجی: Sara
```

**توضیح کد:**
1. **`var`**: استفاده از `var` برای ذخیره Anonymous Type **اجباری** است، زیرا شما نام کلاس را نمی‌دانید تا آن را به‌صورت صریح بنویسید.
2. **`new`**: دستور ساخت شیء جدید را به کامپایلر می‌دهد.
3. **`{ ... }`**: همان Object Initializer است که Propertyها و مقادیر آن‌ها را تعریف می‌کند.

---

## تعریف Propertyها (صریح و استنباط‌شده)

شما می‌توانید نام Propertyها را به دو روش مشخص کنید:

### ۱. نام‌گذاری صریح (Explicit Naming)
همان‌طور که در مثال قبل دیدید، نام Property را خودتان تعیین می‌کنید:
```csharp
var product = new { ProductName = "Laptop", Price = 1200 };
```

### ۲. نام‌گذاری استنباط‌شده (Inferred / Projection Initializers)
از نسخه C# 8.0 به بعد، اگر از یک متغیر یا فیلد برای مقداردهی استفاده کنید، کامپایلر **نام همان متغیر** را به‌عنوان نام Property در نظر می‌گیرد.

```csharp
string city = "Tehran";
int population = 9000000;

// نام Propertyها دقیقاً همان city و population خواهد بود
var location = new { city, population }; 

Console.WriteLine(location.city); // خروجی: Tehran
```

---

## استفاده از متغیرها و Member Access Expression

شما می‌توانید مقادیر را از متغیرهای موجود یا Propertyهای اشیاء دیگر استخراج کنید.

```csharp
public class User 
{
    public string Username { get; set; }
    public string Email { get; set; }
}

User activeUser = new User { Username = "Admin", Email = "admin@test.com" };
int loginCount = 5;

// ترکیب Member Access و متغیر
var userInfo = new 
{ 
    activeUser.Username, // استنباط شده از User.Username
    activeUser.Email,    // استنباط شده از User.Email
    Attempts = loginCount // نام‌گذاری صریح
};

Console.WriteLine(userInfo.Username); // خروجی: Admin
```

---

## ایجاد Anonymous Type تو در تو (Nested)

شما می‌توانید یک Anonymous Type را به‌عنوان مقدارِ یک Property در یک Anonymous Type دیگر قرار دهید.

```csharp
var order = new 
{
    OrderId = 1001,
    OrderDate = DateTime.Now,
    // یک Anonymous Type تو در تو
    Customer = new 
    { 
        CustomerId = 55, 
        Name = "Reza" 
    }
};

// دسترسی به داده‌های تو در تو
Console.WriteLine($"Order #{order.OrderId} for {order.Customer.Name}");
```

---

## نحوه دسترسی به Propertyها

دسترسی به داده‌ها دقیقاً مانند کلاس‌های معمولی و با استفاده از **Dot Notation (نقطه)** انجام می‌شود. کامپایلر به لطف `var` و IntelliSense، نوع را می‌شناسد و پیشنهاد تکمیل خودکار (Auto-Complete) را به شما می‌دهد.

```csharp
var book = new { Title = "C# in Depth", Author = "Jon Skeet", Pages = 700 };

string t = book.Title;
int p = book.Pages;
```

---

## نکات مهم و رفتار کامپایلر (سطح پیشرفته)

درک این بخش برای برنامه‌نویسان حرفه‌ای سی‌شارپ الزامی است:

### ۱. تغییرناپذیری (Immutability)
تمام Propertyهای یک Anonymous Type به‌صورت **Read-Only** تولید می‌شوند. شما نمی‌توانید پس از ایجاد، مقدار آن‌ها را تغییر دهید.
```csharp
var point = new { X = 10, Y = 20 };
// point.X = 15; // ❌ خطای کامپایل: Property or indexer cannot be assigned to -- it is read only
```

### ۲. ارث‌بری از Object
انواع ناشناس مستقیماً از `System.Object` ارث‌بری می‌کنند. کامپایلر به‌صورت خودکار متدهای `Equals()`، `GetHashCode()` و `ToString()` را برای آن‌ها Override می‌کند تا مقایسه و نمایش آن‌ها ممکن باشد.
```csharp
var p1 = new { Name = "Ali", Age = 30 };
var p2 = new { Name = "Ali", Age = 30 };

Console.WriteLine(p1.Equals(p2)); // خروجی: True (مقایسه بر اساس مقدار)
Console.WriteLine(p1.ToString()); // خروجی: { Name = Ali, Age = 30 }
```

### ۳. قانون برابری نوع (Type Equality)
اگر دو Anonymous Type در **یک Assembly واحد** داشته باشند و Propertyهای آن‌ها از نظر **نام، نوع داده و ترتیب قرارگیری** دقیقاً یکسان باشد، کامپایلر آن‌ها را **یک نوع واحد** در نظر می‌گیرد.
```csharp
var a = new { X = 1, Y = 2 };
var b = new { X = 3, Y = 4 };

// این کد بدون خطا کامپایل می‌شود چون نوع a و b یکی است!
a = b; 
```
*نکته: اگر ترتیب Propertyها عوض شود (مثلاً `{ Y = 1, X = 2 }`)، کامپایلر آن را یک نوع کاملاً متفاوت می‌شناسد.*

---

## اشتباهات رایج در Syntax

### ❌ اشتباه ۱: عدم استفاده از `var`
```csharp
// ❌ خطا: شما نمی‌توانید نام نوع ناشناس را بنویسید چون وجود خارجی در کد شما ندارد
AnonymousType obj = new { Name = "Test" }; 

// ✅ راه‌حل: فقط از var استفاده کنید
var obj = new { Name = "Test" };
```

### ❌ اشتباه ۲: تلاش برای تغییر مقدار (Mutability)
```csharp
var data = new { Count = 10 };
// data.Count = 20; // ❌ خطا: Propertyها Read-Only هستند
```

### ❌ اشتباه ۳: بازگشت دادن (Return) از متد
شما نمی‌توانید یک Anonymous Type را به‌عنوان خروجی یک متد تعریف کنید، زیرا نامی برای Return Type ندارید.
```csharp
// ❌ خطا
public var GetData() {
    return new { Id = 1 }; 
}

// ✅ راه‌حل ۱: استفاده از object (اما IntelliSense و Type-Safety را از دست می‌دهید)
public object GetData() {
    return new { Id = 1 }; 
}

// ✅ راه‌حل ۲ (پیشنهادی): استفاده از Recordها یا Tupleها در سی‌شارپ مدرن
public (int Id, string Name) GetDataModern() {
    return (1, "Test");
}
```

### ❌ اشتباه ۴: فراموش کردن کاما (,) بین Propertyها
```csharp
// ❌ خطای Syntax
var err = new { A = 1 B = 2 }; 

// ✅ صحیح
var correct = new { A = 1, B = 2 };
```

---

## جمع‌بندی

* **Anonymous Type** راهی سریع برای ایجاد اشیاء موقت بدون نیاز به تعریف کلاس است.
* حتماً باید با کلمه کلیدی **`var`** مقداردهی شوند.
* Propertyهای آن‌ها **تغییرناپذیر (Read-Only)** هستند.
* برای نام‌گذاری Propertyها می‌توانید از نام متغیرها به‌صورت **استنباط‌شده (Inferred)** استفاده کنید.
* بهترین کاربرد آن‌ها در **LINQ** و انتقال موقت داده‌ها در سطح یک متد (Local Scope) است.
* در سی‌شارپ مدرن (نسخه‌های جدیدتر)، برای بسیاری از کاربردها استفاده از **Records** یا **Tuples** به دلیل قابلیت بازگشت از متدها، ترجیح داده می‌شود.

---

## منابع معتبر

برای مطالعه بیشتر و عمیق‌تر، منابع رسمی زیر پیشنهاد می‌شوند:

1. **Microsoft Learn - Anonymous Types (C# Programming Guide)**
   [لینک مستقیم به مستندات مایکروسافت](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types)
   
2. **C# Language Specification - Anonymous Object Creation Expressions**
   [لینک مستقیم به مشخصات زبان سی‌شارپ (ECMA/MS Spec)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128115-anonymous-object-creation-expressions)

---
*اگر این مقاله برای شما مفید بود، فراموش نکنید که به این Repository ستاره (⭐) بدهید!*