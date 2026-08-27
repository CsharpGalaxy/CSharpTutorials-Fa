# آموزش جامع Tuple و GetHashCode در C#؛ کلیدهای ترکیبی و چالش‌های Mutable

در برنامه‌نویسی C#، هنگام کار با مجموعه‌های مبتنی بر Hash (مانند `Dictionary` و `HashSet`)، درک نحوه محاسبه و مقایسه کلیدها از اهمیت بالایی برخوردار است. در این مقاله آموزشی، به بررسی عمیق رابطه بین **Tupleها** و **GetHashCode** می‌پردازیم و می‌آموزیم که چگونه از آن‌ها به عنوان Composite Key استفاده کنیم و از چه تله‌هایی باید دوری کنیم.

---

## 📑 فهرست مطالب
- [HashCode چیست؟](#hashcode-چیست)
- [چرا Tupleها GetHashCode دارند؟](#چرا-tupleها-gethashcode-دارند)
- [رابطه Equals و GetHashCode (قرارداد طلایی)](#رابطه-equals-و-gethashcode)
- [Tuple به عنوان Dictionary Key و HashSet Element](#tuple-به-عنوان-dictionary-key-و-hashset-element)
- [مفهوم Composite Key](#مفهوم-composite-key)
- [مثال عملی Dictionary](#مثال-عملی-dictionary)
- [چرا Equality و HashCode باید هماهنگ باشند؟](#چرا-equality-و-hashcode-باید-هماهنگ-باشند)
- [مشکلات تغییر Mutable Tuple پس از استفاده به عنوان Key](#مشکلات-تغییر-mutable-tuple)
- [مثال صحیح و غلط](#مثال-صحیح-و-غلط)
- [نکات مهم و اشتباهات رایج](#نکات-مهم-و-اشتباهات-رایج)
- [جمع‌بندی](#جمع‌بندی)
- [منابع معتبر](#منابع-معتبر)

---

## HashCode چیست؟
در زبان C#، متد `GetHashCode()` یک مقدار عددی صحیح (`Int32`) را برمی‌گرداند که به عنوان شناسه‌ای برای اشیاء در **مجموعه‌های مبتنی بر جدول Hash** (Hash-based Collections) مانند `Dictionary<TKey, TValue>` و `HashSet<T>` استفاده می‌شود.

وقتی شما یک کلید را در یک `Dictionary` ذخیره می‌کنید، Runtime ابتدا `GetHashCode()` را فراخوانی می‌کند تا مشخص کند این شیء باید در کدام «باکت» (Bucket) یا خانه از جدول Hash قرار گیرد. سپس برای بررسی تطابق دقیق، از متد `Equals()` استفاده می‌کند.

> ⚠️ **نکته مهم معماری:** الگوریتم داخلی و ریاضیاتی که .NET برای تولید HashCode استفاده می‌کند، یک **Implementation Detail** است و ممکن است بین نسخه‌های مختلف .NET (مثلاً .NET Framework و .NET Core/5+) یا حتی بین پلتفرم‌های ۳۲ و ۶۴ بیتی متفاوت باشد. بنابراین، هرگز نباید HashCode را برای مقاصدی مانند ذخیره‌سازی در دیتابیس یا رمزنگاری استفاده کنید.

---

## چرا Tupleها GetHashCode دارند؟
از زمان معرفی `ValueTuple` در C# 7.0، تیوپل‌ها برای **برابری ساختاری (Structural Equality)** طراحی شده‌اند. این یعنی دو تیوپل `(1, 2)` و `(1, 2)` از نظر مقدار کاملاً یکسان هستند.

برای اینکه تیوپل‌ها بتوانند در مجموعه‌هایی مثل `Dictionary` به درستی کار کنند، فریم‌ورک .NET به‌صورت خودکار متد `GetHashCode()` را برای آن‌ها Overrride کرده است. HashCode یک تیوپل، ترکیبی (Composite) از HashCode تک‌تک عناصر درون آن است. این ویژگی باعث می‌شود تیوپل‌ها برای استفاده به عنوان کلیدهای ترکیبی آماده باشند.

---

## رابطه Equals و GetHashCode (قرارداد طلایی)
برای هر شیء در C#، یک قرارداد سفت و سخت بین `Equals` و `GetHashCode` وجود دارد که رعایت آن برای عملکرد صحیح `Dictionary` و `HashSet` الزامی است:

1. **اگر `a.Equals(b)` مقدار `true` برگرداند، آنگاه حتماً باید `a.GetHashCode() == b.GetHashCode()` باشد.**
2. اگر `GetHashCode` دو شیء برابر باشد، **لزومی ندارد** که `Equals` آن‌ها نیز برابر باشد (به این حالت Collision یا برخورد می‌گویند).
3. اگر `GetHashCode` دو شیء متفاوت باشد، آنگاه `Equals` آن‌ها **حتماً** متفاوت است.

تیوپل‌ها در .NET به‌گونه‌ای پیاده‌سازی شده‌اند که این قرارداد را به‌صورت خودکار و بر اساس مقادیر عناصرشان رعایت می‌کنند.

---

## Tuple به عنوان Dictionary Key و HashSet Element
به دلیل پیاده‌سازی صحیح `IEquatable<T>` و `GetHashCode` در تیوپل‌ها، شما می‌توانید بدون نیاز به نوشتن هیچ‌گونه کد اضافی (Custom Comparer)، از آن‌ها به عنوان کلید دیکشنری یا عضو HashSet استفاده کنید.

*   **در Dictionary:** تیوپل به عنوان `TKey` عمل کرده و بر اساس مقادیر درونش جستجو می‌شود.
*   **در HashSet:** تیوپل‌هایی که مقادیر درون آن‌ها کاملاً یکسان باشد، به عنوان یک عنصر تکراری (Duplicate) شناخته شده و وارد مجموعه نمی‌شوند.

---

## مفهوم Composite Key
گاهی اوقات برای شناسایی یک رکورد، یک مقدار کافی نیست و به ترکیبی از چند مقدار نیاز داریم. به این مفهوم **Composite Key** (کلید ترکیبی) می‌گویند.
مثال‌های رایج:
*   مختصات یک خانه در صفحه شطرنج `(X, Y)`
*   نام و نام خانوادگی یک کاربر `(FirstName, LastName)`
*   شناسه کاربر و شناسه نقش `(UserId, RoleId)`

در گذشته، برنامه‌نویسان مجبور بودند برای هر Composite Key یک کلاس یا ساختار سفارشی بسازند و `Equals` و `GetHashCode` را پیاده‌سازی کنند. امروزه **Tupleها** این مشکل را به‌طور کامل حل کرده‌اند.

---

## مثال عملی Dictionary
فرض کنید می‌خواهیم وضعیت خانه‌های یک بازی یا نقشه را در یک `Dictionary` ذخیره کنیم:

```csharp
// ایجاد یک Dictionary با کلید ترکیبی (مختصات X و Y)
var gameBoard = new Dictionary<(int X, int Y), string>();

// اضافه کردن مقادیر
gameBoard.Add((0, 0), "Player1");
gameBoard.Add((1, 2), "Obstacle");
gameBoard[(5, 5)] = "Treasure"; // استفاده از Indexer

// جستجو بر اساس Tuple
if (gameBoard.TryGetValue((1, 2), out var cellContent))
{
    Console.WriteLine($"محتوای خانه (1,2): {cellContent}"); 
    // خروجی: Obstacle
}

// بررسی وجود یک کلید ترکیبی
bool hasPlayer = gameBoard.ContainsKey((0, 0)); // true
```

---

## چرا Equality و HashCode باید هماهنگ باشند؟
اگر شما یک کلاس یا ساختار سفارشی بسازید و فقط `Equals` را Override کنید اما `GetHashCode` را دست نزنید (یا برعکس)، `Dictionary` دچار رفتارهای غیرمنتظره و باگ‌های سخت‌یاب می‌شود.

**دلیل فنی:** دیکشنری برای پیدا کردن یک آیتم، ابتدا با `O(1)` به باکتِ مربوط به HashCode می‌رود. اگر HashCode اشتباه محاسبه شود، دیکشنری به باکت غلط می‌رود و حتی اگر آیتم آنجا باشد، آن را پیدا نمی‌کند. هماهنگی این دو متد، تضمین‌کننده **صحت منطقی** و **عملکرد `O(1)`** در جستجوهاست.

---

## مشکلات تغییر Mutable Tuple پس از استفاده به عنوان Key
یکی از بزرگترین تله‌ها در C# مربوط به **تغییرپذیری (Mutability)** است.
ما دو نوع تیوپل در C# داریم:
1.  `System.Tuple<T1, T2>` (Reference Type - غیرقابل تغییر / Immutable)
2.  `System.ValueTuple<T1, T2>` (Value Type - قابل تغییر / Mutable)

از آنجا که `ValueTuple` یک `struct` است، هنگام ورود به `Dictionary` **کپی** می‌شود. بنابراین تغییر دادن متغیر محلی شما، تاثیری روی کلید داخل دیکشنری ندارد.
**اما خطر اصلی کجاست؟**
اگر داخل `ValueTuple` خود از **Reference Typeهای تغییرپذیر (Mutable Reference Types)** استفاده کنید، با تغییر دادن آن شیء مرجع، HashCode کلید تغییر می‌کند و کلید در دیکشنری **گم می‌شود**!

---

## مثال صحیح و غلط

### ❌ مثال غلط (استفاده از Reference Type تغییرپذیر داخل Tuple)
```csharp
var dict = new Dictionary<(List<int> Numbers, string Name), string>();

var myList = new List<int> { 1, 2 };
var key = (myList, "Test");

dict.Add(key, "Value");

// تغییر دادن محتوای Reference Type داخل Tuple
myList.Add(3); 

// تلاش برای پیدا کردن کلید
bool found = dict.ContainsKey(key); 
Console.WriteLine(found); // خروجی: False ❌ (کلید گم شده است!)
```
*توضیح:* چون `List` یک Reference Type است، تغییر آن باعث تغییر HashCode محاسبه شده برای Tuple می‌شود. دیکشنری دیگر نمی‌تواند این کلید را در باکت قبلی پیدا کند.

### ✅ مثال صحیح (استفاده از Value Types یا Record Structs)
```csharp
// روش اول: استفاده از Tuple با Value Typeهای خالص (ایمن)
var safeDict = new Dictionary<(int Id, string Code), string>();
safeDict.Add((1, "A"), "Item1"); // کاملاً ایمن

// روش دوم (پیشنهادی در C# 10 به بعد): استفاده از record struct
public readonly record struct Coordinate(int X, int Y);

var modernDict = new Dictionary<Coordinate, string>();
modernDict.Add(new Coordinate(10, 20), "Target");

// چون record struct ذاتاً Immutable است، هرگز دچار مشکل تغییر HashCode نمی‌شود.
```

---

## نکات مهم و اشتباهات رایج

1. **اشتباه رایج:** تصور اینکه `ValueTuple` به دلیل Mutable بودن، برای Dictionary خطرناک است.
   * **واقعیت:** `ValueTuple` تا زمانی که فقط شامل Value Typeها (مثل `int`, `string`, `DateTime`) باشد، به دلیل کپی شدن در هنگام `Add`، کاملاً ایمن است. خطر فقط زمانی است که داخل آن Reference Typeهای Mutable قرار دهید.
2. **نکته_performance:** استفاده از `ValueTuple` به عنوان کلید، به دلیل Value Type بودن، از تخصیص حافظه در Heap (و در نتیجه فشار به Garbage Collector) جلوگیری می‌کند و بسیار بهینه‌تر از استفاده از کلاس‌های سفارشی یا `System.Tuple` است.
3. **اشتباه رایج:** استفاده از `Tuple` برای ذخیره‌سازی دائمی (Persistent Storage).
   * **واقعیت:** همانطور که گفته شد، الگوریتم HashCode تیوپل‌ها در نسخه‌های مختلف .NET ممکن است تغییر کند. هرگز HashCode تیوپل را در فایل یا دیتابیس ذخیره نکنید.
4. **نکته جایگزین:** اگر در حال توسعه پروژه‌ای با C# 10 یا بالاتر هستید، برای Composite Keyها به جای Tuple، از **`record struct`** استفاده کنید. این کار باعث می‌شود کد شما Self-Documenting باشد و از نظر منطقی (Domain-Driven Design) مفاهیم را بهتر منتقل کند.

---

## جمع‌بندی
تیوپل‌ها (`ValueTuple`) به لطف پیاده‌سازی داخلی `Equals` و `GetHashCode` بر اساس مقادیر، ابزاری فوق‌العاده برای ایجاد **Composite Key** در `Dictionary` و `HashSet` هستند. آن‌ها نیاز به نوشتن کلاس‌های Boilerplate را از بین برده و عملکرد بالایی دارند.
با این حال، به عنوان یک توسعه‌دهنده حرفه‌ای C#، باید همیشه **قرارداد طلایی HashCode** را به یاد داشته باشید و مراقب باشید که کلیدهای شما (چه مستقیم و چه از طریق Reference Typeهای درون Tuple) پس از درج در مجموعه‌های Hash-based، **تغییر حالت (Mutate)** ندهند.

---

## منابع معتبر
برای مطالعه بیشتر و اطمینان از صحت مباحث، می‌توانید به مستندات رسمی مایکروسافت مراجعه کنید:

1. [Microsoft Docs: Object.GetHashCode Method](https://learn.microsoft.com/en-us/dotnet/api/system.object.gethashcode)
2. [Microsoft Docs: ValueTuple Struct](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple)
3. [Microsoft Docs: How to define value equality for a class or struct](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/equality)
4. [Microsoft Docs: Dictionary<TKey,TValue> Class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2)