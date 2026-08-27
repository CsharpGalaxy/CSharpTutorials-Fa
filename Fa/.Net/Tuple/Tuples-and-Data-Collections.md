# 📘 Tuple در Collectionها در C# — راهنمای جامع

---

## فهرست مطالب

- [۱. مقدمه](#۱-مقدمه)
- [۲. پیش‌نیازها: تفاوت Tuple و ValueTuple](#۲-پیشنیازها-تفاوت-tuple-و-valuetuple)
- [۳. List\<Tuple\>](#۳-listtuple)
- [۴. List\<ValueTuple\>](#۴-listvaluetuple)
- [۵. Dictionary با Tuple Key](#۵-dictionary-با-tuple-key)
- [۶. Dictionary با Tuple به‌عنوان Value](#۶-dictionary-با-tuple-بهعنوان-value)
- [۷. HashSet با Tuple](#۷-hashset-با-tuple)
- [۸. Queue\<Tuple\>](#۸-queuetuple)
- [۹. Stack\<Tuple\>](#۹-stacktuple)
- [۱۰. SortedSet با Tuple](#۱۰-sortedset-با-tuple)
- [۱۱. SortedDictionary با Tuple Key](#۱۱-sorteddictionary-با-tuple-key)
- [۱۲. Composite Key با Tuple](#۱۲-composite-key-با-tuple)
- [۱۳. Tupleهای نام‌گذاری‌شده در Collection](#۱۳-tupleهای-نامگذاریشده-در-collection)
- [۱۴. جدول مقایسه جامع](#۱۴-جدول-مقایسه-جامع)
- [۱۵. نکات مهم و بهترین شیوه‌ها](#۱۵-نکات-مهم-و-بهترین-شیوهها)
- [۱۶. اشتباهات رایج](#۱۶-اشتباهات-رایج)
- [۱۷. جمع‌بندی](#۱۷-جمع‌بندی)
- [۱۸. منابع](#۱۸-منابع)

---

## ۱. مقدمه

در بسیاری از سناریوهای واقعی، داده‌ها به‌صورت **تکی** معنا ندارند. مثلاً وقتی می‌خواهید لیستی از «نام دانش‌آموز + نمره + کلاس» را نگه دارید، ساخت یک کلاس جداگانه برای هر ترکیب ساده، گاهی زیاده‌روی است. اینجاست که **Tuple** وارد میدان می‌شود.

ترکیب Tuple با Collectionهای .NET به شما اجازه می‌دهد **ساختارهای داده‌ای سبک، سریع و خوانا** بسازید بدون آن‌که نیاز به تعریف کلاس یا struct جداگانه داشته باشید.

> **هدف این مقاله:** بررسی عملی تمام Collectionهای پرکاربرد .NET در ترکیب با Tuple و ValueTuple، همراه با مثال‌های واقعی.

---

## ۲. پیش‌نیازها: تفاوت Tuple و ValueTuple

قبل از ورود به Collectionها، تفاوت این دو نوع را مرور کنیم:

| ویژگی | `Tuple<T1, T2, ...>` | `ValueTuple<T1, T2, ...>` |
|---|---|---|
| **نوع** | Reference Type (کلاس) | Value Type (استراکت) |
| **محل ذخیره** | Heap | Stack (معمولاً) |
| **تغییرپذیری** | غیرقابل‌تغییر (Immutable) | قابل‌تغییر (Mutable) |
| **نام فیلدها** | `Item1`, `Item2`, ... | قابل نام‌گذاری دلخواه |
| **سینتکس** | `Tuple.Create(1, "A")` | `(1, "A")` یا `(Id: 1, Name: "A")` |
| **عملکرد** | کندتر (تخصیص Heap) | سریع‌تر |
| **برابری** | پیاده‌سازی شده | پیاده‌سازی شده |
| **نسخه C#** | از C# 4 | از C# 7 |

> 💡 **توصیه:** در کد جدید، **همیشه** از `ValueTuple` استفاده کنید مگر دلیل خاصی برای `Tuple` کلاسیک داشته باشید.

---

## ۳. List\<Tuple\>

### سناریو واقعی: گزارش نمرات دانش‌آموزان

فرض کنید می‌خواهید لیستی از نمرات را بدون ساخت کلاس جداگانه نگهداری کنید:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// ساخت لیست با Tuple کلاسیک
var grades = new List<Tuple<string, string, double>>
{
    Tuple.Create("علی", "ریاضی", 18.5),
    Tuple.Create("سارا", "فیزیک", 19.0),
    Tuple.Create("رضا", "ریاضی", 16.75),
    Tuple.Create("مریم", "فیزیک", 17.25),
    Tuple.Create("علی", "فیزیک", 15.0)
};

// فیلتر: نمرات بالای ۱۷
var topGrades = grades
    .Where(g => g.Item3 >= 17.0)
    .OrderByDescending(g => g.Item3);

Console.WriteLine("=== نمرات برتر ===");
foreach (var g in topGrades)
{
    Console.WriteLine($"{g.Item1} | {g.Item2} | {g.Item3}");
}
// خروجی:
// سارا | فیزیک | 19
// علی | ریاضی | 18.5
// مریم | فیزیک | 17.25
```

### ⚠️ مشکل خوانایی
همان‌طور که می‌بینید، `Item1`، `Item2` و `Item3` خوانایی کد را به‌شدت کاهش می‌دهند. این دقیقاً دلیلی است که `ValueTuple` نام‌گذاری‌شده را ترجیح می‌دهیم.

---

## ۴. List\<ValueTuple\>

### سناریو واقعی: مدیریت موجودی انبار

```csharp
// ساخت لیست با ValueTuple نام‌گذاری‌شده
var inventory = new List<(string Sku, string ProductName, int Quantity, decimal Price)>
{
    ("SKU-001", "لپ‌تاپ ایسوس", 25, 45_000_000m),
    ("SKU-002", "ماوس بی‌سیم", 150, 350_000m),
    ("SKU-003", "کیبورد مکانیکی", 80, 2_500_000m),
    ("SKU-004", "مانیتور ۲۷ اینچ", 12, 18_000_000m),
    ("SKU-005", "هدست گیمینگ", 0, 1_200_000m)
};

// موجودی کل انبار
int totalItems = inventory.Sum(i => i.Quantity);
Console.WriteLine($"موجودی کل: {totalItems} عدد");

// محصولات ناموجود
var outOfStock = inventory.Where(i => i.Quantity == 0);
foreach (var item in outOfStock)
{
    Console.WriteLine($"⚠️ {item.ProductName} ({item.Sku}) ناموجود است!");
}

// ارزش کل انبار
decimal totalValue = inventory.Sum(i => i.Quantity * i.Price);
Console.WriteLine($"ارزش کل انبار: {totalValue:N0} تومان");

// ویرایش موجودی (ValueTuple قابل‌تغییر است!)
var laptop = inventory.First(i => i.Sku == "SKU-001");
laptop.Quantity = 30; // ✅ این کار ممکن است
```

> 💡 **نکته:** `ValueTuple` چون **mutable** است، می‌توانید مقادیر آن را بعد از ساخت تغییر دهید. این رفتار با `Tuple` کلاسیک متفاوت است.

---

## ۵. Dictionary با Tuple Key

### سناریو واقعی: سیستم نمره‌دهی دانشگاه (Composite Key)

در یک دانشگاه، هر نمره با ترکیب **(شماره دانشجویی، کد درس)** یکتا می‌شود:

```csharp
// کلید ترکیبی: (شماره دانشجویی، کد درس)
var gradeBook = new Dictionary<(int StudentId, string CourseCode), double>
{
    [(4011001, "CS101")] = 18.5,
    [(4011001, "CS201")] = 17.0,
    [(4011002, "CS101")] = 19.25,
    [(4011002, "MATH101")] = 16.0,
    [(4011003, "CS101")] = 14.75
};

// دسترسی سریع با کلید ترکیبی
double aliCsGrade = gradeBook[(4011001, "CS101")];
Console.WriteLine($"نمره علی در CS101: {aliCsGrade}");

// بررسی وجود نمره
if (gradeBook.TryGetValue((4011003, "CS201"), out double grade))
{
    Console.WriteLine($"نمره: {grade}");
}
else
{
    Console.WriteLine("نمره‌ای ثبت نشده است.");
}

// میانگین نمرات یک دانشجو
var studentAvg = gradeBook
    .Where(kvp => kvp.Key.StudentId == 4011001)
    .Average(kvp => kvp.Value);
Console.WriteLine($"میانگین دانشجوی ۴۰۱۱۰۰۱: {studentAvg:F2}");
```

### چرا Tuple Key عالی کار می‌کند؟

`ValueTuple` در .NET متدهای `GetHashCode()` و `Equals()` را **به‌صورت خودکار** پیاده‌سازی می‌کند. بنابراین به‌عنوان کلید Dictionary بدون نیاز به `IEqualityComparer` سفارشی عمل می‌کند:

```csharp
var t1 = (1, "A");
var t2 = (1, "A");
Console.WriteLine(t1.Equals(t2));           // True
Console.WriteLine(t1.GetHashCode() == t2.GetHashCode()); // True
```

---

## ۶. Dictionary با Tuple به‌عنوان Value

### سناریو واقعی: دفترچه تلفن با اطلاعات تکمیلی

```csharp
// کلید: نام شخص | مقدار: (شماره تلفن، ایمیل، شهر)
var contacts = new Dictionary<string, (string Phone, string Email, string City)>
{
    ["علی احمدی"] = ("09121234567", "ali@email.com", "تهران"),
    ["سارا رضایی"] = ("09351234567", "sara@email.com", "اصفهان"),
    ["رضا کریمی"] = ("09191234567", "reza@email.com", "شیراز")
};

// دسترسی
var aliInfo = contacts["علی احمدی"];
Console.WriteLine($"تلفن: {aliInfo.Phone}");
Console.WriteLine($"شهر: {aliInfo.City}");

// Deconstruction مستقیم
foreach (var (name, (phone, email, city)) in contacts)
{
    Console.WriteLine($"{name} | {phone} | {city}");
}
```

> 💡 **نکته حرفه‌ای:** Deconstruction تو‌در‌تو (Nested Deconstruction) در C# 7+ پشتیبانی می‌شود و خوانایی کد را فوق‌العاده بالا می‌برد.

---

## ۷. HashSet با Tuple

### سناریو واقعی: سیستم لایک/علاقه‌مندی کاربران

هر کاربر فقط **یک بار** می‌تواند یک محصول را لایک کند. ترکیب `(UserId, ProductId)` باید **یکتا** باشد:

```csharp
var userLikes = new HashSet<(int UserId, int ProductId)>
{
    (1, 101),
    (1, 102),
    (2, 101),
    (3, 103)
};

// تلاش برای لایک تکراری
bool added = userLikes.Add((1, 101));
Console.WriteLine($"لایک تکراری اضافه شد؟ {added}"); // False

// لایک جدید
bool newLike = userLikes.Add((2, 103));
Console.WriteLine($"لایک جدید اضافه شد؟ {newLike}"); // True

// آیا کاربر ۳ محصول ۱۰۱ را لایک کرده؟
bool hasLiked = userLikes.Contains((3, 101));
Console.WriteLine($"کاربر ۳ محصول ۱۰۱ را لایک کرده؟ {hasLiked}"); // False

// تعداد لایک‌های هر محصول
var productLikeCounts = userLikes
    .GroupBy(x => x.ProductId)
    .Select(g => new { ProductId = g.Key, Likes = g.Count() });

foreach (var p in productLikeCounts)
{
    Console.WriteLine($"محصول {p.ProductId}: {p.Likes} لایک");
}
```

### ⚠️ هشدار مهم درباره Tuple کلاسیک در HashSet

```csharp
// ❌ Tuple کلاسیک در HashSet مشکل‌ساز است!
var badSet = new HashSet<Tuple<int, int>>
{
    Tuple.Create(1, 2)
};
Console.WriteLine(badSet.Contains(Tuple.Create(1, 2))); // True (چون Equals پیاده‌سازی شده)
// اما GetHashCode سربار بیشتری دارد و عملکرد ضعیف‌تر است.

// ✅ همیشه ValueTuple ترجیح داده می‌شود
var goodSet = new HashSet<(int, int)> { (1, 2) };
Console.WriteLine(goodSet.Contains((1, 2))); // True — سریع‌تر
```

---

## ۸. Queue\<Tuple\>

### سناریو واقعی: صف پردازش وظایف با اولویت

```csharp
// صف وظایف: (اولویت، عنوان وظیفه، زمان تخمینی به دقیقه)
var taskQueue = new Queue<(int Priority, string Title, int EstMinutes)>();

taskQueue.Enqueue((1, "بکاپ دیتابیس", 30));
taskQueue.Enqueue((3, "ارسال ایمیل خبرنامه", 5));
taskQueue.Enqueue((2, "بروزرسانی سرور", 45));
taskQueue.Enqueue((1, "پاکسازی لاگ‌ها", 10));

Console.WriteLine($"تعداد وظایف در صف: {taskQueue.Count}");

// پردازش به ترتیب FIFO
while (taskQueue.Count > 0)
{
    var task = taskQueue.Dequeue();
    Console.WriteLine($"⚙️ [{task.Priority}] {task.Title} ({task.EstMinutes} دقیقه)");
}
// خروجی:
// ⚙️ [1] بکاپ دیتابیس (30 دقیقه)
// ⚙️ [3] ارسال ایمیل خبرنامه (5 دقیقه)
// ⚙️ [2] بروزرسانی سرور (45 دقیقه)
// ⚙️ [1] پاکسازی لاگ‌ها (10 دقیقه)
```

> 💡 **نکته:** اگر نیاز به صف **اولویت‌دار** (Priority Queue) دارید، از .NET 6+ می‌توانید از `PriorityQueue<TElement, TPriority>` استفاده کنید که Tuple را به‌عنوان Element می‌پذیرد:

```csharp
// .NET 6+
var pq = new PriorityQueue<string, int>();
pq.Enqueue("بکاپ دیتابیس", 1);     // اولویت بالاتر = عدد کمتر
pq.Enqueue("ارسال ایمیل", 3);
pq.Enqueue("بروزرسانی سرور", 2);

while (pq.Count > 0)
{
    Console.WriteLine(pq.Dequeue());
}
// خروجی: بکاپ دیتابیس → بروزرسانی سرور → ارسال ایمیل
```

---

## ۹. Stack\<Tuple\>

### سناریو واقعی: تاریخچه عملیات Undo در ویرایشگر متن

```csharp
// پشته Undo: (نوع عملیات، محتوای قبلی، timestamp)
var undoStack = new Stack<(string Action, string Content, DateTime Timestamp)>();

// شبیه‌سازی عملیات کاربر
undoStack.Push(("تایپ", "سلام", DateTime.Now.AddSeconds(-30)));
undoStack.Push(("حذف", "سلام دنیا", DateTime.Now.AddSeconds(-20)));
undoStack.Push(("جایگزینی", "سلام دنیا!", DateTime.Now.AddSeconds(-10)));

Console.WriteLine($"تعداد عملیات قابل بازگشت: {undoStack.Count}");

// Undo
while (undoStack.Count > 0)
{
    var operation = undoStack.Pop();
    Console.WriteLine(
        $"↩️ Undo: {operation.Action} | " +
        $"بازگشت به: \"{operation.Content}\" | " +
        $"زمان: {operation.Timestamp:HH:mm:ss}"
    );
}
// خروجی (LIFO — آخرین عملیات اول برمی‌گردد):
// ↩️ Undo: جایگزینی | بازگشت به: "سلام دنیا!" | زمان: ...
// ↩️ Undo: حذف | بازگشت به: "سلام دنیا" | زمان: ...
// ↩️ Undo: تایپ | بازگشت به: "سلام" | زمان: ...
```

---

## ۱۰. SortedSet با Tuple

### سناریو واقعی: نگهداری مختصات یکتا و مرتب‌شده

`SortedSet` عناصر را **مرتب** و **یکتا** نگه می‌دارد. برای Tuple، باید یک `IComparer` سفارشی تعریف کنید:

```csharp
// مقایسه‌گر سفارشی برای مختصات (x, y)
var comparer = Comparer<(int X, int Y)>.Create((a, b) =>
{
    int cmp = a.X.CompareTo(b.X);
    return cmp != 0 ? cmp : a.Y.CompareTo(b.Y);
});

var coordinates = new SortedSet<(int X, int Y)>(comparer)
{
    (3, 5),
    (1, 2),
    (3, 1),
    (1, 2),  // تکراری — اضافه نمی‌شود
    (2, 4)
};

Console.WriteLine($"تعداد مختصات یکتا: {coordinates.Count}"); // 4

foreach (var coord in coordinates)
{
    Console.WriteLine($"({coord.X}, {coord.Y})");
}
// خروجی (مرتب‌شده):
// (1, 2)
// (2, 4)
// (3, 1)
// (3, 5)
```

> ⚠️ **نکته مهم:** `SortedSet` برعکس `HashSet`، از `IComparer` استفاده می‌کند نه `GetHashCode`. بنابراین **حتماً** باید مقایسه‌گر صریح ارائه دهید.

---

## ۱۱. SortedDictionary با Tuple Key

### سناریو واقعی: جدول زمان‌بندی جلسات بر اساس (تاریخ، ساعت)

```csharp
var schedule = new SortedDictionary<(DateOnly Date, TimeOnly Time), string>
{
    [(new(1405, 6, 5), new(9, 0))] = "جلسه تیم توسعه",
    [(new(1405, 6, 5), new(14, 30))] = "بررسی کد",
    [(new(1405, 6, 4), new(10, 0))] = "جلسه با مشتری",
    [(new(1405, 6, 6), new(11, 0))] = "ارائه اسپرینت"
};

// خروجی خودکار به ترتیب تاریخ و ساعت
foreach (var (key, meeting) in schedule)
{
    Console.WriteLine($"📅 {key.Date} ساعت {key.Time} — {meeting}");
}
// خروجی:
// 📅 1405-06-04 ساعت 10:00 — جلسه با مشتری
// 📅 1405-06-05 ساعت 09:00 — جلسه تیم توسعه
// 📅 1405-06-05 ساعت 14:30 — بررسی کد
// 📅 1405-06-06 ساعت 11:00 — ارائه اسپرینت
```

---

## ۱۲. Composite Key با Tuple

### مفهوم

**Composite Key** (کلید ترکیبی) زمانی لازم است که یک فیلد به‌تنهایی برای شناسایی یکتای یک رکورد کافی نباشد. Tuple بهترین گزینه برای پیاده‌سازی Composite Key در حافظه است.

### سناریو واقعی: سیستم رزرو صندلی سینما

هر صندلی با ترکیب **(سالن، ردیف، شماره صندلی)** مشخص می‌شود:

```csharp
var reservations = new Dictionary<(string Hall, int Row, int Seat), (string Customer, bool IsVip)>
{
    [("سالن ۱", 3, 12)] = ("علی احمدی", false),
    [("سالن ۱", 3, 13)] = ("سارا رضایی", true),
    [("سالن ۲", 1, 1)] = ("رضا کریمی", true),
    [("سالن ۱", 5, 7)] = ("مریم حسینی", false)
};

// رزرو صندلی جدید
var newKey = ("سالن ۱", 3, 14);
if (!reservations.ContainsKey(newKey))
{
    reservations[newKey] = ("حسین نوری", false);
    Console.WriteLine("✅ صندلی رزرو شد.");
}
else
{
    Console.WriteLine("❌ این صندلی قبلاً رزرو شده است.");
}

// جستجوی تمام صندلی‌های VIP یک سالن
var vipSeats = reservations
    .Where(r => r.Key.Hall == "سالن ۱" && r.Value.IsVip)
    .Select(r => $"ردیف {r.Key.Row} صندلی {r.Key.Seat} — {r.Value.Customer}");

Console.WriteLine("\nصندلی‌های VIP سالن ۱:");
foreach (var seat in vipSeats)
    Console.WriteLine($"  ⭐ {seat}");
```

---

## ۱۳. Tupleهای نام‌گذاری‌شده در Collection

### چرا نام‌گذاری مهم است؟

```csharp
// ❌ بدون نام — کد غیرقابل‌خوانا
var bad = new List<(int, string, decimal)> { (1, "لپ‌تاپ", 45000000) };
Console.WriteLine(bad[0].Item1);  // چیست Item1؟

// ✅ با نام — کد خودمستند
var good = new List<(int Id, string Name, decimal Price)> { (1, "لپ‌تاپ", 45000000) };
Console.WriteLine(good[0].Name);  // واضح و خوانا
```

### Deconstruction در حلقه‌ها

```csharp
var employees = new List<(int Id, string Name, string Department, decimal Salary)>
{
    (101, "علی", "فنی", 35_000_000),
    (102, "سارا", "مالی", 30_000_000),
    (103, "رضا", "فنی", 40_000_000)
};

// Deconstruction مستقیم در foreach
foreach (var (id, name, dept, salary) in employees)
{
    Console.WriteLine($"[{id}] {name} | {dept} | {salary:N0} تومان");
}

// Deconstruction در LINQ
var techSalaries = employees
    .Where(e => e.Department == "فنی")
    .Select(e => (e.Name, e.Salary));

foreach (var (name, salary) in techSalaries)
{
    Console.WriteLine($"{name}: {salary:N0}");
}
```

### ⚠️ یک نکته ظریف درباره نام‌ها

نام‌های Tuple در ValueTuple **فقط در زمان کامپایل** وجود دارند و در Runtime از بین می‌روند. بنابراین:

```csharp
var list = new List<(int Id, string Name)> { (1, "علی") };

// این دو خط دقیقاً یکسان عمل می‌کنند:
Console.WriteLine(list[0].Id);    // ✅ در زمان کامپایل
Console.WriteLine(list[0].Item1); // ✅ همیشه کار می‌کند

// اما در Reflection نام‌ها دیده نمی‌شوند!
var type = list[0].GetType();
Console.WriteLine(type.GetFields()[0].Name); // "Item1" نه "Id"
```

---

## ۱۴. جدول مقایسه جامع

| Collection | Tuple به‌عنوان | مرتب‌سازی | یکتایی | ترتیب درج | سناریوی مناسب |
|---|---|---|---|---|---|
| `List<T>` | عنصر | ❌ خیر | ❌ خیر | ✅ بله | لیست ساده رکوردها |
| `Dictionary<K,V>` | Key یا Value | ❌ خیر | ✅ Key یکتا | ❌ خیر | جستجوی سریع با کلید ترکیبی |
| `HashSet<T>` | عنصر | ❌ خیر | ✅ بله | ❌ خیر | مجموعه مقادیر یکتا |
| `Queue<T>` | عنصر | ❌ خیر | ❌ خیر | ✅ FIFO | صف پردازش |
| `Stack<T>` | عنصر | ❌ خیر | ❌ خیر | ✅ LIFO | تاریخچه Undo |
| `SortedSet<T>` | عنصر | ✅ بله | ✅ بله | ❌ خیر | مختصات مرتب یکتا |
| `SortedDictionary<K,V>` | Key | ✅ بله | ✅ Key یکتا | ❌ خیر | زمان‌بندی مرتب |

---

## ۱۵. نکات مهم و بهترین شیوه‌ها

### ✅ ۱. همیشه ValueTuple را ترجیح دهید
```csharp
// ✅ خوب
var list = new List<(int Id, string Name)>();

// ❌ اجتناب کنید (مگر در کد قدیمی)
var list = new List<Tuple<int, string>>();
```

### ✅ ۲. از نام‌گذاری معنادار استفاده کنید
```csharp
// ✅
(int StudentId, string CourseCode, double Grade)

// ❌
(int, string, double)
```

### ✅ ۳. برای بیش از ۳-۴ فیلد، Record یا Class بسازید
```csharp
// ❌ Tuple با ۷ فیلد — غیرقابل‌نگهداری
(string, string, int, decimal, DateTime, bool, string)

// ✅ Record بسازید
record Product(string Sku, string Name, int Qty, decimal Price,
               DateTime Created, bool Active, string Category);
```

### ✅ ۴. برای SortedSet/SortedDictionary مقایسه‌گر صریح بنویسید
```csharp
var set = new SortedSet<(int, int)>(
    Comparer<(int, int)>.Create((a, b) =>
        a.Item1 != b.Item1 ? a.Item1.CompareTo(b.Item1)
                           : a.Item2.CompareTo(b.Item2))
);
```

### ✅ ۵. مراقب Box شدن ValueTuple باشید
وقتی `ValueTuple` را در یک Collection غیرجنریک (مثل `ArrayList`) یا یک `IEnumerable` بدون نوع قرار می‌دهید، **Box** می‌شود و مزیت عملکردی خود را از دست می‌دهد.

---

## ۱۶. اشتباهات رایج

### ❌ اشتباه ۱: استفاده از Tuple کلاسیک به‌عنوان کلید Dictionary بدون درک سربار

```csharp
// ❌ سربار Heap برای هر کلید
var dict = new Dictionary<Tuple<int, int>, string>();
dict[Tuple.Create(1, 2)] = "A"; // تخصیص روی Heap!

// ✅ بدون سربار اضافی
var dict2 = new Dictionary<(int, int), string>();
dict2[(1, 2)] = "A"; // روی Stack
```

### ❌ اشتباه ۲: فراموش کردن IComparer برای SortedSet

```csharp
// ❌ خطای زمان اجرا یا رفتار غیرمنتظره
var set = new SortedSet<(int, int)>();
// ممکن است با Comparer پیش‌فرض کار کند اما رفتار مرتب‌سازی تضمین‌شده نیست

// ✅ همیشه صریح بنویسید
var set2 = new SortedSet<(int, int)>(
    Comparer<(int, int)>.Create((a, b) => ...));
```

### ❌ اشتباه ۳: تغییر ValueTuple داخل foreach

```csharp
var list = new List<(int Id, string Name)> { (1, "علی"), (2, "سارا") };

// ❌ این کار مقدار لیست را تغییر نمی‌دهد! (کپی برمی‌گردد)
foreach (var item in list)
{
    item.Name = "تغییر"; // ❌ خطای کامپایل: item فقط‌خواندنی است
}

// ✅ با ایندکس کار کنید
for (int i = 0; i < list.Count; i++)
{
    var item = list[i];
    list[i] = (item.Id, "تغییر"); // ✅ جایگزینی کامل
}
```

### ❌ اشتباه ۴: مقایسه Tuple با `==` در سناریوهای پیچیده

```csharp
var t1 = (1, "A");
var t2 = (1, "A");
Console.WriteLine(t1 == t2); // ✅ True — برای ValueTuple کار می‌کند

// اما برای Tuple کلاسیک:
var t3 = Tuple.Create(1, "A");
var t4 = Tuple.Create(1, "A");
// Console.WriteLine(t3 == t4); // ❌ خطای کامپایل! باید Equals استفاده کنید
Console.WriteLine(t3.Equals(t4)); // ✅ True
```

### ❌ اشتباه ۵: استفاده از Tuple به‌جای مدل دامنه

```csharp
// ❌ Tuple در لایه‌های مختلف اپلیکیشن — کد شکننده
List<(int, string, decimal)> GetOrders() { ... }

// ✅ Tuple فقط برای سناریوهای محلی و موقت
// برای انتقال بین لایه‌ها از DTO یا Record استفاده کنید
record OrderDto(int Id, string Product, decimal Amount);
```

---

## ۱۷. جمع‌بندی

| نکته کلیدی | توضیح |
|---|---|
| **ValueTuple > Tuple** | سریع‌تر، خواناتر، قابل نام‌گذاری |
| **Dictionary + Tuple Key** | بهترین راه برای Composite Key در حافظه |
| **HashSet + Tuple** | یکتایی ترکیبی بدون کلاس اضافه |
| **Queue/Stack + Tuple** | ساختارهای داده سبک برای پردازش و تاریخچه |
| **SortedSet/SortedDictionary** | نیاز به `IComparer` صریح دارند |
| **محدودیت فیلدها** | بیش از ۳-۴ فیلد → Record/Class بسازید |
| **دامنه استفاده** | Tuple برای کد محلی و موقت، نه مدل دامنه |

> 🎯 **قانون طلایی:** Tuple در Collectionها ابزاری عالی برای **کد سریع، موقت و محلی** است. به محض اینکه ساختار داده‌ای شما پیچیده‌تر شد یا بین لایه‌ها حرکت کرد، به سراغ `record` یا `class` بروید.

---

## ۱۸. منابع

- 📖 [مستندات رسمی مایکروسافت — Tuple Types (C#)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
- 📖 [مستندات رسمی — System.Tuple](https://learn.microsoft.com/en-us/dotnet/api/system.tuple)
- 📖 [مستندات رسمی — Collections Overview](https://learn.microsoft.com/en-us/dotnet/standard/collections/)
- 📖 [مستندات رسمی — Dictionary\<TKey,TValue\>](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2)
- 📖 [C# in Depth — Jon Skeet (فصل Tupleها)](https://csharpindepth.com/)
- 📖 [Pro C# 10 with .NET 6 — Andrew Troelsen](https://www.apress.com/gp/book/9781484278680)

---

> **ساخته شده با ❤️ برای یادگیری بهتر C#**
> اگر این مقاله مفید بود، ⭐ Repository را ستاره‌دار کنید!