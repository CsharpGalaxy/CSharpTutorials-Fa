# بررسی Mutable و Immutable بودن در Tuple های سی‌شارپ
**(ValueTuple در برابر System.Tuple و Anonymous Type)**

به نام خدا. به مستندات آموزشی این Repository خوش آمدید. در این مقاله، یکی از مهم‌ترین و در عین حال چالش‌برانگیزترین مباحث مربوط به Tupleها در سی‌شارپ، یعنی **تفاوت در Mutable (تغییرپذیر) و Immutable (تغییرناپذیر) بودن** را بررسی می‌کنیم. درک این موضوع برای جلوگیری از باگ‌های سخت‌پیدا (Hard-to-find bugs) در پروژه‌های واقعی حیاتی است.

---

## 📑 فهرست مطالب
1. [مفاهیم پایه: Mutable و Immutable](#1-مفاهیم-پایه-mutable-و-immutable)
2. [آیا ValueTuple Mutable است؟](#2-آیا-valuetuple-mutable-است)
3. [تغییر اعضای ValueTuple](#3-تغییر-اعضای-valuetuple)
4. [تفاوت با Anonymous Type](#4-تفاوت-با-anonymous-type)
5. [تفاوت با System.Tuple](#5-تفاوت-با-systemtuple)
6. [تأثیر Mutable بودن بر Equality](#6-تأثیر-mutable-بودن-بر-equality)
7. [تأثیر Mutable بودن بر Dictionary Key (بسیار مهم)](#7-تأثیر-mutable-بودن-بر-dictionary-key-بسیار-مهم)
8. [مثال‌های صحیح و غلط](#8-مثالهای-صحیح-و-غلط)
9. [جدول مقایسه‌ای جامع](#9-جدول-مقایسه‌ای-جامع)
10. [Best Practice ها](#10-best-practice-ها)
11. [نکات مهم و اشتباهات رایج](#11-نکات-مهم-و-اشتباهات-رایج)
12. [جمع‌بندی](#12-جمع‌بندی)
13. [منابع معتبر](#13-منابع-معتبر)

---

## 1. مفاهیم پایه: Mutable و Immutable

قبل از ورود به بحث Tupleها، باید تعریف دقیقی از این دو مفهوم داشته باشیم:

*   **Mutable (تغییرپذیر):** یک شیء یا ساختار Mutable است اگر پس از ایجاد (Instantiation)، بتوان وضعیت (State) یا مقادیر درونی آن را تغییر داد.
*   **Immutable (تغییرناپذیر):** یک شیء Immutable است اگر پس از ایجاد، وضعیت درونی آن **هرگز** قابل تغییر نباشد. اگر بخواهید تغییری ایجاد کنید، در واقع باید یک شیء کاملاً جدید بسازید.

> 💡 **لینک داخلی پیشنهادی:** برای درک بهتر Immutable بودن در سی‌شارپ، پیشنهاد می‌کنیم مقاله [آشنایی با Record ها در سی‌شارپ](./Records.md) را نیز مطالعه کنید.

---

## 2. آیا ValueTuple Mutable است؟

**پاسخ کوتاه: بله، کاملاً Mutable است.**

از سی‌شارپ 7.0، نوع `ValueTuple` (که با سینتکس `(T1, T2)` شناخته می‌شود) معرفی شد. این نوع در واقع یک **Struct** (نوع مقدار) است. ویژگی مهم `ValueTuple` این است که فیلدهای درونی آن (`Item1`, `Item2`, ...) به صورت **Public Field** تعریف شده‌اند، نه Property. به همین دلیل، کامپایلر اجازه تغییر مستقیم آن‌ها را می‌دهد.

---

## 3. تغییر اعضای ValueTuple

چون فیلدهای `ValueTuple` عمومی (Public) هستند، شما می‌توانید آن‌ها را مستقیماً تغییر دهید:

```csharp
// ایجاد یک ValueTuple
var person = (Name: "Ali", Age: 30);

Console.WriteLine($"Before: {person.Name}, {person.Age}"); 
// خروجی: Before: Ali, 30

// تغییر مستقیم اعضا (چون Mutable است)
person.Name = "Reza";
person.Age = 35;

Console.WriteLine($"After: {person.Name}, {person.Age}"); 
// خروجی: After: Reza, 35
```

---

## 4. تفاوت با Anonymous Type

برخلاف `ValueTuple`، **Anonymous Typeها کاملاً Immutable هستند.**
وقتی شما یک Anonymous Type می‌سازید، کامپایلر یک کلاس با **Read-Only Properties** تولید می‌کند.

```csharp
var anonPerson = new { Name = "Ali", Age = 30 };

// ❌ خطای کامپایل: Property or indexer 'AnonymousType.Name' cannot be assigned to -- it is read only
// anonPerson.Name = "Reza"; 

Console.WriteLine(anonPerson.Name); // خروجی: Ali
```

---

## 5. تفاوت با System.Tuple

قبل از سی‌شارپ 7.0، ما از `System.Tuple` استفاده می‌کردیم. این نوع نیز **Immutable** است، اما تفاوت‌های ساختاری مهمی با `ValueTuple` دارد:

*   `System.Tuple` یک **Class** (نوع مرجع) است، اما `ValueTuple` یک **Struct** (نوع مقدار) است.
*   `System.Tuple` دارای **Read-Only Properties** است (Immutable).
*   `System.Tuple` به دلیل Reference Type بودن، باعث **Boxing** و افت عملکرد در حلقه‌ها می‌شود.

```csharp
// System.Tuple (Immutable)
var sysTuple = Tuple.Create("Ali", 30);
// ❌ خطای کامپایل: Property or indexer 'Tuple<string, int>.Item1' cannot be assigned to -- it is read only
// sysTuple.Item1 = "Reza"; 
```

---

## 6. تأثیر Mutable بودن بر Equality

در سی‌شارپ، `ValueTuple` متدهای `Equals` و `GetHashCode` را بر اساس **مقادیر فعلی فیلدها** پیاده‌سازی کرده است. چون فیلدها تغییر می‌کنند، نتیجه Equality هم تغییر می‌کند!

```csharp
var tuple1 = (1, 2);
var tuple2 = (1, 2);

Console.WriteLine(tuple1 == tuple2); // True

// تغییر مقدار
tuple1.Item1 = 99;

Console.WriteLine(tuple1 == tuple2); // False (چون Mutable است و مقدار تغییر کرد)
```

---

## 7. تأثیر Mutable بودن بر Dictionary Key (بسیار مهم)

🚨 **این مهم‌ترین بخش مقاله است که بسیاری از توسعه‌دهندگان از آن غافل هستند.**

وقتی یک شیء را به عنوان **Key** در یک `Dictionary` یا عضوی از یک `HashSet` قرار می‌دهید، آن شیء نباید Mutable باشد. اگر `ValueTuple` را به عنوان Key استفاده کنید و سپس مقادیر آن را تغییر دهید، **Hash Code آن تغییر می‌کند** و Dictionary دیگر نمی‌تواند آن را پیدا کند!

### ❌ مثال غلط (خطرناک):
```csharp
var dictionary = new Dictionary<(int, int), string>();
var key = (1, 2);

dictionary.Add(key, "Point A");

// تغییر مقدار Key (چون ValueTuple Mutable است)
key.Item1 = 99; 

// تلاش برای دسترسی به مقدار
if (dictionary.TryGetValue(key, out var value))
{
    Console.WriteLine(value);
}
else
{
    Console.WriteLine("Key not found!"); // ❌ این پیام چاپ می‌شود!
}
// حتی dictionary.ContainsKey((1, 2)) هم دیگر کار نمی‌کند چون ساختار داخلی Dictionary خراب شده است.
```

---

## 8. مثال‌های صحیح و غلط

### ✅ استفاده صحیح از ValueTuple:
1.  **بازگرداندن چندین مقدار از یک متد:**
    ```csharp
    public (int Id, string Name) GetUser() => (1, "Ali");
    ```
2.  **گروه‌بندی موقت داده‌ها (Temporary Data Grouping):**
    ```csharp
    var tempData = (Count: items.Count(), TotalPrice: items.Sum(i => i.Price));
    ```

### ❌ استفاده غلط از ValueTuple:
1.  **استفاده به عنوان Dictionary Key یا HashSet Item.**
2.  **ذخیره در State های طولانی‌مدت** که نیاز به تضمین تغییرناپذیری دارند (مثل State در Blazor یا React اگر از Immutable بودن مطمئن نیستید).
3.  **جایگزین کردن کلاس‌های Domain Model:** اگر داده‌های شما منطق (Logic) دارند، از Struct یا Class استفاده کنید، نه Tuple.

---

## 9. جدول مقایسه‌ای جامع

| ویژگی | ValueTuple `(T1, T2)` | System.Tuple `Tuple<T1, T2>` | Anonymous Type `new { ... }` |
| :--- | :--- | :--- | :--- |
| **نوع داده (Type)** | Struct (Value Type) | Class (Reference Type) | Class (Reference Type) |
| **Mutability** | **Mutable** (Public Fields) | **Immutable** (Read-Only Props) | **Immutable** (Read-Only Props) |
| **سینتکس** | `(1, "Ali")` | `Tuple.Create(1, "Ali")` | `new { Id = 1, Name = "Ali" }` |
| **نام‌گذاری اعضا** | پشتیبانی می‌کند (Named Tuples) | خیر (فقط Item1, Item2) | بله (بر اساس نام Property) |
| **عملکرد (Performance)**| بسیار بالا (بدون Boxing) | پایین‌تر (دارای Boxing) | متوسط (نیاز به Reflection در برخی سناریوها) |
| **استفاده به عنوان Key** | ❌ **ممنوع** (به دلیل Mutable بودن) | ✅ مجاز (به دلیل Immutable بودن) | ✅ مجاز (به دلیل Immutable بودن) |
| **پشتیبانی از LINQ** | ضعیف (نیاز به Select دستی) | ضعیف | خوب (در Select های اولیه) |
| **محدودیت تعداد اعضا**| تا 8 عضو (با ValueTuple.Rest) | تا 8 عضو (با Tuple.Rest) | نامحدود (تئوریکاً) |

---

## 10. Best Practice ها

1.  **قانون طلایی Dictionary:** هرگز، هرگز و هرگز از `ValueTuple` به عنوان Key در `Dictionary` یا عضو `HashSet` استفاده نکنید. اگر به Tuple به عنوان Key نیاز دارید، از `System.Tuple` یا `record struct` (از سی‌شارپ 10 به بعد) استفاده کنید.
2.  **ترجیح Named Tuples:** همیشه برای اعضای Tuple نام تعیین کنید تا خوانایی کد (Readability) حفظ شود. `(int id, string name)` بسیار بهتر از `(int, string)` است.
3.  **استفاده از `record` به جای Tuple برای مدل‌ها:** اگر نیاز به یک ساختار داده‌ای Immutable دارید که چندین فیلد داشته باشد، به جای `System.Tuple` یا `ValueTuple`، از `record` یا `record struct` استفاده کنید.
4.  **محدود کردن Scope:** `ValueTuple` را برای متغیرهای محلی (Local variables) و بازگشت متدها نگه دارید. آن را به عنوان فیلد کلاس (Class Field) یا Property عمومی (Public Property) در API خود قرار ندهید.

---

## 11. نکات مهم و اشتباهات رایج

*   **اشتباه رایج 1:** *"چون Tuple ها در سی‌شارپ 7 معرفی شدند، پس همه Tuple ها ValueTuple هستند."*
    *   **تصحیح:** خیر، `System.Tuple` هنوز در فریم‌ورک وجود دارد و کاملاً متفاوت (Immutable و Reference Type) است.
*   **اشتباه رایج 2:** *"من Tuple را به متد دیگری پاس دادم، پس اگر آنجا تغییر کند، روی متغیر اصلی من اثر نمی‌گذارد."*
    *   **تصحیح:** چون `ValueTuple` یک Struct است، به صورت **Pass by Value** پاس داده می‌شود. تغییر آن در متد دیگر، روی متغیر اصلی اثر **نمی‌گذارد**. اما اگر آن را به عنوان `ref` یا `in` پاس دهید، یا در یک کلاس (Reference Type) قرار دهید و آن کلاس را پاس دهید، تغییرات اعمال می‌شود.
*   **نکته مهم:** اگر از سی‌شارپ 10 به بعد استفاده می‌کنید، می‌توانید `record struct` بسازید که هم مزایای Struct (Performance) را دارد و هم Immutable است. این بهترین جایگزین برای `ValueTuple` در سناریوهایی است که به Immutability نیاز دارید.

---

## 12. جمع‌بندی

درک تفاوت بین `ValueTuple` (که Mutable است) و سایر انواع Tuple (که Immutable هستند) برای نوشتن کد سی‌شارپ ایمن و بهینه ضروری است. 
*   از **ValueTuple** برای کارهای موقت، بازگشت چندین مقدار از متد و افزایش Performance استفاده کنید.
*   هرگز آن را در ساختارهای مبتنی بر Hash (مثل Dictionary) به عنوان Key قرار ندهید.
*   اگر به ساختاری نیاز دارید که هم سبک باشد (Value Type) و هم تغییرناپذیر (Immutable)، به جای `ValueTuple` از **`record struct`** استفاده کنید.

---

## 13. منابع معتبر

1.  [Microsoft Docs: Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples) - مستندات رسمی مایکروسافت درباره ValueTuples.
2.  [Microsoft Docs: System.Tuple](https://learn.microsoft.com/en-us/dotnet/api/system.tuple) - مستندات رسمی System.Tuple.
3.  کتاب *C# in Depth* نوشته Jon Skeet - فصل مربوط به Tuples و Value Types.
4.  [Nick Chapsas YouTube: Why you shouldn't use ValueTuple as Dictionary Key](https://www.youtube.com/) - ویدیوی آموزشی درباره باگ‌های Dictionary و ValueTuple.

---
*این مقاله بخشی از Repository آموزشی C# است. برای مشاهده سایر مباحث، به [فهرست اصلی مستندات](../README.md) مراجعه کنید.*