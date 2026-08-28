# درک عمیق Tuple و Assembly در #C: از Metadata تا Binary Compatibility

استفاده از `Tuple`ها در #C یکی از جذاب‌ترین و پرکاربردترین ویژگی‌های نسخه‌های جدید این زبان است. اما آیا تا به حال فکر کرده‌اید که وقتی یک Tuple را از یک Library به Application دیگر برمی‌گردانید، دقیقاً چه اتفاقی در سطح پایین‌تر (Assembly و Metadata) می‌افتد؟ 

در این مقاله آموزشی، ما از سطح مقدماتی فراتر رفته و به کالبدشکافی Tupleها در سطح Assembly، Metadata و چالش‌های Binary Compatibility می‌پردازیم.

---

## 📑 فهرست مطالب
- [Assembly چیست؟](#assembly-چیست)
- [Tuple Type در Assembly](#tuple-type-در-assembly)
- [Metadata و نقش TupleElementNamesAttribute](#metadata-و-نقش-tupleelementnamesattribute)
- [Tuple در Library و Application](#tuple-در-library-و-application)
- [مثال عملی با دو پروژه](#مثال-عملی-با-دو-پروژه)
- [Return Type در Assembly دیگر و Binary Compatibility](#return-type-در-assembly-دیگر-و-binary-compatibility)
- [در Runtime چه چیزهایی قابل مشاهده هستند؟](#در-runtime-چه-چیزهایی-قابل-مشاهده-هستند)
- [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
- [جمع‌بندی](#جمع‌بندی)
- [منابع معتبر](#منابع-معتبر)

---

## Assembly چیست؟

در دنیای #C و .NET، **Assembly** واحد اصلی کامپایل، استقرار و امنیت است. وقتی شما کد #C خود را کامپایل می‌کنید، خروجی آن یک Assembly است که معمولاً با پسوند `.dll` (برای کتابخانه‌ها) یا `.exe` (برای برنامه‌های اجرایی) ذخیره می‌شود.

یک Assembly شامل دو بخش اصلی است:
1. **IL Code (Intermediate Language):** کدهای میانه‌ای که توسط CLR اجرا می‌شوند.
2. **Metadata:** داده‌های توصیفی که شامل اطلاعاتی درباره Typeها، Methodها، Propertyها و Attributeهاست.

---

## Tuple Type در Assembly

وقتی شما در کد خود از سینتکس زیبای Tuple استفاده می‌کنید:
```csharp
(int Id, string Name) user = (1, "Ali");
```
کامپایلر #C یک کار جادویی انجام می‌دهد. در سطح Assembly و IL، چیزی به نام "Tuple با نام‌گذاری اختصاصی" وجود ندارد! 

کامپایلر این سینتکس را به یک **Struct عمومی** به نام `ValueTuple<T1, T2>` ترجمه می‌کند. بنابراین، در سطح Assembly، نوع متغیر `user` دقیقاً همان `System.ValueTuple<System.Int32, System.String>` است.

**نکته کلیدی:** CLR (محیط اجرای .NET) هیچ درکی از نام‌های `Id` و `Name` ندارد. برای CLR، این متغیر فقط یک `ValueTuple` با دو فیلد به نام‌های پیش‌فرض `Item1` و `Item2` است.

---

## Metadata و نقش TupleElementNamesAttribute

اگر CLR نام‌های Tuple را نمی‌فهمد، پس چگونه کامپایلر وقتی در پروژه دیگری از `user.Id` استفاده می‌کنید، متوجه می‌شود که منظور شما `Item1` است؟

اینجا **Metadata** وارد عمل می‌شود. وقتی کامپایلر #C با یک Tuple نام‌گذاری شده مواجه می‌شود، یک Attribute خاص به نام **`TupleElementNamesAttribute`** را در Metadata آن Method یا Property تزریق می‌کند.

این Attribute یک آرایه از رشته‌ها (نام‌های المان‌ها) را در خود ذخیره می‌کند. وقتی شما در پروژه دیگری به آن Library مراجعه می‌کنید، کامپایلر شما Metadata آن Assembly را می‌خواند، این Attribute را پیدا می‌کند و می‌فهمد که `Item1` در واقع `Id` است.

---

## Tuple در Library و Application

برای درک بهتر، بیایید دو محیط را در نظر بگیریم:
1. **Library (کتابخانه):** جایی که یک Method خروجی Tuple برمی‌گرداند.
2. **Application (برنامه):** جایی که از آن Method استفاده می‌کند.

ارتباط بین این دو صرفاً از طریق **Metadata** و **Binary Compatibility** برقرار می‌شود.

---

## مثال عملی با دو پروژه

بیایید یک سناریوی واقعی را شبیه‌سازی کنیم.

### پروژه اول: Library (نام پروژه: `MyLibrary`)
```csharp
namespace MyLibrary
{
    public class UserService
    {
        // کامپایلر در اینجا به صورت خودکار TupleElementNamesAttribute را به Metadata اضافه می‌کند
        public (int Id, string Name) GetUser()
        {
            return (1, "Ali");
        }
    }
}
```

### پروژه دوم: Application (نام پروژه: `MyApp`)
```csharp
using MyLibrary;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            var service = new UserService();
            
            // کامپایلر MyApp با خواندن Metadata کتابخانه می‌فهمد که Item1 همان Id است
            var user = service.GetUser(); 
            
            Console.WriteLine($"ID: {user.Id}, Name: {user.Name}");
            
            // استفاده از Item1 و Item2 نیز کاملاً مجاز است
            Console.WriteLine($"Item1: {user.Item1}"); 
        }
    }
}
```

---

## Return Type در Assembly دیگر و Binary Compatibility

این بخش یکی از **مهم‌ترین و خطرناک‌ترین** مباحث در استفاده از Tupleهاست.

فرض کنید شما `MyLibrary` را کامپایل کرده‌اید و `MyApp` به آن وابسته است. حالا در `MyLibrary` تصمیم می‌گیرید نام‌های Tuple را تغییر دهید:

```csharp
// تغییر در MyLibrary
public (int UserId, string FullName) GetUser() // نام‌ها تغییر کردند!
{
    return (1, "Ali");
}
```

### چه اتفاقی می‌افتد؟ (تحلیل Binary Compatibility)
از آنجایی که نوع داده‌ها (`int` و `string`) تغییر نکرده است، **امضای متد (Method Signature) در سطح IL هیچ تغییری نکرده است** (هر دو `ValueTuple<int, string>` هستند).

1. **اگر MyApp را Recompile کنید:** کامپایلر خطا می‌دهد! چون در کد `MyApp` از `user.Id` استفاده شده، اما در Metadata جدید `MyLibrary` نامی به اسم `Id` وجود ندارد (فقط `UserId` هست). **(Source Compatibility شکسته شده است).**
2. **اگر فقط DLL جدید را جایگزین کنید (بدون Recompile):** برنامه شما اجرا می‌شود! اما `user.Id` در برنامه شما همچنان به `Item1` اشاره می‌کند. **(Binary Compatibility حفظ شده است، اما با یک تله بزرگ!).**

**تله بزرگ:** اگر در `MyLibrary` علاوه بر نام، **نوع** یکی از المان‌ها را تغییر دهید (مثلاً `Id` را از `int` به `string` تغییر دهید)، امضای IL تغییر می‌کند و **Binary Compatibility کاملاً از بین می‌رود** و برنامه شما هنگام اجرا با خطای `MissingMethodException` مواجه می‌شود.

---

## در Runtime چه چیزهایی قابل مشاهده هستند؟

اگر از **Reflection** در زمان اجرا (Runtime) برای بررسی نوع بازگشتی متد استفاده کنید، چه می‌بینید؟

```csharp
var method = typeof(UserService).GetMethod("GetUser");
var returnType = method.ReturnType;

Console.WriteLine(returnType.Name); 
// خروجی: ValueTuple`2

// برای دیدن نام‌ها در Runtime باید خودتان Attribute را بخوانید:
var namesAttribute = method.ReturnTypeCustomAttributeData
    .FirstOrDefault(a => a.AttributeType.Name == "TupleElementNamesAttribute");

if (namesAttribute != null)
{
    var names = (IList<string>)namesAttribute.ConstructorArguments[0].Value;
    Console.WriteLine(string.Join(", ", names)); 
    // خروجی: Id, Name
}
```

**نتیجه‌گیری Runtime:** در زمان اجرا، نام‌های Tuple وجود خارجی ندارند. آن‌ها فقط در Metadata برای کمک به کامپایلر ذخیره شده‌اند. اگر از کتابخانه‌هایی مثل `System.Text.Json` برای Serialize کردن Tuple استفاده کنید، ممکن است خروجی شما به جای `{"Id":1, "Name":"Ali"}` به صورت `{"Item1":1, "Item2":"Ali"}` باشد (مگر اینکه Serializer به صورت خاص برای خواندن این Attribute برنامه‌نویسی شده باشد).

---

## نکات مهم و اشتباهات رایج

### ⚠️ اشتباهات رایج:
1. **استفاده از Tuple در Public API:** هرگز از Tupleهای نام‌گذاری شده برای متدهای Public در Libraryهای خود استفاده نکنید. تغییر نام المان‌ها در آینده، کدهای مصرف‌کنندگان را می‌شکند.
2. **تکیه بر نام‌ها در Serialization:** فرض نکنید که نام‌های Tuple در JSON یا XML حفظ می‌شوند.
3. **استفاده از Tupleهای پیچیده:** اگر Tuple شما بیش از ۳ یا ۴ المان دارد، خوانایی کد به شدت افت می‌کند.

### 💡 نکات مهم (Best Practices):
- **جایگزین‌ها:** برای Public APIها، به جای Tuple از **`class`**، **`struct`** یا **`record`** استفاده کنید. آن‌ها Metadata غنی‌تری دارند و Binary Compatibility بهتری ارائه می‌دهند.
- **مصرف داخلی:** Tupleها برای استفاده‌های **Private** یا **Internal** و یا بازگرداندن چند مقدار از یک متد در داخل همان کلاس، فوق‌العاده‌اند.
- **نام‌گذاری معنادار:** اگر مجبور به استفاده از Tuple هستید، حتماً المان‌ها را نام‌گذاری کنید تا کد در سمت مصرف‌کننده خوانا باشد.

---

## جمع‌بندی

Tupleها در #C ترکیبی از **سینتکس شیک** و **پشتیبانی عمیق کامپایلر** هستند. 
- در سطح **Assembly**، آن‌ها فقط `ValueTuple` هستند.
- در سطح **Metadata**، نام‌های آن‌ها توسط `TupleElementNamesAttribute` حفظ می‌شود.
- در سطح **Runtime**، نام‌ها ناپدید می‌شوند و فقط `Item1` و `Item2` باقی می‌مانند.

درک این مفاهیم به شما کمک می‌کند تا از تله‌های **Binary Compatibility** فرار کنید و معماری بهتری برای Libraryها و Applicationهای خود طراحی نمایید. به یاد داشته باشید: **Tuple برای راحتی برنامه‌نویس است، نه برای طراحی APIهای عمومی!**

---

## منابع معتبر

برای مطالعه بیشتر و عمیق‌تر، منابع زیر پیشنهاد می‌شوند:
1. [Microsoft Docs: Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)
2. [Microsoft Docs: TupleElementNamesAttribute](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.tupleelementnamesattribute)
3. [C# in Depth (4th Edition) by Jon Skeet - Chapter on Tuples](http://csharpindepth.com/)
4. [Pro .NET Memory Management by Konrad Kokosa (بخش مربوط به ValueTypes و Stack)](https://prodotnetmemory.com/)

---
*اگر این مقاله برای شما مفید بود، آن را Star کنید و به اشتراک بگذارید. نظرات و پرسش‌های خود را در بخش Issues مطرح کنید!*