# درک عمیق ValueTuple، Stack و Heap در #C (فراتر از افسانه‌ها)

یکی از رایج‌ترین سوءتفاهم‌ها در جامعه برنامه‌نویسان #C این جمله است: *"Value Typeها همیشه روی Stack ذخیره می‌شوند و Reference Typeها روی Heap."* این جمله نه تنها از نظر فنی **غلط** است، بلکه می‌تواند منجر به تصمیمات اشتباه در مدیریت حافظه و پرفورمنس شود.

در این مقاله آموزشی، با کالبدشکافی `ValueTuple`، `Stack` و `Heap`، این افسانه را برای همیشه از بین می‌بریم.

---

## 📑 فهرست مطالب
1. [مفاهیم پایه: Stack و Heap](#1-مفاهیم-پایه-stack-و-heap)
2. [انواع داده: Value Type در برابر Reference Type](#2-انواع-داده-value-type-در-برابر-reference-type)
3. [ValueTuple چیست و چرا Value Type است؟](#3-valuetuple-چیست-و-چرا-value-type-است)
4. [افسانه بزرگ: آیا Value Typeها همیشه روی Stack هستند؟](#4-افسانه-بزرگ-آیا-value-typeها-همیشه-روی-stack-هستند)
5. [محل ذخیره ValueTuple در Contextهای مختلف](#5-محل-ذخیره-valuetuple-در-contextهای-مختلف)
   - [ValueTuple به عنوان متغیر محلی](#الف-متغیر-محلی-local-variable)
   - [ValueTuple داخل یک Object (کلاس)](#ب-valuetuple-داخل-یک-object-کلاس)
   - [ValueTuple داخل یک Array](#ج-valuetuple-داخل-یک-array)
6. [حالت‌های خاص و لبه‌ای (Edge Cases)](#6-حالتهخاص-و-لبه‌ای-edge-cases)
   - [Boxing](#1-boxing)
   - [Closureها](#2-closureها)
   - [Async State Machine](#3-async-state-machine)
7. [جدول خلاصه محل ذخیره ValueTuple](#7-جدول-خلاصه-محل-ذخیره-valuetuple)
8. [اشتباهات رایج](#8-اشتباهات-رایج)
9. [جمع‌بندی](#9-جمع‌بندی)
10. [منابع رسمی و برای مطالعه بیشتر](#10-منابع-رسمی-و-برای-مطالعه-بیشتر)

---

## 1. مفاهیم پایه: Stack و Heap

برای درک محل ذخیره داده‌ها، ابتدا باید بدانیم Stack و Heap چه هستند:

*   **Stack (پشته):** حافظه‌ای است ساختاریافته، بسیار سریع و به‌صورت LIFO (آخرین ورودی، اولین خروجی) مدیریت می‌شود. Stack برای ذخیره **Context اجرای برنامه** (مانند متغیرهای محلی یک متد، پارامترها و آدرس بازگشت) استفاده می‌شود. هر Thread در برنامه یک Stack مختص به خود دارد. وقتی متدی به پایان می‌رسد، حافظه اختصاص‌یافته به آن روی Stack بلافاصله و بدون نیاز به Garbage Collector (GC) آزاد می‌شود.
*   **Heap (توده):** حافظه‌ای بزرگ، نامنظم و کندتر است که برای ذخیره **داده‌های پیچیده و اشیاء** استفاده می‌شود. Heap بین تمام Threadها مشترک است. مدیریت حافظه در Heap بر عهده **Garbage Collector (GC)** است.

---

## 2. انواع داده: Value Type در برابر Reference Type

در #C، انواع داده به دو دسته اصلی تقسیم می‌شوند:

*   **Value Type (نوع مقداری):** داده‌ای که **مقدار** را مستقیماً در خود نگه می‌دارد. شامل `struct`ها، `enum`ها، انواع عددی (`int`, `float`)، `bool`، `char`، `DateTime` و **`ValueTuple`**.
*   **Reference Type (نوع مرجعی):** داده‌ای که **آدرس (Reference)** به محل ذخیره مقدار را نگه می‌دارد. شامل `class`ها، `string`، `array`ها، `delegate`ها و `interface`ها.

> **نکته کلیدی:** تفاوت این دو در **نحوه کپی شدن** و **تخصیص حافظه** است، نه لزوماً محل فیزیکی ذخیره‌سازی!

---

## 3. ValueTuple چیست و چرا Value Type است؟

قبل از C# 7.0، ما `System.Tuple` را داشتیم که یک **کلاس (Reference Type)** بود. اما در C# 7.0 مایکروسافت `ValueTuple` را معرفی کرد.

چرا `ValueTuple` یک Value Type است؟
زیرا در سطح IL (زبان میانی)، `ValueTuple<T1, T2, ...>` به عنوان یک **`struct`** تعریف شده است. از آنجا که تمام `struct`ها در #C از `System.ValueType` ارث‌بری می‌کنند، `ValueTuple` نیز ذاتاً یک **Value Type** است. این طراحی عمدی بوده تا از سربار (Overhead) تخصیص حافظه روی Heap و فشار به Garbage Collector هنگام ایجاد Tupleهای موقت جلوگیری شود.

---

## 4. افسانه بزرگ: آیا Value Typeها همیشه روی Stack هستند؟

**خیر! این جمله کاملاً اشتباه است.**
قانون طلایی و دقیق CLR این است:
> **"Value Typeها دقیقاً در همان جایی ذخیره می‌شوند که تعریف (Declare) شده‌اند."**

*   اگر یک Value Type درون یک متد به عنوان متغیر محلی تعریف شود -> **روی Stack**.
*   اگر یک Value Type به عنوان فیلد درون یک کلاس (Reference Type) تعریف شود -> **روی Heap** (چون خودِ کلاس روی Heap است).
*   اگر یک Value Type درون یک آرایه تعریف شود -> **روی Heap** (چون آرایه‌ها در #C همیشه Reference Type هستند).

---

## 5. محل ذخیره ValueTuple در Contextهای مختلف

بیایید رفتار `ValueTuple` را در سناریوهای مختلف بررسی کنیم:

### الف) متغیر محلی (Local Variable)
وقتی `ValueTuple` را درون یک متد تعریف می‌کنید، روی Stack قرار می‌گیرد.
```csharp
public void Calculate()
{
    // این ValueTuple دقیقاً روی Stackِ همین متد قرار می‌گیرد
    var point = (X: 10, Y: 20); 
}
```

### ب) ValueTuple داخل یک Object (کلاس)
اگر `ValueTuple` را به عنوان فیلد در یک کلاس استفاده کنید، چون خودِ کلاس روی Heap ساخته می‌شود، `ValueTuple` نیز **درون همان بلاک حافظه Heapِ مربوط به آن شیء** قرار می‌گیرد.
```csharp
public class Player
{
    // این ValueTuple روی Heap ذخیره می‌شود (چون Player یک Reference Type است)
    public (int Health, int Mana) Stats; 
}
```

### ج) ValueTuple داخل یک Array
آرایه‌ها در #C همیشه Reference Type هستند و روی Heap قرار می‌گیرند. بنابراین آرایه‌ای از `ValueTuple`ها نیز روی Heap خواهد بود.
```csharp
// آرایه روی Heap است و عناصر ValueTuple آن نیز درون همان Heap قرار می‌گیرند
var coordinates = new (int X, int Y)[100]; 
```

---

## 6. حالت‌های خاص و لبه‌ای (Edge Cases)

گاهی اوقات کامپایلر یا CLR مجبور می‌شوند یک Value Type را از Stack به Heap منتقل کنند. این اتفاقات در `ValueTuple` نیز رخ می‌دهد:

### 1. Boxing
وقتی یک Value Type را به یک `object` یا یک `interface` که پیاده‌سازی کرده است Cast می‌کنید، CLR مجبور است یک کپی از آن روی **Heap** بسازد (به این عمل Boxing می‌گویند).
```csharp
var myTuple = (1, "Hello");
object boxedTuple = myTuple; // Boxing رخ داد! حالا روی Heap است.
```
*نکته: `ValueTuple`ها وقتی به `object` تبدیل می‌شوند، ممکن است به عنوان یک `Tuple` معمولی (Reference Type) بازنمایی شوند تا از پیاده‌سازی اینترفیس‌ها پشتیبانی کنند، اما در هر صورت روی Heap قرار می‌گیرند.*

### 2. Closureها
اگر یک متغیر محلی (از جمله `ValueTuple`) توسط یک Lambda Expression یا Anonymous Method "Capture" شود، کامپایلر #C یک کلاس مخفی (Closure Class) روی **Heap** می‌سازد و متغیر شما را به عنوان فیلد آن کلاس قرار می‌دهد تا طول عمر آن تضمین شود.
```csharp
public void DoAction()
{
    var myTuple = (10, 20); // در حالت عادی روی Stack بود
    
    // به دلیل Closure، myTuple به یک کلاس مخفی روی Heap منتقل می‌شود
    Action print = () => Console.WriteLine(myTuple.Item1); 
}
```

### 3. Async State Machine
وقتی از `async` و `await` استفاده می‌کنید، کامپایلر متد شما را به یک State Machine تبدیل می‌کند. متغیرهای محلی شما (از جمله `ValueTuple`ها) به فیلدهای این State Machine تبدیل می‌شوند.
وقتی متد به `await` می‌رسد و suspend می‌شود، State Machine باید در حافظه زنده بماند. بنابراین، State Machine (و در نتیجه تمام `ValueTuple`های درون آن) روی **Heap** قرار می‌گیرند (معمولاً از طریق Boxing یا ذخیره در یک کلاس Wrapper).
```csharp
public async Task ProcessAsync()
{
    var data = (Id: 1, Name: "Test"); // در حالت عادی روی Stack است
    
    await Task.Delay(100); // در این لحظه، data به همراه State Machine به Heap منتقل می‌شود!
    
    Console.WriteLine(data.Name);
}
```

---

## 7. جدول خلاصه محل ذخیره ValueTuple

| سناریو (Context) | نوع والد / مکان تعریف | محل ذخیره ValueTuple |
| :--- | :--- | :--- |
| متغیر محلی در متد معمولی | Stack Frame | **Stack** |
| فیلد در یک `class` یا `record class` | Heap Object | **Heap** |
| فیلد در یک `struct` دیگر | بستگی به struct والد دارد | **بستگی دارد** |
| عنصر درون یک `Array` | Heap Object | **Heap** |
| متغیر محلی در متد `async` (قبل از await) | Stack Frame | **Stack** |
| متغیر محلی در متد `async` (بعد از await) | Async State Machine | **Heap** |
| متغیر Capture شده در Closure | Closure Class | **Heap** |
| Cast به `object` یا `interface` | Boxing | **Heap** |

---

## 8. اشتباهات رایج

1.  ❌ **اشتباه:** "چون `ValueTuple` یک Value Type است، پس همیشه روی Stack است و Allocation صفر دارد."
    ✅ **درست:** اگر در Closure، متد Async یا داخل یک کلاس استفاده شود، روی Heap رفته و باعث Allocation می‌شود.
2.  ❌ **اشتباه:** "برای جلوگیری از فشار به GC باید همه چیز را `struct` یا `ValueTuple` کنم."
    ✅ **درست:** اگر `struct`ها را در آرایه‌های بزرگ، داخل کلاس‌ها یا در Closureها استفاده کنید، همچنان روی Heap می‌روند و GC را درگیر می‌کنند.
3.  ❌ **اشتباه:** "`System.Tuple` و `ValueTuple` یکی هستند."
    ✅ **درست:** `System.Tuple` یک کلاس (Reference Type) است و همیشه روی Heap می‌رود. `System.ValueTuple` یک ساختار (Value Type) است و رفتار متفاوتی دارد.

---

## 9. جمع‌بندی

جمله‌ی *"ValueTuple همیشه روی Stack است"* یک **ساده‌سازی نادرست و خطرناک** است. 
`ValueTuple` یک `struct` (Value Type) است و قانون طلایی Value Typeها این است که **در همان جایی ذخیره می‌شوند که تعریف شده‌اند**. اگر درون یک متد ساده باشند، روی Stack هستند (که بسیار سریع و بهینه است). اما اگر درون یک کلاس، آرایه، Closure یا یک متد Async باشند، روی Heap قرار می‌گیرند.

درک این موضوع به شما کمک می‌کند تا در معماری نرم‌افزار، هنگام کار با Dapper، Entity Framework یا نوشتن APIهای پرformance، تصمیمات درستی درباره استفاده از `ValueTuple` در برابر `class`ها بگیرید.

---

## 10. منابع رسمی و برای مطالعه بیشتر

برای اثبات علمی این مفاهیم، می‌توانید به منابع زیر که از معتبرترین مراجع جامعه #C و مایکروسافت هستند مراجعه کنید:

1.  **Eric Lippert's Blog (از طراحان سابق کامپایلر C#):**
    *   [The Stack Is An Implementation Detail (Part 1)](https://docs.microsoft.com/en-us/archive/blogs/ericlippert/the-stack-is-an-implementation-detail)
    *   [The Stack Is An Implementation Detail (Part 2)](https://docs.microsoft.com/en-us/archive/blogs/ericlippert/the-stack-is-an-implementation-detail-part-two)
    *(این دو مقاله، مرجع اصلی برای رد کردن افسانه "Value Type = Stack" هستند).*
2.  **Microsoft C# Documentation:**
    *   [Value Types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)
    *   [Tuples (ValueTuple)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
3.  **کتاب CLR via C# (نوشته Jeffrey Richter):**
    *   فصل 5 (Primitive, Reference, and Value Types) - مرجع کامل برای درک نحوه مدیریت حافظه توسط CLR.
4.  **مستندات ECMA-334 (C# Language Specification):**
    *   بخش مربوط به Value Types و متغیرهای محلی.