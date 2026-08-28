# Record یا Class؟ راهنمای جامع تصمیم‌گیری برای برنامه‌نویسان C#

از زمان معرفی `record` در C# 9.0، یکی از چالش‌های رایج برنامه‌نویسان این است که در هر سناریو از کدام نوع داده‌ای استفاده کنند. انتخاب اشتباه می‌تواند منجر به کدهای پرگو (Verbose)، باگ‌های مربوط به برابری (Equality Bugs) و مشکلات عملکردی شود.

این مقاله یک راهنمای تصمیم‌گیری کامل و مبتنی بر مفاهیم مهندسی نرم‌افزار (به‌ویژه Domain-Driven Design) برای انتخاب صحیح بین **Class** و **Record Class** است.

---

## بخش اول: درک مفاهیم پایه (Core Concepts)

برای انتخاب صحیح، ابتدا باید ماهیت داده‌ها و رفتارهای خود را درک کنید:

### ۱. Identity (هویت) در مقابل Value (مقدار)
*   **Identity (هویت):** آیا شیء شما در دنیای واقعی دارای یک هویت منحصر‌به‌فرد است؟ (مثلاً دو کاربر با نام و ایمیل یکسان، همچنان دو شخصیت متفاوت هستند). اگر بله، شما با **Entity** سر و کار دارید.
*   **Value (مقدار):** آیا شیء شما صرفاً با ویژگی‌هایش تعریف می‌شود؟ (مثلاً دو اسکناس ۱۰ دلاری، دقیقاً یک ارزش دارند و تفاوتی بین آن‌ها نیست). اگر بله، شما با **Value Object** سر و کار دارید.

### ۲. Mutable State (حالت تغییرپذیر) در مقابل Immutable State (حالت تغییرناپذیر)
*   **Mutable:** داده‌های شیء در طول چرخه حیات تغییر می‌کنند (مثلاً موجودی یک محصول کم و زیاد می‌شود).
*   **Immutable:** داده‌های شیء در لحظه ساخت تنظیم شده و هرگز تغییر نمی‌کنند. برای تغییر، باید یک شیء جدید ساخت.

### ۳. Lifecycle (چرخه حیات)
*   آیا شیء شما توسط یک ORM (مثل Entity Framework) ردیابی می‌شود و در دیتابیس ذخیره می‌گردد؟ اگر بله، معمولاً نیاز به هویت و تغییرپذیری دارد.
*   آیا شیء شما فقط برای انتقال داده از یک لایه به لایه دیگر (یا از طریق شبکه) استفاده می‌شود و سپس دور ریخته می‌شود؟

### ۴. Equality (برابری)
*   **Reference Equality (پیش‌فرض Class):** دو متغیر تنها زمانی برابرند که به یک آدرس حافظه اشاره کنند.
*   **Value Equality (پیش‌فرض Record):** دو متغیر زمانی برابرند که تمام فیلدها و پراپرتی‌های آن‌ها مقدار یکسانی داشته باشند.

### ۵. Inheritance (وراثت)
*   کلاس‌ها از وراثت عمیق و چندلایه پشتیبانی می‌کنند.
*   رکوردها هم از وراثت پشتیبانی می‌کنند، اما پیاده‌سازی متدهای برابری (Equality) در وراثت رکوردها کمی پیچیده‌تر است و نیازمند دقت بیشتری است.

### ۶. Performance (عملکرد)
*   **Class و Record Class:** هر دو Reference Type هستند و روی Heap حافظهallocat می‌شوند. اما `Record` به دلیل پیاده‌سازی متدهای `Equals`، `GetHashCode` و اپراتور `with` (برای کپی عمیق)، سربار (Overhead) حافظه و پردازشی بیشتری نسبت به کلاس معمولی دارد.
*   *(نکته: اگر پرفورمنس حیاتی است و داده‌ها کوچک هستند، از `record struct` استفاده کنید که Value Type است).*

### ۷. Data-Centric (داده‌محور) در مقابل Behavior-Centric (رفتارمحور)
*   **Data-Centric:** شیء صرفاً یک ظرف برای نگهداری داده‌هاست (مثل DTOها).
*   **Behavior-Centric:** شیء دارای متدهای پیچیده، قوانین تجاری (Business Rules) و تغییردهنده State است.

---

## بخش دوم: مقایسه مستقیم Class و Record Class

