# برابری (Equality) در Recordهای دارای ارث‌بری (Inheritance) در C#

رکوردها (Records) در C# 9 معرفی شدند تا توسعه‌دهندگان بتوانند به سادگی اشیایی با **برابری مبتنی بر مقدار (Value-based Equality)** ایجاد کنند. اما وقتی پای ارث‌بری (Inheritance) به میان می‌آید، مقوله برابری پیچیده‌تر می‌شود. 

در این آموزش پیشرفته، بررسی می‌کنیم که کامپایلر C# چگونه برابری را در سلسله‌مراتب رکوردها مدیریت می‌کند و چرا مفاهیمی مثل `EqualityContract` و `Runtime Type` برای حفظ منطق برابری حیاتی هستند.

---

## ۱. مثال پایه (Base و Derived)

برای درک بهتر، از مثال کلاسیک `Person` و `Employee` استفاده می‌کنیم:

```csharp
public record Person(string Name);

public record Employee(string Name, int EmployeeId) : Person(Name);
```

در اینجا:
*   **Base Record:** `Person` (دارای یک Property به نام `Name`)
*   **Derived Record:** `Employee` (دارای `Name` از کلاس پایه و `EmployeeId` اختصاصی)

---

## ۲. سناریوهای مقایسه (رفتارها)

بیایید سه سناریوی اصلی را بررسی کنیم تا رفتار رکوردها را درک کنیم:

### الف) مقایسه Person با Person
```csharp
var p1 = new Person("Ali");
var p2 = new Person("Ali");
Console.WriteLine(p1 == p2); // True
```
**رفتار:** کامپایلر ابتدا `EqualityContract` هر دو را چک می‌کند (هر دو `typeof(Person)` هستند). سپس Propertyهای پایه (`Name`) را مقایسه می‌کند. چون مقادیر برابرند، نتیجه `true` است.

### ب) مقایسه Employee با Employee
```csharp
var e1 = new Employee("Ali", 101);
var e2 = new Employee("Ali", 101);
Console.WriteLine(e1 == e2); // True
```
**رفتار:** `EqualityContract` هر دو `typeof(Employee)` است. سپس متد `Equals` کلاس پایه (`Person`) فراخوانی می‌شود تا `Name` مقایسه شود. در نهایت، Propertyهای اختصاصی Derived (`EmployeeId`) مقایسه می‌شوند. چون همه چیز برابر است، نتیجه `true` است.

### ج) مقایسه Person با Employee (چالش اصلی)
```csharp
var p = new Person("Ali");
var e = new Employee("Ali", 101);
Console.WriteLine(p == e); // False
Console.WriteLine(e == p); // False
```
**رفتار:** با اینکه `Name` در هر دو یکسان است، **نتیجه قطعاً `false` است.** 
چرا؟ چون `Runtime Type` (و در نتیجه `EqualityContract`) آن‌ها متفاوت است. یک رکورد پایه هرگز نمی‌تواند با یک رکورد مشتق‌شده برابر باشد، حتی اگر تمام Propertyهای مشترک آن‌ها یکسان باشد.

---

## ۳. مفاهیم کلیدی: EqualityContract و Runtime Type

### چرا Runtime Type مهم است؟
در ریاضیات و برنامه‌نویسی، برابری باید **متقارن (Symmetric)** باشد. یعنی اگر `A == B` باشد، حتماً باید `B == A` نیز باشد.
اگر `Person("Ali")` با `Employee("Ali", 101)` برابر بود، باید برعکس آن هم صادق می‌بود. اما `Person` اصلاً از وجود `EmployeeId` خبر ندارد! این عدم تقارن باعث باگ‌های وحشتناک در Collections (مثل Dictionary یا HashSet) می‌شد. 
برای حل این مشکل، C# از **Runtime Type** استفاده می‌کند تا مطمئن شود دو شیء دقیقاً از یک "نوع قراردادی" هستند.

### نقش EqualityContract چیست؟
`EqualityContract` یک Property از نوع `protected virtual Type` است که به صورت خودکار توسط کامپایلر تولید می‌شود. این Property، `Runtime Type` دقیق‌ترین نوع رکورد را برمی‌گرداند.
*   در `Person`، مقدار آن `typeof(Person)` است.
*   در `Employee`، مقدار آن `typeof(Employee)` است (چون `override` شده است).

هنگام مقایسه، **اولین و مهم‌ترین قدم**، مقایسه `EqualityContract` دو شیء است. اگر این دو متفاوت باشند، مقایسه بلافاصله `false` برمی‌گرداند و کاری به Propertyها ندارد.

---

## ۴. زیر کاپوت: رفتار کامپایلر (Compiler Behavior)

بیایید ببینیم کامپایلر C# دقیقاً چه کدهایی را برای این سلسله‌مراتب تولید می‌کند (بر اساس C# Language Specification).

