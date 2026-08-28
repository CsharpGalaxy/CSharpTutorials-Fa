# انواع Record در C# - راهنمای کامل

## مقدمه

Records در C# 9.0 معرفی شدند و نوع خاصی از type هستند که برای مدل‌سازی داده‌های immutable طراحی شده‌اند. در C# 10، قابلیت‌های record گسترش یافت و انواع مختلفی از آن اضافه شد.

---

## 1. Record (Record Class)

### تعریف
```csharp
public record Person(string FirstName, string LastName, int Age);
```

### ویژگی‌ها

**Reference Type یا Value Type:**
- Reference Type است (در heap ذخیره می‌شود)
- متغیرها reference به object را نگه می‌دارند

**Equality:**
- Value-based equality دارد (نه reference equality)
- کامپایلر متد `Equals`، `GetHashCode` و `==` operator را به صورت خودکار تولید می‌کند
- مقایسه عمیق (deep equality) روی تمام propertyها انجام می‌شود

**Immutability:**
- به طور پیش‌فرض immutable است
- Propertyها readonly هستند (init-only setter)
- نمی‌توانید بعد از ساخت، مقدار propertyها را تغییر دهید

**Copy Semantics:**
- از `with` expression برای ایجاد کپی استفاده می‌شود
- Shallow copy ایجاد می‌کند
```csharp
var person1 = new Person("John", "Doe", 30);
var person2 = person1 with { Age = 31 }; // کپی با تغییر Age
```

**Inheritance:**
- از کلاس‌های دیگر می‌تواند ارث‌بری کند
- می‌تواند sealed باشد
- از interface می‌تواند implement کند

**Property Behavior:**
- Positional parameters به صورت init-only property تبدیل می‌شوند
- می‌توانید propertyهای اضافی با getter تعریف کنید

**کاربرد مناسب:**
- مدل‌سازی domain entities
- DTO (Data Transfer Objects)
- Configuration objects
- هر جایی که نیاز به value equality دارید

**محدودیت‌ها:**
- نمی‌تواند از چند کلاس ارث‌بری کند
- Performance overhead به دلیل heap allocation
- GC pressure بیشتر نسبت به struct

---

## 2. Record Class (صریح)

### تعریف
```csharp
public record class Person(string FirstName, string LastName, int Age);
```

### ویژگی‌ها
- دقیقاً همان رفتار `record` ساده را دارد
- فقط به صورت صریح مشخص می‌کند که این یک reference type است
- برای وضوح بیشتر در کد استفاده می‌شود

**تفاوت با `record` ساده:**
- هیچ تفاوت عملکردی ندارد
- فقط برای خوانایی بهتر است

---

## 3. Record Struct

### تعریف
```csharp
public record struct Point(double X, double Y);
```

### ویژگی‌ها

**Reference Type یا Value Type:**
- Value Type است (در stack ذخیره می‌شود)
- Performance بهتر برای داده‌های کوچک

**Equality:**
- Value-based equality دارد
- کامپایلر `Equals` و `GetHashCode` را تولید می‌کند
- مقایسه سریع‌تر از record class

**Immutability:**
- **به طور پیش‌فرض mutable است!**
- Propertyها دارای setter هستند (نه init-only)
- می‌توانید مقدار propertyها را تغییر دهید

**Copy Semantics:**
- `with` expression پشتیبانی می‌شود
- Copy by value انجام می‌شود
```csharp
var point1 = new Point(10, 20);
var point2 = point1 with { X = 30 };
```

**Inheritance:**
- نمی‌تواند از struct دیگر ارث‌بری کند
- فقط می‌تواند interface implement کند
- نمی‌تواند sealed باشد (structها به طور پیش‌فرض sealed هستند)

**Property Behavior:**
- Positional parameters به صورت property با getter و setter تبدیل می‌شوند
- می‌توانید propertyها را تغییر دهید

**کاربرد مناسب:**
- داده‌های کوچک و ساده
- Performance-critical scenarios
- وقتی نیاز به value type دارید
- Geometric objects (Point, Vector, etc.)

**محدودیت‌ها:**
- نمی‌تواند ارث‌بری کند
- Boxing ممکن است اتفاق بیفتد
- برای داده‌های بزرگ مناسب نیست

---

## 4. Readonly Record Struct

### تعریف
```csharp
public readonly record struct Point(double X, double Y);
```

### ویژگی‌ها

**Reference Type یا Value Type:**
- Value Type است
- تمام فیلدها به طور ضمنی readonly هستند

