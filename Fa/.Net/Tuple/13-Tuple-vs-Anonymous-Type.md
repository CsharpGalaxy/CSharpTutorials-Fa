# نام‌گذاری اعضای Tuple در C# — راهنمای کامل و کاربردی

> **سطح:** مبتدی تا پیشرفته  
> **نسخه مورد نیاز:** C# 7.0 به بالا (.NET Core 2.0+ / .NET Framework 4.7+)  
> **زمان مطالعه:** حدود ۱۵ دقیقه

---

## فهرست مطالب

1. [مقدمه](#مقدمه)
2. [Tuple Element Names چیست؟](#tuple-element-names-چیست)
3. [چرا اعضای Tuple را نام‌گذاری می‌کنیم؟](#چرا-اعضای-tuple-را-نامگذاری-میکنیم)
4. [Syntax نام‌گذاری](#syntax-نامگذاری)
5. [تفاوت `(string, int)` با `(string Name, int Age)`](#تفاوت-string-int-با-string-name-int-age)
6. [استفاده از نام متغیرها](#استفاده-از-نام-متغیرها)
7. [Name Inference (استنتاج خودکار نام)](#name-inference-استنتاج-خودکار-نام)
8. [استفاده از نام Field](#استفاده-از-نام-field)
9. [استفاده از نام Property](#استفاده-از-نام-property)
10. [چه زمانی کامپایلر نام عضو را استنتاج می‌کند؟](#چه-زمانی-کامپایلر-نام-عضو-را-استنتاج-میکند)
11. [چه زمانی از Item1 و Item2 استفاده می‌شود؟](#چه-زمانی-از-item1-و-item2-استفاده-میشود)
12. [محدودیت‌های نام‌گذاری](#محدودیتهای-نامگذاری)
13. [تفاوت نام منطقی عضو با نام Runtime Field](#تفاوت-نام-منطقی-عضو-با-نام-runtime-field)
14. [مثال‌های کاربردی](#مثالهای-کاربردی)
15. [نکات مهم](#نکات-مهم)
16. [اشتباهات رایج](#اشتباهات-رایج)
17. [جمع‌بندی](#جمع‌بندی)
18. [منابع معتبر](#منابع-معتبر)

---

## مقدمه

در نسخه‌های قدیمی C#، وقتی می‌خواستیم چند مقدار را با هم برگردانیم، یا باید از `out` parameter استفاده می‌کردیم، یا یک کلاس اختصاصی می‌ساختیم، یا از `Tuple` قدیمی استفاده می‌کردیم که اعضای آن فقط با نام‌های بی‌معنای `Item1`، `Item2` و ... قابل دسترسی بودند.

از **C# 7.0** به بعد، مایکروسافت قابلیت **نام‌گذاری اعضای Tuple** را اضافه کرد تا کدها خواناتر، معناتر و قابل‌نگهداری‌تر شوند.

🔗 [برای آشنایی اولیه با Tuple، مقاله «Tuple در C# چیست؟» را بخوانید.](#)

---

## Tuple Element Names چیست؟

**Tuple Element Names** (نام اعضای Tuple) به شما اجازه می‌دهد برای هر عضو یک Tuple، یک **نام معنادار** تعریف کنید تا به‌جای `Item1` و `Item2`، از نام‌هایی مثل `Name` و `Age` استفاده کنید.

```csharp
// بدون نام‌گذاری
var person = ("Ali", 30);
Console.WriteLine(person.Item1); // "Ali"

// با نام‌گذاری
var person = (Name: "Ali", Age: 30);
Console.WriteLine(person.Name); // "Ali"
Console.WriteLine(person.Age);  // 30
```

در واقع، نام‌گذاری فقط یک **برچسب معنایی** روی اعضای Tuple است و نوع داده را تغییر نمی‌دهد.

---

## چرا اعضای Tuple را نام‌گذاری می‌کنیم؟

دلایل اصلی:

| دلیل | توضیح |
|------|--------|
| **خوانایی کد** | `result.Success` بسیار خواناتر از `result.Item1` است |
| **قصد نویسنده** | نام‌ها، نیت برنامه‌نویس را نشان می‌دهند |
| **کاهش خطا** | دسترسی اشتباه به `Item3` به‌جای `Item2` سخت تشخیص داده می‌شود |
| **IntelliSense بهتر** | IDE نام معنادار را نشان می‌دهد |
| **جایگزین کلاس‌های موقت** | دیگر نیازی به ساخت کلاس‌های کوچک برای برگرداندن چند مقدار نیست |

---

## Syntax نام‌گذاری

ساختار کلی به شکل زیر است:

```csharp
(نوع نام, نوع نام, ...)
```

### مثال ۱: تعریف نوع با نام

```csharp
(string Name, int Age) person = ("Sara", 25);
Console.WriteLine(person.Name); // Sara
Console.WriteLine(person.Age);  // 25
```

### مثال ۲: مقداردهی مستقیم

```csharp
var point = (X: 10, Y: 20);
Console.WriteLine(point.X); // 10
Console.WriteLine(point.Y); // 20
```

### مثال ۳: در متد برگرداننده

```csharp
(string City, int Population) GetCityInfo()
{
    return ("Tehran", 9000000);
}

var info = GetCityInfo();
Console.WriteLine($"{info.City} - {info.Population}");
```

---

## تفاوت `(string, int)` با `(string Name, int Age)`

این دو از نظر **نوع داده** کاملاً یکسان هستند، اما از نظر **خوانایی** تفاوت دارند:

```csharp
// نوع بدون نام
(string, int) p1 = ("Ali", 30);
Console.WriteLine(p1.Item1); // "Ali"

// نوع با نام
(string Name, int Age) p2 = ("Ali", 30);
Console.WriteLine(p2.Name); // "Ali"
```

### نکته مهم: تطابق نام‌ها در انتساب

وقتی دو Tuple با نام‌های متفاوت را به هم منتسب می‌کنیم، کامپایلر از **نام‌های سمت چپ** استفاده می‌کند:

```csharp
(string Name, int Age) person = ("Ali", 30);
(string FullName, int Years) data = person;

Console.WriteLine(data.FullName); // "Ali" - نام سمت چپ اعمال می‌شود
Console.WriteLine(data.Years);    // 30
```

> ⚠️ نام‌ها در حین انتساب، از روی متغیر سمت چپ برداشته می‌شوند، نه سمت راست.

---

## استفاده از نام متغیرها

اگر از یک متغیر در ساخت Tuple استفاده کنید، کامپایلر به‌طور خودکار نام متغیر را به‌عنوان نام عضو Tuple در نظر می‌گیرد:

```csharp
string name = "Reza";
int age = 40;

var person = (name, age);

Console.WriteLine(person.name); // "Reza"
Console.WriteLine(person.age);  // 40
```

این ویژگی بسیار کاربردی است و از تکرار نام‌ها جلوگیری می‌کند.

---

## Name Inference (استنتاج خودکار نام)

**Name Inference** یکی از قابلیت‌های هوشمند C# است که در آن کامپایلر نام اعضای Tuple را از روی **نام متغیرها، فیلدها یا Propertyهای** استفاده‌شده در مقداردهی، حدس می‌زند.

### قانون کلی:
اگر نام عضوی را مشخص نکنید، کامپایلر سعی می‌کند نام را از منبع مقدار استنتاج کند.

```csharp
string firstName = "Ali";
int age = 30;

var person = (firstName, age);
// معادل است با:
// (string firstName, int age) person = (firstName, age);

Console.WriteLine(person.firstName); // "Ali"
Console.WriteLine(person.age);       // 30
```

### ترکیب نام دستی و استنتاجی:

```csharp
string name = "Sara";
int age = 25;

var person = (FullName: name, age);
Console.WriteLine(person.FullName); // "Sara"
Console.WriteLine(person.age);      // 25
```

---

## استفاده از نام Field

اگر مقدار یک عضو Tuple از یک **Field** گرفته شود، کامپایلر نام آن Field را به‌عنوان نام عضو در نظر می‌گیرد:

```csharp
class Person
{
    public string Name;
    public int Age;

    public void Show()
    {
        var info = (Name, Age);
        Console.WriteLine(info.Name); // مقدار فیلد Name
        Console.WriteLine(info.Age);  // مقدار فیلد Age
    }
}

var p = new Person { Name = "Mina", Age = 28 };
p.Show();
```

---

## استفاده از نام Property

همین قانون برای **Property**ها نیز صادق است:

```csharp
class User
{
    public string UserName { get; set; }
    public int Score { get; set; }

    public void Display()
    {
        var result = (UserName, Score);
        Console.WriteLine(result.UserName); // "User1"
        Console.WriteLine(result.Score);    // 100
    }
}

var u = new User { UserName = "User1", Score = 100 };
u.Display();
```

---

## چه زمانی کامپایلر نام عضو را استنتاج می‌کند؟

کامپایلر در موارد زیر نام را استنتاج می‌کند:

| منبع مقدار | نام استنتاج‌یافته |
|------------|--------------------|
| متغیر محلی | نام متغیر |
| فیلد کلاس | نام فیلد |
| Property | نام Property |
| عبارت ثابت (Constant) | ❌ استنتاج نمی‌شود |
| متد یا فراخوانی | ❌ استنتاج نمی‌شود |
| عبارت پیچیده | ❌ استنتاج نمی‌شود |

### مثال‌ها:

```csharp
int count = 5;
string title = "Hello";

// ✅ استنتاج می‌شود
var t1 = (count, title);        // (int count, string title)

// ❌ استنتاج نمی‌شود
var t2 = (10, "World");         // (int, string) -> Item1, Item2
var t3 = (GetCount(), title);   // (int, string title) - فقط title استنتاج می‌شود
```

---

## چه زمانی از Item1 و Item2 استفاده می‌شود؟

در حالت‌های زیر، اعضای Tuple فقط با `Item1`، `Item2` و ... در دسترس هستند:

### ۱. وقتی هیچ نامی مشخص نکرده باشید:

```csharp
var point = (10, 20);
Console.WriteLine(point.Item1); // 10
Console.WriteLine(point.Item2); // 20
```

### ۲. وقتی نوع Tuple را بدون نام تعریف کنید:

```csharp
(int, int) point = (10, 20);
Console.WriteLine(point.Item1); // 10
```

### ۳. وقتی نام‌ها در سمت چپ انتساب حذف شوند:

```csharp
(string, int) data = (Name: "Ali", Age: 30);
Console.WriteLine(data.Item1); // "Ali"
// data.Name ❌ خطا - چون نوع بدون نام تعریف شده
```

> 💡 **نکته:** حتی اگر نام‌گذاری کرده باشید، همچنان می‌توانید از `Item1`، `Item2` و ... استفاده کنید:

```csharp
var person = (Name: "Ali", Age: 30);
Console.WriteLine(person.Item1); // "Ali" - همچنان کار می‌کند
Console.WriteLine(person.Name);  // "Ali"
```

---

## محدودیت‌های نام‌گذاری

نام‌گذاری اعضای Tuple قوانین خاصی دارد:

### ۱. نام‌ها نمی‌توانند رزرو شده باشند

نام‌های `Item1` تا `Item17` و `Rest` برای استفاده داخلی Tuple محفوظ هستند:

```csharp
// ❌ خطا
var bad = (Item1: 10, Item2: 20); // Error CS8126
```

### ۲. نام‌ها باید یکتا باشند

```csharp
// ❌ خطا
var bad = (Value: 10, Value: 20); // Error CS8139
```

### ۳. نام‌ها باید شناسه معتبر C# باشند

```csharp
// ❌ خطا
var bad = (123abc: 10); // Error

// ✅ با @ می‌توان از کلمات کلیدی استفاده کرد
var ok = (@class: 10, @int: 20);
Console.WriteLine(ok.@class); // 10
```

### ۴. نام‌گذاری روی `ValueTuple` خالی مجاز نیست

```csharp
// ❌ خطا
var empty = (Name:); // Error
```

### ۵. نام‌ها در Signature متد تأثیری ندارند

دو متد زیر از نظر کامپایلر **یکسان** هستند و نمی‌توانند Overload شوند:

```csharp
// ❌ Error CS0111 - Type already defines a member
(string Name, int Age) GetPerson() => ("Ali", 30);
(string FullName, int Years) GetPerson() => ("Ali", 30);
```

> ⚠️ این یک محدودیت بسیار مهم است: نام اعضای Tuple بخشی از **نوع** نیستند، بلکه فقط **متادیتا** هستند.

---

## تفاوت نام منطقی عضو با نام Runtime Field

این یکی از مهم‌ترین نکات فنی است که باید درک کنید.

### در زمان کامپایل (Compile Time):
- کامپایلر نام‌هایی که شما تعیین کرده‌اید را می‌شناسد
- IntelliSense و type checking با این نام‌ها کار می‌کنند

### در زمان اجرا (Runtime):
- نام‌های شما **حذف می‌شوند**
- فیلدهای واقعی در شیء، همیشه `Item1`, `Item2`, ... هستند
- نام‌ها فقط در **متادیتای اسمبلی** (از طریق `TupleElementNamesAttribute`) ذخیره می‌شوند

### اثبات عملی:

```csharp
var person = (Name: "Ali", Age: 30);

var type = person.GetType();
Console.WriteLine(type.FullName);
// System.ValueTuple`2[System.String,System.Int32]

foreach (var field in type.GetFields())
{
    Console.WriteLine(field.Name);
}
// خروجی:
// Item1
// Item2
```

### بررسی متادیتا با Reflection:

```csharp
var method = typeof(Program).GetMethod(nameof(GetPerson));
var attr = method.ReturnTypeCustomAttributes
    .GetCustomAttributes(typeof(System.Runtime.CompilerServices.TupleElementNamesAttribute), false)
    .FirstOrDefault() as System.Runtime.CompilerServices.TupleElementNamesAttribute;

if (attr != null)
{
    foreach (var name in attr.TransformNames)
        Console.WriteLine(name);
    // Name
    // Age
}

(string Name, int Age) GetPerson() => ("Ali", 30);
```

> 💡 **نتیجه:** نام‌های Tuple فقط یک **توهم کامپایلر** (Compiler Illusion) هستند که خوانایی را بهبود می‌بخشند، اما در Runtime وجود خارجی ندارند.

### تأثیر در Serialization:

```csharp
var person = (Name: "Ali", Age: 30);
var json = JsonSerializer.Serialize(person);
// خروجی: {"Item1":"Ali","Item2":30}
// ❌ نام‌ها در JSON ظاهر نمی‌شوند!
```

> ⚠️ اگر به نام‌ها در JSON نیاز دارید، باید از `record` یا کلاس استفاده کنید، نه Tuple.

---

## مثال‌های کاربردی

### مثال ۱: برگرداندن چند مقدار از یک متد

```csharp
(bool Success, string Message, int ErrorCode) ValidateUser(string username, string password)
{
    if (string.IsNullOrEmpty(username))
        return (false, "Username is required", 1001);

    if (password.Length < 8)
        return (false, "Password too short", 1002);

    return (true, "Valid", 0);
}

var result = ValidateUser("ali", "123");
if (result.Success)
    Console.WriteLine("Welcome!");
else
    Console.WriteLine($"Error {result.ErrorCode}: {result.Message}");
```

### مثال ۲: محاسبه مختصات

```csharp
(double X, double Y, double Distance) GetPointWithDistance(double x, double y)
{
    double distance = Math.Sqrt(x * x + y * y);
    return (x, y, distance);
}

var point = GetPointWithDistance(3, 4);
Console.WriteLine($"({point.X}, {point.Y}) - Distance: {point.Distance}");
// (3, 4) - Distance: 5
```

### مثال ۳: LINQ و نام‌گذاری

```csharp
var students = new[] {
    new { Name = "Ali", Score = 18 },
    new { Name = "Sara", Score = 20 },
    new { Name = "Reza", Score = 15 }
};

var results = students.Select(s => (StudentName: s.Name, Grade: s.Score >= 17 ? "A" : "B"));

foreach (var r in results)
    Console.WriteLine($"{r.StudentName}: {r.Grade}");
```

### مثال ۴: Deconstruction با نام

```csharp
(string City, int Population) GetCity() => ("Isfahan", 2000000);

var (city, population) = GetCity();
Console.WriteLine($"{city} has {population} people");
```

### مثال ۵: Tuple تودرتو با نام

```csharp
var data = (
    Person: (Name: "Ali", Age: 30),
    Address: (City: "Tehran", Zip: 12345)
);

Console.WriteLine(data.Person.Name);  // "Ali"
Console.WriteLine(data.Address.City); // "Tehran"
```

---

## نکات مهم

✅ **نکته ۱:** نام‌گذاری اعضای Tuple اختیاری است. می‌توانید برخی اعضا را نام‌گذاری کنید و برخی را نه:

```csharp
var mixed = (Name: "Ali", 30);
Console.WriteLine(mixed.Name);  // "Ali"
Console.WriteLine(mixed.Item2); // 30
```

✅ **نکته ۲:** نام‌ها در **تطابق نوع** تأثیری ندارند:

```csharp
(string Name, int Age) p1 = ("Ali", 30);
(string FullName, int Years) p2 = p1; // ✅ بدون خطا
```

✅ **نکته ۳:** Tupleها **مقداری (Value Type)** هستند و روی Stack ذخیره می‌شوند (مگر اینکه box شوند).

✅ **نکته ۴:** حداکثر ۷ عضو مستقیم می‌توانید داشته باشید. برای بیشتر، از `Rest` استفاده می‌شود:

```csharp
var bigTuple = (1, 2, 3, 4, 5, 6, 7, 8); // 8 تا
Console.WriteLine(bigTuple.Rest.Item1); // 8
```

✅ **نکته ۵:** Tupleها **تغییرناپذیر (Immutable) نیستند** — می‌توانید مقادیرشان را تغییر دهید:

```csharp
var point = (X: 10, Y: 20);
point.X = 30; // ✅ مجاز
```

✅ **نکته ۶:** برای مقایسه دو Tuple، از `==` استفاده کنید (برابر بودن مقادیر):

```csharp
var a = (Name: "Ali", Age: 30);
var b = (Name: "Ali", Age: 30);
Console.WriteLine(a == b); // True
```

---

## اشتباهات رایج

### ❌ اشتباه ۱: انتظار داشتن نام‌ها در Reflection یا Serialization

```csharp
var person = (Name: "Ali", Age: 30);
var json = JsonSerializer.Serialize(person);
// ❌ انتظار دارید {"Name":"Ali","Age":30}
// ✅ در واقع: {"Item1":"Ali","Item2":30}
```

**راه‌حل:** از `record` یا کلاس استفاده کنید.

### ❌ اشتباه ۲: تلاش برای Overload کردن متد با نام‌های متفاوت

```csharp
// ❌ Error
(int X, int Y) GetPoint() => (0, 0);
(int A, int B) GetPoint() => (0, 0); // CS0111
```

### ❌ اشتباه ۳: استفاده از نام‌های رزرو

```csharp
// ❌ Error CS8126
var t = (Item1: 1, Item2: 2);
```

### ❌ اشتباه ۴: فرض اینکه نام‌ها در نوع ذخیره شده‌اند

```csharp
(string Name, int Age) p1 = ("Ali", 30);
(string FullName, int Years) p2 = p1;
Console.WriteLine(p2.Name); // ❌ Error - نام‌های p2 متفاوتند
Console.WriteLine(p2.FullName); // ✅
```

### ❌ اشتباه ۵: فراموش کردن `@` برای کلمات کلیدی

```csharp
// ❌ Error
var t = (class: 10);

// ✅ درست
var t = (@class: 10);
```

### ❌ اشتباه ۶: استفاده از Tuple به‌جای record برای داده‌های پیچیده

اگر داده‌های شما نیاز به Validation، Method، یا Serialization دارند، از `record` استفاده کنید:

```csharp
// ❌ Tuple برای داده‌های پیچیده مناسب نیست
(string Name, int Age) person = ("Ali", 30);

// ✅ record بهتر است
public record Person(string Name, int Age);
var person = new Person("Ali", 30);
```

---

## جمع‌بندی

| موضوع | توضیح |
|-------|--------|
| **نام‌گذاری** | `(string Name, int Age)` برای خوانایی بیشتر |
| **Name Inference** | کامپایلر نام متغیر/فیلد/Property را استنتاج می‌کند |
| **Item1, Item2** | زمانی استفاده می‌شوند که نامی تعیین نکرده باشید |
| **محدودیت‌ها** | نام‌های رزرو، تکراری، و کلمات کلیدی ممنوع |
| **Runtime** | نام‌ها در Runtime وجود ندارند، فقط `Item1`, `Item2` هستند |
| **Serialization** | نام‌ها در JSON ظاهر نمی‌شوند |
| **بهترین کاربرد** | برگرداندن چند مقدار از متد، LINQ، Deconstruction |

### 🎯 قوانین طلایی:

1. **برای کدهای داخلی و موقت** → از Tuple با نام استفاده کنید
2. **برای APIهای عمومی و داده‌های ماندگار** → از `record` یا `class` استفاده کنید
3. **همیشه نام‌گذاری کنید** تا کد خوانا باشد، مگر در موارد بسیار ساده
4. **مراقب Serialization باشید** — Tuple نام‌ها را حفظ نمی‌کند

---

## منابع معتبر

📚 **مستندات رسمی مایکروسافت:**
- [Tuple types - C# documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
- [TupleElementNamesAttribute](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.tupleelementnamesattribute)

📖 **کتاب‌ها:**
- *C# 12 in a Nutshell* — Joseph Albahari
- *C# in Depth* — Jon Skeet (فصل مربوط به Tuple)

🎥 **ویدیوهای آموزشی:**
- [Nick Chapsas — Tuples in C#](https://www.youtube.com/results?search_query=tuples+c%23)
- [IAmTimCorey — C# Tuples Explained](https://www.youtube.com/results?search_query=tim+corey+tuples)

🔗 **مقالات مرتبط در همین Repository:**
- [آشنایی با Tuple در C#](#)
- [تفاوت Tuple با Record و Class](#)
- [Deconstruction در C#](#)
- [Value Type vs Reference Type](#)

---

> 💬 **سوال یا پیشنهادی دارید؟**  
> می‌توانید در بخش Issues همین Repository مطرح کنید یا Pull Request ارسال نمایید.

**نویسنده:** [نام شما]  
**تاریخ بازبینی:** August 2026  
**نسخه C#:** 7.0 تا 13.0

---

*اگر این مقاله برای شما مفید بود، لطفاً با ⭐ دادن به Repository از ما حمایت کنید.*