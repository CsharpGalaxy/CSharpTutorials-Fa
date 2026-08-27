در ادامه، یک مقاله جامع، حرفه‌ای و ساختاریافته برای قرار گرفتن در Repository آموزشی شما تهیه شده است. این مقاله با لحنی مهندسی و بر اساس استانداردهای روز توسعه نرم‌افزار با #C نوشته شده است.

---

# 📘 بهترین روش‌ها (Best Practices) در استفاده از Tuple در #C

از زمان معرفی `ValueTuple` در #C 7.0، Tuples به یکی از ابزارهای محبوب برای گروه‌بندی موقت داده‌ها تبدیل شده‌اند. با این حال، استفاده نادرست از آن‌ها می‌تواند منجر به کدهایی غیرقابل نگهداری، مشکلات در Serialization و نقض اصول طراحی شی‌گرا شود. 

در این مقاله، بررسی می‌کنیم که **کجا، چگونه و چه زمانی** باید از Tuple استفاده کنیم و از چه کارهایی باید پرهیز کنیم.

---

## ۱. چه زمانی از Tuple استفاده کنیم؟ (و چه زمانی نکنیم؟)

### ✅ چه زمانی استفاده کنیم؟
* **بازگرداندن چندین مقدار از یک متد:** زمانی که نمی‌خواهید فقط به خاطر بازگرداندن دو یا سه مقدار، یک کلاس جدید بسازید.
* **داده‌های موقت و گذرا (Transient Data):** زمانی که داده‌ها فقط در محدوده (Scope) یک متد یا یک کوئری LINQ زندگی می‌کنند.
* **کلیدهای ترکیبی (Composite Keys):** برای استفاده به عنوان کلید در `Dictionary`.
* **توسعه سریع (Prototyping):** در مراحل اولیه توسعه که هنوز ساختار نهایی داده‌ها مشخص نیست.

### ❌ چه زمانی استفاده نکنیم؟
* **مدل‌های دامنه (Domain Models):** اشیاء دامنه باید هویت، رفتار (متد) و مفهوم مشخص داشته باشند.
* **توافق‌نامه‌های عمومی (Public Contracts):** برای APIها، DTOها و تنظیمات.
* **داده‌های با طول عمر بالا:** Tuples برای ذخیره‌سازی در دیتابیس، کش (Cache) یا Session طراحی نشده‌اند.

---

## ۲. اندازه Tuple: کوچک یا بزرگ؟
**قانون سرانگشتی:** Tuple باید **کوچک** باشد (ترجیحاً ۲ تا ۴ عضو).
اگر Tuple شما به `Item5`، `Item6` و بالاتر نیاز پیدا کرد، این یک **بوی بد کد (Code Smell)** است. در این حالت، داده‌های شما در حال تبدیل شدن به یک Entitiy یا DTO هستند و باید بلافاصله آن را به یک `class`، `struct` یا `record` تبدیل کنید.

---

## ۳. نام‌گذاری اعضا (Naming Members)
**همیشه** برای اعضای Tuple نام معنادار انتخاب کنید.
```csharp
// ❌ بد: خوانایی صفر
(int, string) GetUser() => (1, "Ali");

// ✅ خوب: خودتوضیح‌دهنده (Self-documenting)
(int UserId, string UserName) GetUser() => (1, "Ali");
```
**نکته مهم مهندسی:** نام‌گذاری اعضای Tuple در #C فقط یک **ویژگی کامپایلر (Compiler Feature)** است. در زمان اجرا (Runtime)، نام‌ها از بین می‌روند و اعضا به صورت `Item1`، `Item2` و... شناخته می‌شوند (مگر اینکه از Reflection و `TupleElementNamesAttribute` استفاده کنید).

---

## ۴. استفاده در Return Type
* **متدهای Private/Internal:** استفاده از Tuple به عنوان Return Type **بسیار عالی** است. این کار از ایجاد کلاس‌های بی‌معنی (Junk Classes) جلوگیری می‌کند.
* **متدهای Public:** **هرگز** از Tuple در Public API استفاده نکنید. تغییر ساختار Tuple در آینده (مثلاً اضافه کردن یک فیلد) باعث Breaking Change می‌شود و برای مصرف‌کننده API، درک `(int, bool, string)` بدون مستندات بسیار دشوار است.

---

## ۵. استفاده در LINQ
Tuple در LINQ برای `Select` و `Join` بسیار کاربردی است، به خصوص زمانی که می‌خواهید نتیجه کوئری را از متد خارج کنید (Anonymous Types فقط در همان Scope قابل استفاده‌اند).
```csharp
var userRoles = users
    .Join(roles, u => u.RoleId, r => r.Id, (u, r) => (u.Name, r.Title))
    .ToList();
```

---

## ۶. استفاده در Collection و Dictionary Key
* **Collection:** استفاده از `List<(int, string)>` برای لیست‌های موقت مجاز است، اما برای لیست‌هایی که قرار است به لایه‌های دیگر پاس داده شوند، ممنوع است.
* **Dictionary Key:** **نقطه قوت Tuple!** از آنجا که `ValueTuple` یک `struct` است و `IEquatable<T>` و `GetHashCode()` را به درستی پیاده‌سازی کرده است، بهترین گزینه برای **کلیدهای ترکیبی** در دیکشنری است.
```csharp
// ✅ عالی برای کلید ترکیبی
var matrix = new Dictionary<(int row, int col), string>();
matrix[(0, 1)] = "Value";
```

---

