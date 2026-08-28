# آموزش جامع `readonly record struct` در C#

از زمانی که C# 10 معرفی شد، قابلیت `record` برای `struct`ها نیز اضافه شد. ترکیب `readonly` و `record struct` یکی از قدرتمندترین ویژگی‌های مدرن C# برای ایجاد انواع داده‌ای (Data Types) تغییرناپذیر، سریع و ایمن است.

در این آموزش، به صورت کامل و عمیق به بررسی `readonly record struct` می‌پردازیم.

---

## ۱. `readonly struct` چیست؟

در C#، `struct` یک **نوع مقداری (Value Type)** است. به صورت پیش‌فرض، فیلدهای یک `struct` می‌توانند تغییر کنند (Mutable). 
وقتی کلمه کلیدی `readonly` را به یک `struct` اضافه می‌کنیم، به کامپایلر می‌گوییم که **تمام فیلدهای این ساختار باید فقط خواندنی (Read-only) باشند** و هیچ متدی نباید بتواند وضعیت (State) داخلی آن را تغییر دهد.

```csharp
public readonly struct Point
{
    public readonly double X;
    public readonly double Y;

    public Point(double x, double y)
    {
        X = x;
        Y = y;
    }
}
```

---

## ۲. چرا از `readonly` استفاده می‌کنیم؟

استفاده از `readonly` برای ساختارها سه دلیل اصلی دارد:

1. **تغییرناپذیری (Immutability):** تضمین می‌کند که پس از ایجاد یک شیء، وضعیت آن در طول عمرش تغییر نمی‌کند. این موضوع باگ‌های ناشی از تغییرات ناخواسته را به شدت کاهش می‌دهد.
2. **امنیت در برنامه‌نویسی چندنخی (Thread-Safety):** اشیاء تغییرناپذیر ذاتاً Thread-Safe هستند، زیرا هیچ نخی نمی‌تواند داده‌های آن‌ها را تغییر دهد.
3. **بهبود عملکرد (Performance):** جلوگیری از ایجاد **کپی‌های تدافعی (Defensive Copies)** توسط کامپایلر (در بخش Performance توضیح داده شده است).

---

## ۳. `readonly record struct` چیست؟

`readonly record struct` ترکیبی از سه مفهوم است:
* **`struct`**: نوع مقداری (Value Type) که روی Stack (یا به صورت Inline در اشیاء مرجع) ذخیره می‌شود.
* **`record`**: سینتکس شیک برای تولید خودکار رفتارهای ارزشی (Value Semantics) مثل بررسی برابری، متد `ToString` و اکسپرشن `with`.
* **`readonly`**: تضمین تغییرناپذیری کامل.

این نوع داده، بهترین انتخاب برای **DTOها (Data Transfer Objects)**، **مقادیر دامنه (Domain Values)** و **تنظیمات (Configurations)** است.

---

## ۴. مفاهیم کلیدی

### الف) Immutability (تغییرناپذیری)
در یک `readonly record struct`، شما نمی‌توانید هیچ فیلد یا پراپرتی را پس از ساخت تغییر دهید. هر تغییری نیازمند ساخت یک نمونه جدید است.

### ب) Property Behavior (رفتار پراپرتی‌ها)
پارامترهای Positional (پارامترهای داخل پرانتز بعد از نام) به صورت خودکار پراپرتی‌هایی با دسترسی `init` تولید می‌کنند. چون ساختار `readonly` است، این پراپرتی‌ها فقط در زمان ساخت (Constructor) مقداردهی می‌شوند.

### ج) Value Semantics (مقادیر ارزشی)
برخلاف کلاس‌ها که بر اساس **آدرس حافظه (Reference)** مقایسه می‌شوند، `readonly record struct` بر اساس **مقدار داده‌ها** مقایسه می‌شود. کامپایلر به صورت خودکار `Equals`، `==`، `!=` و `GetHashCode` را پیاده‌سازی می‌کند.

### د) Copy Semantics (کپی بر اساس مقدار)
مانند هر `struct` دیگری، وقتی یک `readonly record struct` به یک متد پاس داده می‌شود یا به متغیر دیگری اختصاص می‌یابد، یک **کپی کامل** از آن ایجاد می‌شود.

---

## ۵. تفاوت رفتار Positional Property در `readonly record struct` و `record struct`

این یکی از مهم‌ترین و ظریف‌ترین تفاوت‌ها در C# 10 به بعد است:

* **در `record struct` (بدون readonly):** پارامترهای Positional پراپرتی‌هایی با `init` accessor تولید می‌کنند. اما چون خودِ `struct` تغییرپذیر (Mutable) است، شما نمی‌توانید مستقیماً پراپرتی را تغییر دهید، اما می‌توانید **کل نمونه (Instance) را با یک نمونه جدید جایگزین کنید** یا فیلدهای تغییرپذیر دیگری به ساختار اضافه کنید. این موضوع گاهی باعث رفتارهای گیج‌کننده می‌شود.
* **در `readonly record struct`:** کامپایلر می‌داند که کل ساختار تغییرناپذیر است. بنابراین، پراپرتی‌های Positional نه تنها `init` هستند، بلکه کامپایلر **هیچ راهی** برای تغییر مستقیم یا غیرمستقیم آن‌ها باز نمی‌گذارد. شما برای تغییر مقدار، **مجبور** هستید از اکسپرشن `with` استفاده کنید که یک کپی جدید با مقادیر تغییر یافته برمی‌گرداند.

