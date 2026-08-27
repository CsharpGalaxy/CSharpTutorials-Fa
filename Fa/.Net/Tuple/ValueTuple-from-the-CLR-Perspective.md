# ساختار `ValueTuple` از دید CLR

> **مخاطب هدف:** توسعه‌دهندگان C# که می‌خواهند بدانند در پشت پرده‌ی سینتکس `(int, string)` دقیقاً چه چیزی در CLR ساخته می‌شود.
> **پیش‌نیاز:** آشنایی اولیه با مفاهیم Value Type / Reference Type و Generic در CLR.

---

## فهرست مطالب

1. [مقدمه](#مقدمه)
2. [ValueTuple چیست؟](#valuetuple-چیست)
3. [جایگاه در CLR و کتابخانه‌های پایه](#جایگاه-در-clr-و-کتابخانههای-پایه)
4. [چرا `struct` است؟](#چرا-struct-است)
5. [Generic Struct بودن](#generic-struct-بودن)
6. [Value Type بودن و رفتار در Heap/Stack](#value-type-بودن-و-رفتار-در-heapstack)
7. [آریته‌های مختلف `ValueTuple`](#آریتههای-مختلف-valuetuple)
   - 7.1 [`ValueTuple` (بدون پارامتر نوع)](#valuetuple-بدون-پارامتر-نوع)
   - 7.2 [`ValueTuple<T1, T2>` تا `ValueTuple<T1,...,T7>`](#valuetuplet1-t2-تا-valuetuplet1t7)
   - 7.3 [`ValueTuple<T1,...,T7, TRest>`](#valuetuplet1t7-trest)
8. [فیلدهای `Item1` تا `Item7`](#فیلدهای-item1-تا-item7)
9. [فیلد `Rest`](#فیلد-rest)
10. [تفاوت Syntax سطح C# با نمایش Runtime](#تفاوت-syntax-سطح-c-با-نمایش-runtime)
11. [رابطه‌ی Tuple Type و `ValueTuple`](#رابطهی-tuple-type-و-valuetuple)
12. [مثال IL](#مثال-il)
13. [نکات مهم](#نکات-مهم)
14. [جمع‌بندی](#جمع‌بندی)
15. [منابع](#منابع)

---

## مقدمه

از C# 7.0 به بعد، سینتکس `(int x, string y)` به‌عنوان یک Tuple Type در زبان معرفی شد. اما CLR هیچ نوعی به نام «Tuple» به‌صورت ذاتی ندارد؛ چیزی که در سطح CLR ساخته می‌شود، یک **Generic Struct** به نام `System.ValueTuple` است. درک این موضوع برای بهینه‌سازی عملکرد، تحلیل IL و درک عمیق رفتار برنامه ضروری است.

---

## ValueTuple چیست؟

`ValueTuple` یک خانواده از **ساختارهای Generic** در فضای نام `System` است که توسط CLR به‌عنوان یک نوع معمولی (نه یک نوع ویژه‌ی زبان) شناخته می‌شود. این ساختارها برای نگهداری‌ی تعداد ثابتی از فیلدها با انواع دلخواه طراحی شده‌اند.

از دید CLR، `(int, string)` هیچ تفاوت ساختاری با `KeyValuePair<int, string>` ندارد — هر دو یک `struct` با دو فیلد عمومی هستند. تفاوت تنها در **نام‌گذاری فیلدها** (`Item1`, `Item2`) و **آریته‌های از پیش تعریف‌شده** است.

---

## جایگاه در CLR و کتابخانه‌های پایه

`ValueTuple` بسته به‌ نسخه‌ی فریم‌ورک از یکی از منابع زیر تأمین می‌شود:

| فریم‌ورک | محل تعریف |
|---|---|
| .NET Framework 4.7+ | `mscorlib.dll` |
| .NET Core 2.0+ / .NET 5+ | `System.Private.CoreLib.dll` |
| .NET Standard 2.0 | `netstandard.dll` |
| پروژه‌های قدیمی‌تر | بسته‌ی NuGet به نام `System.ValueTuple` |

> **نکته:** نوع `ValueTuple` در CLI Specification (ECMA-335) به‌عنوان یک نوع ویژه تعریف **نشده** است. این صرفاً یک کتابخانه‌ی استاندارد است که کامپایلر C# به آن وابسته است.

---

## چرا `struct` است؟

تعریف رسمی در منبع باز .NET (dotnet/runtime) به این شکل است:

```csharp
public struct ValueTuple<T1, T2> : IEquatable<ValueTuple<T1, T2>>,
                                   IComparable,
                                   IComparable<ValueTuple<T1, T2>>,
                                   IStructuralEquatable,
                                   IStructuralComparable
{
    public T1 Item1;
    public T2 Item2;
    // ...
}
```

دلایل انتخاب `struct`:

- **اجتناب از Synchronization Overhead:** چون tupleها معمولاً immutable از دید منطقی (هرچند فیلدها public و قابل تغییرند) و کوتاه‌مدت هستند، allocation روی Heap و GC overhead نامطلوب است.
- **هم‌ترازی با سایر Value Typeها:** رفتار یکسان با `int`, `DateTime`, و سایر انواع مقدار.
- **کارایی در آرایه‌ها:** آرایه‌ای از `ValueTuple<int,int>` به‌صورت یک بلوک پیوسته از مقادیر ذخیره می‌شود، نه آرایه‌ای از referenceها.

---

## Generic Struct بودن

هر `ValueTuple` یک **constructed generic type** است. برای مثال `(int, string)` در CLR معادل است با:

```
System.ValueTuple`2<Int32, String>
```

که در آن `` `2 `` نشان‌دهنده‌ی آریته (تعداد پارامترهای نوع) است. این موضوع یعنی:

- `ValueTuple<int, string>` و `ValueTuple<string, int>` دو نوع **متمایز** در CLR هستند.
- متادیتای هر constructed type جداگانه در Assembly ذخیره می‌شود.
- JIT برای هر ترکیب منحصر‌به‌فرد از پارامترهای نوع، یک نسخه‌ی native جدا تولید می‌کند (به‌جز reference types که معمولاً code-sharing می‌شوند).

---

## Value Type بودن و رفتار در Heap/Stack

چون `ValueTuple` یک struct است:

- وقتی به‌عنوان متغیر محلی اعلام شود → روی **Stack** (یا دقیق‌تر: در stack frame یا register، بسته به JIT).
- وقتی فیلدی از یک کلاس باشد → **درون Heap**، به‌عنوان بخشی از آبجکت والد (نه به‌عنوان یک reference جدا).
- وقتی در آرایه باشد → **درون Heap**، به‌صورت پیوسته.
- هنگام Boxing → یک کپی روی Heap ساخته می‌شود و reference به آن برگردانده می‌شود.

> **هشدار:** اگر `ValueTuple` را به‌عنوان `object` یا یکی از اینترفیس‌هایش (`IEquatable<>`, `IComparable` و ...) پاس دهید، **Boxing** رخ می‌دهد و مزیت Value Type از بین می‌رود.

---

## آریته‌های مختلف `ValueTuple`

خانواده‌ی `ValueTuple` شامل ۹ تعریف مجزا است:

### `ValueTuple` (بدون پارامتر نوع)

```csharp
public struct ValueTuple : IComparable, IStructuralEquatable, IStructuralComparable, ITuple
```

این نوع معادل tuple صفر عضوی `()` است. در عمل تنها یک واحد (unit) خالی است.

### `ValueTuple<T1>` تا `ValueTuple<T1,...,T7>`

هفت ساختار با آریته‌ی ۱ تا ۷:

```csharp
public struct ValueTuple<T1>
public struct ValueTuple<T1, T2>
public struct ValueTuple<T1, T2, T3>
// ...
public struct ValueTuple<T1, T2, T3, T4, T5, T6, T7>
```

هر کدام به‌ترتیب فیلدهایی با نام `Item1`, `Item2`, ... تا `ItemN` دارند.

### `ValueTuple<T1,...,T7, TRest>`

این ساختار کلید پشتیبانی از tupleهای با بیش از ۷ عضو است:

```csharp
public struct ValueTuple<T1, T2, T3, T4, T5, T6, T7, TRest>
    where TRest : struct
```

وقتی شما یک tuple با ۸ یا بیشتر عضو تعریف می‌کنید، کامپایلر C# آن را به‌صورت **تودرتو (nested)** بازنویسی می‌کند. برای مثال:

```csharp
(int a, int b, int c, int d, int e, int f, int g, int h, int i) t = (1,2,3,4,5,6,7,8,9);
```

در CLR به این شکل بازنمایی می‌شود:

```
ValueTuple<Int32, Int32, Int32, Int32, Int32, Int32, Int32,
           ValueTuple<Int32, Int32>>
```

یعنی ۷ عضو اول در فیلدهای `Item1` تا `Item7` قرار می‌گیرند و بقیه درون فیلد `Rest` که خودش یک `ValueTuple` است. این تودرتویی می‌تواند برای tupleهای بسیار بزرگ ادامه یابد.

> **قید مهم:** `TRest` باید خودش یک `struct` باشد (معمولاً یک `ValueTuple` دیگر). اگر در runtime نوع `TRest` یک `ValueTuple` نباشد، constructor یک `ArgumentException` پرتاب می‌کند.

---

## فیلدهای `Item1` تا `Item7`

تمام فیلدهای `ItemN`:

- **Public** هستند.
- **Field** هستند (نه Property).
- **قابل تغییر (mutable)** هستند.
- نامشان در metadata دقیقاً `Item1`, `Item2`, ... است.

```csharp
public T1 Item1;
public T2 Item2;
// ...
```

این یعنی شما می‌توانید مستقیماً بنویسید:

```csharp
var t = (1, "a");
t.Item1 = 42;   // مجاز است
```

> **نکته:** نام‌های سفارشی مثل `x` و `y` که در سینتکس C# می‌نویسید `(int x, string y)`، در سطح CLR **وجود ندارند**. این نام‌ها صرفاً در metadata به‌عنوان attribute ذخیره می‌شوند (در بخش [TupleElementNamesAttribute](#رابطهی-tuple-type-و-valuetuple) توضیح داده شده است).

---

## فیلد `Rest`

فیلد `Rest` **فقط** در `ValueTuple<T1,...,T7, TRest>` وجود دارد:

```csharp
public TRest Rest;
```

این فیلد:

- از نوع `TRest` است که باید خود یک `struct` (معمولاً `ValueTuple`) باشد.
- برای دسترسی به اعضای بعد از هفتم استفاده می‌شود.
- در سینتکس C# شما مستقیماً به `Rest` دسترسی ندارید؛ کامپایلر آن را به `Item8`, `Item9`, ... ترجمه می‌کند.

برای مثال برای دسترسی به `Item8` در یک tuple 9 عضوی، IL معادل این را تولید می‌کند:

```
ldloca.s  t
ldfld     ValueTuple<int,int> System.ValueTuple<...>::Rest
ldfld     int32 System.ValueTuple<int,int>::Item1
```

---

## تفاوت Syntax سطح C# با نمایش Runtime

| سینتکس C# | نمایش در CLR |
|---|---|
| `()` | `System.ValueTuple` |
| `(int x)` | `System.ValueTuple<Int32>` |
| `(int x, string y)` | `System.ValueTuple<Int32, String>` |
| `(int, int, ..., int)` (۸ عضو) | `ValueTuple<int,...,int, ValueTuple<int>>` |
| `(int x, string y)` | فیلدها: `Item1: int`, `Item2: string` — نام‌های `x`, `y` در attribute ذخیره می‌شوند |

**نکات کلیدی:**

1. کامپایلر C# سینتکس `(T1 x1, T2 x2)` را به `ValueTuple<T1, T2>` بازنویسی می‌کند.
2. نام‌های سفارشی (`x1`, `x2`) در IL به‌عنوان نام فیلد ظاهر **نمی‌شوند**.
3. این نام‌ها در یک attribute به نام `TupleElementNamesAttribute` ذخیره می‌شوند.
4. Reflection معمولی (مثل `typeof(T).GetFields()`) فقط `Item1`, `Item2` را برمی‌گرداند.

---

## رابطه‌ی Tuple Type و `ValueTuple`

در مستندات مایکروسافت دو مفهوم متمایز وجود دارد:

- **Tuple Type (در C#):** یک مفهوم سطح زبان که شامل نام‌های سفارشی برای اعضا است.
- **Underlying Type (در CLR):** همیشه یکی از انواع `System.ValueTuple<...>`.

برای حفظ نام‌های سفارشی، کامپایلر attribute زیر را تولید می‌کند:

```csharp
[System.Runtime.CompilerServices.TupleElementNames(new string[] { "x", "y" })]
public System.ValueTuple<int, string> MyMethod() { ... }
```

این attribute روی:

- نوع بازگشتی متد
- پارامترهای متد
- فیلدها و propertyها

اعمال می‌شود. در runtime، با خواندن این attribute می‌توان نام‌های سفارشی را بازیابی کرد:

```csharp
var attrs = method.ReturnTypeCustomAttributes
    .GetCustomAttributes(typeof(TupleElementNamesAttribute), false);
```

> **هشدار:** `TupleElementNamesAttribute` فقط روی **reference typeها** (مثل `ValueTuple` که در field/parameter به‌صورت boxed یا به‌عنوان نوع property ظاهر می‌شود) قابل اعمال است. روی متغیرهای محلی (local variables) این attribute ذخیره نمی‌شود، چون local variable metadata نام‌ها را به شکل دیگری (از طریق debug symbolها یا PDB) حفظ می‌کند.

---

## مثال IL

کد C#:

```csharp
public (int x, string y) GetTuple()
{
    return (1, "hello");
}
```

IL تولید‌شده (تقریبی):

```il
.method public hidebysig 
    instance valuetype [System.Runtime]System.ValueTuple`2<int32, string> 
    GetTuple() cil managed 
{
    .custom instance void [System.Runtime]
        System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[]) 
        = ( 01 00 02 00 00 00 01 78 01 79 00 00 )   // "x", "y"
    
    .maxstack 3
    .locals init (valuetype [System.Runtime]System.ValueTuple`2<int32, string> V_0)
    
    // ساخت ValueTuple با مقدار (1, "hello")
    ldloca.s   V_0
    ldc.i4.1
    ldstr      "hello"
    call       instance void valuetype [System.Runtime]System.ValueTuple`2<int32, string>::.ctor(!0, !1)
    
    ldloc.0
    ret
}
```

**نکات قابل مشاهده در IL:**

1. نوع بازگشتی دقیقاً `ValueTuple`2<int32, string>` است.
2. نام‌های `x` و `y` در constructor attribute ذخیره شده‌اند (`01 78` = طول 1 + کاراکتر 'x'، `01 79` = طول 1 + کاراکتر 'y').
3. مقداردهی با فراخوانی constructor ساختار انجام می‌شود (نه newobj روی heap).
4. `ldloca.s` نشان می‌دهد که ساختار روی stack (در local variable) قرار دارد، نه heap.

---

## نکات مهم

- **تغییرپذیری فیلدها:** فیلدهای `ItemN` public و قابل تغییرند. اگر tuple را readonly می‌خواهید، باید از `readonly` modifier در C# 7.4+ استفاده کنید: `readonly (int x, int y) t = ...`.
- **تودرتویی نامرئی:** برای tupleهای بیش از ۷ عضو، ساختار تودرتو ایجاد می‌شود که در performance و boxing تأثیر دارد.
- **Boxing:** هر بار پاس‌دادن tuple به پارامتر `object` یا اینترفیس، boxing ایجاد می‌کند.
- **مقایسه و تساوی:** `ValueTuple` پیاده‌سازی عمیق (deep) `Equals` و `GetHashCode` دارد که همه‌ی اعضا را بررسی می‌کند.
- **ITuple:** اینترفیس `System.Runtime.CompilerServices.ITuple` (اضافه‌شده در .NET Core 2.0) دسترسی غیرجنریک به اعضا از طریق ایندکس و `Length` فراهم می‌کند.
- **سازگاری با `System.Tuple`:** نوع قدیمی‌تر `System.Tuple` (از .NET 4.0) یک **reference type** است و با `ValueTuple` متفاوت است. تبدیل صریح بین این دو وجود ندارد.

---

## جمع‌بندی

از دید CLR:

1. `ValueTuple` یک خانواده از **generic structها** در فضای نام `System` است.
2. هر سینتکس tuple در C# به یکی از ۹ نوع از پیش تعریف‌شده‌ی `ValueTuple<...>` ترجمه می‌شود.
3. برای بیش از ۷ عضو، ساختار به‌صورت **تودرتو** با فیلد `Rest` بازنمایی می‌شود.
4. نام‌های سفارشی اعضا در **attribute** ذخیره می‌شوند، نه در نام فیلدها.
5. به‌عنوان یک **value type**، allocation روی Heap ندارد مگر در حالت boxing یا قرارگیری درون یک reference type.
6. در IL، tupleها دقیقاً مانند هر struct دیگری با `ldloca`, `ldfld`, `call` constructor مدیریت می‌شوند.

درک این موضوع به شما کمک می‌کند تا:
- IL تولید‌شده را درست تحلیل کنید
- از boxing ناخواسته اجتناب کنید
- با Reflection روی tupleها درست کار کنید
- تفاوت رفتار `ValueTuple` و `Tuple` را پیش‌بینی کنید

---

## منابع

1. **مستندات رسمی مایکروسافت — `System.ValueTuple` Struct**
   [https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple)

2. **مستندات رسمی — Tuple types (C# reference)**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)

3. **مستندات رسمی — `TupleElementNamesAttribute`**
   [https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.tupleelementnamesattribute](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.tupleelementnamesattribute)

4. **منبع باز .NET Runtime — تعریف `ValueTuple`**
   [https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/ValueTuple.cs](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/ValueTuple.cs)

5. **ECMA-335: Common Language Infrastructure (CLI) Specification**
   [https://www.ecma-international.org/publications-and-standards/standards/ecma-335/](https://www.ecma-international.org/publications-and-standards/standards/ecma-335/)
   *(تأیید می‌کند که `ValueTuple` یک نوع ویژه در CLI نیست، بلکه یک کتابخانه‌ی استاندارد است.)*

6. **مقاله‌ی رسمی — Introducing Tuples in C# 7 (Microsoft DevBlogs)**
   [https://devblogs.microsoft.com/dotnet/2017/02/24/introducing-tuples-in-c-7/](https://devblogs.microsoft.com/dotnet/2017/02/24/introducing-tuples-in-c-7/)

---

> **پیشنهاد برای مطالعه‌ی بیشتر:** برای مشاهده‌ی IL دقیق کد خودتان، از ابزار [SharpLab](https://sharplab.io/) یا `ildasm` / `dotnet-ildasm` استفاده کنید و خروجی را با توضیحات این مقاله مقایسه نمایید.