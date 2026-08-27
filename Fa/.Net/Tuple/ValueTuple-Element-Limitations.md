این یک مقاله جامع، دقیق و ساختاریافته است که دقیقاً برای قرار گرفتن در یک Repository آموزشی سی‌شارپ (مانند گیت‌هاب) طراحی شده است. لحن مقاله از مفاهیم پایه شروع شده و به تدریج به عمق مشخصات زبان (Language Specification) و ساختار داخلی CLR می‌رسد.

***

# کالبدشکافی ValueTuple در C#: راز محدودیت ۸ عضو و نقش TRest

## فهرست مطالب
1. [مقدمه: آیا ValueTuple واقعاً فقط ۸ عضو دارد؟](#مقدمه)
2. [تفاوت Arity و تعداد عناصر (Element Count)](#تفاوت-arity-و-تعداد-عناصر)
3. [ساختار ValueTupleهای ۱ تا ۷ عضوی](#ساختار-والیو-تیوپل-های-۱-تا-۷-عضوی)
4. [ورود به قلمرو هشتم: نقش جادویی TRest](#نقش-جادویی-trest)
5. [بررسی Overloadهای متد ValueTuple.Create](#overloadهای-متد-valuetuplecreate)
6. [چرا ساختار برای بیش از ۷ عضو متفاوت می‌شود؟](#چرا-ساختار-برای-بیش-از-۷-عضو-متفاوت-میشود)
7. [مثال عملی: Tuple هشت‌عضوی و بیشتر](#مثال-عملی)
8. [کالبدشکافی ساختار داخلی (Internal Structure)](#کالبدشکافی-ساختار-داخلی)
9. [اشتباهات رایج برنامه‌نویسان](#اشتباهات-رایج)
10. [نکات مهم و کلیدی](#نکات-مهم)
11. [جمع‌بندی](#جمع‌بندی)
12. [منابع رسمی](#منابع-رسمی)

---

## مقدمه
وقتی برای اولین بار در C# با Tupleها کار می‌کنید، ممکن است فکر کنید که `(int a, int b, ...)` می‌تواند بی‌نهایت عضو داشته باشد. اما اگر سعی کنید یک Tuple با ۸ عضو تعریف کنید، با یک رفتار عجیب در IntelliSense یا هنگام بررسی نوع (Type) در دیباگر مواجه می‌شوید. 
آیا `ValueTuple` در سی‌شارپ محدود به ۸ عضو است؟ پاسخ کوتاه **خیر** است، اما پاسخ دقیق‌تر به درک معماری ژنریک‌ها در CLR و مفهوم `TRest` برمی‌گردد. در این مقاله، این محدودیت ظاهری را از نظر_SPECIFICATION_ کالبدشکافی می‌کنیم.

---

## تفاوت Arity و تعداد عناصر
برای درک این موضوع، ابتدا باید تفاوت دو مفهوم را بدانیم:
* **تعداد عناصر (Element Count):** تعداد داده‌هایی که در عمل در Tuple ذخیره می‌کنید (مثلاً ۱۰ عدد صحیح).
* **آرییتی (Arity):** در مبحث ژنریک‌ها، Arity به معنای **تعداد پارامترهای نوع (Type Parameters)** است که یک ساختار می‌پذیرد. 

وقتی شما یک Tuple 9 عضوی می‌سازید، تعداد عناصر شما ۹ است، اما آرییتی (تعداد پارامترهای ژنریک) آن در سطح CLR همچنان ۸ است!

---

## ساختار والیو تیوپل‌های ۱ تا ۷ عضوی
در کتابخانه `System`، کلاس‌های `ValueTuple` به صورت زیر تعریف شده‌اند:
```csharp
public struct ValueTuple<T1>
public struct ValueTuple<T1, T2>
public struct ValueTuple<T1, T2, T3>
// ... تا ...
public struct ValueTuple<T1, T2, T3, T4, T5, T6, T7>
```
تا ۷ عضو، هر عضو دقیقاً متناظر با یک پارامتر ژنریک (`T1` تا `T7`) است و فیلدهای `Item1` تا `Item7` را در اختیار شما قرار می‌دهد.

---

## نقش جادویی TRest
وقتی به عضو هشتم می‌رسیم، CLR نمی‌تواند بی‌نهایت ساختار ژنریک (`ValueTuple<T1...T100>`) تولید کند. راه‌حل چه چیست؟ **تودرتو کردن (Nesting)**.
ساختار هشتم به این شکل تعریف می‌شود:

```csharp
public struct ValueTuple<T1, T2, T3, T4, T5, T6, T7, TRest>
    where TRest : struct
```
**نکته کلیدی:** در این ساختار، `T1` تا `T7` اعضای اول تا هفتم هستند، اما `TRest` **خودش باید یک ValueTuple دیگر باشد** که اعضای هشتم و بعد از آن را در خود جای داده است.

---

## Overloadهای متد ValueTuple.Create
متد `ValueTuple.Create` نیز از همین قانون پیروی می‌کند. اگر به سورس‌کد [.NET Reference Source](https://github.com/dotnet/runtime) نگاه کنید، متوجه می‌شوید که این متد Overloadهای زیر را دارد:

```csharp
// برای ۱ تا ۷ عضو
public static ValueTuple<T1> Create<T1>(T1 item1)
public static ValueTuple<T1, T2> Create<T1, T2>(T1 item1, T2 item2)
// ... تا ...
public static ValueTuple<T1, T2, T3, T4, T5, T6, T7> Create<T1...T7>(...)

// برای ۸ عضو و بیشتر!
public static ValueTuple<T1, T2, T3, T4, T5, T6, T7, TRest> 
    Create<T1, T2, T3, T4, T5, T6, T7, TRest>(
        T1 item1, T2 item2, T3 item3, T4 item4, T5 item5, T6 item6, T7 item7, 
        TRest rest) // <-- توجه کنید: پارامتر هشتم یک Tuple است!
```

---

## چرا ساختار برای بیش از ۷ عضو متفاوت می‌شود؟
این یک محدودیت طراحی در **Common Language Runtime (CLR)** است. تعریف بی‌نهایت نوع ژنریک در Metadata اسمبلی‌ها غیرمنطقی و از نظر حافظه ناکارآمد است. 
به جای ساخت `ValueTuple<T1...T9>`, کامپایلر و CLR از یک الگوی بازگشتی (Recursive) استفاده می‌کنند. ساختار هشتم، یک `ValueTuple` هشت‌پارامتری است که پارامتر آخر آن (`TRest`) می‌تواند دوباره یک `ValueTuple` باشد و این زنجیره تا زمانی که عناصر تمام شوند ادامه می‌یابد (دقیقاً مانند ساختار Cons در زبان Lisp).

---

## مثال عملی
بیایید یک Tuple با ۹ عضو بسازیم و نحوه دسترسی به آن را بررسی کنیم:

```csharp
// ساخت یک Tuple 9 عضوی با سینتکس استاندارد سی‌شارپ
var myTuple = (1, 2, 3, 4, 5, 6, 7, 8, 9);

// دسترسی به 7 عضو اول (مستقیم)
Console.WriteLine(myTuple.Item1); // خروجی: 1
Console.WriteLine(myTuple.Item7); // خروجی: 7

// دسترسی به اعضای هشتم و نهم
// در سطح سورس‌کد، عضو هشتم در واقع Item1 از Rest است!
Console.WriteLine(myTuple.Rest.Item1); // خروجی: 8
Console.WriteLine(myTuple.Rest.Item2); // خروجی: 9
```

**جادوی کامپایلر C#:**
اگر از سینتکس `(1, 2, 3, 4, 5, 6, 7, 8, 9)` استفاده کنید، کامپایلر C# به شما اجازه می‌دهد مستقیماً بنویسید `myTuple.Item8`. اما در پشت صحنه (IL Generated)، کامپایلر این را به `myTuple.Rest.Item1` ترجمه می‌کند تا با ساختار CLR سازگار باشد.

---

## کالبدشکافی ساختار داخلی
اگر بخواهیم ساختار یک Tuple 9 عضوی را در حافظه (Memory Layout) به صورت درختی رسم کنیم، شبیه به این خواهد بود:

```text
ValueTuple<int, int, int, int, int, int, int, TRest>
 ├── Item1 (1)
 ├── Item2 (2)
 ├── ...
 ├── Item7 (7)
 └── Rest (TRest)  ---> این فیلد خودش یک ValueTuple است!
      ├── Item1 (8)  <-- همان عضو هشتم ما
      └── Rest (TRest) ---> اگر عضو دهمی داشتیم، اینجا قرار می‌گرفت
           └── Item1 (9) <-- همان عضو نهم ما
```

---

## اشتباهات رایج
1. **تلاش برای تعریف دستی TRest با نوع غیر Tuple:**
   ```csharp
   // ❌ خطای کامپایل: TRest باید حتماً یک ValueTuple باشد
   var badTuple = new ValueTuple<int, int, int, int, int, int, int, int>(1,2,3,4,5,6,7,8); 
   ```
   *راه‌حل:* برای ساخت دستی باید `TRest` را یک Tuple دیگر قرار دهید:
   ```csharp
   // ✅ صحیح
   var goodTuple = new ValueTuple<int, int, int, int, int, int, int, ValueTuple<int>>(
       1, 2, 3, 4, 5, 6, 7, ValueTuple.Create(8));
   ```

2. **فراموش کردن نقش Rest در Reflection:**
   اگر با Reflection کار می‌کنید، انتظار نداشته باشید فیلدی به نام `Item8` در ساختار پیدا کنید. شما باید فیلد `Rest` را پیدا کرده و سپس `Item1` آن را بخوانید. (برای حل این مشکل، مایکروسافت اینترفیس [`ITuple`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.ituple) را معرفی کرد).

---

## نکات مهم
* **عملکرد (Performance):** با اینکه ساختار تودرتو به نظر پیچیده می‌آید، اما چون `ValueTuple` یک `struct` (Value Type) است، هیچ‌گونه Heap Allocation رخ نمی‌دهد و تمام داده‌ها به صورت پیوسته در Stack (یا درون ساختار والد) قرار می‌گیرند. بنابراین از نظر پرفورمنس کاملاً بهینه است.
* **اینترفیس ITuple:** از .NET Core 2.0 به بعد، تمام `ValueTuple`ها اینترفیس `ITuple` را پیاده‌سازی می‌کنند. این اینترفیس دارای پراپرتی‌های `Length` و `this[int index]` است که دسترسی به اعضای تودرتو را بدون درگیر شدن با `Rest` ممکن می‌سازد.
* **لینک به مباحث دیگر:** برای درک بهتر نحوه استخراج این اعضا، حتماً مقاله [آشنایی با Deconstruction در سی‌شارپ](#) را مطالعه کنید.

---

## جمع‌بندی
محدودیت ۸ عضوی در `ValueTuple` یک محدودیت واقعی در تعداد داده‌ها نیست، بلکه محدودیتی در **تعداد پارامترهای ژنریک (Arity)** در سطح CLR است. سی‌شارپ با استفاده از پارامتر `TRest` و تکنیک تودرتو کردن (Nesting)، این محدودیت را دور زده و به شما اجازه می‌دهد تا بی‌نهایت عضو در یک Tuple داشته باشید، بدون اینکه کوچکترین سرباری در عملکرد برنامه ایجاد شود.

---

## منابع رسمی
برای مطالعه بیشتر و بررسی سورس‌کد، لینک‌های زیر پیشنهاد می‌شوند:
1. [Microsoft Docs: ValueTuple Struct](https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple)
2. [C# Language Specification: Tuples](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#tuples)
3. [Source Code: System.ValueTuple in .NET Runtime](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/ValueTuple.cs)
4. [ITuple Interface Documentation](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.ituple)

***
*نویسنده: [نام شما / نام تیم آموزشی]*
*تاریخ بازبینی: August 2026*