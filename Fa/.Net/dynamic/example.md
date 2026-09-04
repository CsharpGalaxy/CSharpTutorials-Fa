---

### ۱. کار با داده‌های JSON با ساختار متغیر یا ناشناخته
وقتی با APIهایی کار می‌کنید که ساختار پاسخ آن‌ها ثابت نیست یا بسیار تو در تو (Nested) است، ساختن ده‌ها کلاس DTO (Data Transfer Object) منطقی نیست. در اینجا `dynamic` (معمولاً همراه با `ExpandoObject` یا `JsonNode`) نجات‌بخش است.

```csharp
using System;
using System.Dynamic;
using System.Text.Json;
using System.Text.Json.Nodes;

public class JsonExample
{
    public static void Run()
    {
        string jsonResponse = @"{
            ""user"": {
                ""name"": ""Ali"",
                ""roles"": [""Admin"", ""Developer""],
                ""metadata"": { ""lastLogin"": ""2023-10-01"" }
            }
        }";

        // تبدیل JsonNode به dynamic برای دسترسی راحت‌تر
        dynamic data = JsonNode.Parse(jsonResponse)!;

        // دسترسی مستقیم و خوانا بدون نیاز به کلاس‌های تو در تو
        Console.WriteLine($"Name: {data.user.name}");
        Console.WriteLine($"First Role: {data.user.roles[0]}");
        Console.WriteLine($"Last Login: {data.user.metadata.lastLogin}");
    }
}
```
* **چرا اینجا خوب است؟** چون از نوشتن ۳ کلاس جداگانه (`UserResponse`, `User`, `Metadata`) جلوگیری می‌کند و کد بسیار خوانا می‌شود.

---

### ۲. خودکارسازی نرم‌افزارهای Office (COM Interop)
این **دلیل اصلی معرفی `dynamic` در C# 4.0** بود. کار با COM (مثل Excel یا Word) بدون `dynamic` کابوسی از `ref object` و `Type.Missing` بود.

```csharp
using System;
// نیاز به نصب NuGet: Microsoft.Office.Interop.Excel

public class ExcelAutomation
{
    public static void Run()
    {
        // ایجاد نمونه از اکسل
        var excelApp = new Microsoft.Office.Interop.Excel.Application();
        excelApp.Visible = true;

        // استفاده از dynamic برای جلوگیری از کست‌های زشت و پارامترهای ref
        dynamic app = excelApp;
        dynamic workbook = app.Workbooks.Add();
        dynamic worksheet = workbook.ActiveSheet;

        // نوشتن در سلول بدون نیاز به C# 4.0 style: 
        // worksheet.Range["A1"].set_Value(Type.Missing, "Hello World");
        worksheet.Cells[1, 1].Value = "سلام دنیا!";
        worksheet.Cells[1, 2].Value = 100;

        Console.WriteLine("Excel updated successfully.");
    }
}
```
* **چرا اینجا خوب است؟** کامپایلر C# نمی‌تواند نوع دقیق اشیای COM را در زمان کامپایل بداند. `dynamic` این پیچیدگی را پنهان می‌کند.

---

### ۳. ساده‌سازی Reflection در سیستم‌های پلاگین‌محور
فرض کنید یک سیستم پلاگین دارید که اسمبل‌های خارجی را بارگذاری می‌کند. می‌دانید که همه پلاگین‌ها یک متد به نام `Execute` دارند، اما نمی‌خواهید همه آن‌ها را مجبور به پیاده‌سازی یک اینترفیس مشترک کنید (یا به آن دسترسی ندارید).

```csharp
using System;
using System.Reflection;

public class PluginSystem
{
    public static void Run()
    {
        // فرض کنید این نوع از یک DLL خارجی بارگذاری شده است
        Type pluginType = typeof(ExternalCalculator); 
        object instance = Activator.CreateInstance(pluginType)!;

        // روش قدیمی با Reflection (پیچیده و کند)
        // MethodInfo method = pluginType.GetMethod("Execute");
        // method.Invoke(instance, new object[] { 10, 20 });

        // روش مدرن با dynamic (تمیز و خوانا)
        dynamic plugin = instance;
        int result = plugin.Execute(10, 20);
        
        Console.WriteLine($"Result: {result}");
    }
}

// شبیه‌سازی یک کلاس در اسمبلی خارجی
public class ExternalCalculator
{
    public int Execute(int a, int b) => a + b;
}
```
* **چرا اینجا خوب است؟** خوانایی کد را به‌شدت افزایش می‌دهد و سربار نوشتن کدهای تکراری `MethodInfo.Invoke` را حذف می‌کند.

