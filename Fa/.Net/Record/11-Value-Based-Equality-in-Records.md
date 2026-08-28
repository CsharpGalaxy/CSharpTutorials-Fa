# آموزش کامل Immutability در Recordهای سی‌شارپ (C#)

یکی از جذاب‌ترین ویژگی‌هایی که در سی‌شارپ ۹ معرفی شد، **Record**ها بودند. هدف اصلی آن‌ها ساده‌سازی مدل‌های داده‌ای و ایجاد آبجکت‌های تغییرناپذیر (Immutable) بود. اما یک باور غلط و بسیار رایج بین برنامه‌نویسان وجود دارد:
> *"چون از Record استفاده کرده‌ام، پس آبجکت من ۱۰۰٪ تغییرناپذیر (Immutable) است."*

**این جمله کاملاً اشتباه است!** در این آموزش، به صورت عمیق و با مثال بررسی می‌کنیم که Immutability در Recordها دقیقاً چگونه کار می‌کند و مرزهای آن کجاست.

---

## ۱. Immutable Object (آبجکت تغییرناپذیر) چیست؟
یک آبجکت Immutable، آبجکتی است که **پس از ساخته شدن (Instantiation)، وضعیت (State) یا داده‌های درونی آن به هیچ وجه قابل تغییر نیستند.** اگر بخواهید تغییری در آن ایجاد کنید، باید یک آبجکت کاملاً جدید با مقادیر جدید بسازید.
رشته‌ها (`string`) در سی‌شارپ معروف‌ترین مثال برای Immutable بودن هستند.

## ۲. چرا Immutability مهم است؟
* **امنیت در برنامه‌نویسی چندنخی (Thread-Safety):** آبجکت‌های Immutable ذاتاً Thread-Safe هستند. چون هیچ Threadای نمی‌تواند داده‌های آن را تغییر دهد، نیازی به Lock کردن (قفل کردن) آن‌ها نیست.
* **قابل پیش‌بینی بودن (Predictability):** وقتی یک آبجکت را به یک متد پاس می‌دهید، مطمئن هستید که آن متد نمی‌تواند داده‌های اصلی شما را خراب کند.
* **امنیت در کلیدهای Dictionary:** برای استفاده از یک آبجکت به عنوان کلید در `Dictionary` یا `HashSet`، مقدار `HashCode` آن باید ثابت بماند. آبجکت‌های Mutable این تضمین را نمی‌دهند.

---

## ۳. نقش کلمه کلیدی `init`
در سی‌شارپ ۹، مایکروسافت اکسسور جدیدی به نام `init` را معرفی کرد. این اکسسور به شما اجازه می‌دهد مقدار یک Property را **فقط در زمان ساخت آبجکت** (در Constructor یا Object Initializer) تنظیم کنید. پس از ساخته شدن آبجکت، آن Property دیگر `set` ندارد و قابل تغییر نیست.

```csharp
public class Person
{
    public string Name { get; init; } // فقط در زمان ساخت قابل تنظیم است
    public int Age { get; set; }      // همیشه قابل تغییر است
}

var p = new Person { Name = "Ali", Age = 20 };
// p.Name = "Reza"; // ❌ Error: نام قابل تغییر نیست
p.Age = 25;         // ✅ OK: سن قابل تغییر است
```

---

## ۴. آیا Record همیشه Immutable است؟
**خیر!** Record بودن به معنی Immutable بودن کامل Object نیست.
وقتی شما یک `record` تعریف می‌کنید، کامپایلر سی‌شارپ به صورت پیش‌فرض برای تمام Propertyهای شما اکسسور `init` تولید می‌کند. این یعنی **Referenceها (ارجاعات)** تغییرناپذیر هستند، اما **محتوای داخلی آن‌ها** لزوماً اینطور نیست.

---

## ۵. Record Class و Immutability
وقتی از `record` (یا به طور دقیق‌تر `record class`) استفاده می‌کنید، کامپایلر Propertyها را با `init` می‌سازد:

```csharp
public record User(string Username, string Email);

var user = new User("admin", "admin@test.com");
// user.Username = "root"; // ❌ Error: چون اکسسور init است
```
در اینجا `user.Username` تغییرناپذیر است. اما آیا کل آبجکت `user` تغییرناپذیر است؟ در بخش‌های بعدی می‌بینیم که خیر.

---