| ویژگی | Class (کلاس معمولی) | Record Class (رکورد کلاس) |
| :--- | :--- | :--- |
| **نوع داده** | Reference Type | Reference Type |
| **برابری پیش‌فرض** | Reference Equality (مقایسه آدرس حافظه) | Value Equality (مقایسه مقدار داده‌ها) |
| **تغییرپذیری (Mutability)** | پیش‌فرض Mutable (تغییرپذیر) | طراحی شده برای Immutable (با `init`) |
| **تمرکز اصلی** | Behavior-Centric (رفتارمحور) | Data-Centric (داده‌محور) |
| **کپی کردن شیء** | نیاز به پیاده‌سازی دستی (Deep Copy) | پشتیبانی داخلی با عبارت `with` |
| **سربار عملکرد** | پایین | متوسط (به دلیل متدهای برابری و `with`) |
| **مناسب برای** | Entityها، سرویس‌ها، کلاس‌های دارای منطق | DTOها، Value Objectها، Domain Events |

---

## بخش سوم: تحلیل سناریوها (کدام را کجا استفاده کنیم؟)

در این بخش، ۱۱ سناریوی رایج را بررسی کرده و بهترین انتخاب را مشخص می‌کنیم:

### ۱. User (کاربر) -> **Class**
*   **دلیل:** کاربر یک **Entity** است. دارای هویت منحصر‌به‌فرد (UserId) است، چرخه حیات مشخصی در دیتابیس دارد و State آن (مثل تاریخ آخرین ورود، وضعیت اکانت) تغییر می‌کند (Mutable).

### ۲. Customer (مشتری) -> **Class**
*   **دلیل:** مشابه User، مشتری در دامنه کسب‌وکار دارای هویت است. دو مشتری با مشخصات یکسان، دو شخصیت حقوقی/حقیقی متفاوت هستند.

### ۳. Order (سفارش) -> **Class**
*   **دلیل:** سفارش یک **Entity** است. دارای OrderId است و State آن در طول زمان تغییر می‌کود (از Pending به Shipped و سپس Delivered).

### ۴. Product (محصول) -> **Class**
*   **دلیل:** محصول دارای هویت (ProductId) است و ویژگی‌های Mutable دارد (مثل موجودی انبار که مدام کم و زیاد می‌شود).

### ۵. Money (پول / مبلغ) -> **Record** (یا Record Struct)
*   **دلیل:** پول یک **Value Object** کلاسیک است. ۱۰ دلار با ۱۰ دلار دیگر هیچ تفاوتی ندارد (Value Equality). کاملاً Immutable است و نیازی به هویت مستقل ندارد.

### ۶. Address (آدرس) -> **Record** (یا Record Struct)
*   **دلیل:** آدرس یک **Value Object** است. اگر دو کاربر در یک آدرس زندگی کنند، خودِ آدرس (شهر، خیابان، پلاک) کاملاً یکسان است. Immutable است.

### ۷. DTO (Data Transfer Object) -> **Record**
*   **دلیل:** DTOها کاملاً **Data-Centric** هستند. معمولاً Immutable طراحی می‌شوند تا در حین انتقال بین لایه‌ها تغییر نکنند. Value Equality برای تست‌نویسی و مقایسه آن‌ها بسیار مفید است.

### ۸. API Response -> **Record**
*   **دلیل:** دقیقاً هم‌کاربرد DTO است. داده‌ها را از API دریافت کرده و به لایه‌های بالاتر می‌برد. نیازی به تغییر State یا هویت ندارد.

### ۹. Domain Event (رویداد دامنه) -> **Record**
*   **دلیل:** رویدادها (مثل `OrderPlacedEvent`) نشان‌دهنده یک اتفاق در گذشته هستند. گذشته تغییر نمی‌کند (Immutable). همچنین برای بررسی اینکه آیا یک رویداد خاص در تست‌ها رخ داده یا نه، Value Equality بسیار کارآمد است.

### ۱۰. Entity (موجودیت دامنه) -> **Class**
*   **دلیل:** بر اساس تعریف DDD، موجودیت‌ها (Entities) با هویت (Identity) تعریف می‌شوند، نه با ویژگی‌هایشان. بنابراین `Class` انتخاب قطعی است.

