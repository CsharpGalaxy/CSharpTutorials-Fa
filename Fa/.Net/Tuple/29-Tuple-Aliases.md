این یک مقاله آموزشی جامع، ساختاریافته و روان برای قرار گرفتن در Repository آموزشی شماست. لحن مقاله آموزشی، گام‌به‌گام و مناسب برای توسعه‌دهندگانی است که می‌خواهند کدهای مدرن‌تر و تمیزتری بنویسند.

---

# آموزش جامع Tuple Pattern و Pattern Matching در C#

آیا تا به حال مجبور شده‌اید برای بررسی چندین شرط مختلف، زنجیره‌های طولانی و تو در توی `if-else` بنویسید؟ یا از `switch`های سنتی استفاده کنید که فقط روی یک مقدار کار می‌کنند؟ 
از نسخه 7 به بعد، و به‌ویژه با ورود **Pattern Matching** در C# 8 و 9، مایکروسافت راهی انقلابی برای بررسی "شکل" و "ساختار" داده‌ها ارائه کرده است.

در این مقاله، از مفاهیم پایه Pattern Matching شروع کرده و به سراغ یکی از قدرتمندترین ابزارهای آن، یعنی **Tuple Pattern** می‌رویم.

---

## ۱. Pattern Matching چیست؟ (مقدماتی)

تطبیق الگو (Pattern Matching) تکنیکی است که به شما اجازه می‌دهد بررسی کنید آیا یک متغیر "شکل" یا "الگوی" خاصی دارد یا خیر، و در عین حال داده‌های آن را استخراج کنید.

ساده‌ترین شکل آن، **Type Pattern** است:

```csharp
object data = "Hello World";

// روش قدیمی
if (data is string)
{
    string str = (string)data;
    Console.WriteLine(str.Length);
}

// روش مدرن با Pattern Matching
if (data is string str)
{
    Console.WriteLine(str.Length);
}
```
در روش مدرن، همزمان که چک کردیم `data` از نوع `string` است، آن را در متغیر `str` ریختیم.

---

## ۲. Tuple Pattern چیست؟

تا پیش از این، `switch` فقط می‌توانست **یک مقدار** را بررسی کند. اما در دنیای واقعی، ما اغلب نیاز داریم **چندین مقدار را همزمان** بررسی کنیم. 
اینجاست که **Tuple Pattern** وارد عمل می‌شود. این الگو به شما اجازه می‌دهد چندین متغیر را در قالب یک Tuple ترکیب کرده و در یک `switch expression` آن‌ها را با هم مقایسه کنید.

### استفاده از Tuple در Switch

فرض کنید می‌خواهیم بر اساس "روز هفته" و "وضعیت هوا" تصمیم بگیریم چه کاری انجام دهیم:

```csharp
bool isWeekend = true;
bool isRaining = false;

string activity = (isWeekend, isRaining) switch
{
    (true, true)  => "ماندن در خانه و فیلم دیدن",
    (true, false) => "پیک‌نیک در پارک",
    (false, true) => "رفتن به محل کار با چتر",
    (false, false)=> "رفتن به محل کار",
};

Console.WriteLine(activity); // خروجی: پیک‌نیک در پارک
```
**نکته کلیدی:** ما دو متغیر را در `(isWeekend, isRaining)` قرار دادیم و در هر `case`، یک Tuple متناظر با آن ساختیم.

---

## ۳. مقایسه چند مقدار همزمان و شرط‌های ترکیبی

شما مجبور نیستید همیشه همه مقادیر را چک کنید. با استفاده از **Discard (`_`)** می‌توانید بگویید "این مقدار برایم مهم نیست" (هر چیزی می‌تواند باشد).

```csharp
bool hasTicket = true;
bool isVip = false;

string accessLevel = (hasTicket, isVip) switch
{
    (true, true)   => "دسترسی به بخش VIP",
    (true, _)      => "دسترسی عادی", // اگر بلیط دارد، فرقی نمی‌کند VIP هست یا نه
    (false, _)     => "دسترسی ممنوع",
};
```

### ترکیب با Logical Patterns (`and`, `or`, `not`)
شما می‌توانید الگوها را با کلمات کلیدی منطقی ترکیب کنید:

```csharp
int dayOfWeek = 6; // شنبه
bool isHoliday = true;

string status = (dayOfWeek, isHoliday) switch
{
    (6 or 7, _) => "آخر هفته است",
    (_, true)   => "روز تعطیل است",
    _           => "روز کاری است"
};
```

---

## ۴. سایر الگوهای مرتبط (Property, Positional, Relational)

برای درک بهتر قدرت Pattern Matching، باید با سایر الگوها نیز آشنا باشید:

### الف) Relational Pattern (الگوی رابطه‌ای)
برای مقایسه مقادیر با عملگرهای `<`، `>`، `<=`، `>=`:
```csharp
int score = 85;
string grade = score switch
{
    >= 90 => "A",
    >= 80 => "B", // Relational Pattern
    >= 70 => "C",
    _     => "F"
};
```

### ب) Property Pattern (الگوی پراپرتی)
برای بررسی ویژگی‌های (Properties) یک شیء:
```csharp
var order = new Order { TotalPrice = 1500, IsPaid = true };

string discount = order switch
{
    { TotalPrice: > 1000, IsPaid: true } => "تخفیف ۱۰ درصدی",
    { TotalPrice: > 500 }                => "تخفیف ۵ درصدی",
    _                                    => "بدون تخفیف"
};
```

