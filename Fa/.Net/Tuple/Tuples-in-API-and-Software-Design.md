این یک مقاله جامع، ساختاریافته و کاربردی است که دقیقاً برای قرار گرفتن در یک Repository آموزشی C# طراحی شده است. می‌توانید این متن را به عنوان یک فایل Markdown (مثلاً `Tuple-in-API-Design.md`) در پروژه خود استفاده کنید.

***

# طراحی API در C#: هنر و ظرافت استفاده از Tupleها

در سی‌شارپ مدرن (از نسخه 7.0 به بعد)، `ValueTuple`ها ابزار قدرتمندی برای گروه‌بندی داده‌ها هستند. اما استفاده از آن‌ها مانند یک شمشیر دو لبه است؛ در جای درست، کد را تمیز و خوانا می‌کنند و در جای اشتباه، کابوسی برای نگهداری، نسخه‌بندی و قراردادهای API ایجاد می‌کنند.

در این مقاله، بررسی می‌کنیم که دقیقاً **کجا**، **چگونه** و **چرا** باید از Tupleها در طراحی نرم‌افزار استفاده کنیم (یا نکنیم!).

---

## ۱. جایگاه Tuple در لایه‌های مختلف نرم‌افزار

### Private Method (متدهای خصوصی)
**وضعیت: ✅ عالی و بسیار توصیه‌شده**
در متدهای داخلی و خصوصی، Tupleها پادشاهی می‌کنند. وقتی نیاز دارید چند مقدار مرتبط را از یک متد خصوصی برگردانید، ساختن یک کلاس یا استراکتِ یک‌بار مصرف (One-off class) فقط برای این کار، باعث ایجاد آلودگی کد (Code Clutter) می‌شود. Tuple این مشکل را حل می‌کند.

### Public API و Library (کتابخانه‌ها و APIهای عمومی)
**وضعیت: ❌ خطرناک و به‌شدت نهی‌شده**
در APIهای عمومی (Public APIs) که توسط توسعه‌دهندگان دیگر مصرف می‌شوند، Tupleها ایده بدی هستند. APIهای عمومی نیاز به ثبات، مستندات دقیق و قراردادهای مشخص دارند که Tupleها از پسِ آن برنمی‌آیند.

### Return Type (نوع بازگشتی)
استفاده از Tuple به عنوان Return Type برای **مرزهای داخلی** (Internal Boundaries) خوب است، اما برای **مرزهای بیرونی** (External Boundaries مثل Web API یا Public Library) ممنوع است.
*نکته خوانایی:* استفاده از Named Tuples مثل `(int Id, string Name)` خوانایی را در کد بالا می‌برد، اما به یاد داشته باشید که این نام‌ها فقط در زمان کامپایل وجود دارند و در زمان اجرا (Runtime) از بین می‌روند.

---

## ۲. چالش‌های Tuple در مهندسی نرم‌افزار

### خوانایی و Maintainability (نگهداری‌پذیری)
وقتی یک Tuple بیشتر از ۳ یا ۴ المان داشته باشد (مثلاً `(int, string, bool, DateTime, Guid)`)، خوانایی به‌شدت افت می‌کند. همچنین، شما **نمی‌توانید** برای المان‌های یک Tuple مستندات XML (XML Comments) بنویسید تا در IntelliSense نمایش داده شود. این موضوع نگهداری‌پذیری را در پروژه‌های بزرگ نابود می‌کند.

### Versioning و Breaking Changes (تغییرات مخرب)
فرض کنید در نسخه 1 کتابخانه خود متدی دارید که `(int Id, string Name)` برمی‌گرداند. در نسخه 2، نیاز به اضافه کردن یک `Email` پیدا می‌کنید. 
تغییر نوع بازگشتی به `(int Id, string Name, string Email)` یک **Breaking Change** بزرگ است. هر کدی که از متد شما استفاده می‌کرده و عمل Deconstruct انجام می‌داد، در زمان کامپایل با خطا مواجه می‌شود.

### Binary Compatibility (سازگاری باینری)
این مورد حتی از Breaking Change هم بدتر است. در سی‌شارپ، `ValueTuple<A, B>` و `ValueTuple<A, B, C>` دو **Type** کاملاً متفاوت در سطح CLR هستند. اگر شما یک المان به Tuple اضافه کنید، امضای متد در سطح باینری (IL) تغییر می‌کند. این یعنی حتی اگر کد مصرف‌کننده را دوباره کامپایل نکنید، با خطای `MissingMethodException` در زمان اجرا مواجه خواهد شد.

### Serialization, JSON و API Contract
این بزرگترین نقطه ضعف Tupleهاست. 
* **قرارداد API:** نام المان‌های Tuple در JSON خروجی حفظ نمی‌شود.
* **Serialization:** فریم‌ورک‌هایی مثل `System.Text.Json` یا `Newtonsoft.Json` نمی‌توانند به درستی Tupleها را Serialize/Deserialize کنند. آن‌ها معمولاً به صورت آرایه یا با نام‌های پیش‌فرض `Item1`, `Item2` در JSON ظاهر می‌شوند و شما نمی‌توانید از Attributeهایی مثل `[JsonPropertyName]` روی المان‌های Tuple استفاده کنید.

```csharp
// خروجی JSON یک Tuple فاجعه‌بار است!
var user = (Id: 1, Name: "Ali");
// خروجی System.Text.Json: {"Item1":1,"Item2":"Ali"} 
// نام‌های Id و Name کاملاً گم شده‌اند!
```