**Equality:**
- Value-based equality دارد
- عملکرد مشابه record struct

**Immutability:**
- **کاملاً immutable است**
- تمام propertyها init-only هستند
- کامپایلر خطا می‌دهد اگر سعی کنید propertyها را تغییر دهید

**Copy Semantics:**
- `with` expression پشتیبانی می‌شود
- Copy by value

**Inheritance:**
- همان محدودیت‌های record struct

**Property Behavior:**
- Propertyها readonly هستند
- فقط getter دارند

**کاربرد مناسب:**
- Immutable value types
- Thread-safe data structures
- وقتی نیاز به value type با immutability دارید
- Functional programming patterns

**محدودیت‌ها:**
- همان محدودیت‌های record struct
- نمی‌توان propertyها را تغییر داد

---

## جدول مقایسه کامل

| ویژگی | record | record class | record struct | readonly record struct |
|-------|--------|--------------|---------------|------------------------|
| **Type** | Reference Type | Reference Type | Value Type | Value Type |
| **Storage** | Heap | Heap | Stack | Stack |
| **Equality** | Value-based | Value-based | Value-based | Value-based |
| **Immutability** | Immutable | Immutable | **Mutable** | Immutable |
| **Property Setters** | init-only | init-only | set | init-only |
| **Inheritance** | بله | بله | خیر | خیر |
| **Copy Syntax** | `with` | `with` | `with` | `with` |
| **Performance** | متوسط | متوسط | **عالی** | **عالی** |
| **GC Pressure** | بله | بله | خیر | خیر |
| **Default Behavior** | پیش‌فرض | صریح | صریح | صریح |
| **Introduced In** | C# 9.0 | C# 10 | C# 10 | C# 10 |

---

## مثال‌های عملی

### مثال 1: Record Class
```csharp
// تعریف
public record Person(string FirstName, string LastName, int Age)
{
    // Property اضافی
    public string FullName => $"{FirstName} {LastName}";
}

// استفاده
var person1 = new Person("John", "Doe", 30);
var person2 = new Person("John", "Doe", 30);

Console.WriteLine(person1 == person2); // True (value equality)
Console.WriteLine(person1.FullName); // John Doe

// Copy with modification
var person3 = person1 with { Age = 31 };
Console.WriteLine(person3.Age); // 31
Console.WriteLine(person1.Age); // 30 (original unchanged)

// person1.Age = 32; // Error: cannot modify init-only property
```

### مثال 2: Record Class (صریح)
```csharp
public record class Customer(string Name, string Email);

var customer1 = new Customer("Alice", "alice@example.com");
var customer2 = customer1 with { Email = "alice.new@example.com" };

Console.WriteLine(customer1.Email); // alice@example.com
Console.WriteLine(customer2.Email); // alice.new@example.com
```

### مثال 3: Record Struct (Mutable)
```csharp
public record struct Point(double X, double Y)
{
    public double DistanceFromOrigin => Math.Sqrt(X * X + Y * Y);
}

var point1 = new Point(3, 4);
Console.WriteLine(point1.DistanceFromOrigin); // 5

// می‌توان propertyها را تغییر داد
point1.X = 10; // OK!
Console.WriteLine(point1.X); // 10

var point2 = point1 with { Y = 20 };
Console.WriteLine(point2.Y); // 20
```

### مثال 4: Readonly Record Struct (Immutable)
```csharp
public readonly record struct Vector3D(double X, double Y, double Z)
{
    public double Magnitude => Math.Sqrt(X * X + Y * Y + Z * Z);
    
    public Vector3D Normalize()
    {
        var mag = Magnitude;
        return new Vector3D(X / mag, Y / mag, Z / mag);
    }
}

var vector1 = new Vector3D(1, 2, 3);
Console.WriteLine(vector1.Magnitude); // 3.74...

// vector1.X = 10; // Error: cannot modify readonly property

var vector2 = vector1 with { X = 10 };
Console.WriteLine(vector2.X); // 10
Console.WriteLine(vector1.X); // 1 (original unchanged)
```

---

## توضیح سینتکس‌ها

### `public record Point;`
- این یک record class است
- کلمه `class` به صورت ضمنی در نظر گرفته می‌شود
- معادل `public record class Point;` است

### `public record class Point;`
- صریحاً مشخص می‌کند که این یک reference type است
- برای وضوح بیشتر استفاده می‌شود
- همان رفتار `record` ساده را دارد

### `public record struct Point;`
- مشخصاً یک value type است
- **Mutable** است (propertyها setter دارند)
- برای performance-critical scenarios مناسب است

