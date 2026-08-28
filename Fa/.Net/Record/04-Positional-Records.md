# اعضای تولیدشده توسط Compiler در Recordها

## مقدمه

Recordها در C# 9 معرفی شدند و در C# 10 و نسخه‌های بعدی تکامل یافتند. هدف اصلی آن‌ها ارائه یک روش مختصر و امن برای تعریف کلاس‌هایی است که عمدتاً داده نگهداری می‌کنند. نکته جالب اینجاست که وقتی شما یک Record تعریف می‌کنید، Compiler به طور خودکار مجموعه‌ای از اعضا را برای شما تولید می‌کند تا رفتار value-based equality، immutability و الگوهای رایج را پیاده‌سازی کند.

در این مقاله، این اعضای تولیدشده (Synthesized Members) را به صورت عمیق بررسی می‌کنیم.

---

## یک Record ساده برای بررسی

برای درک بهتر، یک Record ساده تعریف می‌کنیم:

```csharp
public record Person(string FirstName, string LastName);
```

این تعریف مختصر، در واقع منجر به تولید کد بسیار بیشتری می‌شود. در ادامه نسخه مفهومی (Conceptual) کد تولیدشده را بررسی می‌کنیم.

> **توجه مهم:** کدی که در ادامه می‌بینید، معادل دقیق Source Code تولیدشده توسط Compiler نیست. برخی از این اعضا کاملاً compiler-synthesized هستند و در IL به صورت مستقیم وجود دارند، اما برای درک رفتار آن‌ها، معادل مفهومی آن‌ها را نمایش می‌دهیم.

---

## نسخه مفهومی اعضای تولیدشده

```csharp
public class Person : IEquatable<Person>
{
    // 1. Init Properties
    public string FirstName { get; init; }
    public string LastName { get; init; }

    // 2. Primary Constructor
    public Person(string FirstName, string LastName)
    {
        this.FirstName = FirstName;
        this.LastName = LastName;
    }

    // 3. EqualityContract
    protected virtual Type EqualityContract => typeof(Person);

    // 4. PrintMembers (کمکی برای ToString)
    protected virtual bool PrintMembers(StringBuilder builder)
    {
        builder.Append(", FirstName = ");
        builder.Append(FirstName);
        builder.Append(", LastName = ");
        builder.Append(LastName);
        return true;
    }

    // 5. ToString
    public override string ToString()
    {
        StringBuilder builder = new StringBuilder();
        builder.Append("Person");
        if (PrintMembers(builder))
        {
            return builder.ToString();
        }
        return "Person";
    }

    // 6. Deconstruct
    public void Deconstruct(out string FirstName, out string LastName)
    {
        FirstName = this.FirstName;
        LastName = this.LastName;
    }

    // 7. Clone (Copy Constructor)
    protected Person(Person original)
    {
        this.FirstName = original.FirstName;
        this.LastName = original.LastName;
    }

    public Person Clone() => new Person(this);

    // 8. Equals
    public virtual bool Equals(Person? other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;
        if (EqualityContract != other.EqualityContract) return false;
        return EqualityComparer<string>.Default.Equals(FirstName, other.FirstName)
            && EqualityComparer<string>.Default.Equals(LastName, other.LastName);
    }

    public override bool Equals(object? obj) => Equals(obj as Person);

    // 9. GetHashCode
    public override int GetHashCode()
    {
        return HashCode.Combine(EqualityContract, 
            EqualityComparer<string>.Default.GetHashCode(FirstName),
            EqualityComparer<string>.Default.GetHashCode(LastName));
    }

    // 10. == Operator
    public static bool operator ==(Person? left, Person? right)
    {
        if (left is null) return right is null;
        return left.Equals(right);
    }

    // 11. != Operator
    public static bool operator !=(Person? left, Person? right)
        => !(left == right);
}
```

---

## بررسی دقیق هر عضو

### 1. Primary Constructor

**چیست؟**
Constructor اصلی که از positional parameters تعریف Record تولید می‌شود.

**چرا تولید می‌شود؟**
برای مقداردهی اولیه properties و تضمین اینکه تمام داده‌های ضروری هنگام ساخت object فراهم باشند.

**چه زمانی استفاده می‌شود؟**
هنگامی که با `new Person("Ali", "Rezaei")` یک نمونه جدید می‌سازید.

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید constructorهای اضافی تعریف کنید، اما constructor اصلی را نمی‌توانید حذف کنید.

**محدودیت‌ها:**
- Constructor اصلی همیشه public است
- نمی‌توانید آن را private یا protected کنید
- می‌توانید constructorهای اضافی با امضاهای متفاوت اضافه کنید

---

### 2. Init Properties

**چیست؟**
Properties با accessor `init` که فقط هنگام ساخت object قابل مقداردهی هستند.

**چرا تولید می‌شود؟**
برای تضمین immutability پس از ساخت object. این یکی از ویژگی‌های کلیدی Recordهاست.

**چه زمانی استفاده می‌شود؟**
هنگام دسترسی به properties برای خواندن یا هنگام استفاده از object initializer.

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید properties را با `get; init;` یا `get; set;` تعریف کنید.