**مثال تفاوت رفتار:**
```csharp
// record struct معمولی (Mutable)
public record struct MutablePoint(int X, int Y);

// readonly record struct (Immutable)
public readonly record struct ImmutablePoint(int X, int Y);

var mutable = new MutablePoint(1, 2);
// mutable.X = 5; // خطا! چون init است.
mutable = new MutablePoint(5, 2); // مجاز! چون خود struct قابل تغییر است.

var immutable = new ImmutablePoint(1, 2);
// immutable.X = 5; // خطا!
// immutable = new ImmutablePoint(5, 2); // خطا! چون struct readonly است و نمی‌توانید کل نمونه را reassign کنید (در برخی context ها).
// راه حل صحیح:
immutable = immutable with { X = 5 }; // مجاز و استاندارد
```

---

## ۶. Performance (عملکرد) و کپی‌های تدافعی

وقتی یک `struct` معمولی (غیر readonly) را به یک متد پاس می‌دهید یا متدی را روی آن صدا می‌زنید، کامپایلر C# یک **کپی تدافعی (Defensive Copy)** ایجاد می‌کند. 
چرا؟ چون کامپایلر تضمین نمی‌کند که متد صدا زده شده، ساختار را تغییر ندهد (چون struct ها Value Type هستند و تغییر در متد، روی متغیر اصلی اثر می‌گذارد). برای جلوگیری از این باگ، کامپایلر قبل از صدا زدن متد، یک کپی از struct می‌سازد.

**اما در `readonly record struct`:**
چون کامپایلر می‌داند ساختار تغییرناپذیر است، نیازی به کپی تدافعی نیست. کامپایلر به صورت خودکار پارامتر را به صورت `in` (By Reference) پاس می‌دهد. این کار **allocations (تخصیص حافظه) را به شدت کاهش داده و سرعت را بالا می‌برد.**

---

## ۷. تفاوت با `record class`

| ویژگی | `readonly record struct` | `record class` (یا `record` خالی) |
| :--- | :--- | :--- |
| **نوع داده** | Value Type (مقداری) | Reference Type (مرجعی) |
| **محل ذخیره‌سازی** | Stack (یا Inline در Heap) | Heap (با Reference روی Stack) |
| **Nullability** | نمی‌تواند null باشد (مگر `Nullable<T>`) | می‌تواند null باشد |
| **Equality** | بر اساس مقدار (Value) | بر اساس مقدار (Value) |
| **Inheritance** | ارث‌بری نمی‌کند و ارث‌بری نمی‌دهد | می‌تواند ارث‌بری کند |
| **Allocation** | بدون سربار GC (مگر در Boxing) | دارای سربار GC |

---

## ۸. مثال‌های ساده و واقعی

### مثال ساده: مختصات جغرافیایی
```csharp
public readonly record struct Coordinate(double Latitude, double Longitude)
{
    // می‌توانید متدها و پراپرتی‌های محاسباتی هم اضافه کنید
    public double DistanceTo(Coordinate other)
    {
        // محاسبات ریاضی فاصله...
        return Math.Sqrt(Math.Pow(Latitude - other.Latitude, 2) + 
                         Math.Pow(Longitude - other.Longitude, 2));
    }
}

// استفاده:
var tehran = new Coordinate(35.6892, 51.3890);
var isfahan = new Coordinate(32.6546, 51.6680);

// استفاده از with expression
var tehranModified = tehran with { Latitude = 35.7000 };
```

### مثال واقعی: پول و ارز (Money)
در سیستم‌های مالی، استفاده از `decimal` به تنهایی خطرناک است. یک `readonly record struct` می‌تواند امنیت نوع (Type Safety) را تضمین کند:

```csharp
public readonly record struct Money(decimal Amount, string Currency)
{
    public static Money operator +(Money a, Money b)
    {
        if (a.Currency != b.Currency)
            throw new InvalidOperationException("Cannot add different currencies.");
            
        return a with { Amount = a.Amount + b.Amount };
    }

    public override string ToString() => $"{Amount:F2} {Currency}";
}

// استفاده:
var price = new Money(100.50m, "USD");
var tax = new Money(9.00m, "USD");
var total = price + tax; // خروجی: 109.50 USD
```

---

## ۹. جدول مقایسه نهایی

| ویژگی | `struct` | `readonly struct` | `record struct` | `readonly record struct` | `record class` |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **نوع (Type)** | Value | Value | Value | Value | Reference |
| **تغییرپذیری (Mutability)** | Mutable | Immutable | Mutable | Immutable | Mutable |
| **برابری (Equality)** | Reference | Reference | Value | Value | Value |
| **Positional Properties** | ندارد | ندارد | `init` (روی struct mutable) | `init` (روی struct immutable) | `init` |
| **اکسپرشن `with`** | ❌ | ❌ | ✅ | ✅ | ✅ |
| **کپی تدافعی (Defensive Copy)**| دارد | **ندارد** | دارد | **ندارد** | ندارد |
| **متد `PrintMembers` / `ToString`**| پیش‌فرض | پیش‌فرض | تولید شده | تولید شده | تولید شده |

---

## ۱۰. منابع رسمی Microsoft

برای مطالعه بیشتر و عمیق‌تر، لینک‌های زیر به مستندات رسمی مایکروسافت (Microsoft Learn) مراجعه کنید:

1. **[Record types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)**
   * مرجع کامل برای درک تفاوت `record class` و `record struct`.
2. **[Structure types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct)**
   * توضیحات مربوط به `struct` و `readonly struct`.
3. **[readonly (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly)**
   * بررسی دقیق کلمه کلیدی `readonly` و تاثیر آن روی ساختارها و جلوگیری از Defensive Copies.
4. **[How to use records in C#](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/records)**
   * آموزش عملی و سناریوهای استفاده از Recordها.