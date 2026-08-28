# آموزش کامل Non-Destructive Mutation در Recordهای C#

## ۱. مقدمه: مفهوم برای مبتدیان
فرض کنید یک فرم کاغذی چاپ شده دارید که اطلاعات یک شخص در آن نوشته شده است. شما اجازه ندارید چیزی را از روی این فرم پاک کنید یا تغییر دهید (این یعنی **Immutability** یا تغییرناپذیری). اما اگر بخواهید سن این شخص را در فرم تغییر دهید، چه می‌کنید؟ 
شما از فرم اصلی یک **کپی (Photocopy)** می‌گیرید، سپس در کپی جدید، سن را تغییر می‌دهید. حالا شما دو فرم دارید: فرم اصلی که دست‌نخورده باقی مانده و فرم جدید که اطلاعاتش به‌روز شده است.

در برنامه‌نویسی C#، **Non-Destructive Mutation** (تغییر بدون تخریب) دقیقاً همین کار را می‌کند. به جای اینکه داده‌های یک Object را در همان حافظه تغییر دهیم (که برای Objectهای تغییرناپذیر ممنوع است)، یک کپی از آن می‌سازیم و فقط تغییرات دلخواه را در کپی جدید اعمال می‌کنیم.

---

## ۲. چرا Mutation معمولی با Immutability تضاد دارد؟
در برنامه‌نویسی، **Immutability** (تغییرناپذیری) به این معناست که پس از ایجاد یک Object، وضعیت (State) یا داده‌های درونی آن تا پایان عمرش تغییر نکند. 

اگر شما از Mutation معمولی (مثل `person.Age = 30;`) استفاده کنید:
1. **نقض قرارداد:** شما مستقیماً State یک Object تغییرناپذیر را تغییر داده‌اید.
2. **مشکلات Thread-Safety:** اگر چند Thread همزمان به یک Object دسترسی داشته باشند، تغییر مقادیر در حافظه باعث Race Condition و باگ‌های پیچیده می‌شود.
3. **از بین رفتن Predictability:** شما نمی‌توانید تضمین کنید که یک Object در طول زمان چه مقادیری داشته است.

به همین دلیل، زبان C# برای Recordها (که ذاتاً تغییرناپذیر طراحی شده‌اند) اجازه Mutation معمولی را نمی‌دهد و راه حل **Non-Destructive Mutation** را ارائه می‌کند.

---

## ۳. Non-Destructive Mutation چیست؟
این مفهوم یعنی **«ایجاد یک Instance جدید از روی یک Instance موجود، با این تفاوت که یک یا چند Property در Instance جدید مقدار متفاوتی دارند، در حالی که Instance اصلی کاملاً دست‌نخورده باقی می‌ماند.»**
کلمه *Non-Destructive* (بدون تخریب) به این معناست که Object اصلی تخریب یا تغییر نمی‌کند.

---

## ۴. چگونه Record بدون تغییر Object اصلی یک Object جدید ایجاد می‌کند؟
وقتی شما درخواست تغییر یک Record را می‌دهید، کامپایلر C# مراحل زیر را در پشت صحنه انجام می‌دهد:
1. **تخصیص حافظه جدید:** یک فضای جدید در Heap (حافظه دینامیک) برای Object جدید allocate می‌کند.
2. **کپی کردن داده‌ها:** تمام فیلدهای Object اصلی را در حافظه جدید کپی می‌کند.
3. **اعمال تغییرات:** فقط Propertyهایی که شما مشخص کرده‌اید را با مقادیر جدید در Object جدید بازنویسی (Override) می‌کند.
4. **برگرداندن Reference:** آدرس حافظه Object جدید را به شما برمی‌گرداند.

---

## ۵. Shallow Copy (کپی سطحی) چیست؟
وقتی Record یک Object جدید می‌سازد، از **Shallow Copy** استفاده می‌کند. 
* **Value Types** (مثل `int`, `bool`, `struct`): مقدار آن‌ها مستقیماً کپی می‌شود.
* **Reference Types** (مثل `string`, `List`, `class`ها): فقط **آدرس (Reference)** آن‌ها کپی می‌شود، نه خودِ Object.

**نکته مهم:** اگر Record شما حاوی یک `List` یا یک `class` دیگر باشد، Object اصلی و Object جدید هر دو به **یک لیست یا یک Object مشترک** در حافظه اشاره می‌کنند. تغییر در آن Object داخلی، در هر دو Record دیده می‌شود.

---

## ۶. نقش Copy Constructor
کامپایلر C# برای هر Record، یک **Copy Constructor** محافظت‌شده (Protected) تولید می‌کند. این Constructor یک Instance از همان Record را به عنوان ورودی می‌گیرد و تمام فیلدهای آن را در Instance جدید کپی می‌کند.
```csharp
// چیزی که کامپایلر در پشت صحنه تولید می‌کند:
protected Person(Person original) 
{
    this.FirstName = original.FirstName;
    this.LastName = original.LastName;
    this.Age = original.Age;
}
```

---

