
# آموزش جامع Dynamic در C# (از صفر تا پیشرفته)

این سند یک مرجع آموزشی کامل، ساختاریافته و عمیق برای درک مفهوم `dynamic` در زبان C# است. این راهنما از مفاهیم کاملاً مبتدی شروع شده و به مباحث پیشرفته‌ای مانند DLR، Expression Tree، Custom Binding و معماری نرم‌افزار می‌پردازد.

---

## فهرست مطالب

1. [مقدمه و مفاهیم پایه Dynamic](#1-مقدمه-و-مفاهیم-پایه-dynamic)
2. [Dynamic در مقابل Static Typing و Object](#2-dynamic-در-مقابل-static-typing-و-object)
3. [زیر کاپوت: CLR، Metadata و Roslyn](#3-زیر-کاپوت-clr-metadata-و-roslyn)
4. [مفاهیم Binding: Static vs Dynamic](#4-مفاهیم-binding-static-vs-dynamic)
5. [Runtime Binder و مدیریت خطاها](#5-runtime-binder-و-مدیریت-خطاها)
6. [Dynamic Expressions و Conversion](#6-dynamic-expressions-و-conversion)
7. [تعامل Dynamic با سایر ویژگی‌های C#](#7-تعامل-dynamic-با-سایر-ویژگیهای-c)
8. [Dynamic Language Runtime (DLR)](#8-dynamic-language-runtime-dlr)
9. [پیاده‌سازی اشیای پویا: DynamicObject و IDynamicMetaObjectProvider](#9-پیاده‌سازی-اشیای-پویا-dynamicobject-و-idynamicmetaobjectprovider)
10. [کاربردهای واقعی: COM، Reflection و JSON](#10-کاربردهای-واقعی-com-reflection-و-json)
11. [عملکرد (Performance) و IL](#11-عملکرد-performance-و-il)
12. [معماری نرم‌افزار و اشتباهات رایج](#12-معماری-نرم‌افزار-و-اشتباهات-رایج)
13. [جداول مقایسه‌ای جامع](#13-جداول-مقایسه‌ای-جامع)
14. [پروژه‌های عملی](#14-پروژه‌های-عملی)
15. [Testing، Debugging و Cheat Sheet](#15-testing-debugging-و-cheat-sheet)
16. [جمع‌بندی نهایی و منابع](#16-جمع‌بندی-نهایی-و-منابع)

---

## 1. مقدمه و مفاهیم پایه Dynamic

### Dynamic چیست؟
کلمه کلیدی `dynamic` در C# به کامپایلر می‌گوید: **«بررسی نوع (Type Checking) و حل‌وفصل فراخوانی اعضا (Member Resolution) را به زمان اجرا (Runtime) موکول کن.»**

```csharp
dynamic value = 10;
Console.WriteLine(value); // در زمان اجرا مشخص می‌شود که value چه متدی دارد.
```
* **چرا Dynamic Type گفته می‌شود؟** زیرا نوع متغیر در زمان کامپایل ثابت است (خود کلمه `dynamic`)، اما *رفتار* و *اعضای* آن در زمان اجرا تعیین می‌شود.
* **آیا `dynamic` یک Type در CLR است؟** خیر. در سطح CLR، `dynamic` دقیقاً همان `System.Object` است. تفاوت فقط در Metadata و رفتار کامپایلر C# است.
* **تفاوت Compile Time و Runtime:** در Compile Time، کامپایلر فرض می‌کند `dynamic` هر عضوی (متد، پراپرتی) را دارد. در Runtime، موتور DLR بررسی می‌کند که آیا آن عضو واقعاً وجود دارد یا خیر.

---

## 2. Dynamic در مقابل Static Typing و Object

### Static Typing vs Dynamic Typing
```csharp
int x = 10;          // Static: کامپایلر می‌داند x فقط متدهای int را دارد.
dynamic y = 10;      // Dynamic: کامپایلر هیچ فرضی درباره متدهای y نمی‌کند.
```
| ویژگی | Static Typing | Dynamic Typing |
| :--- | :--- | :--- |
| **Type Checking** | Compile Time | Runtime |
| **IntelliSense** | کامل و دقیق | محدود یا عدم وجود |
| **Refactoring** | امن و خودکار | پرخطر و دستی |
| **Type Safety** | بالا | پایین (موکول به Runtime) |

### Dynamic و Object (تفاوت حیاتی)
بسیاری فکر می‌کنند `dynamic` فقط یک `object` راحت‌تر است، اما تفاوت بنیادین در **رفتار کامپایلر** است:

```csharp
object obj = "Hello";
// obj.Length; // ❌ خطای کامپایل: 'object' تعریفی برای 'Length' ندارد.
int len1 = ((string)obj).Length; // ✅ نیاز به Cast صریح دارد.

dynamic dyn = "Hello";
int len2 = dyn.Length; // ✅ کامپایل می‌شود. در Runtime بررسی می‌شود.
```
* **شباهت Runtime:** در نهایت، هر دو در حافظه به عنوان یک شیء ذخیره می‌شوند.
* **تفاوت Compiler:** کامپایلر برای `object` قوانین سخت‌گیرانه Type Safety را اعمال می‌کند، اما برای `dynamic` این قوانین را غیرفعال کرده و کد تولید می‌کند که از DLR برای حل‌وفصل در Runtime استفاده کند.

---

## 3. زیر کاپوت: CLR، Metadata و Roslyn

### آیا Dynamic واقعاً یک Type است؟
خیر. CLR نوعی به نام `Dynamic` ندارد. وقتی شما می‌نویسید `dynamic x;`، کامپایلر Roslyn آن را به `object x;` تبدیل می‌کند، اما یک اتریبیوت خاص به Metadata اضافه می‌کند: `[DynamicAttribute]`.

### DynamicAttribute
این اتریبیوت (`System.Runtime.CompilerServices.DynamicAttribute`) به ابزارها (مثل Debugger یا کامپایلرهای دیگر) می‌گوید که این `object` در اصل به عنوان `dynamic` در نظر گرفته شده است.

### فرآیند کامپایل (Roslyn Pipeline)
```text
Source Code (dynamic x = 10;)
      ↓
Roslyn Compiler (تشخیص کلمه dynamic)
      ↓
Binding Phase (ایجاد گره‌های CallSite به جای Call معمولی)
      ↓
IL Generation + Metadata (تولید object + [DynamicAttribute] + کدهای DLR)
      ↓
CLR Execution (اجرای کد و استفاده از Runtime Binder)
```

---

## 4. مفاهیم Binding: Static vs Dynamic

### Static Binding چیست؟
کامپایلر دقیقاً می‌داند کدام متد باید فراخوانی شود.
```csharp
string name = "Ali";
name.ToUpper(); // کامپایلر آدرس متد String.ToUpper را مستقیماً در IL قرار می‌دهد.
```

### Dynamic Binding چیست؟
کامپایلر نمی‌داند نوع واقعی شیء در زمان اجرا چیست، بنابراین تصمیم‌گیری را به Runtime واگذار می‌کند.
```csharp
dynamic value = GetValue();
value.ToString(); // کامپایلر کدی تولید می‌کند که در Runtime نوع value را بررسی و متد را پیدا کند.
```

### جدول مقایسه Binding
| ویژگی | Static Binding | Dynamic Binding |
| :--- | :--- | :--- |
| **زمان Binding** | Compile Time | Runtime |
| **اطلاعات نوع** | کامل و موجود | ممکن است ناکافی باشد (تا زمان اجرا) |
| **تشخیص خطا** | Compile Time | Runtime (`RuntimeBinderException`) |
| **Performance** | بسیار سریع (مستقیم) | دارای سربار (Overhead) به دلیل Cache و Resolution |
| **انعطاف‌پذیری** | کمتر | بسیار زیاد |
| **پشتیبانی ابزار (Tooling)**| عالی (Refactoring, Go to Def) | محدود |

---

## 5. Runtime Binder و مدیریت خطاها

### Runtime Binder چیست؟
کامپوننتی در `Microsoft.CSharp.dll` است که مسئولیت حل‌وفصل (Resolution) فراخوانی‌های Dynamic را در زمان اجرا بر عهده دارد. این Binder نوع واقعی شیء را بررسی کرده، متد/پراپرتی مناسب را پیدا می‌کند و آن را اجرا می‌کند.

### RuntimeBinderException
اگر Runtime Binder نتواند عضوی را پیدا کند، به جای خطای کامپایل، یک استثنا در زمان اجرا پرتاب می‌کند:
```csharp
dynamic value = 10;
value.Hello(); // ✅ کامپایل می‌شود.
               // ❌ Runtime: Microsoft.CSharp.RuntimeBinder.RuntimeBinderException
```
* **تفاوت با Compile-Time Error:** خطای کامپایل مانع ساخت برنامه می‌شود. `RuntimeBinderException` یعنی برنامه ساخته شده، اما در حین اجرا با داده‌ای مواجه شده که انتظارش را نداشت.

---

## 6. Dynamic Expressions و Conversion

### Dynamic Propagation (گسترش پویایی)
اگر *هر یک* از عملوندها در یک عبارت (Expression) از نوع `dynamic` باشد، نتیجه کل عبارت نیز `dynamic` خواهد بود.
```csharp
dynamic x = 2;
var result = x * 3; // نوع استاتیک result در کامپایلر 'dynamic' است، نه 'int'.
```

### Cast کردن و Conversion
تبدیل `dynamic` به نوع دیگر در زمان اجرا (Runtime) بررسی می‌شود:
```csharp
dynamic value = 10;
int number = (int)value; // ✅ موفق (Implicit/Explicit conversion در Runtime)

dynamic str = "123";
// int num = str; // ❌ RuntimeBinderException: نمی‌تواند ضمنی تبدیل کند.
int num2 = (int)str; // ❌ RuntimeBinderException: Cast نامعتبر است.
int num3 = int.Parse((string)str); // ✅ موفق (ابتدا به string cast شده، سپس Parse)
```

---

## 7. تعامل Dynamic با سایر ویژگی‌های C#

### Method Overloading
در Static Binding، Overload Resolution در Compile Time بر اساس *نوع اعلام‌شده* انجام می‌شود. در Dynamic Binding، این کار در Runtime بر اساس *نوع واقعی* انجام می‌شود.
```csharp
void Print(int x) => Console.WriteLine("Int");
void Print(string x) => Console.WriteLine("String");

dynamic val = 10;
Print(val); // خروجی: "Int" (در Runtime تشخیص داده می‌شود)
```

### Polymorphism vs Dynamic Dispatch
* **Virtual Dispatch (Polymorphism):** کامپایلر می‌داند متد `virtual` است، اما پیاده‌سازی نهایی بر اساس نوع واقعی شیء در Runtime انتخاب می‌شود (از طریق vtable).
* **Dynamic Dispatch:** کامپایلر اصلاً نمی‌داند چه متدی وجود دارد. کل فرآیند جستجو در Runtime انجام می‌شود.

### Inheritance و Interface Members
Dynamic از وراثت پشتیبانی می‌کند، اما یک **محدودیت بزرگ** دارد: **Explicit Interface Implementation**.
```csharp
interface ITest { void Run(); }
class Test : ITest { void ITest.Run() { } }

dynamic x = new Test();
// x.Run(); // ❌ RuntimeBinderException: متد Run در سطح public کلاس Test دیده نمی‌شود.
((ITest)x).Run(); // ✅ راه‌حل: ابتدا به اینترفیس Cast کنید.
```

### Extension Methods
**مهم:** Extension Methodها با `dynamic` کار **نمی‌کنند**.
دلیل: Extension Methodها در واقع متدهای استاتیک هستند که کامپایلر در زمان کامپایل آن‌ها را به فراخوانی متد استاتیک تبدیل می‌کند. Runtime Binder به دنبال Extension Methodها در اسکوپ جاری نمی‌گردد.
*راه‌حل:* فراخوانی مستقیم متد استاتیک: `MyExtensions.MyMethod(x)`.

### Constructor و Indexing
```csharp
dynamic cap = 10;
var sb = new StringBuilder(cap); // ✅ کامپایلر کد Dynamic Invocation برای Constructor تولید می‌کند.

dynamic idx = 1;
int[] arr = [10, 20, 30];
var val = arr[idx]; // ✅ Dynamic Indexing پشتیبانی می‌شود.
```

---

## 8. Dynamic Language Runtime (DLR)

DLR لایه‌ای بالای CLR است که خدمات Dynamic Binding را برای زبان‌هایی مثل C#، IronPython و IronRuby فراهم می‌کند.

### معماری DLR
```text
Dynamic Object
      ↓
Call Site (محل فراخوانی در کد شما)
      ↓
Binder (مترجم قوانین زبان، مثلا CSharpBinder)
      ↓
Dynamic Meta Object (توصیف‌کننده رفتار شیء)
      ↓
Runtime Type & Target Method (اجرای نهایی)
```

### Call Site Caching (چرا Dynamic همیشه کند نیست؟)
DLR نتایج Binding را در `Call Site` کش (Cache) می‌کند. اگر یک فراخوانی Dynamic با *همان نوع* شیء تکرار شود، DLR به جای جستجوی مجدد، از Rule کش‌شده استفاده می‌کند که سرعت را به شدت افزایش می‌دهد.

---

## 9. پیاده‌سازی اشیای پویا: DynamicObject و IDynamicMetaObjectProvider

### DynamicObject (روش ساده)
کلاسی پایه که متدهای مجازی مانند `TryGetMember`، `TrySetMember`، `TryInvokeMember` را برای بازنویسی (Override) فراهم می‌کند.
```csharp
public class DynamicUser : DynamicObject
{
    private readonly Dictionary<string, object> _data = new();

    public override bool TryGetMember(GetMemberBinder binder, out object? result)
    {
        return _data.TryGetValue(binder.Name, out result);
    }

    public override bool TrySetMember(SetMemberBinder binder, object? value)
    {
        _data[binder.Name] = value;
        return true;
    }
}
// استفاده:
dynamic user = new DynamicUser();
user.Name = "Ali"; // TrySetMember فراخوانی می‌شود.
Console.WriteLine(user.Name); // TryGetMember فراخوانی می‌شود.
```

### IDynamicMetaObjectProvider (روش پیشرفته)
رابطی (Interface) که به کلاس اجازه می‌دهد کنترل کاملی بر فرآیند Binding از طریق تولید **Expression Tree** و تعریف **Binding Restrictions** داشته باشد. `DynamicObject` خودش این اینترفیس را پیاده‌سازی می‌کند.
* **DynamicMetaObject:** شیئی که توسط `GetMetaObject` برگردانده می‌شود و به DLR می‌گوید: "برای این عملیات خاص، این Expression Tree را اجرا کن، به شرطی که این Restrictions (مثلاً نوع شیء تغییر نکند) برقرار باشد."

### Custom Binding vs Language Binding
| ویژگی | Custom Binding | Language Binding |
| :--- | :--- | :--- |
| **پیاده‌سازی** | `IDynamicMetaObjectProvider` | ندارد (شیء معمولی) |
| **کنترل Binding** | توسط خود شیء (متا آبجکت) | توسط `CSharpRuntimeBinder` |
| **کاربرد** | ساخت DSL، ORMهای پویا، Proxyها | تعامل با اشیای ناشناخته خارجی |
| **پیچیدگی** | بسیار بالا (نیاز به Expression Trees) | پایین (شفاف برای توسعه‌دهنده) |

---

## 10. کاربردهای واقعی: COM، Reflection و JSON

### COM Interop (دلیل تاریخی اصلی)
قبل از `dynamic`، کار با Office Interop نیازمند پارامترهای متعدد `ref object` بود. `dynamic` این کد را تمیز کرد:
```csharp
// قدیمی: excelApp.Cells[1, 1].Value2 = "Test"; (با کست‌های زیاد)
// جدید:
dynamic excel = new Application();
excel.Cells[1, 1].Value = "Test"; // تمیز و خوانا
```

### Dynamic vs Reflection
| ویژگی | Reflection | Dynamic |
| :--- | :--- | :--- |
| **نحو (Syntax)** | پیچیده و پرحرف (`GetMethod`, `Invoke`) | ساده و طبیعی (`obj.Method()`) |
| **سرعت** | کند (مگر اینکه Cache شود) | کندتر از Static، اما سریع‌تر از Reflection خام (به لطف Cache) |
| **کاربرد** | بررسی Metadata، ساخت نمونه از Type ناشناخته | فراخوانی اعضا وقتی Type را می‌شناسیم اما در Compile Time در دسترس نیست |

### Dynamic و JSON
در .NET مدرن، استفاده از `dynamic` برای JSON کمتر توصیه می‌شود، اما همچنان ممکن است:
* **Newtonsoft.Json:** `dynamic obj = JsonConvert.DeserializeObject<dynamic>(json);`
* **.NET مدرن:** استفاده از `System.Text.Json.Nodes.JsonNode` یا `JsonDocument` ایمن‌تر و سریع‌تر از `dynamic` است، زیرا Type Safety نسبی را حفظ می‌کنند.

---

## 11. عملکرد (Performance) و IL

### Benchmark مفهومی (ترتیب سرعت از سریع به کند)
1. **Direct Call / Interface Call:** (سریع‌ترین، JIT کاملاً بهینه می‌کند)
2. **Cached Reflection (Delegate):** (بسیار نزدیک به Direct)
3. **Dynamic (با Cache فعال):** (سربار کم، مناسب برای اکثر سناریوها)
4. **Dynamic (اولین فراخوانی / Polymorphic شدید):** (سربار بالا به دلیل Cache Miss)
5. **Reflection (`MethodInfo.Invoke`):** (کندترین، سربار زیاد Boxing/Unboxing و Security Checks)

### IL و Metadata
وقتی کد `dynamic` را در SharpLab یا ILSpy بررسی می‌کنید، به جای `call`، چیزی شبیه به این می‌بینید:
```csharp
CallSite<Func<CallSite, object, object>>.Create(
    CSharpBinderFlags.None, 
    typeof(Program), 
    new CSharpArgumentInfo[] { ... }
)
```
این نشان می‌دهد که کامپایلر زیرساخت DLR را برای شما ساخته است.

---

## 12. معماری نرم‌افزار و اشتباهات رایج

### اشتباهات رایج
1. **استفاده به جای Generics:** اگر نوع داده در نهایت مشخص است، از `<T>` استفاده کنید، نه `dynamic`.
2. **استفاده در Core Domain:** منطق تجاری (Business Logic) باید کاملاً Type-Safe باشد. Dynamic فقط برای لایه‌های Integration (مثل خواندن فایل پیکربندی ناشناخته یا پاسخ API خارجی) مناسب است.
3. **نادیده گرفتن Extension Methods:** انتظار کار کردن Extension Methodها با Dynamic.
4. **استفاده در Public API:** قراردادهای عمومی باید شفاف و Strongly Typed باشند.

### Decision Guide (چه زمانی استفاده کنیم؟)
```text
آیا نوع داده در Compile Time مشخص است؟ 
   ↳ بله → از Strongly Typed استفاده کن.
آیا فقط نیاز به کد عمومی (Generic) برای انواع مختلف داری؟ 
   ↳ بله → از Generics `<T>` استفاده کن.
آیا نیاز به بررسی Metadata یا ساخت اشیای پویا در Runtime داری؟ 
   ↳ بله → از Reflection استفاده کن.
آیا با COM، IronPython، یا ساختارهای درختی ناشناخته (مثل JSON خام) سر و کار داری؟ 
   ↳ بله → از dynamic استفاده کن.
آیا می‌خواهی یک شیء سفارشی رفتار پویا (مثل دیکشنری) داشته باشد؟ 
   ↳ بله → از DynamicObject یا ExpandoObject استفاده کن.
```

---

## 13. جداول مقایسه‌ای جامع

### Dynamic vs Var
| ویژگی | `var` | `dynamic` |
| :--- | :--- | :--- |
| **زمان تعیین نوع** | Compile Time (Type Inference) | Runtime |
| **Type Safety** | کامل | ندارد (موکول به Runtime) |
| **IntelliSense** | دارد | ندارد |
| **تغییر مقدار** | فقط به نوع استنتاج‌شده | به هر نوعی می‌تواند تغییر کند |

### Dynamic vs Source Generator
| ویژگی | Dynamic | Source Generator |
| :--- | :--- | :--- |
| **هزینه Runtime** | دارد (DLR Overhead) | صفر (کد در Compile Time تولید می‌شود) |
| **Type Safety** | ندارد | کامل |
| **انعطاف‌پذیری** | بسیار بالا (تغییر در Runtime) | محدود به آنچه در Compile Time قابل محاسبه است |
| **نتیجه** | کد پویا | کد استاتیک تولیدشده |

---

## 14. پروژه‌های عملی

### پروژه ۱: Dynamic Configuration Object
```csharp
public class Config : DynamicObject
{
    private readonly Dictionary<string, object> _sections = new();
    public override bool TryGetMember(GetMemberBinder binder, out object? result)
    {
        if (!_sections.ContainsKey(binder.Name)) _sections[binder.Name] = new Config();
        result = _sections[binder.Name];
        return true;
    }
    public override bool TrySetMember(SetMemberBinder binder, object? value)
    {
        _sections[binder.Name] = value;
        return true;
    }
}
// استفاده:
dynamic config = new Config();
config.Database.ConnectionString = "Server=...";
```

### پروژه ۲: Dynamic Command Dispatcher
```csharp
public class CommandDispatcher : DynamicObject
{
    private readonly Dictionary<string, Action<object[]>> _handlers = new();

    public void Register(string command, Action<object[]> handler) => _handlers[command] = handler;

    public override bool TryInvokeMember(InvokeMemberBinder binder, object?[]? args, out object? result)
    {
        if (_handlers.TryGetValue(binder.Name, out var handler))
        {
            handler(args ?? Array.Empty<object>());
            result = null;
            return true;
        }
        result = null;
        return false; // باعث پرتاب RuntimeBinderException می‌شود
    }
}
// استفاده:
dynamic dispatcher = new CommandDispatcher();
dispatcher.Register("CreateUser", args => Console.WriteLine($"Created {args[0]}"));
dispatcher.CreateUser("Ali");
```

---

## 15. Testing، Debugging و Cheat Sheet

### Unit Testing برای Dynamic
برای تست `RuntimeBinderException` در xUnit:
```csharp
[Fact]
public void Dynamic_ShouldThrow_WhenMemberMissing()
{
    dynamic obj = new ExpandoObject();
    Assert.Throws<RuntimeBinderException>(() => { var x = obj.NonExistent; });
}
```

### Debugging
در Visual Studio، وقتی روی متغیر `dynamic` هاور می‌کنید، ممکن است نوع آن را `object` نشان دهد. برای دیدن نوع واقعی، از پنجره **Watch** یا **Immediate Window** استفاده کنید و عبارت `obj.GetType()` را ارزیابی کنید.

### Cheat Sheet
```csharp
dynamic x = 10;             // تعریف پویا
var y = 10;                 // استنتاج نوع (ایستا)
object z = 10;              // نوع پایه (نیاز به Cast دارد)

x.Prop = 1;                 // TrySetMember (اگر DynamicObject باشد) یا Runtime Binding
var a = x.Prop;             // TryGetMember یا Runtime Binding

x.Method(1, 2);             // TryInvokeMember یا Runtime Binding
var b = x[0];               // TryGetIndex یا Runtime Index Binding

// کلمات کلیدی مرتبط:
// DynamicObject, ExpandoObject, IDynamicMetaObjectProvider
// RuntimeBinderException, CallSite, Binder, DLR
```

---

## 16. جمع‌بندی نهایی

1. **Dynamic چیست؟** نوعی استاتیک در C# که بررسی عضو را به Runtime موکول می‌کند.
2. **چرا وجود دارد؟** برای ساده‌سازی Interop (مثل COM)، کار با زبان‌های پویا، و کاهش کدهای تکراری Reflection.
3. **آیا CLR آن را می‌شناسد؟** خیر، CLR آن را به عنوان `System.Object` همراه با `[DynamicAttribute]` می‌بیند.
4. **Static Binding چیست؟** تصمیم‌گیری کامپایلر درباره متد/پراپرتی در زمان ساخت برنامه.
5. **Dynamic Binding چیست؟** تصمیم‌گیری Runtime Binder درباره متد/پراپرتی در زمان اجرا.
6. **DLR چیست؟** موتور اجرایی بالای CLR که خدمات Binding، Call Site Caching و Meta Object را فراهم می‌کند.
7. **Runtime Binder چیست؟** کامپوننتی در `Microsoft.CSharp.dll` که قوانین زبان C# را در Runtime برای اشیای پویا اعمال می‌کند.
8. **DynamicObject چیست؟** کلاس پایه‌ای برای ساخت اشیای پویا با بازنویسی متدهای `Try...`.
9. **IDynamicMetaObjectProvider چیست؟** اینترفیس پیشرفته برای کنترل کامل فرآیند Binding از طریق Expression Tree.
10. **هزینه Performance:** سربار دارد، اما به لطف Call Site Caching، در فراخوانی‌های تکراری قابل قبول است. هرگز در Hot Pathهای حساس به عملکرد استفاده نشود.
11. **چه زمانی خوب است؟** لایه‌های Integration، کار با JSON ناشناخته، COM Interop، ساخت DSLهای داخلی.
12. **چه زمانی بد است؟** منطق تجاری (Core Domain)، قراردادهای Public API، جایگزینی برای Generics یا Interfaceهای مشخص.

---

## منابع معتبر

برای مطالعه عمیق‌تر، فقط به منابع رسمی و مستندات زبان اعتماد کنید:

1. **Microsoft Learn: dynamic (C# Reference)**
   * موضوع: مرجع رسمی زبان برای کلمه کلیدی dynamic.
   * لینک: [docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types#the-dynamic-type](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types#the-dynamic-type)
2. **C# Language Specification (Dynamic Binding)**
   * موضوع: مشخصات فنی نحوه کارکرد Dynamic Binding در استاندارد زبان.
   * لینک: [docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#dynamic-binding](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#dynamic-binding)
3. **System.Dynamic Namespace Documentation**
   * موضوع: مستندات کلاس‌های `DynamicObject`، `ExpandoObject` و `IDynamicMetaObjectProvider`.
   * لینک: [docs.microsoft.com/en-us/dotnet/api/system.dynamic](https://learn.microsoft.com/en-us/dotnet/api/system.dynamic)
4. **Dynamic Language Runtime (DLR) Overview**
   * موضوع: معماری و مفاهیم پایه DLR، Call Site و Binder.
   * لینک: [github.com/dotnet/runtime/blob/main/docs/design/features/dynamic-language-runtime.md](https://github.com/dotnet/runtime/blob/main/docs/design/features/dynamic-language-runtime.md)
5. **Microsoft.CSharp.RuntimeBinder Namespace**
   * موضوع: مستندات `RuntimeBinderException` و کلاس‌های Binder.
   * لینک: [docs.microsoft.com/en-us/dotnet/api/microsoft.csharp.runtimebinder](https://learn.microsoft.com/en-us/dotnet/api/microsoft.csharp.runtimebinder)

> **تذکر:** این سند بر اساس استانداردهای C# تا نسخه‌های مدرن (.NET 8/9) تدوین شده است. رفتار `dynamic` در این نسخه‌ها پایدار بوده و تغییر بنیادینی نسبت به معرفی آن در C# 4.0 نداشته است، به جز بهبودهای عملکردی در DLR و یکپارچگی بهتر با Nullable Reference Types.