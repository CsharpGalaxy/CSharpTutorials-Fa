# مقاله جامع: انتخاب صحیح بین Class، Struct، Record Class و Record Struct در #C

با معرفی `record`ها در C# 9 و `record struct`ها در C# 10، مرزهای سنتی بین انواع داده‌ای در #C کمرنگ‌تر و در عین حال قدرتمندتر شده‌اند. انتخاب بین این چهار ساختار (`class`, `struct`, `record class`, `record struct`) دیگر فقط یک انتخاب بین "Reference" و "Value" نیست، بلکه به معنای انتخاب **Semantic (معنای مفهومی)**، **عملکرد (Performance)** و **رویکرد معماری** شماست.

در این مقاله، بر اساس مستندات رسمی Microsoft Learn، به کالبدشکافی این چهار نوع داده می‌پردازیم و یک راهنمای تصمیم‌گیری قطعی ارائه می‌دهیم.

---

## بخش اول: مفاهیم پایه (Core Concepts)

قبل از بررسی انواع داده، باید تفاوت‌های بنیادین آن‌ها را در مفاهیم زیر درک کنیم:

### 1. Reference Type در برابر Value Type
*   **Reference Types (کلاس‌ها و Record Classها):** داده‌ها در **Heap** ذخیره می‌شوند و متغیرها فقط یک **Reference (اشاره‌گر)** به آن داده‌ها را نگه می‌دارند.
*   **Value Types (استراکت‌ها و Record Structها):** داده‌ها مستقیماً در **Stack** (اگر متغیر محلی باشند) یا به صورت **Inline** درون آبجکت والد (اگر فیلد یک کلاس باشند) ذخیره می‌شوند.

### 2. Memory Semantics و Allocation
*   استفاده از Value Typeها فشار (Pressure) را از روی **Garbage Collector (GC)** برمی‌دارد، زیرا نیازی به تخصیص در Heap و پاکسازی آن‌ها نیست.
*   Reference Typeها نیازمند Allocation در Heap هستند و پس از خارج شدن از Scope، باید توسط GC جمع‌آوری شوند.

### 3. Copy Semantics (سمانتیک کپی)
*   **Reference:** کپی کردن یک متغیر Reference Type، فقط یک **کپی سطحی (Shallow Copy)** از اشاره‌گر می‌سازد. هر دو متغیر به یک داده در Heap اشاره می‌کنند.
*   **Value:** کپی کردن یک Value Type، یک **کپی عمیق (Deep Copy)** از تمام فیلدها ایجاد می‌کند. تغییر یکی، تاثیری روی دیگری ندارد.

### 4. Equality (برابری)
*   **Classها:** به صورت پیش‌فرض از **Reference Equality** استفاده می‌کنند (دو آبجکت فقط در صورتی برابرند که به یک مکان در Heap اشاره کنند).
*   **Structها و Recordها:** از **Value Equality** استفاده می‌کنند (دو آبجکت در صورتی برابرند که تمام فیلدهای متناظر آن‌ها مقدار یکسانی داشته باشند). *نکته: Recordها این کار را با بهینه‌سازی کامپایلر و بسیار سریع‌تر از Override کردن دستی Equals در Structها انجام می‌دهند.*

### 5. Immutability (تغییرناپذیری)
*   **Class و Struct:** به صورت پیش‌فرض **Mutable (تغییرپذیر)** هستند.
*   **Recordها:** به صورت پیش‌فرض **Immutable (تغییرناپذیر)** هستند. پراپرتی‌ها به صورت `init-only` ساخته می‌شوند.

### 6. Inheritance (وراثت)
*   **Class و Record Class:** از وراثت پشتیبانی می‌کنند.
*   **Struct و Record Struct:** **هرگز** از وراثت پشتیبانی نمی‌کنند (فقط می‌توانند Interface پیاده‌سازی کنند).

### 7. Performance و Boxing
*   **Boxing:** وقتی یک Value Type به یک Reference Type (مثل `object` یا یک `interface`) تبدیل می‌شود، دچار Boxing شده و در Heap کپی می‌شود که هزینه عملکردی دارد.
*   **عملکرد Recordها:** کامپایلر #C متدهای `Equals`, `GetHashCode` و `==` را برای Recordها به شدت بهینه‌سازی می‌کند. استفاده از `readonly record struct` بالاترین عملکرد را در بین انواع داده‌ای دارد.

---

## بخش دوم: کالبدشکافی چهار نوع داده

