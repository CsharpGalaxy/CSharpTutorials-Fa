# Tuple در C# — جمع‌بندی جامع و راهنمای انتخاب

> **مقاله پایانی مجموعه‌ی Tuple** | آخرین به‌روزرسانی: August 2026
> در این مقاله، تمام مفاهیم Tuple از مقدماتی تا پیشرفته را مرور می‌کنیم و یک راهنمای تصمیم‌گیری کامل ارائه می‌دهیم تا در هر سناریو، بهترین انتخاب را داشته باشید.

---

## فهرست مطالب

1. [مقدمه](#مقدمه)
2. [لینک‌دهی داخلی به مقالات مجموعه](#لینکدهی-داخلی)
3. [مرور مفاهیم پایه](#مرور-مفاهیم-پایه)
   - [Tuple چیست؟](#tuple-چیست)
   - [System.Tuple در مقابل ValueTuple](#systemtuple-در-مقابل-valuetuple)
   - [Tuple Element Names](#tuple-element-names)
   - [Type Identity و Type Inference](#type-identity-و-type-inference)
4. [مرور مفاهیم پیشرفته](#مرور-مفاهیم-پیشرفته)
   - [محدودیت ۷ عضو و TRest](#محدودیت-۷-عضو-و-trest)
   - [Runtime Representation، IL و CLR](#runtime-representation-il-و-clr)
   - [Stack و Heap، Boxing و Closure](#stack-و-heap-boxing-و-closure)
   - [Async State Machine و JIT](#async-state-machine-و-jit)
5. [قابلیت‌های زبان](#قابلیتهای-زبان)
   - [Deconstruction](#deconstruction)
   - [Pattern Matching و Switch Expression](#pattern-matching-و-switch-expression)
   - [Equality و GetHashCode](#equality-و-gethashcode)
   - [Mutable بودن](#mutable-بودن)
6. [مقایسه با سایر ساختارها](#مقایسه-با-سایر-ساختارها)
   - [Anonymous Type](#anonymous-type)
   - [Class، Record و Struct](#class-record-و-struct)
   - [out Parameters](#out-parameters)
   - [DTO](#dto)
7. [جدول مقایسه نهایی](#جدول-مقایسه-نهایی)
8. [Decision Guide — اگر شک داشتم چه تصمیمی بگیرم؟](#decision-guide)
9. [Common Mistakes](#common-mistakes)
10. [Best Practices](#best-practices)
11. [جمع‌بندی](#جمع‌بندی)
12. [منابع و مطالعه بیشتر](#منابع-و-مطالعه-بیشتر)

---

## مقدمه

از زمان معرفی `ValueTuple` در C# 7.0، Tuple به یکی از پرکاربردترین ابزارهای زبان تبدیل شده است. اما این سهولت استفاده، گاهی باعث انتخاب نادرست آن به‌جای `Record`، `Class` یا `Struct` می‌شود. این مقاله به‌عنوان **نقطه‌ی پایانی مجموعه‌ی Tuple**، تمام مفاهیم را یک‌جا جمع می‌کند و یک راهنمای تصمیم‌گیری عملی ارائه می‌دهد.

---

## لینک‌دهی داخلی

برای مطالعه‌ی هر بخش به‌صورت جداگانه، به مقالات زیر در این Repository مراجعه کنید:

| شماره | مقاله | لینک |
|---|---|---|
| 01 | Tuple چیست؟ | [`./01-what-is-tuple.md`](./01-what-is-tuple.md) |
| 02 | System.Tuple در مقابل ValueTuple | [`./02-system-tuple-vs-valuetuple.md`](./02-system-tuple-vs-valuetuple.md) |
| 03 | Tuple Element Names | [`./03-tuple-element-names.md`](./03-tuple-element-names.md) |
| 04 | Type Identity و Type Inference | [`./04-type-identity-and-inference.md`](./04-type-identity-and-inference.md) |
| 05 | بازگرداندن چند مقدار | [`./05-return-multiple-values.md`](./05-return-multiple-values.md) |
| 06 | Deconstruction | [`./06-deconstruction.md`](./06-deconstruction.md) |
| 07 | محدودیت ۷ عضو و TRest | [`./07-seven-element-limit-and-trest.md`](./07-seven-element-limit-and-trest.md) |
| 08 | Runtime Representation و IL | [`./08-runtime-representation-and-il.md`](./08-runtime-representation-and-il.md) |
| 09 | Performance و Boxing | [`./09-performance-and-boxing.md`](./09-performance-and-boxing.md) |
| 10 | Tupleهای بزرگ و Alias | [`./10-large-tuples-and-alias.md`](./10-large-tuples-and-alias.md) |
| 11 | Tuple در Collectionها | [`./11-tuples-in-collections.md`](./11-tuples-in-collections.md) |
| 12 | Pattern Matching با Tuple | [`./12-pattern-matching-with-tuples.md`](./12-pattern-matching-with-tuples.md) |
| 13 | Tuple در API Design | [`./13-tuple-in-api-design.md`](./13-tuple-in-api-design.md) |

---

## مرور مفاهیم پایه

### Tuple چیست؟

Tuple یک ساختار زبانی برای **گروه‌بندی چند مقدار** در یک واحد منطقی است. در C# دو نوع Tuple داریم:

- **`System.Tuple`** (معرفی‌شده در .NET 4.0): یک `class` — یعنی Reference Type.
- **`System.ValueTuple`** (معرفی‌شده در C# 7.0 / .NET 4.7): یک `struct` — یعنی Value Type.

هر دو هدف یکسانی دارند (گروه‌بندی داده‌ها)، اما رفتار، عملکرد و محل قرارگیری در حافظه‌ی آن‌ها کاملاً متفاوت است.

### System.Tuple در مقابل ValueTuple

| ویژگی | `System.Tuple` | `System.ValueTuple` |
|---|---|---|
| نوع | `class` (Reference Type) | `struct` (Value Type) |
| تغییرپذیری | **Immutable** | **Mutable** |
| نام‌گذاری اعضا | ❌ خیر | ✅ بله (فقط در زمان کامپایل) |
| Deconstruction | ❌ خیر | ✅ بله |
| محل ذخیره | Heap | Stack (معمولاً) |
| Boxing | دارد | ندارد (در حالت عادی) |
| نسخه‌ی .NET | 4.0+ | 4.7+ / .NET Standard 2.0 |
| پشتیبانی زبان | محدود | کامل (C# 7.0+) |

> 📌 **نکته**: از C# 7.0 به بعد، سینتکس `(int x, string y)` همیشه به `ValueTuple` ترجمه می‌شود، نه `System.Tuple`.

### Tuple Element Names

نام اعضا در `ValueTuple` **فقط در زمان کامپایل** وجود دارند و توسط Attribute به‌نام `TupleElementNamesAttribute` در IL ذخیره می‌شوند. در زمان اجرا (Runtime)، این نام‌ها از بین می‌روند:

```csharp
(int Id, string Name) person = (1, "Ali");
Console.WriteLine(person.Id);   // ✅ کامپایل می‌شود
Console.WriteLine(person.Item1); // ✅ هم Item1 کار می‌کند
```

در Reflection، شما فقط `Item1`، `Item2` و ... را می‌بینید. نام‌های سفارشی از طریق `TupleElementNamesAttribute` قابل بازیابی هستند.

### Type Identity و Type Inference

**Type Identity**: نوع Tuple بر اساس **نوع اعضا** تعیین می‌شود، نه نام آن‌ها:

```csharp
(int Id, string Name) a = (1, "A");
(int Code, string Title) b = a; // ✅ مجاز است — نوع یکسان است
```

اما ترتیب اعضا مهم است:

```csharp
(int, string) x = (1, "A");
(string, int) y = x; // ❌ خطای کامپایل — نوع متفاوت است
```

**Type Inference**: کامپایلر نوع `ValueTuple` را از مقادیر استنباط می‌کند:

```csharp
var t = (1, "Ali"); // نوع: ValueTuple<int, string>
```

---

## مرور مفاهیم پیشرفته

### محدودیت ۷ عضو و TRest

هر `ValueTuple` به‌صورت مستقیم تا **۷ عضو** تعریف می‌کند. برای بیش از ۷ عضو، از فیلد `TRest` استفاده می‌شود که خودش یک Tuple دیگر است:

```csharp
// ۸ عضو — به‌صورت خودکار با TRest پیاده‌سازی می‌شود
var t = (1, 2, 3, 4, 5, 6, 7, 8);
// معادل: ValueTuple<int,int,int,int,int,int,int, ValueTuple<int>>
```

این طراحی به‌خاطر محدودیت CLR در تعداد Generic Type Parameterها است.

### Runtime Representation، IL و CLR

در IL، یک Tuple مانند `(int, string)` به این صورت نمایش داده می‌یابد:

```il
.valuector valuetype [System.Runtime]System.ValueTuple`2<int32, string>
```

نام‌های اعضا در Attribute جداگانه‌ای ذخیره می‌شوند:

```il
.custom instance void [System.Runtime]System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[])
```

CLR هیچ مفهوم خاصی از Tuple ندارد — این یک ویژگی **صرفاً زبانی** است که توسط کامپایلر C# پیاده‌سازی می‌شود.

### Stack و Heap، Boxing و Closure

- **ValueTuple** به‌عنوان یک `struct`، معمولاً روی **Stack** قرار می‌گیرد (مگر اینکه فیلد یک کلاس باشد یا Boxing شود).
- **System.Tuple** همیشه روی **Heap** قرار می‌گیرد و نیاز به GC دارد.
- **Boxing**: اگر یک `ValueTuple` را به `object` یا یک interface (مثل `IEquatable<T>`) تبدیل کنید، Boxing رخ می‌دهد و روی Heap کپی می‌شود.
- **Closure**: اگر در یک lambda یا closure از Tuple استفاده کنید، کامپایلر ممکن است آن را در یک کلاس مخفی کپچر کند که روی Heap قرار می‌گیرد.

### Async State Machine و JIT

در متدهای `async`، کامپایلر یک **State Machine** تولید می‌کند که متغیرهای محلی (شامل Tupleها) را به‌عنوان فیلد در یک `struct` ذخیره می‌کند. اگر Tuple بزرگ باشد، کپی‌کردن آن در هر `await` هزینه‌بر خواهد بود.

JIT معمولاً Tupleهای کوچک را در رجیستر نگه می‌دارد و هزینه‌ی آن‌ها نزدیک به صفر است. اما Tupleهای بزرگ (بیش از ۴ فیلد) ممکن است روی Stack ریخته شوند و هزینه‌ی کپی داشته باشند.

---

## قابلیت‌های زبان

### Deconstruction

از C# 7.0 می‌توانید Tuple را به متغیرهای جداگانه تجزیه کنید:

```csharp
var (id, name) = GetPerson();
(int id, string name) = (1, "Ali"); // با نوع مشخص
```

Deconstruction برای کلاس‌ها و رکوردها نیز از طریق متد `Deconstruct` قابل پیاده‌سازی است.

### Pattern Matching و Switch Expression

Tuple با Pattern Matching ترکیب قدرتمندی می‌سازد:

```csharp
var result = (state, code) switch
{
    (State.Active, >= 200 and < 300) => "Success",
    (State.Active, 404)              => "Not Found",
    (State.Inactive, _)              => "Disabled",
    _                                => "Unknown"
};
```

### Equality و GetHashCode

`ValueTuple` به‌صورت خودکار `Equals` و `GetHashCode` را بر اساس **تمام اعضا** پیاده‌سازی می‌کند. این رفتار **Value Equality** است:

```csharp
var a = (1, "A");
var b = (1, "A");
Console.WriteLine(a.Equals(b)); // ✅ True
Console.WriteLine(a == b);      // ✅ True (از C# 7.3)
```

### Mutable بودن

`ValueTuple` **Mutable** است — می‌توانید اعضای آن را تغییر دهید:

```csharp
var t = (1, "A");
t.Item1 = 2; // ✅ مجاز
```

این رفتار در مقایسه با `Record` (که Immutable است) تفاوت اساسی دارد و می‌تواند باعث باگ‌های ظریف شود.

---

## مقایسه با سایر ساختارها

### Anonymous Type

```csharp
var anon = new { Id = 1, Name = "Ali" };
```

- **Reference Type** (کلاس تولیدشده توسط کامپایلر)
- **Immutable**
- فقط در محدوده‌ی محلی قابل استفاده (نمی‌تواند از متد بازگردد)
- در Expression Treeها قابل استفاده است
- از C# 3.0

### Class، Record و Struct

- **Class**: Reference Type، Mutable به‌صورت پیش‌فرض، بدون Value Equality خودکار.
- **Record** (C# 9.0): Reference Type (یا `record struct`)، Immutable، دارای Value Equality، Deconstruction، با سینتکس مختصر.
- **Struct**: Value Type، معمولاً برای انواع کوچک و Immutable استفاده می‌شود.

### out Parameters

روش قدیمی برای بازگرداندن چند مقدار:

```csharp
bool TryParse(string s, out int value);
```

مزایا: Performance بالا، بدون تخصیص Heap.
معایب: سینتکس قدیمی، عدم پشتیبانی از async، عدم قابلیت استفاده در Expression.

### DTO (Data Transfer Object)

یک کلاس یا Record برای انتقال داده بین لایه‌ها. معمولاً:

- نام‌گذاری معنادار دارد
- قابل Serialize است
- در Public API استفاده می‌شود
- می‌تواند اعتبارسنجی (Validation) داشته باشد

---

## جدول مقایسه نهایی

| معیار | Tuple / ValueTuple | System.Tuple | Anonymous Type | Class | Record | Struct | DTO | out |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Temporary Data** | ✅ عالی | ⚠️ | ✅ عالی | ❌ | ⚠️ | ⚠️ | ❌ | ✅ |
| **Return Multiple Values** | ✅ عالی | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Public API** | ❌ | ⚠️ | ❌ | ✅ | ✅ | ⚠️ | ✅ | ⚠️ |
| **Domain Model** | ❌ | ❌ | ❌ | ✅ | ✅ | ⚠️ | ✅ | ❌ |
| **Serialization** | ⚠️ | ⚠️ | ❌ | ✅ | ✅ | ⚠️ | ✅ | ❌ |
| **Mutability** | ✅ Mutable | ❌ Immutable | ❌ Immutable | ✅ | ❌ | ⚠️ | ✅ | ⚠️ |
| **Deconstruction** | ✅ | ❌ | ❌ | ⚠️ | ✅ | ⚠️ | ⚠️ | ❌ |
| **Equality** | ✅ Value | ✅ Reference | ✅ Value | ❌ Reference | ✅ Value | ⚠️ | ❌ | ❌ |
| **Performance** | ✅ عالی | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ✅ عالی | ⚠️ | ✅ عالی |
| **Maintainability** | ⚠️ | ⚠️ | ⚠️ | ✅ | ✅ | ⚠️ | ✅ | ⚠️ |
| **Expression Tree** | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ |
| **قابلیت توسعه** | ❌ | ❌ | ❌ | ✅ | ✅ | ⚠️ | ✅ | ❌ |

**راهنمای جدول**: ✅ عالی / مناسب | ⚠️ با احتیاط | ❌ نامناسب

---

## Decision Guide

### اگر شک داشتم از Tuple استفاده کنم یا نه، چه تصمیمی بگیرم؟

این درخت تصمیم به شما کمک می‌کند بهترین انتخاب را داشته باشید:

```
آیا داده فقط در محدوده‌ی محلی یک متد استفاده می‌شود؟
│
├─ بله ──► آیا فقط موقت است و نیازی به Serialize/نام معنادار در API نیست؟
│          │
│          ├─ بله ──► ✅ از (ValueTuple) استفاده کنید
│          │
│          └─ خیر ──► آیا Expression Tree لازم است؟
│                     │
│                     ├─ بله ──► ✅ Anonymous Type
│                     │
│                     └─ خیر ──► ✅ Record محلی (Local Record)
│
└─ خیر ──► آیا از متد بازگردانده می‌شود یا در Public API استفاده می‌شود؟
           │
           ├─ بله ──► آیا معنای دامنه‌ای دارد (Domain Concept)؟
           │          │
           │          ├─ بله ──► آیا Immutable بودن لازم است؟
           │          │          │
           │          │          ├─ بله ──► ✅ Record
           │          │          │
           │          │          └─ خیر ──► ✅ Class یا DTO
           │          │
           │          └─ خیر ──► فقط انتقال داده است؟
           │                     │
           │                     ├─ بله ──► ✅ DTO (Class یا Record)
           │                     │
           │                     └─ خیر ──► ✅ Record
           │
           └─ خیر ──► آیا Performance حیاتی است (Hot Path)؟
                      │
                      ├─ بله ──► آیا اندازه کوچک است؟
                      │          │
                      │          ├─ بله ──► ✅ Struct یا ValueTuple
                      │          │
                      │          └─ خیر ──► ✅ Class (از Boxing اجتناب شود)
                      │
                      └─ خیر ──► ✅ Record
```

### خلاصه‌ی سریع تصمیم‌گیری

| سناریو | انتخاب پیشنهادی |
|---|---|
| بازگرداندن ۲-۳ مقدار موقت از متد خصوصی | **ValueTuple** |
| داده‌ی موقت در یک متد با LINQ | **Anonymous Type** یا **ValueTuple** |
| انتقال داده بین لایه‌های برنامه | **DTO (Record)** |
| مدل دامنه‌ی Immutable | **Record** |
| مدل دامنه‌ی Mutable با رفتار | **Class** |
| نوع کوچک با Performance بالا | **Struct** |
| متد `Try*` با الگوی موفقیت/شکست | **out parameter** یا **ValueTuple** |
| نیاز به Deconstruction عمومی | **Record** |
| API عمومی با معنای مشخص | **DTO با نام معنادار** |
| Hot Path با میلیون‌ها فراخوانی | **Struct** یا **ValueTuple کوچک** |

---

## Common Mistakes

### ❌ 1. استفاده از Tuple در Public API

```csharp
// ❌ بد
public (int, string) GetUser(int id);

// ✅ خوب
public UserDto GetUser(int id);
```

**دلیل**: نام اعضا در Metadata از بین می‌رود و مصرف‌کننده‌ی API (به‌خصوص در زبان‌های دیگر) گیج می‌شود.

### ❌ 2. استفاده از Tuple به‌عنوان کلید Dictionary بدون درک Boxing

```csharp
var dict = new Dictionary<(int, int), string>();
// اگر از interface استفاده کنید، Boxing رخ می‌دهد
```

### ❌ 3. تغییر اعضای Tuple و انتقال آن

```csharp
var t = (1, "A");
var t2 = t;
t.Item1 = 2;
Console.WriteLine(t2.Item1); // 1 — چون Value Type است، کپی شده
```

این رفتار برای کسانی که به Reference Type عادت دارند، گیج‌کننده است.

### ❌ 4. Tuple بزرگ در حلقه‌های داغ

```csharp
// ❌ Tuple با ۸ عضو در هر تکرار کپی می‌شود
for (int i = 0; i < 1_000_000; i++)
{
    var t = (i, i+1, i+2, i+3, i+4, i+5, i+6, i+7);
    Process(t);
}
```

### ❌ 5. استفاده از Tuple به‌جای Domain Model

```csharp
// ❌ بد — معنای دامنه‌ای گم می‌شود
public (decimal Price, int Quantity, decimal Tax) CalculateOrder(...);

// ✅ خوب
public OrderTotal CalculateOrder(...);
```

### ❌ 6. تکیه بر نام اعضا در Reflection

نام اعضا در Runtime وجود ندارد. اگر به آن‌ها نیاز دارید، از `TupleElementNamesAttribute` استفاده کنید.

### ❌ 7. استفاده از System.Tuple در کد جدید

در کد مدرن C#، همیشه از `ValueTuple` استفاده کنید مگر اینکه به‌طور خاص به `System.Tuple` نیاز داشته باشید (مثل APIهای قدیمی .NET).

---

## Best Practices

### ✅ 1. Tuple را برای داده‌ی مخصوصاً محلی نگه دارید

Tuple برای **داده‌های موقت و داخلی** عالی است، نه برای مدل‌های دامنه.

### ✅ 2. همیشه نام اعضا را مشخص کنید

```csharp
// ✅ خوب
public (decimal Total, int Count) GetStats();

// ❌ بد
public (decimal, int) GetStats();
```

### ✅ 3. برای API عمومی از Record یا DTO استفاده کنید

```csharp
public record StatsResult(decimal Total, int Count);
public StatsResult GetStats();
```

### ✅ 4. از Deconstruction برای خوانایی استفاده کنید

```csharp
var (success, result) = TryProcess(data);
if (success) Use(result);
```

### ✅ 5. Tupleهای بزرگ را با `ref readonly` عبور دهید

```csharp
void Process(in (int, int, int, int, int) data) { ... }
```

### ✅ 6. برای الگوی Try از Tuple یا out استفاده کنید

```csharp
// گزینه ۱ — مدرن
(bool Success, User User) TryGetUser(int id);

// گزینه ۲ — سنتی
bool TryGetUser(int id, out User user);
```

### ✅ 7. در Hot Path، از Struct اختصاصی استفاده کنید

اگر Tuple شما در حلقه‌های داغ استفاده می‌شود و بیش از ۴-۵ عضو دارد، یک `struct` اختصاصی بسازید.

### ✅ 8. از Pattern Matching برای کد تمیز استفاده کنید

```csharp
return (state, code) switch
{
    (State.Active, >= 200 and < 300) => Result.Ok,
    (State.Active, 404)              => Result.NotFound,
    _                                => Result.Error
};
```

### ✅ 9. از `ValueTuple.Create` برای ساخت صریح استفاده کنید

```csharp
var t = ValueTuple.Create(1, "A"); // وقتی نوع صریح لازم است
```

### ✅ 10. Tuple را با Alias معنادار کنید

```csharp
using Coordinate = (double Lat, double Lon);
// استفاده:
Coordinate c = (35.6892, 51.3890);
```

---

## جمع‌بندی

Tuple در C# یک ابزار **قدرتمند اما محدود** است. برای درک درست آن، این سه اصل را به خاطر بسپارید:

1. **`ValueTuple` یک Value Type است** — روی Stack قرار می‌گیرد، کپی می‌شود و Mutable است.
2. **نام اعضا فقط در زمان کامپایل است** — در Runtime و Reflection، فقط `Item1`، `Item2` و ... وجود دارد.
3. **Tuple برای داده‌ی موقت است** — برای مدل دامنه، DTO یا Public API، از `Record` یا `Class` استفاده کنید.

**قانون طلایی**:
> اگر نمی‌توانید برای Tuple خود یک **نام معنادار** در دامنه‌ی مسئله بگذارید، احتمالاً به `Record` یا `Class` نیاز دارید.

با این مقاله، مجموعه‌ی Tuple به پایان می‌رسد. امیدواریم این راهنما به شما کمک کند در هر سناریو، انتخاب درستی داشته باشید. 🎯

---

## منابع و مطالعه بیشتر

| # | منبع | موضوع | لینک |
|---|---|---|---|
| 1 | Microsoft Learn — Value Tuple Types | معرفی کامل ValueTuple، سینتکس و قابلیت‌ها | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples |
| 2 | Microsoft Learn — System.Tuple Class | مستندات رسمی System.Tuple | https://learn.microsoft.com/en-us/dotnet/api/system.tuple |
| 3 | Microsoft Learn — System.ValueTuple Struct | مستندات رسمی ValueTuple | https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple |
| 4 | C# Language Proposal — Value Tuples (C# 7.0) | طراحی زبانی و تصمیمات پشت ValueTuple | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-7.0/value-tuples |
| 5 | Microsoft Learn — TupleElementNamesAttribute | مستندات Attribute نام اعضا | https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.tupleelementnamesattribute |
| 6 | Microsoft Learn — Record Types | معرفی Record و مقایسه با Tuple | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record |
| 7 | Microsoft Learn — Anonymous Types | معرفی Anonymous Type | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types |
| 8 | C# Language Specification — Tuple Types | مشخصات رسمی زبان برای Tuple | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/specifications/ |
| 9 | Microsoft Learn — Deconstruction | آموزش Deconstruction در C# | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct |
| 10 | Microsoft Learn — Pattern Matching | آموزش Pattern Matching با Tuple | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching |
| 11 | Microsoft Learn — Records vs Classes | راهنمای انتخاب بین Record و Class | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records#how-records-differ-from-classes-and-structs |
| 12 | .NET Blog — All About Tuple | مقاله‌ی تاریخی معرفی ValueTuple | https://devblogs.microsoft.com/dotnet/2017/04/25/all-about-tuple/ |

---

> 📝 **یادداشت پایانی**: این مقاله به‌عنوان آخرین بخش از مجموعه‌ی آموزشی Tuple در این Repository تهیه شده است. برای بازگشت به فهرست اصلی، به [`README.md`](./README.md) مراجعه کنید.
>
> 💬 اگر سؤال یا پیشنهادی دارید، خوشحال می‌شویم در بخش Issues مطرح کنید.