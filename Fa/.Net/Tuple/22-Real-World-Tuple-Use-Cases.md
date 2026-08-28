# 📚 مقایسه جامع Tuple و Record در #C

در زبان سی‌شارپ، هم **Tuple** (معرفی شده در C# 7.0) و هم **Record** (معرفی شده در C# 9.0) برای گروه‌بندی داده‌ها استفاده می‌شوند. اما با وجود شباهت‌های ظاهری، این دو ویژگی فلسفه، کاربرد و رفتار کاملاً متفاوتی دارند. 

در این مقاله آموزشی، این دو مفهوم را از ۱۷ زاویه مختلف بررسی می‌کنیم تا دقیقاً بدانید در هر سناریو کدام‌یک را انتخاب کنید.

---

## 📊 جدول مقایسه سریع

| ویژگی | Tuple `(T1, T2)` | Record `record Name(T1, T2)` |
| :--- | :--- | :--- |
| **Value Equality** | ✅ دارد (مقایسه تک‌تک عناصر) | ✅ دارد (مقایسه عمیق Properties) |
| **Immutability** | ❌ ندارد (Mutable است) | ✅ دارد (Init-only Properties) |
| **Mutability** | ✅ فیلدها Public و قابل تغییرند | ❌ پیش‌فرض تغییرناپذیر است |
| **`with` Expression** | ❌ پشتیبانی نمی‌شود | ✅ پشتیبانی می‌شود (Non-destructive mutation) |
| **Deconstruction** | ✅ دارد | ✅ دارد (برای Positional Records) |
| **Type Identity** | ساختاری (Structural) | اسمی (Nominal) |
| **Named Type** | ❌ خیر (نوع `ValueTuple` است) | ✅ بله (نام اختصاصی دارد) |
| **Method** | ❌ نمی‌تواند متد داشته باشد | ✅ می‌تواند متد و رفتار داشته باشد |
| **Constructor** | ❌ (استفاده از Constructor نوع پایه) | ✅ (Primary Constructor خودکار) |
| **Property** | ❌ (فقط Public Field دارد) | ✅ (Property های Init-only) |
| **Behavior** | ❌ فقط حامل داده (Passive) | ✅ می‌تواند رفتار و Validation داشته باشد |
| **Domain Modeling** | ❌ نامناسب (نشت انتزاع) | ✅ عالی (Entities, Value Objects) |
| **API Contract** | ❌ نامناسب برای API عمومی | ✅ عالی (DTOs, Responses) |
| **Serialization** | ❌ ضعیف (نام‌های `Item1`, `Item2`) | ✅ عالی (بر اساس نام Propertyها) |
| **Maintainability** | ⚠️ پایین (در استفاده‌های پیچیده) | ✅ بالا (تایپ‌سیف، مستند و واضح) |
| **Performance** | 🚀 بسیار بالا (Struct، بدون GC) | ⚠️ متوسط (Reference Type، دارای GC)* |
| **قابلیت توسعه** | ❌ ارث‌بری و پیاده‌سازی Interface ندارد | ✅ ارث‌بری، Interface، اکستنشن |

*\*نکته: `record struct` وجود دارد که مانند Tuple روی Stack قرار می‌گیرد، اما `record` کلاسیک یک Reference Type است.*

---

## 🔍 بررسی عمیق تفاوت‌ها

### ۱. هویت نوع (Type Identity) و Named Type
*   **Tuple**: هویت نوع در Tuple **ساختاری (Structural)** است. یعنی `(int Id, string Name)` و `(int UserId, string FullName)` در زمان اجرا (Runtime) دقیقاً یک نوع واحد (`ValueTuple<int, string>`) هستند. نام عناصر فقط در زمان کامپایل برای راحتی شماست.
*   **Record**: هویت نوع **اسمی (Nominal)** است. یک `record User` با `record Admin` حتی اگر فیلدهای یکسانی داشته باشند، دو نوع کاملاً مجزا هستند. همچنین Record یک **Named Type** است که می‌توانید برای آن نام معنادار انتخاب کنید.

### ۲. تغییرناپذیری (Immutability) و `with`
*   **Tuple**: به‌صورت پیش‌فرض **Mutable** است. شما می‌توانید `myTuple.Item1 = 10;` را صدا بزنید.
*   **Record**: به‌صورت پیش‌فرض **Immutable** است. Properties از نوع `init` یا `get` هستند. اگر نیاز به تغییر داشته باشید، از عبارت `with` استفاده می‌کنید که یک کپی جدید با تغییرات دلخواه می‌سازد (Non-destructive mutation).

### ۳. رفتار (Behavior) و Domain Modeling
*   **Tuple**: صرفاً یک ظرف برای داده است. نمی‌توانید برای آن متد، Validation یا منطق تجاری تعریف کنید. استفاده از آن در Domain Modeling باعث ایجاد "Leaky Abstractions" می‌شود.
*   **Record**: یک کلاس/ساختار کامل است. می‌تواند متد، Property های محاسباتی، و منطق Validation در Constructor داشته باشد. برای مدل‌سازی دامنه (مثل Value Objects در DDD) بی‌نظیر است.

### ۴. قرارداد API و Serialization
*   **Tuple**: برای APIهای عمومی سم است! وقتی یک Tuple را به JSON تبدیل می‌کنید، کلیدها `Item1`, `Item2` خواهند بود که برای مصرف‌کننده API هیچ معنایی ندارد. همچنین تغییر ترتیب عناصر در Tuple باعث Breaking Change می‌شود.
*   **Record**: نام Properties دقیقاً در JSON منعکس می‌شود. قرارداد API شفاف، مستند و قابل تغییر (با اضافه کردن فیلدهای Optional) است.

---

## 💻 سناریوهای واقعی (Real-World Scenarios)

### سناریو ۱: بازگرداندن چند مقدار از یک متد داخلی (استفاده از Tuple)
وقتی می‌خواهید از یک متد `private` چند خروجی بگیرید و نیازی به انتقال آن به لایه‌های دیگر ندارید، Tuple بهترین و سبک‌ترین گزینه است.

```csharp
// استفاده صحیح از Tuple
private (bool isSuccess, string errorMessage) ValidateUser(string username)
{
    if (string.IsNullOrEmpty(username))
        return (false, "Username is required");
        
    return (true, string.Empty);
}

// نحوه استفاده
var (isSuccess, error) = ValidateUser("Ali");
```

### سناریو ۲: تعریف DTO برای پاسخ API (استفاده از Record)
وقتی می‌خواهید داده‌ای را از طریق Web API به کلاینت بفرستید، حتماً از Record استفاده کنید تا نام فیلدها حفظ شده و قابلیت Serialize شدن داشته باشد.

```csharp
// استفاده صحیح از Record
public record UserResponse(int Id, string FullName, string Email);

// در Controller
app.MapGet("/users/{id}", (int id) => 
{
    return new UserResponse(id, "Ali Ahmadi", "ali@example.com");
});
```

### سناریو ۳: کلید دیکشنری یا کش (Cache Key)
اگر نیاز به یک کلید ترکیبی دارید، Record به دلیل داشتن `Value Equality` و `GetHashCode` خودکار، گزینه امن‌تری نسبت به Tuple است (هرچند Tuple هم کار می‌کند، اما Record معنای بهتری می‌رساند).

```csharp
// Record برای کلید کش
public record CacheKey(string TenantId, string ResourceId);

var cache = new Dictionary<CacheKey, string>();
cache.Add(new CacheKey("T1", "R1"), "Data");
```

### سناریو ۴: پروژه‌های LINQ و گروه‌بندی موقت (استفاده از Tuple)
در کوئری‌های LINQ وقتی می‌خواهید بر اساس چند فیلد گروه‌بندی کنید، Tuple بسیار تمیزتر از ساختن یک کلاس موقت است.

```csharp
var groupedData = orders
    .GroupBy(o => (o.CustomerId, o.ProductId))
    .Select(g => new 
    {
        Key = g.Key, // یک Tuple است
        TotalAmount = g.Sum(o => o.Amount)
    });
```

---

## 🧭 راهنمای انتخاب (Decision Guide)

برای تصمیم‌گیری سریع، از این چک‌لیست استفاده کنید:

### 🟢 از **Tuple** استفاده کنید اگر:
1. داده‌ها فقط در **داخل یک متد یا کلاس** (Internal/Private) جابجا می‌شوند.
2. نیاز به بازگرداندن **چند مقدار** از یک متد دارید (جایگزین `out parameters`).
3. در حال نوشتن **کوئری‌های LINQ** هستید و نیاز به گروه‌بندی (GroupBy) بر اساس چند فیلد دارید.
4. **Performance** در حلقه‌های سنگین (Hot paths) حیاتی است و می‌خواهید از تخصیص Heap (GC) جلوگیری کنید.
5. داده‌ها صرفاً موقت هستند و **معنای دامنه‌ای (Domain Meaning)** خاصی ندارند.

### 🔵 از **Record** استفاده کنید اگر:
1. داده‌ها از **مرزهای کلاس یا لایه‌های برنامه** عبور می‌کنند (Public API, DTOs).
2. نیاز به **Serialization** (JSON, XML) دارید.
3. در حال **مدل‌سازی دامنه (Domain Modeling)** هستید (Entities, Value Objects).
4. نیاز به **تغییرناپذیری (Immutability)** و استفاده از `with` دارید.
5. می‌خواهید برای داده‌های خود **رفتار (Behavior)**، متد یا Validation تعریف کنید.
6. **Maintainability** و خوانایی کد برای تیم اولویت دارد.

---

## 📚 منابع معتبر برای مطالعه بیشتر

برای تسلط بیشتر، پیشنهاد می‌شود مستندات رسمی زیر را مطالعه کنید:

### Microsoft Learn
*   [Records (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) - بررسی کامل ویژگی‌های Record.
*   [Tuples (C# Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples) - راهنمای استفاده از ValueTuple.
*   [with expression - C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression) - نحوه استفاده از تغییر غیرمخرب.
*   [Deconstructing tuples and types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct) - آموزش Deconstruction.

### C# Language Specification
*   [C# Language Specification - Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-record-declarations) - جزئیات کامپایلر و Spec برای Recordها.
*   [C# Language Specification - Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#12815-tuple-expressions) - نحوه ارزیابی Tupleها در سطح زبان.
*   [C# Standard - ECMA-334](https://www.ecma-international.org/publications-and-standards/standards/ecma-334/) - استاندارد رسمی زبان سی‌شارپ.

---
*این مقاله بخشی از Repository آموزشی C# است. در صورت داشتن سوال یا پیشنهاد برای بهبود، خوشحال می‌شویم در بخش Issues مطرح کنید یا Pull Request بفرستید!* 🚀