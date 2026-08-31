

## فهرست مطالب
1. [Pattern Matching چیست؟](#1-pattern-matching-چیست)
2. [محل استفاده Patternها در C#](#2-محل-استفاده-patternها-در-c)
3. [Pattern Matching و Compiler](#3-pattern-matching-و-compiler)
4. [Constant Pattern](#4-constant-pattern)
5. [Type Pattern](#5-type-pattern)
6. [Declaration Pattern](#6-declaration-pattern)
7. [Relational Patterns](#7-relational-patterns)
8. [Logical Pattern Combinators](#8-logical-pattern-combinators)
9. [اولویت Pattern Combinatorها](#9-اولویت-pattern-combinatorها)
10. [Var Pattern](#10-var-pattern)
11. [Discard Pattern](#11-discard-pattern)
12. [Parenthesized Pattern](#12-parenthesized-pattern)
13. [Property Pattern](#13-property-pattern)
14. [Nested Property Pattern](#14-nested-property-pattern)
15. [Property Pattern و when](#15-property-pattern-و-when)
16. [Positional Pattern](#16-positional-pattern)
17. [Tuple Pattern](#17-tuple-pattern)
18. [List Pattern](#18-list-pattern)
19. [Discard در List Pattern](#19-discard-در-list-pattern)
20. [Var Pattern در List Pattern](#20-var-pattern-در-list-pattern)
21. [Slice Pattern](#21-slice-pattern)
22. [Slice Pattern همراه با Variable](#22-slice-pattern-همراه-با-variable)
23. [Recursive Patterns](#23-recursive-patterns)
24. [Patternهای ترکیبی](#24-patternهای-ترکیبی)
25. [Pattern Matching با null](#25-pattern-matching-با-null)
26. [Pattern Matching و switch statement](#26-pattern-matching-و-switch-statement)
27. [Pattern Matching و switch expression](#27-pattern-matching-و-switch-expression)
28. [Exhaustive Pattern Matching](#28-exhaustive-pattern-matching)
29. [Pattern Order و Subsumption](#29-pattern-order-و-subsumption)
30. [Pattern Matching و Type Hierarchy](#30-pattern-matching-و-type-hierarchy)
31. [Pattern Matching و Generics](#31-pattern-matching-و-generics)
32. [Pattern Matching و Performance](#32-pattern-matching-و-performance)
33. [Pattern Matching و IL](#33-pattern-matching-و-il)
34. [Pattern Matching در طراحی کد](#34-pattern-matching-در-طراحی-کد)
35. [Pattern Matching در Domain Modeling](#35-pattern-matching-در-domain-modeling)
36. [Pattern Matching در ASP.NET Core](#36-pattern-matching-در-aspnet-core)
37. [Pattern Matching و Result Type](#37-pattern-matching-و-result-type)
38. [اشتباهات رایج](#38-اشتباهات-رایج)
39. [Pattern Matching و Readability](#39-pattern-matching-و-readability)
40. [تکامل Pattern Matching (C# 7 تا 13)](#40-pattern-matching-از-c-7-تا-نسخه‌های-جدید)
41. [مقایسه Patternها](#41-مقایسه-patternها)
42. [Decision Guide](#42-decision-guide)
43. [مثال‌های واقعی](#43-مثال‌های-واقعی)
44. [تمرین‌ها](#44-تمرین‌ها)
45. [پروژه عملی: Order Processing Engine](#45-پروژه-عملی)
46. [جمع‌بندی نهایی و Cheat Sheet](#46-جمع‌بندی-نهایی)
- [منابع](#منابع)

---

## 1. Pattern Matching چیست؟
**Pattern (الگو)**: یک قالب یا ساختار مشخص است که داده‌ها را توصیف می‌کند.
**Pattern Matching (تطبیق الگو)**: فرآیندی است که در آن کامپایلر بررسی می‌کند آیا یک داده با یک "الگو" مطابقت دارد یا خیر. اگر مطابقت داشت، می‌تواند بخش‌هایی از آن داده را استخراج (Extract) کند.

**چرا به C# اضافه شد؟**
- کاهش کدهای تودرتو (Nested `if`/`else`).
- حذف Castهای دستی و ناامن.
- نوشتن کد Declarative (توصیف "چه چیزی" می‌خواهیم) به جای Imperative (توصیف "چگونه" آن را بگیریم).

**تفاوت با مقایسه معمولی**: مقایسه معمولی (`==`) فقط برابری مقدار را چک می‌کند. Pattern Matching می‌تواند **نوع (Type)**، **ساختار (Structure)** و **مقدار** را همزمان بررسی و استخراج کند.
**نکته مهم**: Pattern Matching یک قابلیت زبانی (Language Feature) است که توسط Roslyn مدیریت می‌شود، نه یک ویژگی مستقل در CLR. کامپایلر این الگوها را به عملیات سطح پایین‌تر (مانند `isinst`، `brtrue` و مقایسه‌های معمولی) تبدیل (Lower) می‌کند.

---

## 2. محل استفاده Patternها در C#
الگوها را می‌توان در Contextهای زیر استفاده کرد:
1. **`is` expression**: برای بررسی شرطی ساده.
   ```csharp
   if (obj is string) { }
   ```
2. **`switch` statement**: برای کنترل جریان برنامه بر اساس الگو.
   ```csharp
   switch (obj) { case string s: break; }
   ```
3. **`switch` expression**: (از C# 8) برای مقداردهی بر اساس الگو.
   ```csharp
   var result = obj switch { string s => s.Length, _ => 0 };
   ```
4. **`case` و `when`**: برای ترکیب الگو با شرط‌های اضافی (Guard).

---

## 3. Pattern Matching و Compiler
هنگامی که شما می‌نویسید:
```csharp
if (obj is 3) { }
```
کامپایلر (Roslyn) این کد را در مرحله Compilation به IL (Intermediate Language) تبدیل می‌کند. فرآیند به این صورت است:
1. **Source Code**: کد C# شما.
2. **Roslyn**: تجزیه و تحلیل Syntax و Semantic.
3. **Lowering**: تبدیل Pattern به عملیات ساده‌تر (مثلاً بررسی `obj != null` و سپس `obj.Equals(3)` یا مقایسه مستقیم اگر نوع مشخص باشد).
4. **IL**: کد بایتی که شامل دستورالعمل‌هایی مانند `isinst` (برای Type Pattern) یا `ceq` (برای مقایسه) است.
5. **Runtime**: CLR کد IL را اجرا می‌کند. CLR هیچ مفهومی به نام "Pattern Matching" ندارد؛ این فقط کامپایلر است که آن را به منطق‌های موجود ترجمه کرده است.

> ⚠️ **هشدار**: جزئیات دقیق IL تولیدشده ممکن است بین نسخه‌های کامپایلر تغییر کند. هرگز رفتار Compiler را به عنوان Contract زبان در نظر نگیرید، مگر اینکه در C# Language Specification ذکر شده باشد.

---

## 4. Constant Pattern
**Constant Pattern** بررسی می‌کند که آیا مقدار ورودی دقیقاً با یک مقدار ثابت (Constant) برابر است یا خیر.

**Syntax**: `input is constant_value`
**انواع مقادیر مجاز**: اعداد، رشته‌ها، `boolean`، `null`، `enum` و فیلدهای `const`.

```csharp
object value = 5;
if (value is 5) // Constant Pattern
{
    Console.WriteLine("value is 5");
}
```
**تفاوت با `==`**: عبارت `value == 5` اگر `value` از نوع `object` باشد، ممکن است باعث Boxing شود یا اگر `==` Overload شده باشد، رفتار متفاوتی داشته باشد. اما `value is 5` همیشه از نظر معنایی (Semantics) برابری مقدار را با ایمنی نوع بررسی می‌کند و برای `null` به‌صورت ذاتی ایمن است.

---

## 5. Type Pattern
**Type Pattern** بررسی می‌کند که آیا یک شیء از نوع مشخصی است یا خیر و در صورت موفقیت، آن را به آن نوع Cast می‌کند.

```csharp
object obj = "Hello";
if (obj is string text) // Type/Declaration Pattern
{
    Console.WriteLine(text.Length); // text از نوع string است
}
```
**چرا بهتر از روش قدیمی است؟**
روش قدیمی:
```csharp
if (obj is string) {
    var text = (string)obj; // دو بار بررسی نوع (Double Type Check)
}
```
روش جدید با یک بار بررسی، هم نوع را چک می‌کند و هم متغیر را مقداردهی می‌کند.

---

## 6. Declaration Pattern
این الگو در واقع ترکیبی از Type Pattern و تعریف متغیر است. متغیر تعریف‌شده (Pattern Variable) فقط در Scope بلوک `if` یا `case` معتبر است (Definite Assignment). اگر مقدار ورودی `null` باشد، الگو Match نمی‌شود (مگر اینکه نوع Nullable باشد).

---

## 7. Relational Patterns
(معرفی شده در **C# 9**)
امکان مقایسه با عملگرهای `<`، `>`، `<=`، `>=` را فراهم می‌کند.

```csharp
int score = 85;
if (score is >= 80 and <= 89)
{
    Console.WriteLine("Grade B");
}

// در Switch Expression
string grade = score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _ => "F"
};
```
**نکته**: ترتیب در `switch` مهم است. کامپایلر از بالا به پایین بررسی می‌کند. اگر `>= 70` قبل از `>= 80` بیاید، هیچ‌گاه به `>= 80` نمی‌رسد (Subsumption).

---

## 8. Logical Pattern Combinators
(معرفی شده در **C# 9**)
شامل `and`، `or`، و `not`. این‌ها عملگرهای منطقی مخصوص الگوها هستند و با `&&`، `||`، `!` تفاوت دارند (این‌ها بخشی از Syntax الگو هستند).

```csharp
if (value is >= 10 and <= 100) { }
if (value is 1 or 2 or 3) { }
if (value is not null) { }
```

---

## 9. اولویت Pattern Combinatorها
ترتیب اولویت (Precedence) به این صورت است:
1. `not` (بالاترین)
2. `and`
3. `or` (پایین‌ترین)

**مثال خطرناک**:
```csharp
// اشتباه: به معنای (not (>= 10 and <= 100)) است
if (value is not >= 10 and <= 100) 

// صحیح و خوانا: استفاده از پرانتز
if (value is (>= 10) and (<= 100))
```
همیشه برای ترکیب‌های پیچیده از **Parenthesized Pattern** استفاده کنید تا از خطاهای منطقی جلوگیری شود.

---

## 10. Var Pattern
الگوی `var` همیشه Match می‌شود (حتی برای `null`) و مقدار ورودی را در یک متغیر محلی Capture می‌کند. نوع متغیر توسط کامپایلر استنتاج (Infer) می‌شود.

```csharp
string name = "janet";
if (name.ToUpper() is var upper && (upper == "JANET" || upper == "JOHN"))
{
    // upper از نوع string است و مقدار "JANET" دارد
}
```
**کاربرد**: زمانی که می‌خواهید یک مقدار را یک بار محاسبه کنید و چندین بار روی آن شرط بگذارید.
**هشدار**: استفاده بی‌رویه از `var` در الگوها خوانایی را کاهش می‌دهد.

---

## 11. Discard Pattern (`_`)
نماد `_` به معنای "هر مقداری" است و آن مقدار را نادیده می‌گیرد (Assign نمی‌کند).

```csharp
object obj = "test";
if (obj is string _) // فقط چک می‌کند که آیا string است، بدون ساخت متغیر
{
}
```
در `switch` برای حالت پیش‌فرض (Default) استفاده می‌شود. تفاوت آن با Discard Variable در این است که در الگو، `_` هیچ فضایی در حافظه اشغال نمی‌کند و صرفاً یک ساختار Syntax است.

---

## 12. Parenthesized Pattern
استفاده از `()` برای کنترل اولویت و افزایش خوانایی.
```csharp
if (value is (1 or 2) and not 0)
```
این الگو تضمین می‌کند که `or` قبل از `and` ارزیابی می‌شود، صرف‌نظر از اولویت پیش‌فرض.

---

## 13. Property Pattern
(معرفی شده در **C# 8**)
امکان بررسی Propertyهای یک شیء بدون نیاز به استخراج کامل آن را می‌دهد.

```csharp
public class Person { public int Age { get; set; } public bool IsActive { get; set; } }

bool CanVote(Person person) => person switch
{
    { Age: >= 18, IsActive: true } => true,
    _ => false
};
```
**مزیت**: وابستگی به ترتیب اعضا (برخلاف Positional Pattern) ندارد و کد را Self-documenting می‌کند.

---

## 14. Nested Property Pattern
امکان بررسی Propertyهای تو در تو.
```csharp
if (person is { Address: { Country: "Iran", City: "Tehran" } })
```
**Extended Property Pattern (C# 10)**: سینتکس تمیزتر برای دسترسی به اعضای تو در تو:
```csharp
if (person is { Address.Country: "Iran", Address.City: "Tehran" })
```

---

## 15. Property Pattern و `when`
گاهی الگو به تنهایی کافی نیست و نیاز به منطق پیچیده‌تری دارید. از `when` (Guard Clause) استفاده می‌شود.

```csharp
person switch
{
    { Age: >= 18 } when IsAllowed(person) => true,
    _ => false
};
```
**تفاوت**: الگو (Pattern) ساختار داده را چک می‌کند. `when` یک عبارت بولی (Boolean Expression) است که پس از Match شدن الگو ارزیابی می‌شود. اگر منطق صرفاً مقایسه مقدار است، از Relational Pattern استفاده کنید؛ اگر نیاز به فراخوانی متد یا منطق پیچیده است، از `when` استفاده کنید.

---

## 16. Positional Pattern
بر اساس قابلیت **Deconstruction** کار می‌کند. اگر کلاسی متد `Deconstruct` داشته باشد، می‌توان مقادیر آن را بر اساس موقعیت (Position) تطبیق داد.

```csharp
public record Point(int X, int Y); // Recordها به صورت پیش‌فرض Deconstruct دارند

Point p = new(10, 20);
if (p is (10, 20))
{
    Console.WriteLine("Origin offset");
}
```
کامپایلر متد `Deconstruct(out int x, out int y)` را فراخوانی می‌کند.

---

## 17. Tuple Pattern
برای تصمیم‌گیری بر اساس ترکیب چند مقدار به صورت همزمان بسیار ایده‌آل است.

```csharp
enum Season { Spring, Summer, Autumn, Winter }

int GetTemp(Season season, bool isDayTime) => (season, isDayTime) switch
{
    (Season.Spring, true) => 20,
    (Season.Spring, false) => 16,
    (Season.Summer, true) => 27,
    _ => 0
};
```
این روش از تودرتو شدن `if` یا `switch`های متعدد جلوگیری می‌کند.

---

## 18. List Pattern
(معرفی شده در **C# 11**)
امکان تطبیق ساختار آرایه‌ها یا لیست‌ها را فراهم می‌کند.

```csharp
int[] numbers = [1, 2, 3];
if (numbers is [1, 2, 3])
{
    Console.WriteLine("Exact match");
}
```
**شرط کامپایلر**: نوع داده باید دارای Property `Length` یا `Count` و یک Indexer (`[]`) باشد (مانند `Array`, `List<T>`, `Span<T>`, `ReadOnlySpan<T>`).

---

## 19. Discard در List Pattern
برای نادیده گرفتن یک عنصر خاص در موقعیتی مشخص.
```csharp
if (numbers is [1, _, 3]) // آرایه‌ای با 3 عنصر که اولی 1 و سومی 3 است
```

---

## 20. Var Pattern در List Pattern
برای استخراج (Capture) یک عنصر خاص از لیست.
```csharp
if (numbers is [1, 2, var third, 4, 5])
{
    Console.WriteLine(third); // third از نوع int است
}
```

---

## 21. Slice Pattern
(معرفی شده در **C# 11**)
با استفاده از `..` می‌توان صفر یا چند عنصر را در لیست نادیده گرفت یا تطبیق داد. فقط یک `..` در هر List Pattern مجاز است.

```csharp
if (numbers is [1, 2, .., 8, 9]) // با 1 و 2 شروع و به 8 و 9 ختم می‌شود
```

---

## 22. Slice Pattern همراه با Variable
می‌توان Slice را در یک متغیر Capture کرد.
```csharp
if (numbers is [1, .. var middle, 9])
{
    // middle از نوع int[] یا ReadOnlySpan<T> (بسته به نوع ورودی) خواهد بود
}
```
**محدودیت**: Capture کردن Slice برای برخی Collectionها ممکن است باعث Allocation شود. برای `Span<T>` به صورت `ReadOnlySpan<T>` برگردانده می‌شود که بهینه است.

---

## 23. Recursive Patterns
الگوها می‌توانند به صورت بازگشتی درون یکدیگر قرار گیرند. Property Pattern، Positional Pattern و Tuple Pattern ذاتاً Recursive هستند زیرا اجازه می‌دهند الگوهای کوچک‌تری را درون خود تعریف کنید.

```csharp
if (person is { Address: { City: "Tehran" } and not null })
```

---

## 24. Patternهای ترکیبی
ترکیب چند الگو برای بیان منطق پیچیده به صورت خوانا:
```csharp
// ترکیب Type، Relational و Logical
if (value is int number and >= 0 and <= 100) { }

// ترکیب Property و Nested
if (person is { Age: >= 18, Address.Country: "Iran" }) { }

// ترکیب List و Discard و Slice
if (numbers is [1, _, ..]) { }
```

---

## 25. Pattern Matching با null
استفاده از `is null` یا `is not null` بهترین روش برای بررسی تهی بودن است.

```csharp
if (value is null) { }
```
**چرا بهتر از `value == null` است؟**
اگر کلاس `value` عملگر `==` را Overload کرده باشد، `value == null` ممکن است منطق سفارشی (و گاهی باگ‌دار) را اجرا کند. اما `value is null` همیشه بررسی هویت (Reference Equality) سطح پایین را انجام می‌دهد و تحت تأثیر Overloading قرار نمی‌گیرد.

---

## 26. Pattern Matching و switch statement
استفاده از الگوها در `switch` سنتی:
```csharp
switch (obj)
{
    case null:
        throw new ArgumentNullException();
    case string s when s.Length > 5:
        Console.WriteLine("Long string");
        break;
    case int i and >= 10:
        Console.WriteLine("Large int");
        break;
    case _:
        Console.WriteLine("Other");
        break;
}
```

---

## 27. Pattern Matching و switch expression
(معرفی شده در **C# 8**)
نسخه Expression-based که همیشه یک مقدار برمی‌گرداند و نیازی به `break` یا `return` در هر شاخه ندارد.

```csharp
string result = value switch
{
    1 => "One",
    2 => "Two",
    _ => "Unknown" // Discard Pattern برای حالت پیش‌فرض الزامی است
};
```
اگر هیچ الگویی Match نشود و حالت پیش‌فرض (`_`) وجود نداشته باشد، در Runtime استثنام `SwitchExpressionException` پرتاب می‌شود.

---

## 28. Exhaustive Pattern Matching
کامپایلر بررسی می‌کند که آیا تمام حالت‌های ممکن پوشش داده شده‌اند یا خیر.
- برای `enum`ها (بدون پرچم Flags) و `bool`، اگر همه موارد ذکر شوند، Exhaustive است.
- برای سایر انواع، وجود الگوی Discard (`_`) برای اطمینان از Exhaustive بودن الزامی است.
- نادیده گرفتن Exhaustiveness منجر به Warning کامپایلر یا خطای Runtime در Switch Expression می‌شود.

---

## 29. Pattern Order و Subsumption
ترتیب `case`ها در `switch` حیاتی است. کامپایلر از بالا به پایین بررسی می‌کند.
```csharp
var result = value switch
{
    _ => "Anything", // این الگو همه چیز را Match می‌کند
    5 => "Five"      // Warning: Unreachable code (Subsumed Pattern)
};
```
**Subsumed Pattern**: الگویی که هرگز اجرا نمی‌شود زیرا یک الگوی عمومی‌تر قبل از آن قرار گرفته است. همیشه الگوهای خاص‌تر (Specific) را قبل از الگوهای عمومی‌تر (General) قرار دهید.

---

## 30. Pattern Matching و Type Hierarchy
هنگام بررسی سلسله‌مراتب انواع، همیشه نوع Derived (مشتق‌شده) را قبل از Base (پایه) بررسی کنید.
```csharp
object obj = new Dog();
if (obj is Dog d) { } // صحیح
else if (obj is Animal a) { }
```
اگر جای این دو عوض شود، `Animal` همیشه Match می‌شود و `Dog` هرگز اجرا نمی‌شود.

---

## 31. Pattern Matching و Generics
استفاده از Pattern Matching در کد Generic نیازمند دقت است.
```csharp
void Process<T>(T value)
{
    if (value is string s) // در C# مجاز است، اما اگر T یک ValueType باشد، همیشه false است
    {
    }
}
```
کامپایلر این کد را می‌پذیرد، اما در Runtime اگر `T` یک `struct` باشد، بررسی `is string` همیشه `false` برمی‌گرداند (مگر اینکه `T` محدود به `class` شده باشد).

---

## 32. Pattern Matching و Performance
**آیا Pattern Matching سریع‌تر است؟**
پاسخ مطلق وجود ندارد. بستگی دارد به:
1. **نوع الگو**: Type Pattern ممکن است به `isinst` در IL تبدیل شود که سریع است.
2. **Boxing**: استفاده از الگو روی `object` که حاوی Value Type است، ممکن است باعث Boxing شود (مانند روش‌های قدیمی).
3. **Compiler Optimization**: کامپایلرهای جدید (C# 9+) Switch Expressionها را به جداول پرش (Jump Tables) یا دستورالعمل‌های بهینه‌شده تبدیل می‌کنند که اغلب از `if/else` زنجیره‌ای سریع‌تر است.

> ⚠️ **توصیه**: بدون پروفایلینگ (BenchmarkDotNet) ادعا نکنید که Pattern Matching سریع‌تر است. هدف اصلی آن **خوانایی** و **ایمنی** است، نه لزوماً پرفورمنس محض.

---

## 33. Pattern Matching و IL
برای بررسی رفتار کامپایلر، از ابزارهایی مانند **SharpLab.io** استفاده کنید.
مثال: `if (obj is string s)`
در IL به این صورت Lowering می‌شود:
1. `isinst [System.Runtime]System.String` (بررسی نوع و Cast همزمان)
2. `dup` (کپی مرجع برای بررسی null)
3. `brtrue.s` (اگر null نبود، به بلوک کد برو)
4. `pop` (در غیر این صورت، مقدار را از استک حذف کن)

این نشان می‌دهد که Pattern Matching یک جادوی Runtime نیست، بلکه ترکیب هوشمندانه‌ای از دستورالعمل‌های موجود IL است.

---

## 34. Pattern Matching در طراحی کد
**مزایا**:
- کاهش Nested `if`ها (کاهش Cyclomatic Complexity).
- حذف Castهای صریح (`(string)obj`).
- کد Declarativeتر و نزدیک‌تر به منطق تجاری.

**معایب**:
- استفاده بیش از حد (Overuse) می‌تواند کد را غیرقابل خواندن کند (مخصوصاً الگوهای تو در توی عمیق).

---

## 35. Pattern Matching در Domain Modeling
الگوها برای مدل‌سازی وضعیت‌های دامنه (Domain States) عالی هستند.
```csharp
// تصمیم‌گیری بر اساس ترکیب وضعیت‌ها
var action = (order.Status, payment.Status) switch
{
    (OrderStatus.Pending, PaymentStatus.Paid) => ProcessOrder,
    (OrderStatus.Pending, PaymentStatus.Failed) => NotifyUser,
    _ => LogError
};
```

---

## 36. Pattern Matching در ASP.NET Core
کاربرد در Middleware یا Controllerها برای پردازش نتیجه:
```csharp
public IActionResult Handle(Result result) => result switch
{
    SuccessResult(var data) => Ok(data),
    NotFoundResult => NotFound(),
    ValidationErrorResult(var errors) => BadRequest(errors),
    _ => StatusCode(500)
};
```

---

## 37. Pattern Matching و Result Type
پیاده‌سازی الگوی Result برای مدیریت خطا بدون Exception:
```csharp
public abstract record Result;
public record Success(string Data) : Result;
public record Error(string Message) : Result;

string Process(Result res) => res switch
{
    Success s => $"Success: {s.Data}",
    Error e => $"Failed: {e.Message}",
    _ => "Unknown"
};
```

---

## 38. اشتباهات رایج
| اشتباه | مثال بد | مشکل | نسخه بهتر |
| :--- | :--- | :--- | :--- |
| **ترتیب اشتباه** | `case var x:` قبل از `case 5:` | الگوی `5` هرگز اجرا نمی‌شود (Unreachable). | الگوهای خاص را اول بگذارید. |
| **استفاده نادرست از `not`** | `value is not 1 or 2` | به معنای `(not 1) or 2` است، نه `not (1 or 2)`. | `value is not (1 or 2)` |
| **پیچیدگی بی‌مورد** | `if (obj is { Prop: { SubProp: 5 } })` | خوانایی کم. | گاهی `if (obj.Prop.SubProp == 5)` خواناتر است. |
| **نادیده گرفتن Exhaustiveness** | حذف `_` در Switch Expression | پرتاب استثنا در Runtime. | همیشه `_` را برای موارد پیش‌بینی‌نشده اضافه کنید. |

---

## 39. Pattern Matching و Readability
- **`if/else`**: برای شرط‌های ساده، بولی و تک‌بعدی.
- **`switch` statement**: وقتی نیاز به انجام چندین عملیات جانبی (Side Effects) در هر شاخه دارید.
- **`switch` expression**: وقتی هدف اصلی **تخصیص مقدار** (Assignment) یا **Return** بر اساس شرایط است. این خواناترین حالت برای نگاشت (Mapping) است.

---

## 40. Pattern Matching از C# 7 تا نسخه‌های جدید
- **C# 7.0**: `is` با Type Pattern، `switch` با Type/Constant.
- **C# 8.0**: Switch Expressions، Property، Tuple، Positional Patterns.
- **C# 9.0**: Relational Patterns (`<`, `>`), Logical Patterns (`and`, `or`, `not`), Parenthesized Patterns، Type Pattern در `is` بدون متغیر.
- **C# 10.0**: Extended Property Patterns (`{ Prop1.Prop2: value }`).
- **C# 11.0**: List Patterns، Slice Patterns (`..`).
- **C# 12/13**: بهبودهای جزئی در استنتاج نوع و پشتیبانی بهتر از `ReadOnlySpan<T>` در List Patterns.

---

## 41. مقایسه Patternها

| Pattern | هدف | مثال | مناسب برای |
| :--- | :--- | :--- | :--- |
| **Constant** | بررسی مقدار ثابت | `is 5` | مقادیر مشخص، `null` |
| **Type** | بررسی نوع | `is string` | چک کردن نوع بدون استخراج |
| **Declaration** | بررسی نوع + استخراج | `is string s` | جایگزین `is` + Cast |
| **Relational** | مقایسه محدوده | `is >= 10` | بازه‌های عددی یا کاراکتری |
| **Logical** | ترکیب شرایط | `is > 0 and < 10` | ترکیب چند الگوی ساده |
| **Var** | ذخیره موقت مقدار | `is var x` | کپچر مقدار برای استفاده بعدی |
| **Discard** | نادیده گرفتن مقدار | `_` | حالت پیش‌فرض، نادیده گرفتن فیلد |
| **Property** | بررسی فیلدهای شیء | `{ Age: 20 }` | اشیاء با Propertyهای متعدد |
| **Positional** | بررسی بر اساس موقعیت | `(10, 20)` | Recordها و کلاس‌های دارای Deconstruct |
| **Tuple** | بررسی چند مقدار همزمان | `(a, b) is (1, 2)` | تصمیم‌گیری ترکیبی |
| **List** | بررسی ساختار آرایه/لیست | `[1, 2, 3]` | اعتبارسنجی ترتیب عناصر |
| **Slice** | نادیده گرفتن بخشی از لیست | `[1, .., 3]` | بررسی شروع/پایان لیست |

---

## 42. Decision Guide
```text
آیا می‌خواهی مقدار خاصی (مثل 5 یا null) را بررسی کنی؟
    → Constant Pattern
آیا فقط نوع Object را بررسی می‌کنی؟
    → Type Pattern
آیا نوع Object را می‌خواهی و به مقدارش هم نیاز داری؟
    → Declaration Pattern
آیا بازه عددی یا مقایسه‌ای داری؟
    → Relational Pattern
آیا چند شرط ساده را می‌خواهی ترکیب کنی؟
    → Logical Pattern (and/or/not)
آیا Propertyهای داخلی Object برایت مهم هستند؟
    → Property Pattern
آیا چند متغیر مستقل را می‌خواهی همزمان بررسی کنی؟
    → Tuple Pattern
آیا ترتیب و ساختار عناصر یک Collection مهم است؟
    → List Pattern / Slice Pattern
آیا کلاس متد Deconstruct دارد و می‌خواهی بر اساس موقعیت چک کنی؟
    → Positional Pattern
```

---

## 43. مثال‌های واقعی

1. **Validation**:
   - *قدیمی*: `if (name == null || name.Length == 0)`
   - *جدید*: `if (name is null or { Length: 0 })`
2. **HTTP Result Mapping**:
   - *جدید*: `response.StatusCode is >= 200 and < 300`
3. **Collection Validation**:
   - *جدید*: `if (items is [var first, ..] and first.IsActive)`

*(به دلیل محدودیت فضا، 7 مثال دیگر در پروژه عملی و تمرین‌ها پوشش داده شده‌اند).*

---

## 44. تمرین‌ها

### Beginner
1. **Constant**: تابعی بنویسید که اگر ورودی `5` بود، "High" و در غیر این صورت "Low" برگرداند (با `is`).
2. **Type**: بررسی کنید آیا یک `object` از نوع `List<int>` است. اگر بله، تعداد عناصر آن را چاپ کنید.
3. **Null**: با استفاده از `is not null` بررسی کنید شیء تهی نیست.
4. **Switch**: یک `switch` statement بنویسید که روزهای هفته را به "Weekday" یا "Weekend" دسته‌بندی کند.

### Intermediate
5. **Relational**: نمره را با `switch expression` به A, B, C, F تبدیل کنید.
6. **Logical**: بررسی کنید آیا عدد بین 10 تا 20 است (با `and`).
7. **Property**: کلاس `Car` با Property `Speed`. اگر `Speed > 120` بود، "Fast" برگرداند.
8. **Tuple**: تابعی که `(month, isLeapYear)` را گرفته و تعداد روزهای ماه را برگرداند.
9. **Positional**: یک `record Point(int X, int Y)` بسازید و بررسی کنید آیا در مبدأ `(0,0)` است.

### Advanced
10. **List**: بررسی کنید آیا آرایه با `[1, 2]` شروع می‌شود.
11. **Slice**: بررسی کنید آیا لیست با عدد `9` تمام می‌شود (با `..`).
12. **Recursive**: بررسی کنید آیا `Order` دارای `Customer`ای است که `Country` آن "Iran" باشد.
13. **Exhaustive**: یک `switch expression` برای `enum` با 3 مقدار بنویسید که کامپایلر هیچ هشداری ندهد.
14. **Domain Modeling**: سیستم وضعیت `Payment` (Pending, Success, Failed) را با الگوها مدیریت کنید.
15. **Refactoring**: یک بلوک `if/else` تودرتوی 4 سطحی را به یک `switch expression` تمیز تبدیل کنید.

---

## 45. پروژه عملی: Order Processing Engine

```csharp
using System;
using System.Collections.Generic;

public record Customer(string Type); // Types: "VIP", "Regular"
public record Payment(string Status); // Status: "Paid", "Pending", "Failed"
public record Order(string Status, Payment Payment, Customer Customer, decimal Amount, List<string> Items);

public class OrderProcessor
{
    public string ProcessOrder(Order order) => (order.Status, order.Payment.Status, order.Customer.Type) switch
    {
        // 1. سفارش لغو شده
        ("Cancelled", _, _) => "Order is cancelled. No action.",

        // 2. پرداخت ناموفق
        (_, "Failed", _) => "Payment failed. Notify customer.",

        // 3. سفارش VIP با پرداخت تکمیل شده و مبلغ بالا
        ("Pending", "Paid", "VIP") when order.Amount > 1000 => "Priority processing for VIP.",

        // 4. سفارش عادی با پرداخت تکمیل شده
        ("Pending", "Paid", "Regular") => "Standard processing.",

        // 5. بررسی لیست آیتم‌ها (List Pattern)
        ("Pending", "Paid", _) when order.Items is ["SpecialItem", ..] => "Contains special item, manual review required.",

        // 6. حالت پیش‌فرض
        _ => "Unknown order state. Log for manual inspection."
    };
}
```
**تحلیل**: این کد به جای 4 سطح `if` تو در تو، تمام منطق تصمیم‌گیری را در یک `switch expression` خوانا، ایمن (Exhaustive) و قابل تست خلاصه کرده است.

---

## 46. جمع‌بندی نهایی

### Pattern Matching چیست؟
ابزاری برای بررسی ساختار، نوع و مقدار داده‌ها و استخراج ایمن بخش‌هایی از آن‌ها.

### مزایا
- کاهش کدهای Boilerplate (مانند Cast).
- افزایش خوانایی و Declarative بودن کد.
- ایمنی بیشتر در برابر `null` و خطاهای منطقی.

### معایب
- منحنی یادگیری برای الگوهای پیچیده (مثل Slice و Recursive).
- امکان کاهش خوانایی در صورت استفاده افراطی (Over-engineering).

### چه زمانی استفاده کنیم؟
- نگاشت (Mapping) مقادیر (با Switch Expression).
- بررسی نوع و استخراج همزمان (با Declaration Pattern).
- اعتبارسنجی ساختار داده‌ها (با List/Property Pattern).

### چه زمانی استفاده نکنیم؟
- وقتی یک `if (a == b)` ساده کاملاً گویاست.
- وقتی الگوها بیش از 3 سطح تو در تو می‌شوند (در این صورت کد را بشکنید).

### Cheat Sheet
| الگو | سینتکس |
| :--- | :--- |
| **Constant** | `is 5`, `is null` |
| **Type** | `is string` |
| **Declaration** | `is string s` |
| **Relational** | `is >= 10` |
| **Logical** | `is > 0 and < 10`, `is not null` |
| **Property** | `is { Age: 20 }` |
| **Tuple** | `is (1, 2)` |
| **List** | `is [1, 2, 3]` |
| **Slice** | `is [1, .., 3]` |

---

## منابع
1. **Microsoft Learn: Pattern matching**  
   - *سازمان*: Microsoft  
   - *توضیح*: مرجع رسمی و به‌روز تمامی قابلیت‌های Pattern Matching در C#.  
   - *لینک*: [docs.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)

2. **C# Language Reference: Patterns**  
   - *سازمان*: Microsoft  
   - *توضیح*: مستندات دقیق Syntax و قوانین زبانی برای هر الگو.  
   - *لینک*: [docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns)

3. **C# Language Design Notes (GitHub)**  
   - *سازمان*: dotnet/csharplang  
   - *توضیح*: یادداشت‌های طراحی زبان که دلیل اضافه شدن هر قابلیت (مثل List Patterns در C# 11) را توضیح می‌دهد.  
   - *لینک*: [github.com/dotnet/csharplang](https://github.com/dotnet/csharplang)

4. **SharpLab**  
   - *سازمان*: SharpLab.io  
   - *توضیح*: ابزار آنلاین برای مشاهده کد IL و Lowering شده‌ی الگوهای C#.  
   - *لینک*: [sharplab.io](https://sharplab.io)

---
*این سند بر اساس استانداردهای C# تا نسخه 13 و مستندات رسمی Microsoft تدوین شده است. برای به‌روزرسانی‌های آینده، بخش "تکامل Pattern Matching" را بررسی کنید.*