## ۷. نقش متد `<Clone>$`
کامپایلر همچنین یک متد خاص به نام `<Clone>$` تولید می‌کند. وظیفه این متد این است که Copy Constructor را صدا بزند و یک کپی از Object فعلی را برگرداند.
```csharp
// چیزی که کامپایلر در پشت صحنه تولید می‌کند:
protected virtual Person <Clone>$() 
{
    return new Person(this); // صدا زدن Copy Constructor
}
```

---

## ۸. نقش اکسپرشن `with`
اکسپرشن `with` در واقع **Syntactic Sugar** (شکر syntactic) است. وقتی شما از `with` استفاده می‌کنید، کامپایلر کد شما را به شکل زیر بازنویسی می‌کند:
1. ابتدا متد `<Clone>$` را صدا می‌زند تا یک کپی از Object ساخته شود.
2. سپس از **Object Initializer** برای تنظیم Propertyهای جدید روی Object کپی شده استفاده می‌کند.

---

## ۹. تغییر Propertyها در Instance جدید
پس از اینکه متد `Clone` یک کپی دقیق از Object اصلی ساخت، اکسپرشن `with` وارد عمل می‌شود. هر Propertyای که بعد از `with` مشخص کرده‌اید، روی Object کپی شده بازنویسی (Set) می‌شود. Propertyهایی که مشخص نکرده‌اید، همان مقادیر کپی شده از Object اصلی را حفظ می‌کنند.

---

## ۱۰. مثال مرحله‌به‌مرحله

بیایید این مفهوم را با یک مثال عملی بررسی کنیم:

```csharp
// تعریف Record
public record Person(string FirstName, string LastName, int Age);

// مرحله ۱: ایجاد Instance اصلی
Person person1 = new Person("Ali", "Rezaei", 30);

// مرحله ۲: ایجاد Instance جدید با Non-Destructive Mutation
Person person2 = person1 with { Age = 31, LastName = "Mohammadi" };
```

### تحلیل وضعیت در حافظه:

**۱. person1 تغییر نکرده است:**
اگر `person1` را چاپ کنید، همچنان مقادیر اولیه را دارد:
`Person { FirstName = Ali, LastName = Rezaei, Age = 30 }`

**۲. person2 یک Instance جدید است:**
`person1` و `person2` دو آدرس حافظه کاملاً متفاوت دارند (`ReferenceEquals(person1, person2)` برابر با `false` است).

**۳. چه داده‌هایی کپی شده‌اند؟**
از `person1` به `person2` کپی شده است:
* `FirstName`: مقدار `"Ali"` (چون در `with` تغییری برای آن تعریف نکردیم).

**۴. چه Propertyهایی تغییر کرده‌اند؟**
مقادیر جدید جایگزین مقادیر کپی شده شدند:
* `Age`: از `30` به `31` تغییر یافت.
* `LastName`: از `"Rezaei"` به `"Mohammadi"` تغییر یافت.

نتیجه نهایی `person2`:
`Person { FirstName = Ali, LastName = Mohammadi, Age = 31 }`

---

## ۱۱. جدول مقایسه: Mutation در برابر Non-Destructive Mutation

| ویژگی | Mutation معمولی (Destructive) | Non-Destructive Mutation (با `with`) |
| :--- | :--- | :--- |
| **تأثیر روی Object اصلی** | Object اصلی تغییر می‌کند (تخریب می‌شود). | Object اصلی کاملاً دست‌نخورده باقی می‌ماند. |
| **تخصیص حافظه** | حافظه جدیدی تخصیص داده نمی‌شود (همان Object قبلی). | یک Object کاملاً جدید در حافظه (Heap) ایجاد می‌شود. |
| **مناسب برای Immutability** | ❌ خیر (تضاد دارد). | ✅ بله (حالت تغییرناپذیری را حفظ می‌کند). |
| **Thread-Safety** | ❌ نیاز به Lock و همگام‌سازی دارد. | ✅ ذاتاً Thread-Safe است. |
| **سینتکس (Syntax)** | `person.Age = 31;` | `var newPerson = person with { Age = 31 };` |
| **عملکرد (Performance)** | سریع‌تر (چون Object جدیدی ساخته نمی‌شود). | کندتر (به دلیل تخصیص حافظه و Garbage Collection). |
| **نوع کپی** | ندارد (تغییر درجا). | Shallow Copy (کپی سطحی). |

---

## ۱۲. منابع رسمی برای مطالعه بیشتر

برای عمیق‌تر شدن در این مفهوم، پیشنهاد می‌شود منابع رسمی زیر را مطالعه کنید:

1. **Microsoft Learn - Records (C# Reference):**
   * بخش `with` expression و نحوه کارکرد Non-destructive mutation.
   * لینک: [Records - C# reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)

2. **C# Language Specification (مشخصات زبان سی‌شارپ):**
   * بخش `11.11.16 With expressions` و بخش `15.12 Records` که دقیقاً توضیح می‌دهد کامپایلر چگونه متد `<Clone>$` و Copy Constructor را تولید می‌کند.
   * لینک: [C# Language Specification | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/)

3. **Microsoft Learn - Object initialization (Expressions):**
   * برای درک بهتر اینکه چگونه `with` از Object Initializer برای ست کردن مقادیر جدید استفاده می‌کند.
   * لینک: [Object and collection initializers - C# Programming Guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers)