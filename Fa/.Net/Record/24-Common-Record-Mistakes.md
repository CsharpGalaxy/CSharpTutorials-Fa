# معرفی Records در C#

به دنیای **Records** در سی‌شارپ خوش آمدید! اگر تا به حال با ساخت کلاس‌هایی مواجه شده‌اید که فقط وظیفه نگهداری داده را دارند و مجبور بوده‌اید کدهای زیادی برای مقایسه یا کپی کردن آن‌ها بنویسید، این آموزش دقیقاً برای شماست.

---

## Record چیست؟

به زبان ساده، **Record** (رکورد) در سی‌شارپ یک نوع داده‌ای است که **بیشتر از آنکه برای رفتار (Behavior) طراحی شده باشد، برای داده (Data) طراحی شده است.** 
رکوردها به طور پیش‌فرض **تغییرناپذیر (Immutable)** هستند؛ یعنی پس از ایجاد یک رکورد، نمی‌توانید مقادیر درون آن را تغییر دهید.

---

## چرا Record به C# اضافه شد؟ (چه مشکلی را حل می‌کند؟)

قبل از نسخه C# 9، اگر می‌خواستید یک کلاس بسازید که فقط داده نگهداری کند (مثل یک DTO) و بخواهید این کلاس تغییرناپذیر باشد، باید کدهای بسیار زیادی (Boilerplate Code) می‌نوشتید. 

شما مجبور بودید:
1. برای هر ویژگی (Property) یک فیلد خصوصی و یک `get` فقط خواندنی بنویسید.
2. یک Constructor (سازنده) برای مقداردهی اولیه بسازید.
3. متدهای `Equals` و `GetHashCode` را بازنویسی کنید تا بتوانید دو شیء با داده‌های یکسان را مقایسه کنید.
4. متد `ToString` را بازنویسی کنید تا خروجی خوانا داشته باشید.
5. اپراتورهای `==` و `!=` را پیاده‌سازی کنید.

**Record تمام این کارها را با یک خط کد برای شما انجام می‌دهد!**

---

## تفاوت مفهومی Record با Class و Struct

برای درک بهتر، بیایید این سه را مقایسه کنیم:

| ویژگی | Class (کلاس) | Struct (ساختار) | Record (رکورد) |
| :--- | :--- | :--- | :--- |
| **نوع داده** | مرجع (Reference Type) | مقدار (Value Type) | مرجع (Reference Type)* |
| **محل ذخیره** | Heap (معمولاً) | Stack (معمولاً) | Heap (معمولاً) |
| **تغییرپذیری** | معمولاً تغییرپذیر (Mutable) | معمولاً تغییرپذیر | **تغییرناپذیر (Immutable)** |
| **نوع برابری** | بر اساس **مرجع** (آدرس حافظه) | بر اساس **مقدار** | بر اساس **مقدار** (Value-based) |

*\*نکته: در C# 10 نوع `record struct` نیز اضافه شد که مقدار است، اما وقتی کلمه `record` را به تنهایی می‌بینید، منظور یک Reference Type است.*

---

## مهم‌ترین ویژگی‌های Record

1. **برابری بر اساس مقدار (Value Equality):** دو رکورد اگر مقادیر یکسانی داشته باشند، با هم برابر هستند (حتی اگر در آدرس‌های حافظه متفاوتی باشند).
2. **تغییرناپذیری (Immutability):** ویژگی‌ها به صورت پیش‌فرض فقط خواندنی (`init`) هستند.
3. **عبارت with:** امکان کپی کردن یک رکورد و تغییر یک یا چند ویژگی در همان لحظه را می‌دهد (Non-destructive mutation).
4. **ToString هوشمند:** به صورت خودکار نام ویژگی‌ها و مقادیر آن‌ها را به شکل خوانا چاپ می‌کند.
5. **Deconstruction:** امکان تجزیه رکورد به متغیرهای مجزا.

---

## Record برای چه نوع داده‌هایی مناسب است؟

رکوردها برای داده‌هایی مناسب هستند که **نباید در طول حیات خود تغییر کنند**. بهترین کاربردها:
* **DTOها (Data Transfer Objects):** برای انتقال داده بین لایه‌های برنامه یا در APIها.
* **تنظیمات (Configuration):** مقادیری که در زمان اجرای برنامه ثابت هستند.
* **رویدادها (Events):** داده‌هایی که نشان می‌دهند یک اتفاق در سیستم رخ داده است.
* **مقادیر محاسباتی:** مثل مختصات جغرافیایی (Latitude, Longitude) یا پول (Amount, Currency).

---

## مثال‌ها و کاربردها

### ۱. مثال بسیار ساده: مشکل Class و راه‌حل Record

**مسئله:** ما می‌خواهیم دو شیء از یک شخص بسازیم که نام و سن یکسانی دارند و می‌خواهیم ببینیم آیا این دو شیء با هم برابر هستند یا خیر.

