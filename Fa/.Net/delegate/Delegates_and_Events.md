

# 📘 راهنمای جامع Delegate و Event در سی‌شارپ (از مقدماتی تا پیشرفته)

به ریپازیتوری آموزشی سی‌شارپ خوش آمدید! در این مقاله، ما دو مفهوم بسیار مهم و قدرتمند در سی‌شارپ، یعنی **Delegate** و **Event** را از صفر مطلق تا مباحث پیشرفته (مثل Immutability و Event Accessors) بررسی می‌کنیم. 

اگر مبتدی هستید، نگران نباشید؛ همه چیز با مثال‌های ساده و تشبیه‌های کاربردی توضیح داده شده است.

---

## 📑 فهرست مطالب
1. [Delegate چیست؟](#1-delegate-چیست)
2. [توابع مرتبه بالاتر (Higher-Order Functions)](#2-توابع-مرتبه-بالاتر-higher-order-functions)
3. [Multicast Delegate (دلیگیت‌های چندگانه)](#3-multicast-delegate-دلیگیتهای-چندگانه)
4. [Invocation List (لیست فراخوانی)](#4-invocation-list-لیست-فراخوانی)
5. [Immutability در Delegate (تغییرناپذیری)](#5-immutability-در-delegate-تغییرناپذیری)
6. [Event (رویدادها) و فلسفه آن](#6-event-رویدادها-و-فلسفه-آن)
7. [Event Accessors (دسترسی‌کننده‌های رویداد)](#7-event-accessors-دسترسیکنندههای-رویداد)
8. [منابع معتبر](#8-منابع-معتبر)

---

## 1. Delegate چیست؟

به زبان ساده، **Delegate** یک نوع (Type) در سی‌شارپ است که می‌تواند **اشاره‌گر به یک متد (Method)** باشد. 
اگر با زبان C کار کرده باشید، Delegate شبیه به "Function Pointers" است، اما با این تفاوت که در سی‌شارپ **Type-Safe** (امن از نظر نوع داده) و شیءگرا است.

### ساختار کلی Delegate
برای تعریف یک Delegate، از کلمه کلیدی `delegate` استفاده می‌کنیم. امضای Delegate (نوع خروجی و ورودی‌ها) باید دقیقاً با متدی که به آن اشاره می‌کند، همخوانی داشته باشد.

```csharp
// تعریف Delegate
public delegate int MathOperation(int a, int b);

// متدهایی که با Delegate همخوانی دارند
public int Add(int x, int y) => x + y;
public int Multiply(int x, int y) => x * y;
```

### Target و Method در Delegate
هر Delegate در سی‌شارپ در باطن یک شیء است که دو ویژگی (Property) بسیار مهم دارد:
1. **`Target`**: اشاره به **شیء (Instance)** دارد که متد روی آن صدا زده می‌شود. اگر متد Static باشد، این مقدار `null` است.
2. **`Method`**: یک شیء از نوع `MethodInfo` است که **اطلاعات خودِ متد** (مثل نام متد) را در خود نگه می‌دارد.

### رابطه Target و Method (Static vs Instance)
* **Instance Method (متد نمونه):** وقتی Delegate به یک متد معمولی (غیر استاتیک) اشاره می‌کند، `Target` برابر با آبجکتی است که متد متعلق به آن است، و `Method` نام متد را نشان می‌دهد.
* **Static Method (متد استاتیک):** چون متد استاتیک متعلق به هیچ آبجکت خاصی نیست، `Target` برابر با `null` خواهد بود و `Method` اطلاعات متد استاتیک را نگه می‌دارد.

```csharp
MathOperation del = new MathOperation(myObj.Add); 
// del.Target == myObj
// del.Method.Name == "Add"

MathOperation staticDel = new MathOperation(MyClass.StaticAdd);
// staticDel.Target == null
// staticDel.Method.Name == "StaticAdd"
```

---

## 2. توابع مرتبه بالاتر (Higher-Order Functions)

### Higher-Order Function چیست؟
در برنامه‌نویسی، به تابعی **Higher-Order** گفته می‌شود که حداقل یکی از دو کار زیر را انجام دهد:
1. یک تابع را به عنوان **ورودی** (Parameter) دریافت کند.
2. یک تابع را به عنوان **خروجی** (Return) برگرداند.

### رابطه Higher-Order Function با Delegate
در سی‌شارپ، چون Delegateها "نوعِ تابع" هستند، هر متدی که یک Delegate را ورودی بگیرد یا خروجی بدهد، یک Higher-Order Function است.

### استفاده از Func و Action
سی‌شارپ برای جلوگیری از تعریف Delegateهای تکراری، دو Delegate آماده (Built-in) دارد:
* **`Action`**: برای متدهایی که **خروجی ندارند** (void).
* **`Func`**: برای متدهایی که **خروجی دارند** (نوع خروجی آخرین Generic آن است).

```csharp
// Action: ورودی دو عدد، بدون خروجی
Action<int, int> printSum = (a, b) => Console.WriteLine(a + b);

// Func: ورودی دو عدد، خروجی int
Func<int, int, int> addFunc = (a, b) => a + b;
```

### استفاده از Higher-Order Function در LINQ
کتابخانه **LINQ** در سی‌شارپ کاملاً بر پایه Higher-Order Functions و Delegateها (معمولاً `Func`) ساخته شده است.

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// متد Where یک Func را به عنوان ورودی می‌گیرد (Higher-Order Function)
var evenNumbers = numbers.Where(n => n % 2 == 0); 
```

---

## 3. Multicast Delegate (دلیگیت‌های چندگانه)

### Multicast Delegate چیست؟
یک Delegate می‌تواند همزمان به **چندین متد** اشاره کند! به این حالت Multicast Delegate می‌گویند. وقتی Delegate را صدا می‌زنید، تمام متدهای ثبت‌شده به ترتیب اجرا می‌شوند.

### اضافه کردن (`+=`) و حذف (`-=`) متدها
برای اضافه کردن متد از `+=` و برای حذف از `-=` استفاده می‌کنیم.

```csharp
Action myDelegate = MethodA;
myDelegate += MethodB; // حالا هم A و هم B اجرا می‌شوند
myDelegate -= MethodA; // متد A حذف شد
```

### ترتیب اجرای Methodها
متدها دقیقاً به **همین ترتیبی که اضافه شده‌اند** (از چپ به راست / بالا به پایین) اجرا می‌شوند.

### Return Value و Exception در Multicast Delegate
* **Return Value:** اگر Delegate خروجی داشته باشد (مثلاً `Func<int>`)، فقط **آخرین متد** اجرا شده، مقدار خروجی‌اش برگردانده می‌شود و بقیه نادیده گرفته می‌شوند. (به همین دلیل پیشنهاد می‌شود Multicast Delegate را فقط برای متدهای `void` یا `Action` استفاده کنید).
* **Exception:** اگر یکی از متدها خطا (Exception) بدهد، اجرای بقیه متدها **متوقف می‌شود** (مگر اینکه خودتان آن را مدیریت کنید - در بخش Invocation List توضیح داده شده).

---

## 4. Invocation List (لیست فراخوانی)

### Invocation List چیست؟
در باطن، هر Multicast Delegate آرایه‌ای از Delegateها را نگه می‌دارد که به آن **Invocation List** می‌گویند. این لیست مشخص می‌کند کدام متدها باید اجرا شوند.

### GetInvocationList() و رابطه آن با Target و Method
با متد `GetInvocationList()` می‌توانیم به تک‌تک Delegateهای درون لیست دسترسی پیدا کنیم و `Target` و `Method` هر کدام را جداگانه ببینیم.

```csharp
Delegate[] delegates = myDelegate.GetInvocationList();
foreach (Delegate del in delegates)
{
    Console.WriteLine($"Target: {del.Target}, Method: {del.Method.Name}");
}
```

### اجرای اعضای Invocation List و مدیریت Exception
اگر بخواهیم جلوی توقف اجرا در صورت بروز خطا در یکی از متدها را بگیریم، باید Invocation List را دستی حلقه (Loop) کنیم:

```csharp
foreach (Action del in myDelegate.GetInvocationList())
{
    try {
        del(); // اجرای تک تک متدها
    } catch (Exception ex) {
        Console.WriteLine($"خطا در متد: {ex.Message}");
        // ادامه حلقه و اجرای متدهای بعدی
    }
}
```

### Invocation List در Eventها
رویدادها (Events) در سی‌شارپ در واقع همان Multicast Delegateها هستند. وقتی شما `+=` می‌زنید، در واقع دارید یک Delegate جدید به Invocation Listِ آن Event اضافه می‌کنید.

---

## 5. Immutability در Delegate (تغییرناپذیری)

### Immutable بودن Delegate یعنی چه؟
شیءهای Delegate در سی‌شارپ **Immutable (تغییرناپذیر)** هستند. یعنی وقتی یک Delegate ساخته می‌شود، **نمی‌توان** محتویات درونی آن را تغییر داد.

### چرا `+=` و `-=` Delegate را تغییر نمی‌دهند؟
شاید بپرسید: *"مگر `+=` محتویات Delegate را عوض نمی‌کند؟"*
خیر! وقتی شما `myDelegate += MethodB` می‌نویسید، در واقع Delegate قبلی تغییر نکرده است، بلکه **یک شیء Delegate کاملاً جدید** در حافظه ساخته شده که شامل متدهای Delegate قبلی + متد جدید است، و سپس **آدرس این شیء جدید** در متغیر `myDelegate` قرار می‌گیرد.

### Delegate جدید و Reference قبلی
```csharp
Action original = MethodA;
Action copy = original; // کپی از روی Reference

original += MethodB; // ساخت Delegate جدید

// نتیجه:
// original -> شامل MethodA و MethodB
// copy -> فقط شامل MethodA (چون Delegate قبلی تغییر نکرده بود)
```

### رابطه Immutability با Delegate.Combine و Delegate.Remove
عملگرهای `+=` و `-=` در واقع Syntactic Sugar (شکر syntactic) برای متدهای استاتیک زیر هستند:
* `+=` معادل `Delegate.Combine(del1, del2)` است (یک Delegate جدید برمی‌گرداند).
* `-=` معادل `Delegate.Remove(source, value)` است (یک Delegate جدید برمی‌گرداند).

---

## 6. Event (رویدادها) و فلسفه آن

### فلسفه Event چیست؟ (Publisher و Subscriber)
Event یک مکانیزم **Notification (اعلان)** است. 
* **Publisher (ناشر):** کلاسی که اتفاق مهمی در آن می‌افتد (مثل کلیک دکمه) و Event را **Fire (آزاد/صدا)** می‌کند.
* **Subscriber (مشترک):** کلاس‌هایی که به Event علاقه‌مندند و خود را ثبت نام می‌کنند تا از اتفاق باخبر شوند.

### تفاوت Event و Delegate (Encapsulation)
اگر Event فقط یک Delegate بود، هر کسی در کدهای خارجی می‌توانست `=` بگذارد و بقیه Subscriberها را پاک کند، یا خودش Event را Invoke کند!
**Event در واقع یک Wrapper (پوسته) دور Delegate است که Encapsulation را رعایت می‌کند.**

### چرا Publisher باید Fire کند و Subscriber فقط `+=` و `-=` دارد؟
کلمه کلیدی `event` دسترسی‌ها را محدود می‌کند:
1. کدهای **خارج از کلاس** فقط می‌توانند `+=` و `-=` کنند (ثبت و لغو اشتراک).
2. کدهای **خارج از کلاس** نمی‌توانند Event را `Invoke` کنند (صدا بزنند).
3. کدهای **خارج از کلاس** نمی‌توانند از `=` برای مقداردهی مستقیم استفاده کنند (چون لیست قبلی را نابود می‌کند).
4. فقط **داخل همان کلاس (Publisher)** می‌تواند Event را Invoke کند.

```csharp
public class Button {
    // تعریف Event
    public event Action OnClick; 

    public void Click() {
        OnClick?.Invoke(); // فقط اینجا می‌تواند صدا زده شود
    }
}

// در کلاس دیگر:
btn.OnClick += DoSomething; // ✅ مجاز
btn.OnClick();              // ❌ خطای کامپایل!
btn.OnClick = DoSomething;  // ❌ خطای کامپایل!
```

---

## 7. Event Accessors (دسترسی‌کننده‌های رویداد)

### Event Accessor چیست؟ (add و remove)
همانطور که Propertyها دارای `get` و `set` هستند، Eventها نیز می‌توانند دارای `add` و `remove` باشند. این به شما اجازه می‌دهد کنترل کاملی روی نحوه ثبت و لغو اشتراک Subscriberها داشته باشید.

* **`add`**: وقتی از `+=` استفاده می‌شود، کدهای درون `add` اجرا می‌شوند.
* **`remove`**: وقتی از `-=` استفاده می‌شود، کدهای درون `remove` اجرا می‌شوند.
* **متغیر `value`**: Delegateای که کاربر در حال اضافه یا کم کردن آن است، درون متغیر ضمنی `value` قرار می‌گیرد.

### Implicit vs Explicit Event Accessor
* **Implicit (ضمنی):** وقتی فقط می‌نویسید `public event Action MyEvent;`. کامپایلر خودش به صورت خودکار یک Delegate در پس‌زمینه می‌سازد و `add/remove` را پیاده‌سازی می‌کند.
* **Explicit (صریح):** وقتی خودتان `add` و `remove` را می‌نویسید.

### کاربردهای Explicit Event Accessor
1. **Forwarding Event به کلاس دیگر:** وقتی می‌خواهید Event یک کلاس داخلی را مستقیماً به بیرون暴露 (Expose) کنید.
2. **مدیریت حافظه (Memory Management):** اگر یک کلاس 100 تا Event داشته باشد، برای هر کدام یک فیلد Delegate در حافظه ساخته می‌شود. با Explicit Accessor می‌توانیم Delegateها را در یک **Dictionary** یا `EventHandlerList` ذخیره کنیم تا فقط در صورت ثبت شدن Subscriber، حافظه اشغال شود.

```csharp
// ذخیره در Dictionary برای مدیریت حافظه
private Dictionary<string, Delegate> _events = new();

public event Action MyEvent
{
    add {
        if (_events.ContainsKey("MyEvent"))
            _events["MyEvent"] = Delegate.Combine(_events["MyEvent"], value);
        else
            _events["MyEvent"] = value;
    }
    remove {
        if (_events.ContainsKey("MyEvent"))
            _events["MyEvent"] = Delegate.Remove(_events["MyEvent"], value);
    }
}
```

### Explicit Implementation برای Event در Interface
اگر دو Interface داشته باشید که هر دو یک Event با نام یکسان دارند، کلاس شما نمی‌تواند هر دو را به صورت معمولی پیاده‌سازی کند (چون تداخل نام دارند). باید از **Explicit Interface Implementation** برای Eventها استفاده کنید:

```csharp
public interface IA { event Action MyEvent; }
public interface IB { event Action MyEvent; }

public class MyClass : IA, IB
{
    private Action _eventA;
    private Action _eventB;

    // پیاده‌سازی صریح برای IA
    event Action IA.MyEvent 
    { 
        add => _eventA += value; 
        remove => _eventA -= value; 
    }

    // پیاده‌سازی صریح برای IB
    event Action IB.MyEvent 
    { 
        add => _eventB += value; 
        remove => _eventB -= value; 
    }
}
```

---

## 8. منابع معتبر

برای مطالعه بیشتر و عمیق‌تر، منابع زیر که از معتبرترین مراجع سی‌شارپ هستند پیشنهاد می‌شوند:

1. **مستندات رسمی مایکروسافت (Microsoft Learn):**
   * [Delegates (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
   * [Events (C# Programming Guide)](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/)
   * [How to: Raise and Consume Events](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/how-to-raise-and-consume-events)
2. **کتاب CLR via C# (نوشته Jeffrey Richter):**
   * فصل 17 (Delegates) و فصل 18 (Events) - *بهترین مرجع برای درک عمیق Immutability و Invocation List.*
3. **کتاب C# in Depth (نوشته Jon Skeet):**
   * بخش‌های مربوط به Delegate و Event - *عالی برای درک Higher-Order Functions و LINQ.*

---
