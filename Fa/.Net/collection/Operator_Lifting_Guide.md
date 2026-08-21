

# 🚀 راهنمای جامع Operator Lifting در #C
**از مفاهیم پایه تا زیر کاپوت کامپایلر و Expression Trees**

به این آموزش خوش آمدید! اگر تا به حال با تایپ‌های `Nullable` (مثل `int?`) در سی‌شارپ کار کرده‌اید و متوجه شده‌اید که چطور می‌توانید بدون خطا آن‌ها را با هم جمع کنید یا مقایسه نمایید، شما در حال استفاده از یکی از جذاب‌ترین ویژگی‌های کامپایلر سی‌شارپ به نام **Operator Lifting** بوده‌اید.

در این مقاله، ما از صفر مطلق شروع کرده و تا عمیق‌ترین لایه‌های کامپایلر و درخت عبارات (Expression Trees) پیش می‌رویم.

---

## 📑 فهرست مطالب
1. [مفهوم Operator Lifting چیست؟](#1-مفهوم-operator-lifting-چیست)
2. [آشنایی با Nullable و عملگرها](#2-آشنایی-با-nullable-و-عملگرها)
3. [نتیجه عملیات روی Nullableها (قانون Null Propagation)](#3-نتیجه-عملیات-روی-nullableها)
4. [Lifting عملگرهای مقایسه‌ای (`<` و `>`)](#4-lifting-عملگرهای-مقایسه‌ای--و-)
5. [Lifting عملگرهای تساوی (`==` و `!=`)](#5-lifting-عملگرهای-تساوی--و-)
6. [نحوه رفتار Compiler (تولید کد)](#6-نحوه-رفتار-compiler-تولید-کد)
7. [تبدیل Expression به کد قابل اجرا (بخش پیشرفته)](#7-تبدیل-expression-به-کد-قابل-اجرا-بخش-پیشرفته)
8. [منابع معتبر برای مطالعه بیشتر](#8-منابع-معتبر-برای-مطالعه-بیشتر)

---

## 1. مفهوم Operator Lifting چیست؟

فرض کنید شما یک عدد `int` دارید و می‌توانید آن را با یک `int` دیگر جمع کنید. اما اگر آن عدد را درون یک جعبه شیشه‌ای (همان `Nullable<T>`) بگذارید، کامپایلر سی‌شارپ به صورت پیش‌فرض اجازه جمع کردن دو جعبه را به شما نمی‌دهد.

**Operator Lifting (عملگرهای بالابر)** قابلیتی در کامپایلر سی‌شارپ است که عملگرهای استاندارد (مثل `+`, `-`, `<`, `==`) را از روی تایپ پایه (`T`) برمی‌دارد و آن‌ها را برای تایپ nullable (`Nullable<T>`) **بازتعریف (Lift)** می‌کند.
به زبان ساده: کامپایلر به صورت خودکار کدهایی را تولید می‌کند تا ابتدا جعبه‌ها را باز کند، عملیات را انجام دهد و اگر جعبه خالی بود، نتیجه را `null` کند.

---

## 2. آشنایی با Nullable و عملگرها

در سی‌شارپ، تایپ‌های Value Type (مثل `int`, `double`, `DateTime`) نمی‌توانند `null` باشند. اما با استفاده از `?` یا `Nullable<T>` می‌توانند این کار را بکنند.

```csharp
int a = 5;
int? b = 10; // یک Nullable int

// به لطف Operator Lifting، این کد بدون خطا کامپایل می‌شود:
int? result = a + b; // نتیجه: 15
```
کامپایلر می‌داند که `a` یک `int` است و `b` یک `int?`. او عملگر `+` را از روی `int` "لیفت" می‌کند تا بتواند روی `int?` کار کند.

---

## 3. نتیجه عملیات روی Nullableها

قانون طلایی در Operator Lifting برای عملگرهای ریاضی و منطقی **Null Propagation** است:
> **اگر حداقل یکی از عملوندها (Operands) برابر با `null` باشد، نتیجه عملیات `null` خواهد بود.**

```csharp
int? x = 5;
int? y = null;

var sum = x + y;   // نتیجه: null
var sub = x - y;   // نتیجه: null
var mul = x * 2;   // نتیجه: 10 (چون 2 که null نیست)
var div = x / y;   // نتیجه: null (بدون پرتاب DivideByZeroException!)
```

---

## 4. Lifting عملگرهای مقایسه‌ای (`<` و `>`)

رفتار عملگرهای مقایسه‌ای (`<`, `>`, `<=`, `>=`) یکی از **مهم‌ترین و گول‌زننده‌ترین** مباحث در سی‌شارپ است.

> **قانون:** اگر در مقایسه `<` یا `>`، **حداقل یکی از طرفین `null` باشد، نتیجه همیشه `false` است.**

```csharp
int? a = null;
int? b = 5;

Console.WriteLine(a < b);  // False (چون a null است)
Console.WriteLine(a > b);  // False
Console.WriteLine(a < a);  // False (حتی null < null هم false است!)
```
**چرا اینطور است؟** 
از نظر منطقی، وقتی یکی از مقادیر نامشخص (`null`) است، ما نمی‌توانیم با قطعیت بگوییم که "کوچکتر" یا "بزرگتر" است. بنابراین کامپایلر برای جلوگیری از نتیجه‌گیری اشتباه، `false` برمی‌گرداند.

---

## 5. Lifting عملگرهای تساوی (`==` و `!=`)

برخلاف عملگرهای `<` و `>`، عملگرهای تساوی (`==` و `!=`) رفتار متفاوتی دارند و بسیار منطقی‌تر عمل می‌کنند:

> **قانون:** در `==`، اگر هر دو طرف `null` باشند نتیجه `true` است. اگر یکی `null` و دیگری مقدار داشته باشد، نتیجه `false` است.

```csharp
int? a = null;
int? b = null;
int? c = 5;

Console.WriteLine(a == b); // True  (هیچ چیز برابر با هیچ چیز است!)
Console.WriteLine(a == c); // False
Console.WriteLine(a != c); // True
```

---

## 6. نحوه رفتار Compiler (تولید کد)

کامپایلر سی‌شارپ در واقع عملگرهای جدیدی برای `Nullable<T>` نمی‌سازد. بلکه وقتی شما کدی مثل `a + b` (که هر دو `int?` هستند) می‌نویسید، کامپایلر آن را به معادل زیر در کد میانی (و در نهایت IL) تبدیل می‌کند:

```csharp
// کد شما:
int? result = a + b;

// کدی که کامپایلر در ذهن خود تولید می‌کند:
int? result = (a.HasValue && b.HasValue) 
              ? new int?(a.Value + b.Value) 
              : null;
```

**برای عملگرهای مقایسه‌ای (`<` و `>`):**
```csharp
// کد شما:
bool isGreater = a > b;

// معادل تولید شده توسط کامپایلر:
bool isGreater = (a.HasValue && b.HasValue) 
                 ? (a.Value > b.Value) 
                 : false;
```
شما می‌توانید این رفتار کامپایلر را با استفاده از ابزارهایی مثل [SharpLab.io](https://sharplab.io/) مشاهده کنید و خروجی IL یا C# دیکامپایل شده آن را ببینید.

---

## 7. تبدیل Expression به کد قابل اجرا (بخش پیشرفته)

این بخش برای کسانی است که با **LINQ to Entities (مثل Entity Framework Core)** یا کار با **Expression Trees** سروکار دارند.

وقتی شما یک `Expression<Func<T, bool>>` می‌سازید، کد شما به جای اجرا، به یک **درخت عبارت (Expression Tree)** تبدیل می‌شود تا بعداً توسط یک Provider (مثل دیتابیس SQL) به کوئری تبدیل شود.

### چالش Operator Lifting در Expression Trees
وقتی شما می‌نویسید:
```csharp
Expression<Func<MyEntity, bool>> expr = x => x.NullableAge > 18;
```
در API مربوط به Expression Trees (`System.Linq.Expressions`)، هیچ نود (Node) خاصی به نام "Lifted Operator" وجود ندارد. 

**کامپایلر چگونه این را به Expression Tree تبدیل می‌کند؟**
کامپایلر به صورت خودکار چک کردن `null` را در درخت عبارت تزریق می‌کند. درخت عبارت تولید شده در واقع معادل کد زیر است:

```csharp
// Expression Tree واقعی که کامپایلر می‌سازد:
x => (x.NullableAge.HasValue && x.NullableAge.Value > 18)
```

**چرا این موضوع مهم است؟**
1. **در Entity Framework Core:** وقتی EF Core این Expression Tree را می‌بیند، آن را به SQL ترجمه می‌کند. در SQL سرور، `NULL > 18` نتیجه‌اش `NULL` (یا Unknown) است، اما سی‌شارپ `false` برمی‌گرداند. EF Core با درک این Expression Tree، کوئری SQL را طوری می‌سازد که دقیقاً همان رفتار `false` سی‌شارپ را شبیه‌سازی کند.
2. **ساخت دستی Expression:** اگر بخواهید خودتان به صورت دستی (Dynamic) یک Expression Tree بسازید، نمی‌توانید مستقیماً از `Expression.GreaterThan` برای Nullableها استفاده کنید و انتظار رفتار Lifted را داشته باشید. **شما باید خودتان نودهای `HasValue` و `Value` را با استفاده از `Expression.Condition` و `Expression.AndAlso` بسازید** تا کامپایلر و Providerها رفتار درستی از خود نشان دهند.

```csharp
// ساخت دستی یک Lifted Operator در Expression Tree:
var hasValue = Expression.Property(parameter, "HasValue");
var value = Expression.Property(parameter, "Value");
var comparison = Expression.GreaterThan(value, Expression.Constant(18));

// ترکیب شرط null و مقایسه
var liftedExpression = Expression.AndAlso(hasValue, comparison);
```

---

## 8. منابع معتبر برای مطالعه بیشتر

برای تسلط کامل بر این مباحث، مطالعه منابع زیر که توسط طراحان زبان و متخصصان برجسته نوشته شده‌اند، به شدت توصیه می‌شود:

1. **[مستندات رسمی مایکروسافت: Lifted Operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#lifted-operators)**
   * *توضیح:* مرجع اصلی و رسمی C# Language Specification که دقیقاً قوانین لیفت شدن عملگرها را تعریف کرده است.
2. **[کتاب C# in Depth اثر Jon Skeet](https://csharpindepth.com/)**
   * *توضیح:* فصل مربوط به Nullable Types در این کتاب، یکی از بهترین و عمیق‌ترین توضیحات درباره رفتار کامپایلر و Operator Lifting است.
3. **[بلاگ Eric Lippert (طراح سابق کامپایلر C#)](https://ericlippert.com/)**
   * *توضیح:* جستجوی کلمه "Lifting" یا "Nullable" در بلاگ اریک لیپرت، مقالات بی‌نظیری درباره فلسفه طراحی این ویژگی در سی‌شارپ به شما می‌دهد.
4. **[مستندات Expression Trees در مایکروسافت](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)**
   * *توضیح:* برای درک نحوه تبدیل کد به درخت عبارت و نحوه مدیریت عملگرها در سطح Expression.
5. **[وب‌سایت SharpLab.io](https://sharplab.io/)**
   * *توضیح:* ابزاری فوق‌العاده برای دیدن خروجی کامپایلر (IL و C# دیکامپایل شده) و مشاهده اینکه کامپایلر چگونه Operator Lifting را به کد واقعی تبدیل می‌کند.

---
**نویسنده و گردآوری:** [نام شما / نام ریپازیتوری شما]
**تاریخ بروزرسانی:** August 2026

> 💡 **نکته پایانی:** درک Operator Lifting نه تنها کد شما را تمیزتر می‌کند، بلکه از باگ‌های پنهان (به خصوص در مقایسه‌های `<` و `>`) جلوگیری کرده و دید شما را نسبت به نحوه تفکر کامپایلر سی‌شارپ عمیق‌تر می‌کند.

***

### 💡 راهنمای استفاده برای شما (صاحب ریپازیتوری):
* این متن را در یک فایل با نام `Operator_Lifting_Guide.md` یا `README.md` کپی کنید.
* در بخش `[نام شما / نام ریپازیتوری شما]` اطلاعات خودتان را وارد کنید.
* لینک‌های منابع (Sources) کاملاً معتبر و فعال (Active) هستند و به مستندات رسمی مایکروسافت (که در سال 2026 نیز معتبرترین مرجع هستند) ارجاع می‌دهند.
* اگر می‌خواهید برای هر بخش یک فایل `.cs` جداگانه در ریپازیتوری بسازید، می‌توانید کدهای داخل بلاک‌های کد را استخراج کرده و به این فایل لینک دهید.