---

### ۴. ساخت اشیای پویا در زمان اجرا (ExpandoObject)
زمانی که نیاز دارید یک شیء بسازید که ویژگی‌های (Properties) آن در زمان اجرا و بر اساس ورودی کاربر تعیین می‌شود (مثلاً در یک Form Builder یا سیستم گزارش‌گیری پویا).

```csharp
using System;
using System.Dynamic;
using System.Collections.Generic;

public class DynamicFormExample
{
    public static void Run()
    {
        // ساخت یک شیء کاملاً پویا
        dynamic formData = new ExpandoObject();

        // اضافه کردن ویژگی‌ها در زمان اجرا
        formData.FirstName = "Reza";
        formData.LastName = "Mohammadi";
        formData.Age = 35;
        
        // حتی می‌توان لیست یا شیء دیگر به آن اضافه کرد
        formData.Skills = new List<string> { "C#", "SQL", "Azure" };

        // تبدیل به دیکشنری برای ذخیره در دیتابیس یا ارسال به API
        var dict = (IDictionary<string, object>)formData;
        
        Console.WriteLine($"User {dict["FirstName"]} has {((List<string>)dict["Skills"]).Count} skills.");
    }
}
```
* **چرا اینجا خوب است؟** `ExpandoObject` پیاده‌سازی `IDynamicMetaObjectProvider` را برای شما انجام داده و اجازه می‌دهد مانند یک کلاس واقعی با دیکشنری داده‌ها کار کنید.

---

### ۵. تعامل با زبان‌های اسکریپت‌نویسی (مثل Python)
اگر در پروژه .NET خود نیاز به اجرای کد Python (مثلاً برای هوش مصنوعی یا پردازش داده) دارید، کتابخانه‌هایی مثل `IronPython` یا `Python.NET` از `dynamic` برای یکپارچه‌سازی استفاده می‌کنند.

```csharp
using System;
// نیاز به NuGet: pythonnet (Python.Runtime)

public class PythonInterop
{
    public static void Run()
    {
        using (Py.GIL()) // قفل کردن Global Interpreter Lock پایتون
        {
            // بارگذاری یک ماژول پایتون (فرضی)
            dynamic mathModule = Py.Import("math");
            
            // فراخوانی مستقیم توابع پایتون از داخل C#
            double result = mathModule.sqrt(25);
            Console.WriteLine($"Square root of 25 is: {result}");

            // یا فراخوانی یک اسکریپت سفارشی
            dynamic customScript = Py.Import("my_data_script");
            var data = customScript.ProcessData(new[] { 1, 2, 3, 4 });
        }
    }
}
```
* **چرا اینجا خوب است؟** مرز بین دو زبان (ایستا و پویا) را از بین می‌برد و فراخوانی توابع زبان مقصد را بومی (Native) جلوه می‌دهد.

---

### ⚠️ هشدار مهم: کجا از آن استفاده **نکنیم**؟
برای درک بهتر، بدانید این مثال‌ها **بد** هستند:
1. **جایگزین کردن Interface:** اگر همه کلاس‌ها متد `Save()` دارند، یک اینترفیس `ISaveable` بسازید، نه اینکه همه را `dynamic` کنید.
2. **جایگزین کردن Generics:** به جای نوشتن `public void Process(dynamic data)`، از `public void Process<T>(T data)` استفاده کنید تا Type Safety حفظ شود.
3. **منطق تجاری پیچیده (Business Logic):** هرگز قوانین اصلی برنامه (مثل محاسبه مالیات یا اعتبارسنجی کاربر) را بر اساس `dynamic` ننویسید، زیرا خطاهای آن فقط در Runtime مشخص می‌شود و Refactoring را غیرممکن می‌کند.

**خلاصه:** از `dynamic` به عنوان یک **ابزار تخصصی برای لبه‌های سیستم** (ورودی/خروجی ناشناخته، COM، Reflection) استفاده کنید، نه به عنوان جایگزینی برای طراحی خوب شیءگرا.