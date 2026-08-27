# Tuple و IComparable در #C — راهنمای جامع

> **مخاطب این مقاله:** توسعه‌دهندگانی که با C# آشنا هستند و می‌خواهند رفتار مقایسه‌ای `ValueTuple` را به‌صورت عمیق درک کنند.
> **نسخه‌های پوشش‌داده‌شده:** .NET Framework 4.7+، .NET Core 1.1+، .NET 5/6/7/8/9

---

## فهرست مطالب

1. [مقدمه](#مقدمه)
2. [IComparable چیست؟](#icompurable-چیست)
   - [IComparable غیرعمومی](#icompurable-غیرعمومی)
   - [IComparable<T> عمومی](#icompurablet-عمومی)
   - [IStructuralComparable](#istructuralcomparable)
3. [آیا Tupleها قابلیت مقایسه ترتیبی دارند؟](#آیا-tupleها-قابلیت-مقایسه-ترتیبی-دارند)
4. [مقایسه عناصر Tuple چگونه انجام می‌شود؟](#مقایسه-عناصر-tuple-چگونه-انجام-میشود)
5. [ترتیب مقایسه اعضا (Lexicographical)](#ترتیب-مقایسه-اعضا)
6. [مقایسه Tupleهای عددی](#مقایسه-tupleهای-عددی)
7. [Sort کردن Collectionهای Tuple](#sort-کردن-collectionهای-tuple)
8. [استفاده با SortedSet](#استفاده-با-sortedset)
9. [استفاده با SortedDictionary](#استفاده-با-sorteddictionary)
10. [تفاوت Equality و Ordering](#تفاوت-equality-و-ordering)
11. [نکات مهم](#نکات-مهم)
12. [اشتباهات رایج](#اشتباهات-رایج)
13. [جمع‌بندی](#جمع‌بندی)
14. [منابع رسمی](#منابع-رسمی)

---

## مقدمه

در بسیاری از سناریوها نیاز داریم دو شیء را با هم مقایسه کنیم تا مشخص شود کدام «کوچک‌تر» یا «بزرگ‌تر» است. این کار پایه و اساس مرتب‌سازی (Sort)، جست‌وجوی باینری (Binary Search) و ساختارهای داده‌ای مرتب مانند `SortedSet` و `SortedDictionary` است.

در #C این قابلیت از طریق رابط `IComparable` فراهم می‌شود. یکی از سؤالات جالب این است که آیا `Tuple`ها (به‌ویژه `ValueTuple` که از .NET Framework 4.7 به بعد معرفی شد) این قابلیت را به‌صورت پیش‌فرض دارند یا خیر؟

پاسخ **مثبت** است و در این مقاله به‌صورت کامل بررسی می‌کنیم که این مقایسه چگونه پیاده‌سازی شده است.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## IComparable چیست؟

رابط `IComparable` یک قرارداد (Contract) است که به یک نوع اعلام می‌کند «من می‌توانم خودم را با یک شیء هم‌نوع مقایسه کنم و نتیجه‌ای ترتیبی (Ordering) تولید کنم».

### IComparable غیرعمومی

```csharp
public interface IComparable
{
    int CompareTo(object? obj);
}
```

- **بازگشتی منفی:** `this` کوچک‌تر از `obj` است.
- **بازگشتی صفر:** دو شیء از نظر ترتیب برابرند.
- **بازگشتی مثبت:** `this` بزرگ‌تر از `obj` است.

### IComparable<T> عمومی

```csharp
public interface IComparable<in T>
{
    int CompareTo(T? other);
}
```

نسخهٔ عمومی (Generic) از نوع‌پذیری (Type-Safety) برخوردار است و از Boxing برای انواع مقدار (Value Types) جلوگیری می‌کند.

### IStructuralComparable

این رابط برای مقایسهٔ ساختاری (Structural) به‌کار می‌رود و به شما اجازه می‌دهد `IComparer` دلخواهی را برای مقایسهٔ عناصر داخلی تزریق کنید:

```csharp
public interface IStructuralComparable
{
    int CompareTo(object? other, IComparer comparer);
}
```

`ValueTuple` و `Tuple` هر دو این رابط را پیاده‌سازی می‌کنند.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## آیا Tupleها قابلیت مقایسه ترتیبی دارند؟

بله. هم `System.Tuple` (نوع مرجعی که از .NET 4.0 وجود دارد) و هم `System.ValueTuple` (نوع مقداری که از .NET Framework 4.7 / .NET Core 1.1 اضافه شد) رابط‌های زیر را پیاده‌سازی می‌کنند:

| رابط | `System.Tuple` | `System.ValueTuple` |
|---|---|---|
| `IComparable` | ✅ | ✅ |
| `IComparable<TSelf>` | ✅ | ✅ |
| `IStructuralComparable` | ✅ | ✅ |
| `IEquatable<TSelf>` | ✅ | ✅ |
| `IStructuralEquatable` | ✅ | ✅ |

> 🔖 **نکتهٔ نسخه‌ای:** `ValueTuple` در .NET Framework قبل از 4.7 وجود خارجی ندارد. اگر روی نسخه‌های قدیمی‌تر کار می‌کنید، فقط `System.Tuple` در دسترس است که رفتار مقایسه‌ای یکسانی دارد.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## مقایسه عناصر Tuple چگونه انجام می‌شود؟

مقایسهٔ Tupleها به‌صورت **Lexicographical** (واژه‌نامه‌ای / عنصر به عنصر از چپ به راست) انجام می‌شود. برای هر عضو، از `Comparer<T>.Default` استفاده می‌شود.

پیاده‌سازی داخلی `ValueTuple<T1, T2>.CompareTo` به‌صورت تقریبی چنین است:

```csharp
public int CompareTo(ValueTuple<T1, T2> other)
{
    int c = Comparer<T1>.Default.Compare(Item1, other.Item1);
    if (c != 0) return c;
    return Comparer<T2>.Default.Compare(Item2, other.Item2);
}
```

برای `ValueTuple<T1, T2, T3>` نیز به همین ترتیب تا `Item3` ادامه می‌یابد.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## ترتیب مقایسه اعضا

ترتیب، **همیشه از چپ‌ترین عضو (Item1) به سمت راست** است:

1. ابتدا `Item1` مقایسه می‌شود.
2. اگر برابر بود، `Item2` مقایسه می‌شود.
3. اگر باز هم برابر بود، `Item3` و الی آخر.
4. به محض پیدا شدن تفاوت، همان نتیجه برگردانده می‌شود و بقیهٔ اعضا بررسی نمی‌شوند.

### مثال

```csharp
var a = (1, "B");
var b = (1, "A");
var c = (2, "A");

Console.WriteLine(a.CompareTo(b)); // مثبت  → چون "B" > "A"
Console.WriteLine(a.CompareTo(c)); // منفی  → چون 1 < 2 (Item2 اصلاً بررسی نمی‌شود)
```

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## مقایسه Tupleهای عددی

برای انواع عددی، ترتیب به‌صورت طبیعی (کمینه به بیشینه) است:

```csharp
var t1 = (10, 20);
var t2 = (10, 30);
var t3 = (5, 100);

Console.WriteLine(t1.CompareTo(t2)); // -1 (چون 20 < 30)
Console.WriteLine(t1.CompareTo(t3)); //  1 (چون 10 > 5)
```

### رفتار با اعداد اعشاری خاص

از آنجا که `Comparer<T>.Default` استفاده می‌شود، رفتارهای خاص `double` و `float` نیز منتقل می‌شود:

- `NaN` در مقایسه از هر عددی **بزرگ‌تر** در نظر گرفته می‌شود (حتی `PositiveInfinity`).
- `-0.0` و `+0.0` برابرند.

```csharp
var a = (double.NaN, 0);
var b = (1.0, 0);
Console.WriteLine(a.CompareTo(b)); // مثبت → NaN بزرگ‌تر از 1.0 است
```

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## Sort کردن Collectionهای Tuple

چون `ValueTuple` رابط `IComparable` را پیاده‌سازی می‌کند، می‌توان آن را مستقیماً مرتب کرد:

```csharp
var list = new List<(int Age, string Name)>
{
    (30, "Ali"),
    (25, "Sara"),
    (30, "Reza"),
    (20, "Mina")
};

list.Sort(); // مرتب‌سازی پیش‌فرض

foreach (var item in list)
    Console.WriteLine(item);

// خروجی:
// (20, Mina)
// (25, Sara)
// (30, Ali)
// (30, Reza)
```

### مرتب‌سازی معکوس با `IComparer` سفارشی

```csharp
list.Sort((x, y) => y.CompareTo(x)); // نزولی
```

### مرتب‌سازی بر اساس یک ستون خاص (مثلاً فقط Name)

```csharp
list.Sort((x, y) => string.Compare(x.Name, y.Name, StringComparison.Ordinal));
```

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## استفاده با SortedSet

`SortedSet<T>` برای نگهداری عناصر منحصربه‌فرد و مرتب، به `IComparable<T>` یا یک `IComparer<T>` نیاز دارد. از آنجا که `ValueTuple` هر دو را دارد، به‌صورت پیش‌فرض کار می‌کند:

```csharp
var set = new SortedSet<(int Priority, string Task)>
{
    (2, "Write tests"),
    (1, "Fix bug"),
    (3, "Deploy"),
    (1, "Review PR")
};

foreach (var item in set)
    Console.WriteLine(item);

// خروجی:
// (1, Fix bug)
// (1, Review PR)
// (2, Write tests)
// (3, Deploy)
```

> ⚠️ **نکتهٔ بسیار مهم:** `SortedSet` برای تشخیص تکراری‌بودن از **همان مقایسه‌گر ترتیبی** استفاده می‌کند، نه از `Equals`/`GetHashCode`. یعنی اگر `CompareTo` دو عنصر را برابر (صفر) برگرداند، آن‌ها **یکسان** فرض می‌شوند.

### مثال رفتار غیرمنتظره

```csharp
var set = new SortedSet<(int Id, string Name)>
{
    (1, "Ali"),
    (1, "Reza")  // این عضو اضافه نمی‌شود!
};

Console.WriteLine(set.Count); // 1 — چون Id برابر است
```

اگر این رفتار را نمی‌خواهید، باید `IComparer` سفارشی بنویسید که همهٔ اعضا را در نظر بگیرد.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## استفاده با SortedDictionary

`SortedDictionary<TKey, TValue>` کلیدها را بر اساس `IComparable<TKey>` مرتب می‌کند. بنابراین می‌توان از Tuple به‌عنوان کلید چندبخشی استفاده کرد:

```csharp
var dict = new SortedDictionary<(int Year, int Month), string>
{
    [(2024, 3) ] = "March Report",
    [(2024, 1) ] = "January Report",
    [(2023, 12)] = "December Report",
    [(2024, 2) ] = "February Report"
};

foreach (var kv in dict)
    Console.WriteLine($"{kv.Key} => {kv.Value}");

// خروجی:
// (2023, 12) => December Report
// (2024, 1)  => January Report
// (2024, 2)  => February Report
// (2024, 3)  => March Report
```

### کاربرد عملی: کلید ترکیبی برای دادهٔ دوبعدی

```csharp
// ماتریس اسپارس با کلید (ردیف، ستون)
var sparse = new SortedDictionary<(int Row, int Col), double>();
sparse[(0, 0)] = 1.5;
sparse[(2, 3)] = 4.2;
sparse[(1, 1)] = 3.0;

foreach (var kv in sparse)
    Console.WriteLine($"[{kv.Key.Row},{kv.Key.Col}] = {kv.Value}");
```

این الگو برای داده‌هایی که به‌صورت طبیعی دارای دو بُعد مرتب هستند (مانند رویدادهای زمانی با `(Date, Id)` یا مختصات `(X, Y)`) بسیار کارآمد است.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## تفاوت Equality و Ordering

یکی از مهم‌ترین مفاهیمی که باید درک شود، تفاوت بین **برابری (Equality)** و **ترتیب (Ordering)** است. این دو مفهوم در #C مستقل از یکدیگرند:

| مفهوم | رابط‌ها | هدف |
|---|---|---|
| Equality | `IEquatable<T>`، `Equals`، `GetHashCode` | آیا دو شیء «یکسان» هستند؟ |
| Ordering | `IComparable<T>`، `CompareTo` | کدام شیء «بزرگ‌تر» است؟ |

### قرارداد مهم

- برای `Dictionary<TKey, TValue>` و `HashSet<T>` → فقط Equality مهم است.
- برای `SortedDictionary<TKey, TValue>` و `SortedSet<T>` → فقط Ordering مهم است.

### مثال تضاد

می‌توان نوعی نوشت که از نظر `Equals` برابر باشد ولی از نظر `CompareTo` نابرابر (هرچند توصیه نمی‌شود):

```csharp
// در ValueTuple این تضاد وجود ندارد و هر دو بر اساس همهٔ اعضا سنجیده می‌شوند.
// اما اگر IComparer سفارشی بنویسید که فقط Item1 را ببیند:
var comparer = Comparer<(int, string)>.Create((x, y) => x.Item1.CompareTo(y.Item1));

var set = new SortedSet<(int, string)>(comparer);
set.Add((1, "Ali"));
set.Add((1, "Reza")); // اضافه نمی‌شود، با اینکه Equals آن‌ها false است!
```

> 💡 **قانون طلایی:** اگر `CompareTo(a, b) == 0` باشد، باید `Equals(a, b) == true` باشد. در غیر این‌صورت رفتار ساختارهای مرتب غیرقابل‌پیش‌بینی می‌شود.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## نکات مهم

1. **مقایسهٔ Tuple با `null`:** از آنجا که `ValueTuple` یک نوع مقداری (struct) است، هرگز `null` نیست. اما `System.Tuple` یک کلاس است و `CompareTo(null)` همیشه مقداری مثبت برمی‌گرداند.

2. **عملکرد:** `ValueTuple` به‌دلیل نوع مقداری بودن، از `System.Tuple` سریع‌تر است و Boxing ندارد. برای حلقه‌های داخلی و دادهٔ حجیم، همیشه `ValueTuple` را ترجیح دهید.

3. **نام‌گذاری اعضا:** نام‌هایی مانند `Age` و `Name` که در `(int Age, string Name)` استفاده می‌کنید، صرفاً **نام‌های مستعار (Alias)** زمان کامپایل هستند و در زمان اجرا همان `Item1` و `Item2` خواهند بود. این موضوع روی رفتار مقایسه تأثیری ندارد.

4. **حداکثر تعداد اعضا:** `ValueTuple` تا ۷ عضو مستقیم دارد. برای بیشتر از آن، عضو هشتم به‌صورت `Rest` (که خودش یک Tuple است) ذخیره می‌شود و مقایسه نیز به‌صورت بازگشتی روی `Rest` انجام می‌شود.

5. **انواع غیرقابل‌مقایسه:** اگر یکی از اعضای Tuple نوعی باشد که `IComparable` را پیاده‌سازی نکرده باشد، در زمان اجرا `InvalidOperationException` پرتاب می‌شود.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## اشتباهات رایج

### ❌ اشتباه ۱: فرض اینکه `SortedSet` از `Equals` برای تشخیص تکراری استفاده می‌کند

```csharp
var set = new SortedSet<(int, string)>();
set.Add((1, "A"));
set.Add((1, "B")); // اضافه نمی‌شود!
Console.WriteLine(set.Count); // 1
```

**راه‌حل:** یا `IComparer` کامل بنویسید، یا از `HashSet` استفاده کنید.

### ❌ اشتباه ۲: استفاده از Tuple با نوعی که `IComparable` ندارد

```csharp
class MyClass { } // IComparable پیاده‌سازی نکرده
var t = (new MyClass(), 5);
var list = new List<typeof(t)>();
list.Add(t);
list.Sort(); // ❌ InvalidOperationException در زمان اجرا
```

**راه‌حل:** یا `IComparable` را برای آن نوع پیاده‌سازی کنید، یا `IComparer` سفارشی به `Sort` بدهید.

### ❌ اشتباه ۳: نادیده‌گرفتن رفتار `NaN` در اعداد اعشاری

```csharp
var a = (double.NaN, 1);
var b = (double.PositiveInfinity, 2);
Console.WriteLine(a.CompareTo(b)); // مثبت! چون NaN از همه بزرگ‌تر است
```

اگر این رفتار مطلوب نیست، باید `IComparer` سفارشی بنویسید.

### ❌ اشتباه ۴: استفاده از `System.Tuple` به‌جای `ValueTuple` در کد جدید

```csharp
var old = Tuple.Create(1, 2);      // System.Tuple — مرجعی، کندتر
var modern = (1, 2);                // ValueTuple — مقداری، سریع‌تر
```

در کد جدید همیشه از سینتکسی `(a, b)` استفاده کنید.

### ❌ اشتباه ۵: انتظار ترتیب الفبایی برای `string` بدون توجه به فرهنگ

مقایسهٔ پیش‌فرض `string` در `Comparer<string>.Default` از مقایسهٔ **فرهنگ جاری (Current Culture)** استفاده می‌کند که ممکن است در سیستم‌های مختلف نتایج متفاوتی بدهد. برای ترتیب پایدار، `StringComparer.Ordinal` را ترجیح دهید.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## جمع‌بندی

- `ValueTuple` و `System.Tuple` هر دو `IComparable` را پیاده‌سازی می‌کنند و می‌توانند به‌صورت پیش‌فرض مرتب شوند.
- مقایسه به‌صورت **Lexicographical** و از چپ به راست انجام می‌شود.
- برای هر عضو از `Comparer<T>.Default` استفاده می‌شود.
- می‌توان از Tuple به‌عنوان کلید در `SortedDictionary` و عضو در `SortedSet` استفاده کرد.
- `SortedSet` و `SortedDictionary` برای تشخیص تکراری‌بودن از `CompareTo` استفاده می‌کنند، نه از `Equals`.
- بین Equality و Ordering تفاوت قائل شوید و قرارداد `CompareTo == 0 ⟹ Equals == true` را رعایت کنید.

[↑ بازگشت به فهرست](#فهرست-مطالب)

---

## منابع رسمی

- [Microsoft Docs — `ValueTuple<T1,T2>` Struct](https://learn.microsoft.com/dotnet/api/system.valuetuple-2)
- [Microsoft Docs — `IComparable<T>` Interface](https://learn.microsoft.com/dotnet/api/system.icomparable-1)
- [Microsoft Docs — `IStructuralComparable` Interface](https://learn.microsoft.com/dotnet/api/system.collections.istructuralcomparable)
- [Microsoft Docs — Tuples (C# Programming Guide)](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/value-tuples)
- [Source Reference Source — `ValueTuple<T1,T2>.CompareTo`](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/ValueTuple.cs)
- [Microsoft Docs — `SortedSet<T>` Class](https://learn.microsoft.com/dotnet/api/system.collections.generic.sortedset-1)
- [Microsoft Docs — `SortedDictionary<TKey,TValue>` Class](https://learn.microsoft.com/dotnet/api/system.collections.generic.sorteddictionary-2)

---

> 📝 این مقاله بخشی از یک Repository آموزشی دربارهٔ #C است. برای مباحث مرتبط مانند `IEquatable`، `IComparer` و `Pattern Matching` با Tupleها، به سایر مقالات همین Repository مراجعه کنید.