### 1. `class` (کلاس سنتی)
*   **نوع:** Reference Type
*   **ویژگی بارز:** دارای **Identity (هویت)** است. دو کلاس با مقادیر یکسان، دو موجودیت متفاوت هستند.
*   **کاربرد:** مدل‌سازی Entityها، سرویس‌ها، و اشیایی که هویت و چرخه حیات (Lifecycle) دارند و نیاز به تغییر وضعیت (Mutation) در طول زمان دارند.

### 2. `struct` (استراکت سنتی)
*   **نوع:** Value Type
*   **ویژگی بارز:** ساختارهای داده‌ای کوچک و سبک. به صورت پیش‌فرض Mutable هستند که می‌تواند خطرناک باشد (اگر `readonly` نباشند، کامپایلر برای جلوگیری از تغییر تصادفی، Defensive Copying انجام می‌دهد که پرفورمنس را کاهش می‌دهد).
*   **کاربرد:** ساختارهای ریاضی (مثل Vector)، داده‌های بسیار کوچک و کوتاه‌عمر که نیاز به وراثت ندارند.

### 3. `record class` (یا فقط `record`)
*   **نوع:** Reference Type
*   **ویژگی بارز:** ترکیب سبک حافظه Reference Typeها با سمانتیک Value Equality و Immutability. پشتیبانی از `with` expression برای ایجاد کپی‌های تغییر یافته.
*   **کاربرد:** DTOها، Domain Models تغییرناپذیر، و پیام‌های Event.

### 4. `record struct`
*   **نوع:** Value Type
*   **ویژگی بارز:** ترکیب سبک حافظه Value Typeها با سمانتیک Value Equality. (توجه: برخلاف record class، این نوع به صورت پیش‌فرض **Mutable** است، مگر اینکه کلمه `readonly` را قبل از آن اضافه کنید).
*   **کاربرد:** داده‌های کوچک، تغییرناپذیر، با کارایی بسیار بالا (High-Performance) که نیاز به Value Equality دارند.

---

## بخش سوم: سناریوهای واقعی (Real-World Scenarios)

در این بخش، ۶ سناریوی معروف را بررسی کرده و بهترین انتخاب را برای آن‌ها مشخص می‌کنیم:

### 1. Point / Coordinate (نقطه / مختصات)
*   **تحلیل:** داده‌های ریاضی، بسیار کوچک، کوتاه‌عمر، بدون هویت، و نیازمند عملیات ریاضی.
*   **انتخاب صحیح:** `readonly record struct`
*   **دلیل:** تغییرناپذیری (readonly) جلوی Defensive Copying را می‌گیرد و پرفورمنس را به حداکثر می‌رساند. Value Equality برای مقایسه دو نقطه ضروری است.
```csharp
public readonly record struct Point(double X, double Y);
// یا
public readonly record struct Coordinate(double Latitude, double Longitude);
```

### 2. Money (پول / ارز)
*   **تحلیل:** باید تغییرناپذیر باشد (یک مبلغ ۱۰ دلاری نباید به ۲۰ دلار تغییر کند). نیاز به Value Equality دارد (۱۰ دلار برابر با ۱۰ دلار دیگر است).
*   **انتخاب صحیح:** `readonly record struct`
*   **دلیل:** جلوگیری از Boxing در محاسبات مالی حجیم، سمانتیک Value Equality، و Immutability.
```csharp
public readonly record struct Money(decimal Amount, string Currency);
```

### 3. Customer (مشتری)
*   **تحلیل:** یک **Entity** است. دارای هویت (CustomerId) است. ممکن است اطلاعات آن در طول زمان تغییر کند (آپدیت آدرس، تغییر نام). ممکن است نیاز به وراثت داشته باشد (مثل `CorporateCustomer` که از `Customer` ارث‌بری می‌کند).
*   **انتخاب صحیح:** `class`
*   **دلیل:** نیاز به Identity، Mutation و احتمالاً Inheritance.
```csharp
public class Customer
{
    public int Id { get; set; } // Identity
    public string Name { get; set; } // Mutable
    public string Email { get; set; }
}
```

### 4. DTO (Data Transfer Object)
*   **تحلیل:** صرفاً برای انتقال داده بین لایه‌ها (مثلاً از API به فرانت‌اند). معمولاً تغییرناپذیر است و در Unit Testها نیاز به Value Equality داریم تا بتوانیم `Assert.Equal(expectedDto, actualDto)` را صدا بزنیم.
*   **انتخاب صحیح:** `record class` (یا `record`)
*   **دلیل:** Immutability پیش‌فرض، Value Equality برای تست‌ها، و پشتیبانی از `with` برای ایجاد DTOهای جدید بر اساس DTOهای قبلی.
```csharp
public record CustomerDto(int Id, string Name, string Email);
```

