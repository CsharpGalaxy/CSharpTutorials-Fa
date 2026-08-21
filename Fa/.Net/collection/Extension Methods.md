

# 🚀 آموزش جامع Extension Methods در #C (از مقدماتی تا پیشرفته)

به مستندات جامع **Extension Methods** (متدهای توسعه‌دهنده) خوش آمدید! اگر تا به حال از LINQ در سی‌شارپ استفاده کرده‌اید، بدون اینکه بدانید در حال استفاده از قدرتمندترین قابلیت Extension Methods بوده‌اید. در این آموزش، از صفر مطلق شروع کرده و تا تکنیک‌های پیشرفته پیش می‌رویم.

## 📑 فهرست مطالب
1. [بخش اول: مقدماتی](#بخش-اول-مقدماتی)
   - [مفهوم Extension Method چیست؟](#۱-مفهوم-extension-method-چیست)
   - [هدف و کاربرد](#۲-هدف-و-کاربرد-چرا-به-آن-نیاز-داریم)
   - [ساختار و قوانین پایه (Static Class, this, پارامتر اول)](#۳-ساختار-و-قوانین-پایه)
2. [بخش دوم: متوسط](#بخش-دوم-متوسط)
   - [Extension Method روی Typeهای موجود (Built-in Types)](#۴-افزودن-متد-به-typeهای-موجود)
   - [Extension Method روی Interfaceها](#۵-افزودن-متد-به-interfaceها)
3. [بخش سوم: پیشرفته](#بخش-سوم-پیشرفته)
   - [متدهای توسعه‌دهنده Generic](#۶-متدهای-generic-در-extension-methods)
   - [محدودیت‌ها و هشدارهای مهم](#۷-محدودیت‌ها-و-هشدارهای-مهم-best-practices)
4. [منابع و مراجع](#-منابع-و-مراجع)

---

## بخش اول: مقدماتی

### ۱. مفهوم Extension Method چیست؟
فرض کنید یک جعبه ابزار (یک کلاس) دارید که توسط شخص دیگری ساخته شده و شما به کدهای اصلی آن دسترسی ندارید (یا نمی‌خواهید آن‌ها را دستکاری کنید). حالا شما نیاز به یک ابزار جدید دارید. 
**Extension Method** به شما اجازه می‌دهد بدون اینکه کلاس اصلی را تغییر دهید، یا از آن ارث‌بری (Inheritance) کنید، **متدهای جدیدی** به آن اضافه کنید.

> 💡 **نکته کلیدی:** در واقعیت، متد جدیدی به کلاس اضافه *نمی‌شود*؛ بلکه کامپایلر سی‌شارپ یک **متد استاتیک** را به گونه‌ای ترجمه می‌کند که انگار روی آبجکت شما صدا زده شده است (Syntax شیک و تمیز).

### ۲. هدف و کاربرد (چرا به آن نیاز داریم؟)
* **خوانایی کد (Readability):** به جای نوشتن `MyClass.DoSomething(obj)`، می‌نویسیم `obj.DoSomething()`.
* **افزودن قابلیت به کلاس‌های Sealed:** کلاس‌هایی مثل `string` در سی‌شارپ `sealed` هستند و نمی‌توان از آن‌ها ارث‌بری کرد. Extension Method تنها راه افزودن متد به این کلاس‌هاست.
* **طراحی Fluent Interface:** همان‌طور که در LINQ می‌بینیم، می‌توانیم متدها را پشت سر هم زنجیره کنیم (Chain).
* **جلوگیری از تکرار کد (Utility Methods):** به جای ساخت کلاس‌های `StringHelper` یا `IntHelper`، متدها را مستقیماً به خود تایپ‌ها می‌چسبانیم.

### ۳. ساختار و قوانین پایه
برای ساخت یک Extension Method، باید **۳ قانون طلایی** زیر را رعایت کنید:

1. **Static Class:** متد باید حتماً درون یک **کلاس استاتیک** (`static class`) تعریف شود.
2. **Static Method:** خود متد نیز باید حتماً **استاتیک** (`static`) باشد.
3. **Modifier this:** پارامتر اول متد باید حتماً کلمه کلیدی **`this`** را قبل از نوع داده داشته باشد.

**پارامتر اول چیست؟**
پارامتر اول مشخص می‌کند که این متد قرار است به **کدام نوع داده (Type)** اضافه شود. کلمه `this` به کامپایلر می‌گوید: "این متد را به عنوان یک متد نمونه (Instance Method) برای این تایپ در نظر بگیر".

#### 📝 مثال عملی (ساختار پایه):
فرض کنید می‌خواهیم متدی بسازیم که تعداد کلمات یک رشته را بشمارد.

```csharp
// ۱. کلاس باید Static باشد
public static class StringExtensions
{
    // ۲. متد باید Static باشد
    // ۳. پارامتر اول باید this داشته باشد و نوع آن string است
    public static int GetWordCount(this string text)
    {
        if (string.IsNullOrWhiteSpace(text)) return 0;
        return text.Split(new char[] { ' ', '.', ',' }, StringSplitOptions.RemoveEmptyEntries).Length;
    }
}
```

**نحوه استفاده:**
```csharp
string myText = "Hello World, Welcome to C#";
int count = myText.GetWordCount(); // انگار GetWordCount جزو خود string است!
```

---

## بخش دوم: متوسط

### ۴. افزودن متد به Typeهای موجود
شما می‌توانید به هر تایپ موجود در .NET (مثل `int`, `DateTime`, `List<T>`) متد اضافه کنید. این کار باعث می‌شود کدهای شما بسیار طبیعی‌تر خوانده شوند.

**مثال: افزودن متد به `int` برای بررسی زوج یا فرد بودن:**
```csharp
public static class IntegerExtensions
{
    public static bool IsEven(this int number)
    {
        return number % 2 == 0;
    }
}

// استفاده:
int myNumber = 10;
if (myNumber.IsEven())
{
    Console.WriteLine("عدد زوج است");
}
```

### ۵. افزودن متد به Interfaceها
یکی از قدرتمندترین ویژگی‌های Extension Methods این است که شما می‌توانید یک **Interface** را گسترش دهید. وقتی این کار را می‌کنید، **تمام کلاس‌هایی که آن Interface را پیاده‌سازی (Implement) می‌کنند**، به صورت خودکار آن متد را به ارث می‌برند!

> 🌟 **آیا می‌دانستید؟** کل کتابخانه **LINQ** در سی‌شارپ بر پایه Extension Method روی Interface `IEnumerable<T>` ساخته شده است!

**مثال: گسترش یک Interface سفارشی:**
```csharp
public interface IAnimal
{
    string Name { get; }
}

public static class AnimalExtensions
{
    // این متد به Interface اضافه می‌شود
    public static string GetLoudName(this IAnimal animal)
    {
        return $"!!! {animal.Name.ToUpper()} !!!";
    }
}

// کلاس‌هایی که IAnimal را پیاده سازی می‌کنند:
public class Dog : IAnimal { public string Name { get; set; } }
public class Cat : IAnimal { public string Name { get; set; } }

// استفاده:
Dog myDog = new Dog { Name = "Rex" };
Console.WriteLine(myDog.GetLoudName()); // خروجی: !!! REX !!!
```

---

## بخش سوم: پیشرفته

### ۶. متدهای Generic در Extension Methods
شما می‌توانید Extension Methodها را **Generic** کنید تا روی هر نوع داده‌ای کار کنند.

```csharp
public static class ObjectExtensions
{
    // یک متد Generic که هر آبجکتی را به JSON تبدیل می‌کند
    public static string ToJson<T>(this T obj)
    {
        return System.Text.Json.JsonSerializer.Serialize(obj);
    }
}

// استفاده:
var user = new { Name = "Ali", Age = 30 };
string json = user.ToJson(); // نیازی به مشخص کردن <T> نیست، کامپایلر تشخیص می‌دهد
```

### ۷. محدودیت‌ها و هشدارهای مهم (Best Practices)
برای اینکه کدهای شما حرفه‌ای و تمیز بمانند، این نکات را حتماً رعایت کنید:

1. **اولویت با متدهای Instance:** اگر کلاسی از قبل متدی هم‌نام با Extension Method شما داشته باشد، **همیشه متد خود کلاس (Instance) اجرا می‌شود** و Extension Method نادیده گرفته می‌شود.
2. **عدم دسترسی به اعضای Private:** چون Extension Method در واقع یک متد استاتیک در یک کلاس دیگر است، **نمی‌تواند** به فیلدها یا متدهای `private` یا `protected` کلاس اصلی دسترسی داشته باشد.
3. **فقط متد، نه Property:** شما فقط می‌توانید **متد** گسترش دهید. نمی‌توانید Property، Event یا Operator اضافه کنید.
4. **آلوده کردن IntelliSense (مهمترین نکته):** اگر Extension Methodها را در `Namespace`های عمومی (مثل `System`) قرار دهید، در تمام پروژه به همه کلاس‌ها می‌چسبند و IntelliSense را شلوغ می‌کنند. 
   * **راه‌حل:** همیشه یک `Namespace` اختصاصی (مثل `MyProject.Extensions`) بسازید و فقط جاهایی که نیاز دارید `using` کنید.
5. **نام‌گذاری کلاس:** عرف جامعه سی‌شارپ این است که نام کلاس استاتیک را با پسوند **`Extensions`** تمام کنید (مثل `StringExtensions`).

---

## 📚 منابع و مراجع

این آموزش بر اساس مستندات رسمی و کتب مرجع زیر تدوین شده است. برای مطالعه عمیق‌تر، لینک‌های زیر پیشنهاد می‌شوند:

1. **[Microsoft Learn - Extension Methods (Official C# Docs)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)**
   * *توضیح:* مرجع اصلی و رسمی مایکروسافت برای زبان سی‌شارپ. (بسیار معتبر)
2. **[Microsoft Learn - How to implement a custom extension method](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-implement-and-call-a-custom-extension-method)**
   * *توضیح:* آموزش گام‌به‌گام مایکروسافت برای پیاده‌سازی و فراخوانی.
3. **کتاب C# in Depth (نوشته Jon Skeet)**
   * *توضیح:* فصل مربوط به LINQ و Extension Methods در این کتاب، دید بسیار عمیقی از نحوه کار کامپایلر به شما می‌دهد.
4. **[GeeksforGeeks - Extension Methods in C#](https://www.geeksforgeeks.org/extension-methods-in-c-sharp/)**
   * *توضیح:* مثال‌های متنوع و ساده برای درک بهتر مفاهیم پایه.

---
**💡 پیشنهاد برای مدیر ریپازیتوری:** 
*شما می‌توانید برای هر بخش یک پوشه (Folder) در ریپازیتوری خود بسازید و فایل‌های `.cs` حاوی کدهای این آموزش را همراه با کامنت‌های فارسی در آن‌ها قرار دهید تا کاربران بتوانند کدها را Clone کرده و در Visual Studio اجرا کنند.*

موفق باشید! 🚀