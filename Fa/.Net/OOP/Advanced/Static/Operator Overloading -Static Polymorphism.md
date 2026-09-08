
# آموزش جامع Operator Overloading و Static Polymorphism در C# و .NET

> **سطح:** مقدماتی تا پیشرفته
> **نسخه:** C# 14 / .NET 10 (با ذکر دقیق نسخه برای هر قابلیت)
> **مخاطب:** برنامه‌نویس C# آشنا با Syntax پایه
> **آخرین به‌روزرسانی:** سپتامبر ۲۰۲۶

---

## فهرست مطالب

- [بخش اول — Operator Overloading](#بخش-اول--operator-overloading)
  - [1. Operator چیست؟](#1-operator-چیست)
  - [2. Operator Overloading چیست؟](#2-operator-overloading-چیست)
  - [3. چه Operatorهایی قابل Overload هستند؟](#3-چه-operatorهایی-قابل-overload-هستند)
  - [4. Operatorهایی که غیرمستقیم Overload می‌شوند](#4-operatorهایی-که-غیرمستقیم-overload-میشوند)
- [بخش دوم — Operator Function](#بخش-دوم--operator-function)
  - [5. Operator Function چیست؟](#5-operator-function-چیست)
  - [6. قوانین Operator Overloading](#6-قوانین-operator-overloading)
  - [7. Operator Resolution](#7-operator-resolution)
- [بخش سوم — Equality و Comparison](#بخش-سوم--equality-و-comparison)
  - [8. Overloading عملگرهای Equality](#8-overloading-عملگرهای-equality)
  - [9. Equals و GetHashCode](#9-equals-و-gethashcode)
  - [10. قوانین GetHashCode](#10-قوانین-gethashcode)
  - [11. Comparison Operators](#11-comparison-operators)
- [بخش چهارم — Custom Conversion](#بخش-چهارم--custom-conversion)
  - [12. Implicit Conversion](#12-implicit-conversion)
  - [13. Explicit Conversion](#13-explicit-conversion)
  - [14. Implicit در برابر Explicit](#14-implicit-در-برابر-explicit)
  - [15. Conversion و is/as](#15-conversion-و-isas)
  - [16. Constructor و ToXXX/FromXXX](#16-constructor-و-toxxxfromxxx)
- [بخش پنجم — Checked Operators](#بخش-پنجم--checked-operators)
  - [17. Checked Operator چیست؟](#17-checked-operator-چیست)
  - [18. Checked Operator در C# مدرن](#18-checked-operator-در-c-مدرن)
- [بخش ششم — Operator از دید Compiler](#بخش-ششم--operator-از-دید-compiler)
  - [19. Operator Overloading از دید Roslyn](#19-operator-overloading-از-دید-roslyn)
  - [20. Operator از دید IL و CLR](#20-operator-از-دید-il-و-clr)
  - [21. Operator از دید JIT](#21-operator-از-دید-jit)
  - [22. Performance و Memory](#22-performance-و-memory)
- [بخش هفتم — اصول طراحی Operator](#بخش-هفتم--اصول-طراحی-operator)
  - [23. Best Practices](#23-best-practices)
  - [24. چه زمانی Operator Overloading نکنیم؟](#24-چه-زمانی-operator-overloading-نکنیم)
- [بخش هشتم — Static Polymorphism](#بخش-هشتم--static-polymorphism)
  - [25. Static Polymorphism چیست؟](#25-static-polymorphism-چیست)
  - [26. Static Abstract Interface Members](#26-static-abstract-interface-members)
  - [27. Static Abstract Operator](#27-static-abstract-operator)
- [بخش نهم — Generic Math](#بخش-نهم--generic-math)
  - [28. Generic Math](#28-generic-math)
  - [29. INumber\<T\>](#29-inumbert)
  - [30. Static Polymorphism از Roslyn تا CPU](#30-static-polymorphism-از-roslyn-تا-cpu)
- [بخش‌های تکمیلی](#بخشهای-تکمیلی)
  - [Cheat Sheet](#cheat-sheet)
  - [Comparison Table](#comparison-table)
  - [Common Mistakes](#common-mistakes)
  - [Exercises](#exercises)
  - [Interview Questions](#interview-questions)
  - [Real-World Design](#real-world-design)
- [منابع](#منابع)

---

# بخش اول — Operator Overloading

## 1. Operator چیست؟

### Operator

در زبان‌های برنامه‌نویسی، **Operator** نمادی است که یک عملیات خاص را روی یک یا چند مقدار (Operand) انجام می‌دهد و نتیجه‌ای تولید می‌کند. به بیان ساده‌تر، Operator دستوری است که به Compiler می‌گوید «چه کاری روی داده‌ها انجام بده».

### Operand

**Operand** مقداری است که Operator روی آن عمل می‌کند. در عبارت `3 + 5`:
- `+` Operator است
- `3` و `5` Operand هستند

### Unary Operator

Operatorی که فقط روی **یک** Operand عمل می‌کند:

```csharp
int x = 5;
int y = -x;   // Unary minus → y = -5
bool b = !true; // Logical NOT → b = false
int z = ++x;  // Pre-increment → z = 6
```

### Binary Operator

Operatorی که روی **دو** Operand عمل می‌کند:

```csharp
int sum = 3 + 5;       // Addition
bool eq = (3 == 5);    // Equality
int shifted = 1 << 3;  // Left shift
```

### مثال از Operatorهای داخلی C#

| دسته | Operatorها | مثال |
|------|-----------|------|
| حسابی | `+`, `-`, `*`, `/`, `%` | `10 / 3` → `3` |
| مقایسه‌ای | `==`, `!=`, `<`, `>`, `<=`, `>=` | `5 > 3` → `true` |
| منطقی | `&&`, `\|\|`, `!` | `true && false` → `false` |
| بیتی | `&`, `\|`, `^`, `~`, `<<`, `>>` | `0xFF & 0x0F` → `0x0F` |
| انتسابی | `=`, `+=`, `-=`, `*=`, `/=` | `x += 5` |
| سایر | `?.`, `??`, `=>`, `is`, `as` | `x ?? 0` |

### تفاوت Operator با Method

| ویژگی | Operator | Method |
|-------|----------|--------|
| Syntax | Infix (`a + b`) یا Prefix/Postfix | `a.Add(b)` |
| نام | نماد (`+`, `-`, `==`) | شناسه (`Add`, `Subtract`) |
| خوانایی برای عملیات ریاضی | بالاتر | پایین‌تر |
| قابلیت کشف (Discoverability) | پایین‌تر (باید بدانید وجود دارد) | بالاتر (IntelliSense) |
| Overloading | محدود به Operatorهای مشخص | آزاد |

### چرا C# اجازه Operator Overloading را می‌دهد؟

C# یک زبان **چندپارادایمی** است که هم از برنامه‌نویسی شیءگرا و هم از مفاهیم ریاضی/ساختاریافته پشتیبانی می‌کند. هدف از Operator Overloading این است که **Typeهای سفارشی** (مانند `Vector3D`, `Complex`, `Money`) بتوانند با همان Syntax طبیعی و آشنای Typeهای داخلی (`int`, `double`) استفاده شوند. این قابلیت:

1. **خوانایی** کد را برای دامنه‌های ریاضی و علمی افزایش می‌دهد
2. **یکپارچگی** بین Typeهای داخلی و سفارشی ایجاد می‌کند
3. **Expressiveness** زبان را بالا می‌برد

---

## 2. Operator Overloading چیست؟

### تعریف

**Operator Overloading** به معنای تعریف رفتار جدید برای یک Operator موجود، برای Typeهای سفارشی (User-Defined Types) است. به‌عبارت دیگر، شما به Compiler می‌گویید وقتی `+` را بین دو شیء از Type `Money` دیدی، دقیقاً چه کاری انجام بده.

### هدف از طراحی آن در C#

طراحان C# (به‌ویژه Anders Hejlsberg) Operator Overloading را با رویکردی **محتاطانه‌تر** از C++ پیاده‌سازی کردند:

- فقط Operatorهای مشخصی قابل Overload هستند (نه همه)
- قوانین سخت‌گیرانه‌ای برای Signature وجود دارد
- Operatorها باید `public static` باشند
- هدف: **Expressiveness بدون از دست دادن پیش‌بینی‌پذیری**

### چگونه Typeهای سفارشی شبیه Typeهای Built-in رفتار می‌کنند؟

```csharp
// Built-in
int total = price1 + price2;

// User-defined (با Operator Overloading)
Money total = price1 + price2;  // دقیقاً همان Syntax!
```

### چه زمانی Operator Overloading منطقی است؟

- Type شما یک **مفهوم ریاضی** یا **عددی** را مدل می‌کند (Vector, Matrix, Complex, Fraction)
- Type شما یک **واحد اندازه‌گیری** است (Money, Temperature, Distance)
- عملیات **Semantic واضح و universally understood** دارد (`+` برای جمع دو پول)
- انتظار می‌رود کاربر Type شما از Operatorها استفاده کند

### چه زمانی نباید استفاده کنیم؟

- عملیات **Side Effect** دارد (مثلاً `+` داده‌ای را در دیتابیس ذخیره کند)
- Semantic **مبهم** است (`+` برای Merge دو لیست؟ Concatenate؟ Union؟)
- رفتار **غیرمنتظره** دارد (`+` که تفریق کند!)
- یک **Method نام‌دار** خوانایی بیشتری دارد

### مثال ساده: `struct Point`

```csharp
// C# 14 / .NET 10
public readonly record struct Point(int X, int Y)
{
    public static Point operator +(Point left, Point right) =>
        new(left.X + right.X, left.Y + right.Y);

    public static Point operator -(Point left, Point right) =>
        new(left.X - right.X, left.Y - right.Y);

    public static Point operator -(Point point) =>
        new(-point.X, -point.Y);
}

// استفاده
var p1 = new Point(3, 4);
var p2 = new Point(1, 2);
var p3 = p1 + p2; // Point(4, 6)
var p4 = -p1;     // Point(-3, -4)
```

---

## 3. چه Operatorهایی قابل Overload هستند؟

### Unary Operators

| Operator | نام | توضیح |
|----------|-----|-------|
| `+` | Unary Plus | معمولاً بدون تغییر |
| `-` | Unary Negation | قرینه کردن |
| `!` | Logical NOT | نقیض منطقی |
| `~` | Bitwise Complement | مکمل بیتی |
| `++` | Increment | افزایش |
| `--` | Decrement | کاهش |
| `true` | True Operator | ارزیابی صحت |
| `false` | False Operator | ارزیابی عدم صحت |

### Binary Operators

| Operator | نام |
|----------|-----|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulus |
| `&` | Bitwise AND |
| `\|` | Bitwise OR |
| `^` | Bitwise XOR |
| `<<` | Left Shift |
| `>>` | Right Shift |
| `>>>` | Unsigned Right Shift *(C# 11+)* |
| `==` | Equality |
| `!=` | Inequality |
| `<` | Less Than |
| `>` | Greater Than |
| `<=` | Less Than or Equal |
| `>=` | Greater Than or Equal |

### Conversion Operators

| Operator | توضیح |
|----------|-------|
| `implicit` | تبدیل ضمنی (بدون Cast) |
| `explicit` | تبدیل صریح (با Cast) |

### Operatorهای غیرقابل Overload

| Operator | دلیل |
|----------|------|
| `=` | Assignment همیشه توسط Compiler مدیریت می‌شود |
| `&&`, `\|\|` | از `&`, `\|`, `true`, `false` مشتق می‌شوند |
| `[]` | از Indexer استفاده کنید |
| `()` | از Delegate/Invoke استفاده کنید |
| `+=`, `-=`, `*=`, `/=`, `%=` | از Operator اصلی مشتق می‌شوند |
| `is`, `as` | عملگرهای Type-checking |
| `?.`, `??`, `??=` | عملگرهای Null-handling |
| `=>`, `->` | Syntax-level constructs |
| `new`, `typeof`, `sizeof`, `nameof` | عملگرهای خاص زبان |
| `.` | Member access |
| `checked`, `unchecked` | Context operators |

---

## 4. Operatorهایی که غیرمستقیم Overload می‌شوند

### Compound Assignment Operators

وقتی شما `+` را Overload می‌کنید، `+=` به‌صورت خودکار توسط Compiler بر اساس آن تولید می‌شود:

```csharp
public readonly record struct Money(decimal Amount, string Currency)
{
    public static Money operator +(Money left, Money right)
    {
        if (left.Currency != right.Currency)
            throw new InvalidOperationException("Currency mismatch");
        return new(left.Amount + right.Amount, left.Currency);
    }
}

var m1 = new Money(100m, "USD");
var m2 = new Money(50m, "USD");

m1 += m2; // Compiler این را به m1 = m1 + m2 تبدیل می‌کند
// نتیجه: Money(150, "USD")
```

**نکته مهم:** برای `struct`های `readonly`، عبارت `m1 += m2` در واقع یک متغیر جدید تولید می‌کند و به `m1` نسبت می‌دهد. برای `class`ها، این رفتار ممکن است متفاوت به نظر برسد اما در واقع همان `m1 = m1 + m2` است.

### Conditional Operators: `&&` و `||`

این Operatorها مستقیماً قابل Overload نیستند، اما از ترکیب `&`/`|` با `true`/`false` مشتق می‌شوند:

```csharp
public struct TruthValue
{
    public bool Value { get; }
    public TruthValue(bool value) => Value = value;

    // برای && و || به اینها نیاز داریم:
    public static bool operator true(TruthValue t) => t.Value;
    public static bool operator false(TruthValue t) => !t.Value;

    // عملگر بیتی که پایه && و || است:
    public static TruthValue operator &(TruthValue a, TruthValue b) =>
        new(a.Value && b.Value);

    public static TruthValue operator |(TruthValue a, TruthValue b) =>
        new(a.Value || b.Value);
}

// حالا این کار می‌کند:
TruthValue a = new(true);
TruthValue b = new(false);
TruthValue c = a && b; // از operator & و operator false استفاده می‌شود
```

**مکانیزم `&&`:**
```
a && b
→ اگر operator false(a) true باشد، نتیجه a است (short-circuit)
→ در غیر این صورت، نتیجه a & b است
```

**مکانیزم `||`:**
```
a || b
→ اگر operator true(a) true باشد، نتیجه a است (short-circuit)
→ در غیر این صورت، نتیجه a | b است
```

---

# بخش دوم — Operator Function

## 5. Operator Function چیست؟

### تعریف

**Operator Function** یک Method خاص با Syntax ویژه است که رفتار یک Operator را برای یک Type تعریف می‌کند. این Method با کلمه کلیدی `operator` تعریف می‌شود.

### Syntax

```csharp
public static ReturnType operator OperatorSymbol(Parameters)
{
    // implementation
}
```

### تحلیل خط‌به‌خط مثال

```csharp
public static Point operator +(Point left, Point right)
{
    return new Point(
        left.X + right.X,
        left.Y + right.Y);
}
```

| بخش | توضیح |
|-----|-------|
| `public` | Operator باید همیشه `public` باشد تا از بیرون Type قابل استفاده باشد |
| `static` | Operator به Instance خاصی تعلق ندارد؛ روی Type تعریف می‌شود |
| `Point` | Return Type — نوع مقداری که Operator برمی‌گرداند |
| `operator` | کلمه کلیدی که به Compiler می‌گوید این یک Operator Function است |
| `+` | نماد Operatorی که Overload می‌شود |
| `Point left` | اولین Operand (سمت چپ `+`) |
| `Point right` | دومین Operand (سمت راست `+`) |
| `return new Point(...)` | محاسبه و برگرداندن نتیجه |

### نقش `public`

Operatorها **باید** `public` باشند. این یک الزام زبان است. دلیل: Operatorها بخشی از Public API یک Type هستند و باید از هر جایی که Type قابل دسترسی است، قابل استفاده باشند.

### نقش `static`

Operatorها **باید** `static` باشند. دلیل:
- Operatorها قبل از اینکه شیئی وجود داشته باشد باید قابل فراخوانی باشند
- این رفتار مشابه Built-in Operators است (`int + int` نیازی به Instance ندارد)
- در سطح CLR، Operatorها به‌صورت Static Methodهای ویژه (`op_*`) کامپایل می‌شوند

### پارامترها به‌عنوان Operand

- در **Unary Operator**: دقیقاً یک پارامتر (همان Type تعریف‌کننده)
- در **Binary Operator**: دقیقاً دو پارامتر (حداقل یکی باید Type تعریف‌کننده باشد)

### Return Type

Return Type می‌تواند هر Typeای باشد، اما معمولاً:
- برای عملیات حسابی: همان Type (`Point + Point → Point`)
- برای مقایسه: `bool` (`Point == Point → bool`)
- برای Conversion: Type مقصد

---

## 6. قوانین Operator Overloading

### قانون ۱: Operator باید `public` باشد

```csharp
// ✅ صحیح
public static Point operator +(Point a, Point b) => ...;

// ❌ غلط - Compiler Error CS0558
private static Point operator +(Point a, Point b) => ...;
internal static Point operator +(Point a, Point b) => ...;
```

### قانون ۲: Operator باید `static` باشد

```csharp
// ✅ صحیح
public static Point operator +(Point a, Point b) => ...;

// ❌ غلط - Compiler Error CS0558
public Point operator +(Point a, Point b) => ...;
```

### قانون ۳: حداقل یکی از Operandها باید Type تعریف‌کننده باشد

```csharp
public struct Money
{
    // ✅ صحیح - Money یکی از پارامترهاست
    public static Money operator +(Money m, decimal d) => ...;

    // ❌ غلط - هیچ‌کدام Money نیستند
    // Compiler Error CS0563
    public static int operator +(int a, int b) => ...;
}
```

### قانون ۴: Operatorهای جفتی باید با هم تعریف شوند

| اگر این را تعریف کنید | باید این را هم تعریف کنید |
|----------------------|--------------------------|
| `==` | `!=` |
| `<` | `>` |
| `<=` | `>=` |
| `true` | `false` |

```csharp
// ❌ غلط - Compiler Error CS0216
public static bool operator ==(Money a, Money b) => ...;
// بدون != تعریف شده

// ✅ صحیح
public static bool operator ==(Money a, Money b) => ...;
public static bool operator !=(Money a, Money b) => ...;
```

### قانون ۵: Signature محدودیت دارد

- تعداد پارامترها ثابت است (۱ برای Unary، ۲ برای Binary)
- نمی‌توانید Precedence یا Associativity را تغییر دهید
- نمی‌توانید Operator جدید بسازید

### مثال‌های صحیح و غلط

```csharp
public struct Fraction
{
    public int Numerator { get; }
    public int Denominator { get; }

    // ✅ صحیح: Binary operator
    public static Fraction operator +(Fraction a, Fraction b) =>
        new(a.Numerator * b.Denominator + b.Numerator * a.Denominator,
            a.Denominator * b.Denominator);

    // ✅ صحیح: Unary operator
    public static Fraction operator -(Fraction f) =>
        new(-f.Numerator, f.Denominator);

    // ✅ صحیح: Conversion
    public static implicit operator double(Fraction f) =>
        (double)f.Numerator / f.Denominator;

    // ❌ غلط: نمی‌توانید operator جدید بسازید
    // public static Fraction operator **(Fraction a, Fraction b) => ...;

    // ❌ غلط: نمی‌توانید تعداد پارامترها را تغییر دهید
    // public static Fraction operator +(Fraction a, Fraction b, Fraction c) => ...;
}
```

---

## 7. Operator Resolution

وقتی می‌نویسید `var result = a + b;`، Compiler مراحل زیر را طی می‌کند:

```
Source Code: a + b
     ↓
[1] Syntax Analysis (Roslyn)
    → تشخیص AddExpression در Syntax Tree
     ↓
[2] Semantic Analysis
    → تعیین Typeهای a و b
     ↓
[3] Operator Resolution
    → جستجوی Candidate Operators:
       a) Built-in operators (int+int, double+double, ...)
       b) User-defined operators (در Type a و Type b)
       c) Lifted operators (برای Nullable<T>)
     ↓
[4] Overload Resolution
    → انتخاب Best Candidate بر اساس:
       - Exact match
       - Implicit conversions
       - Better conversion rules
     ↓
[5] Best Candidate Selected
    → اگر دقیقاً یک best وجود دارد: ✅
    → اگر Ambiguous باشد: ❌ Compiler Error CS0034
    → اگر هیچ‌کدام نباشد: ❌ Compiler Error CS0019
     ↓
[6] IL Generation
    → تبدیل به call op_Addition یا add (برای Built-in)
```

### مثال عملی

```csharp
Money a = new(100m, "USD");
Money b = new(50m, "USD");
var result = a + b;

// Compiler:
// 1. Type(a) = Money, Type(b) = Money
// 2. جستجو: آیا Money operator +(Money, Money) دارد؟ بله!
// 3. Best candidate: Money.operator +(Money, Money)
// 4. IL: call valuetype Money Money::op_Addition(valuetype Money, valuetype Money)
```

---

# بخش سوم — Equality و Comparison

## 8. Overloading عملگرهای Equality

### `==` و `!=`

در C#، `==` و `!=` **باید** همیشه با هم تعریف شوند. Compiler این را اجبار می‌کند (CS0216).

### چرا معمولاً باید با هم تعریف شوند؟

از نظر منطقی، `!=` نقیض `==` است. اگر فقط یکی را تعریف کنید، رفتار ناسازگار ایجاد می‌شود. Compiler با اجبار کردن تعریف هر دو، از این ناسازگاری جلوگیری می‌کند.

### تفاوت Equality Reference و Value

| نوع | `==` پیش‌فرض | توضیح |
|-----|-------------|-------|
| `class` (Reference Type) | Reference Equality | آیا دو متغیر به یک شیء اشاره می‌کنند؟ |
| `struct` (Value Type) | Value Equality (با Reflection) | آیا تمام Fieldها برابرند؟ (کند!) |
| `record` | Value Equality | بر اساس تمام Propertyها |
| `record struct` | Value Equality | بر اساس تمام Fieldها |

### مثال با `struct`

```csharp
public readonly struct Money : IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public static bool operator ==(Money left, Money right) =>
        left.Amount == right.Amount && left.Currency == right.Currency;

    public static bool operator !=(Money left, Money right) =>
        !(left == right);

    public bool Equals(Money other) =>
        Amount == other.Amount && Currency == other.Currency;

    public override bool Equals(object? obj) =>
        obj is Money other && Equals(other);

    public override int GetHashCode() =>
        HashCode.Combine(Amount, Currency);
}
```

### مثال با `class`

```csharp
public class MoneyClass : IEquatable<MoneyClass>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public MoneyClass(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public static bool operator ==(MoneyClass? left, MoneyClass? right)
    {
        if (left is null) return right is null;
        return left.Equals(right);
    }

    public static bool operator !=(MoneyClass? left, MoneyClass? right) =>
        !(left == right);

    public bool Equals(MoneyClass? other)
    {
        if (other is null) return false;
        return Amount == other.Amount && Currency == other.Currency;
    }

    public override bool Equals(object? obj) =>
        Equals(obj as MoneyClass);

    public override int GetHashCode() =>
        HashCode.Combine(Amount, Currency);
}
```

> **نکته مهم برای `class`:** در `operator ==` برای Reference Typeها، از `ReferenceEquals` یا `is null` استفاده کنید تا از infinite recursion جلوگیری شود. هرگز `left == null` ننویسید چون دوباره `operator ==` را فراخوانی می‌کند!

---

## 9. Equals و GetHashCode

### چرا باید `Equals()` و `GetHashCode()` را هم Override کنیم؟

وقتی `==` را Overload می‌کنید، سه مکانیسم مقایسه در .NET وجود دارد:

1. **`==` Operator** — استفاده مستقیم در کد
2. **`Object.Equals()`** — استفاده توسط Collections، LINQ، و Framework
3. **`Object.GetHashCode()`** — استفاده توسط `Dictionary`, `HashSet`

اگر فقط `==` را Overload کنید و `Equals` را Override نکنید:

```csharp
var m1 = new Money(100m, "USD");
var m2 = new Money(100m, "USD");

Console.WriteLine(m1 == m2);           // True ✅ (operator ==)
Console.WriteLine(m1.Equals(m2));      // ممکن است False باشد ❌
Console.WriteLine(object.Equals(m1, m2)); // ممکن است False باشد ❌

var set = new HashSet<Money> { m1 };
Console.WriteLine(set.Contains(m2));   // ممکن است False باشد ❌
```

### مثال کامل

```csharp
public readonly record struct Temperature(decimal Celsius)
{
    // Operator Overloading
    public static bool operator ==(Temperature left, Temperature right) =>
        left.Celsius == right.Celsius;

    public static bool operator !=(Temperature left, Temperature right) =>
        !(left == right);

    // record struct خودش Equals و GetHashCode تولید می‌کند
    // اما اگر struct معمولی بود:

    // public bool Equals(Temperature other) => Celsius == other.Celsius;
    // public override bool Equals(object? obj) => obj is Temperature t && Equals(t);
    // public override int GetHashCode() => Celsius.GetHashCode();
}
```

> **نکته:** `record` و `record struct` به‌صورت خودکار `Equals`, `GetHashCode`, `==`, `!=`, `ToString` و `Deconstruct` را تولید می‌کنند. استفاده از `record` برای Typeهای Value-based توصیه می‌شود.

---

## 10. قوانین GetHashCode

### رابطه `Equals` و `GetHashCode`

این قرارداد (Contract) باید همیشه رعایت شود:

| شرط | الزام |
|-----|-------|
| اگر `a.Equals(b)` → `true` | **حتماً** `a.GetHashCode() == b.GetHashCode()` |
| اگر `a.GetHashCode() == b.GetHashCode()` | **الزامی نیست** `a.Equals(b)` → `true` |
| `GetHashCode()` برای یک شیء | باید در طول عمر شیء **ثابت** بماند (اگر در Hash Collection استفاده می‌شود) |

### Hash Collision چیست؟

وقتی دو شیء نابرابر HashCode یکسان دارند. این **اجتناب‌ناپذیر** است (چون تعداد HashCodeها محدود به `int` است ولی تعداد اشیاء نامحدود). Collections مانند `Dictionary` با Collision از طریق Bucket و زنجیره‌سازی (chaining) برخورد می‌کنند.

### چرا Mutable Field مشکل ایجاد می‌کند؟

```csharp
public struct BadKey
{
    public int Value; // Mutable!

    public override int GetHashCode() => Value.GetHashCode();
}

var dict = new Dictionary<BadKey, string>();
var key = new BadKey { Value = 42 };
dict[key] = "hello";

key.Value = 99; // HashCode تغییر کرد!
// dict.ContainsKey(key) → ممکن است false برگرداند!
// شیء در Dictionary گم شده است
```

### چرا `Dictionary` و `HashSet` به HashCode نیاز دارند؟

این Collections از **Hash Table** استفاده می‌کنند. HashCode تعیین می‌کند شیء در کدام Bucket قرار بگیرد. بدون HashCode صحیح، Lookup از O(1) به O(n) تنزل می‌کند.

### استفاده از `HashCode.Combine`

```csharp
public readonly struct Money : IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public override int GetHashCode() =>
        HashCode.Combine(Amount, Currency);

    // HashCode.Combine تا 8 پارامتر را پشتیبانی می‌کند
    // برای بیشتر:
    // var hash = new HashCode();
    // hash.Add(field1);
    // hash.Add(field2);
    // ...
    // return hash.ToHashCode();
}
```

---

## 11. Comparison Operators

### `<`, `>`, `<=`, `>=`

این Operatorها نیز باید به‌صورت جفتی تعریف شوند: `<` با `>` و `<=` با `>=`.

### سازگاری منطقی

اگر `a < b` → `true` باشد، آنگاه:
- `a > b` → `false`
- `a <= b` → `true`
- `b > a` → `true`
- `b < a` → `false`

### ارتباط با `IComparable<T>`

```csharp
public readonly struct Money : IComparable<Money>, IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public int CompareTo(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot compare different currencies");
        return Amount.CompareTo(other.Amount);
    }

    public static bool operator <(Money left, Money right) =>
        left.CompareTo(right) < 0;

    public static bool operator >(Money left, Money right) =>
        left.CompareTo(right) > 0;

    public static bool operator <=(Money left, Money right) =>
        left.CompareTo(right) <= 0;

    public static bool operator >=(Money left, Money right) =>
        left.CompareTo(right) >= 0;

    public static bool operator ==(Money left, Money right) =>
        left.Currency == right.Currency && left.Amount == right.Amount;

    public static bool operator !=(Money left, Money right) =>
        !(left == right);

    public bool Equals(Money other) => this == other;
    public override bool Equals(object? obj) => obj is Money m && Equals(m);
    public override int GetHashCode() => HashCode.Combine(Amount, Currency);
}
```

> **نکته:** `record struct` در C# 10+ به‌صورت خودکار `IComparable<T>` را implement نمی‌کند. اگر به Comparison نیاز دارید، باید خودتان implement کنید.

---

# بخش چهارم — Custom Conversion

## 12. Implicit Conversion

### تعریف

**Implicit Conversion** تبدیلی است که **بدون Cast صریح** و به‌صورت خودکار توسط Compiler انجام می‌شود.

```csharp
public static implicit operator TargetType(SourceType value)
{
    // conversion logic
}
```

### چه زمانی مناسب است؟

فقط زمانی که تبدیل:
1. **همیشه موفق** باشد (هرگز Exception ندهد)
2. **اطلاعاتی از دست نرود** (No data loss)
3. **از نظر Semantic منطقی** باشد

### مثال تبدیل امن

```csharp
public readonly struct Celsius
{
    public decimal Value { get; }
    public Celsius(decimal value) => Value = value;

    // تبدیل از Celsius به Kelvin همیشه امن است
    // هیچ اطلاعاتی از دست نمی‌رود و هرگز شکست نمی‌خورد
    public static implicit operator Kelvin(Celsius c) =>
        new(c.Value + 273.15m);
}

public readonly struct Kelvin
{
    public decimal Value { get; }
    public Kelvin(decimal value) => Value = value;
}

// استفاده
Celsius boiling = new(100m);
Kelvin k = boiling; // بدون Cast! → Kelvin(373.15)
```

### شرط مهم

> **قانون طلایی Implicit Conversion:** اگر تبدیل می‌تواند شکست بخورد یا اطلاعات از دست بدهد، **هرگز** آن را `implicit` نکنید.

---

## 13. Explicit Conversion

### تعریف

**Explicit Conversion** تبدیلی است که **نیاز به Cast صریح** دارد. این نشان‌دهنده آن است که تبدیل ممکن است شکست بخورد یا اطلاعات از دست برود.

```csharp
public static explicit operator TargetType(SourceType value)
{
    // conversion logic
}
```

### چه زمانی استفاده شود؟

- احتمال **از دست رفتن اطلاعات** وجود دارد
- احتمال **Exception** وجود دارد
- تبدیل از نظر Semantic **بدیهی نیست**

### مثال

```csharp
public readonly struct Kelvin
{
    public decimal Value { get; }
    public Kelvin(decimal value) => Value = value;

    // تبدیل از Kelvin به Celsius: ممکن است مقدار غیرفیزیکی باشد
    // (زیر صفر مطلق) → اطلاعات معنایی از دست می‌رود
    public static explicit operator Celsius(Kelvin k)
    {
        if (k.Value < 0m)
            throw new ArgumentOutOfRangeException(nameof(k), "Kelvin cannot be negative");
        return new Celsius(k.Value - 273.15m);
    }
}

// استفاده
Kelvin k = new(373.15m);
Celsius c = (Celsius)k; // Cast صریح لازم است → Celsius(100)
```

---

## 14. Implicit در برابر Explicit

| ویژگی | Implicit | Explicit |
|-------|----------|----------|
| نیاز به Cast | ❌ خیر | ✅ بله |
| احتمال Failure | ❌ هرگز | ✅ ممکن است |
| Loss of Data | ❌ هرگز | ✅ ممکن است |
| خوانایی | بالاتر (طبیعی‌تر) | واضح‌تر (هشدار به Developer) |
| کاربرد مناسب | تبدیل‌های امن و بدون ضرر | تبدیل‌های خطرناک یا با احتمال خطا |
| مثال Built-in | `int` → `long` | `long` → `int` |
| Syntax | `Target t = source;` | `Target t = (Target)source;` |

---

## 15. Conversion و is/as

### آیا Custom Conversionها توسط `is` و `as` پشتیبانی می‌شوند؟

**خیر.** این یکی از مهم‌ترین نکاتی است که بسیاری از برنامه‌نویسان اشتباه می‌کنند.

`is` و `as` فقط **Inheritance Hierarchy** و **Interface Implementation** را بررسی می‌کنند. آنها Custom Conversion Operatorها را **نمی‌شناسند**.

```csharp
Celsius c = new(100m);

// ❌ این کار نمی‌کند!
var result1 = c is Kelvin;    // false (همیشه!)
var result2 = c as Kelvin;    // Compiler Error: 'as' فقط برای Reference Typeها

// ✅ این کار می‌کند:
Kelvin k = c; // implicit conversion

// برای بررسی، باید از Pattern Matching با نوع استفاده کنید:
object obj = c;
if (obj is Celsius) { /* true */ }
if (obj is Kelvin)  { /* false - conversion اعمال نمی‌شود */ }
```

### تفاوت کلیدی

| مکانیسم | Custom Conversion | Inheritance/Interface |
|---------|-------------------|----------------------|
| `is` | ❌ پشتیبانی نمی‌شود | ✅ پشتیبانی می‌شود |
| `as` | ❌ پشتیبانی نمی‌شود | ✅ پشتیبانی می‌شود |
| Cast `(T)` | ✅ پشتیبانی می‌شود | ✅ پشتیبانی می‌شود |
| Assignment | ✅ (implicit) | ✅ (upcast) |

---

## 16. Constructor و ToXXX/FromXXX

### چه زمانی به‌جای Operator Conversion از Method استفاده کنیم؟

| روش | مناسب برای |
|-----|-----------|
| `implicit operator` | تبدیل‌های امن، بدون ضرر، و obvious |
| `explicit operator` | تبدیل‌های ممکن است با ضرر |
| `Constructor` | وقتی ساخت شیء جدید با ورودی مشخص مدنظر است |
| `ToXXX()` / `FromXXX()` | وقتی نام تبدیل مهم است و Semantic خاصی دارد |

### مثال

```csharp
public readonly struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    // Factory Method - خوانا و واضح
    public static Money FromDollar(decimal amount) => new(amount, "USD");
    public static Money FromEuro(decimal amount) => new(amount, "EUR");

    // تبدیل صریح با نام مشخص
    public Money ToEuro(decimal exchangeRate) =>
        new(Amount * exchangeRate, "EUR");

    public Money ToDollar(decimal exchangeRate) =>
        new(Amount * exchangeRate, "USD");
}

// استفاده
var usd = Money.FromDollar(100m);
var eur = usd.ToEuro(0.85m);
```

### معیار انتخاب

| معیار | Operator Conversion | Method (ToXXX/FromXXX) |
|-------|-------------------|----------------------|
| تبدیل ساده و بدون پارامتر اضافی | ✅ | ⚠️ |
| نیاز به پارامتر (مثل exchange rate) | ❌ | ✅ |
| Semantic مبهم | ❌ | ✅ |
| خوانایی بالا با نام | ⚠️ | ✅ |
| استفاده در Generic Code | ✅ (با Interface) | ❌ |

---

# بخش پنجم — Checked Operators

## 17. Checked Operator چیست؟

### Checked Context

در C#، عملیات حسابی روی Typeهای عددی صحیح (`int`, `long`, ...) می‌توانند **Overflow** کنند. رفتار پیش‌فرض C# در حالت `unchecked` است (یعنی Overflow بدون خطا رخ می‌دهد و مقدار wrap around می‌شود).

```csharp
int max = int.MaxValue;
int result = max + 1; // unchecked: result = -2147483648 (wrap around)

checked
{
    int result2 = max + 1; // OverflowException!
}

int result3 = checked(max + 1); // OverflowException!
```

### Unchecked

```csharp
unchecked
{
    int result = int.MaxValue + 1; // -2147483648 (بدون Exception)
}
```

---

## 18. Checked Operator در C# مدرن

> **نسخه:** C# 11+ / .NET 7+

از C# 11، می‌توانید نسخه `checked` یک Operator را تعریف کنید:

```csharp
public readonly struct Note
{
    public int Value { get; } // 0-127 (MIDI range)

    public Note(int value) => Value = value;

    // نسخه Unchecked (پیش‌فرض)
    public static Note operator +(Note left, int right) =>
        new((left.Value + right) % 128); // Wrap around

    // نسخه Checked (C# 11+)
    public static Note operator checked +(Note left, int right)
    {
        int result = left.Value + right;
        if (result is < 0 or > 127)
            throw new OverflowException($"Note value {result} is out of MIDI range (0-127)");
        return new(result);
    }
}

// استفاده
var note = new Note(120);

// Unchecked context (پیش‌فرض)
var n1 = note + 20; // Note(12) → wrap around

// Checked context
checked
{
    var n2 = note + 20; // OverflowException!
}
```

### چرا این قابلیت اضافه شده؟

قبل از C# 11، Typeهای سفارشی نمی‌توانستند رفتار متفاوتی در `checked` و `unchecked` داشته باشند. این قابلیت به‌ویژه برای **Generic Math** (`INumber<T>`) ضروری بود، چون الگوریتم‌های عددی Generic باید بتوانند Overflow را کنترل کنند.

### چه زمانی نسخه Checked فراخوانی می‌شود؟

- در `checked` block یا expression
- در پروژه‌ای که `<CheckForOverflowUnderflow>true</CheckForOverflowUnderflow>` تنظیم شده
- وقتی Compiler صریحاً `checked` را درخواست کند

---

# بخش ششم — Operator از دید Compiler

## 19. Operator Overloading از دید Roslyn

Roslyn (C# Compiler) هنگام مشاهده `a + b` مراحل زیر را طی می‌کند:

### 1. Syntax Analysis

```
Source: a + b
  ↓
SyntaxTree:
  BinaryExpression (AddExpression)
    ├── Left: IdentifierName "a"
    ├── OperatorToken: PlusToken
    └── Right: IdentifierName "b"
```

### 2. Semantic Analysis

```
Type(a) = Money
Type(b) = Money
  ↓
Symbol Resolution:
  a → LocalSymbol of type Money
  b → LocalSymbol of type Money
```

### 3. Operator Resolution

Roslyn مجموعه‌ای از **Candidate Operators** را جمع‌آوری می‌کند:

```
Candidates:
  1. Built-in: int + int, long + long, double + double, ...
  2. User-defined: Money.operator +(Money, Money) ← در Type Money
  3. Lifted: (Money?) + (Money?) ← اگر Nullable باشد
```

### 4. Overload Resolution

```
Best Candidate Selection:
  - Money.operator +(Money, Money) → Exact match ✅
  - Built-in operators → نیاز به Conversion دارند ❌
  → Winner: Money.operator +(Money, Money)
```

### 5. تولید IL

```
IL:
  ldloc.0  // load a
  ldloc.1  // load b
  call valuetype Money Money::op_Addition(valuetype Money, valuetype Money)
  stloc.2  // store result
```

---

## 20. Operator از دید IL و CLR

> **نکته کلیدی:** CLR مفهومی به نام "Operator" در سطح C# ندارد. تمام Operatorها به **Static Methodهای ویژه** با نام‌گذاری خاص (`op_*`) تبدیل می‌شوند.

### نگاشت Operator به Method نام IL

| C# Operator | IL Method Name |
|-------------|---------------|
| `+` (binary) | `op_Addition` |
| `-` (binary) | `op_Subtraction` |
| `*` | `op_Multiply` |
| `/` | `op_Division` |
| `%` | `op_Modulus` |
| `-` (unary) | `op_UnaryNegation` |
| `+` (unary) | `op_UnaryPlus` |
| `++` | `op_Increment` |
| `--` | `op_Decrement` |
| `!` | `op_LogicalNot` |
| `~` | `op_OnesComplement` |
| `==` | `op_Equality` |
| `!=` | `op_Inequality` |
| `<` | `op_LessThan` |
| `>` | `op_GreaterThan` |
| `<=` | `op_LessThanOrEqual` |
| `>=` | `op_GreaterThanOrEqual` |
| `&` | `op_BitwiseAnd` |
| `\|` | `op_BitwiseOr` |
| `^` | `op_ExclusiveOr` |
| `<<` | `op_LeftShift` |
| `>>` | `op_RightShift` |
| `>>>` | `op_UnsignedRightShift` |
| `true` | `op_True` |
| `false` | `op_False` |
| `implicit` | `op_Implicit` |
| `explicit` | `op_Explicit` |

### مثال Pseudo-IL

```csharp
// C# Source
Money result = a + b;
```

```il
// IL (Pseudo)
.method public hidebysig specialname static
    valuetype Money op_Addition(
        valuetype Money left,
        valuetype Money right) cil managed
{
    // ... body ...
}

// در محل استفاده:
ldloc.0  // a
ldloc.1  // b
call valuetype Money Money::op_Addition(valuetype Money, valuetype Money)
stloc.2  // result
```

> **نکته:** Attribute `specialname` در IL به CLR و ابزارها (مثل IntelliSense) می‌گوید این Method یک Operator است و نباید به‌عنوان Method معمولی نمایش داده شود.

### Built-in vs User-defined در IL

```il
// Built-in int + int:
ldloc.0
ldloc.1
add              // IL instruction مستقیم

// User-defined Money + Money:
ldloc.0
ldloc.1
call valuetype Money Money::op_Addition(...)  // Method call
```

---

## 21. Operator از دید JIT

### JIT چه چیزی را می‌بیند؟

JIT (Just-In-Time Compiler) کد IL را به Machine Code تبدیل می‌کند. JIT **مفهوم C# Operator را نمی‌شناسد**. آنچه JIT می‌بیند:

- برای Built-in: IL instructions مانند `add`, `sub`, `mul`
- برای User-defined: یک `call` به یک Static Method (`op_Addition`)

### Inlining

> **نکته:** Inlining یک **Optimization وابسته به شرایط** است و قطعی نیست.

JIT **ممکن است** Methodهای کوچک Operator را Inline کند:

```
// قبل از Inlining:
call Money::op_Addition(Money, Money)

// بعد از Inlining (اگر JIT تصمیم بگیرد):
// کد بدنه op_Addition مستقیماً در محل call قرار می‌گیرد
// → حذف overhead فراخوانی Method
```

**شرایط Inlining (به‌طور کلی):**
- Method کوچک باشد (معمولاً < 32 بایت IL)
- حلقه یا branching پیچیده نداشته باشد
- در Tier 1 JIT (Optimized) بیشتر رخ می‌دهد
- `MethodImplOptions.AggressiveInlining` می‌تواند کمک کند

### Constant Propagation

> **وابسته به شرایط JIT.**

اگر Operandها Compile-time constant باشند، JIT **ممکن است** محاسبه را در زمان کامپایل انجام دهد.

### Dead Code Elimination

> **وابسته به شرایط JIT.**

اگر نتیجه Operator استفاده نشود، JIT **ممکن است** کل محاسبه را حذف کند.

### Devirtualization

> **مربوط به Static Polymorphism (بخش هشتم).**

برای `static abstract` members در Generic Code، JIT در زمان Runtime (وقتی Type واقعی مشخص شد) **ممکن است** فراخوانی را Devirtualize کند و به یک Direct Call تبدیل کند.

### خلاصه قطعی vs وابسته به شرایط

| Optimization | وضعیت |
|-------------|-------|
| Built-in `add` → Machine `add` | قطعی |
| User-defined `call op_Addition` | قطعی (حداقل یک call) |
| Inlining | وابسته به شرایط JIT |
| Constant Propagation | وابسته به شرایط JIT |
| Dead Code Elimination | وابسته به شرایط JIT |
| Devirtualization (Generic) | وابسته به شرایط JIT و Type |

---

## 22. Performance و Memory

### هزینه Operator Overloading

| عامل | Built-in Operator | User-defined Operator |
|------|------------------|----------------------|
| IL | `add`, `sub` (دستور مستقیم) | `call op_Addition` |
| Method Call Overhead | ندارد | دارد (مگر Inline شود) |
| Allocation | معمولاً ندارد | بستگی به Return Type دارد |

### Value Type vs Reference Type

```csharp
// Value Type (struct) - معمولاً بهتر
public static Point operator +(Point a, Point b) =>
    new(a.X + b.X, a.Y + b.Y);
// → Stack allocation, بدون GC pressure

// Reference Type (class) - ممکن است allocation داشته باشد
public static Vector operator +(Vector a, Vector b) =>
    new(a.X + b.X, a.Y + b.Y);
// → Heap allocation, GC pressure
```

### ایجاد Object جدید در Operator

هر بار که `a + b` را روی یک `class` صدا می‌زنید، یک شیء جدید در Heap ساخته می‌شود:

```csharp
// در یک حلقه بزرگ:
Vector sum = Vector.Zero;
for (int i = 0; i < 1_000_000; i++)
{
    sum = sum + vectors[i]; // هر بار یک Vector جدید در Heap!
}
// → 1,000,000 allocation → فشار روی GC
```

**راه‌حل:** از `struct` استفاده کنید یا `ref struct` و `Span<T>` برای سناریوهای Performance-critical.

### ⚠️ هشدار مهم درباره Performance

> **هرگز** بدون اندازه‌گیری واقعی درباره Performance ادعای مطلق نکنید. نتایج به شدت به شرایط (CPU، JIT version، Tier، Inlining decisions، workload) وابسته‌اند. برای اندازه‌گیری دقیق از **BenchmarkDotNet** استفاده کنید.

```csharp
[MemoryDiagnoser]
public class OperatorBenchmarks
{
    [Benchmark]
    public Point StructAdd()
    {
        var a = new Point(1, 2);
        var b = new Point(3, 4);
        return a + b;
    }

    [Benchmark]
    public VectorClass ClassAdd()
    {
        var a = new VectorClass(1, 2);
        var b = new VectorClass(3, 4);
        return a + b;
    }
}
```

---

# بخش هفتم — اصول طراحی Operator

## 23. Best Practices

### ۱. قابل پیش‌بینی باشید

`+` باید جمع کند، نه تفریق! Developerها انتظار دارند Operatorها رفتار استاندارد داشته باشند.

### ۲. Semantic واضح داشته باشید

```csharp
// ✅ خوب - Semantic واضح
public static Money operator +(Money a, Money b) => ...;

// ❌ بد - Semantic مبهم
public static Document operator +(Document a, Document b) => ...;
// Merge? Append? Concatenate? Union?
```

### ۳. Pure باشید (بدون Side Effect)

```csharp
// ✅ خوب - Pure
public static Point operator +(Point a, Point b) =>
    new(a.X + b.X, a.Y + b.Y);

// ❌ بد - Side Effect
public static Point operator +(Point a, Point b)
{
    Logger.Log("Addition happened!"); // Side Effect!
    Database.Save(a); // Side Effect!
    return new(a.X + b.X, a.Y + b.Y);
}
```

### ۴. با انتظار Developer سازگار باشید

```csharp
// ✅ خوب
public static bool operator ==(Money a, Money b) =>
    a.Amount == b.Amount && a.Currency == b.Currency;

// ❌ بد
public static bool operator ==(Money a, Money b) =>
    a.Amount == b.Amount; // Currency را نادیده می‌گیرد! غیرمنتظره!
```

### مثال Operator خوب و بد

```csharp
// ✅ خوب: Fraction + Fraction
public static Fraction operator +(Fraction a, Fraction b) =>
    new(a.N * b.D + b.N * a.D, a.D * b.D);
// هر ریاضی‌دانی این را می‌فهمد

// ❌ بد: Employee + Employee
public static Employee operator +(Employee a, Employee b) =>
    new Manager(a, b);
// چه معنایی دارد؟ کاملاً غیرمنتظره!
```

---

## 24. چه زمانی Operator Overloading نکنیم؟

| دلیل | توضیح | مثال جایگزین |
|------|-------|-------------|
| رفتار پیچیده | Operator باید ساده و سریع قابل فهم باشد | `a.MergeWith(b)` |
| Side Effect | Operator نباید وضعیت خارجی تغییر دهد | `a.SaveToDatabase()` |
| عملیات Business | قوانین تجاری پیچیده در Operator جایی ندارند | `order.ApplyDiscount(coupon)` |
| Semantic مبهم | اگر `+` می‌تواند چند معنا داشته باشد | `list.Union(other)` |
| تغییر رفتار در آینده | Operatorها API عمومی هستند و تغییر آنها Breaking Change است | Method قابل Versioning |
| Method نام‌دار خواناتر است | گاهی نام Method اطلاعات بیشتری می‌دهد | `money.ConvertTo("EUR")` |

### مقایسه

```csharp
// ❌ مبهم
var result = doc1 + doc2;

// ✅ واضح
var result = doc1.MergeWith(doc2);
var result = doc1.Append(doc2);
var result = doc1.Concatenate(doc2);
```

---

# بخش هشتم — Static Polymorphism

## 25. Static Polymorphism چیست؟

### تعریف

**Static Polymorphism** (Compile-time Polymorphism) به معنای انتخاب رفتار صحیح در **زمان کامپایل** است، نه در زمان اجرا. این در مقابل **Runtime Polymorphism** (Dynamic Polymorphism) قرار می‌گیرد که از `virtual`/`override` و Interface dispatch استفاده می‌کند.

### مقایسه

```
Runtime / Instance Polymorphism
     ↓
virtual / override / interface
     ↓
Dispatch در زمان اجرا (vtable / interface table)
     ↓
Overhead: Indirect call, ممکن است Devirtualize نشود

Static Polymorphism
     ↓
compile-time resolution
     ↓
generics / overload resolution / static abstract members
     ↓
Dispatch در زمان کامپایل یا JIT
     ↓
Overhead: معمولاً Direct call (بعد از JIT optimization)
```

### مثال ساده

```csharp
// Runtime Polymorphism
public interface IShape
{
    double Area(); // virtual dispatch
}

public class Circle : IShape
{
    public double Radius { get; set; }
    public double Area() => Math.PI * Radius * Radius;
}

// Static Polymorphism (با Generics)
public static T Add<T>(T a, T b) where T : IAddable<T>
{
    return a + b; // در زمان JIT، Type واقعی T مشخص است
                  // → Direct call به op_Addition
}
```

---

## 26. Static Abstract Interface Members

> **نسخه:** C# 11 / .NET 7+

قبل از C# 11، Interface فقط می‌توانست Memberهای Instance تعریف کند. از C# 11، Interface می‌تواند `static abstract` و `static virtual` Member داشته باشد.

### `static abstract`

یعنی: «هر Typeای که این Interface را Implement می‌کند، **باید** این Static Member را تعریف کند.»

### `static virtual`

یعنی: «هر Typeای که این Interface را Implement می‌کند، **می‌تواند** این Static Member را Override کند، وگرنه نسخه پیش‌فرض استفاده می‌شود.»

### مثال

```csharp
// C# 11+ / .NET 7+
public interface IAddable<T> where T : IAddable<T>
{
    static abstract T operator +(T left, T right);
}
```

تحلیل خط‌به‌خط:

| بخش | توضیح |
|-----|-------|
| `public interface` | تعریف Interface |
| `IAddable<T>` | Generic Interface با Type Parameter |
| `where T : IAddable<T>` | Constraint: T باید خودش IAddable باشد (CRTP pattern) |
| `static` | این Member متعلق به Type است، نه Instance |
| `abstract` | Type پیاده‌ساز **باید** این را تعریف کند |
| `T operator +(T left, T right)` | Operator `+` که دو T می‌گیرد و T برمی‌گرداند |

---

## 27. Static Abstract Operator

### تعریف در Interface

```csharp
public interface IAddable<T> where T : IAddable<T>
{
    static abstract T operator +(T left, T right);
}
```

### پیاده‌سازی در Type

```csharp
public record struct Point(int X, int Y) : IAddable<Point>
{
    public static Point operator +(Point left, Point right) =>
        new(left.X + right.X, left.Y + right.Y);
}
```

### استفاده در Generic Code

```csharp
public static T Sum<T>(T[] values) where T : IAddable<T>
{
    if (values.Length == 0)
        throw new ArgumentException("Array cannot be empty");

    T total = values[0];
    for (int i = 1; i < values.Length; i++)
    {
        total += values[i]; // Compiler می‌داند T دارای operator + است
    }
    return total;
}

// استفاده
var points = new[] { new Point(1, 2), new Point(3, 4), new Point(5, 6) };
var sum = Sum(points); // Point(9, 12)
```

> **نکته:** در مثال بالا، `T` باید یک مقدار اولیه (Identity Element) داشته باشد. در عمل، از `IAdditionOperators<T, T, T>` که در `System.Numerics` تعریف شده و `AdditiveIdentity` دارد استفاده می‌شود.

---

# بخش نهم — Generic Math

## 28. Generic Math

### مشکل قبل از Generic Math

قبل از .NET 7، اگر می‌خواستید یک الگوریتم عددی Generic بنویسید، با مشکل مواجه می‌شدید:

```csharp
// ❌ نمی‌توانید بنویسید:
static T Sum<T>(T[] values)
{
    T total = 0; // ❌ 0 از نوع T نیست
    foreach (var v in values)
        total += v; // ❌ Compiler نمی‌داند T دارای + هست
    return total;
}

// ✅ مجبور بودید برای هر Type بنویسید:
static int Sum(int[] values) { ... }
static double Sum(double[] values) { ... }
static decimal Sum(decimal[] values) { ... }
static float Sum(float[] values) { ... }
```

### راه‌حل: Generic Math با `static abstract`

از .NET 7، Interfaceهای عددی مانند `INumber<T>` با استفاده از `static abstract` members، امکان نوشتن الگوریتم‌های عددی Generic را فراهم کردند.

---

## 29. `INumber<T>`

### تعریف

`System.Numerics.INumber<T>` یک Interface جامع است که تمام عملیات عددی را برای یک Type تعریف می‌کند. این Interface سلسله‌مراتبی از Interfaceهای کوچک‌تر را به ارث می‌برد:

```
INumber<T>
├── INumberBase<T>
│   ├── IAdditionOperators<T, T, T>
│   ├── ISubtractionOperators<T, T, T>
│   ├── IMultiplyOperators<T, T, T>
│   ├── IDivisionOperators<T, T, T>
│   ├── IComparisonOperators<T, T, bool>
│   ├── IEqualityOperators<T, T, bool>
│   └── ...
├── IComparable<T>
├── IEquatable<T>
└── ...
```

### مثال عملی

```csharp
// C# 11+ / .NET 7+
using System.Numerics;

static T Sum<T>(params T[] values) where T : INumber<T>
{
    T total = T.Zero; // ✅ INumber<T> دارای static abstract Zero است

    foreach (var value in values)
    {
        total += value; // ✅ INumber<T> دارای operator + است
    }

    return total;
}

// استفاده
int intSum = Sum(1, 2, 3, 4, 5);           // 15
double dblSum = Sum(1.5, 2.5, 3.0);        // 7.0
decimal decSum = Sum(100m, 200m, 300m);    // 600m
```

### تحلیل خط‌به‌خط

| خط | توضیح |
|----|-------|
| `where T : INumber<T>` | T باید یک Type عددی باشد که `INumber<T>` را implement کند |
| `T.Zero` | `static abstract` property که مقدار صفر Type را برمی‌گرداند |
| `total += value` | از `operator +` تعریف‌شده در `IAdditionOperators<T,T,T>` استفاده می‌شود |
| `return total` | نتیجه از همان Type T است |

### مثال پیشرفته‌تر: Generic Average

```csharp
static T Average<T>(T[] values)
    where T : INumber<T>
{
    if (values.Length == 0)
        throw new ArgumentException("Cannot average empty array");

    T sum = T.Zero;
    foreach (var v in values)
        sum += v;

    // تبدیل int به T برای تقسیم
    return sum / T.CreateChecked(values.Length);
}
```

---

## 30. Static Polymorphism از Roslyn تا CPU

### مسیر کامل یک Generic Method با Static Abstract Operator

```
Source Code:
  static T Sum<T>(T[] values) where T : INumber<T>
  { T total = T.Zero; total += value; ... }
      ↓
[Roslyn - Syntax Analysis]
  → Generic Method Declaration
  → Generic Constraint: INumber<T>
      ↓
[Roslyn - Semantic Analysis]
  → T.Zero → static abstract property در INumber<T>
  → total += value → operator + از IAdditionOperators<T,T,T>
  → Constraint satisfaction check
      ↓
[Roslyn - IL Generation]
  → constrained. prefix + callvirt برای static abstract members
  → IL: constrained.!T callvirt T::get_Zero()
  → IL: constrained.!T callvirt T::op_Addition(T, T)
      ↓
[CLR - Assembly Loading]
  → Type metadata بارگذاری می‌شود
  → Generic Method Definition ثبت می‌شود
      ↓
[JIT - Runtime Compilation]
  → وقتی Sum<int>() برای اولین بار صدا زده می‌شود:
    → JIT Type واقعی T=int را می‌بیند
    → Devirtualization: constrained call → direct call
    → int::op_Addition → inline → machine ADD instruction
      ↓
[Machine Code]
  → mov eax, [esi]   ; load value
  → add edi, eax     ; total += value
  → (بدون هیچ call یا vtable lookup!)
      ↓
[CPU]
  → دستور ADD مستقیماً روی رجیسترها اجرا می‌شود
```

### تفاوت با سایر مکانیسم‌ها

| مکانیسم | Dispatch | Overhead | Type Safety | زمان تصمیم‌گیری |
|---------|----------|----------|-------------|----------------|
| **Static Polymorphism (Generic + static abstract)** | Direct call (بعد از JIT) | حداقل | Compile-time | Compile-time + JIT-time |
| **Interface Dispatch (virtual)** | Indirect call (vtable) | متوسط | Compile-time | Runtime |
| **Virtual Dispatch (override)** | Indirect call (vtable) | متوسط | Compile-time | Runtime |
| **Reflection** | Dynamic invoke | بالا | Runtime | Runtime |
| **Dynamic (DLR)** | Dynamic dispatch | بسیار بالا | Runtime | Runtime |
| **Delegate** | Indirect call | متوسط | Compile-time | Runtime |

### مزیت کلیدی Static Polymorphism

با Static Polymorphism، شما **هم** Type Safety زمان کامپایل دارید و **هم** Performance نزدیک به کد غیر-Generic (بعد از JIT optimization). این ترکیبی است که با Runtime Polymorphism ممکن نیست.

---

# بخش‌های تکمیلی

## Cheat Sheet

### Operator Overloading
```csharp
public static ReturnType operator OpSymbol(ParamType1 a, ParamType2 b) { ... }
// باید public static باشد
// حداقل یکی از پارامترها باید Type تعریف‌کننده باشد
```

### Equality
```csharp
public static bool operator ==(T a, T b) { ... }
public static bool operator !=(T a, T b) => !(a == b);
public override bool Equals(object? obj) { ... }
public override int GetHashCode() => HashCode.Combine(...);
// همیشه == و != را با هم تعریف کنید
// Equals و GetHashCode را هم Override کنید
```

### Comparison
```csharp
public static bool operator <(T a, T b) => a.CompareTo(b) < 0;
public static bool operator >(T a, T b) => a.CompareTo(b) > 0;
// < و > را با هم، <= و >= را با هم تعریف کنید
```

### Conversion
```csharp
public static implicit operator Target(Source s) { ... } // بدون Cast
public static explicit operator Target(Source s) { ... } // با Cast
// implicit فقط برای تبدیل‌های امن
```

### Checked Operators (C# 11+)
```csharp
public static T operator checked +(T a, T b) { ... }
```

### Static Abstract Members (C# 11+)
```csharp
public interface IMyOp<T> where T : IMyOp<T>
{
    static abstract T operator +(T a, T b);
}
```

### Generic Math (.NET 7+)
```csharp
static T Sum<T>(T[] values) where T : INumber<T>
{
    T total = T.Zero;
    foreach (var v in values) total += v;
    return total;
}
```

### Static Polymorphism
- انتخاب رفتار در Compile-time/JIT-time
- بدون vtable overhead
- با `static abstract` interface members و Generics

---

## Comparison Table

| ویژگی | Method Overloading | Operator Overloading | Runtime Polymorphism | Static Polymorphism | Generics | Reflection | Dynamic |
|-------|-------------------|---------------------|---------------------|--------------------|---------|-----------|---------|
| زمان تصمیم | Compile-time | Compile-time | Runtime | Compile-time + JIT | Compile-time + JIT | Runtime | Runtime |
| Dispatch | Direct call | Direct call | Indirect (vtable) | Direct (بعد از JIT) | Direct (بعد از JIT) | Invoke | DLR |
| Type Safety | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Performance | بالا | بالا | متوسط | بالا | بالا | پایین | بسیار پایین |
| خوانایی | بالا | بالا (برای ریاضی) | بالا | بالا | بالا | پایین | متوسط |
| Flexibility | متوسط | محدود | بالا | بالا | بالا | بسیار بالا | بالا |
| Syntax | `a.Add(b)` | `a + b` | `a.Method()` | `Method<T>(a)` | `Method<T>(a)` | `type.InvokeMember(...)` | `a + b` (runtime) |
| Use Case | API عمومی | Typeهای ریاضی/عددی | OOP, Plugin systems | Generic algorithms | Type-safe containers | Metaprogramming | Interop, scripting |

---

## Common Mistakes

### ۱. تعریف `==` بدون `!=`
```csharp
// ❌ CS0216
public static bool operator ==(Money a, Money b) => ...;
// باید != هم تعریف شود
```

### ۲. تعریف `<` بدون `>`
```csharp
// ❌ CS0216
public static bool operator <(Money a, Money b) => ...;
// باید > هم تعریف شود
```

### ۳. فراموش کردن `Equals` و `GetHashCode` هنگام Overload کردن `==`
```csharp
// ❌ رفتار ناسازگار
public static bool operator ==(Money a, Money b) => ...;
// Dictionary و HashSet از Equals/GetHashCode استفاده می‌کنند!
```

### ۴. استفاده از `==` داخل `operator ==` برای class
```csharp
// ❌ Infinite Recursion → StackOverflowException
public static bool operator ==(MyClass a, MyClass b)
{
    if (a == null) return b == null; // دوباره operator == صدا زده می‌شود!
    return a.Equals(b);
}
// ✅ از `a is null` یا `ReferenceEquals(a, null)` استفاده کنید
```

### ۵. Implicit Conversion که اطلاعات از دست می‌دهد
```csharp
// ❌ از دست رفتن اطلاعات
public static implicit operator int(Money m) => (int)m.Amount;
// decimal → int ممکن است truncation داشته باشد
```

### ۶. Side Effect در Operator
```csharp
// ❌ Operator نباید Side Effect داشته باشد
public static Order operator +(Order a, Order b)
{
    Database.MergeOrders(a, b); // Side Effect!
    return new Order(...);
}
```

### ۷. Semantic غیرمنتظره
```csharp
// ❌ + که تفریق کند!
public static Point operator +(Point a, Point b) =>
    new(a.X - b.X, a.Y - b.Y);
```

### ۸. Mutable struct با Operator
```csharp
// ❌ رفتار گیج‌کننده
public struct Counter
{
    public int Value;
    public static Counter operator ++(Counter c)
    {
        c.Value++; // روی copy کار می‌کند
        return c;
    }
}
```

### ۹. انتظار `is`/`as` برای Custom Conversion
```csharp
Celsius c = new(100);
// ❌ این false برمی‌گرداند
if (c is Kelvin) { ... }
```

### ۱۰. تعریف Operator غیر `public`
```csharp
// ❌ CS0558
private static Point operator +(Point a, Point b) => ...;
```

### ۱۱. تعریف Operator غیر `static`
```csharp
// ❌ CS0558
public Point operator +(Point a, Point b) => ...;
```

### ۱۲. Operator برای Type نامرتبط
```csharp
public struct Money
{
    // ❌ CS0563 - هیچ پارامتری Money نیست
    public static int operator +(int a, int b) => a + b;
}
```

### ۱۳. `GetHashCode` ناسازگار با `Equals`
```csharp
// ❌ دو شیء برابر با HashCode متفاوت
public override bool Equals(object? obj) => obj is Money m && Amount == m.Amount;
public override int GetHashCode() => Currency.GetHashCode(); // Currency در Equals نیست!
```

### ۱۴. استفاده از Mutable Field در `GetHashCode`
```csharp
// ❌ اگر Value تغییر کند، HashCode تغییر می‌کند
public int Value { get; set; }
public override int GetHashCode() => Value.GetHashCode();
```

### ۱۵. فراموش کردن `IEquatable<T>`
```csharp
// ❌ Boxing برای structها هنگام Equals(object)
public override bool Equals(object? obj) => ...;
// ✅ IEquatable<T> را هم implement کنید
```

### ۱۶. `checked` Operator بدون نسخه `unchecked`
```csharp
// ❌ باید هر دو نسخه را تعریف کنید
public static Note operator checked +(Note a, int b) { ... }
// باید operator + (بدون checked) هم تعریف شود
```

### ۱۷. استفاده از `dynamic` به‌جای Generic Math
```csharp
// ❌ Performance پایین، بدون Type Safety
static dynamic Sum(dynamic[] values) { ... }
// ✅ از INumber<T> استفاده کنید
```

### ۱۸. عدم استفاده از `record struct` برای Value Types
```csharp
// ⚠️ باید دستی Equals, GetHashCode, ==, != بنویسید
public struct Money { ... }
// ✅ record struct خودش همه اینها را تولید می‌کند
public readonly record struct Money(decimal Amount, string Currency);
```

### ۱۹. تعریف `static abstract` بدون Generic Constraint
```csharp
// ❌ بدون constraint، Compiler نمی‌تواند Operator را resolve کند
public interface IAddable
{
    static abstract IAddable operator +(IAddable a, IAddable b);
}
// ✅ از CRTP pattern استفاده کنید: IAddable<T> where T : IAddable<T>
```

### ۲۰. نادیده گرفتن `checked` context در Generic Math
```csharp
// ⚠️ ممکن است Overflow بدون خطا رخ دهد
static T Sum<T>(T[] values) where T : INumber<T>
{
    T total = T.Zero;
    foreach (var v in values) total += v; // unchecked!
    return total;
}
// ✅ اگر Overflow مهم است، از checked استفاده کنید
```

---

## Exercises

### Beginner

**تمرین ۱: Point Addition**
یک `readonly record struct Point(int X, int Y)` بسازید که `+` و `-` (binary و unary) را Overload کند.

**تمرین ۲: Money Equality**
یک `readonly record struct Money(decimal Amount, string Currency)` بسازید که `==` و `!=` را Overload کند. `Equals` و `GetHashCode` را هم implement کنید.

**تمرین ۳: Temperature Conversion**
دو struct `Celsius` و `Fahrenheit` بسازید. `implicit` conversion از `Celsius` به `Fahrenheit` و `explicit` conversion از `Fahrenheit` به `Celsius` تعریف کنید.

### Intermediate

**تمرین ۴: Vector3D**
یک `readonly record struct Vector3D(double X, double Y, double Z)` بسازید که:
- `+`, `-` (binary و unary), `*` (scalar multiplication)
- `==`, `!=`
- `Magnitude` property
- `Dot Product` به‌عنوان `*` بین دو Vector

**تمرین ۵: Fraction**
یک `readonly struct Fraction(int Numerator, int Denominator)` بسازید که:
- `+`, `-`, `*`, `/`
- `==`, `!=`, `<`, `>`, `<=`, `>=`
- `implicit` conversion از `int`
- `explicit` conversion به `double`
- ساده‌سازی خودکار کسر (GCD)

**تمرین ۶: Money Comparison**
به struct `Money` از تمرین ۲، Operatorهای `<`, `>`, `<=`, `>=` و `IComparable<Money>` اضافه کنید.

**تمرین ۷: Complex Number**
یک `readonly record struct Complex(double Real, double Imaginary)` بسازید که:
- `+`, `-`, `*`, `/`
- `==`, `!=`
- `implicit` از `double`
- `Conjugate` property
- `Magnitude` property

### Advanced

**تمرین ۸: Checked Operator**
یک `readonly struct Byte256(byte Value)` بسازید که:
- `operator +` با wrap-around (mod 256)
- `operator checked +` که در صورت Overflow exception بدهد

**تمرین ۹: Generic IAddable**
یک Interface `IAddable<T>` با `static abstract operator +` بسازید. آن را برای `Point` و `Money` implement کنید. یک Generic `Sum<T>` بنویسید.

**تمرین ۱۰: Generic Math - Statistics**
با استفاده از `INumber<T>`، توابع Generic زیر را بنویسید:
- `Mean<T>`
- `Variance<T>`
- `StandardDeviation<T>`

**تمرین ۱۱: Matrix\<T\>**
یک Generic `Matrix<T>` بسازید که `where T : INumber<T>` باشد و `+`, `*` (matrix multiplication) را Overload کند.

**تمرین ۱۲: Static Abstract Interface Design**
یک Interface `IMeasurable<T>` طراحی کنید که شامل:
- `static abstract T Zero`
- `static abstract T operator +(T a, T b)`
- `static abstract T operator -(T a, T b)`
- `static abstract bool operator <(T a, T b)`
باشد. آن را برای `Distance` و `Duration` implement کنید.

**تمرین ۱۳: Operator Resolution Challenge**
کد زیر را تحلیل کنید و بگویید کدام Operator فراخوانی می‌شود و چرا:
```csharp
public struct A { public static A operator +(A a, B b) => ...; }
public struct B { public static A operator +(A a, B b) => ...; }
A x = new(); B y = new();
var z = x + y; // کدام؟
```

**تمرین ۱۴: Performance Benchmark**
با BenchmarkDotNet، Performance `+` را برای `struct Point` در مقابل `class Point` مقایسه کنید. نتایج را تحلیل کنید.

**تمرین ۱۵: Generic Math Pipeline**
یک Pipeline پردازش سیگنال Generic بنویسید که:
- `T[] Normalize<T>(T[] signal) where T : IFloatingPoint<T>`
- `T[] Scale<T>(T[] signal, T factor) where T : INumber<T>`
- `T RMS<T>(T[] signal) where T : IFloatingPoint<T>`

---

## Interview Questions

### مقدماتی

**سؤال ۱:** Operator Overloading چیست و چرا در C# استفاده می‌شود؟
**پاسخ:** تعریف رفتار جدید برای Operatorهای موجود برای Typeهای سفارشی. هدف: خوانایی و یکپارچگی با Typeهای Built-in.

**سؤال ۲:** آیا Operator Overloading در C# باید `static` باشد؟ چرا؟
**پاسخ:** بله. چون Operatorها قبل از وجود Instance باید قابل استفاده باشند و در سطح CLR به‌صورت Static Methodهای `op_*` پیاده‌سازی می‌شوند.

**سؤال ۳:** کدام Operatorها قابل Overload نیستند؟
**پاسخ:** `=`, `&&`, `||`, `[]`, `()`, `?.`, `??`, `is`, `as`, `new`, `typeof`, `.`, `->`, `=>`.

**سؤال ۴:** چرا `==` و `!=` باید با هم تعریف شوند؟
**پاسخ:** Compiler اجبار می‌کند (CS0216) تا از ناسازگاری منطقی جلوگیری شود.

**سؤال ۵:** تفاوت `implicit` و `explicit` conversion چیست؟
**پاسخ:** `implicit` بدون Cast و فقط برای تبدیل‌های امن. `explicit` با Cast و برای تبدیل‌هایی که ممکن است اطلاعات از دست برود.

**سؤال ۶:** آیا `is` و `as` از Custom Conversion پشتیبانی می‌کنند؟
**پاسخ:** خیر. فقط Inheritance و Interface را بررسی می‌کنند.

**سؤال ۷:** `+=` چگونه با `+` مرتبط است؟
**پاسخ:** Compiler `a += b` را به `a = a + b` تبدیل می‌کند. اگر `+` Overload شده باشد، `+=` خودکار کار می‌کند.

### متوسط

**سؤال ۸:** هنگام Overload کردن `==` برای یک `class`، چه نکته مهمی باید رعایت شود؟
**پاسخ:** از `is null` یا `ReferenceEquals` استفاده کنید، نه `== null`، چون infinite recursion ایجاد می‌شود.

**سؤال ۹:** رابطه `Equals` و `GetHashCode` چیست؟
**پاسخ:** اگر `a.Equals(b)` → `true`، آنگاه `a.GetHashCode() == b.GetHashCode()` **الزامی** است. عکس آن الزامی نیست.

**سؤال ۱۰:** Hash Collision چیست و چرا اجتناب‌ناپذیر است؟
**پاسخ:** وقتی دو شیء نابرابر HashCode یکسان دارند. اجتناب‌ناپذیر است چون تعداد HashCodeها (`int`) محدود ولی تعداد اشیاء نامحدود است.

**سؤال ۱۱:** چرا Mutable Field برای `GetHashCode` مشکل‌ساز است؟
**پاسخ:** اگر بعد از اضافه شدن به `Dictionary`/`HashSet` مقدار Field تغییر کند، HashCode تغییر می‌کند و شیء در Collection «گم» می‌شود.

**سؤال ۱۲:** `&&` چگونه از `&` و `true`/`false` مشتق می‌شود؟
**پاسخ:** `a && b`: اگر `operator false(a)` → `true`، نتیجه `a` (short-circuit). در غیر این صورت، نتیجه `a & b`.

**سؤال ۱۳:** Operator Overloading در IL چگونه نمایش داده می‌شود؟
**پاسخ:** به‌صورت Static Methodهای `op_*` با attribute `specialname`. مثلاً `+` → `op_Addition`.

**سؤال ۱۴:** آیا JIT مفهوم Operator را می‌شناسد؟
**پاسخ:** خیر. JIT فقط Method call (`call op_Addition`) یا IL instruction (`add`) می‌بیند.

**سؤال ۱۵:** چه زمانی نباید از Operator Overloading استفاده کنیم؟
**پاسخ:** وقتی Semantic مبهم است، Side Effect دارد، عملیات Business پیچیده است، یا Method نام‌دار خواناتر است.

### پیشرفته

**سؤال ۱۶:** Static Polymorphism چیست و چه تفاوتی با Runtime Polymorphism دارد؟
**پاسخ:** Static Polymorphism انتخاب رفتار در Compile-time/JIT-time (با Generics و `static abstract`). Runtime Polymorphism از vtable و indirect call استفاده می‌کند. Static معمولاً Performance بهتری دارد.

**سؤال ۱۷:** `static abstract` Interface Members در چه نسخه‌ای معرفی شدند و چه مشکلی را حل کردند؟
**پاسخ:** C# 11 / .NET 7. امکان تعریف Contract برای Static Members (مثل Operatorها) در Interface را فراهم کردند. مشکل اصلی: عدم امکان نوشتن Generic Math.

**سؤال ۱۸:** `INumber<T>` چگونه کار می‌کند؟
**پاسخ:** یک Interface سلسله‌مراتبی که `static abstract` Operatorها و Properties (مثل `Zero`, `One`) را تعریف می‌کند. تمام Typeهای عددی .NET آن را implement می‌کنند.

**سؤال ۱۹:** مسیر `T.Zero` در یک Generic Method از Source تا CPU را توضیح دهید.
**پاسخ:** Roslyn → IL با `constrained.` prefix → JIT با دانستن Type واقعی → Devirtualization → Direct call یا Inline → Machine code.

**سؤال ۲۰:** `constrained.` IL prefix چیست و چرا برای `static abstract` لازم است؟
**پاسخ:** به JIT می‌گوید که Type واقعی T را در نظر بگیرد و اگر T یک Value Type است، بدون Boxing فراخوانی کند.

**سؤال ۲۱:** تفاوت `static abstract` و `static virtual` در Interface چیست؟
**پاسخ:** `static abstract`: Type پیاده‌ساز **باید** تعریف کند. `static virtual`: Type پیاده‌ساز **می‌تواند** Override کند وگرنه نسخه پیش‌فرض استفاده می‌شود.

**سؤال ۲۲:** آیا Inlining Operatorها قطعی است؟
**پاسخ:** خیر. Inlining یک Optimization وابسته به شرایط JIT است. به اندازه Method، پیچیدگی، و Tier بستگی دارد.

**سؤال ۲۳:** `checked` Operator (C# 11) چه زمانی فراخوانی می‌شود؟
**پاسخ:** در `checked` block/expression یا وقتی پروژه `<CheckForOverflowUnderflow>true</CheckForOverflowUnderflow>` دارد.

**سؤال ۲۴:** CRTP (Curiously Recurring Template Pattern) در C# چیست و چرا در `IAddable<T> where T : IAddable<T>` استفاده می‌شود؟
**پاسخ:** الگویی که Type خودش را به‌عنوان Type Parameter به Interface پاس می‌دهد. این اجازه می‌دهد Operatorها `T` را به‌جای Interface Type بگیرند و برگردانند.

**سؤال ۲۵:** اگر یک `struct` Operator `+` را Overload کند و در یک حلقه ۱ میلیون بار استفاده شود، چه نگرانی‌های Performanceای وجود دارد؟
**پاسخ:** برای `struct`: معمولاً خوب (Stack allocation). اما اگر `struct` بزرگ باشد، Copy overhead دارد. برای `class`: هر بار Heap allocation → GC pressure. باید با BenchmarkDotNet اندازه‌گیری شود.

---

## Real-World Design

### سناریو: سیستم مالی با محاسبات عددی Generic

```csharp
// ============================================
// 1. Money - Operator Overloading (Domain Type)
// ============================================
// چرا Operator Overloading؟ چون Money یک مفهوم عددی/مالی است
// و +، -، ==، < کاملاً Semantic واضح دارند.

public readonly record struct Money(decimal Amount, string Currency)
    : IComparable<Money>, IEquatable<Money>
{
    // --- Arithmetic ---
    public static Money operator +(Money left, Money right)
    {
        EnsureSameCurrency(left, right);
        return new(left.Amount + right.Amount, left.Currency);
    }

    public static Money operator -(Money left, Money right)
    {
        EnsureSameCurrency(left, right);
        return new(left.Amount - right.Amount, left.Currency);
    }

    public static Money operator *(Money money, decimal factor) =>
        new(money.Amount * factor, money.Currency);

    public static Money operator *(decimal factor, Money money) =>
        money * factor;

    // --- Comparison ---
    public int CompareTo(Money other)
    {
        EnsureSameCurrency(this, other);
        return Amount.CompareTo(other.Amount);
    }

    public static bool operator <(Money left, Money right) => left.CompareTo(right) < 0;
    public static bool operator >(Money left, Money right) => left.CompareTo(right) > 0;
    public static bool operator <=(Money left, Money right) => left.CompareTo(right) <= 0;
    public static bool operator >=(Money left, Money right) => left.CompareTo(right) >= 0;

    // --- Conversion ---
    // explicit: چون exchange rate لازم است و تبدیل همیشه ممکن نیست
    public Money ConvertTo(string targetCurrency, decimal exchangeRate) =>
        new(Amount * exchangeRate, targetCurrency);

    // Factory Methods به‌جای implicit conversion
    public static Money FromDollar(decimal amount) => new(amount, "USD");
    public static Money FromEuro(decimal amount) => new(amount, "EUR");

    private static void EnsureSameCurrency(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException(
                $"Cannot operate on {a.Currency} and {b.Currency}");
    }
}

// ============================================
// 2. Point2D - Operator Overloading (Geometry)
// ============================================
// چرا Operator Overloading؟ مفاهیم ریاضی واضح.

public readonly record struct Point2D(double X, double Y)
{
    public static Point2D operator +(Point2D a, Point2D b) =>
        new(a.X + b.X, a.Y + b.Y);

    public static Point2D operator -(Point2D a, Point2D b) =>
        new(a.X - b.X, a.Y - b.Y);

    public static Point2D operator *(Point2D p, double scalar) =>
        new(p.X * scalar, p.Y * scalar);

    public static Point2D operator -(Point2D p) =>
        new(-p.X, -p.Y);

    public double DistanceTo(Point2D other) =>
        Math.Sqrt(Math.Pow(X - other.X, 2) + Math.Pow(Y - other.Y, 2));
}

// ============================================
// 3. Vector3D - Operator Overloading (Physics)
// ============================================

public readonly record struct Vector3D(double X, double Y, double Z)
{
    public static Vector3D operator +(Vector3D a, Vector3D b) =>
        new(a.X + b.X, a.Y + b.Y, a.Z + b.Z);

    public static Vector3D operator *(Vector3D v, double s) =>
        new(v.X * s, v.Y * s, v.Z * s);

    // Dot Product
    public static double operator *(Vector3D a, Vector3D b) =>
        a.X * b.X + a.Y * b.Y + a.Z * b.Z;

    public double Magnitude => Math.Sqrt(this * this);
}

// ============================================
// 4. Temperature - Operator Overloading + Conversion
// ============================================

public readonly record struct Celsius(decimal Value)
{
    public static implicit operator Kelvin(Celsius c) =>
        new(c.Value + 273.15m);

    public static Celsius operator +(Celsius a, Celsius b) =>
        new(a.Value + b.Value);

    public static bool operator <(Celsius a, Celsius b) =>
        a.Value < b.Value;
    public static bool operator >(Celsius a, Celsius b) =>
        a.Value > b.Value;
}

public readonly record struct Kelvin(decimal Value)
{
    public static explicit operator Celsius(Kelvin k) =>
        new(k.Value - 273.15m);
}

// ============================================
// 5. Generic Numeric Algorithm - Static Polymorphism
// ============================================
// چرا Static Polymorphism؟ چون الگوریتم باید برای int, double,
// decimal, float و هر Type عددی دیگر کار کند.
// Operator Overloading به‌تنهایی کافی نیست چون Generic است.

using System.Numerics;

public static class NumericAlgorithms
{
    public static T Sum<T>(ReadOnlySpan<T> values) where T : INumber<T>
    {
        T total = T.Zero;
        foreach (var v in values)
            total += v;
        return total;
    }

    public static T Mean<T>(ReadOnlySpan<T> values) where T : IFloatingPoint<T>
    {
        if (values.IsEmpty) throw new ArgumentException("Empty");
        T sum = T.Zero;
        foreach (var v in values)
            sum += v;
        return sum / T.CreateChecked(values.Length);
    }

    public static T Max<T>(ReadOnlySpan<T> values) where T : INumber<T>
    {
        if (values.IsEmpty) throw new ArgumentException("Empty");
        T max = values[0];
        for (int i = 1; i < values.Length; i++)
            if (values[i] > max) max = values[i];
        return max;
    }

    public static T DotProduct<T>(ReadOnlySpan<T> a, ReadOnlySpan<T> b)
        where T : INumber<T>
    {
        if (a.Length != b.Length) throw new ArgumentException("Length mismatch");
        T result = T.Zero;
        for (int i = 0; i < a.Length; i++)
            result += a[i] * b[i];
        return result;
    }
}

// ============================================
// استفاده
// ============================================
// Operator Overloading:
var price1 = Money.FromDollar(100m);
var price2 = Money.FromDollar(50m);
var total = price1 + price2; // Money(150, "USD")
var isExpensive = total > Money.FromDollar(200m); // false

var p1 = new Point2D(1, 2);
var p2 = new Point2D(3, 4);
var p3 = p1 + p2; // Point2D(4, 6)

Celsius boiling = new(100m);
Kelvin k = boiling; // implicit → Kelvin(373.15)
Celsius back = (Celsius)k; // explicit → Celsius(100)

// Static Polymorphism (Generic Math):
int[] ints = [1, 2, 3, 4, 5];
double[] doubles = [1.5, 2.5, 3.0];
decimal[] decimals = [100m, 200m, 300m];

var intSum = NumericAlgorithms.Sum<int>(ints);         // 15
var dblMean = NumericAlgorithms.Mean<double>(doubles);  // 2.333...
var decMax = NumericAlgorithms.Max<decimal>(decimals);  // 300m
```

### خلاصه تصمیم‌گیری

| Type/الگوریتم | روش | دلیل |
|--------------|-----|------|
| `Money` | Operator Overloading | Domain عددی با Semantic واضح |
| `Point2D` | Operator Overloading | مفهوم ریاضی |
| `Vector3D` | Operator Overloading | مفهوم فیزیکی/ریاضی |
| `Temperature` | Operator Overloading + Conversion | تبدیل بین واحدها |
| `Sum<T>`, `Mean<T>` | Static Polymorphism (Generic Math) | الگوریتم Generic برای تمام اعداد |

---

# منابع

## منابع رسمی

| عنوان | نویسنده/سازمان | موضوع | لینک |
|-------|---------------|-------|------|
| C# Language Reference - Operator Overloading | Microsoft | Operator Overloading syntax و قوانین | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading |
| C# Language Specification | Microsoft / ECMA | مشخصات رسمی زبان C# | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/ |
| C# 11 Feature: Static Abstract Members in Interfaces | Microsoft | Static abstract/virtual interface members | https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/static-virtual-interface-members |
| Generic Math | Microsoft | INumber\<T\> و Generic Math | https://learn.microsoft.com/en-us/dotnet/standard/generics/math |
| System.Numerics.INumber\<T\> API | Microsoft | API Reference | https://learn.microsoft.com/en-us/dotnet/api/system.numerics.inumber-1 |
| .NET Runtime - JIT Optimization | .NET Runtime Team | JIT internals و optimizations | https://github.com/dotnet/runtime/blob/main/docs/design/coreclr/jit/ryujit-overview.md |
| ECMA-335: Common Language Infrastructure | ECMA | CLR و IL specification | https://www.ecma-international.org/publications-and-standards/standards/ecma-335/ |
| Roslyn Compiler Platform | Microsoft | Compiler architecture | https://github.com/dotnet/roslyn |
| C# Feature Specifications | Microsoft | مشخصات قابلیت‌های C# | https://github.com/dotnet/csharplang/tree/main/proposals |

## منابع تکمیلی

| عنوان | نویسنده | موضوع | لینک |
|-------|---------|-------|------|
| C# in Depth (4th Edition) | Jon Skeet | مفاهیم عمیق C# شامل Operator Overloading و Generics | https://csharpindepth.com/ |
| Pro C# 10 with .NET 6 | Andrew Troelsen, Phil Japikse | آموزش جامع C# و .NET | https://www.apress.com/gp/book/9781484278680 |
| CLR via C# (4th Edition) | Jeffrey Richter | CLR internals, JIT, Performance | https://www.microsoftpressstore.com/store/clr-via-c-sharp-9780735667457 |
| Writing High-Performance .NET Code | Ben Watson | Performance, JIT, Memory, GC | https://www.writinghighperf.net/ |
| Pro .NET Memory Management | Konrad Kokosa | Memory management, GC, Value Types | https://prodotnetmemory.com/ |
| BenchmarkDotNet Documentation | .NET Foundation | Performance benchmarking | https://benchmarkdotnet.org/ |
| The C# Programming Language (Annotated) | Anders Hejlsberg et al. | طراحی زبان C# و تصمیمات طراحی | https://www.amazon.com/Programming-Language-Coverage-Microsoft-Technology/dp/0321154916 |
| .NET Blog - Generic Math | Tanner Gooding / Microsoft | معرفی Generic Math | https://devblogs.microsoft.com/dotnet/dotnet-7-generic-math/ |

---

> **توجه:** این آموزش بر اساس C# 14 / .NET 10 نوشته شده است. قابلیت‌هایی مانند `static abstract` Interface Members از C# 11 / .NET 7 و Checked Operators از C# 11 / .NET 7 در دسترس هستند. همیشه نسخه دقیق مورد نیاز هر قابلیت در متن ذکر شده است. برای اطلاعات به‌روز، به [Microsoft Learn](https://learn.microsoft.com/) مراجعه کنید.