# محدودیت‌های Anonymous Type در C#؛ بررسی جامع، دلایل فنی و راهکارهای جایگزین

به نام خدا. به بخش **«محدودیت‌های Anonymous Type»** از این Repository آموزشی خوش آمدید. 
در این مقاله، ما از سطح مقدماتی شروع کرده و به عمق معماری کامپایلر C# نفوذ می‌کنیم تا بدانیم چرا `Anonymous Type` (نوع ناشناس) با وجود کاربرد فراوان در LINQ، محدودیت‌های ساختاری شدیدی دارد و چه زمانی باید قید آن را بزنیم.

---

## 📑 فهرست مطالب
- [مقدمه: Anonymous Type چیست؟](#مقدمه)
- [۱. چرا نمی‌توان Anonymous Type را نام‌گذاری کرد؟](#۱-هویت-و-نام-گذاری)
- [۲. محدودیت در امضای متدها (Return Type و Parameters)](#۲-امضای-متدها)
- [۳. محدودیت در اعضای کلاس (Field, Property, Constructor و...)](#۳-اعضای-کلاس)
- [۴. محدودیت‌های Scope و استفاده بین متدها](#۴-محدودیت-scope)
- [۵. محدودیت در API و معماری پروژه](#۵-محدودیت-در-api)
- [۶. راهکارهای جایگزین (Class, Record, Tuple, DTO)](#۶-راهکارهای-جایگزین)
- [۷. چه زمانی باید Anonymous Type را کنار گذاشت؟](#۷-ماتریس-تصمیم-گیری)
- [نکات مهم و اشتباهات رایج](#نکات-و-اشتباهات)
- [جمع‌بندی](#جمع-بندی)
- [منابع معتبر](#منابع)

---

<a name="مقدمه"></a>
## مقدمه: Anonymous Type چیست؟
نوع ناشناس در C# (از نسخه 3.0 به بعد) یک راه میانبر (Syntactic Sugar) برای ایجاد سریع یک شیء بدون نیاز به تعریف صریح یک `Class` است.
```csharp
var user = new { Name = "Ali", Age = 30 };
```
اما پشت پرده، کامپایلر C# یک کلاس واقعی و کاملاًtyped ایجاد می‌کند. پس چرا ما به عنوان برنامه‌نویس نمی‌توانیم از تمام قابلیت‌های یک کلاس معمولی برای آن استفاده کنیم؟ در ادامه به تک‌تک این محدودیت‌ها می‌پردازیم.

---

<a name="۱-هویت-و-نام-گذاری"></a>
## ۱. چرا نمی‌توان Anonymous Type را نام‌گذاری کرد؟

**دلیل فنی:** طبق **C# Language Specification**، نام کلاس‌های تولید شده توسط کامپایلر برای Anonymous Types، نام‌هایی غیرقابل تلفظ (Unutterable) و تولید شده به‌صورت خودکار مانند `<>f__AnonymousType0` هستند. 
طراحان C# عمداً این قابلیت را مسدود کردند تا از **Coupling (وابستگی) شدید** جلوگیری کنند. هدف از طراحی این نوع، صرفاً استفاده به‌عنوان یک ظرف موقت داده (Data Projection) در داخل همان بلوک کد (معمولاً در LINQ) بوده است، نه تبدیل شدن به یک موجودیت پایدار در معماری نرم‌افزار.

---

<a name="۲-امضای-متدها"></a>
## ۲. محدودیت در امضای متدها (Return Type و Parameters)

از آنجا که شما نمی‌توانید نام این نوع را بنویسید، عملاً نمی‌توانید آن را در امضای متدها استفاده کنید.

### ❌ مثال اشتباه (Return Type)
```csharp
// خطای کامپایلر: CS1001 Identifier expected
public var GetUserData() 
{
    return new { Id = 1, Name = "Sara" }; 
}
```
**راه‌حل اشتباه و خطرناک:** استفاده از `object` یا `dynamic`.
```csharp
public dynamic GetUserData() => new { Id = 1 }; // از دست رفتن Type Safety و افت شدید Performance
```

### ❌ مثال اشتباه (Method Parameter)
```csharp
// خطای کامپایلر: CS0825 Contextual keyword 'var' can only appear within a local variable declaration
public void PrintUser(var user) 
{
    Console.WriteLine(user.Name);
}
```

---

<a name="۳-اعضای-کلاس"></a>
## ۳. محدودیت در اعضای کلاس (Field, Property, Constructor و...)

یک Anonymous Type صرفاً یک **Data Container** است. کامپایلر فقط Propertyهای Read-Only، متدهای `Equals`، `GetHashCode` و `ToString` را برای آن تولید می‌کند.

### ❌ Field یا Property
شما نمی‌توانید یک Field یا Property از این نوع در کلاس خود داشته باشید، زیرا نوع آن در زمان کامپایل برای اعضای کلاس باید مشخص باشد.
```csharp
public class UserService
{
    // خطای کامپایلر: CS0825
    private var _cachedUser; 
    
    // خطای کامپایلر: CS0825
    public var CurrentUser { get; set; } 
}
```

### ❌ Constructor، Method، Event و Custom Operator
شما نمی‌توانید برای Anonymous Type سازنده، متد، رویداد یا اپراتور سفارشی تعریف کنید.
**دلیل معماری:** این انواع بر اساس اصل **Immutability (تغییرناپذیری)** و **Single Responsibility** طراحی شده‌اند. آن‌ها فقط برای نگهداری داده طراحی شده‌اند و اضافه کردن Behavior (رفتار) به آن‌ها نقض طراحی آن‌هاست. شما حتی نمی‌توانید از آن‌ها ارث‌بری (Inheritance) کنید.

---

<a name="۴-محدودیت-scope"></a>
## ۴. محدودیت‌های Scope و استفاده بین متدها

مهم‌ترین محدودیت عملیاتی این است که Anonymous Typeها **فقط در Scope همان متدی** که در آن `new` شده‌اند، زنده و قابل استفاده‌اند.

### ❌ مثال اشتباه (استفاده بین متدها)
```csharp
public void ProcessData()
{
    var tempData = new { Status = "Success", Code = 200 };
    
    // خطای کامپایلر: نمی‌توانید tempData را به متد دیگری پاس دهید 
    // مگر اینکه پارامتر متد مقصد object یا dynamic باشد.
    LogData(tempData); 
}

public void LogData(object data)
{
    // برای دسترسی به پراپرتی‌ها باید از Reflection استفاده کنید!
    var status = data.GetType().GetProperty("Status").GetValue(data);
}
```
> 💡 **نکته مهم:** استفاده از `dynamic` برای دور زدن این محدودیت، اگرچه کد را کامپایل می‌کند، اما **IntelliSense** را از بین می‌برد و خطاهای Runtime (به جای Compile-time) ایجاد می‌کند.

---

<a name="۵-محدودیت-در-api"></a>
## ۵. محدودیت در API و معماری پروژه

### 🚫 استفاده در Public API
شما نمی‌توانید یک Anonymous Type را به عنوان Contract در یک Public API (مثل یک Library) استفاده کنید، زیرا مصرف‌کننده Library شما راهی برای اشاره به نوع بازگشتی شما ندارد.

### 🚫 استفاده در Web API و Serialization
اگرچه برخی Serializerهای مدرن (مثل `System.Text.Json`) ممکن است بتوانند Anonymous Types را Serialize کنند، اما این کار یک **Anti-Pattern** در معماری است.
**دلیل:** در معماری لایه‌ای و APIها، شما نیاز به **Versioning**، **Documentation (مثل Swagger/OpenAPI)** و **Validation** دارید. Swagger نمی‌تواند Schema یک نوع ناشناس را تولید کند.

### 🚫 استفاده در Entity Framework (به عنوان Entity)
شما نمی‌توانید یک Anonymous Type را به عنوان Entity برای مپینگ به دیتابیس در EF Core استفاده کنید، زیرا EF نیاز به یک `Type` نام‌گذاری شده و قابل ردیابی (Trackable) دارد. (البته استفاده از آن در `Select` برای پروjection مجاز است).

---

<a name="۶-راهکارهای-جایگزین"></a>
## ۶. راهکارهای جایگزین

وقتی به محدودیت‌های بالا برخوردید، باید از جایگزین‌های زیر استفاده کنید:

### ✅ ۱. استفاده از Record (بهترین جایگزین مدرن - C# 9.0+)
اگر به یک ساختار داده‌ای تغییرناپذیر (Immutable) با سینتکس کوتاه نیاز دارید که بتوانید آن را نام‌گذاری کرده و بین متدها پاس دهید، `Record` پادشاه این حوزه است.
```csharp
// تعریف
public record UserDto(string Name, int Age);

// استفاده
var user = new UserDto("Ali", 30);
public UserDto GetUser() => new UserDto("Ali", 30); // کاملاً مجاز و Type-Safe
```

### ✅ ۲. استفاده از Tuple (برای بازگشت‌های سریع و محلی)
اگر نمی‌خواهید یک کلاس جدید تعریف کنید و فقط می‌خواهید چند مقدار را از یک متد برگردانید.
```csharp
public (string Name, int Age) GetUserTuple()
{
    return ("Ali", 30);
}
```

### ✅ ۳. استفاده از Class (برای نیازهای OOP)
اگر داده‌های شما نیاز به Validation، متد، یا تغییرپذیری (Mutability) دارند.
```csharp
public class User 
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

### ✅ ۴. استفاده از DTO (برای لایه‌های معماری و API)
برای انتقال داده بین لایه‌ها (مثلاً از Domain به Presentation)، همیشه از کلاس‌های صریح به نام DTO استفاده کنید.

---

<a name="۷-ماتریس-تصمیم-گیری"></a>
## ۷. چه زمانی باید Anonymous Type را کنار گذاشت؟

از جدول زیر به عنوان چک‌لیست استفاده کنید:

| نیاز شما | آیا Anonymous Type مناسب است؟ | جایگزین پیشنهادی |
| :--- | :---: | :--- |
| پروjection در LINQ (`Select`) | ✅ بله | همان Anonymous Type |
| متغیر موقت داخل یک متد | ✅ بله | همان Anonymous Type |
| بازگشت از متد (Return Type) | ❌ خیر | `Record` یا `Tuple` |
| پارامتر ورودی متد | ❌ خیر | `Record` یا `Class` |
| ذخیره در Field/Property کلاس | ❌ خیر | `Record` یا `Class` |
| انتقال بین لایه‌های معماری | ❌ خیر | `DTO (Class)` |
| استفاده در Swagger / API | ❌ خیر | `DTO (Class)` |
| نیاز به متد یا رفتار (Behavior) | ❌ خیر | `Class` |

---

<a name="نکات-و-اشتباهات"></a>
## 💡 نکات مهم و اشتباهات رایج

1. **اشتباه رایج: تلاش برای تغییر مقادیر (Mutability)**
   پراپرتی‌های Anonymous Type فقط `get` دارند.
   ```csharp
   var obj = new { Name = "Ali" };
   obj.Name = "Reza"; // ❌ خطای کامپایلر: CS0200 Property or indexer cannot be assigned to
   ```
2. **اشتباه رایج: استفاده از `dynamic` برای عبور از محدودیت Scope**
   هرگز برای پاس دادن Anonymous Type به متد دیگر از `dynamic` استفاده نکنید. این کار DLR (Dynamic Language Runtime) را درگیر کرده و Performance برنامه شما را به شدت کاهش می‌دهد.
3. **نکته طلایی: Equality در Anonymous Types**
   دو Anonymous Type اگر نام پراپرتی‌ها، نوع آن‌ها و ترتیب قرارگیری‌شان یکسان باشد، از نظر مقدار (Value Equality) برابرند.
   ```csharp
   var a = new { X = 1, Y = 2 };
   var b = new { X = 1, Y = 2 };
   Console.WriteLine(a.Equals(b)); // ✅ خروجی: True
   ```

---

<a name="جمع-بندی"></a>
## 🎯 جمع‌بندی

`Anonymous Type` در C# یک شاهکار مهندسی برای ساده‌سازی کوئری‌های `LINQ` و ایجاد پروjectionهای محلی است. اما این ابزار برای **«استفاده محلی و موقت»** طراحی شده است، نه برای **«معماری و انتقال داده»**. 
به محض اینکه داده شما نیاز به عبور از مرزهای یک متد، لایه یا API پیدا کرد، باید فوراً آن را به یک `Record` (برای داده‌های Immutable) یا یک `Class/DTO` (برای داده‌های Mutable و پیچیده) ارتقا دهید. با معرفی `Record` در C# 9، عملاً نیاز به استفاده از Anonymous Type در سناریوهای پیچیده به حداقل رسیده است.

---

<a name="منابع"></a>
## 📚 منابع معتبر

1. [Microsoft Learn - Anonymous Types (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types)
2. [C# Language Specification - Anonymous Object Creation Expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#11715-anonymous-object-creation-expressions)
3. [Microsoft Learn - Records (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)

---
*اگر این مقاله برای شما مفید بود، لطفاً به این Repository ستاره (⭐) بدهید و برای توسعه‌دهندگان دیگر ارسال کنید. نظرات و PRهای شما برای بهبود این مستندات باعث افتخار است.*