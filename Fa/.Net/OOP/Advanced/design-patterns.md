

## 📚 فهرست مطالب

1. [مقدمه: طراحی مبتنی بر اصول SOLID و الگوهای طراحی](#مقدمه-طراحی-مبتنی-بر-اصول-solid-و-الگوهای-طراحی)  
2. [الگوی Factory (کارخانه)](#الگوی-factory-کارخانه)  
3. [الگوی Singleton (تک‌شی)](#الگوی-singleton-تکشی)  
4. [الگوی Strategy (استراتژی)](#الگوی-strategy-استراتژی)  
5. [الگوی Observer (مشاهده‌گر)](#الگوی-observer-مشاهدهگر)  
6. [الگوی Adapter (سازگارکننده)](#الگوی-adapter-سازگارکننده)  
7. [الگوی Decorator (آرایش‌دهنده)](#الگوی-decorator-آرایشدهنده)  
8. [جمع‌بندی و توصیه‌های کاربردی](#جمعبندی-و-توصیههای-کاربردی)  
9. [منابع معتبر](#منابع-معتبر)

---

## مقدمه: طراحی مبتنی بر اصول SOLID و الگوهای طراحی

**الگوهای طراحی (Design Patterns)** راه‌حل‌های اثبات‌شده‌ای برای مشکلات رایج در طراحی نرم‌افزار هستند. این الگوها **کد را انعطاف‌پذیرتر، قابل نگهداری‌تر و قابل تست‌تر** می‌کنند و بر پایه **اصول SOLID** و دیگر **اصول طراحی نرم‌افزار** (مانند *Encapsulation*, *Composition over Inheritance*) ساخته شده‌اند.

در این راهنما، شش الگوی پرکاربرد را در زبان **C#** بررسی می‌کنیم، همراه با مثال‌های ساده، مفاهیم کلیدی و بهترین شیوه‌های استفاده.

---

## الگوی Factory (کارخانه)

### 🔍 هدف
**ایجاد اشیاء بدون مشخص کردن کلاس دقیق آن‌ها در زمان کامپایل.**

### 🧠 اصل طراحی پشت آن
- **اصول SOLID**:  
  - **Open/Closed Principle**: سیستم به گسترش باز، به تغییر بسته است.  
  - **Dependency Inversion**: وابستگی به اینترفیس یا کلاس انتزاعی، نه پیاده‌سازی ملموس.

### 💡 مثال ساده در C#

```csharp
public interface IShape
{
    void Draw();
}

public class Circle : IShape
{
    public void Draw() => Console.WriteLine("Drawing Circle");
}

public class Rectangle : IShape
{
    public void Draw() => Console.WriteLine("Drawing Rectangle");
}

public class ShapeFactory
{
    public IShape CreateShape(string type)
    {
        return type.ToLower() switch
        {
            "circle" => new Circle(),
            "rectangle" => new Rectangle(),
            _ => throw new ArgumentException("Unknown shape type")
        };
    }
}
```

### ✅ نکات کاربردی
- برای **جایگزینی `new` در کد‌های پیچیده** و کاهش وابستگی استفاده شود.
- در ترکیب با **Dependency Injection** بسیار قدرتمند است.

---

## الگوی Singleton (تک‌شی)

### 🔍 هدف
**اطمینان از اینکه فقط یک نمونه (instance) از یک کلاس در سراسر برنامه وجود دارد.**

### 🧠 اصل طراحی پشت آن
- **Encapsulation** و **Global Access Point** با کنترل شده.

### 💡 مثال ساده در C#

```csharp
public sealed class Logger
{
    private static readonly Lazy<Logger> _instance = new Lazy<Logger>(() => new Logger());

    private Logger() { }

    public static Logger Instance => _instance.Value;

    public void Log(string message) => Console.WriteLine($"[LOG] {message}");
}
```

> ⚠️ **هشدار**: از Singleton **به دلیل تست‌ناپذیری و وابستگی جهانی** تنها در موارد ضروری (مانند کش‌های سراسری، لاگر، تنظیمات ثابت) استفاده کنید.

---

## الگوی Strategy (استراتژی)

### 🔍 هدف
**تغییر رفتار یک کلاس در زمان اجرا (runtime)** با جایگزینی الگوریتم‌های مختلف.

### 🧠 اصل طراحی پشت آن
- **Open/Closed Principle**  
- **Composition over Inheritance**

### 💡 مثال ساده در C#

```csharp
public interface ISortStrategy
{
    void Sort(List<int> list);
}

public class BubbleSort : ISortStrategy
{
    public void Sort(List<int> list) => Console.WriteLine("Bubble sort used");
}

public class QuickSort : ISortStrategy
{
    public void Sort(List<int> list) => Console.WriteLine("Quick sort used");
}

public class Sorter
{
    private ISortStrategy _strategy;

    public Sorter(ISortStrategy strategy) => _strategy = strategy;

    public void SetStrategy(ISortStrategy strategy) => _strategy = strategy;

    public void Sort(List<int> list) => _strategy.Sort(list);
}
```

### ✅ نکات کاربردی
- برای **تغییر رفتار بدون تغییر کد مرکزی** ایده‌آل است.
- در C# اغلب با **Func<> یا Action<>** یا **LINQ** نیز جایگزین می‌شود.

---

## الگوی Observer (مشاهده‌گر)

### 🔍 هدف
**اطلاع‌رسانی خودکار به یک یا چند شیء (observer) زمانی که وضعیت یک شیء (subject) تغییر می‌کند.**

### 🧠 اصل طراحی پشت آن
- **Loose Coupling**  
- **Depend upon abstractions**

### 💡 مثال ساده در C# (با استفاده از `IObservable<T>` یا ساده‌تر با Event)

```csharp
public class NewsAgency
{
    public event Action<string> NewsPublished;

    public void PublishNews(string news)
    {
        Console.WriteLine($"Agency: {news}");
        NewsPublished?.Invoke(news);
    }
}

public class NewsChannel
{
    public string Name { get; set; }

    public NewsChannel(string name, NewsAgency agency)
    {
        Name = name;
        agency.NewsPublished += OnNewsReceived;
    }

    private void OnNewsReceived(string news)
    {
        Console.WriteLine($"{Name} received: {news}");
    }
}
```

> 📌 در C# معمولاً از **Event/Delegate** یا **Reactive Extensions (Rx.NET)** برای پیاده‌سازی Observer استفاده می‌شود.

---

## الگوی Adapter (سازگارکننده)

### 🔍 هدف
**اجازه کار با کلاس‌هایی که رابط (interface) سازگاری با آنچه نیاز داریم ندارند.**

### 🧠 اصل طراحی پشت آن
- **Single Responsibility Principle**  
- **Interface Segregation**

### 💡 مثال ساده در C#

```csharp
// Legacy system
public class LegacyPrinter
{
    public void PrintDocument(string content) => 
        Console.WriteLine($"Legacy printer: {content}");
}

// Target interface
public interface IPrinter
{
    void Print(string content);
}

// Adapter
public class LegacyPrinterAdapter : IPrinter
{
    private readonly LegacyPrinter _printer;

    public LegacyPrinterAdapter(LegacyPrinter printer) => _printer = printer;

    public void Print(string content) => _printer.PrintDocument(content);
}
```

### ✅ نکات کاربردی
- زمانی کاربرد دارد که **با سیستم‌های خارجی یا قدیمی کار می‌کنید**.

---

## الگوی Decorator (آرایش‌دهنده)

### 🔍 هدف
**افزودن رفتار یا ویژگی جدید به یک شیء بدون تغییر کد آن و بدون استفاده از ارث‌بری.**

### 🧠 اصل طراحی پشت آن
- **Open/Closed Principle**  
- **Composition over Inheritance**

### 💡 مثال ساده در C#

```csharp
public interface IComponent
{
    string Operation();
}

public class ConcreteComponent : IComponent
{
    public string Operation() => "Base Component";
}

public abstract class Decorator : IComponent
{
    protected IComponent _component;

    public Decorator(IComponent component) => _component = component;

    public virtual string Operation() => _component.Operation();
}

public class BoldDecorator : Decorator
{
    public BoldDecorator(IComponent component) : base(component) { }

    public override string Operation() => $"<b>{base.Operation()}</b>";
}
```

### ✅ نکات کاربردی
- در **ASP.NET Core Middleware** و **Stream Wrappers** (مثل `BufferedStream`) پیاده‌سازی شده است.

---

## جمع‌بندی و توصیه‌های کاربردی

| الگو | زمان استفاده مناسب | خطرات |
|------|------------------|--------|
| **Factory** | وقتی نیاز به ایجاد اشیاء پیچیده بدون وابستگی به کلاس‌های ملموس دارید | سوءاستفاده: کارخانه‌های بی‌هدف |
| **Singleton** | برای شیء‌هایی که واقعاً باید یکتایی داشته باشند (مثل تنظیمات سراسری) | تست‌ناپذیری، state sharing |
| **Strategy** | وقتی الگوریتم‌ها قابل تعویض هستند | اضافه‌کردن کلاس‌های بیش از حد |
| **Observer** | زمانی که باید چندین شیء به تغییرات یک شیء واکنش نشان دهند | Memory leak اگر event unsubscribe نشود |
| **Adapter** | ادغام با API یا کتابخانه‌های غیرسازگار | پیچیدگی اضافه در لایه‌بندی |
| **Decorator** | وقتی می‌خواهید ویژگی‌های قابل ترکیب داشته باشید | عمیق‌شدن پشته فراخوانی (stack overflow) |

> ✅ **نکته طلایی**: الگوها **ارجاع‌نامه برای حل مسئله‌اند**، نه چک لیستی که باید همیشه اعمال شوند.

---

## منابع معتبر

1. **GoF (Gang of Four)** – *Design Patterns: Elements of Reusable Object-Oriented Software* (1994)  
   📘 کتاب مرجع تمام الگوهای کلاسیک  
   [https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)

2. **Microsoft Learn – C# Design Patterns**  
   🌐 مستندات رسمی مایکروسافت  
   [https://learn.microsoft.com/en-us/dotnet/architecture/patterns](https://learn.microsoft.com/en-us/dotnet/architecture/patterns)

3. **Refactoring.Guru – Design Patterns in C#**  
   ✅ مثال‌های بصری + کد C# + توضیحات ساده  
   [https://refactoring.guru/design-patterns/csharp](https://refactoring.guru/design-patterns/csharp)

4. **Head First Design Patterns (2nd Edition)** – O’Reilly  
   📚 مناسب مبتدی‌ها با رویکرد یادگیری بصری  
   [https://www.oreilly.com/library/view/head-first-design/9781492078005/](https://www.oreilly.com/library/view/head-first-design/9781492078005/)

5. **C# in Depth (Jon Skeet)** – برای درک عمیق‌تر مفاهیم مانند `Lazy<T>` در Singleton  
   [https://csharpindepth.com/](https://csharpindepth.com/)
*
