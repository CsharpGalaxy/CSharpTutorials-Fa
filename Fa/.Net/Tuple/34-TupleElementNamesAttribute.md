# آموزش جامع استفاده از Tuple به عنوان Return Type در C#

به مستند آموزشی شماره ۴ از سری آموزش‌های C# خوش آمدید. در این مقاله، یکی از کاربردی‌ترین ویژگی‌های اضافه شده در C# 7.0، یعنی استفاده از **Tupleها به عنوان نوع بازگشتی (Return Type)** متدها را بررسی می‌کنیم.

---

## 📑 فهرست مطالب
- [مقدمه](#مقدمه)
- [۱. سینتکس پایه: Tuple نام‌گذاری‌شده و بدون نام](#۱-سینتکس-پایه-tuple-نامگذاریشده-و-بدون-نام)
- [۲. Return کردن و دریافت Tuple در Caller](#۲-return-کردن-و-دریافت-tuple-در-caller)
- [۳. دسترسی به اعضا و استفاده از var](#۳-دسترسی-به-اعضا-و-استفاده-از-var)
- [۴. Deconstruction مستقیم (تخریب ساختار)](#۴-deconstruction-مستقیم-تخریب-ساختار)
- [۵. Tuple در متدهای Async](#۵-tuple-در-متدهای-async)
- [۶. Tuple در متدهای Generic](#۶-tuple-در-متدهای-generic)
- [۷. طراحی API: Tuple در متدهای Private در برابر Public](#۷-طراحی-api-tuple-در-متدهای-private-در-برابر-public)
- [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
- [جمع‌بندی](#جمع‌بندی)
- [منابع معتبر](#منابع-معتبر)

---

## مقدمه
در بسیاری از مواقع، نیاز داریم یک متد بیش از یک مقدار را برگرداند. پیش از C# 7.0، مجبور بودیم از `out parameters`، کلاس‌های موقت (DTO) یا `System.Tuple` (که سنگین و مبتنی بر Reference بود) استفاده کنیم. 
با معرفی **ValueTuple** (که با سینتکس `(T1, T2)` شناخته می‌شود)، ما یک ساختار سبک (Struct)، مبتنی بر Value و بسیار خوانا برای بازگرداندن چندین مقدار داریم.

---

## ۱. سینتکس پایه: Tuple نام‌گذاری‌شده و بدون نام

شما می‌توانید Tupleها را به دو صورت تعریف کنید:

### Tuple بدون نام (Unnamed)
در این حالت، اعضا فقط با شماره (Item1, Item2, ...) شناخته می‌شوند.
```csharp
// تعریف نوع بازگشتی بدون نام
public (int, string) GetBasicInfo()
{
    return (101, "Ali");
}
```

### Tuple نام‌گذاری‌شده (Named) - ⭐️ پیشنهاد می‌شود
در این حالت، برای هر عضو یک نام معنادار تعیین می‌کنیم که خوانایی کد را به شدت افزایش می‌دهد.
```csharp
// تعریف نوع بازگشتی با نام‌های معنادار
public (int UserId, string UserName) GetUserInfo()
{
    return (101, "Ali");
}
```

---

## ۲. Return کردن و دریافت Tuple در Caller

### نحوه Return کردن
شما می‌توانید مقادیر را به ترتیب تعریف شده در پرانتز برگردانید:
```csharp
public (bool IsSuccess, string Message) ProcessData(int data)
{
    if (data > 0)
        return (true, "Data processed successfully.");
    
    return (false, "Invalid data.");
}
```

### نحوه دریافت در Caller
```csharp
var result = ProcessData(10);
```

---

## ۳. دسترسی به اعضا و استفاده از var

اگر از `var` برای دریافت نتیجه استفاده کنید، کامپایلر به صورت خودکار نوع Tuple را استنتاج (Infer) می‌کند.

```csharp
var result = GetUserInfo();

// دسترسی به اعضا (تفاوت Tuple نام‌دار و بدون نام)
Console.WriteLine(result.UserId);   // خروجی: 101 (خوانا و عالی)
Console.WriteLine(result.UserName); // خروجی: Ali

// اگر متد Unnamed بود، مجبور بودید بنویسید:
// result.Item1, result.Item2 (که اصلا خوانا نیست!)
```

---

## ۴. Deconstruction مستقیم (تخریب ساختار)

یکی از زیباترین قابلیت‌های Tuple، **Deconstruction** است. شما می‌توانید اعضای Tuple را مستقیماً به متغیرهای جداگانه تخصیص دهید:

```csharp
// Deconstruction کامل
var (id, name) = GetUserInfo();
Console.WriteLine($"ID: {id}, Name: {name}");

// استفاده از Discard (_) برای نادیده گرفتن یک عضو
var (userId, _) = GetUserInfo(); 
Console.WriteLine($"Just ID: {userId}");
```
*نکته: در Deconstruction، نام متغیرهای سمت چپ (`id`, `name`) می‌تواند با نام اعضای Tuple (`UserId`, `UserName`) متفاوت باشد.*

---

## ۵. Tuple در متدهای Async

استفاده از Tuple در متدهای ناهمگام (Async) کاملاً پشتیبانی می‌شود و نیازی به ساخت کلاس واسط ندارید.

```csharp
public async Task<(bool IsFound, User User)> FindUserAsync(int id)
{
    // شبیه‌سازی یک عملیات دیتابیس
    await Task.Delay(100); 
    
    if (id == 1)
        return (true, new User { Id = 1, Name = "Sara" });
        
    return (false, null!);
}

// نحوه استفاده:
public async Task CallerAsync()
{
    var (isFound, user) = await FindUserAsync(1);
    if (isFound)
        Console.WriteLine(user.Name);
}
```

---

## ۶. Tuple در متدهای Generic

شما می‌توانید Tupleها را با Type Parameterها ترکیب کنید تا متدهای بسیار انعطاف‌پذیری بسازید:

```csharp
public (T1, T2) CreatePair<T1, T2>(T1 first, T2 second)
{
    return (first, second);
}

// استفاده:
var (num, text) = CreatePair(42, "Hello");
```

---

## ۷. طراحی API: Tuple در متدهای Private در برابر Public

این بخش **مهم‌ترین** قسمت برای مهندسی نرم‌افزار و طراحی API است.

### ✅ استفاده در متدهای Private / Internal (مجاز و عالی)
اگر متد شما فقط در داخل همان کلاس یا Assembly استفاده می‌شود، Tuple انتخابی فوق‌العاده است. از ایجاد کلاس‌های بی‌معنی (مثل `UserAndStatusResult`) جلوگیری می‌کند.

```csharp
private (bool IsValid, string ErrorMessage) ValidateUser(User user)
{
    if (user == null) return (false, "User is null");
    if (string.IsNullOrEmpty(user.Email)) return (false, "Email is required");
    
    return (true, string.Empty);
}
```

### ❌ استفاده در Public API (ممنوع / Bad Practice)
هرگز Tuple را به عنوان Return Type در **Public API** (مثل متدهای یک Library، Controllerهای API، یا Serviceهای در معرض دید سایر تیم‌ها) قرار ندهید.

**دلایل ممنوعیت در Public API:**
1. **مشکل در مستندسازی (XML Docs):** نام اعضای Tuple در XML Documentation به درستی برای مصرف‌کننده نمایش داده نمی‌شود.
2. **مشکلات Cross-Language:** اگر مصرف‌کننده API شما از زبان دیگری (مثل F# یا VB.NET یا حتی زبان‌های غیر .NET در صورت استفاده از gRPC/REST) استفاده کند، نام‌گذاری اعضا ممکن است از بین برود و فقط `Item1` و `Item2` ببیند.
3. **شکنندگی در تغییرات (Breaking Changes):** اگر در نسخه بعدی نام `UserId` را به `Id` تغییر دهید، کد مصرف‌کننده‌ای که با Deconstruction یا نام خاص صدا زده بود، Compile Error می‌دهد.
4. **محدودیت در تعداد اعضا:** Tupleها برای ۲ الی ۳ عضو خوب هستند. اگر API شما نیاز به ۵ فیلد داشته باشد، Tuple کد را زشت می‌کند.

**راه‌حل جایگزین برای Public API:**
استفاده از `record` (در C# 9 به بعد) یا `class` / `struct`.

```csharp
// ❌ بد (Public API)
public (int Id, string Name, string Email) GetUserPublic(int id);

// ✅ عالی (Public API)
public record UserDto(int Id, string Name, string Email);
public UserDto GetUserPublic(int id);
```

---

## نکات مهم و اشتباهات رایج

### ⚠️ اشتباهات رایج:
1. **استفاده از Item1 و Item2:** همیشه از Tupleهای نام‌گذاری‌شده (Named Tuples) استفاده کنید. کدهای `result.Item1` به شدت غیرقابل نگهداری (Unmaintainable) هستند.
2. **استفاده از `System.Tuple` به جای `ValueTuple`:** سینتکس `(int, string)` از نوع `System.ValueTuple` است که روی Stack قرار می‌گیرد و سریع است. هرگز از `Tuple.Create()` (که کلاس و روی Heap است) استفاده نکنید مگر اینکه مجبور باشید.
3. **Tupleهای بزرگ:** اگر نیاز به بازگرداندن بیش از ۴-۵ عضو دارید، به جای Tuple یک `record` یا `class` بسازید.

### 💡 نکات مهم:
* **برابری مقادیر (Value Equality):** دو Tuple اگر مقادیر یکسانی داشته باشند، با هم برابرند (`==`).
  ```csharp
  var t1 = (1, "A");
  var t2 = (1, "A");
  Console.WriteLine(t1 == t2); // True
  ```
* **تغییرناپذیری (Immutability) در Deconstruction:** وقتی Tuple را Deconstruct می‌کنید، متغیرهای جدید ایجاد می‌شوند و به ساختار اصلی ارجاع ندارند (چون ValueTuple یک Struct است).

---

## جمع‌بندی

استفاده از **Tuple به عنوان Return Type** یکی از بهترین ویژگی‌های C# برای کاهش Boilerplate Code و افزایش خوانایی در سطح داخلی (Internal/Private) برنامه است. با استفاده از **Named Tuples** و **Deconstruction** می‌توانید کدهایی تمیز و مدرن بنویسید. 
اما به عنوان یک قانون طلایی در طراحی نرم‌افزار: **Tupleها را در مرزهای عمومی (Public APIs) نگه ندارید** و برای آن‌ها از `record`ها یا `DTO`های کلاسیک استفاده کنید.

---

## منابع معتبر

1. [Microsoft Learn: Tuple types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
2. [Microsoft Learn: Deconstruct tuples and other types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct)
3. کتاب C# in Depth (نوشته Jon Skeet) - فصل مربوط به ویژگی‌های C# 7.0

---
*اگر این مقاله برای شما مفید بود، لطفاً به Repository ما Star بدهید و برای مطالعه مقالات آموزشی بعدی (مثل `Pattern Matching` و `Records`) ما را دنبال کنید.*