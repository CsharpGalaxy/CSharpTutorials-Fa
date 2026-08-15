## فهرست مطالب

1. [Generic چیست؟](#what)
2. [چرا به Generic نیاز داریم؟](#why)
3. [اولین مثال ساده با Generic](#basic-syntax)
4. [متدهای Generic](#generic-methods)
5. [کلاس‌های Generic](#generic-classes)
6. [پارامترهای نوع و نام‌گذاری استاندارد](#type-parameters)
7. [محدودیت‌ها یا Constraints](#constraints)
8. [رابط‌های Generic](#generic-interfaces)
9. [دلیگیت‌های Generic](#generic-delegates)
10. [کالکشن‌های Generic](#generic-collections)
11. [default، typeof و کار با نوع در Generic](#default-typeof)
12. [Covariance و Contravariance](#variance)
13. [Generic و OOP](#oop)
14. [مباحث پیشرفته Generic](#advanced)
15. [اشتباه‌های رایج](#mistakes)
16. [بهترین روش‌ها](#best-practices)
17. [تمرین‌های پیشنهادی برای ریپازیتوری آموزشی](#exercises)
18. [منابع معتبر](#sources)

---

<a id="what"></a>
## ۱) Generic چیست؟

در سی‌شارپ، **Generic** یعنی توانایی نوشتن کلاس، متد، رابط، دلیگیت یا ساختاری که **نوع داده** به‌جای اینکه از ابتدا ثابت باشد، مثل یک پارامتر ورودی در زمان استفاده مشخص شود.

به زبان ساده:

> Generic به ما اجازه می‌دهد کدی بنویسیم که با **نوع‌های مختلف** کار کند، ولی هنوز **Type Safety** یا امنیت نوع را در زمان کامپایل حفظ کند.

مثلاً به‌جای اینکه یک متد فقط برای `int` یا فقط برای `string` بنویسیم، می‌نویسیم:

```csharp
public static T Echo<T>(T value)
{
    return value;
}
```

اینجا `T` یک **Type Parameter** است؛ یعنی نوعی که بعداً توسط استفاده‌کننده مشخص می‌شود.

استفاده:

```csharp
int a = Echo(10);          // T = int
string b = Echo("Salam");  // T = string
double c = Echo(3.14);     // T = double
```

Generic در سی‌شارپ از C# 2.0 معرفی شد و یکی از مهم‌ترین امکانات زبان برای نوشتن کد قابل‌استفاده مجدد، امن و کارآمد است.  
منبع: [C# Programming Guide - Generics](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)

---

<a id="why"></a>
## ۲) چرا به Generic نیاز داریم؟

قبل از Generic، اگر می‌خواستیم یک لیست از هر نوع داده‌ای داشته باشیم، ممکن بود از `ArrayList` استفاده کنیم:

```csharp
using System.Collections;

ArrayList list = new ArrayList();

list.Add(10);
list.Add("Ali"); // هیچ خطای کامپایلی نمی‌دهد

int first = (int)list[0]; // نیاز به Cast دارد
```

مشکلات این روش:

1. **امنیت نوع وجود ندارد**  
   می‌توان هر چیزی را داخل لیست ریخت.

2. **نیاز به Cast دارد**  
   هنگام خواندن باید نوع را تبدیل کنیم.

3. **احتمال خطای Runtime وجود دارد**  
   ممکن است موقع Cast کردن خطا بگیریم.

4. **برای Value Typeها Boxing/Unboxing رخ می‌دهد**  
   مثلاً `int` به `object` تبدیل می‌شود و این می‌تواند هزینه عملکردی داشته باشد.

اما با Generic:

```csharp
using System.Collections.Generic;

List<int> numbers = new List<int>();

numbers.Add(10);

// numbers.Add("Ali"); // خطای کامپایل — چون لیست فقط int قبول می‌کند

int first = numbers[0]; // بدون Cast
```

مزایا:

- **Type Safety در زمان کامپایل**
- **حذف Cast اضافی**
- **کاهش Boxing/Unboxing**
- **کد تمیزتر و قابل‌استفاده مجدد**
- **بهتر شدن عملکرد**
- **پشتیبانی بهتر از OOP و Abstraction**

---

<a id="basic-syntax"></a>
## ۳) اولین مثال ساده با Generic

یک متد Generic ساده:

```csharp
public static class Demo
{
    public static T Echo<T>(T value)
    {
        return value;
    }
}
```

استفاده:

```csharp
Console.WriteLine(Demo.Echo(5));        // 5
Console.WriteLine(Demo.Echo("Hello"));  // Hello
Console.WriteLine(Demo.Echo(true));     // True
```

در این مثال:

- `Echo` نام متد است.
- `<T>` یعنی این متد Generic است.
- `T` یک پارامتر نوع است.
- وقتی متد را صدا می‌زنیم، کامپایلر معمولاً نوع `T` را از روی آرگومان تشخیص می‌دهد.

به این قابلیت **Type Inference** می‌گویند.

```csharp
var result = Demo.Echo("Salam");
// کامپایلر می‌فهمد T برابر string است
```

اگر بخواهیم نوع را دستی مشخص کنیم:

```csharp
var result = Demo.Echo<string>("Salam");
```

---

<a id="generic-methods"></a>
## ۴) متدهای Generic

متد Generic متدی است که یک یا چند Type Parameter دارد.

### ۴.۱ مثال: جابه‌جا کردن دو مقدار

```csharp
public static void Swap<T>(ref T left, ref T right)
{
    T temp = left;
    left = right;
    right = temp;
}
```

استفاده:

```csharp
int a = 1;
int b = 2;

Swap(ref a, ref b);

Console.WriteLine(a); // 2
Console.WriteLine(b); // 1
```

برای string:

```csharp
string x = "Ali";
string y = "Reza";

Swap(ref x, ref y);
```

### ۴.۲ مثال: پیدا کردن اولین مقدار یا مقدار پیش‌فرض

```csharp
using System.Collections.Generic;

public static T FirstOrDefault<T>(IEnumerable<T> source)
{
    foreach (T item in source)
    {
        return item;
    }

    return default(T);
}
```

استفاده:

```csharp
var numbers = new List<int> { 1, 2, 3 };
var empty = new List<int>();

Console.WriteLine(FirstOrDefault(numbers)); // 1
Console.WriteLine(FirstOrDefault(empty));   // 0
```

### ۴.۳ مثال: بیشترین مقدار با محدودیت `IComparable<T>`

```csharp
using System;
using System.Collections.Generic;

public static T Max<T>(IEnumerable<T> source) where T : IComparable<T>
{
    using var enumerator = source.GetEnumerator();

    if (!enumerator.MoveNext())
    {
        throw new InvalidOperationException("The source is empty.");
    }

    T max = enumerator.Current;

    while (enumerator.MoveNext())
    {
        if (enumerator.Current.CompareTo(max) > 0)
        {
            max = enumerator.Current;
        }
    }

    return max;
}
```

استفاده:

```csharp
var numbers = new List<int> { 3, 9, 4, 7 };
var names = new List<string> { "Ali", "Sara", "Mohammad" };

Console.WriteLine(Max(numbers)); // 9
Console.WriteLine(Max(names));   // Sara
```

چرا `where T : IComparable<T>` گذاشتیم؟  
چون می‌خواهیم مقایسه کردن مقادیر از نوع `T` ممکن باشد. بدون این محدودیت، کامپایلر اجازه نمی‌دهد از `CompareTo` استفاده کنیم.

منبع: [Generic Methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods)

---

<a id="generic-classes"></a>
## ۵) کلاس‌های Generic

کلاس Generic کلاسی است که یک یا چند Type Parameter دارد.

### ۵.۱ مثال ساده: Storage

```csharp
using System.Collections.Generic;

public class Storage<T>
{
    private readonly List<T> _items = new();

    public void Add(T item)
    {
        _items.Add(item);
    }

    public int Count => _items.Count;

    public T this[int index] => _items[index];
}
```

استفاده:

```csharp
Storage<int> numbers = new Storage<int>();
numbers.Add(10);
numbers.Add(20);

Console.WriteLine(numbers.Count);    // 2
Console.WriteLine(numbers[0]);       // 10
```

```csharp
Storage<string> names = new Storage<string>();
names.Add("Ali");
names.Add("Sara");

Console.WriteLine(names[0]); // Ali
```

### ۵.۲ کلاس Generic با دو پارامتر نوع

```csharp
public class Pair<TFirst, TSecond>
{
    public TFirst First { get; set; }
    public TSecond Second { get; set; }

    public override string ToString()
    {
        return $"({First}, {Second})";
    }
}
```

استفاده:

```csharp
var personAge = new Pair<string, int>
{
    First = "Ali",
    Second = 25
};

Console.WriteLine(personAge); // (Ali, 25)
```

### ۵.۳ ارث‌بری با Generic

یک کلاس Generic می‌تواند از یک کلاس Generic ارث ببرد:

```csharp
public class BaseRepository<T>
{
    public virtual string Name => "BaseRepository";
}

public class CustomerRepository<T> : BaseRepository<T>
{
    public override string Name => "CustomerRepository";
}
```

همچنین می‌توان نوع را بسته یا Closed کرد:

```csharp
public class IntRepository : BaseRepository<int>
{
}
```

منبع: [Generic Classes](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes)

---

<a id="type-parameters"></a>
## ۶) پارامترهای نوع و نام‌گذاری استاندارد

به `T` در `class Storage<T>` می‌گوییم **Type Parameter**.

در سی‌شارپ معمولاً نام Type Parameter با `T` شروع می‌شود.

### مثال‌های رایج

| نام | کاربرد رایج |
|---|---|
| `T` | حالت عمومی |
| `TItem` | آیتم داخل کالکشن |
| `TEntity` | موجودیت یا Entity در الگوی Repository |
| `TKey` | نوع کلید |
| `TValue` | نوع مقدار |
| `TResult` | نوع خروجی |
| `TFirst`, `TSecond` | وقتی دو یا چند پارامتر نوع داریم |

مثال:

```csharp
public interface IRepository<TEntity>
{
    void Add(TEntity entity);
}

public class Dictionary<TKey, TValue>
{
    // TKey نوع کلید است
    // TValue نوع مقدار است
}
```

### نکته مهم

`T` یک نوع واقعی نیست؛ بلکه یک placeholder است.  
یعنی تا زمانی که کسی `Storage<int>` یا `Storage<string>` نسازد، `T` هنوز مشخص نیست.

منبع: [Generic Type Parameters](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters)

---

<a id="constraints"></a>
## ۷) محدودیت‌ها یا Constraints

Constraints مشخص می‌کنند که Type Parameter باید چه شرایطی داشته باشد.

Syntax:

```csharp
public class MyClass<T> where T : class, new()
{
}
```

یعنی `T` باید:

1. یک Reference Type باشد.
2. سازنده بدون پارامتر داشته باشد.

---

### ۷.۱ جدول Constraintهای مهم

| Constraint | معنی |
|---|---|
| `where T : struct` | `T` باید Value Type باشد |
| `where T : class` | `T` باید Reference Type باشد |
| `where T : class?` | `T` باید Reference Type nullable باشد، در زمینه nullable |
| `where T : notnull` | `T` نباید nullable باشد |
| `where T : new()` | `T` باید سازنده بدون پارامتر داشته باشد |
| `where T : BaseClass` | `T` باید از یک کلاس خاص ارث ببرد |
| `where T : IInterface` | `T` باید یک رابط را پیاده‌سازی کند |
| `where T : U` | `T` باید از پارامتر نوع `U` ارث ببرد یا به آن قابل تبدیل باشد |
| `where T : System.Enum` | `T` باید Enum باشد |
| `where T : System.Delegate` | `T` باید Delegate باشد |
| `where T : unmanaged` | `T` باید unmanaged باشد؛ بیشتر برای کدهای unsafe و pointerها |

---

### ۷.۲ مثال: محدودیت `new()`

```csharp
public static T CreateInstance<T>() where T : new()
{
    return new T();
}
```

استفاده:

```csharp
var list = CreateInstance<List<int>>();
```

اگر `new()` نگذاریم، این کد کامپایل نمی‌شود:

```csharp
public static T CreateInstance<T>()
{
    return new T(); // خطا
}
```

چون کامپایلر نمی‌داند `T` حتماً سازنده بدون پارامتر دارد.

---

### ۷.۳ مثال: محدودیت کلاسی و رابطی

```csharp
public interface IEntity
{
    int Id { get; }
}

public class EntityValidator<T> where T : class, IEntity, new()
{
    public bool IsValid(T entity)
    {
        return entity.Id > 0;
    }
}
```

در این مثال `T` باید:

- Reference Type باشد.
- `IEntity` را پیاده‌سازی کند.
- سازنده بدون پارامتر داشته باشد.

---

### ۷.۴ مثال: محدودیت `struct`

```csharp
public static string Describe<T>(T value) where T : struct
{
    return $"Value: {value}, Type: {typeof(T)}";
}
```

استفاده:

```csharp
Console.WriteLine(Describe(10));       // OK
Console.WriteLine(Describe(3.14));     // OK
// Describe("Ali");                    // خطا، چون string struct نیست
```

---

### ۷.۵ نکته مهم درباره Type Inference و Constraint

کامپایلر نوع را فقط از روی آرگومان‌ها تشخیص می‌دهد، نه از روی Constraint یا نوع بازگشتی.

مثال:

```csharp
public static T Create<T>() where T : new()
{
    return new T();
}
```

این کد خطا می‌دهد:

```csharp
var value = Create(); // خطا
```

چون کامپایلر نمی‌داند `T` چیست.

باید نوع را مشخص کنیم:

```csharp
var value = Create<List<int>>();
```

منبع: [Constraints on Type Parameters](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)

---

<a id="generic-interfaces"></a>
## ۸) رابط‌های Generic

رابط Generic رابطی است که یک یا چند Type Parameter دارد.

مثال‌های معروف:

```csharp
IEnumerable<T>
IComparable<T>
IEquatable<T>
IList<T>
IReadOnlyList<T>
IDictionary<TKey, TValue>
```

### ۸.۱ مثال ساده

```csharp
public interface IStorage<T>
{
    void Add(T item);
    int Count { get; }
}
```

پیاده‌سازی:

```csharp
using System.Collections.Generic;

public class ListStorage<T> : IStorage<T>
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);

    public int Count => _items.Count;
}
```

استفاده:

```csharp
IStorage<int> storage = new ListStorage<int>();

storage.Add(10);
storage.Add(20);

Console.WriteLine(storage.Count); // 2
```

### ۸.۲ مثال کاربردی: Repository

```csharp
public interface IRepository<T>
{
    void Add(T entity);
    IEnumerable<T> GetAll();
}
```

این یکی از معروف‌ترین کاربردهای Generic در معماری‌های شیءگرا است.

### ۸.۳ چرا رابط Generic مهم است؟

- باعث **Abstraction** می‌شود.
- وابستگی به پیاده‌سازی را کم می‌کند.
- Type Safety را حفظ می‌کند.
- کد را قابل‌استفاده مجدد می‌کند.

منبع: [Generic Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces)

---

<a id="generic-delegates"></a>
## ۹) دلیگیت‌های Generic

دلیگیت Generic دلیگیتی است که Type Parameter دارد.

### ۹.۱ دلیگیت‌های معروف .NET

| دلیگیت | توضیح |
|---|---|
| `Action<T>` | متدی که یک ورودی می‌گیرد و چیزی برنمی‌گرداند |
| `Func<T, TResult>` | متدی که یک ورودی می‌گیرد و یک خروجی برمی‌گرداند |
| `Predicate<T>` | متدی که یک مقدار می‌گیرد و `bool` برمی‌گرداند |

مثال:

```csharp
using System;

Action<string> print = message => Console.WriteLine(message);
print("Hello");

Func<int, int> square = x => x * x;
Console.WriteLine(square(5)); // 25

Predicate<int> isEven = n => n % 2 == 0;
Console.WriteLine(isEven(4)); // True
```

### ۹.۲ تعریف دلیگیت Generic سفارشی

```csharp
public delegate TResult Transformer<TInput, TResult>(TInput input);
```

استفاده:

```csharp
Transformer<int, string> intToString = x => $"Number: {x}";

Console.WriteLine(intToString(10)); // Number: 10
```

### ۹.۳ دلیگیت Generic با Constraint

```csharp
public delegate void EntityHandler<TEntity>(TEntity entity)
    where TEntity : class;
```

منبع: [Generic Delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates)

---

<a id="generic-collections"></a>
## ۱۰) کالکشن‌های Generic

یکی از مهم‌ترین کاربردهای Generic، کالکشن‌ها هستند.

فضای نام:

```csharp
System.Collections.Generic
```

### ۱۰.۱ کالکشن‌های رایج

| نوع | توضیح |
|---|---|
| `List<T>` | لیست پویا |
| `Dictionary<TKey, TValue>` | ذخیره کلید/مقدار |
| `HashSet<T>` | مجموعه مقادیر یکتا |
| `Queue<T>` | صف، FIFO |
| `Stack<T>` | پشته، LIFO |
| `LinkedList<T>` | لیست پیوندی |
| `IReadOnlyList<T>` | رابط برای لیست فقط‌خواندنی |
| `IEnumerable<T>` | رابط برای پیمایش یک مجموعه |

---

### ۱۰.۲ مثال List

```csharp
using System.Collections.Generic;

List<string> names = new List<string>();

names.Add("Ali");
names.Add("Sara");
names.Add("Reza");

foreach (string name in names)
{
    Console.WriteLine(name);
}
```

---

### ۱۰.۳ مثال Dictionary

```csharp
using System.Collections.Generic;

var scores = new Dictionary<string, int>
{
    ["Ali"] = 18,
    ["Sara"] = 20
};

scores["Reza"] = 15;

Console.WriteLine(scores["Ali"]); // 18
```

---

### ۱۰.۴ مثال HashSet

```csharp
using System.Collections.Generic;

var uniqueNumbers = new HashSet<int> { 1, 2, 2, 3, 3, 3 };

Console.WriteLine(uniqueNumbers.Count); // 3
```

---

### ۱۰.۵ مثال Queue و Stack

```csharp
using System;
using System.Collections.Generic;

var queue = new Queue<string>();
queue.Enqueue("First");
queue.Enqueue("Second");

Console.WriteLine(queue.Dequeue()); // First


var stack = new Stack<int>();
stack.Push(1);
stack.Push(2);

Console.WriteLine(stack.Pop()); // 2
```

---

### ۱۰.۶ چرا کالکشن Generic بهتر است؟

به‌جای:

```csharp
ArrayList list = new ArrayList();
```

از:

```csharp
List<int> list = new List<int>();
```

استفاده کنید.

دلایل:

- امنیت نوع
- حذف Cast
- کاهش Boxing/Unboxing
- خوانایی بهتر
- پشتیبانی بهتر از LINQ

منابع:

- [System.Collections.Generic Namespace](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic)
- [List<T> Class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)
- [Dictionary<TKey,TValue> Class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2)
- [IEnumerable<T> Interface](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1)

---

<a id="default-typeof"></a>
## ۱۱) default، typeof و کار با نوع در Generic

وقتی با `T` کار می‌کنیم، گاهی نمی‌دانیم نوع چیست. در این حالت چند ابزار مهم داریم.

---

### ۱۱.۱ مقدار پیش‌فرض با `default`

```csharp
public static T GetDefault<T>()
{
    return default(T);
}
```

یا در C# جدیدتر:

```csharp
public static T GetDefault<T>()
{
    return default;
}
```

مثال:

```csharp
Console.WriteLine(GetDefault<int>());     // 0
Console.WriteLine(GetDefault<string>() ?? "null"); // null
Console.WriteLine(GetDefault<bool>());    // False
```

مقدار `default` برای انواع مختلف:

| نوع | مقدار پیش‌فرض |
|---|---|
| `int` | `0` |
| `bool` | `false` |
| `string` | `null` |
| کلاس‌ها | `null` |
| ساختارها | مقدار پیش‌فرض فیلدها |

منبع: [default operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/default)

---

### ۱۱.۲ گرفتن نوع با `typeof`

```csharp
public static string GetTypeName<T>()
{
    return typeof(T).Name;
}
```

استفاده:

```csharp
Console.WriteLine(GetTypeName<int>());      // Int32
Console.WriteLine(GetTypeName<string>());   // String
Console.WriteLine(GetTypeName<List<int>>()); // List`1
```

منبع: [typeof operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#typeof-operator)

---

### ۱۱.۳ بررسی نوع با `is`

```csharp
public static string Check<T>(object value)
{
    if (value is T typedValue)
    {
        return $"It is {typeof(T).Name}: {typedValue}";
    }

    return "Not compatible";
}
```

---

<a id="variance"></a>
## ۱۲) Covariance و Contravariance

این بخش کمی پیشرفته‌تر است، ولی برای درک عمیق Genericها بسیار مهم است.

در سی‌شارپ بعضی Generic Interfaceها و Delegateها می‌توانند تبدیل نوع داشته باشند.

---

### ۱۲.۱ Covariance با `out`

اگر یک Type Parameter فقط به‌عنوان خروجی استفاده شود، می‌توان آن را با `out` علامت زد.

```csharp
public interface IProducer<out T>
{
    T Get();
}
```

یعنی می‌توان یک `IProducer<string>` را به `IProducer<object>` تبدیل کرد.

مثال واقعی:

```csharp
using System.Collections.Generic;

IEnumerable<string> names = new List<string>();
IEnumerable<object> objects = names; // OK
```

چرا؟  
چون `IEnumerable<out T>` covariant است.

---

### ۱۲.۲ Contravariance با `in`

اگر یک Type Parameter فقط به‌عنوان ورودی استفاده شود، می‌توان آن را با `in` علامت زد.

```csharp
public interface IConsumer<in T>
{
    void Take(T item);
}
```

مثال واقعی:

```csharp
using System;

Action<object> printObject = obj => Console.WriteLine(obj);
Action<string> printString = printObject; // OK

printString("Hello");
```

چرا؟  
چون `Action<in T>` contravariant است.

---

### ۱۲.۳ چرا `IList<T>` کوواریانت نیست؟

```csharp
IList<string> strings = new List<string>();

// IList<object> objects = strings; // خطا
```

اگر این تبدیل مجاز بود، می‌توانستیم از طریق `IList<object>` یک شیء غیر string به لیست string اضافه کنیم و امنیت نوع از بین برود.

---

### ۱۲.۴ قوانین `in` و `out`

| کلمه | کاربرد | محدودیت |
|---|---|---|
| `out` | خروجی | نمی‌توان به‌عنوان پارامتر ورودی متد استفاده شود |
| `in` | ورودی | نمی‌توان به‌عنوان نوع بازگشتی استفاده شود |

---

### ۱۲.۵ نکته مهم

Variance فقط برای Interfaceها و Delegateها کاربرد دارد، نه برای کلاس‌ها.

همچنین برای Value Typeها معمولاً تبدیل variance معنا ندارد.

منبع: [Covariance and Contravariance](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/)

---

<a id="oop"></a>
## ۱۳) Generic و OOP

Genericها با مفاهیم OOP ارتباط زیادی دارند.

---

### ۱۳.۱ Abstraction

Generic به ما کمک می‌کند رفتار کلی را تعریف کنیم بدون اینکه به نوع دقیق وابسته باشیم.

```csharp
public interface IRepository<T>
{
    void Add(T entity);
}
```

این یعنی:  
من یک Repository دارم، اما نوع Entity بعداً مشخص می‌شود.

---

### ۱۳.۲ Encapsulation

می‌توانیم جزئیات نگهداری داده را داخل یک کلاس Generic پنهان کنیم.

```csharp
public class Storage<T>
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);

    public int Count => _items.Count;
}
```

کاربر فقط از `Add` و `Count` استفاده می‌کند و نمی‌داند داخل از `List<T>` استفاده شده است.

---

### ۱۳.۳ Polymorphism

با رابط‌های Generic می‌توانیم چند پیاده‌سازی مختلف داشته باشیم:

```csharp
public interface IStorage<T>
{
    void Add(T item);
}

public class MemoryStorage<T> : IStorage<T>
{
    public void Add(T item)
    {
        // ذخیره در حافظه
    }
}

public class FileStorage<T> : IStorage<T>
{
    public void Add(T item)
    {
        // ذخیره در فایل
    }
}
```

---

### ۱۳.۴ Dependency Inversion

به‌جای وابستگی به کلاس مشخص، به رابط Generic وابسته می‌شویم:

```csharp
public class UserService<T> where T : class
{
    private readonly IRepository<T> _repository;

    public UserService(IRepository<T> repository)
    {
        _repository = repository;
    }
}
```

---

### ۱۳.۵ مثال کامل‌تر: Generic Repository

```csharp
#nullable enable
using System.Collections.Generic;
using System.Linq;

public interface IEntity
{
    int Id { get; }
}

public interface IRepository<TEntity> where TEntity : class, IEntity
{
    void Add(TEntity entity);
    IReadOnlyList<TEntity> GetAll();
    TEntity? FindById(int id);
}

public class InMemoryRepository<TEntity> : IRepository<TEntity>
    where TEntity : class, IEntity
{
    private readonly List<TEntity> _entities = new();

    public void Add(TEntity entity)
    {
        _entities.Add(entity);
    }

    public IReadOnlyList<TEntity> GetAll()
    {
        return _entities;
    }

    public TEntity? FindById(int id)
    {
        return _entities.FirstOrDefault(e => e.Id == id);
    }
}
```

مثال استفاده:

```csharp
public class Person : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
}
```

```csharp
IRepository<Person> repository = new InMemoryRepository<Person>();

repository.Add(new Person { Id = 1, Name = "Ali" });
repository.Add(new Person { Id = 2, Name = "Sara" });

var person = repository.FindById(1);

Console.WriteLine(person?.Name); // Ali
```

این مثال نشان می‌دهد Generic چطور به معماری تمیز و OOP کمک می‌کند.

---

<a id="advanced"></a>
## ۱۴) مباحث پیشرفته Generic

---

### ۱۴.۱ Open Generic و Closed Generic

به این نوع می‌گوییم Closed Generic:

```csharp
typeof(List<int>)
```

به این نوع می‌گوییم Open Generic:

```csharp
typeof(List<>)
```

نمی‌توان مستقیماً از Open Generic شیء ساخت:

```csharp
// new List<>(); // خطا
```

اما با Reflection می‌توان نوع بسته ساخت:

```csharp
using System;

Type openType = typeof(List<>);
Type closedType = openType.MakeGenericType(typeof(int));

object list = Activator.CreateInstance(closedType)!;

Console.WriteLine(list.GetType()); // System.Collections.Generic.List`1[System.Int32]
```

منبع: [Reflection and Generics](https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection-and-generics)

---

### ۱۴.۲ تشخیص Generic بودن نوع

```csharp
Type type = typeof(List<int>);

Console.WriteLine(type.IsGenericType);        // True
Console.WriteLine(type.GetGenericTypeDefinition()); // System.Collections.Generic.List`1[T]
Console.WriteLine(type.GetGenericArguments()[0]);   // System.Int32
```

---

### ۱۴.۳ محدودیت `unmanaged`

```csharp
public static int SizeOf<T>() where T : unmanaged
{
    return System.Runtime.InteropServices.Marshal.SizeOf<T>();
}
```

این محدودیت بیشتر در سناریوهای low-level و unsafe کاربرد دارد.

---

### ۱۴.۴ محدودیت `notnull`

```csharp
public void Add<T>(T item) where T : notnull
{
    if (item is null)
    {
        throw new ArgumentNullException(nameof(item));
    }
}
```

---

### ۱۴.۵ محدودیت `Enum`

```csharp
public static string GetEnumName<TEnum>(TEnum value) where TEnum : Enum
{
    return value.ToString();
}
```

---

### ۱۴.۶ محدودیت `Delegate`

```csharp
public static void Invoke<TDelegate>(TDelegate action) where TDelegate : Delegate
{
    action.DynamicInvoke();
}
```

---

### ۱۴.۷ Generic و Nullable Value Typeها

برای Value Typeها می‌توان از `Nullable<T>` استفاده کرد.

```csharp
int? nullableNumber = null;
```

یعنی:

```csharp
Nullable<int> nullableNumber = null;
```

در متد Generic:

```csharp
using System.Collections.Generic;

public static T? FirstOrNull<T>(IEnumerable<T> source) where T : struct
{
    foreach (T item in source)
    {
        return item;
    }

    return null;
}
```

استفاده:

```csharp
var numbers = new List<int> { 5, 8 };
var empty = new List<int>();

int? first1 = FirstOrNull(numbers); // 5
int? first2 = FirstOrNull(empty);   // null
```

منبع: [Nullable Value Types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)

---

### ۱۴.۸ تفاوت Generic در C# با بعضی زبان‌ها

در سی‌شارپ Genericها در Runtime نیز نوع واقعی را حفظ می‌کنند.  
به این مفهوم **Reified Generics** گفته می‌شود.

یعنی:

```csharp
typeof(List<int>) != typeof(List<string>)
```

این دو، نوع‌های متفاوتی در Runtime هستند.

---

### ۱۴.۹ نکات عملکردی

Genericها معمولاً عملکرد را بهتر می‌کنند چون:

- Cast حذف می‌شود.
- Boxing/Unboxing برای Value Typeها کاهش می‌یابد.
- کامپایلر و JIT می‌توانند کد بهینه‌تری تولید کنند.

مثلاً:

```csharp
List<int> numbers = new List<int>();
```

بهتر از:

```csharp
ArrayList numbers = new ArrayList();
```

است، چون برای `int` نیازی به Boxing نیست.

---

<a id="mistakes"></a>
## ۱۵) اشتباه‌های رایج

---

### ۱۵.۱ استفاده از `object` به‌جای Generic

بد:

```csharp
public void Print(object value)
{
}
```

بهتر:

```csharp
public void Print<T>(T value)
{
}
```

یا حتی:

```csharp
public void Print(string value)
{
}
```

اگر نوع مشخص است، Generic بی‌دلیل استفاده نکنید.

---

### ۱۵.۲ فراموش کردن Constraint

```csharp
public static T Create<T>()
{
    return new T(); // خطا
}
```

درست:

```csharp
public static T Create<T>() where T : new()
{
    return new T();
}
```

---

### ۱۵.۳ انتظار تبدیل بین کالکشن‌ها

```csharp
List<string> names = new List<string>();

// List<object> objects = names; // خطا
```

`List<T>` invariant است.

اما:

```csharp
IEnumerable<string> names = new List<string>();
IEnumerable<object> objects = names; // OK
```

---

### ۱۵.۴ استفاده زیاد از Generic بدون نیاز

Genericها عالی هستند، ولی نباید همیشه بدون دلیل استفاده شوند.  
اگر نوع مشخص است، کد ساده‌تر معمولاً بهتر است.

---

### ۱۵.۵ اشتباه گرفتن `default(T)` با `null`

برای Value Typeها:

```csharp
default(int) == 0
```

نه `null`.

---

### ۱۵.۶ استفاده از عملگرها روی `T`

```csharp
public static T Add<T>(T a, T b)
{
    return a + b; // خطا
}
```

چون کامپایلر نمی‌داند `T` از عملگر `+` پشتیبانی می‌کند یا نه.

راه‌حل ساده: استفاده از Constraintهای مناسب یا طراحی خاص.  
در C#های جدیدتر برای سناریوهای پیشرفته می‌توان از static abstract interface members استفاده کرد، اما این موضوع پیشرفته است.

---

<a id="best-practices"></a>
## ۱۶) بهترین روش‌ها

1. **از کالکشن‌های Generic استفاده کنید**  
   `List<T>`، `Dictionary<TKey, TValue>`، `HashSet<T>` و غیره.

2. **نام Type Parameter را معنادار انتخاب کنید**  
   `TEntity`، `TKey`، `TValue`، `TResult`.

3. **تا حد ممکن از Constraint استفاده کنید**  
   Constraintها قرارداد کد را واضح‌تر می‌کنند.

4. **برای APIهای عمومی از رابط‌های Generic استفاده کنید**  
   مثل `IEnumerable<T>`، `IReadOnlyList<T>`.

5. **اگر نوع مشخص است، Generic بی‌دلیل اضافه نکنید**  
   خوانایی مهم است.

6. **از `IEnumerable<T>` برای پیمایش استفاده کنید**  
   نه لزوماً `List<T>` در امضای متدها.

7. **برای خروجی فقط‌خواندنی از `IReadOnlyList<T>` استفاده کنید**

8. **Variance را فقط وقتی نیاز دارید استفاده کنید**

9. **مستندسازی کنید که `T` چه انتظاری دارد**

10. **Generic را با OOP ترکیب کنید، اما پیچیدگی را زیاد نکنید**

---

<a id="exercises"></a>
## ۱۷) تمرین‌های پیشنهادی برای ریپازیتوری آموزشی

این تمرین‌ها برای مخزن آموزش OOP مناسب هستند.

---

### تمرین ۱: ساخت `Box<T>`

یک کلاس Generic بسازید که یک مقدار از نوع `T` نگه دارد.

```csharp
public class Box<T>
{
    public T Value { get; set; }
}
```

هدف:
- آشنایی با کلاس Generic
- آشنایی با Property

---

### تمرین ۲: ساخت `Pair<TFirst, TSecond>`

یک کلاس با دو Type Parameter بسازید.

هدف:
- کار با چند پارامتر نوع

---

### تمرین ۳: متد `Swap<T>`

یک متد Generic بنویسید که دو مقدار را جابه‌جا کند.

هدف:
- متد Generic
- کار با `ref`

---

### تمرین ۴: متد `Max<T>`

متدی بنویسید که بیشترین مقدار یک مجموعه را برگرداند.

Constraint:

```csharp
where T : IComparable<T>
```

هدف:
- آشنایی با Constraint
- آشنایی با `IComparable<T>`

---

### تمرین ۵: ساخت `IStorage<T>`

یک رابط Generic با متدهای `Add`، `Remove` و `Count` بسازید.

سپس دو پیاده‌سازی داشته باشید:

- `MemoryStorage<T>`
- `LoggedStorage<T>` که فقط لاگ می‌کند و عملیات را به یک Storage دیگر می‌سپارد.

هدف:
- Interface Generic
- Polymorphism
- Composition

---

### تمرین ۶: ساخت `InMemoryRepository<T>`

یک Repository Generic بسازید که موجودیت‌ها را در `List<T>` نگه دارد.

الزامات:

```csharp
where TEntity : class, IEntity
```

متدها:

```csharp
Add
GetAll
FindById
RemoveById
```

هدف:
- ترکیب Generic با OOP
- تمرین معماری ساده

---

### تمرین ۷: کار با Variance

یک مثال بنویسید که نشان دهد:

```csharp
IEnumerable<string> -> IEnumerable<object>
```

مجاز است، اما:

```csharp
IList<string> -> IList<object>
```

مجاز نیست.

هدف:
- درک Covariance و invariance

---

### تمرین ۸: ساخت Delegate Generic

یک Delegate Generic بسازید:

```csharp
public delegate void Processor<T>(T item);
```

سپس با Lambda از آن استفاده کنید.

هدف:
- آشنایی با Delegateهای Generic

---

<a id="sources"></a>
## ۱۸) منابع معتبر

این سند بر اساس مستندات رسمی Microsoft Learn و منابع رسمی زبان C# تنظیم شده است.

### منابع اصلی Generic در C#

1. **C# Programming Guide - Generics**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)

2. **Generic Type Parameters**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters)

3. **Generic Classes**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes)

4. **Generic Methods**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods)

5. **Generic Interfaces**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces)

6. **Generic Delegates**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates)

7. **Constraints on Type Parameters**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)

8. **Covariance and Contravariance**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/)

---

### منابع کالکشن‌ها و تایپ‌های کاربردی

9. **System.Collections.Generic Namespace**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic)

10. **List<T> Class**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)

11. **Dictionary<TKey,TValue> Class**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2)

12. **IEnumerable<T> Interface**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1)

13. **IReadOnlyList<T> Interface**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlylist-1](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlylist-1)

14. **Commonly Used Collection Types**  
   [https://learn.microsoft.com/en-us/dotnet/standard/collections/commonly-used-collection-types](https://learn.microsoft.com/en-us/dotnet/standard/collections/commonly-used-collection-types)

---

### منابع Delegateها و Interfaceهای معروف

15. **Action<T> Delegate**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.action-1](https://learn.microsoft.com/en-us/dotnet/api/system.action-1)

16. **Func<T,TResult> Delegate**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.func-2](https://learn.microsoft.com/en-us/dotnet/api/system.func-2)

17. **Predicate<T> Delegate**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.predicate-1](https://learn.microsoft.com/en-us/dotnet/api/system.predicate-1)

18. **IComparable<T> Interface**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.icomparable-1](https://learn.microsoft.com/en-us/dotnet/api/system.icomparable-1)

19. **IEquatable<T> Interface**  
   [https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1](https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1)

---

### منابع Nullable، default و Reflection

20. **Nullable Value Types**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)

21. **default Operator**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/default](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/default)

22. **typeof Operator**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#typeof-operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#typeof-operator)

23. **Reflection and Generics**  
   [https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection-and-generics](https://learn.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection-and-generics)

---

### منابع رسمی زبان C#

24. **C# Language Specification - Microsoft Learn**  
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/)

25. **ECMA-334 C# Language Specification**  
   [https://www.ecma-international.org/publications-and-standards/standards/ecma-334/](https://www.ecma-international.org/publications-and-standards/standards/ecma-334/)

