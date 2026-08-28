# آموزش عمیق Closure و ValueTuple در #C

به بخش پیشرفته‌تر و بسیار جذاب زبان #C خوش آمدید. در این مقاله از Repository آموزشی، قرار است به یکی از مفاهیمی بپردازیم که اگرچه کامپایلر به‌صورت خودکار برای ما انجام می‌دهد، اما درک عمیق آن برای نوشتن کدهای بهینه (Performance-friendly) و جلوگیری از باگهای عجیب، حیاتی است: **کلوژر (Closure)** و تقاطع آن با **ValueTuple**.

---

## بخش اول: مفاهیم پایه (برای شروع از صفر)

برای درک Closure، ابتدا باید با دو مفهوم ساده آشنا شویم:

### ۱. Lambda Expression (عبارت لامبدا)
لامبدا در واقع یک تابع بی‌نام (Anonymous) و کوتاه است. 
```csharp
// یک تابع معمولی
int Add(int a, int b) { return a + b; }

// معادل آن با Lambda
Func<int, int, int> add = (a, b) => a + b;
```

### ۲. Captured Variable (متغیر Capture شده)
فرض کنید داخل یک متد، یک متغیر محلی (Local Variable) تعریف می‌کنید. حالا داخل یک Lambda که در همان متد قرار دارد، از آن متغیر استفاده می‌کنید. به آن متغیر محلی، یک **Captured Variable** می‌گویند.

```csharp
public void DoSomething()
{
    int messageCode = 100; // متغیر محلی
    
    // Lambda که از متغیر محلی استفاده کرده است
    Action print = () => Console.WriteLine($"Code is: {messageCode}");
    
    print();
}
```
در اینجا `messageCode` یک متغیر Capture شده است.

### ۳. Closure (کلوژر یا بستار)
حالا سوال اصلی: **اگر متد `DoSomething` تمام شود و از بین برود، تکلیف متغیر `messageCode` چه می‌شود؟** مگر قرار نبود متغیرهای محلی با پایان متد از بین بروند؟
اینجاست که **Closure** وارد عمل می‌شود. کلوژر ترکیبی از «تابع» و «محیطی» است که متغیرهای آن را در آغوش گرفته و زنده نگه می‌دارد. 
به زبان ساده: **کلوژر باعث می‌شود عمر متغیرهای محلی، بیشتر از عمر خودِ متد باشد.**

---

## بخش دوم: زیر کاپوت (Memory و Heap Allocation)

### افسانه Stack و Heap در #C
قبل از هر چیز، یک نکته بسیار مهم: **در #C هرگز نمی‌توانیم با قطعیت ۱۰۰٪ بگوییم متغیرهای محلی (Local Variables) همیشه روی Stack ذخیره می‌شوند.** 
کامپایلر #C و CLR بسیار هوشمند هستند. آن‌ها ممکن است متغیرهای محلی را روی Stack، روی Heap، یا حتی مستقیماً داخل CPU Registers ذخیره کنند (بسته به بهینه‌سازی‌ها).

**اما یک قانون قطعی وجود دارد:**
وقتی شما یک متغیر محلی را **Capture** می‌کنید (یعنی داخل یک Lambda یا Local Function از آن استفاده می‌کنید که قرار است بعداً اجرا شود)، کامپایلر دیگر نمی‌تواند آن متغیر را در Stack یا Register رها کند، چون ممکن است Scope اصلی متد تمام شده باشد اما Lambda هنوز بخواهد به آن دسترسی داشته باشد.

### کامپایلر چه می‌کند؟ (Heap Allocation)
وقتی کامپایلر متوجه Capture شدن یک متغیر می‌شود، در پشت صحنه (Behind the scenes) یک **کلاس مخفی (Compiler-generated class)** روی **Heap** می‌سازد. متغیر Capture شده را به عنوان یک Field درون این کلاس قرار می‌دهد و Lambda شما در واقع به یک متد از همین کلاس مخفی تبدیل می‌شود.

**نتیجه:** Capture کردن متغیرها = **تخصیص حافظه روی Heap (Heap Allocation)** = **فشار روی Garbage Collector**.

---

## بخش سوم: مثال‌های عملی

