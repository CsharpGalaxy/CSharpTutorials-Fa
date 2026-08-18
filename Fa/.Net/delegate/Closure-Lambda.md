
## 📑 فهرست مطالب
1. [مبانی Closure و Capture](#1-مبانی-closure-و-capture)
2. [انواع Lambda در C#](#2-انواع-lambda-در-c)
3. [Capture متغیرهای بیرونی (Outer Variables)](#3-capture-متغیرهای-بیرونی-outer-variables)
4. [متدهای ناشناس (Anonymous Methods)](#4-متدهای-ناشنناس-anonymous-methods)
5. [لامبدا در برابر Local Function](#5-لامبدا-در-برابر-local-function)
6. [مباحث پیشرفته: Instanceها، حلقه‌ها و حافظه](#6-مباحث-پیشرفته-instanceها-حلقهها-و-حافظه)
7. [Closure و Expression Tree](#7-closure-و-expression-tree)
8. [منابع معتبر برای مطالعه بیشتر](#8-منابع-معتبر-برای-مطالعه-بیشتر)

---

## 1. مبانی Closure و Capture

### Closure چیست؟
به زبان ساده، **Closure** یک تابع (Function) به همراه "محیط" (Environment) یا همان متغیرهای بیرونی‌اش است. وقتی یک تابع، متغیری از_SCOPE_ بیرونی خود را استفاده می‌کند، آن متغیر را "Capture" (ضبط) می‌کند. Closure تضمین می‌کند که حتی اگر Scope بیرونی از بین رفته باشد، متغیرهای Capture شده زنده بمانند.

### Captured Variables چیستند؟
متغیرهایی که در یک Scope تعریف شده‌اند اما در یک Scope داخلی (مثل یک Lambda یا Local Function) استفاده می‌شوند، Captured Variable نام دارند.

### چه کسانی می‌توانند متغیرها را Capture کنند؟
*   **Lambda Expression / Statement**
*   **Anonymous Method**
*   **Local Function**

---

## 2. انواع Lambda در C#

### Lambda Expression چیست؟
یک عبارت کوتاه که مستقیماً یک مقدار را برمی‌گرداند.
```csharp
// Lambda Expression
Func<int, int> square = x => x * x; 
```
**ویژگی مهم:** فقط Lambda Expressionها می‌توانند به **Expression Tree** تبدیل شوند (مناسب برای LINQ to SQL / EF Core).

### Lambda Statement چیست؟
یک بلاک کد (شبیه متد معمولی) که با `{}` محصور شده و می‌تواند شامل چندین خط کد، حلقه و شرط باشد.
```csharp
// Lambda Statement
Action<string> greet = name => 
{ 
    string msg = "Hello " + name; 
    Console.WriteLine(msg); 
};
```
**تفاوت اصلی:** Lambda Statement را نمی‌توان به Expression Tree تبدیل کرد، زیرا Expression Tree فقط می‌تواند "عبارت" (Expression) را درک کند، نه "دستور" (Statement) را.

---

## 3. Capture متغیرهای بیرونی (Outer Variables)

### متغیرهای بیرونی (Outer Variables) چیستند؟
هر متغیری که قبل از تعریف Lambda/Local Function وجود داشته و در داخل آن‌ها استفاده شود.

### مواردی که می‌توانند Capture شوند:
1.  **Local Variables:** متغیرهای محلی تعریف شده در متد والد.
2.  **Parameters:** پارامترهای ورودی متد والد.
3.  **Fields & Properties:** (نکته مهم: فیلدها و پراپرتی‌ها مستقیماً کپچر نمی‌شوند، بلکه مرجع `this` کپچر می‌شود).

### زمان ارزیابی متغیرهای Capture شده (بسیار مهم!)
متغیرهای Capture شده در زمان **اجرا (Execution)** ارزیابی می‌شوند، نه در زمان **تعریف (Definition)**.
```csharp
int x = 10;
Action print = () => Console.WriteLine(x);
x = 20;
print(); // خروجی: 20 (نه 10!)
```

---

## 4. متدهای ناشناس (Anonymous Methods)

### تاریخچه و تعریف
قبل از C# 3.0 (که Lambda معرفی شد)، در C# 2.0 برای نوشتن توابع بی‌نام از Anonymous Method استفاده می‌شد.
```csharp
// C# 2.0 Style
Action<int> print = delegate(int number) 
{ 
    Console.WriteLine(number); 
};
```

### حذف پارامترها در Anonymous Method
در C# 2.0 اگر به پارامترها نیازی نداشتید، می‌توانستید آن‌ها را حذف کنید (این قابلیت در Lambda وجود ندارد):
```csharp
button.Click += delegate { MessageBox.Show("Clicked!"); };
```

### Anonymous Method در مقایسه با Lambda
| ویژگی | Anonymous Method | Lambda Expression |
| :--- | :--- | :--- |
| **سینتکس** | سنگین و طولانی (`delegate { }`) | کوتاه و تمیز (`=>`) |
| **Expression Tree** | ❌ پشتیبانی نمی‌کند | ✅ پشتیبانی می‌کند |
| **Type Inference** | ❌ نیاز به تایپ پارامترها | ✅ استنتاج نوع خودکار |
| **حذف پارامتر** | ✅ ممکن است | ❌ ممکن نیست |

### Static Lambdas (C# 9.0)
برای جلوگیری از Capture تصادفی متغیرها و بهبود Performance، کلمه `static` اضافه شد:
```csharp
int y = 10;
// خطای کامپایلر! چون static اجازه capture متغیر y را نمی‌دهد.
Func<int, int> add = static (x) => x + y; 
```

---

## 5. لامبدا در برابر Local Function

### Local Function چیست؟
متدی که درون یک متد دیگر تعریف می‌شود (معرفی در C# 7.0).
```csharp
void ParentMethod()
{
    int localVar = 5;
    void LocalHelper() => Console.WriteLine(localVar);
    LocalHelper();
}
```

### شباهت‌ها و تفاوت‌ها با Lambda
*   **شباهت:** هر دو می‌توانند متغیرهای متد والد را Capture کنند.
*   **Recursion (بازگشت):** Local Functionها به راحتی و بدون نیاز به Delegate می‌توانند Recursion داشته باشند. در Lambda باید از Delegate استفاده کنید.
*   **دسترسی به متغیرها:** هر دو به متغیرهای متد والد دسترسی دارند.

### Performance و Heap Allocation
*   **Lambda:** تقریباً همیشه نیاز به **Heap Allocation** دارد (چون باید یک شیء Delegate بسازد).
*   **Local Function:** اگر متغیری را Capture نکند و به Delegate تبدیل نشود، کامپایلر آن را به یک متد `static` در سطح کلاس تبدیل می‌کند و **هیچ سربار حافظه‌ای (Heap Allocation)** ندارد!

### چه زمانی کدام بهتر است؟
*   **Local Function:** وقتی نیاز به Recursion دارید، وقتی می‌خواهید از `yield return` یا `async/await` در یک تابع داخلی استفاده کنید (قبل از C# 10)، و وقتی می‌خواهید از Heap Allocation جلوگیری کنید.
*   **Lambda:** وقتی می‌خواهید کد را به عنوان پارامتر پاس دهید (مثل LINQ)، یا می‌خواهید Expression Tree بسازید.

---

## 6. مباحث پیشرفته: Instanceها، حلقه‌ها و حافظه

### Closure Instance چیست و تعداد آن‌ها چقدر است؟
وقتی کامپایلر #C یک Closure می‌بیند، یک کلاس مخفی (مثل `<>c__DisplayClass`) تولید می‌کند. متغیرهای Capture شده به فیلدهای این کلاس تبدیل می‌شوند.
*   **تعداد Instance:** به ازای هر **Scope** که متغیرها در آن به اشتراک گذاشته می‌شوند، **یک Instance** ساخته می‌شود. اگر دو Lambda در یک Scope، متغیرهای یکسانی را Capture کنند، هر دو به **یک Instance مشترک** اشاره می‌کنند.

### تأثیر Closure بر Lambdaهای داخل Loop (باگ کلاسیک!)
```csharp
var actions = new List<Action>();
for (int i = 0; i < 5; i++)
{
    actions.Add(() => Console.WriteLine(i));
}
foreach (var act in actions) act();
```
**خروجی:** پنج بار عدد `5` چاپ می‌شود!
**دلیل:** متغیر `i` یک بار Capture شده است. همه Lambdaها به **یک فیلد مشترک** در Closure Instance اشاره می‌کنند.
**راه‌حل:** تعریف یک متغیر محلی درون حلقه:
```csharp
for (int i = 0; i < 5; i++)
{
    int localCopy = i; // هر بار یک Instance جدید ساخته می‌شود
    actions.Add(() => Console.WriteLine(localCopy));
}
```

### طول عمر متغیر Capture شده و تأثیر بر GC
وقتی متغیری Capture می‌شود، طول عمر (Lifetime) آن از Scope اصلی خارج شده و به طول عمر Delegate گره می‌خورد.
**خطر Memory Leak:** اگر یک Lambda را به یک Event Handler طولانی‌مدت (مثل یک Event در یک Singleton) متصل کنید، تمام متغیرهای Capture شده (و حتی `this` اگر فیلدی استفاده شده باشد) تا زمانی که Event را Unsubscribe نکنید، در Heap زنده می‌مانند و توسط **GC** جمع‌آوری نمی‌شوند!

---

## 7. Closure و Expression Tree

### Expression Tree چیست؟
ساختاری از داده‌ها که کد را به جای "اجرا"، به صورت "داده" (Data) ذخیره می‌کند (بسیار پرکاربرد در Entity Framework برای تبدیل کد #C به SQL).

### محدودیت‌های Closure در Expression Tree
1.  فقط **Lambda Expression** می‌تواند Expression Tree باشد.
2.  وقتی یک Closure به Expression Tree تبدیل می‌شود (مثلاً در `IQueryable.Where(x => x.Id == myVar)`)، کامپایلر متغیر `myVar` را به صورت یک `ConstantExpression` یا `MemberExpression` در درخت تزریق می‌کند.
3.  **نکته پیشرفته:** در Expression Treeها، متغیرهای Capture شده در زمان ساخت درخت ارزیابی و "منجمد" (Freeze) می‌شوند، برخلاف Lambdaهای معمولی که در زمان اجرا ارزیابی می‌شوند.

---

## 8. منابع معتبر برای مطالعه بیشتر

برای درک عمیق‌تر و اثبات علمی این مباحث، منابع زیر "انجیل" برنامه‌نویسان #C محسوب می‌شوند:

1.  **کتاب C# in Depth (نوشته Jon Skeet)**
    *   *فصل‌های مربوط به Delegates و Closures.* (بهترین منبع برای درک نحوه کار کامپایلر در تولید Display Classes).
2.  **کتاب CLR via C# (نوشته Jeffrey Richter)**
    *   *بخش‌های مربوط به Delegate و Memory Management.* (منبع عالی برای درک Heap Allocation و GC).
3.  **مستندات رسمی Microsoft (Microsoft Learn)**
    *   [Lambda expressions (C# reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions)
    *   [Local functions (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions)
4.  **بلاگ Eric Lippert (Fabulous Adventures in Coding)**
    *   مقالات مربوط به *Closing over the loop variable considered harmful*. (اریک یکی از طراحان اصلی کامپایلر #C در مایکروسافت بوده است).
5.  **کتاب Pro C# 10 with .NET 6 (نوشته Andrew Troelsen)**
    *   فصل‌های مربوط به Delegates, Events و Lambda.

---
💡 **نکته پایانی:** درک Closure فقط برای نوشتن کد تمیز نیست؛ بلکه برای جلوگیری از Memory Leakها و باگ‌های عجیب در حلقه‌ها و Event Handlerها حیاتی است. کدهای این ریپازیتوری را اجرا کنید و با Debugger رفتار Heap را مشاهده کنید!

