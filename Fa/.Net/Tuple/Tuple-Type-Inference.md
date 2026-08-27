# آموزش جامع Tuple و Type Inference در C#

به مستندات آموزشی Repository ما خوش آمدید! در این مقاله، به یکی از جذاب‌ترین و کاربردی‌ترین ویژگی‌های مدرن C#، یعنی **Tupleها** و مفهوم بسیار مهم **Type Inference (استنتاج نوع)** می‌پردازیم. اگر می‌خواهید کدهای تمیزتر، کوتاه‌تر و خواناتری بنویسید، این مقاله برای شماست.

---

## 📑 فهرست مطالب
1. [Type Inference چیست؟](#type-inference-چیست)
2. [نقش var در Tuple](#نقش-var-در-tuple)
3. [چگونه کامپایلر Type اعضای Tuple را تشخیص می‌دهد؟](#چگونه-کامپایلر-type-اعضای-tuple-را-تشخیص-میدهد)
4. [Type Inference برای اعضای مختلف](#type-inference-برای-اعضای-مختلف)
5. [Tupleهای نام‌گذاری‌شده (Named Tuples)](#tupleهای-نامگذاریشده)
6. [Name Inference (استنتاج نام)](#name-inference-استنتاج-نام)
7. [تفاوت Type Inference و Name Inference](#تفاوت-type-inference-و-name-inference)
8. [مثال‌های ساده و پیچیده‌تر](#مثالهای-ساده-و-پیچیدهتر)
9. [مواردی که کامپایلر نمی‌تواند Type را استنتاج کند](#مواردی-که-کامپایلر-نمیتواند-type-را-استنتاج-کند)
10. [اشتباهات رایج](#اشتباهات-رایج)
11. [جمع‌بندی](#جمع بندی)
12. [منابع معتبر](#منابع-معتبر)

---

<h2 id="type-inference-چیست">۱. Type Inference چیست؟</h2>

قبل از ورود به مبحث Tuple، باید بدانیم **Type Inference (استنتاج نوع)** چیست. در زبان‌هایی مثل C#، شما معمولاً باید نوع متغیر را صریحاً اعلام کنید:
```csharp
int myNumber = 10;
string myName = "Ali";
```
اما از نسخه‌های قدیمی‌تر C#، کلمه کلیدی `var` معرفی شد. وقتی از `var` استفاده می‌کنید، به کامپایلر می‌گویید: *"خودت از روی مقدار سمت راست، نوع متغیر را حدس بزن (استنتاج کن)."*

```csharp
var myNumber = 10; // کامپایلر می‌فهمد که این یک int است
var myName = "Ali"; // کامپایلر می‌فهمد که این یک string است
```

> 💡 **نکته مهم:** `var` به معنای Dynamic یا `object` نیست! نوع متغیر در **زمان کامپایل (Compile-time)** تعیین می‌شود و کاملاً Static و Type-Safe است. یعنی اگر بعداً سعی کنید یک `string` را به `myNumber` اختصاص دهید، کامپایلر خطا می‌دهد.

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="نقش-var-در-tuple">۲. نقش var در Tuple</h2>

در C# 7.0، **Tupleها** معرفی شدند تا بتوانیم چندین مقدار را در یک بسته واحد برگردانیم یا ذخیره کنیم. استفاده از `var` در کنار Tupleها بسیار رایج و کاربردی است.

```csharp
var myTuple = (1, "Hello", true);
```

در اینجا، ما نیازی نداریم بنویسیم `(int, string, bool) myTuple = ...`. کلمه `var` به کامپایلر اجازه می‌دهد تا ساختار کامل Tuple را استنتاج کند.

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="چگونه-کامپایلر-type-اعضای-tuple-را-تشخیص-میدهد">۳. چگونه کامپایلر Type اعضای Tuple را تشخیص می‌دهد؟</h2>

کامپایلر C# با بررسی **مقدار اختصاص‌داده‌شده (Assigned Value)** در سمت راست تساوی `=`، نوع هر عضو را به صورت مستقل تشخیص می‌دهد.

```csharp
var data = (42, 3.14, "C#");
```

**توضیح خط‌به‌خط:**
* `var data`: کامپایلر می‌داند که `data` یک Tuple است، اما هنوز نوع دقیق آن را نمی‌داند.
* `42`: چون بدون اعشار است، کامپایلر آن را `int` در نظر می‌گیرد.
* `3.14`: چون دارای اعشار است، کامپایلر آن را `double` تشخیص می‌دهد.
* `"C#"`: چون داخل کوتیشن است، کامپایلر آن را `string` می‌شناسد.
* **نتیجه نهایی:** کامپایلر نوع `data` را به صورت `(int, double, string)` در نظر می‌گیرد.

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="type-inference-برای-اعضای-مختلف">۴. Type Inference برای اعضای مختلف</h2>

شما می‌توانید هر ترکیبی از انواع داده (Primitive، Reference، Struct و حتی Nullable) را در یک Tuple داشته باشید و کامپایلر همه را به درستی استنتاج می‌کند:

```csharp
DateTime now = DateTime.Now;
var complexTuple = (100, "Text", true, now, (int?)null);
```
**ساختار استنتاج‌شده توسط کامپایلر:**
`(int, string, bool, DateTime, int?)`

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="tupleهای-نامگذاریشده">۵. Tupleهای نام‌گذاری‌شده (Named Tuples)</h2>

دسترسی به اعضای Tuple با نام‌های پیش‌فرض `Item1`، `Item2` و... بسیار سخت و غیرخوانا است. برای حل این مشکل، می‌توانیم برای اعضای Tuple **نام (Label)** تعیین کنیم:

```csharp
var user = (Id: 1, Username: "Ali_Dev", IsActive: true);

Console.WriteLine(user.Id);       // خروجی: 1
Console.WriteLine(user.Username); // خروجی: Ali_Dev
```

> 💡 **نکته مهم:** نام‌گذاری اعضا فقط در **زمان کامپایل** برای راحتی برنامه‌نویس است. در زمان اجرا (Runtime)، این نام‌ها از بین می‌روند و فقط `Item1`، `Item2` و... باقی می‌مانند (مگر اینکه از Reflection خاصی استفاده کنید).

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="name-inference-استنتاج-نام">۶. Name Inference (استنتاج نام)</h2>

از **C# 7.1** به بعد، ویژگی فوق‌العاده‌ای به نام **Name Inference** اضافه شد. اگر نام متغیرها یا Propertyهایی که داخل Tuple قرار می‌دهید را صریحاً مشخص نکنید، کامپایلر به صورت خودکار **نام متغیر** را به عنوان **نام عضو Tuple** استنتاج می‌کند!

```csharp
int age = 28;
string name = "Sara";

// کامپایلر به صورت خودکار نام اعضا را age و name قرار می‌دهد
var person = (age, name); 

Console.WriteLine(person.age);  // خروجی: 28
Console.WriteLine(person.name); // خروجی: Sara
```

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="تفاوت-type-inference-و-name-inference">۷. تفاوت Type Inference و Name Inference</h2>

بسیاری از مبتدیان این دو مفهوم را با هم اشتباه می‌گیرند. بیایید تفاوت آن‌ها را در یک جدول و مثال بررسی کنیم:

| ویژگی | Type Inference (استنتاج نوع) | Name Inference (استنتاج نام) |
| :--- | :--- | :--- |
| **مربوط به چیست؟** | تشخیص **جنس داده** (int, string, ...) | تشخیص **برچسب/نام** (Id, Name, ...) |
| **از کجا می‌فهمد؟** | از روی **مقدار** سمت راست (`10` -> int) | از روی **نام متغیر** سمت راست (`age` -> age) |
| **نسخه C#** | از C# 1.0 (با var) و C# 7.0 (Tuples) | از C# 7.1 به بعد |

**مثال مقایسه‌ای:**
```csharp
int userId = 5;

// Type Inference: کامپایلر می‌فهمد این یک int است
// Name Inference: کامپایلر می‌فهمد نام این عضو userId است
var tuple1 = (userId); 

// اگر نام را دستی عوض کنیم، Name Inference رخ نمی‌دهد اما Type Inference همچنان هست
var tuple2 = (Id: userId); 
```

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="مثالهای-ساده-و-پیچیدهتر">۸. مثال‌های ساده و پیچیده‌تر</h2>

### مثال ساده: بازگرداندن چند مقدار از یک متد
فرض کنید می‌خواهیم کوچک‌ترین و بزرگ‌ترین عدد یک آرایه را پیدا کنیم:

```csharp
(int Min, int Max) GetMinMax(int[] numbers)
{
    return (numbers.Min(), numbers.Max());
}

// استفاده:
var result = GetMinMax(new[] { 5, 2, 9, 1 });
Console.WriteLine($"Min: {result.Min}, Max: {result.Max}");
```

### مثال پیچیده‌تر: استفاده در LINQ
Tupleها در LINQ برای پروژه کردن (Project) چند فیلد بدون نیاز به ساخت کلاس جدید، معجزه می‌کنند:

```csharp
var users = new List<User> { /* ... */ };

var userSummaries = users
    .Where(u => u.IsActive)
    .Select(u => (u.Id, u.FullName, u.Email)) // Name Inference در اینجا رخ می‌دهد
    .ToList();

foreach (var summary in userSummaries)
{
    // دسترسی راحت به gracias Name Inference
    Console.WriteLine($"{summary.FullName} ({summary.Email})"); 
}
```

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="مواردی-که-کامپایلر-نمیتواند-type-را-استنتاج-کند">۹. مواردی که کامپایلر نمی‌تواند Type مناسب را استنتاج کند</h2>

گاهی اوقات کامپایلر در استنتاج نوع (چه Type و چه Name) شکست می‌خورد و خطا (Error) می‌دهد:

**۱. استفاده از `null` بدون Cast:**
کامپایلر نمی‌تواند حدس بزند `null` چه نوعی است (چون `null` می‌تواند هر Reference Type یا Nullable باشد).
```csharp
// ❌ خطای کامپایلر: The tuple literal does not have a best type
var errorTuple = (null, "Hello"); 

// ✅ راه حل: باید نوع را صریح یا Cast کنید
var fixedTuple = ((string?)null, "Hello");
```

**۲. عدم مقداردهی اولیه به `var`:**
```csharp
// ❌ خطا: Implicitly-typed variables must be initialized
var myTuple; 
myTuple = (1, 2);
```

**۳. تناقض در نوع اعضا هنگام انتساب:**
```csharp
var t1 = (1, "A");
// ❌ خطا: نمی‌توانید Tuple از نوع (string, string) را به (int, string) نسبت دهید
t1 = ("B", "C"); 
```

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="اشتباهات-رایج">۱۰. اشتباهات رایج</h2>

1. **استفاده بیش از حد به جای کلاس/Record:**
   Tupleها برای انتقال داده‌های موقت (مثلاً خروجی یک متد) عالی هستند. اما برای مدل‌های دامنه (Domain Models) یا دیتابیس، **هرگز** از Tuple استفاده نکنید. به جای آن از `class` یا `record` استفاده کنید.
2. **توهم Reference Equality:**
   Tupleها در C# **Struct (نوع مقداری)** هستند. یعنی اگر دو Tuple مقادیر یکسانی داشته باشند، با هم برابرند (`==`).
   ```csharp
   var t1 = (1, 2);
   var t2 = (1, 2);
   Console.WriteLine(t1 == t2); // خروجی: True (برخلاف کلاس‌ها)
   ```
3. **فراموش کردن نام‌ها در Serialization:**
   اگر Tuple را به JSON تبدیل کنید (مثلاً با `System.Text.Json`)، نام‌های اختصاصی شما (مثل `Id` یا `Name`) حفظ **نمی‌شوند** و به صورت `Item1` و `Item2` ذخیره می‌شوند.

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="جمع-بندی">۱۱. جمع‌بندی</h2>

در این مقاله یاد گرفتیم که:
* **Type Inference** با استفاده از `var` به ما کمک می‌کند کدهای کوتاه‌تر و تمیزتری بنویسیم، بدون اینکه از مزایای Static Typing محروم شویم.
* **Tupleها** بسته‌های lightweight و مقداری (Value-type) برای گروه‌بندی داده‌ها هستند.
* کامپایلر C# به صورت هوشمندانه‌ای هم **نوع (Type)** و هم **نام (Name)** اعضای Tuple را از روی مقادیر و متغیرهای سمت راست استنتاج می‌کند.
* باید مراقب باشیم که Tupleها را به جای کلاس‌ها و Recordها برای مدل‌های پیچیده استفاده نکنیم و در استفاده از `null` در آن‌ها دقت کنیم.

استفاده درست از Tuple و Type Inference می‌تواند کدهای شما را از حالت "پر از کلاس‌های یک‌بار مصرف" (One-off classes) خارج کرده و به کدهایی خوانا و مدرن تبدیل کند.

[بازگشت به فهرست مطالب](#فهرست-مطالب)

---

<h2 id="منابع-معتبر">۱۲. منابع معتبر</h2>

برای مطالعه بیشتر و عمیق‌تر، منابع زیر پیشنهاد می‌شوند:
1. [Microsoft Docs - Tuple Types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
2. [Microsoft Docs - var (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations#implicitly-typed-local-variables)
3. [C# 7.1 and .NET Core 2.0 - Modern Cross-Platform Application Development (Book)](https://www.packtpub.com/product/c-71-and-net-core-20-modern-cross-platform-application-development)
4. [Nick Chapsas - YouTube: C# Tuples Explained](https://www.youtube.com/results?search_query=c%23+tuples+explained)

---
*تاریخ آخرین به‌روزرسانی: مرداد ۱۴۰۵ (August 2026) | نویسنده: تیم آموزشی Repository*