## ۷. مرزهای معماری: Public vs Private API
* **Private/Internal API:** استفاده آزادانه مجاز است.
* **Public API / Library:** **ممنوع.** کتابخانه‌ها باید قراردادهای پایداری داشته باشند. به جای Tuple از کلاس‌ها، ساختارها یا `record`ها استفاده کنید.

---

## ۸. مقایسه با Domain Model، DTO و Record
| مفهوم | آیا Tuple مناسب است؟ | جایگزین صحیح |
| :--- | :---: | :--- |
| **Domain Model** | ❌ خیر | `class` (با رفتار و هویت) |
| **DTO** | ❌ خیر | `record` یا `class` (با Validation) |
| **ViewModel** | ❌ خیر | `class` یا `record` |
| **Config / Options** | ❌ خیر | `class` یا `record` |

**چرا Record بهتر است؟** از #C 9.0، `record`ها جایگزین بی‌نقص Tuple برای DTOها و داده‌های_immutable_ شده‌اند. Recordها نام‌گذاری مشخص، Value Equality و پشتیبانی کامل از Serialization را دارند.

---

## ۹. ملاحظات فنی

### ⚡ Performance
* **سرعت و حافظه:** `ValueTuple` یک `struct` است و معمولاً در **Stack** تخصیص می‌یابد. این یعنی **بدون فشار به Garbage Collector (GC)** و سرعت بسیار بالا.
* **هشدار Boxing:** اگر Tuple را به `object` یا `ValueType` کست کنید، دچار Boxing شده و در Heap تخصیص می‌یابد که پرفورمنس را به شدت کاهش می‌دهد.
* **Tuple های بزرگ:** اگر تعداد اعضا بیشتر از ۷ شود، کامپایلر از فیلد `TRest` استفاده می‌کند که یک Tuple تو در تو می‌سازد و ممکن است اندکی سربار محاسباتی ایجاد کند.

### 👁️ خوانایی (Readability) و نگهداری (Maintainability)
* **خوانایی:** با نام‌گذاری مناسب، خوانایی بالایی دارد. اما اگر نام‌ها حذف شوند، کد غیرقابل فهم می‌شود.
* **Maintainability:** **بسیار ضعیف.** شما نمی‌توانید به یک Tuple موجود فیلد جدیدی اضافه کنید (چون Signature متد تغییر می‌کند). اگر داده‌های شما نیاز به تکامل (Evolution) دارند، از Tuple استفاده نکنید.

### 📦 Serialization و Versioning
* **Serialization:** **بسیار خطرناک!** بسیاری از Serializerها (مثل نسخه‌های قدیمی Newtonsoft یا تنظیمات پیش‌فرض System.Text.Json) نام‌های سفارشی شما (`UserId`, `Name`) را نادیده گرفته و داده‌ها را به صورت آرایه `[1, "Ali"]` یا با نام‌های `Item1`، `Item2` سریال می‌کنند.
* **Versioning:** برای نسخه‌بندی (Versioning) مناسب نیستند. نمی‌توانید فیلدهای Optional به آن‌ها اضافه کنید بدون اینکه کدهای کلاینت را بشکنید.

---

## 📋 Checklist: قبل از استفاده از Tuple این موارد را بررسی کن

قبل از نوشتن `(Type1, Type2)` در کد خود، این چک‌لیست را مرور کنید:

- [ ] آیا این داده‌ها **موقت** هستند و فقط در یک Scope مشخص استفاده می‌شوند؟
- [ ] آیا تعداد اعضا **کم (زیر ۵ تا)** است؟
- [ ] آیا برای همه اعضا **نام معنادار** انتخاب کرده‌ام؟
- [ ] آیا این متد **Private/Internal** است؟ (برای Public هرگز استفاده نکنید).
- [ ] آیا نیاز به **Serialization** (JSON, XML, DB) دارد؟ (اگر بله، از `record` یا `class` استفاده کنید).
- [ ] آیا این داده‌ها در آینده نیاز به **اضافه شدن فیلد جدید** (Versioning) دارند؟ (اگر بله، از `class` یا `record` استفاده کنید).
- [ ] آیا این داده‌ها نیاز به **اعتبارسنجی (Validation)** یا **رفتار (متد)** دارند؟ (اگر بله، Tuple ممنوع است).
- [ ] آیا می‌خواهم از آن به عنوان **کلید Dictionary** استفاده کنم؟ (اگر بله، Tuple بهترین انتخاب است).

---

## 📚 منابع رسمی و معتبر (Microsoft Documentation)

برای مطالعه بیشتر و ارجاع به مستندات رسمی، لینک‌های زیر پیشنهاد می‌شوند:

1. **[Tuple types - C# reference | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)**
   *مرجع اصلی معرفی ValueTuple، نحوه تعریف و Deconstruction.*
2. **[Tuples and unnamed types in C# - Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct)**
   *بررسی تفاوت Tuple با Anonymous Types و نحوه استفاده در Pattern Matching.*
3. **[Record types - C# reference | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)**
   *مقایسه غیرمستقیم و درک اینکه چرا برای DTOها و مدل‌های داده‌ای باید به جای Tuple از Record استفاده کرد.*
4. **[System.ValueTuple Struct | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple)**
   *بررسی ساختار داخلی، پیاده‌سازی `IEquatable` و دلیل مناسب بودن آن برای Dictionary Key.*
5. **[TupleElementNamesAttribute Class | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.tupleelementnamesattribute)**
   *درک عمیق اینکه چگونه کامپایلر نام‌گذاری اعضا را در سطح Metadata حفظ می‌کند.*

---
*تدوین شده برای Repository آموزشی #C | آخرین بروزرسانی: آگوست ۲۰۲۶*