```csharp
// --- راه‌حل قدیمی با Class ---
public class PersonClass
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// --- راه‌حل جدید با Record ---
public record PersonRecord(string Name, int Age);

public class Program
{
    public static void Main()
    {
        // ایجاد اشیاء
        var class1 = new PersonClass { Name = "Ali", Age = 25 };
        var class2 = new PersonClass { Name = "Ali", Age = 25 };

        var record1 = new PersonRecord("Ali", 25);
        var record2 = new PersonRecord("Ali", 25);

        // بررسی برابری
        Console.WriteLine($"Class Equality: {class1 == class2}");
        Console.WriteLine($"Record Equality: {record1 == record2}");
        
        // نمایش خروجی ToString
        Console.WriteLine($"Record ToString: {record1}");
    }
}
```

**توضیح بخش‌به‌بخش:**
* `public class PersonClass`: یک کلاس معمولی. وقتی دو شیء از آن می‌سازیم، چون آدرس حافظه آن‌ها متفاوت است، سی‌شارپ آن‌ها را برابر نمی‌داند.
* `public record PersonRecord(string Name, int Age)`: تعریف رکورد به صورت Positional (موقعیتی). سی‌شارپ به صورت خودکار Constructor، ویژگی‌ها و متدهای برابری را می‌سازد.
* `class1 == class2`: نتیجه `False` است، چون کلاس‌ها بر اساس **مرجع (آدرس)** مقایسه می‌شوند.
* `record1 == record2`: نتیجه `True` است، چون رکوردها بر اساس **مقدار** مقایسه می‌شوند.

**خروجی:**
```text
Class Equality: False
Record Equality: True
Record ToString: PersonRecord { Name = Ali, Age = 25 }
```

---

### ۲. مثال واقعی‌تر: استفاده از عبارت `with`

**مسئله:** رکوردها تغییرناپذیر هستند. اگر بخواهیم یک رکورد جدید بسازیم که دقیقاً شبیه رکورد قبلی است اما فقط یک ویژگی آن (مثلاً سن) تغییر کرده باشد، چه کنیم؟ (نوشتن Constructor از اول خسته‌کننده است).

```csharp
public record Book(string Title, string Author, int Price);

public class Program
{
    public static void Main()
    {
        var book1 = new Book("Clean Code", "Robert C. Martin", 100);
        
        // استفاده از with برای ایجاد یک کپی با تغییر قیمت
        var book2 = book1 with { Price = 120 };
        
        // استفاده از with برای تغییر چند ویژگی
        var book3 = book1 with { Title = "Clean Architecture", Price = 150 };

        Console.WriteLine(book1);
        Console.WriteLine(book2);
        Console.WriteLine(book3);
    }
}
```

**توضیح بخش‌به‌بخش:**
* `book1`: رکورد اولیه ما.
* `book1 with { Price = 120 }`: کلمه کلیدی `with` یک کپی کامل از `book1` می‌سازد، اما مقدار `Price` را در کپی جدید تغییر می‌دهد. `book1` دست‌نخورده باقی می‌ماند.

**خروجی:**
```text
Book { Title = Clean Code, Author = Robert C. Martin, Price = 100 }
Book { Title = Clean Code, Author = Robert C. Martin, Price = 120 }
Book { Title = Clean Architecture, Author = Robert C. Martin, Price = 150 }
```

---

### ۳. مثال واقعی‌تر: رکورد با ویژگی‌های نامی (Nominal Record)

**مسئله:** گاهی اوقات نمی‌خواهیم از Constructor موقعیتی (Positional) استفاده کنیم و ترجیح می‌دهیم ویژگی‌ها را به صورت کلاسیک تعریف کنیم، اما همچنان می‌خواهیم از مزایای Record بهره ببریم.

```csharp
// تعریف رکورد به صورت Nominal
public record User
{
    // استفاده از init به جای set برای تضمین تغییرناپذیری
    public string Username { get; init; }
    public string Email { get; init; }
}

public class Program
{
    public static void Main()
    {
        // مقداردهی با استفاده از Object Initializer
        var user1 = new User { Username = "admin", Email = "admin@test.com" };
        
        // این کد خطا می‌دهد چون ویژگی‌ها init هستند:
        // user1.Username = "newAdmin"; 

        var user2 = new User { Username = "admin", Email = "admin@test.com" };
        
        Console.WriteLine($"Are they equal? {user1 == user2}");
    }
}
```

**توضیح:**
در این روش، به جای تعریف در پرانتز جلوی کلمه `record`، ویژگی‌ها را داخل بدنه تعریف می‌کنیم. نکته حیاتی استفاده از `init` به جای `set` است تا تغییرناپذیری حفظ شود.

---

## اشتباهات رایج مبتدیان

