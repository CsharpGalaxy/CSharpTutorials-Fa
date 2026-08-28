# آموزش جامع Tuple و Deconstruction در #C (از مبتدی تا پیشرفته)

به مقاله آموزشی **Tuple و Deconstruction** خوش آمدید. این مقاله برای مخزن آموزشی شما طراحی شده است تا توسعه‌دهندگان را از مفاهیم پایه‌ای به سمت تکنیک‌های پیشرفته و کاربردی هدایت کند.

---

## 📑 فهرست مطالب
1. [مقدمه](#مقدمه)
2. [Deconstruction چیست؟](#deconstruction-چیست)
3. [چرا Deconstruction به وجود آمده است؟](#چرا-deconstruction-به-وجود-آمده-است)
4. [سینتکس و قواعد پایه](#سینتکس-و-قواعد-پایه)
5. [Deconstruct کردن Tuple](#deconstruct-کردن-tuple)
6. [نقش `var` در Deconstruction](#نقش-var-در-deconstruction)
7. [استفاده از Type مشخص و Assignment](#استفاده-از-type-مشخص-و-assignment)
8. [Deconstruction هنگام دریافت Return Value](#deconstruction-هنگام-دریافت-return-value)
9. [تفاوت Deconstruction با ساخت Tuple (بسیار مهم)](#تفاوت-deconstruction-با-ساخت-tuple)
10. [مثال‌های ساده](#مثالهای-ساده)
11. [مثال‌های واقعی و کاربردی](#مثالهای-واقعی-و-کاربردی)
12. [مبحث پیشرفته: Discards و Custom Deconstruction](#مبحث-پیشرفته-discards-و-custom-deconstruction)
13. [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
14. [جمع‌بندی](#جمع‌بندی)
15. [منابع معتبر](#منابع-معتبر)

---

<a name="مقدمه"></a>
## مقدمه
یکی از چالش‌های همیشگی برنامه‌نویسان، بازگرداندن یا استخراج چندین مقدار مرتبط از یک منبع (مثل یک متد یا یک شیء) بوده است. در نسخه‌های قدیمی‌تر #C، مجبور بودیم از کلاس‌های واسط، آرایه‌ها یا پارامترهای `out` استفاده کنیم. اما از **#C 7.0** به بعد، مایکروسافت با معرفی **Tuples** و قابلیت جادویی **Deconstruction**، این مشکل را به زیباترین شکل ممکن حل کرد.

---

<a name="deconstruction-چیست"></a>
## Deconstruction چیست؟
به زبان ساده، **Deconstruction (تخریب یا باز کردن)** یعنی unpacking (باز کردن بسته‌بندی) یک شیء واحد و استخراج مقادیر درون آن به متغیرهای جداگانه.
تصور کنید یک چمدان (Tuple) دارید که داخل آن یک کفش و یک پیراهن است. Deconstruction یعنی در چمدان را باز کنید و کفش را در کمد کفش‌ها و پیراهن را در کمد لباس‌ها بگذارید.

---

<a name="چرا-deconstruction-به-وجود-آمده-است"></a>
## چرا Deconstruction به وجود آمده است؟
قبل از #C 7، برای دریافت دو مقدار از یک متد، دو راه اصلی داشتیم:
1. **استفاده از `out` parameters:** کد را شلوغ و خوانایی را کم می‌کرد.
2. **ساخت یک کلاس/ساختار اختصاصی:** برای یک کار ساده، ایجاد یک کلاس جدید Overkill (زیاده‌روی) بود.

حتی پس از معرفی Tupleها، دسترسی به مقادیر از طریق `Item1` و `Item2` بسیار زشت و غیرقابل نگهداری بود:
```csharp
// روش قدیمی و زشت
var result = GetUser();
Console.WriteLine(result.Item1); // نام کاربر
Console.WriteLine(result.Item2); // سن کاربر
```
**Deconstruction** به وجود آمد تا به جای `Item1` و `Item2`، مستقیماً مقادیر را در متغیرهای خوش‌نام و معنادار ذخیره کنیم.

---

<a name="سینتکس-و-قواعد-پایه"></a>
## سینتکس و قواعد پایه
سینتکس Deconstruction بر پایه پرانتز `()` و کاما `,` استوار است.
```csharp
(متغیر۱, متغیر۲) = شیء-منبع;
```

---

<a name="deconstruct-کردن-tuple"></a>
## Deconstruct کردن Tuple
رایج‌ترین کاربرد Deconstruction برای `ValueTuple`ها است. شما می‌توانید یک Tuple را در لحظه ایجاد کرده و همان لحظه آن را باز کنید:

```csharp
// ایجاد Tuple و Deconstruction همزمان
var (name, age) = ("Sara", 28);

Console.WriteLine(name); // خروجی: Sara
Console.WriteLine(age);  // خروجی: 28
```

---

<a name="نقش-var-در-deconstruction"></a>
## نقش `var` در Deconstruction
وقتی از `var` در سمت چپ تساوی استفاده می‌کنید، کامپایلر #C به صورت خودکار **Type** تمام متغیرهای داخل پرانتز را بر اساس مقادیر سمت راست حدس می‌زند (Type Inference).

**قانون طلایی:** کلمه `var` فقط **یک‌بار** و **خارج از پرانتز** نوشته می‌شود.

```csharp
// صحیح
var (x, y, z) = (10, 20.5, "Hello"); 
// x -> int, y -> double, z -> string

// غلط (کامپایلر ارور می‌دهد)
(var x, var y) = (10, 20); 
```

---

<a name="استفاده-از-type-مشخص-و-assignment"></a>
## استفاده از Type مشخص و Assignment
اگر متغیرها از قبل تعریف شده باشند، دیگر نیازی به `var` نیست. در این حالت شما در حال **Assignment (تخصیص)** مقادیر به متغیرهای موجود هستید.

```csharp
string userName;
int userAge;

// متغیرهای از قبل تعریف شده
(userName, userAge) = GetUserData(); 

// یا تخصیص مستقیم:
string city;
int population;
(city, population) = ("Tehran", 9000000);
```

---

<a name="deconstruction-هنگام-دریافت-return-value"></a>
## Deconstruction هنگام دریافت Return Value
وقتی متدی یک Tuple برمی‌گرداند، می‌توانید خروجی آن را مستقیماً Deconstruct کنید.

```csharp
public static (string, int) GetProductInfo()
{
    return ("Laptop", 1200);
}

// استفاده در کد اصلی:
var (productName, productPrice) = GetProductInfo();
```

---

<a name="تفاوت-deconstruction-با-ساخت-tuple"></a>
## تفاوت Deconstruction با ساخت Tuple (بسیار مهم)
بسیاری از مبتدیان سینتکس پرانتزها را با هم اشتباه می‌گیرند. بیایید این تفاوت را شفاف کنیم:

**۱. ساخت Tuple (Tuple Creation):** سمت چپ **تعریف متغیر** است، سمت راست **ایجاد داده**.
```csharp
(int x, int y) point = (10, 20); 
// point یک متغیر از نوع Tuple است.
```

**۲. باز کردن Tuple (Deconstruction):** سمت چپ **لیست متغیرهای جداگانه** است، سمت راست **منبع داده**.
```csharp
(int x, int y) = point; 
// x و y دو متغیر جداگانه int هستند. متغیری به نام point در سمت چپ تعریف نشده.
```
> 💡 **نکته کلیدی:** در Deconstruction، سمت چپ تساوی **هرگز** نام یک متغیر واحد (مثل `point`) نیست، بلکه لیستی از متغیرهای تجزیه شده است.

---

<a name="مثالهای-ساده"></a>
## مثال‌های ساده

### مثال ۱: محاسبه حداقل و حداکثر
```csharp
(int min, int max) = GetMinMax(5, 2, 9, 1);
Console.WriteLine($"Min: {min}, Max: {max}");

static (int, int) GetMinMax(params int[] numbers)
{
    return (numbers.Min(), numbers.Max());
}
```

### مثال ۲: مختصات دوبعدی
```csharp
var (latitude, longitude) = (35.6892, 51.3890);
```

---

<a name="مثالهای-واقعی-و-کاربردی"></a>
## مثال‌های واقعی و کاربردی

### ۱. حلقه روی Dictionary (بسیار پرکاربرد)
به جای استفاده از `KeyValuePair` و دسترسی به `Key` و `Value`، می‌توانید دیکشنری را Deconstruct کنید:

```csharp
var scores = new Dictionary<string, int> 
{ 
    { "Ali", 18 }, 
    { "Reza", 16 } 
};

foreach (var (student, score) in scores)
{
    Console.WriteLine($"{student} got {score}");
}
```

### ۲. Parsing و Split کردن رشته‌ها
```csharp
public static (string firstName, string lastName) SplitFullName(string fullName)
{
    var parts = fullName.Split(' ');
    return (parts[0], parts[1]);
}

// استفاده:
var (fName, lName) = SplitFullName("Kourosh Ghalambir");
```

### ۳. کار با LINQ
```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };
// استفاده از Deconstruction در Select برای تولید Tuple
var result = numbers.Select(n => (Number: n, Square: n * n));

foreach (var (num, sq) in result)
{
    Console.WriteLine($"{num}^2 = {sq}");
}
```

---

<a name="مبحث-پیشرفته-discards-و-custom-deconstruction"></a>
## مبحث پیشرفته: Discards و Custom Deconstruction

### استفاده از Discard (`_`)
اگر فقط به بخشهایی از یک Tuple نیاز دارید، می‌توانید بقیه را با `_` (Discard) دور بریزید تا کد تمیزتر شود و کامپایلر هم منابع اضافی تخصیص ندهد:

```csharp
var (id, _, email) = GetUserDetails(); // نام کاربر (مقدار دوم) نیاز نیست
```

### Custom Deconstruction برای کلاس‌های شخصی
شما می‌توانید قابلیت Deconstruction را به **کلاس‌های خودتان** نیز اضافه کنید! کافیست متدی به نام `Deconstruct` با پارامترهای `out` در کلاس بنویسید:

```csharp
public class Rectangle
{
    public double Width { get; set; }
    public double Height { get; set; }

    // متد جادویی Deconstruct
    public void Deconstruct(out double width, out double height)
    {
        width = Width;
        height = Height;
    }
}

// استفاده:
var myRect = new Rectangle { Width = 10, Height = 20 };
var (w, h) = myRect; // کلاس شما دقیقاً مثل یک Tuple باز شد!
```

---

<a name="نکات-مهم-و-اشتباهات-رایج"></a>
## نکات مهم و اشتباهات رایج

❌ **اشتباه ۱: ترکیب `var` و Type مشخص**
```csharp
// غلط: نمی‌توانید var را با نوع صریح ترکیب کنید
var (int x, y) = (10, 20); // ❌ Compilation Error

// صحیح: یا همه var، یا همه Type مشخص (بدون var)
var (x, y) = (10, 20);     // ✅
(int x, int y) = (10, 20); // ✅
```

❌ **اشتباه ۲: استفاده از `var` برای متغیرهای از قبل تعریف شده**
```csharp
string name;
int age;
// غلط: var متغیر جدید می‌سازد، نه اینکه به متغیر قبلی مقدار بدهد
var (name, age) = ("Ali", 30); // ❌ ارور: متغیر تکراری

// صحیح:
(name, age) = ("Ali", 30);     // ✅
```

❌ **اشتباه ۳: عدم تطابق تعداد متغیرها**
تعداد متغیرهای سمت چپ باید دقیقاً با تعداد پارامترهای `Deconstruct` (یا آیتم‌های Tuple) سمت راست برابر باشد (مگر اینکه از Overloadهای مختلف Deconstruct استفاده کرده باشید).

---

<a name="جمع‌بندی"></a>
## جمع‌بندی
*   **Tuple** راهی برای گروه‌بندی موقت چندین مقدار در یک شیء واحد است.
*   **Deconstruction** تکنیکی برای باز کردن این بسته‌بندی و ریختن مقادیر در متغیرهای جداگانه است.
*   این قابلیت باعث افزایش **خوانایی کد (Readability)**، کاهش **Boilerplate** و جلوگیری از ایجاد کلاس‌های اضافی می‌شود.
*   از `var` برای ساخت متغیرهای جدید و از سینتکس بدون `var` برای تخصیص به متغیرهای موجود استفاده کنید.
*   با `Deconstruct` method می‌توانید این قابلیت را به کلاس‌های شخصی خود نیز تزریق کنید.

---

<a name="منابع-معتبر"></a>
## منابع معتبر برای مطالعه بیشتر
برای عمیق‌تر شدن در این مباحث، منابع رسمی زیر پیشنهاد می‌شوند:
1. [Tuples - Microsoft Learn (مستندات رسمی #C)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
2. [Deconstruction - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct)
3. [Discards in C# - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/discards)

---
*نویسنده: [نام شما / نام تیم آموزشی]*
*تاریخ بروزرسانی: August 27, 2026*
*لینک به مقاله بعدی: [آشنایی با Pattern Matching در #C](#)*