### 5. Event (رویداد دامنه / Domain Event)
*   **تحلیل:** در الگوهایی مثل CQRS، یک Event نشان‌دهنده چیزی است که در گذشته اتفاق افتاده (مثل `OrderPlacedEvent`). رویدادها **همیشه تغییرناپذیر** هستند.
*   **انتخاب صحیح:** `record class`
*   **دلیل:** Immutability تضمین می‌کند که رویداد در طول پردازش تغییر نمی‌کند. Reference Type بودن به آن اجازه می‌دهد در صف‌های پیام (Message Queues) و ساختارهای پیچیده‌تر به راحتی مدیریت شود.
```csharp
public record OrderPlacedEvent(Guid OrderId, DateTime OccurredOn, decimal TotalAmount);
```

---

## بخش چهارم: جدول مقایسه جامع

| ویژگی | `class` | `struct` | `record class` (record) | `record struct` |
| :--- | :--- | :--- | :--- | :--- |
| **نوع داده (Type)** | Reference Type | Value Type | Reference Type | Value Type |
| **محل ذخیره‌سازی** | Heap | Stack / Inline | Heap | Stack / Inline |
| **تغییرپذیری پیش‌فرض** | Mutable | Mutable | **Immutable** (init-only) | Mutable (مگر `readonly` شود) |
| **نوع برابری پیش‌فرض** | Reference Equality | Value Equality | **Value Equality** (بهینه‌شده) | **Value Equality** (بهینه‌شده) |
| **وراثت (Inheritance)** | پشتیبانی می‌کند | پشتیبانی **نمی‌کند** | پشتیبانی می‌کند | پشتیبانی **نمی‌کند** |
| **سینتکس Primary Ctor** | ندارد (مگر C# 12) | ندارد (مگر C# 12) | **دارد** | **دارد** |
| **عبارت `with`** | ندارد | ندارد | **دارد** | **دارد** |
| **فشار روی GC** | دارد (Allocation) | ندارد | دارد (Allocation) | ندارد |
| **خطر Boxing** | ندارد | **دارد** | ندارد | **دارد** |

---

## بخش پنجم: راهنمای تصمیم‌گیری (Decision Guide)

برای انتخاب صحیح، این الگوریتم ذهنی را طی کنید:

**سوال ۱: آیا این داده نیاز به هویت (Identity) دارد یا باید در طول زمان تغییر کند (Mutate شود)؟**
*   ✅ **بله:** از **`class`** استفاده کنید. (مثال: Entityها مثل Customer, Order, User).
*   ❌ **خیر:** به سوال ۲ بروید.

**سوال ۲: آیا این داده نیاز به وراثت (Inheritance) دارد؟**
*   ✅ **بله:** از **`record class`** استفاده کنید. (مثال: یک ساختار درختی از رویدادها).
*   ❌ **خیر:** به سوال ۳ بروید.

**سوال ۳: آیا این داده یک ساختار بزرگ است یا در Domain Model پیچیده استفاده می‌شود؟**
*   ✅ **بله (حجم داده بالا / پیچیدگی بالا):** از **`record class`** استفاده کنید. (مثال: DTOهای بزرگ، Domain Events، تنظیمات پیچیده).
*   ❌ **خیر (داده کوچک و سبک است):** به سوال ۴ بروید.

**سوال ۴: آیا این داده یک ساختار کوچک، ریاضی یا با کارایی بسیار بالا (High-Performance) است؟**
*   ✅ **بله:** از **`readonly record struct`** استفاده کنید. (مثال: Point, Money, Coordinate, Vector, Date-only wrappers).
*   ❌ **خیر (نیاز به تغییرپذیری داخلی دارید و پرفورمنس اولویت اول نیست):** از **`struct`** معمولی استفاده کنید (هرچند در #C مدرن، استفاده از struct معمولی به نفع record structها به شدت کاهش یافته است).

### 💡 یک قانون طلایی (Rule of Thumb) در #C مدرن:
1. برای **Entityها** (دارای هویت و تغییرپذیر) 👈 **`class`**
2. برای **DTOها، Events و Value Objectsهای بزرگ** 👈 **`record class`**
3. برای **Value Objectsهای کوچک، ریاضی و High-Performance** 👈 **`readonly record struct`**

*توجه: در #C 10 به بعد، تقریباً هیچ دلیل موجهی برای استفاده از `struct` یا `record struct`ِ **تغییرپذیر (Mutable)** وجود ندارد. همیشه آن‌ها را `readonly` объявите کنید تا از مشکلات پرفورمنسی Defensive Copying و باگ‌های تغییر تصادفی داده‌ها جلوگیری کنید.*