### ۱. مثال ساده (تفاوت Capture شدن Value در برابر Variable)
بسیاری از مبتدیان فکر می‌کنند "مقدار" متغیر کپی و Capture می‌شود. خیر! **خودِ متغیر (محل ذخیره آن)** Capture می‌شود.

```csharp
public void SimpleClosure()
{
    int counter = 0;
    
    Action increment = () => counter++;
    Action print = () => Console.WriteLine(counter);

    increment();
    increment();
    print(); // خروجی: 2
}
```
*تحلیل:* هر دو Lambda به **یک محل ذخیره واحد** (که حالا روی Heap است) اشاره می‌کنند.

### ۲. مثال با Lambda و حلقه (باگ کلاسیک)
```csharp
public void LambdaInLoop()
{
    var actions = new List<Action>();
    
    // توجه: متغیر i Capture شده است
    for (int i = 0; i < 3; i++)
    {
        actions.Add(() => Console.WriteLine(i));
    }

    foreach (var action in actions) action();
}
// خروجی: 3، 3، 3 (چون همه به یک متغیر i اشاره دارند که در نهایت 3 شده است)
```

### ۳. مثال با Local Function
از #C 7.0 به بعد، می‌توانیم به جای Lambda از Local Function استفاده کنیم. رفتار Closure دقیقاً یکسان است.

```csharp
public void LocalFunctionClosure()
{
    string status = "Pending";

    void UpdateStatus(string newStatus)
    {
        status = newStatus; // متغیر status از Scope بیرونی Capture شده است
    }

    UpdateStatus("Completed");
    Console.WriteLine(status); // خروجی: Completed
}
```

---

## بخش چهارم: بررسی دقیق اینکه چه چیزی Capture می‌شود

وقتی می‌گوییم چیزی Capture می‌شود، دقیقاً منظورمان چیست؟
1. **متغیرهای محلی (Local Variables):** بله، به Field های کلاس مخفی تبدیل می‌شوند.
2. **پارامترهای متد (Method Parameters):** بله، اگر داخل Lambda استفاده شوند، Capture می‌شوند.
3. **متغیر `this` (در کلاس‌ها):** بله! اگر داخل Lambda از `this` یا فیلدهای کلاس استفاده کنید، خودِ مرجع `this` Capture می‌شود. این یعنی **کلاس مخفیِ Closure، یک فیلد از نوع کلاسِ شما خواهد داشت** که می‌تواند باعث Memory Leak (عدم جمع‌آوری زباله‌ها) شود اگر Lambda عمری طولانی‌تر از آبجکت اصلی داشته باشد.

---

## بخش پنجم: ValueTuple داخل Closure و تغییر رفتار ذخیره‌سازی

حالا به سراغ موضوع جذاب و تخصصی این مقاله می‌رویم: **تاثیر ValueTuple بر Closure.**

فرض کنید می‌خواهید چندین متغیر محلی را در یک Closure استفاده کنید:

```csharp
public void MultipleCaptures()
{
    int x = 10;
    int y = 20;
    string name = "Test";

    Action act = () => Console.WriteLine($"{x}, {y}, {name}");
}
```
**اتفاق پشت صحنه:** کامپایلر یک کلاس مخفی روی Heap می‌سازد که ۳ فیلد دارد (`int x`, `int y`, `string name`).

### استفاده از ValueTuple برای سازماندهی
شما می‌توانید متغیرها را در یک `ValueTuple` قرار دهید و سپس آن را Capture کنید:

```csharp
public void ValueTupleClosure()
{
    // تعریف یک ValueTuple
    (int x, int y, string name) myData = (10, 20, "Test");

    Action act = () => Console.WriteLine($"{myData.x}, {myData.y}, {myData.name}");
}
```

### تغییر رفتار محل ذخیره (نکته طلایی)
همانطور که می‌دانید، `ValueTuple` یک **Struct** (نوع مقداری / Value Type) است. پس باید روی Stack باشد، درست است؟
**خیر! در اینجا یک تغییر رفتار رخ می‌دهد.**