### ۱۱. Value Object (شیء مقداری) -> **Record**
*   **دلیل:** بر اساس تعریف DDD، اشیاء مقداری (Value Objects) صرفاً با ویژگی‌هایشان تعریف می‌شوند، هویت مستقل ندارند و Immutable هستند. `Record` دقیقاً برای همین هدف ساخته شده است.

---

## بخش چهارم: درخت تصمیم‌گیری متنی (Decision Tree)

برای انتخاب سریع، این الگوریتم متنی را طی کنید:

```text
[شروع]
  │
  ├─ آیا شیء شما در دنیای واقعی دارای هویت منحصر‌به‌فرد (ID) است؟
  │   (مثل User, Order, Product)
  │   ├─ [بله] ──> استفاده از CLASS
  │   │
  │   └─ [خیر] ──> ادامه به سوال بعد...
  │
  ├─ آیا شیء شما دارای منطق تجاری پیچیده است و State آن در طول زمان تغییر می‌کند؟
  │   (مثل کلاس‌های سرویس، State Machineها)
  │   ├─ [بله] ──> استفاده از CLASS
  │   │
  │   └─ [خیر] ──> ادامه به سوال بعد...
  │
  ├─ آیا شیء شما صرفاً یک ظرف برای انتقال داده است؟
  │   (مثل DTO, API Request/Response, Config)
  │   ├─ [بله] ──> استفاده از RECORD CLASS
  │   │           (اگر داده‌ها بسیار کوچک و پرفورمنس حیاتی است -> RECORD STRUCT)
  │   │
  │   └─ [خیر] ──> ادامه به سوال بعد...
  │
  ├─ آیا شیء شما یک Value Object یا Domain Event است؟
  │   (مثل Money, Address, DateRange, OrderCreatedEvent)
  │   ├─ [بله] ──> استفاده از RECORD CLASS
  │   │           (اگر داده‌ها بسیار کوچک و پرفورمنس حیاتی است -> RECORD STRUCT)
  │   │
  │   └─ [خیر] ──> استفاده از CLASS (به عنوان حالت پیش‌فرض و امن)
```

---

## نکات طلایی و بهترین شیوه‌ها (Best Practices)

1.  **رکوردها تغییرناپذیر (Immutable) نیستند، بلکه برای آن طراحی شده‌اند:** شما می‌توانید در `record` از `set` به جای `init` استفاده کنید و آن را Mutable کنید، اما **این کار خلاف فلسفه Record است**. همیشه از `init` یا `readonly` برای پراپرتی‌های رکورد استفاده کنید.
2.  **سربار وراثت در رکوردها:** اگر قصد دارید از `record` ارث‌بری عمیق (بیش از ۲ سطح) داشته باشید، مراقب باشید. متد `PrintMembers` و مکانیزم `Equality` در رکوردهای ارث‌بری شده ممکن است باعث پیچیدگی و افت عملکرد شود. در این موارد `class` ارجحیت دارد.
3.  **استفاده از `with` Expression:** یکی از بزرگترین مزیت‌های `Record`، عبارت `with` است که کپی عمیق (Deep Clone) را با یک خط کد و به صورت Immutable انجام می‌دهد:
    ```csharp
    var originalAddress = new Address("Tehran", "ValiAsr", "123");
    var newAddress = originalAddress with { City = "Isfahan" }; 
    // یک رکورد جدید ساخته می‌شود و رکورد قبلی دست‌نخورده باقی می‌ماند.
    ```

---

## منابع رسمی Microsoft Learn

برای مطالعه بیشتر و اطمینان از جزئیات فنی، منابع زیر از مستندات رسمی مایکروسافت پیشنهاد می‌شوند:

1.  **Record types (C# Reference):**
    [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
    *(مرجع کامل سینتکس، رفتار برابری، و عبارت with در رکوردها)*

2.  **How to use records (C# Guide):**
    [https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)
    *(راهنمای کاربردی برای پیاده‌سازی رکوردها، تفاوت record class و record struct، و وراثت)*

3.  **Choose between class and struct:**
    [https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct)
    *(اصول طراحی مایکروسافت برای انتخاب بین Reference Type و Value Type)*

4.  **Value Objects in Domain-Driven Design (Conceptual context):**
    [https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects)
    *(چرا مایکروسافت استفاده از Recordها را برای پیاده‌سازی Value Objectها در DDD به شدت توصیه می‌کند)*