### الف) تولید کد برای Base Record (Person)
```csharp
public record Person : IEquatable<Person>
{
    public string Name { get; init; }

    // 1. تولید EqualityContract
    protected virtual Type EqualityContract => typeof(Person);

    // 2. تولید متد Equals تخصصی
    public virtual bool Equals(Person? other)
    {
        // چک کردن null و سپس بررسی قرارداد برابری
        return other is not null &&
               EqualityContract == other.EqualityContract &&
               Name == other.Name; // مقایسه Propertyهای Base
    }

    // 3. بازنویسی Equals عمومی (برای object)
    public override bool Equals(object? obj) => Equals(obj as Person);

    // 4. تولید GetHashCode
    public override int GetHashCode()
    {
        return HashCode.Combine(EqualityContract, Name);
    }
}
```

### ب) تولید کد برای Derived Record (Employee)
```csharp
public record Employee : Person, IEquatable<Employee>
{
    public int EmployeeId { get; init; }

    // 1. Override کردن EqualityContract با Runtime Type خود
    protected override Type EqualityContract => typeof(Employee);

    // 2. تولید متد Equals تخصصی
    public virtual bool Equals(Employee? other)
    {
        // ابتدا فراخوانی Equals کلاس پایه (که EqualityContract و Name را چک می‌کند)
        // سپس مقایسه Propertyهای اختصاصی Derived
        return base.Equals(other) && 
               EmployeeId == other.EmployeeId; 
    }

    // 3. بازنویسی Equals عمومی
    public override bool Equals(object? obj) => Equals(obj as Employee);

    // 4. تولید GetHashCode با ترکیب Hash کلاس پایه و Propertyهای جدید
    public override int GetHashCode()
    {
        return HashCode.Combine(base.GetHashCode(), EmployeeId);
    }
}
```

---

## ۵. تحلیل GetHashCode در سلسله‌مراتب

همانطور که در کد تولید شده توسط کامپایلر می‌بینید، `GetHashCode` نیز از الگوی ارث‌بری پیروی می‌کند:

1.  **در Base (`Person`):** هش کد شامل `EqualityContract` و `Name` است.
2.  **در Derived (`Employee`):** هش کد با فراخوانی `base.GetHashCode()` (که شامل قرارداد و Name است) شروع شده و سپس `EmployeeId` به آن ترکیب (Combine) می‌شود.

**نکته مهم:** چون `EqualityContract` در `GetHashCode` لحاظ می‌شود، حتی اگر دو شیء از نظر Propertyها کاملاً یکسان باشند اما `Runtime Type` متفاوتی داشته باشند، `GetHashCode` آن‌ها نیز متفاوت خواهد بود. این موضوع برای کارکرد صحیح `HashSet` و `Dictionary` حیاتی است.

---

## ۶. مقایسه Propertyها در سلسله‌مراتب (خلاصه جریان)

وقتی شما `e1.Equals(e2)` را صدا می‌زنید، این جریان طی می‌شود:
1.  **چک کردن `EqualityContract`:** آیا `typeof(Employee) == typeof(Employee)`؟ (بله)
2.  **فراخوانی `base.Equals`:** متد `Equals` کلاس `Person` صدا زده می‌شود.
3.  **چک کردن مجدد `EqualityContract` در Base:** (چون هر دو شیء در واقع `Employee` هستند، `EqualityContract` آن‌ها در این مرحله نیز `Employee` است و با هم برابرند).
4.  **مقایسه Propertyهای Base:** آیا `Name` برابر است؟
5.  **بازگشت به Derived:** آیا `EmployeeId` برابر است؟

اگر تمام این مراحل `true` باشند، دو شیء برابرند.

---

## ۷. نکات تکمیلی و Best Practices

*   **تغییر دستی EqualityContract:** شما *نباید* `EqualityContract` را به صورت دستی تغییر دهید. این Property صرفاً برای استفاده داخلی کامپایلر و موتور برابری رکوردهاست.
*   **استفاده از `with` expression:** وقتی از `with` برای کپی کردن یک Derived Record استفاده می‌کنید، `EqualityContract` و `Runtime Type` به درستی حفظ می‌شوند و یک `Employee` جدید، همچنان یک `Employee` باقی می‌ماند.
*   **Record struct:** توجه داشته باشید که `record struct`ها از ارث‌بری (Inheritance) پشتیبانی نمی‌کنند، بنابراین چالش‌های `EqualityContract` و `Runtime Type` منحصراً مربوط به `record class` (یا همان `record` ساده) هستند.

---

## منابع برای مطالعه بیشتر

برای عمیق‌تر شدن در این مبحث، منابع رسمی زیر پیشنهاد می‌شوند:
1.  **Microsoft Learn:** مستندات رسمی [Record Types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) و بخش [Equality in records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#built-in-format-for-value-based-equality).
2.  **C# Language Specification:** بخش [Record Equality](https://github.com/dotnet/csharplang/blob/main/spec/records.md#equality) در ریپازیتوری رسمی `csharplang` که دقیقاً الگوریتم تولید `Equals` و `GetHashCode` توسط کامپایلر را تشریح کرده است.
3.  **C# Language Design Notes:** جلسات طراحی زبان C# (به ویژه جلسات مربوط به C# 9) که در آن‌ها تیم طراحی درباره اهمیت تقارن برابری (Symmetry of Equality) و معرفی `EqualityContract` بحث کرده‌اند.