**محدودیت‌ها:**
- اگر از positional parameters استفاده کنید، properties به طور پیش‌فرض `init` هستند
- می‌توانید با تعریف مجدد property، آن را به `set` تغییر دهید، اما این immutability را از بین می‌برد
- نمی‌توانید accessor `init` را به `private` تغییر دهید

```csharp
public record Person
{
    // سفارشی‌سازی property
    public string FirstName { get; init; }
    public string LastName { get; set; } // تغییر به mutable
}
```

---

### 3. Deconstruct

**چیست؟**
متدی که امکان tuple deconstruction را فراهم می‌کند.

**چرا تولید می‌شود؟**
برای پشتیبانی از pattern matching و استخراج آسان مقادیر properties.

**چه زمانی استفاده می‌شود؟**
هنگامی که از deconstruction syntax استفاده می‌کنید:

```csharp
var person = new Person("Ali", "Rezaei");
var (firstName, lastName) = person;
```

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید Deconstruct متدهای اضافی با پارامترهای متفاوت تعریف کنید.

**محدودیت‌ها:**
- Deconstruct تولیدشده فقط properties اصلی را شامل می‌شود
- نمی‌توانید Deconstruct اصلی را حذف کنید
- می‌توانید overloads اضافی اضافه کنید

---

### 4. ToString

**چیست؟**
متدی که نمایش رشته‌ای از Record را برمی‌گرداند.

**چرا تولید می‌شود؟**
برای ارائه خروجی خوانا و مفید برای debugging و logging.

**چه زمانی استفاده می‌شود؟**
هنگامی که Record را به string تبدیل می‌کنید یا در Console.WriteLine استفاده می‌کنید.

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید ToString را override کنید.

**محدودیت‌ها:**
- اگر ToString را override کنید، نسخه تولیدشده توسط Compiler جایگزین می‌شود
- برای سفارشی‌سازی جزئی، می‌توانید `PrintMembers` را override کنید

```csharp
public record Person(string FirstName, string LastName)
{
    // سفارشی‌سازی ToString
    public override string ToString() => $"{FirstName} {LastName}";
    
    // یا سفارشی‌سازی PrintMembers
    protected virtual bool PrintMembers(StringBuilder builder)
    {
        builder.Append($" ({FirstName}, {LastName})");
        return true;
    }
}
```

خروجی ToString تولیدشده:
```
Person { FirstName = Ali, LastName = Rezaei }
```

---

### 5. Equals

**چیست؟**
متدی که equality بر اساس value را پیاده‌سازی می‌کند.

**چرا تولید می‌شود؟**
Recordها برای value equality طراحی شده‌اند، نه reference equality.

**چه زمانی استفاده می‌شود؟**
هنگام مقایسه دو Record با `Equals()` یا در collectionها مانند `Dictionary` و `HashSet`.

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید Equals را override کنید.

**محدودیت‌ها:**
- اگر Equals را override کنید، باید GetHashCode را هم override کنید
- باید `IEquatable<T>` را پیاده‌سازی کنید
- باید EqualityContract را بررسی کنید

```csharp
public record Person(string FirstName, string LastName)
{
    public virtual bool Equals(Person? other)
    {
        if (other is null) return false;
        if (ReferenceEquals(this, other)) return true;
        if (EqualityContract != other.EqualityContract) return false;
        
        // منطق سفارشی
        return FirstName.ToLower() == other.FirstName.ToLower()
            && LastName.ToLower() == other.LastName.ToLower();
    }
}
```

---

### 6. GetHashCode

**چیست؟**
متدی که hash code بر اساس تمام properties تولید می‌کند.

**چرا تولید می‌شود؟**
برای استفاده صحیح در hash-based collections و تضمین سازگاری با Equals.

**چه زمانی استفاده می‌شود؟**
هنگام استفاده از Record به عنوان key در Dictionary یا عضو در HashSet.

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید GetHashCode را override کنید.

**محدودیت‌ها:**
- باید با Equals سازگار باشد
- اگر دو object برابر هستند، hash code آن‌ها باید یکسان باشد
- باید از تمام properties استفاده کنید که در Equals استفاده شده‌اند

```csharp
public record Person(string FirstName, string LastName)
{
    public override int GetHashCode()
    {
        return HashCode.Combine(
            FirstName.ToLower().GetHashCode(),
            LastName.ToLower().GetHashCode()
        );
    }
}
```

---

### 7. == و != Operators

**چیست؟**
Operatorهای مقایسه که از Equals استفاده می‌کنند.

**چرا تولید می‌شوند؟**
برای پشتیبانی از syntax مقایسه طبیعی و استفاده از value equality.

**چه زمانی استفاده می‌شوند؟**
هنگام مقایسه دو Record با `==` یا `!=`.

**آیا قابل سفارشی‌سازی هستند؟**
خیر، نمی‌توانید این operatorها را در Record سفارشی کنید.

