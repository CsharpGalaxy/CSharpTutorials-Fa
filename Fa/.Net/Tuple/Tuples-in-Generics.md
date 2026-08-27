در ادامه، یک مقاله آموزشی جامع، ساختاریافته و کاملاً منطبق با استانداردهای یک Repository آموزشی (مانند گیت‌هاب) برای شما تهیه شده است. این مقاله با لحنی روان، تخصصی و همراه با مثال‌های کاربردی نگارش یافته است.

***

# 📘 آموزش جامع: استفاده از Tuple در Genericهای C#

به نام خدا. در این مقاله از مجموعه آموزش‌های پیشرفته #C، به یکی از جذاب‌ترین و کاربردی‌ترین ترکیب‌ها در این زبان می‌پردازیم: **استفاده از Tupleها به عنوان آرگومان در تایپ‌های Generic**. 
اگر تا به حال برای گروه‌بندی چند مقدار ساده، کلاس‌ها یا استراکچرهای اضافی نوشته‌اید، این مقاله دیدگاه شما را نسبت به نوشتن کدهای تمیزتر (Clean Code) تغییر خواهد داد.

---

## 📑 فهرست مطالب
- [۱. Generic چیست؟](#۱-گنریک-generic-چیست)
- [۲. استفاده از Tuple به عنوان Generic Argument](#۲-استفاده-از-tuple-به-عنوان-generic-argument)
- [۳. بررسی Collectionهای مختلف با Tuple](#۳-بررسی-collectionهای-مختلف-با-tuple)
  - [List](#list)
  - [Dictionary](#dictionary)
  - [HashSet](#hashset)
  - [Queue و Stack](#queue-و-stack)
- [۴. Tuple به عنوان Composite Key در Dictionary](#۴-tuple-به-عنوان-composite-key-در-dictionary)
- [۵. مفهوم Equality و HashCode در Tupleها](#۵-مفهوم-equality-و-hashcode-در-tupleها)
- [۶. مثال‌های واقعی و کاربردی](#۶-مثالهای-واقعی-و-کاربردی)
- [۷. نکات مهم و اشتباهات رایج](#۷-نکات-مهم-و-اشتباهات-رایج)
- [۸. جمع‌بندی](#۸-جمع-بندی)
- [۹. منابع معتبر](#۹-منابع-معتبر)

---

<a id="۱-گنریک-generic-چیست"></a>
## ۱. Generic چیست؟

قبل از ورود به بحث Tuple، باید یک مرور سریع بر **Generic** داشته باشیم. Generic به ما اجازه می‌دهد کلاس‌ها، متدها و ساختارهای داده‌ای بنویسیم که به یک تایپ خاص وابسته نیستند و در زمان استفاده، تایپ آن‌ها مشخص می‌شود. این کار باعث **Type-Safety** (ایمنی در سطح تایپ) و **Reusability** (قابلیت استفاده مجدد) می‌شود.

```csharp
// یک لیست که فقط می‌تواند اعداد صحیح را در خود نگه دارد
List<int> numbers = new List<int>(); 
```

اما چه می‌شود اگر بخواهیم در یک Collection، به جای یک تایپ ساده، **ترکیبی از چند تایپ** را نگهداری کنیم؟ اینجاست که Tupleها وارد میدان می‌شوند.

---

<a id="۲-استفاده-از-tuple-به-عنوان-generic-argument"></a>
## ۲. استفاده از Tuple به عنوان Generic Argument

از نسخه C# 7.0، ما **ValueTuple** را در اختیار داریم که یک ساختار داده‌ای سبک (Value Type) برای گروه‌بندی چند فیلد است. ما می‌توانیم از یک Tuple به عنوان `T` در یک Generic استفاده کنیم.

**سینتکس پایه:**
```csharp
(T1, T2, T3)
// یا با نام‌گذاری فیلدها برای خوانایی بیشتر:
(Type1 Name1, Type2 Name2)
```

**چرا این کار مهم است؟**
در گذشته برای نگهداری چند مقدار مرتبط (مثلاً طول و عرض جغرافیایی)، مجبور بودیم یک `class` یا `struct` بسازیم. حالا می‌توانیم مستقیماً از `(double Lat, double Lon)` در Genericها استفاده کنیم.

---

<a id="۳-بررسی-collectionهای-مختلف-با-tuple"></a>
## ۳. بررسی Collectionهای مختلف با Tuple

بیایید Tupleها را در مهم‌ترین Collectionهای دات‌نت بررسی کنیم.

### List
ساده‌ترین حالت، نگهداری چندتایی‌ها در یک لیست است.
```csharp
// لیستی از مختصات دوبعدی
List<(int X, int Y)> coordinates = new List<(int, int)>();

coordinates.Add((10, 20));
coordinates.Add((30, 40));

// خط به خط:
// 1. یک List تعریف کردیم که تایپ آن یک Tuple شامل دو int است.
// 2. نام‌های X و Y فقط برای خوانایی هستند (Named Elements).
// 3. با متد Add، یک Tuple جدید به لیست اضافه می‌کنیم.
```

### Dictionary
دیکشنری‌ها از دو Generic Argument استفاده می‌کنند: `Key` و `Value`. ما می‌توانیم `Key` را یک Tuple قرار دهیم.
```csharp
// کلید: (نام شهر، سال)، مقدار: میانگین دما
Dictionary<(string City, int Year), double> temperatures = new();

temperatures.Add(("Tehran", 2023), 18.5);
temperatures.Add(("Tehran", 2024), 19.1);

// دسترسی به مقدار:
double temp = temperatures[("Tehran", 2023)]; 
```

### HashSet
هش‌ست‌ها برای نگهداری **آیتم‌های یکتا** استفاده می‌شوند.
```csharp
// مجموعه‌ای از خانه‌های شطرنج که مهره در آن‌ها قرار دارد
HashSet<(int Row, int Col)> occupiedCells = new();

occupiedCells.Add((1, 1));
occupiedCells.Add((1, 1)); // این خط نادیده گرفته می‌شود چون تکراری است!

Console.WriteLine(occupiedCells.Count); // خروجی: 1
```

### Queue و Stack
برای الگوریتم‌های پیمایش گراف (مثل BFS و DFS) که نیاز به نگهداری مختصات دارند، Tupleها عالی هستند.
```csharp
// صف برای الگوریتم BFS در یک ماتریس
Queue<(int R, int C)> queue = new();
queue.Enqueue((0, 0));

// پشته برای الگوریتم DFS یا Backtracking
Stack<(int X, int Y)> stack = new();
stack.Push((5, 5));
```

---

<a id="۴-tuple-به-عنوان-composite-key-در-dictionary"></a>
## ۴. Tuple به عنوان Composite Key در Dictionary

### چرا به Composite Key نیاز داریم؟
گاهی اوقات یک کلید تکی برای شناسایی یک رکورد کافی نیست. مثلاً در یک سیستم دانشگاهی، یک دانشجو ممکن است در چندین درس ثبت‌نام کند. کلید ما باید ترکیبی از `StudentId` و `CourseId` باشد.

### چرا Tuple برای این کار بی‌نظیر است؟
در گذشته برای ساخت Composite Key مجبور بودیم یک کلاس بسازیم و متدهای `Equals` و `GetHashCode` را به صورت دستی Override کنیم تا دیکشنری بتواند کلیدها را مقایسه کند. 
**اما با Tuple، کامپایلر C# تمام این کارها را به صورت خودکار و بهینه برای شما انجام می‌دهد!**

```csharp
// Dictionary<(string, int), int>
// کلید ترکیبی: (شناسه دانشجو, شناسه درس) -> مقدار: نمره
Dictionary<(string StudentId, int CourseId), int> grades = new();

grades.Add(("S001", 101), 18);
grades.Add(("S001", 102), 19);
grades.Add(("S002", 101), 16);

// بررسی وجود کلید ترکیبی
bool exists = grades.ContainsKey(("S001", 101)); // true
```

---

<a id="۵-مفهوم-equality-و-hashcode-در-tupleها"></a>
## ۵. مفهوم Equality و HashCode در Tupleها

برای اینکه یک شیء بتواند به عنوان **Key** در `Dictionary` یا آیتم در `HashSet` استفاده شود، باید دو شرط را برآورده کند:
1. **Equality (برابری):** بتواند تشخیص دهد آیا دو شیء با هم برابرند یا خیر (`Equals`).
2. **HashCode (کد هش):** بتواند یک عدد یکتا برای مکان‌یابی سریع در HashTable تولید کند (`GetHashCode`).

**معجزه ValueTuple:**
تایپ `(T1, T2)` در واقع `ValueTuple<T1, T2>` است. این ساختار به صورت پیش‌فرض `IEquatable` را پیاده‌سازی کرده است.

```csharp
var key1 = ("Ali", 25);
var key2 = ("Ali", 25);
var key3 = ("Reza", 25);

Console.WriteLine(key1.Equals(key2)); // True (مقدارها برابرند)
Console.WriteLine(key1.Equals(key3)); // False
Console.WriteLine(key1.GetHashCode() == key2.GetHashCode()); // True
```
**توضیح خط به خط:**
- خط ۱ تا ۳: تعریف سه Tuple.
- خط ۵: متد `Equals` در ValueTuple به صورت خودکار تک‌تک فیلدها را با هم مقایسه می‌کند.
- خط ۷: متد `GetHashCode` به صورت خودکار HashCode هر فیلد را ترکیب (Combine) کرده و یک عدد یکتا می‌سازد.

---

<a id="۶-مثالهای-واقعی-و-کاربردی"></a>
## ۶. مثال‌های واقعی و کاربردی

### مثال ۱: سیستم Cache چندبعدی
فرض کنید می‌خواهید نتیجه یک API را کش کنید. کلید کش باید ترکیبی از `UserId` و `Endpoint` باشد.

```csharp
public class CacheService
{
    // دیکشنری برای کش کردن پاسخ‌ها
    private Dictionary<(string UserId, string Endpoint), string> _cache = new();

    public string GetOrSet(string userId, string endpoint, Func<string> fetchData)
    {
        var key = (userId, endpoint);
        
        if (!_cache.ContainsKey(key))
        {
            _cache[key] = fetchData(); // فراخوانی متد و ذخیره در کش
        }
        
        return _cache[key];
    }
}
```

### مثال ۲: پیدا کردن مسیر در ماتریس (Grid)
در بازی‌سازی یا پردازش تصویر، برای جلوگیری از بازدید مجدد از یک خانه در ماتریس:

```csharp
public void TraverseGrid(int[,] grid)
{
    HashSet<(int Row, int Col)> visited = new();
    Queue<(int R, int C)> queue = new();
    
    queue.Enqueue((0, 0));
    visited.Add((0, 0));

    while (queue.Count > 0)
    {
        var current = queue.Dequeue();
        
        // پردازش خانه فعلی...
        
        // فرض کنید می‌خواهیم خانه‌های مجاور را اضافه کنیم
        var nextCell = (current.R + 1, current.C);
        if (!visited.Contains(nextCell))
        {
            visited.Add(nextCell);
            queue.Enqueue(nextCell);
        }
    }
}
```

---

<a id="۷-نکات-مهم-و-اشتباهات-رایج"></a>
## ۷. نکات مهم و اشتباهات رایج

### ❌ اشتباه رایج ۱: استفاده از `Tuple` قدیمی به جای `ValueTuple`
دات‌نت یک کلاس قدیمی به نام `Tuple<T1, T2>` دارد (از نسخه 4.0). این یک **Reference Type** است!
```csharp
// اشتباه (کند و دارای سربار حافظه - Garbage Collection)
Dictionary<Tuple<string, int>, int> oldDict = new(); 

// درست (سریع، سبک و Value Type)
Dictionary<(string, int), int> newDict = new(); 
```
**نکته:** همیشه از سینتکس `(T1, T2)` استفاده کنید که معادل `ValueTuple` است.

### ❌ اشتباه رایج ۲: تغییر دادن (Mutate) کلید در Dictionary
تاپل‌ها (ValueTuple) **Mutable** (قابل تغییر) هستند. اگر کلیدی را از دیکشنری بیرون بکشید، تغییر دهید و دوباره بخواهید استفاده کنید، به مشکل می‌خورید.
```csharp
var key = ("A", 1);
dict.Add(key, 100);

// هرگز فیلدهای یک Tuple که به عنوان Key استفاده شده را تغییر ندهید!
// این کار باعث خراب شدن ساختار HashTable می‌شود.
```

### 💡 نکته طلایی: نام‌گذاری فیلدها (Named Elements)
همیشه فیلدهای Tuple را نام‌گذاری کنید. این کار در زمان کامپایل از بین می‌رود و هیچ سربار اجرایی (Runtime Overhead) ندارد، اما کد شما را برای دیگران (و خودتان در آینده) کاملاً خوانا می‌کند.
```csharp
// بد
Dictionary<(string, int), int> dict1; 

// عالی
Dictionary<(string CityName, int Year), int> dict2; 
```

---

<a id="۸-جمع-بندی"></a>
## ۸. جمع‌بندی

استفاده از Tuple در Genericها، به ویژه در Collectionهایی مانند `Dictionary` و `HashSet`، یکی از بهترین ویژگی‌های مدرن C# است. 
**مزایای کلیدی:**
1. **حذف Boilerplate Code:** دیگر نیازی به ساخت کلاس/استراکچرهای موقت برای Composite Keyها نیست.
2. **عملکرد بالا:** به دلیل استفاده از `ValueTuple`، هیچ سربار اضافی روی Garbage Collector تحمیل نمی‌شود.
3. **پیاده‌سازی خودکار Equality:** متدهای `Equals` و `GetHashCode` به صورت بهینه و خودکار پیاده‌سازی شده‌اند.
4. **خوانایی کد:** با استفاده از Named Tuples، کد شما خود-توضیح‌دهنده (Self-documenting) می‌شود.

---

<a id="۹-منابع-معتبر"></a>
## ۹. منابع معتبر

برای مطالعه بیشتر و عمیق‌تر، منابع زیر پیشنهاد می‌شوند:
1. [Microsoft Learn: Tuple types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples) - مستندات رسمی مایکروسافت درباره ValueTuple.
2. [Microsoft Learn: Generics](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/generics) - مستندات رسمی Genericها.
3. کتاب **C# in Depth** نوشته *Jon Skeet* - فصل مربوط به Tuples و Generics.
4. [Source Code: ValueTuple.cs](https://source.dot.net/System.Private.CoreLib/ValueTuple.cs.html) - بررسی سورس کد پیاده‌سازی GetHashCode در گیت‌هاب دات‌نت.

---
*نویسنده: [نام شما / نام تیم آموزشی]*
*تاریخ بازبینی: August 2026*
*در صورت داشتن سوال یا پیشنهاد برای بهبود این مقاله، خوشحال می‌شویم Issue یا PR ثبت کنید!* 🚀