### ج) Positional Pattern (الگوی موقعیتی)
بسیار شبیه به Tuple Pattern است، اما برای **اشیاء**ای استفاده می‌شود که متد `Deconstruct` دارند (مثل `record`ها):
```csharp
public record Point(int X, int Y);

Point p = new(0, 0);

string location = p switch
{
    Point(0, 0) => "مرکز مختصات",
    Point(0, _) => "روی محور Y",
    Point(_, 0) => "روی محور X",
    _           => "در فضای دو بعدی"
};
```

---

## ۵. مثال واقعی: قبل و بعد از Pattern Matching

بیایید یک سناریوی واقعی را ببینیم. می‌خواهیم **هزینه ارسال** را بر اساس `نوع مشتری` و `وزن بسته` محاسبه کنیم.

### ❌ روش قدیمی (Before)
```csharp
public decimal CalculateShipping(CustomerType type, decimal weight)
{
    if (type == CustomerType.VIP)
    {
        if (weight > 10) return 0; // رایگان
        else return 5;
    }
    else if (type == CustomerType.Regular)
    {
        if (weight > 10) return 15;
        else return 10;
    }
    else // Guest
    {
        return 20;
    }
}
```
*مشکلات: تو در تو بودن، خوانایی پایین، احتمال فراموش کردن یک حالت.*

### ✅ روش مدرن با Tuple Pattern (After)
```csharp
public decimal CalculateShipping(CustomerType type, decimal weight) =>
    (type, weight) switch
    {
        (CustomerType.VIP, > 10)   => 0m,
        (CustomerType.VIP, _)      => 5m,
        (CustomerType.Regular, > 10)=> 15m,
        (CustomerType.Regular, _)  => 10m,
        (_, _)                     => 20m // حالت پیش‌فرض (Guest یا هر چیز دیگر)
    };
```
*مزایا: تخت (Flat)، به‌شدت خوانا، و کامپایلر تضمین می‌کند که تمام حالت‌ها پوشش داده شده‌اند.*

---

## ۶. بخش پیشرفته (Advanced)

### الف) Exhaustiveness (پوشش کامل)
یکی از بزرگترین مزیت‌های `switch expression` این است که کامپایلر C# هوشمند است. اگر شما همه حالت‌های ممکن یک `Enum` یا یک Tuple را پوشش ندهید و از `_` (حالت پیش‌فرض) استفاده نکنید، **کامپایلر به شما خطا (Warning/Error) می‌دهد**. این یعنی باگ‌های ناشی از فراموشکاری در زمان کامپایل شکار می‌شوند!

### ب) Nested Patterns (الگوهای تو در تو)
شما می‌توانید Property Pattern را درون Tuple Pattern یا برعکس استفاده کنید:
```csharp
var request = new HttpRequest { Method = "GET", Path = "/api/users", User = new User { Role = "Admin" } };

string access = (request.Method, request.Path, request.User) switch
{
    ("GET", "/api/users", { Role: "Admin" or "Manager" }) => "دسترسی کامل",
    ("GET", "/api/users", _)                              => "دسترسی محدود",
    ("DELETE", _, { Role: "Admin" })                      => "حذف مجاز است",
    _                                                     => "دسترسی غیرمجاز"
};
```

### ج) عملکرد (Performance)
آیا Pattern Matching کند است؟ **خیر!** 
کامپایلر Roslyn در C# به شدت بهینه شده است. کدهای تولید شده توسط `switch expression` در بسیاری از موارد حتی از زنجیره‌های `if-else` دستی که توسط برنامه‌نویس نوشته شده‌اند نیز سریع‌تر اجرا می‌شوند، زیرا کامپایلر بهترین درخت تصمیم‌گیری (Decision Tree) را برای آن می‌سازد.

---

## نتیجه‌گیری

استفاده از **Tuple Pattern** و سایر الگوهای تطبیق، کدهای C# شما را از حالت "دستوری" (Imperative) به حالت "اعلامی" (Declarative) تبدیل می‌کند. شما به جای اینکه به کامپایلر بگویید *چگونه* شرایط را چک کند، به آن می‌گویید *چه شکلی* از داده مد نظرتان است. این امر باعث کاهش باگ‌ها، افزایش خوانایی و نگهداری آسان‌تر کد می‌شود.

---

## 📚 منابع رسمی برای مطالعه بیشتر

برای تسلط کامل، پیشنهاد می‌کنیم مستندات زیر از مایکروسافت را مطالعه کنید:

1. **[Pattern Matching Overview (C#)](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)**
   *مرجع اصلی برای درک مفهوم کلی Pattern Matching.*
2. **[Switch Expression (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression)**
   *راهنمای جامع استفاده از switch expression.*
3. **[Tuple Patterns (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#tuple-pattern)**
   *توضیحات تخصصی درباره Tuple و Positional Patterns.*
4. **[What's new in C# 9.0 (Pattern Matching)](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#pattern-matching)**
   *بررسی الگوهای منطقی (and, or, not) و Relational patterns.*

---
*نویسنده: [نام شما / نام Repository شما]*
*تاریخ نگارش: August 2026*