چون `myData` داخل Lambda **Capture** شده است، کامپایلر مجبور است آن را در کلاس مخفیِ روی Heap قرار دهد. 
* **ساختار کلاس مخفی:** به جای ۳ فیلد مجزا، این کلاس فقط **یک فیلد** از نوع `ValueTuple<int, int, string>` خواهد داشت.
* **محل ذخیره:** خودِ Struct (یعنی `myData`) چون فیلدی از یک کلاس (Reference Type) روی Heap است، **درون Heap ذخیره می‌شود** (نه Stack).

**قانون مهم:** هر Value Typeای که عضوِ یک Reference Type (مثل کلاسِ مخفیِ Closure) قرار بگیرد، همراه با آن Reference Type روی Heap زندگی می‌کند.

---

## بخش ششم: نکات Performance و بهینه‌سازی

استفاده از Closure و Lambda دارای هزینه‌هایی است که باید در برنامه‌های حساس به Performance (مثل بازی‌سازی یا Backend با ترافیک بالا) به آن‌ها دقت کنید:

1. **هزینه Heap Allocation:** هر بار که متدی که حاوی Closure است فراخوانی می‌شود، کامپایلر یک نمونه جدید از کلاس مخفی را روی Heap می‌سازد (مگر اینکه کامپایلر تشخیص دهد هیچ متغیری تغییر نمی‌کند و آن را Cache کند). این یعنی فشار بر GC.
2. **ValueTuple در برابر متغیرهای مجزا:** 
   * آیا استفاده از ValueTuple در Closure باعث کاهش Allocation می‌شود؟ **خیر.** در هر دو حالت، یک کلاس مخفی روی Heap ساخته می‌شود.
   * **اما** استفاده از ValueTuple ممکن است ساختار کلاس مخفی را کمی فشرده‌تر کند (یک فیلد Struct به جای چندین فیلد Primitive) که در مدیریت Metadata توسط CLR کمی سربار را کاهش می‌دهد، اما از نظر حجم حافظه تخصیص یافته روی Heap تفاوت چشمگیری ندارد.
3. **خطر Boxing:** اگر ValueTuple خود را به `object` یا `ValueType` کست کنید، دچار Boxing شده و روی Heap کپی می‌شود. اما در حالت عادیِ Closure، چون فیلدِ یک کلاس است، نیازی به Boxing ندارد و به‌صورت Value Type درون Heap قرار می‌گیرد.
4. **توصیه Performance:** اگر Lambda شما هیچ متغیر محلی‌ای را از Scope بیرونی نیاز ندارد (Stateless است)، کامپایلر به‌صورت خودکار آن را به یک `static` متد تبدیل می‌کند و **هیچ Allocation روی Heap رخ نمی‌دهد**. پس تا حد ممکن از Capture کردن متغیرهای غیرضروری خودداری کنید.

```csharp
// بد: متغیر غیرضروری Capture شده و باعث Heap Allocation می‌شود
int unusedVar = 5; 
Action bad = () => Console.WriteLine("Hello"); 

// خوب: هیچ متغیری Capture نشده. کامپایلر این را به یک Delegate استاتیک و بدون Allocation تبدیل می‌کند.
Action good = () => Console.WriteLine("Hello"); 
```

---

## منابع و مراجع معتبر (Microsoft Docs)

برای مطالعه بیشتر و اطمینان از صحت مطالب، مستندات رسمی زیر پیشنهاد می‌شوند:

1. **[Lambda expressions (C# reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions)**
   * *بخش‌های مربوط به Captured variables و Scopes.*
2. **[Local functions (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions)**
   * *بررسی نحوه رفتار Local functions به عنوان Closure.*
3. **[C# Language Specification - Anonymous function expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1116-anonymous-function-expressions)**
   * *مستندات فنی درباره نحوه تبدیل Lambda به Delegate و ساخت Closure.*
4. **[Performance tips for C#](https://learn.microsoft.com/en-us/dotnet/csharp/write-safe-efficient-code)**
   * *اشاراتی به نحوه مدیریت حافظه و کاهش Allocatioها در استفاده از Delegateها.*

---
*نویسنده: [نام شما / نام تیم آموزشی]*
*تاریخ بازبینی: August 2026*
*این مقاله برای Repository آموزشی #C تهیه شده است. در صورت داشتن سوال یا پیشنهاد، لطفاً Issue ثبت کنید.*