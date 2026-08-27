# راهنمای جامع Tuple و Equality در C# (از مقدماتی تا پیشرفته)

به مستند آموزشی مبحث **Tuple و Equality** خوش آمدید. این مقاله برای قرارگیری در Repository آموزشی طراحی شده و تمام زوایای مقایسه و برابری در Tupleهای سی‌شارپ را پوشش می‌دهد.

---

## 📑 فهرست مطالب
- [مقدمه](#مقدمه)
- [Equality چیست؟](#equality-چیست)
- [تفاوت Reference Equality و Structural Equality](#تفاوت-reference-equality-و-structural-equality)
- [Structural Equality چیست؟](#structural-equality-چیست)
- [آشنایی با Tuple در C# (Tuple در برابر ValueTuple)](#آشنایی-با-tuple-در-c)
- [مقایسه Tupleها بر اساس تمام عناصر](#مقایسه-tupleها-بر-اساس-تمام-عناصر)
  - [استفاده از متد Equals](#استفاده-از-متد-equals)
  - [استفاده از عملگرهای == و !=](#استفاده-از-عملگرهای--و-)
- [Tupleهای نام‌گذاری‌شده (Named Tuples) و Equality](#tupleهای-نام‌گذاری‌شده-named-tuples-و-equality)
  - [آیا نام اعضا روی Equality تأثیر دارد؟](#آیا-نام-اعضا-روی-equality-تأثیر-دارد)
- [Equality در Tupleهای تو در تو (Nested Tuples)](#equality-در-tupleهای-تو-در-تو-nested-tuples)
- [جدول مقایسه جامع](#جدول-مقایسه-جامع)
- [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
- [جمع‌بندی](#جمع‌بندی)
- [منابع رسمی](#منابع-رسمی)

---

## مقدمه
در برنامه‌نویسی، بررسی برابری دو شیء یکی از پرکاربردترین عملیات‌هاست. در C#، با معرفی `ValueTuple` در نسخه 7.0، نحوه برخورد با مقایسه داده‌ها متحول شد. در این مقاله بررسی می‌کنیم که چگونه می‌توان دو Tuple را مقایسه کرد، تفاوت `Equals` و `==` چیست و نام‌گذاری اعضا چه تأثیری در برابری دارد.

---

## Equality چیست؟
در برنامه‌نویسی شی‌گرا، **Equality** (برابری) به معنای بررسی این موضوع است که آیا دو مرجع (Reference) یا دو مقدار (Value) با هم «یکسان» هستند یا خیر. در C# ما دو نوع اصلی برابری داریم:
1. **Reference Equality (برابری مرجعی):** آیا دو متغیر به یک مکان واحد در حافظه (Heap) اشاره می‌کنند؟
2. **Value / Structural Equality (برابری مقداری/ساختاری):** آیا محتویات و داده‌های درون دو شیء دقیقاً یکسان هستند؟

---

## تفاوت Reference Equality و Structural Equality

برای درک بهتر، این مثال را در نظر بگیرید:

```csharp
string a = "Hello";
string b = "Hello";
string c = a;

// Reference Equality
Console.WriteLine(ReferenceEquals(a, c)); // True (هر دو به یک مکان حافظه اشاره می‌کنند)
Console.WriteLine(ReferenceEquals(a, b)); // ممکن است True یا False باشد (بستگی به String Interning دارد)

// Structural Equality
Console.WriteLine(a == b); // True (محتوای هر دو رشته "Hello" است)
```

---

## Structural Equality چیست؟
**Structural Equality** یعنی دو شیء مستقل از هم، اگر ساختار و مقادیر درونشان دقیقاً یکسان باشد، «برابر» در نظر گرفته می‌شوند. 
مثلاً دو `Point` با مختصات `(10, 20)`، حتی اگر در دو جای مختلف حافظه ساخته شده باشند، از نظر Structural Equality برابرند.
**نکته کلیدی:** `ValueTuple`ها در C# به‌صورت پیش‌فرض از **Structural Equality** پشتیبانی می‌کنند.

---

## آشنایی با Tuple در C# (Tuple در برابر ValueTuple)
قبل از بررسی Equality، باید بدانیم در C# دو نوع Tuple داریم:
1. **`System.Tuple` (معرفی شده در .NET 4.0):** یک **کلاس** (Reference Type) است.
2. **`System.ValueTuple` (معرفی شده در C# 7.0):** یک **ساختار** (Value Type / Struct) است و با سینتکس `(,)` شناخته می‌شود.

*در ادامه مقاله، تمرکز اصلی ما روی `ValueTuple` (نسخه مدرن و پرکاربرد) است، اما تفاوت‌های آن‌ها در بخش جدول مقایسه بررسی می‌شود.*

---

## مقایسه Tupleها بر اساس تمام عناصر
وقتی دو `ValueTuple` را مقایسه می‌کنیم، CLR به‌صورت خودکار **تمام عناصر** آن‌ها را به ترتیب بررسی می‌کند. اگر تمام عناصر متناظر برابر باشند، کل Tupleها برابر هستند.

### استفاده از متد Equals
متد `Equals()` در `ValueTuple` به‌گونه‌ای Override شده است که برابری ساختاری (Structural) را بررسی کند.

```csharp
var tuple1 = (1, "Ali");
var tuple2 = (1, "Ali");
var tuple3 = (2, "Ali");

Console.WriteLine(tuple1.Equals(tuple2)); // خروجی: True (چون هر دو عنصر برابرند)
Console.WriteLine(tuple1.Equals(tuple3)); // خروجی: False (چون عنصر اول متفاوت است)
```

### استفاده از عملگرهای `==` و `!=`
در `ValueTuple`، عملگرهای `==` و `!=` به‌صورت پیش‌فرض **Overload** شده‌اند و دقیقاً همان کار `Equals` را انجام می‌دهند (مقایسه ساختاری).

```csharp
var t1 = (10, 20.5, true);
var t2 = (10, 20.5, true);

Console.WriteLine(t1 == t2); // خروجی: True
Console.WriteLine(t1 != t2); // خروجی: False
```
**توضیح نتیجه:** چون تمام عناصر (یک `int`، یک `double` و یک `bool`) در هر دو Tuple مقدار یکسانی دارند، عملگر `==` مقدار `True` را برمی‌گرداند.

---

## Tupleهای نام‌گذاری‌شده (Named Tuples) و Equality
شما می‌توانید برای عناصر Tuple نام انتخاب کنید تا کد خواناتر شود:

```csharp
var person1 = (Id: 1, Name: "Sara");
var person2 = (Code: 1, FullName: "Sara");
```

### آیا نام اعضا روی Equality تأثیر دارد؟
**خیر، مطلقاً!** 
نام‌گذاری اعضا فقط یک ویژگی در زمان **کامپایل (Compile-Time)** است. کامپایل C# نام‌ها را در IL (کد نهایی) ذخیره نمی‌کند و آن‌ها را به `Item1` و `Item2` تبدیل می‌کند. نام‌ها فقط در Attributeای به نام `TupleElementNames` ذخیره می‌شوند تا IDEها آن‌ها را نمایش دهند.

```csharp
var t1 = (Id: 1, Name: "Ali");
var t2 = (Code: 1, FullName: "Ali");

Console.WriteLine(t1 == t2); // خروجی: True
Console.WriteLine(t1.Equals(t2)); // خروجی: True
```
**توضیح نتیجه:** با وجود اینکه نام‌گذاری‌ها کاملاً متفاوت است (`Id` در برابر `Code`)، چون **نوع داده** و **مقدار** عناصر در جایگاه‌های متناظر یکسان است، این دو Tuple از نظر ساختاری برابرند.

---

## Equality در Tupleهای تو در تو (Nested Tuples)
یکی از قابلیت‌های زیبای `ValueTuple`، پشتیبانی از برابری ساختاری به‌صورت **بازگشتی (Recursive)** در Tupleهای تو در تو است.

```csharp
var nested1 = ((1, 2), "A");
var nested2 = ((1, 2), "A");
var nested3 = ((1, 3), "A");

Console.WriteLine(nested1 == nested2); // خروجی: True
Console.WriteLine(nested1 == nested3); // خروجی: False
```
**توضیح نتیجه:** 
- در مقایسه اول، کامپایلر ابتدا Tuple داخلی `(1, 2)` را با `(1, 2)` مقایسه می‌کند (که برابرند)، سپس `"A"` را با `"A"` مقایسه می‌کند. نتیجه `True` است.
- در مقایسه دوم، Tuple داخلی `(1, 2)` با `(1, 3)` مقایسه شده و چون برابر نیستند، کل مقایسه `False` می‌شود.

---

## جدول مقایسه جامع

درک تفاوت `System.Tuple` و `System.ValueTuple` در مبحث Equality حیاتی است:

| ویژگی | `System.Tuple` (قدیمی) | `System.ValueTuple` (جدید - C# 7+) |
| :--- | :--- | :--- |
| **نوع داده** | کلاس (Reference Type) | ساختار (Value Type / Struct) |
| **سینتکس** | `Tuple.Create(1, "A")` یا `new Tuple<int, string>(1, "A")` | `(1, "A")` |
| **رفتار عملگر `==`** | **مقایسه Reference** (آدرس حافظه) ⚠️ | **مقایسه Structural** (مقادیر) ✅ |
| **رفتار متد `Equals()`** | مقایسه Structural | مقایسه Structural |
| **نام‌گذاری اعضا** | ندارد | دارد (فقط در زمان کامپایل) |
| **تخصیص حافظه** | روی Heap (نیاز به Garbage Collection) | روی Stack (در اکثر مواقع) |

⚠️ **هشدار مهم:** اگر از `System.Tuple` (نسخه قدیمی) استفاده کنید، عملگر `==` فقط چک می‌کند که آیا دو متغیر به یک آبجکت در حافظه اشاره می‌کنند یا خیر، **نه اینکه مقادیرشان یکی باشد!**

```csharp
// رفتار عجیب System.Tuple
var oldTuple1 = Tuple.Create(1, 2);
var oldTuple2 = Tuple.Create(1, 2);

Console.WriteLine(oldTuple1 == oldTuple2);      // خروجی: False (چون Reference متفاوت است)
Console.WriteLine(oldTuple1.Equals(oldTuple2)); // خروجی: True (چون Structural است)
```

---

## نکات مهم و اشتباهات رایج

### 1. اشتباه رایج: استفاده از کلاس‌های سفارشی بدون Override کردن Equals
اگر یک شیء سفارشی (Custom Class) را داخل `ValueTuple` قرار دهید، برای اینکه برابری ساختاری درست کار کند، آن کلاس **باید** متد `Equals` و `GetHashCode` را Override کرده باشد.

```csharp
class Person { public string Name { get; set; } }

var p1 = new Person { Name = "Ali" };
var p2 = new Person { Name = "Ali" };

var t1 = (1, p1);
var t2 = (1, p2);

Console.WriteLine(t1 == t2); // خروجی: False ❌
```
**توضیح:** چون کلاس `Person` متد `Equals` را Override نکرده است، `ValueTuple` برای مقایسه `p1` و `p2` از Reference Equality استفاده می‌کند و نتیجه `False` می‌شود.
**راه‌حل:** استفاده از `record` به جای `class`، یا پیاده‌سازی `IEquatable<T>`.

### 2. مقایسه اعداد اعشاری (Floating-Point)
مقایسه `double` یا `float` داخل Tupleها ممکن است به دلیل ماهیت این اعداد در IEEE 754 با خطا مواجه شود.
```csharp
var t1 = (0.1 + 0.2);
var t2 = (0.3);
Console.WriteLine(t1 == t2); // ممکن است False باشد!
```
**راه‌حل:** برای اعداد دقیق مالی از `decimal` استفاده کنید.

### 3. مقدار null در Tupleها
اگر یکی از عناصر Tuple از نوع Reference (مثل `string`) و مقدار آن `null` باشد، `ValueTuple` به‌درستی `null` بودن را مدیریت می‌کند:
```csharp
var t1 = (1, null);
var t2 = (1, null);
Console.WriteLine(t1 == t2); // خروجی: True
```

---

## جمع‌بندی
1. **`ValueTuple`ها** (سینتکس `(,)`) به‌صورت پیش‌فرض از **Structural Equality** پشتیبانی می‌کنند.
2. در `ValueTuple`، هم متد `Equals()` و هم عملگر `==` **مقادیر** را مقایسه می‌کنند.
3. **نام اعضا** در Named Tuples هیچ تأثیری در Equality ندارد و فقط برای خوانایی کد در زمان کامپایل است.
4. مقایسه به‌صورت **بازگشتی** انجام می‌شود و Tupleهای تو در تو به‌درستی پشتیبانی می‌شوند.
5. مراقب `System.Tuple` (نسخه قدیمی) باشید؛ در آن `==` فقط Reference را مقایسه می‌کند.
6. اگر داخل Tuple از کلاس‌های سفارشی استفاده می‌کنید، مطمئن شوید که `Equals` را به‌درستی پیاده‌سازی کرده‌اند (یا از `record` استفاده کنید).

---

## منابع رسمی
برای مطالعه بیشتر و عمیق‌تر، منابع زیر پیشنهاد می‌شوند:
- [Microsoft Docs: Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
- [Microsoft Docs: Equality comparisons (C#)]https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons)
- [Source Code: System.ValueTuple in .NET Runtime](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/ValueTuple.cs)

---
*این مقاله برای Repository آموزشی C# تهیه شده است. در صورت داشتن سوال یا نیاز به مثال‌های بیشتر، می‌توانید Issue ثبت کنید.*