**محدودیت‌ها:**
- این operatorها همیشه از Equals استفاده می‌کنند
- نمی‌توانید رفتار آن‌ها را تغییر دهید
- اگر نیاز به رفتار متفاوت دارید، باید از class معمولی استفاده کنید

```csharp
var person1 = new Person("Ali", "Rezaei");
var person2 = new Person("Ali", "Rezaei");
var areEqual = person1 == person2; // true - value equality
```

---

### 8. EqualityContract

**چیست؟**
Property محافظت‌شده‌ای که نوع Record را برای equality checking برمی‌گرداند.

**چرا تولید می‌شود؟**
برای پشتیبانی از inheritance و تضمین اینکه دو object از نوع یکسان هستند.

**چه زمانی استفاده می‌شود؟**
در متد Equals برای بررسی اینکه دو object از نوع یکسان هستند.

**آیا قابل سفارشی‌سازی است؟**
بله، در derived records به طور خودکار override می‌شود.

**محدودیت‌ها:**
- این property protected است و مستقیماً قابل دسترسی نیست
- در derived records به طور خودکار override می‌شود
- نمی‌توانید آن را حذف کنید

```csharp
public record Employee(string FirstName, string LastName, string Company) 
    : Person(FirstName, LastName);

// EqualityContract در Employee به typeof(Employee) اشاره می‌کند
// EqualityContract در Person به typeof(Person) اشاره می‌کند
```

---

### 9. Copy Constructor (Clone)

**چیست؟**
Constructor محافظت‌شده‌ای که یک کپی از Record ایجاد می‌کند.

**چرا تولید می‌شود؟**
برای پشتیبانی از `with` expression و ایجاد کپی‌های اصلاح‌شده.

**چه زمانی استفاده می‌شود؟**
هنگام استفاده از `with` expression:

```csharp
var person1 = new Person("Ali", "Rezaei");
var person2 = person1 with { LastName = "Mohammadi" };
```

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید Copy Constructor را override کنید.

**محدودیت‌ها:**
- Copy Constructor باید protected باشد
- باید تمام properties را کپی کند
- در derived records باید base copy constructor را فراخوانی کند

```csharp
public record Person(string FirstName, string LastName)
{
    // سفارشی‌سازی Copy Constructor
    protected Person(Person original)
    {
        FirstName = original.FirstName;
        LastName = original.LastName;
        // منطق سفارشی
    }
}
```

---

### 10. Clone

**چیست؟**
متد عمومی که یک کپی از Record برمی‌گرداند.

**چرا تولید می‌شود؟**
برای ارائه یک API عمومی برای کپی کردن Record.

**چه زمانی استفاده می‌شود؟**
هنگامی که نیاز به ایجاد کپی مستقل از Record دارید.

**آیا قابل سفارشی‌سازی است؟**
بله، می‌توانید Clone را override کنید.

**محدودیت‌ها:**
- Clone باید از Copy Constructor استفاده کند
- نمی‌توانید signature آن را تغییر دهید
- در derived records باید نوع صحیح را برگرداند

```csharp
public record Person(string FirstName, string LastName)
{
    public Person Clone() => new Person(this);
}
```

---

## نکات مهم درباره Synthesized Members

### 1. کد تولیدشده واقعی

کدی که در بالا دیدید، معادل مفهومی است. در واقعیت:
- برخی اعضا به صورت IL code تولید می‌شوند
- Compiler optimizations ممکن است اعمال شود
- برخی متدها ممکن است inline شوند

### 2. سفارشی‌سازی اعضا

شما می‌توانید هر یک از این اعضا را override کنید، اما:
- باید contract اصلی را حفظ کنید
- برای Equals و GetHashCode، سازگاری ضروری است
- برای operatorها، نمی‌توانید رفتار را تغییر دهید

### 3. Inheritance

هنگام inheritance از Record:
- EqualityContract به طور خودکار override می‌شود
- Copy Constructor باید base constructor را فراخوانی کند
- PrintMembers باید base PrintMembers را فراخوانی کند

```csharp
public record Employee(string FirstName, string LastName, string Company) 
    : Person(FirstName, LastName)
{
    protected override bool PrintMembers(StringBuilder builder)
    {
        if (base.PrintMembers(builder))
            builder.Append(", ");
        builder.Append($"Company = {Company}");
        return true;
    }
}
```

---

## جمع‌بندی

Recordها با تولید خودکار این اعضا، تجربه توسعه‌دهنده را بهبود می‌بخشند:

- **Immutability:** از طریق init properties
- **Value Equality:** از طریق Equals, GetHashCode, ==, !=
- **Pattern Matching:** از طریق Deconstruct
- **Easy Copying:** از طریق Copy Constructor و with expression
- **Debugging:** از طریق ToString

این اعضای تولیدشده، Recordها را به انتخابی عالی برای data transfer objects (DTOs)، configuration objects و هر جایی که نیاز به value semantics دارید، تبدیل می‌کنند.

---

## منابع

- [Record Types - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- [Records - C# Language Specification](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/records)
- [What's new in C# 9.0 - Record Types](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#record-types)
- [Record Struct Types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record-struct)