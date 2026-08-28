# آموزش جامع `with` Expression در C# (از مقدماتی تا پیشرفته)

عبارت `with` یکی از جذاب‌ترین و کاربردی‌ترین ویژگی‌هایی است که از **C# 9.0** و همزمان با معرفی **Records** به این زبان اضافه شد. این آموزش شما را از صفر مطلق تا مفاهیم عمیق و سطح پایین (Under the hood) این ویژگی همراهی خواهد کرد.

---

## ۱. `with` چیست؟
در برنامه‌نویسی، ما اغلب با اشیائی سر و کار داریم که **Immutable** (غیرقابل تغییر) هستند. تغییر دادن مستقیم یک Property در این اشیاء ممکن نیست. 
عبارت `with` برای حل این مشکل طراحی شده است. این عبارت به شما اجازه می‌دهد یک **کپی** از یک شیء موجود بگیرید و در همان لحظه، یک یا چند Property آن را تغییر دهید. به این کار **Non-destructive Mutation** (تغییرات غیرمخرب) می‌گویند، زیرا شیء اصلی دست‌نخورده باقی می‌ماند و یک شیء جدید تولید می‌شود.

---

## ۲. Syntax (ساختار نگارش)
ساختار `with` بسیار شبیه به Object Initializer است، با این تفاوت که به جای `new`، از کلمه کلیدی `with` استفاده می‌کنیم:

```csharp
var newObj = existingObj with 
{ 
    Property1 = newValue1, 
    Property2 = newValue2 
};
```

---

## ۳. مثال‌های کاربردی

### مثال ساده و تغییر یک Property
فرض کنید یک Record برای یک `Person` داریم:

```csharp
public record Person(string Name, int Age, string City);

var ali = new Person("Ali", 30, "Tehran");

// تغییر فقط یک Property (سن)
var olderAli = ali with { Age = 31 }; 

Console.WriteLine(ali);       // Output: Person { Name = Ali, Age = 30, City = Tehran }
Console.WriteLine(olderAli);  // Output: Person { Name = Ali, Age = 31, City = Tehran }
```
*توضیح:* شیء `ali` هیچ تغییری نکرد. کامپایلر یک کپی از `ali` گرفت، `Age` را به ۳۱ تغییر داد و در `olderAli` قرار داد.

### تغییر چند Property
شما می‌توانید هر تعداد Property که نیاز دارید را در یک عبارت `with` تغییر دهید:

```csharp
var movedAli = ali with { Age = 31, City = "London" };
// Output: Person { Name = Ali, Age = 31, City = London }
```

---

## ۴. رفتار `with` در انواع مختلف Record

### الف) `with` در Record Class
رکوردهای کلاسی (Reference Type) به صورت پیش‌فرض در C# 9 معرفی شدند. در اینجا `with` یک کپی از آبجکت در Heap ایجاد می‌کند.

```csharp
public record User(string Username, string Email);
var user1 = new User("admin", "admin@test.com");
var user2 = user1 with { Email = "new@test.com" };
```

### ب) `with` در Record Struct
از C# 10، رکوردها می‌توانند Value Type باشند. در اینجا `with` روی Stack (یا درون آبجکت‌های دیگر) یک کپی بایت‌به‌بایت از ساختار ایجاد می‌کند.

```csharp
public record struct Point(int X, int Y);
var p1 = new Point(10, 20);
var p2 = p1 with { X = 50 };
```

### ج) `with` در `readonly record struct`
این ساختار کاملاً Immutable است. تمام Propertyها فقط `get` دارند. در اینجا `with` **حیاتی‌ترین** نقش را دارد، زیرا تنها راه برای ساخت یک مقدار جدید بر اساس مقادیر قبلی است.

```csharp
public readonly record struct Color(byte R, byte G, byte B);

var red = new Color(255, 0, 0);
// چون هیچ set ای وجود ندارد، فقط با with می‌توانیم رنگ جدید بسازیم:
var darkRed = red with { R = 139 }; 
```

---

## ۵. مفاهیم عمیق: Clone، Copy Constructor و Shallow Copy

برای درک پیشرفته `with`، باید بدانیم در پشت صحنه چه می‌گذرد.

### نقش Clone و Copy Constructor
وقتی شما یک Record تعریف می‌کنید، کامپایلر C# به صورت خودکار یک **Copy Constructor** (سازنده کپی) محافظت‌شده (protected) برای آن تولید می‌کند. 
عبارت `with` در واقع از همین Copy Constructor استفاده می‌کند تا یک کپی (Clone) از شیء اصلی بسازد و سپس مقادیر جدید را روی آن اعمال کند.

### Shallow Copy (کپی سطحی) و Reference Typeهای داخلی
**بسیار مهم:** عبارت `with` یک **Shallow Copy** انجام می‌دهد، نه Deep Copy!
اگر Record شما شامل یک Reference Type (مثل یک `List`، `Array` یا یک `class` دیگر) باشد، کپی جدید به **همان instancia** از آن Reference Type در حافظه اشاره می‌کند.

**مثال برای درک بهتر:**

```csharp
public record Student(string Name, List<string> Courses);

var courseList = new List<string> { "Math", "Physics" };
var student1 = new Student("Sara", courseList);

// استفاده از with
var student2 = student1 with { Name = "Sarah" };

// حالا لیست درون student2 را تغییر می‌دهیم:
student2.Courses.Add("Chemistry");

Console.WriteLine(string.Join(", ", student1.Courses)); 
// Output: Math, Physics, Chemistry
```
*توضیح:* با اینکه ما `student2` را ساختیم، اما چون `Courses` یک Reference Type است، `student1.Courses` و `student2.Courses` هر دو به **یک لیست واحد در حافظه** اشاره می‌کنند. تغییر در یکی، روی دیگری اثر می‌گذارد.

