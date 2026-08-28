# جمع‌بندی Records در C# و راهنمای انتخاب

به فصل پایانی و جامع مبحث **Records** در Repository آموزشی C# خوش آمدید. در این فصل، تمام مفاهیم پراکنده را به یک تصویر واحد و منسجم تبدیل می‌کنیم تا بتوانید با دیدی کاملاً باز، بهترین ساختار داده‌ای را برای نیازهای خود انتخاب کنید.

---

## ۱. مرور جامع مفاهیم (Core Concepts)

### Record چیست؟
در C# 9 معرفی شد. `Record` یک کلمه کلیدی و در واقع **Syntactic Sugar** برای ایجاد کلاس‌ها یا ساختارهایی است که تمرکز اصلی آن‌ها روی **ذخیره داده** است تا رفتار. کامپایلر به صورت خودکار کدهای boilerplate (مثل Equality، ToString و Copy) را تولید می‌کند.

### انواع Record
1. **Record Class:** (پیش‌فرض در C# 9) یک Reference Type است. به صورت پیش‌فرض Immutable (غیرقابل تغییر) طراحی می‌شود و بر اساس **Value Equality** مقایسه می‌گردد.
2. **Record Struct:** (معرفی شده در C# 10) یک Value Type است. برخلاف Record Class، به صورت پیش‌فرض **Mutable** (قابل تغییر) است مگر اینکه `readonly` اضافه شود.
3. **Readonly Record Struct:** تمام فیلدها و پراپرتی‌های آن `readonly` هستند. یک Value Type کاملاً Immutable و بسیار بهینه برای داده‌های کوچک.

### ویژگی‌های کلیدی
- **Value Equality:** دو Record زمانی برابرند که تمام مقادیر پراپرتی‌هایشان یکسان باشد (برخلاف کلاس‌های معمولی که Reference Equality دارند).
- **Immutability:** پس از ایجاد، وضعیت (State) آن‌ها تغییر نمی‌کند. این ویژگی آن‌ها را برای Thread-Safety و استفاده در Concurrent Programming ایده‌آل می‌کند.
- **Non-Destructive Mutation & `with`:** عبارت `with` به شما اجازه می‌دهد یک کپی از Record ایجاد کنید و فقط پراپرتی‌های خاصی را تغییر دهید.
  ```csharp
  var newPerson = person with { LastName = "Smith" };
  ```
- **Copy Constructor & Clone:** کامپایلر یک Copy Constructor محافظت‌شده (protected) تولید می‌کند. متد `Clone` (که عملاً یک Shallow Copy است) در پس‌زمینه برای پیاده‌سازی `with` استفاده می‌شود.
- **ToString:** خروجی بسیار خوانا و دیباگ‌پسند تولید می‌کند: `Person { FirstName = Ali, LastName = Rahimi }`.
- **Deconstruction:** امکان تجزیه پراپرتی‌ها به متغیرهای محلی:
  ```csharp
  var (first, last) = person;
  ```
- **EqualityContract:** یک پراپرتی داخلی است که تضمین می‌کند دو Record دقیقاً از یک Runtime Type باشند (جلوگیری از برابری اشتباه در زمان Inheritance).

### ارث‌بری و رابط‌ها (Inheritance & Interface)
- **Record Class:** از Inheritance پشتیبانی می‌کند (اما نمی‌تواند یک کلاس معمولی را ارث ببرد، فقط Record دیگر).
- **Record Struct:** به دلیل ماهیت Value Type، از Inheritance پشتیبانی **نمی‌کند**.
- هر دو نوع می‌توانند **Interface** را پیاده‌سازی کنند.

### کاربردهای عملی (Use Cases)
- **DTO (Data Transfer Object):** بهترین گزینه برای انتقال داده بین لایه‌ها.
- **API Contract:** مدل‌های Request و Response در Web APIها.
- **Domain Event:** در DDD، رویدادها باید Immutable باشند تا تاریخچه آن‌ها دستکاری نشود.
- **Value Object:** در DDD، اشیایی که هویت ندارند و فقط مقدارشان مهم است (مثل `Money` یا `Address`).

### عملکرد (Performance) و اشتباهات رایج
- **Performance:** استفاده از `Record Class` سربار GC (Garbage Collection) دارد. برای داده‌های بسیار کوچک و پرتکرار، `Readonly Record Struct` به دلیل قرارگیری در Stack، عملکردی خیره‌کننده دارد.
- **اشتباهات رایج:**
  1. استفاده از `Record Struct` بدون `readonly` (باعث Mutable شدن و از بین رفتن مزیت اصلی می‌شود).
  2. استفاده از Recordهای بزرگ (Large Structs) که باعث Boxing و کپی‌های پرهزینه در Stack می‌شود.
  3. تلاش برای پیاده‌سازی Business Logic و رفتارهای پیچیده داخل Recordها (Record فقط برای داده است).

---

## ۲. جدول مقایسه جامع

| ویژگی | Class (معمولی) | Struct (معمولی) | Record Class | Record Struct |
| :--- | :--- | :--- | :--- | :--- |
| **نوع زیرین** | Reference Type | Value Type | Reference Type | Value Type |
| **Reference/Value** | Reference | Value | Reference | Value |
| **Equality** | Identity (Reference) | Identity (پیش‌فرض) | **Value** | **Value** |
| **Immutability** | Mutable | Mutable | Immutable (طراحی) | Mutable (مگر `readonly`) |
| **عبارت `with`** | ❌ ندارد | ❌ ندارد | ✅ دارد | ✅ دارد |
| **Inheritance** | ✅ دارد | ❌ ندارد | ✅ دارد | ❌ ندارد |
| **Copy Semantics** | Reference Copy | Value Copy (Shallow) | Deep Copy (توسط Clone) | Value Copy (Shallow) |
| **کاربرد مناسب** | Entityها، سرویس‌ها | داده‌های کوچک هندسی/ریاضی | DTO، Domain Event، Value Object | Value Objectهای کوچک، مختصات |
| **محدودیت‌ها** | سربار GC، عدم برابری مقداری | عدم ارث‌بری، Boxing | سربار GC، عدم ارث‌بری از کلاس معمولی | عدم ارث‌بری، جریمه کپی در ابعاد بزرگ |

---

## ۳. درخت تصمیم‌گیری (Decision Tree)

برای انتخاب بهترین نوع، این مسیر منطقی را طی کنید:

```text
شروع: داده‌ای که می‌خواهید مدل‌سازی کنید چیست؟
│
├── آیا "Identity" (هویت) برای داده مهم است؟ (مثل User, Product)
│   └── بله → از Class معمولی استفاده کنید (Entity).
│
├── آیا "Value" (مقدار) برای داده مهم است؟ (مثل Address, DateRange)
│   ├── بله → آیا نیاز به Inheritance (ارث‌بری) دارید؟
│   │   ├── بله → از Record Class استفاده کنید.
│   │   └── خیر → آیا حجم داده کوچک است؟ (مثلاً زیر ۱۶ بایت یا چند فیلد ساده)
│   │       ├── بله → از Readonly Record Struct استفاده کنید (بهترین Performance).
│   │       └── خیر → از Record Class استفاده کنید (جلوگیری از سربار کپی Struct).
│   └── خیر → (معمولاً داده‌ها یا هویت دارند یا مقدار، این شاخه کمتر پیش می‌آید).
│
├── آیا نیاز به تغییر وضعیت (Mutation) پس از ساخت دارید؟
│   ├── بله → از Class یا Struct معمولی استفاده کنید.
│   └── خیر (Immutable می‌خواهیم) → همان مسیر Value Objectها در بالا.
│
└── آیا این داده برای انتقال بین لایه‌ها یا API است؟
    └── بله → از Record Class استفاده کنید (DTO / API Contract).
```

---

## خلاصه نهایی
`Record`ها انقلابی در نحوه مدل‌سازی داده‌ها در C# بودند. آن‌ها با حذف کدهای تکراری، تمرکز توسعه‌دهنده را از "چگونه بنویسم" به "چه چیزی بنویسم" تغییر دادند. استفاده از `Record Class` برای داده‌های پیچیده و انتقال‌یافته، و `Readonly Record Struct` برای داده‌های کوچک و محاسباتی، بهترین الگوی معماری مدرن در C# است.

---

## نکات طلایی (Pro Tips)
1. **همیشه `readonly record struct` بنویسید:** مگر اینکه دلیل بسیار محکمی برای Mutable بودن `record struct` داشته باشید، `readonly` را اضافه کنید تا از تغییرات تصادفی و سربار کپی جلوگیری کنید.
2. **مراقب `with` در Recordهای تو در تو باشید:** عبارت `with` به صورت **Shallow Copy** عمل می‌کند. اگر یک Record شامل یک Reference Type دیگر باشد، تغییر آن در کپی جدید، روی اصلی هم اثر می‌گذارد.
3. **متدهای `init` را فراموش نکنید:** برای پراپرتی‌هایی که نیاز به اعتبارسنجی (Validation) در زمان ساخت دارند، به جای `init` ساده، از `init` همراه با Logic استفاده کنید.
4. **رکوردها را Seal کنید:** اگر قرار نیست از یک `Record Class` ارث‌بری کنید، حتماً آن را `sealed` کنید تا کامپایلر بتواند `EqualityContract` را بهینه‌تر بررسی کند.

---

## سؤالات مصاحبه (Interview Questions)

1. **تفاوت اصلی `record class` و `record struct` در چیست و کدام یک به صورت پیش‌فرض Immutable است؟**
   *پاسخ:* `record class` یک Reference Type و به صورت پیش‌فرض Immutable است. `record struct` یک Value Type و به صورت پیش‌فرض Mutable است (مگر اینکه `readonly` اضافه شود).
2. **عبارت `with` در پس‌زمینه چگونه کار می‌کند؟**
   *پاسخ:* کامپایلر از یک Copy Constructor محافظت‌شده و یک متد `Clone` (Shallow Copy) استفاده می‌کند تا یک کپی از آبجکت بسازد و سپس پراپرتی‌های مشخص شده را در کپی تغییر دهد.
3. **پراپرتی `EqualityContract` چه نقشی دارد؟**
   *پاسخ:* تضمین می‌کند که دو رکورد در زمان مقایسه، دقیقاً از یک Runtime Type باشند. این از باگ‌هایی که در زمان Inheritance ممکن است رخ دهد (مقایسه یک Derived با Base) جلوگیری می‌کند.
4. **چرا نباید از `record struct` برای داده‌های بزرگ استفاده کرد؟**
   *پاسخ:* چون Value Type است، در هر بار پاس دادن به متدها یا استفاده از `with`، کل داده در Stack کپی می‌شود که برای داده‌های بزرگ سربار عملکردی شدیدی دارد.

---

## تمرین‌های عملی (Practical Exercises)

**تمرین ۱: تبدیل کلاس به Record**
یک کلاس معمولی `Customer` با ۵ پراپرتی بنویسید. سپس آن را به `record` تبدیل کنید. متد `Equals` و `ToString` را به صورت دستی پیاده‌سازی نکنید و خروجی دیباگ را مشاهده کنید.

**تمرین ۲: پیاده‌سازی Value Object**
یک `Readonly Record Struct` به نام `Money` با دو پراپرتی `Amount` و `Currency` بسازید. اپراتورهای `+` و `-` را برای آن Overload کنید.

**تمرین ۳: چالش `with`**
یک `Record Class` به نام `Address` بسازید که شامل یک `Record Class` دیگر به نام `City` باشد. با استفاده از `with`، شهر را تغییر دهید. بررسی کنید که آیا آبجکت `City` در رکورد اصلی هم تغییر کرده است یا خیر (درک Shallow Copy).

---

## پروژه پیشنهادی (Suggested Project)

**پروژه: سیستم سبد خرید فروشگاه (DDD Style)**
یک Console Application بسازید که مفاهیم DDD را با استفاده از Records پیاده‌سازی کند:
1. **Value Objects:** `ProductPrice` و `Address` را به عنوان `readonly record struct` پیاده‌سازی کنید.
2. **Entities:** `Product` و `Customer` را به عنوان `class` معمولی (با Identity) بسازید.
3. **Domain Events:** رویدادهایی مثل `ProductAddedToCartEvent` را به عنوان `sealed record class` بسازید.
4. **API Contracts:** مدل‌های `CreateOrderRequest` و `OrderResponse` را به عنوان `record class` طراحی کنید.

---

## منابع معتبر (References)

برای مطالعه عمیق‌تر و ارجاع به استانداردها، لینک‌های زیر پیشنهاد می‌شوند:

1. **Microsoft Learn - Records (C# Reference)**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
2. **Microsoft Learn - Record Structs**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record-struct](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record-struct)
3. **C# Language Specification - Records**
   [https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#158-records)
4. **.NET Architecture Guides - Domain-Driven Design (Value Objects)**
   [https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects)
5. **Book: Domain-Driven Design Reference (Eric Evans)**
   [https://www.domainlanguage.com/ddd/reference/](https://www.domainlanguage.com/ddd/reference/) *(مرجع اصلی مفاهیم Value Object و Domain Event)*

---
*پایان فصل Records. امیدواریم این مسیر آموزشی به شما در نوشتن کدهای تمیزتر، ایمن‌تر و مدرن‌تر کمک کرده باشد.* 🚀