### `public readonly record struct Point;`
- Value type با immutability کامل
- تمام propertyها readonly هستند
- بهترین گزینه برای immutable value types

---

## چرا `record` به طور پیش‌فرض Record Class است؟

### دلایل طراحی:

1. **سازگاری با کلاس‌های موجود:**
   - بیشتر use caseها نیاز به reference type دارند
   - رفتار مشابه کلاس‌های سنتی با value equality

2. **Immutability پیش‌فرض:**
   - Reference types به طور پیش‌فرض immutable هستند
   - این با فلسفه functional programming همخوانی دارد

3. **Inheritance:**
   - Record classها می‌توانند ارث‌بری کنند
   - انعطاف‌پذیری بیشتر برای domain modeling

4. **سابقه تاریخی:**
   - در C# 9.0 فقط record class وجود داشت
   - در C# 10.0 record struct اضافه شد
   - برای backward compatibility، `record` ساده همان record class باقی ماند

5. **Use caseهای رایج:**
   - DTOها، domain entities، configuration objects
   - اکثر این موارد reference type هستند

---

## راهنمای انتخاب

### از `record` یا `record class` استفاده کنید وقتی:

✅ نیاز به reference type دارید  
✅ می‌خواهید از inheritance استفاده کنید  
✅ داده‌های شما پیچیده است  
✅ نیاز به immutability دارید  
✅ Performance خیلی حیاتی نیست  
✅ می‌خواهید null reference داشته باشید  

**مثال‌ها:**
- User, Customer, Order entities
- DTOs برای API
- Configuration objects
- Domain models

### از `record struct` استفاده کنید وقتی:

✅ Performance خیلی مهم است  
✅ داده‌های شما کوچک و ساده است  
✅ نیازی به inheritance ندارید  
✅ می‌خواهید mutable باشد  
✅ نمی‌خواهید GC pressure داشته باشید  

**مثال‌ها:**
- Point, Vector, Rectangle
- Complex numbers
- Small data structures
- Performance-critical calculations

### از `readonly record struct` استفاده کنید وقتی:

✅ نیاز به value type دارید  
✅ می‌خواهید immutable باشد  
✅ Thread safety مهم است  
✅ Functional programming استفاده می‌کنید  
✅ Performance مهم است  

**مثال‌ها:**
- Immutable geometric types
- Mathematical vectors
- Configuration values
- Value objects در DDD

---

## نکات مهم

### 1. Deconstruction
همه انواع record از deconstruction پشتیبانی می‌کنند:
```csharp
var person = new Person("John", "Doe", 30);
var (firstName, lastName, age) = person;
```

### 2. Pattern Matching
همه انواع record با pattern matching کار می‌کنند:
```csharp
if (person is Person("John", "Doe", var age))
{
    Console.WriteLine($"Age: {age}");
}
```

### 3. Custom Equality
می‌توانید equality را override کنید:
```csharp
public record Person(string FirstName, string LastName, int Age)
{
    public virtual bool Equals(Person? other)
    {
        if (other is null) return false;
        return FirstName == other.FirstName && LastName == other.LastName;
        // Age را در مقایسه لحاظ نمی‌کنیم
    }
}
```

### 4. Print Members
می‌توانید `PrintMembers` را override کنید:
```csharp
public record Person(string FirstName, string LastName, int Age)
{
    private bool PrintMembers(StringBuilder builder)
    {
        builder.Append($"Name: {FirstName} {LastName}");
        return true;
    }
}
```

---

## منابع رسمی

### Microsoft Learn:
- [Record types (C# Reference)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- [Records and positional parameters](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#positional-syntax-for-property-definition)
- [Nondestructive mutation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#nondestructive-mutation)
- [Value equality](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#value-equality)

### C# Language Specification:
- [C# Language Specification - Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/records)
- [C# 10.0 Specification - Record Structs](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-10.0/record-structs)

### مقالات مرتبط:
- [What's new in C# 10.0](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10)
- [What's new in C# 9.0](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9)

---

## نتیجه‌گیری

Records در C# ابزار قدرتمندی برای مدل‌سازی داده‌ها هستند. انتخاب بین انواع مختلف record بستگی به نیازهای شما دارد:

- **record / record class**: برای domain models و DTOها
- **record struct**: برای performance-critical scenarios با mutable data
- **readonly record struct**: برای immutable value types با performance بالا

همیشه immutability را به عنوان پیش‌فرض در نظر بگیرید و فقط وقتی mutable بودن لازم است، از `record struct` استفاده کنید.