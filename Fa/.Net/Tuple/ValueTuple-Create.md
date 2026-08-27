# 📘 متد `ValueTuple.Create` در C# — راهنمای جامع

> **مخاطب هدف:** برنامه‌نویسان C# که می‌خواهند درک عمیق‌تری از `ValueTuple` و روش‌های ساخت آن داشته باشند.
> **سطح:** متوسط
> **زمان مطالعه:** حدود ۱۵ دقیقه

---

## 📑 فهرست مطالب

1. [مقدمه](#مقدمه)
2. [`ValueTuple.Create` چیست؟](#valuetuplecreate-چیست)
3. [Static Method چیست؟](#static-method-چیست)
4. [Syntax](#syntax)
5. [Overloadهای `Create`](#overloadهای-create)
6. [ایجاد Tuple با `Create`](#ایجاد-tuple-با-create)
7. [تفاوت `Create` با Syntax `(1, 2)`](#تفاوت-create-با-syntax-1-2)
8. [Type استنتاج‌شده (Type Inference)](#type-استنتاجشده-type-inference)
9. [مثال‌های مختلف](#مثالهای-مختلف)
10. [تعداد پارامترهای قابل قبول](#تعداد-پارامترهای-قابل-قبول)
11. [استفاده در Genericها](#استفاده-در-genericها)
12. [چه زمانی استفاده از `Create` منطقی است؟](#چه-زمانی-استفاده-از-create-منطقی-است)
13. [نکات مهم](#نکات-مهم)
14. [اشتباهات رایج](#اشتباهات-رایج)
15. [جمع‌بندی](#جمع‌بندی)
16. [منابع رسمی](#منابع-رسمی)

---

## مقدمه

از نسخه **C# 7.0** به بعد، `ValueTuple` به‌عنوان جایگزینی سبک‌تر و سریع‌تر برای `System.Tuple` معرفی شد. برای ساخت یک `ValueTuple` معمولاً از syntax کوتاه `(1, "Ali")` استفاده می‌کنیم، اما در پس‌زمینه، کامپایلر در بسیاری از موارد این syntax را به فراخوانی `ValueTuple.Create(...)` ترجمه می‌کند. در این مقاله به بررسی دقیق این متد، کاربردها و تفاوت‌های آن با tuple literal می‌پردازیم.

---

## `ValueTuple.Create` چیست؟

`ValueTuple.Create` یک **متد ایستا (Static)** در ساختار `System.ValueTuple` است که وظیفه‌ی ساخت یک نمونه‌ی جدید از `ValueTuple<T1>`, `ValueTuple<T1,T2>`, ... را بر عهده دارد.

این متد در واقع **Factory Method** برای ساخت tupleهاست و کامپایلر C# هنگام استفاده از tuple literal، اغلب آن را به فراخوانی این متد تبدیل می‌کند.

```csharp
// این دو خط از نظر کامپایلر معادل هستند:
var t1 = (1, "Ali");
var t2 = ValueTuple.Create(1, "Ali");
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## Static Method چیست؟

قبل از ادامه، بهتر است مفهوم **Static Method** را مرور کنیم:

- یک متد ایستا به **نمونه‌ی (instance)** کلاس نیاز ندارد.
- مستقیماً از طریق نام نوع صدا زده می‌شود: `ValueTuple.Create(...)`.
- معمولاً برای ساخت اشیا (Factory) یا عملیات‌های عمومی استفاده می‌شود.

```csharp
// ❌ اشتباه - نمی‌توان روی نمونه صدا زد
var t = ValueTuple.Create(1, 2);
// t.Create(3, 4); // کامپایل نمی‌شود

// ✅ درست - از طریق نام نوع
var t2 = ValueTuple.Create(3, 4);
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## Syntax

سینتکس کلی `ValueTuple.Create` به این صورت است:

```csharp
public static ValueTuple<T1, T2, ..., TN> Create<T1, T2, ..., TN>(T1 item1, T2 item2, ..., TN itemN)
```

یک نمونه‌ی ساده:

```csharp
var tuple = ValueTuple.Create<int, string>(42, "Hello");
// یا با type inference:
var tuple2 = ValueTuple.Create(42, "Hello");
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## Overloadهای `Create`

کلاس `ValueTuple` دارای **۹ overload** برای متد `Create` است که از ۰ تا ۸ پارامتر را می‌پذیرد:

| Overload | تعداد پارامتر | نوع خروجی |
|----------|---------------|-----------|
| `Create()` | 0 | `ValueTuple` |
| `Create<T1>(T1)` | 1 | `ValueTuple<T1>` |
| `Create<T1,T2>(T1,T2)` | 2 | `ValueTuple<T1,T2>` |
| `Create<T1,T2,T3>(...)` | 3 | `ValueTuple<T1,T2,T3>` |
| `Create<T1,...,T4>(...)` | 4 | `ValueTuple<T1,...,T4>` |
| `Create<T1,...,T5>(...)` | 5 | `ValueTuple<T1,...,T5>` |
| `Create<T1,...,T6>(...)` | 6 | `ValueTuple<T1,...,T6>` |
| `Create<T1,...,T7>(...)` | 7 | `ValueTuple<T1,...,T7>` |
| `Create<T1,...,T8>(...)` | 8 | `ValueTuple<T1,...,T8>` |

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## ایجاد Tuple با `Create`

### مثال ۱: Tuple بدون مقدار

```csharp
var empty = ValueTuple.Create();
Console.WriteLine(empty); // ()
```

### مثال ۲: Tuple با یک مقدار

```csharp
var single = ValueTuple.Create(42);
Console.WriteLine(single.Item1); // 42
```

### مثال ۳: Tuple با چند مقدار

```csharp
var person = ValueTuple.Create("Ali", 30, true);
Console.WriteLine($"{person.Item1}, {person.Item2}, {person.Item3}");
// Ali, 30, True
```

### مثال ۴: با نام‌گذاری فیلدها (از C# 7.1)

```csharp
var person = ValueTuple.Create("Ali", 30);
// نام‌گذاری در زمان اعلان متغیر:
(string Name, int Age) p = ValueTuple.Create("Ali", 30);
Console.WriteLine(p.Name); // Ali
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## تفاوت `Create` با Syntax `(1, 2)`

هر دو روش در نهایت یک `ValueTuple` تولید می‌کنند، اما تفاوت‌های ظریفی دارند:

| ویژگی | `ValueTuple.Create(...)` | `(1, 2)` Literal |
|-------|--------------------------|------------------|
| خوانایی | کمتر | بیشتر ✅ |
| نیاز به نوشتن نام نوع | دارد | ندارد ✅ |
| پشتیبانی از نام فیلد در محل ساخت | ندارد ✅ (مستقیم) | دارد ✅ |
| استفاده در delegate / reflection | الزامی | غیرممکن |
| تولید IL | یکسان | یکسان |

```csharp
// روش مدرن و پیشنهادی:
var t1 = (1, 2);
var t2 = (Name: "Ali", Age: 30);

// روش قدیمی‌تر:
var t3 = ValueTuple.Create(1, 2);
```

> 💡 **نکته:** کامپایلر C#، tuple literal را در IL به فراخوانی `ValueTuple.Create` تبدیل می‌کند؛ پس از نظر عملکرد هیچ تفاوتی ندارند.

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## Type استنتاج‌شده (Type Inference)

یکی از مزایای بزرگ `ValueTuple.Create` این است که کامپایلر می‌تواند **نوع آرگومان‌ها را به‌صورت خودکار تشخیص دهد**:

```csharp
// کامپایلر نوع int و string را تشخیص می‌دهد
var t = ValueTuple.Create(10, "Hello");
// معادل با:
ValueTuple<int, string> t2 = ValueTuple.Create<int, string>(10, "Hello");
```

### استنتاج در Genericها

```csharp
TResult Process<TInput, TResult>(TInput input, Func<TInput, TResult> mapper)
{
    return mapper(input);
}

// نوع TInput و TResult به‌صورت خودکار استنتاج می‌شود
var result = Process(5, x => ValueTuple.Create(x, x * x));
// result: ValueTuple<int, int>
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## مثال‌های مختلف

### مثال ۱: بازگرداندن چند مقدار از متد

```csharp
public static ValueTuple<int, int, int> GetMinMaxAvg(int[] numbers)
{
    return ValueTuple.Create(
        numbers.Min(),
        numbers.Max(),
        (int)numbers.Average()
    );
}

var (min, max, avg) = GetMinMaxAvg(new[] { 1, 5, 3, 9, 2 });
Console.WriteLine($"Min: {min}, Max: {max}, Avg: {avg}");
```

### مثال ۲: استفاده در LINQ

```csharp
var users = new[] { "Ali", "Sara", "Reza" };

var indexed = users.Select((name, index) => ValueTuple.Create(index, name));

foreach (var (idx, name) in indexed)
{
    Console.WriteLine($"{idx}: {name}");
}
```

### مثال ۳: Tuple تودرتو (Nested)

```csharp
var nested = ValueTuple.Create(
    1,
    ValueTuple.Create("inner", true),
    3.14
);

Console.WriteLine(nested.Item2.Item1); // inner
```

### مثال ۴: استفاده به‌عنوان کلید دیکشنری

```csharp
var dict = new Dictionary<ValueTuple<int, int>, string>
{
    { ValueTuple.Create(0, 0), "Origin" },
    { ValueTuple.Create(1, 2), "Point A" }
};

Console.WriteLine(dict[ValueTuple.Create(0, 0)]); // Origin
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## تعداد پارامترهای قابل قبول

`ValueTuple.Create` حداکثر **۸ پارامتر** می‌پذیرد. برای tupleهای بزرگ‌تر باید از **تودرتو کردن (nesting)** استفاده کرد:

```csharp
// ✅ تا ۸ پارامتر مستقیم
var t8 = ValueTuple.Create(1, 2, 3, 4, 5, 6, 7, 8);

// ❌ این کامپایل نمی‌شود:
// var t9 = ValueTuple.Create(1, 2, 3, 4, 5, 6, 7, 8, 9);

// ✅ راه‌حل: استفاده از Rest (پارامتر نهم به‌عنوان tuple)
var t9 = ValueTuple.Create(1, 2, 3, 4, 5, 6, 7,
    ValueTuple.Create(8, 9));

Console.WriteLine(t9.Rest.Item2); // 9
```

> 💡 **نکته:** در tuple literal `(1,2,3,4,5,6,7,8,9)` کامپایلر به‌صورت خودکار این تودرتو کردن را انجام می‌دهد.

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## استفاده در Genericها

از آنجا که `Create` خودش یک متد generic است، در سناریوهای generic بسیار مفید است:

```csharp
public static ValueTuple<T, T> Duplicate<T>(T value)
{
    return ValueTuple.Create(value, value);
}

var intDup = Duplicate(5);          // ValueTuple<int, int>
var strDup = Duplicate("Hi");       // ValueTuple<string, string>

public static ValueTuple<TInput, TResult> Map<TInput, TResult>(
    TInput input, Func<TInput, TResult> transform)
{
    return ValueTuple.Create(input, transform(input));
}

var mapped = Map(10, x => x * x);
Console.WriteLine(mapped); // (10, 100)
```

### استفاده در Delegate

```csharp
Func<int, int, ValueTuple<int, int>> addAndMultiply =
    (a, b) => ValueTuple.Create(a + b, a * b);

var result = addAndMultiply(3, 4);
Console.WriteLine(result); // (7, 12)
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## چه زمانی استفاده از `Create` منطقی است؟

با وجود اینکه tuple literal `(a, b)` خواناتر است، در موارد زیر `Create` منطقی یا ضروری است:

### ✅ ۱. در کانتکست Generic که نوع مشخص نیست

```csharp
public ValueTuple<T1, T2> Wrap<T1, T2>(T1 a, T2 b)
    => ValueTuple.Create(a, b);
```

### ✅ ۲. هنگام کار با Delegate یا `Func<>`

```csharp
Func<int, ValueTuple<int, int>> f = x => ValueTuple.Create(x, x * 2);
```

### ✅ ۳. در Reflection یا Expression Tree

```csharp
var method = typeof(ValueTuple).GetMethod("Create", new[] { typeof(int), typeof(string) });
var tuple = method.Invoke(null, new object[] { 1, "A" });
```

### ✅ ۴. وقتی به نام متد به‌عنوان `MethodGroup` نیاز دارید

```csharp
var factory = new Func<int, int, ValueTuple<int, int>>(ValueTuple.Create);
```

### ❌ در سایر موارد، tuple literal ترجیح داده می‌شود:

```csharp
// ✅ بهتر
var p = ("Ali", 30);

// ❌ غیرضروری طولانی
var p = ValueTuple.Create("Ali", 30);
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## نکات مهم

- 🎯 `ValueTuple` یک **struct** است، یعنی روی **Stack** قرار می‌گیرد و allocation کمی دارد.
- 🎯 `Create` یک **Factory Method** است و نوع خروجی را بر اساس آرگومان‌ها تعیین می‌کند.
- 🎯 از C# 7.0 به بعد، tuple literal `(a, b)` به `ValueTuple.Create` ترجمه می‌شود.
- 🎯 برای بیش از ۸ عنصر، باید از **nested tuple** استفاده کنید (فیلد `Rest`).
- 🎯 `ValueTuple` با `Tuple` (کلاس قدیمی) متفاوت است؛ `Tuple` یک **class** (reference type) است.
- 🎯 فیلدهای `ValueTuple` به‌صورت پیش‌فرض `Item1`, `Item2`, ... نام‌گذاری می‌شوند.

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## اشتباهات رایج

### ❌ ۱. اشتباه گرفتن `ValueTuple` با `Tuple`

```csharp
// ❌ این Tuple قدیمی است (class - روی Heap)
var old = Tuple.Create(1, 2);

// ✅ این ValueTuple جدید است (struct - روی Stack)
var newT = ValueTuple.Create(1, 2);
```

### ❌ ۲. تلاش برای تغییر ناپذیری

```csharp
var t = ValueTuple.Create(1, 2);
t.Item1 = 10; // ✅ مجاز است - ValueTuple قابل تغییر (mutable) است
```

> ⚠️ برخلاف تصور، `ValueTuple` **mutable** است! اگر immutability می‌خواهید، از `record` استفاده کنید.

### ❌ ۳. استفاده از بیش از ۸ پارامتر بدون nesting

```csharp
// ❌ کامپایل نمی‌شود
var bad = ValueTuple.Create(1, 2, 3, 4, 5, 6, 7, 8, 9);

// ✅ درست
var good = ValueTuple.Create(1, 2, 3, 4, 5, 6, 7, ValueTuple.Create(8, 9));
```

### ❌ ۴. استفاده‌ی غیرضروری از `Create`

```csharp
// ❌ غیرضروری طولانی
var p = ValueTuple.Create("Ali", 30);

// ✅ کوتاه‌تر و خواناتر
var p = ("Ali", 30);
```

### ❌ ۵. فراموش کردن `Rest` در tupleهای بزرگ

```csharp
var big = ValueTuple.Create(1, 2, 3, 4, 5, 6, 7, ValueTuple.Create(8, 9));
Console.WriteLine(big.Item8);      // ❌ کامپایل نمی‌شود
Console.WriteLine(big.Rest.Item1); // ✅ درست - مقدار 8
```

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## جمع‌بندی

| موضوع | خلاصه |
|-------|-------|
| **چیست؟** | Factory method ایستا برای ساخت `ValueTuple` |
| **تعداد overload** | ۹ overload (از ۰ تا ۸ پارامتر) |
| **تفاوت با `(a,b)`** | هر دو یک IL تولید می‌کنند، اما literal خواناتر است |
| **Type Inference** | دارد — کامپایلر نوع را تشخیص می‌دهد |
| **حداکثر پارامتر** | ۸ (برای بیشتر از nested استفاده کنید) |
| **زمان استفاده** | در generic، delegate، reflection |
| **پیشنهاد** | در کد عادی از `(a, b)` استفاده کنید |

💡 **قانون طلایی:** در ۹۰٪ مواقع از tuple literal `(a, b)` استفاده کنید. `ValueTuple.Create` را فقط در سناریوهای خاص (generic، delegate، reflection) به کار ببرید.

> 🔗 بازگشت به [فهرست مطالب](#فهرست-مطالب)

---

## منابع رسمی

- 📚 [Microsoft Docs — ValueTuple Structure](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple)
- 📚 [Microsoft Docs — ValueTuple.Create Method](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple.create)
- 📚 [C# Tuples (Microsoft Docs)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
- 📚 [What's new in C# 7.0 — Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7#tuples)
- 📚 [Source Code — CoreFX ValueTuple](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/ValueTuple.cs)

---

> ✍️ **نویسنده:** این مقاله بخشی از یک Repository آموزشی C# است.
> 📅 **آخرین به‌روزرسانی:** August 2026
> 🏷️ **برچسب‌ها:** `#CSharp` `#ValueTuple` `#DotNet` `#Tutorial`

اگر این مقاله برایتان مفید بود، لطفاً ⭐ به Repository بدهید و برای توسعه‌دهندگان دیگر ارسال کنید! 🚀