---

## ۶. تحلیل رفتار کامپایلر (مراحل مفهومی)

بیایید این کد را بررسی کنیم:
```csharp
var updated = original with
{
    Name = "New Name"
};
```

وقتی شما این کد را کامپایل می‌کنید، کامپایلر C# از نظر مفهومی (Conceptually) مراحل زیر را به IL (زبان میانی) تبدیل می‌کند:

1. **فراخوانی Copy Constructor:** کامپایلر ابتدا Copy Constructorِ تولید شده توسط کامپایلر را برای `original` فراخوانی می‌کند. این کار باعث می‌شود یک کپی (Shallow Copy) از تمام فیلدهای `original` در یک شیء موقت جدید کپی شود.
2. **اعمال تغییرات (Assignment):** سپس، کامپایلر کد شما را می‌خواند و مقدار `"New Name"` را در Property `Name`ِ آن شیء موقت جدید تنظیم می‌کند. (دقیقاً مثل اینکه بنویسد `temp.Name = "New Name"`).
3. **انتساب (Assignment):** در نهایت، مرجعِ آن شیء موقتِ تغییر یافته، به متغیر `updated` اختصاص داده می‌شود.

**معادل کد تولید شده توسط کامپایلر (به زبان شبه‌کد):**
```csharp
// 1. فراخوانی کپی سازنده
var temp = new OriginalType(original); 

// 2. اعمال تغییرات با استفاده از with
temp.Name = "New Name";

// 3. مقداردهی نهایی
var updated = temp;
```

---

## ۷. محدودیت‌های `with`

1. **فقط برای Records و Anonymous Types:** عبارت `with` فقط روی `record`ها، `record struct`ها و `anonymous type`ها کار می‌کند. شما نمی‌توانید از آن روی `class`ها یا `struct`های معمولی استفاده کنید (مگر اینکه خودتان آن را پیاده‌سازی کنید که پیچیده و غیرمعمول است).
2. **فقط Propertyهای `init` یا `set`:** شما فقط می‌توانید Propertyهایی را در `with` تغییر دهید که دارای `init` یا `set` accessor باشند. Propertyهایی که فقط `get` دارند (و در constructor مقداردهی نشده‌اند) یا فیلدهای `readonly` قابل تغییر با `with` نیستند.
3. **عدم تغییر نوع (No Polymorphic Type Change):** شما نمی‌توانید با `with` نوع شیء را تغییر دهید. اگر `original` از نوع `Dog` باشد، خروجی `with` هم قطعاً از نوع `Dog` خواهد بود، حتی اگر آن را در متغیری از نوع `Animal` قرار دهید.
4. **عدم پشتیبانی از Indexer:** نمی‌توانید مستقیماً ایندکسرهای یک آرایه یا لیست را درون `with` تغییر دهید (مثلاً `with { Items[0] = "x" }` خطا می‌دهد).

---

## ۸. اشتباهات رایج در استفاده از `with`

### اشتباه ۱: انتظار Deep Copy داشتن
بزرگترین اشتباه توسعه‌دهندگان این است که فکر می‌کنند `with` یک کپی عمیق (Deep Copy) می‌سازد. همانطور که در بخش Reference Typeها دیدیم، اشیاء داخلی (Nested Objects) به صورت مرجع کپی می‌شوند. اگر به Deep Copy نیاز دارید، باید خودتان آن را در یک متد یا Copy Constructor سفارشی پیاده‌سازی کنید.

### اشتباه ۲: استفاده در حلقه‌های با کارایی بالا (Performance Trap)
از آنجا که `with` در هر بار اجرا یک شیء جدید در حافظه (Heap برای کلاس‌ها)_alloc_ می‌کند، استفاده از آن درون حلقه‌های `for` یا `foreach` با میلیون‌ها تکرار، باعث ایجاد فشار شدید به Garbage Collector و کاهش Performance می‌شود. در چنین سناریوهایی از `struct`های معمولی و تغییر مستقیم (Mutation) استفاده کنید.

### اشتباه ۳: تلاش برای تغییر فیلدها به جای Propertyها
عبارت `with` فقط با Propertyها کار می‌کند. اگر Record شما فیلدهای عمومی (Public Fields) داشته باشد، `with` آن‌ها را نمی‌شناسد و خطای کامپایل می‌دهد.

### اشتباه ۴: فراموش کردن `init` در Propertyهای سفارشی
اگر یک Property سفارشی (Custom Property) در Record تعریف کنید، حتماً باید `init` یا `set` داشته باشد تا `with` بتواند آن را تنظیم کند:
```csharp
// اشتباه:
public record MyRecord 
{
    public string Name { get; } // فقط get دارد، with کار نمی‌کند
}

// درست:
public record MyRecord 
{
    public string Name { get; init; } 
}
```

---

## ۹. منابع و مراجع (References)

برای مطالعه بیشتر و اطمینان از صحت مطالب، می‌توانید به منابع رسمی زیر مراجعه کنید:

1. **Microsoft Learn - `with` expression (C# Reference):**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression)
   *(منبع اصلی برای بررسی Syntax و مثال‌های پایه)*

2. **Microsoft Learn - Records (C# Reference):**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
   *(برای درک عمیق‌تر تفاوت Record Class و Record Struct)*

3. **C# Language Specification - Records (GitHub / Microsoft):**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/records)
   *(مستندات رسمی Spec زبان C# که رفتار دقیق Copy Constructor و Shallow Copy در آن توضیح داده شده است)*

4. **C# Language Specification - `with` expression:**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128114-the-with-expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128114-the-with-expression)
   *(توضیحات سطح پایین Spec درباره نحوه ارزیابی عبارت with توسط کامپایلر)*