## ۶. Record Struct و Mutable Property
داستان در مورد `record struct` (معرفی شده در C# 10) کاملاً متفاوت است!
چون Structها نوع ارزشی (Value Type) هستند و هنگام پاس دادن کپی می‌شوند، کامپایلر سی‌شارپ برای Propertyهای `record struct` به صورت پیش‌فرض اکسسور **`get` و `set`** تولید می‌کند.

```csharp
public record struct Point(int X, int Y);

var p = new Point(10, 20);
p.X = 50; // ✅ کاملاً مجاز است! چون record struct به صورت پیش‌فرض Mutable است.
```

---

## ۷. readonly record struct
اگر می‌خواهید یک `record struct` دقیقاً مثل `record class` رفتار کند و Immutable باشد، باید از کلمه کلیدی `readonly` استفاده کنید:

```csharp
public readonly record struct Point(int X, int Y);

var p = new Point(10, 20);
// p.X = 50; // ❌ Error: چون readonly struct است، تمام اعضا تغییرناپذیر می‌شوند.
```

---

## ۸. Mutable Property در Record (شکستن Immutability)
شما می‌توانید داخل یک `record class` به صورت دستی Propertyهایی با `set` تعریف کنید تا Immutability را بشکنید:

```csharp
public record Person(string FirstName)
{
    // تعریف دستی یک Property تغییرپذیر داخل Record
    public string LastName { get; set; } 
}

var person = new Person("Ali");
// person.FirstName = "Reza"; // ❌ Error
person.LastName = "Alavi";    // ✅ OK: چون خودمان set گذاشتیم
```

---

## ۹. Mutable Collection داخل Record (خطر بزرگ!)
این مهم‌ترین بخش آموزش است. فرض کنید یک Record دارید که یک `List` یا `Array` درون خود دارد:

```csharp
public record Team(string Name, List<string> Members);

var team = new Team("Dev", new List<string> { "Ali", "Sara" });

// team.Members = new List<string>(); // ❌ Error: نمی‌توانید Reference لیست را عوض کنید.

// اما...
team.Members.Add("Reza");            // ✅ مجاز است!
team.Members[0] = "Omid";            // ✅ مجاز است!
team.Members.Clear();                // ✅ مجاز است!
```
**نتیجه:** شما نمی‌توانید `Members` را به یک لیست *جدید* اشاره دهید (چون `init` است)، اما می‌توانید **محتویات** لیست را هرطور که بخواهید تغییر دهید!

---

## ۱۰. Reference Typeهای داخلی (Nested Objects)
اگر Record شما به یک کلاس معمولی (Reference Type) اشاره کند، آن کلاس داخلی کاملاً Mutable خواهد بود:

```csharp
public class Address
{
    public string City { get; set; }
}

public record Customer(string Name, Address Address);

var customer = new Customer("Ali", new Address { City = "Tehran" });

// customer.Address = new Address(); // ❌ Error: نمی‌توان آدرس را عوض کرد.
customer.Address.City = "Shiraz";    // ✅ مجاز است! شهر داخل آدرس تغییر کرد.
```

---

## ۱۱. Shallow Immutability در برابر Deep Immutability
* **Shallow Immutability (تغییرناپذیری سطحی):** رفتار پیش‌فرض Recordهاست. فقط Propertyهای سطح اول (Top-level) تغییرناپذیرند (Referenceها ثابتند). اما اگر آن Referenceها به آبجکت‌های Mutable (مثل List یا کلاس‌های معمولی) اشاره کنند، آن‌ها قابل تغییرند.
* **Deep Immutability (تغییرناپذیری عمیق):** یعنی خود آبجکت و **هر چیزی که در درون آن و درون درون آن** قرار دارد، تغییرناپذیر باشد.

**چگونه Deep Immutability ایجاد کنیم؟**
باید از کالکشن‌های Immutable مایکروسافت (`System.Collections.Immutable`) و Recordهای داخلیِ Readonly استفاده کنید:

```csharp
using System.Collections.Immutable;

public record ImmutableTeam(string Name, ImmutableList<string> Members);

var team = new ImmutableTeam("Dev", ImmutableList.Create("Ali", "Sara"));
// team.Members.Add("Reza"); // ❌ Error: ImmutableList متد Add ندارد!
// باید از روش‌های Immutable استفاده کرد که یک کپی جدید برمی‌گردانند:
var newTeam = team with { Members = team.Members.Add("Reza") }; 
```

---

## ۱۲. تفاوت Immutable Reference و Immutable Object
این دو مفهوم اغلب با هم اشتباه گرفته می‌شوند:

### الف) Immutable Reference (ارجاع تغییرناپذیر)
یعنی **متغیر یا Property شما نمی‌تواند به آبجکت دیگری در حافظه اشاره کند.** (مثل Propertyهای `init` یا فیلدهای `readonly`).
* **مثال:** `user.Address` یک Immutable Reference است. شما نمی‌توانید بگویید `user.Address = new Address()`.

### ب) Immutable Object (آبجکت تغییرناپذیر)
یعنی **داده‌های درون آن آبجکتی که در حافظه قرار دارد، قابل تغییر نیستند.** (مثل `string` یا `ImmutableList`).
* **مثال:** خودِ آبجکت `Address` یک Mutable Object است، چون می‌توانید بگویید `user.Address.City = "Shiraz"`.

**یک مثال برای درک بهتر:**
تصور کنید یک تخته وایت‌برد (آبجکت) دارید که با یک زنجیر (Reference) به دیوار بسته شده است.
* **Immutable Reference:** زنجیر محکم است. شما نمی‌توانید تخته وایت‌برد را از دیوار جدا کنید و یک تخته *دیگر* را جای آن ببندید.
* **Mutable Object:** اما شما می‌توانید با ماژیک، نوشته‌های *روی همان تخته وایت‌برد* را پاک کنید و بنویسید.

Recordهای سی‌شارپ به صورت پیش‌فرض فقط **Immutable Reference** را برای شما فراهم می‌کنند، نه **Immutable Object** را!

---

## منابع رسمی Microsoft Learn

برای مطالعه بیشتر و اطمینان از صحت مطالب، می‌توانید به مستندات رسمی مایکروسافت مراجعه کنید:

1. **[Record types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)**
   *توضیح کامل درباره ساختار Recordها و تفاوت Record Class و Record Struct.*
2. **[init (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/init)**
   *توضیح درباره نحوه کارکرد اکسسور init و محدودیت‌های آن.*
3. **[readonly struct (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-struct)**
   *توضیح درباره نحوه ایجاد Structهای تغییرناپذیر و تاثیر آن روی Record Structها.*
4. **[Immutable Collections (System.Collections.Immutable)](https://learn.microsoft.com/en-us/dotnet/api/system.collections.immutable)**
   *مستندات مربوط به کالکشن‌های Immutable برای پیاده‌سازی Deep Immutability.*