---

## ۳. ماتریس تصمیم‌گیری: Tuple در برابر DTO در برابر Record

برای انتخاب ابزار مناسب، از این راهنما استفاده کنید:

| ویژگی | Tuple | Record (C# 9+) | DTO (Class/Struct سنتی) |
| :--- | :--- | :--- | :--- |
| **بهترین کاربرد** | متدهای خصوصی، گروه‌بندی موقت ۲-۳ داده | APIهای عمومی، بازگشت داده‌های Immutable | انتقال داده‌های پیچیده، Mutable، ORMها |
| **Serialization (JSON)** | ❌ بسیار ضعیف | ✅ عالی | ✅ خوب |
| **API Contract** | ❌ نامعتبر | ✅ معتبر و پایدار | ✅ معتبر و پایدار |
| **تغییرپذیری (Mutability)**| ✅ قابل تغییر (مگر با پترن خاصی) | ❌ پیش‌فرض Immutable | ✅ کاملاً Mutable |
| **مستندات‌پذیری (XML Docs)**| ❌ غیرممکن | ✅ ممکن | ✅ ممکن |
| **تعداد المان‌ها** | حداکثر ۳ الی ۴ | محدودیتی ندارد | محدودیتی ندارد |

---

## ۴. سناریوهای واقعی و انتخاب مناسب

### سناریوی ۱: پیدا کردن بیشترین و کمترین مقدار در یک آرایه
**نیاز:** یک متد داخلی که هم `Max` و هم `Min` را برمی‌گرداند.
**انتخاب:** **Tuple**
```csharp
// Private Method
private (int Min, int Max) GetMinMax(int[] numbers)
{
    return (numbers.Min(), numbers.Max());
}
```
**چرا؟** این متد خصوصی است. نیازی به Serialize شدن ندارد. ساختن یک کلاس `MinMaxResult` فقط برای این متد، اضافه‌کاری است.

### سناریوی ۲: بازگرداندن اطلاعات کاربر از یک Web API
**نیاز:** یک Endpoint عمومی که `Id`, `UserName`, `Email` را به کلاینت برمی‌گرداند.
**انتخاب:** **Record**
```csharp
// Public API
public record UserResponseDto(int Id, string UserName, string Email);

[HttpGet("{id}")]
public UserResponseDto GetUser(int id) 
{
    // ...
    return new UserResponseDto(user.Id, user.UserName, user.Email);
}
```
**چرا؟** این یک Public API است. نیاز به JSON Serialization صحیح دارد. Recordها به‌صورت پیش‌فرض Immutable هستند (که برای پاسخ API عالی است)، قابلیت Serialize شدن دارند و می‌توان برای آن‌ها XML Doc نوشت.

### سناریوی ۳: دریافت یک فرم پیچیده و قابل ویرایش از کلاینت
**نیاز:** یک Endpoint که یک آبجکت پیچیده از کلاینت می‌گیرد، نیاز به Validation دارد و فیلدهای آن باید در ح پردازش تغییر کنند (Mutable).
**انتخاب:** **DTO (کلاس سنتی)**
```csharp
// Public API Input
public class UpdateUserRequestDto
{
    [Required] public string UserName { get; set; }
    [EmailAddress] public string Email { get; set; }
    // فیلدهای دیگر...
}
```
**چرا؟** اگرچه Recordها هم می‌توانند استفاده شوند، اما در سناریوهایی که نیاز به `Mutability` (تغییرپذیری) کامل، پیاده‌سازی `INotifyPropertyChanged`، یا مپینگ‌های پیچیده با ORMهای قدیمی دارید، DTOهای کلاسیک (Class) همچنان گزینه استاندارد و انعطاف‌پذیرتری هستند.

---

## ۵. خلاصه و قوانین طلایی

1. **قانون مرزها:** هرگز از Tuple برای عبور از مرزها (Public API، Network، Database) استفاده نکنید.
2. **قانون ۳ المان:** اگر نیاز دارید بیشتر از ۳ داده را گروه‌بندی کنید، از Tuple استفاده نکنید؛ یک کلاس، استراکت یا Record بسازید.
3. **قانون مستندات:** اگر فکر می‌کنید مصرف‌کننده کد نیاز به توضیحات IntelliSense برای هر فیلد دارد، Tuple نسازید.
4. **جایگزین مدرن:** برای APIهای عمومی، به جای DTOهای خالی (Empty DTOs)، تا حد امکان از **Record**ها استفاده کنید تا کد تمیزتر و ایمن‌تر (Thread-safe) باشد.

---

## 📚 منابع رسمی Microsoft

برای مطالعه بیشتر و عمیق‌تر، منابع رسمی زیر پیشنهاد می‌شوند:

1. **[Tuple types - C# reference | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)**
   * مرجع کامل نحوه تعریف، Deconstruct و کار با ValueTupleها.
2. **[Records - C# reference | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)**
   * راهنمای جامع Recordها و دلیل برتری آن‌ها در انتقال داده‌ها.
3. **[Framework Design Guidelines | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/)**
   * بخش Member Design و Returning Collections که به طور غیرمستقیم به عدم استفاده از انواع ناشناخته در Public APIها اشاره دارد.
4. **[System.Text.Json: How to serialize and deserialize | Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/how-to)**
   * بررسی محدودیت‌های Serialize کردن انواع خاص مانند Tupleها.

***
*نویسنده: [نام شما/تیم شما] | تاریخ بازبینی: August 2026*