1. **استفاده از `set` به جای `init` در Nominal Records:**
   اگر در رکوردهای Nominal از `set` استفاده کنید، رکورد شما تغییرپذیر می‌شود و فلسفه وجودی Record زیر سوال می‌رود. همیشه از `init` استفاده کنید.
2. **استفاده از Record برای Entityهای تغییرپذیر:**
   رکوردها برای موجودیت‌هایی که وضعیت آن‌ها مدام تغییر می‌کند (مثل `BankAccount` که موجودی آن کم و زیاد می‌شود) مناسب نیستند. برای این موارد از `class` معمولی استفاده کنید.
3. **انتظار برابری مرجع (Reference Equality):**
   اگر به هر دلیلی نیاز دارید دو رکورد را بر اساس آدرس حافظه مقایسه کنید، باید از `object.ReferenceEquals(rec1, rec2)` استفاده کنید، نه اپراتور `==`.
4. **فراموش کردن `record struct` برای داده‌های کوچک:**
   اگر داده‌های شما بسیار کوچک هستند و تعداد زیادی از آن‌ها ایجاد می‌شود (مثل مختصات X و Y)، استفاده از `record struct` (معرفی شده در C# 10) بهینه‌تر است زیرا روی Stack ذخیره می‌شود و سربار Garbage Collection ندارد.

---

## جمع‌بندی

رکوردها یکی از بهترین ویژگی‌های اضافه شده به سی‌شارپ در سال‌های اخیر هستند که نوشتن کدهای تمیز، امن و کوتاه را ممکن می‌سازند. آن‌ها با حذف کدهای تکراری (Boilerplate) و ارائه برابری بر اساس مقدار، زندگی را برای توسعه‌دهندگانی که با مدل‌های داده‌ای سروکار دارند، بسیار راحت‌تر کرده‌اند.

### نکات مهم
* رکوردها به طور پیش‌فرض **Reference Type** هستند.
* برای تغییر یک رکورد تغییرناپذیر، همیشه از عبارت **`with`** استفاده کنید.
* رکوردها برای **DTOها** و **تنظیمات** عالی هستند، اما برای **Entityهای فعال** مناسب نیستند.
* می‌توانید رکوردها را **به ارث ببرید (Inheritance)**، اما کلاس‌ها نمی‌توانند از یک Record ارث‌بری کنند.

### آنچه باید به خاطر بسپارید
* 📌 **Record = Data + Immutability + Value Equality**
* 📌 کلمه `with` برای کپی کردن و تغییر جزئی رکوردهاست.
* 📌 در رکوردهای Nominal، حتماً از `get; init;` استفاده کنید.
* 📌 متد `ToString` در رکوردها به صورت خودکار تمام ویژگی‌ها را چاپ می‌کند.

---

## سؤالات تمرینی

برای تثبیت یادگیری، سعی کنید به سؤالات زیر پاسخ دهید یا کد آن‌ها را بنویسید:

1. **سؤال مفهومی:** چرا سی‌شارپ برای رکوردها به صورت پیش‌فرض `Reference Type` را انتخاب کرد در حالی که برابری بر اساس مقدار (مثل Structها) دارند؟
2. **تمرین کدنویسی:** یک رکورد به نام `Temperature` بسازید که دو ویژگی `Value` (از نوع `double`) و `Unit` (از نوع `string` مثل "Celsius") داشته باشد. سپس با استفاده از `with` یک رکورد جدید بسازید که مقدار آن به فارنهایت تبدیل شده باشد.
3. **چالش:** تفاوت خروجی `ToString` در یک `class` معمولی و یک `record` که داده‌های یکسانی دارند چیست؟ کدی بنویسید که این موضوع را اثبات کند.
4. **تحلیل کد:** اگر در یک Nominal Record به جای `init` از `set` استفاده کنیم، کدام یک از ویژگی‌های اصلی Record از بین می‌رود؟

---

## منابع معتبر برای مطالعه بیشتر

برای عمیق‌تر شدن در مبحث Records، حتماً منابع رسمی زیر را مطالعه کنید:

1. **Record types (C# Reference)**
   * مرجع کامل و رسمی زبان سی‌شارپ برای سینتکس رکوردها.
   * [لینک مستقیم به Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)

2. **Record types (C# Guide)**
   * راهنمای کاربردی مایکروسافت شامل الگوها و بهترین روش‌های استفاده از رکوردها.
   * [لینک مستقیم به Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)

3. **C# Language Specification - Records**
   * مشخصات دقیق زبان سی‌شارپ برای کسانی که می‌خواهند بدانند کامپایلر دقیقاً چه کدهایی در پس‌زمینه تولید می‌کند.
   * [لینک مستقیم به GitHub (C# Lang Spec)](https://github.com/dotnet/csharplang/blob/main/spec/classes.md#records)

4. **Working with records in .NET**
   * مقاله‌ای عالی درباره نحوه کار با رکوردها در اکوسیستم .NET و مقایسه آن‌ها با کلاس‌ها.
   * [لینک مستقیم به Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/records)