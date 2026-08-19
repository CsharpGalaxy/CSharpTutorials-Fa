## فهرست مطالب

- [1. ImageMagick چیست؟](#1-imagemagick-چیست)
- [2. در C# از چه کتابخانه‌ای استفاده کنیم؟](#2-در-c-از-چه-کتابخانهای-استفاده-کنیم)
- [3. نصب و راه‌اندازی Magick.NET](#3-نصب-و-راهاندازی-magicknet)
- [4. انتخاب پکیج مناسب: Q8، Q16، HDRI، OpenMP](#4-انتخاب-پکیج-مناسب)
- [5. اولین برنامه: خواندن، ذخیره و تبدیل تصویر](#5-اولین-برنامه-خواندن-ذخیره-و-تبدیل-تصویر)
- [6. کار با Stream و Byte Array](#6-کار-با-stream-و-byte-array)
- [7. شناسایی تصویر بدون بارگذاری کامل پیکسل‌ها](#7-شناسایی-تصویر-بدون-بارگذاری-کامل-پیکسلها)
- [8. تغییر اندازه، برش، Trim و Extent](#8-تغییر-اندازه-برش-trim-و-extent)
- [9. چرخش، آینه، Auto-Orient و جهت تصویر](#9-چرخش-آینه-auto-orient-و-جهت-تصویر)
- [10. تنظیم رنگ، نور، کنتراست و Gamma](#10-تنظیم-رنگ-نور-کنتراست-و-gamma)
- [11. فیلترها و افکت‌ها](#11-فیلترها-و-افکتها)
- [12. رسم متن و شکل روی تصویر](#12-رسم-متن-و-شکل-روی-تصویر)
- [13. واترمارک و ترکیب تصاویر](#13-واترمارک-و-ترکیب-تصاویر)
- [14. متادیتا: EXIF، IPTC، XMP و پروفایل رنگ](#14-متادیتا-exif-iptc-xmp-و-پروفایل-رنگ)
- [15. کار با پیکسل‌ها](#15-کار-با-پیکسلها)
- [16. تصاویر انیمیشنی و GIF](#16-تصاویر-انیمیشنی-و-gif)
- [17. کار با PDF](#17-کار-با-pdf)
- [18. بهینه‌سازی حجم و فشرده‌سازی](#18-بهینهسازی-حجم-و-فشردهسازی)
- [19. Defineها و تنظیمات پیشرفته خواندن/نوشتن](#19-defineها-و-تنظیمات-پیشرفته-خواندندنوشتن)
- [20. محدودیت منابع، امنیت و پایداری](#20-محدودیت-منابع-امنیت-و-پایداری)
- [21. مدیریت خطاها](#21-مدیریت-خطاها)
- [22. Best Practices و چک‌لیست پروژه واقعی](#22-best-practices-و-چکلیست-پروژه-واقعی)
- [23. چند مثال کاربردی](#23-چند-مثال-کاربردی)
- [24. منابع معتبر](#24-منابع-معتبر)

---

<a id="1-imagemagick-چیست"></a>

## 1. ImageMagick چیست؟

**ImageMagick** یک مجموعه نرم‌افزاری متن‌باز و بسیار قدرتمند برای:

- تبدیل فرمت تصاویر
- تغییر اندازه
- برش
- چرخش
- تنظیم رنگ و نور
- افزودن متن و واترمارک
- ترکیب تصاویر
- ساخت انیمیشن
- پردازش‌های پیشرفته تصویر
- کار با فرمت‌هایی مثل JPEG، PNG، GIF، WebP، TIFF، BMP، SVG، PDF و بسیاری فرمت‌های دیگر

است.

سایت رسمی ImageMagick:

- https://imagemagick.org

مستندات عمومی ImageMagick:

- https://imagemagick.org/script/command-line-processing.php
- https://imagemagick.org/script/command-line-options.php
- https://imagemagick.org/script/formats.php

ImageMagick ذاتاً با C توسعه داده شده و بیشتر از طریق ابزارهای خط فرمان استفاده می‌شود؛ اما در زبان‌های مختلف، کتابخانه‌های wrapper برای آن وجود دارد.

---

<a id="2-در-c-از-چه-کتابخانهای-استفاده-کنیم"></a>

## 2. در C# از چه کتابخانه‌ای استفاده کنیم؟

در دنیای .NET، کتابخانه اصلی و شناخته‌شده برای استفاده از ImageMagick، کتابخانه **Magick.NET** است.

ریپازیتوری رسمی Magick.NET:

- https://github.com/dlemstra/Magick.NET

Magick.NET یک wrapper بسیار کامل برای ImageMagick است و به شما اجازه می‌دهد بدون اجرای مستقیم دستور `magick` یا `convert` در خط فرمان، از قابلیت‌های ImageMagick داخل کد C# استفاده کنید.

### مزایای Magick.NET

- استفاده مستقیم از ImageMagick داخل C#
- پشتیبانی از تعداد بسیار زیادی فرمت تصویر
- API شیءگرا و مناسب .NET
- عدم نیاز به نصب جداگانه ImageMagick در بسیاری از سناریوها
- پشتیبانی از Stream، byte array، فایل و collection تصاویر
- مناسب برای وب، دسکتاپ، سرویس‌های backend و پردازش دسته‌ای

### نکته مهم

Magick.NET کتابخانه‌های native ImageMagick را داخل پکیج‌های NuGet حمل می‌کند؛ بنابراین در بسیاری از موارد نیازی نیست ImageMagick را جداگانه روی سیستم نصب کنید.

اما برای برخی فرمت‌ها، مثل PDF، ممکن است به ابزارهای جانبی مثل **Ghostscript** نیاز داشته باشید.

Ghostscript:

- https://ghostscript.com

---

<a id="3-نصب-و-راهاندازی-magicknet"></a>

## 3. نصب و راه‌اندازی Magick.NET

ساده‌ترین روش نصب، استفاده از NuGet است.

### نصب با .NET CLI

```bash
dotnet add package Magick.NET-Q16-AnyCPU
```

### نصب با Package Manager Console

```powershell
Install-Package Magick.NET-Q16-AnyCPU
```

### یا استفاده از پکیج Q8

```bash
dotnet add package Magick.NET-Q8-AnyCPU
```

مستند نصب رسمی Magick.NET:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/InstallMagickNet.md

بعد از نصب، فقط کافی است namespace زیر را اضافه کنید:

```csharp
using ImageMagick;
```

---

<a id="4-انتخاب-پکیج-مناسب"></a>

## 4. انتخاب پکیج مناسب

Magick.NET پکیج‌های مختلفی دارد. مهم‌ترین آن‌ها بر اساس **Quantum Depth** و قابلیت‌های اجرایی هستند.

### Quantum Depth چیست؟

Quantum Depth مشخص می‌کند هر کانال رنگی با چند بیت ذخیره و پردازش شود.

| پکیج | توضیح | کاربرد |
|---|---|---|
| Q8 | هر کانال 8 بیت | مناسب بیشتر کارهای عمومی، وب، JPEG/PNG/WebP |
| Q16 | هر کانال 16 بیت | دقت رنگی بالاتر، مناسب TIFF، پردازش‌های دقیق‌تر |
| Q16-HDRI | پشتیبانی از محدوده دینامیک بالا | سناریوهای HDR و پردازش‌های خاص |
| OpenMP | پردازش موازی چند thread | افزایش سرعت در برخی عملیات، معمولاً فقط x64 |

### پکیج‌های رایج

| پکیج | توضیح |
|---|---|
| `Magick.NET-Q8-AnyCPU` | نسخه 8 بیتی، مناسب اکثر پروژه‌ها |
| `Magick.NET-Q16-AnyCPU` | نسخه 16 بیتی، دقت بالاتر |
| `Magick.NET-Q16-HDRI-AnyCPU` | نسخه HDRI |
| `Magick.NET-Q8-OpenMP-x64` | نسخه 8 بیتی با OpenMP |
| `Magick.NET-Q16-OpenMP-x64` | نسخه 16 بیتی با OpenMP |
| `Magick.NET-Q16-HDRI-OpenMP-x64` | نسخه HDRI با OpenMP |

### توصیه برای شروع

اگر تازه کار را شروع می‌کنید:

```bash
dotnet add package Magick.NET-Q16-AnyCPU
```

اگر فقط تصاویر وب مثل JPEG/PNG/WebP پردازش می‌کنید و حافظه برایتان مهم است:

```bash
dotnet add package Magick.NET-Q8-AnyCPU
```

### بررسی نسخه ImageMagick و Magick.NET

```csharp
using ImageMagick;

Console.WriteLine($"Magick.NET Version: {MagickNET.Version}");
Console.WriteLine($"ImageMagick Version: {MagickNET.ImageMagickVersion}");
Console.WriteLine($"Quantum Depth: {MagickNET.QuantumDepth}");
Console.WriteLine($"Is OpenMP: {MagickNET.IsOpenMP}");
```

مستندات مرتبط:

- Quantum: https://github.com/dlemstra/Magick.NET/blob/main/docs/Quantum.md
- OpenMP: https://github.com/dlemstra/Magick.NET/blob/main/docs/OpenMP.md  
  اگر فایل OpenMP در ریپازیتوری موجود نبود، از صفحه docs اصلی دیدن کنید:  
  https://github.com/dlemstra/Magick.NET/tree/main/docs

---

<a id="5-اولین-برنامه-خواندن-ذخیره-و-تبدیل-تصویر"></a>

## 5. اولین برنامه: خواندن، ذخیره و تبدیل تصویر

ساده‌ترین کار، خواندن یک تصویر و ذخیره آن با فرمت دیگر است.

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.Format = MagickFormat.Png;
image.Write("output.png");
```

### توضیح

- `MagickImage`: کلاس اصلی برای کار با یک تصویر
- `Format`: فرمت خروجی را مشخص می‌کند
- `Write`: تصویر را ذخیره می‌کند
- `using`: باعث می‌شود منابع native آزاد شوند

### تبدیل JPEG به WebP

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.Format = MagickFormat.WebP;
image.Quality = 80;
image.Write("photo.webp");
```

### تبدیل PNG به JPEG با پس‌زمینه سفید

JPEG معمولاً از transparency پشتیبانی نمی‌کند. اگر PNG شما شفاف است، بهتر است قبل از تبدیل، پس‌زمینه مشخص کنید.

```csharp
using ImageMagick;

using var image = new MagickImage("image.png");

image.BackgroundColor = MagickColors.White;
image.Alpha(AlphaOption.Remove);

image.Format = MagickFormat.Jpeg;
image.Quality = 85;
image.Write("image.jpg");
```

مستند خواندن و نوشتن تصاویر:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/ReadingAndWritingImages.md

---

<a id="6-کار-با-stream-و-byte-array"></a>

## 6. کار با Stream و Byte Array

در پروژه‌های واقعی، به‌خصوص ASP.NET Core، معمولاً تصویر را از Stream یا آرایه بایت می‌گیریم.

### خواندن از Stream

```csharp
using ImageMagick;

using FileStream inputStream = File.OpenRead("input.jpg");
using var image = new MagickImage(inputStream);

image.Format = MagickFormat.WebP;

using FileStream outputStream = File.Create("output.webp");
image.Write(outputStream);
```

### خواندن از byte array

```csharp
using ImageMagick;

byte[] fileBytes = File.ReadAllBytes("input.jpg");

using var image = new MagickImage(fileBytes);

image.Format = MagickFormat.Png;
byte[] pngBytes = image.ToByteArray();
```

### نوشتن به MemoryStream

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.Format = MagickFormat.WebP;
image.Quality = 80;

using var memoryStream = new MemoryStream();
image.Write(memoryStream);

memoryStream.Position = 0;

byte[] result = memoryStream.ToArray();
```

### دریافت byte array با فرمت مشخص

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

byte[] webpBytes = image.ToByteArray(MagickFormat.WebP);
byte[] pngBytes = image.ToByteArray(MagickFormat.Png);
```

---

<a id="7-شناسایی-تصویر-بدون-بارگذاری-کامل-پیکسلها"></a>

## 7. شناسایی تصویر بدون بارگذاری کامل پیکسل‌ها

اگر فقط می‌خواهید عرض، ارتفاع، فرمت یا اطلاعات کلی تصویر را بخوانید، لازم نیست کل پیکسل‌ها را load کنید.

### استفاده از MagickImageInfo

```csharp
using ImageMagick;

var info = new MagickImageInfo("input.jpg");

Console.WriteLine($"Width: {info.Width}");
Console.WriteLine($"Height: {info.Height}");
Console.WriteLine($"Format: {info.Format}");
Console.WriteLine($"Color Space: {info.ColorSpace}");
```

### استفاده از Ping

```csharp
using ImageMagick;

using var image = new MagickImage();
image.Ping("input.jpg");

Console.WriteLine($"Width: {image.Width}");
Console.WriteLine($"Height: {image.Height}");
```

این روش‌ها برای آپلود فایل، اعتبارسنجی سریع تصویر و ساخت thumbnail index بسیار مفید هستند.

---

<a id="8-تغییر-اندازه-برش-trim-و-extent"></a>

## 8. تغییر اندازه، برش، Trim و Extent

مستند رسمی تغییر اندازه:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/ResizingImages.md

---

### تغییر اندازه ساده با حفظ نسبت

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.Resize(new MagickGeometry(800, 600));
image.Write("resized.jpg");
```

در این حالت، تصویر داخل کادر 800x600 قرار می‌گیرد، ولی نسبت ابعاد حفظ می‌شود.

---

### تغییر اندازه دقیق بدون حفظ نسبت

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.Resize(new MagickGeometry("800x600!"));
image.Write("resized-exact.jpg");
```

علامت `!` یعنی نسبت ابعاد نادیده گرفته شود.

---

### تغییر اندازه به‌صورت Cover یا Fill

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.Resize(new MagickGeometry("800x600^"));
image.Crop(800, 600, Gravity.Center);
image.RePage();

image.Write("cover.jpg");
```

علامت `^` باعث می‌شود تصویر حداقل 800x600 شود، سپس با Crop دقیقاً همان ابعاد را نگه می‌داریم.

---

### تغییر اندازه با درصد

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.Resize(new Percentage(50));
image.Write("half-size.jpg");
```

---

### Thumbnail

برای ساخت thumbnail، متد `Thumbnail` معمولاً مناسب است، چون علاوه بر تغییر اندازه، برخی پروفایل‌ها و متادیتای غیرضروری را هم حذف می‌کند.

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.Thumbnail(new MagickGeometry(300, 300));
image.Write("thumbnail.jpg");
```

---

### برش تصویر Crop

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

// x, y, width, height
image.Crop(new MagickGeometry(100, 100, 400, 300));
image.RePage();

image.Write("cropped.jpg");
```

### نکته مهم: RePage

بعد از `Crop` معمولاً باید `RePage` را صدا بزنید تا offset صفحه مجازی ImageMagick ریست شود.

```csharp
image.Crop(new MagickGeometry(10, 10, 200, 200));
image.RePage();
```

---

### Trim کردن حاشیه‌های اضافی

```csharp
using ImageMagick;

using var image = new MagickImage("logo.png");

image.Trim();
image.RePage();

image.Write("trimmed.png");
```

---

### Extent: قرار دادن تصویر در بوم مشخص

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.BackgroundColor = MagickColors.White;
image.Extent(new MagickGeometry(1000, 1000), Gravity.Center);

image.Write("centered-on-canvas.jpg");
```

این کار تصویر را داخل یک بوم 1000x1000 قرار می‌دهد و فضای خالی را با رنگ پس‌زمینه پر می‌کند.

---

### جدول خلاصه Geometry

| Geometry | معنا |
|---|---|
| `800x600` | حفظ نسبت، جا شدن داخل کادر |
| `800x600!` | تغییر اندازه دقیق، نادیده گرفتن نسبت |
| `800x600^` | پر کردن حداقل یکی از ابعاد، مناسب cover |
| `800x600>` | فقط اگر تصویر بزرگ‌تر باشد کوچک شود |
| `800x600<` | فقط اگر تصویر کوچک‌تر باشد بزرگ شود |
| `50%` | تغییر اندازه درصدی |

---

<a id="9-چرخش-آینه-auto-orient-و-جهت-تصویر"></a>

## 9. چرخش، آینه، Auto-Orient و جهت تصویر

### چرخش

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.BackgroundColor = MagickColors.Transparent;
image.Rotate(90);

image.Write("rotated.png");
```

### چرخش با زاویه دلخواه

```csharp
image.Rotate(45);
```

### Flip و Flop

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

// Flip عمودی
image.Flip();

// Flop افقی
image.Flop();

image.Write("flipped.jpg");
```

### Auto-Orient

بسیاری از تصاویر JPEG دارای اطلاعات EXIF Orientation هستند. اگر می‌خواهید تصویر بر اساس اطلاعات دوربین به‌صورت خودکار درست شود:

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.AutoOrient();
image.Write("oriented.jpg");
```

این متد در پروژه‌های آپلود تصویر بسیار مهم است.

---

<a id="10-تنظیم-رنگ-نور-کنتراست-و-gamma"></a>

## 10. تنظیم رنگ، نور، کنتراست و Gamma

### تبدیل به خاکستری

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.Grayscale();
image.Write("gray.jpg");
```

### Sepia

```csharp
image.SepiaTone();
```

### Negate

```csharp
image.Negate();
```

### Solarize

```csharp
image.Solarize();
```

### تنظیم روشنایی، اشباع رنگ و Hue با Modulate

متد `Modulate` سه مقدار می‌گیرد:

1. Brightness
2. Saturation
3. Hue

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

// روشنایی 110٪، اشباع 90٪، Hue بدون تغییر
image.Modulate(
    new Percentage(110),
    new Percentage(90),
    new Percentage(100)
);

image.Write("modulated.jpg");
```

### تنظیم کنتراست با Level

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.Level(new Percentage(5), new Percentage(95));
image.Write("leveled.jpg");
```

### Gamma Correction

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.GammaCorrect(1.2);
image.Write("gamma.jpg");
```

### Auto Level و Auto Gamma

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.AutoLevel();
image.AutoGamma();

image.Write("auto-corrected.jpg");
```

---

<a id="11-فیلترها-و-افکتها"></a>

## 11. فیلترها و افکت‌ها

Magick.NET بسیاری از افکت‌های ImageMagick را در اختیار شما قرار می‌دهد.

### Blur

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.GaussianBlur(5, 2);
image.Write("blurred.jpg");
```

### Sharpen

```csharp
image.Sharpen();
```

### Unsharp Mask

```csharp
image.UnsharpMask(2, 1, 1, 0.05);
```

### Charcoal

```csharp
image.Charcoal();
```

### Sketch

```csharp
image.Sketch();
```

### Oil Paint

```csharp
image.OilPaint();
```

### Implode

```csharp
image.Implode(0.5);
```

### Swirl

```csharp
image.Swirl(90);
```

### Wave

```csharp
image.Wave(new Percentage(25), new Percentage(150));
```

### Vignette

```csharp
image.Vignette();
```

### Motion Blur

```csharp
image.MotionBlur(10, 10, 10);
```

### Radial Blur

```csharp
image.RadialBlur(10);
```

### مثال ترکیبی

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.Grayscale();
image.Sharpen();
image.Vignette();

image.Write("styled.jpg");
```

---

<a id="12-رسم-متن-و-شکل-روی-تصویر"></a>

## 12. رسم متن و شکل روی تصویر

مستند Drawing:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/Drawing.md

### رسم متن ساده

```csharp
using ImageMagick;

using var canvas = new MagickImage(MagickColors.White, 600, 300);

var drawables = new Drawables()
    .Font("Arial", 36)
    .FillColor(MagickColors.Black)
    .Text(50, 150, "Hello Magick.NET");

canvas.Draw(drawables);
canvas.Write("text.png");
```

### نکته درباره فونت در Linux

در سرورهای Linux ممکن است فونت‌های Windows مثل Arial موجود نباشند. بهتر است از یک فایل فونت مشخص استفاده کنید:

```csharp
using ImageMagick;

using var canvas = new MagickImage(MagickColors.White, 600, 300);

canvas.Settings.Font = "/path/to/font.ttf";

var drawables = new Drawables()
    .FontPointSize(36)
    .FillColor(MagickColors.Black)
    .Text(50, 150, "Hello");

canvas.Draw(drawables);
canvas.Write("text.png");
```

### نکته مهم درباره متن فارسی

ImageMagick برای رسم متن پیچیده فارسی/عربی همیشه بهترین نتیجه را ندارد؛ مخصوصاً از نظر shaping و جهت‌دهی حروف. اگر نیاز به رندر دقیق متن فارسی دارید، بهتر است:

- متن را با کتابخانه‌های مناسب‌تر مثل SkiaSharp یا ابزارهای سیستمی رندر کنید،
- یا از font و تنظیمات مناسب تست بگیرید،
- یا متن فارسی را از قبل به تصویر تبدیل کنید و سپس composite کنید.

### رسم شکل

```csharp
using ImageMagick;

using var canvas = new MagickImage(MagickColors.White, 800, 400);

var drawables = new Drawables()
    .StrokeColor(MagickColors.Black)
    .StrokeWidth(2)
    .FillColor(MagickColors.Red)
    .Rectangle(50, 50, 250, 150)
    .FillColor(MagickColors.Blue)
    .Circle(400, 100, 450, 100)
    .FillColor(MagickColors.Green)
    .Ellipse(600, 250, 80, 40, 0, 360);

canvas.Draw(drawables);
canvas.Write("shapes.png");
```

### متن وسط تصویر

```csharp
using ImageMagick;

using var canvas = new MagickImage(MagickColors.Black, 800, 400);

var drawables = new Drawables()
    .FontPointSize(40)
    .FillColor(MagickColors.White)
    .TextAlignment(TextAlignment.Center)
    .Text(400, 200, "Center Text");

canvas.Draw(drawables);
canvas.Write("center-text.png");
```

---

<a id="13-واترمارک-و-ترکیب-تصاویر"></a>

## 13. واترمارک و ترکیب تصاویر

مستند Combining Images:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/CombiningImages.md

### Composite ساده

```csharp
using ImageMagick;

using var background = new MagickImage("background.jpg");
using var logo = new MagickImage("logo.png");

background.Composite(logo, Gravity.Southeast, CompositeOperator.Over);
background.Write("watermarked.jpg");
```

### واترمارک با شفافیت

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");
using var watermark = new MagickImage("logo.png");

watermark.Resize(new Percentage(15));
watermark.Alpha(AlphaOption.On);
watermark.Evaluate(Channels.Alpha, EvaluateOperator.Multiply, 0.6);

image.Composite(watermark, Gravity.Southeast, CompositeOperator.Over);
image.Write("watermarked.png");
```

### Composite با موقعیت مختصاتی

```csharp
image.Composite(watermark, 10, 10, CompositeOperator.Over);
```

### واترمارک متنی

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

var watermarkText = new Drawables()
    .FontPointSize(24)
    .FillColor(MagickColors.White)
    .TextAlignment(TextAlignment.Center)
    .Text(image.Width / 2, image.Height - 20, "© 2026 My Site");

image.Draw(watermarkText);
image.Write("text-watermark.jpg");
```

### Composite Operatorهای رایج

| Operator | کاربرد |
|---|---|
| `Over` | قرار دادن تصویر روی تصویر دیگر |
| `Multiply` | ضرب رنگ‌ها |
| `Screen` | روشن‌تر کردن |
| `Overlay` | ترکیب overlay |
| `Darken` | تیره‌تر کردن |
| `Lighten` | روشن‌تر کردن |
| `Difference` | اختلاف دو تصویر |
| `Dissolve` | محو شدن با شدت مشخص |

---

<a id="14-متادیتا-exif-iptc-xmp-و-پروفایل-رنگ"></a>

## 14. متادیتا: EXIF، IPTC، XMP و پروفایل رنگ

مستندات رسمی:

- EXIF: https://github.com/dlemstra/Magick.NET/blob/main/docs/ExifProfile.md
- IPTC: https://github.com/dlemstra/Magick.NET/blob/main/docs/IptcProfile.md
- XMP: https://github.com/dlemstra/Magick.NET/blob/main/docs/XmpProfile.md

### خواندن EXIF

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

var exif = image.GetExifProfile();

if (exif is not null)
{
    var cameraMake = exif.GetValue(ExifTag.Make)?.Value;
    var cameraModel = exif.GetValue(ExifTag.Model)?.Value;

    Console.WriteLine($"Make: {cameraMake}");
    Console.WriteLine($"Model: {cameraModel}");
}
```

### تغییر EXIF

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

var exif = image.GetExifProfile();

if (exif is null)
{
    exif = new ExifProfile();
}

exif.SetValue(ExifTag.Copyright, "© 2026 Your Name");

image.SetProfile(exif);
image.Write("photo-with-exif.jpg");
```

### حذف EXIF

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.RemoveProfile("exif");
image.Write("photo-no-exif.jpg");
```

### حذف همه پروفایل‌ها و متادیتا

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.Strip();
image.Write("photo-stripped.jpg");
```

### IPTC

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

var iptc = image.GetIptcProfile();

if (iptc is not null)
{
    var caption = iptc.GetValue(IptcTag.Caption)?.Value;
    Console.WriteLine($"Caption: {caption}");

    iptc.SetValue(IptcTag.CopyrightNotice, "© 2026 Your Name");
    image.SetProfile(iptc);

    image.Write("photo-with-iptc.jpg");
}
```

### XMP

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

var xmp = image.GetXmpProfile();

if (xmp is not null)
{
    string xml = xmp.ToXml();
    Console.WriteLine(xml);
}
```

### پروفایل رنگ ICC

```csharp
using ImageMagick;

using var image = new MagickImage("photo.jpg");

image.SetProfile(new ColorProfile("sRGB.icc"));
image.Write("photo-srgb.jpg");
```

---

<a id="15-کار-با-پیکسلها"></a>

## 15. کار با پیکسل‌ها

مستند Pixel Collection:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/PixelCollection.md

### خواندن رنگ یک پیکسل

```csharp
using ImageMagick;

using var image = new MagickImage("image.png");

using var pixels = image.GetPixels();

var pixel = pixels.GetPixel(0, 0);
var color = pixel.ToColor();

Console.WriteLine(color.ToString());
```

### تغییر رنگ یک پیکسل

```csharp
using ImageMagick;

using var image = new MagickImage("image.png");

using var pixels = image.GetPixels();

var max = (ushort)Quantum.Max;

// قرمز
pixels.SetPixel(0, 0, new ushort[] { max, 0, 0 });

image.Write("changed-pixel.png");
```

### خواندن یک ناحیه

```csharp
using ImageMagick;

using var image = new MagickImage("image.png");

using var pixels = image.GetPixels();

ushort[] area = pixels.GetArea(0, 0, 100, 100);

Console.WriteLine(area.Length);
```

### استفاده از UnsafePixelCollection برای سرعت بیشتر

```csharp
using ImageMagick;

using var image = new MagickImage("image.png");

using var unsafePixels = image.GetPixelsUnsafe();

ushort[] values = unsafePixels.GetArea(0, 0, image.Width, image.Height);

// پردازش values

unsafePixels.SetArea(0, 0, image.Width, image.Height, values);
```

### نکته مهم

کار مستقیم با پیکسل‌ها می‌تواند سریع باشد، ولی اگر فقط می‌خواهید فیلتر یا تغییر رنگ کلی اعمال کنید، بهتر است از متدهای آماده ImageMagick استفاده کنید؛ چون معمولاً بهینه‌تر هستند.

---

<a id="16-تصاویر-انیمیشنی-و-gif"></a>

## 16. تصاویر انیمیشنی و GIF

مستند Animation:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/Animation.md

### خواندن GIF انیمیشنی

```csharp
using ImageMagick;

using var gif = new MagickImageCollection("animation.gif");

Console.WriteLine($"Frames: {gif.Count}");
```

### تغییر اندازه GIF

```csharp
using ImageMagick;

using var gif = new MagickImageCollection("animation.gif");

gif.Coalesce();

foreach (var frame in gif)
{
    frame.Resize(new MagickGeometry(400, 300));
}

gif.Optimize();
gif.Write("animation-resized.gif");
```

### تغییر تأخیر فریم‌ها

```csharp
foreach (var frame in gif)
{
    frame.AnimationDelay = 10; // hundredths of a second
}
```

### حلقه بی‌نهایت GIF

```csharp
if (gif.Count > 0)
{
    gif[0].AnimationIterations = 0;
}
```

### ساخت GIF از چند تصویر

```csharp
using ImageMagick;

using var frames = new MagickImageCollection();

frames.Add(new MagickImage("1.png") { AnimationDelay = 100 });
frames.Add(new MagickImage("2.png") { AnimationDelay = 100 });
frames.Add(new MagickImage("3.png") { AnimationDelay = 100 });

frames.Write("slideshow.gif");
```

### بهینه‌سازی GIF

```csharp
using ImageMagick;

using var gif = new MagickImageCollection("animation.gif");

gif.Coalesce();
gif.Optimize();
gif.OptimizeTransparency();

gif.Write("optimized.gif");
```

### Quantize برای کاهش رنگ‌های GIF

```csharp
using ImageMagick;

using var gif = new MagickImageCollection("animation.gif");

var settings = new QuantizeSettings
{
    Colors = 256,
    DitherMethod = DitherMethod.FloydSteinberg
};

gif.Coalesce();
gif.Quantize(settings);
gif.Optimize();

gif.Write("quantized.gif");
```

---

<a id="17-کار-با-pdf"></a>

## 17. کار با PDF

مستند PDF:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/ConvertPDF.md

### نکته مهم

برای خواندن PDF معمولاً به **Ghostscript** نیاز دارید. بدون Ghostscript ممکن است خطای missing delegate دریافت کنید.

### تبدیل صفحه اول PDF به PNG

```csharp
using ImageMagick;

var settings = new MagickReadSettings
{
    Density = new Density(300),
    Format = MagickFormat.Pdf,
    FrameIndex = 0,
    FrameCount = 1
};

using var page = new MagickImage("document.pdf", settings);
page.Write("page-1.png");
```

### تبدیل همه صفحات PDF به PNG

```csharp
using ImageMagick;

var settings = new MagickReadSettings
{
    Density = new Density(300),
    Format = MagickFormat.Pdf
};

using var pages = new MagickImageCollection("document.pdf", settings);

for (int i = 0; i < pages.Count; i++)
{
    pages[i].Write($"page-{i + 1}.png");
}
```

### ساخت PDF از چند تصویر

```csharp
using ImageMagick;

using var document = new MagickImageCollection();

document.Add(new MagickImage("page1.png"));
document.Add(new MagickImage("page2.png"));
document.Add(new MagickImage("page3.png"));

document.Write("output.pdf");
```

### تنظیم density برای PDF

```csharp
var settings = new MagickReadSettings
{
    Density = new Density(300),
    Format = MagickFormat.Pdf
};
```

Density بالاتر باعث می‌شود خروجی دقیق‌تر و با کیفیت‌تر باشد، ولی حافظه بیشتری مصرف می‌کند.

---

<a id="18-بهینهسازی-حجم-و-فشردهسازی"></a>

## 18. بهینه‌سازی حجم و فشرده‌سازی

### بهینه‌سازی JPEG

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.AutoOrient();
image.Strip();
image.Quality = 82;
image.Interlace = Interlace.Plane;

image.Settings.SetDefine(MagickFormat.Jpeg, "extent", "300kb");

image.Write("optimized.jpg");
```

### بهینه‌سازی PNG

```csharp
using ImageMagick;

using var image = new MagickImage("input.png");

image.Strip();
image.Format = MagickFormat.Png;

image.Settings.SetDefine(MagickFormat.Png, "compression-level", "9");
image.Settings.SetDefine(MagickFormat.Png, "exclude-chunk", "all");

image.Write("optimized.png");
```

### بهینه‌سازی WebP

```csharp
using ImageMagick;

using var image = new MagickImage("input.jpg");

image.Format = MagickFormat.WebP;
image.Quality = 80;

image.Settings.SetDefine(MagickFormat.WebP, "method", "6");

image.Write("optimized.webp");
```

### Quantize برای کاهش تعداد رنگ‌ها

```csharp
using ImageMagick;

using var image = new MagickImage("input.png");

var settings = new QuantizeSettings
{
    Colors = 256,
    DitherMethod = DitherMethod.FloydSteinberg
};

image.Quantize(settings);
image.Write("quantized.png");
```

---

<a id="19-defineها-و-تنظیمات-پیشرفته-خواندندنوشتن"></a>

## 19. Defineها و تنظیمات پیشرفته خواندن/نوشتن

مستندات Define:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/Define.md
- https://github.com/dlemstra/Magick.NET/blob/main/docs/DefaultDefines.md

Defineها تنظیمات خاصی هستند که به encoder یا decoder فرمت‌ها داده می‌شوند.

### مثال‌های رایج

```csharp
image.Settings.SetDefine(MagickFormat.Jpeg, "extent", "200kb");
```

```csharp
image.Settings.SetDefine(MagickFormat.Png, "compression-level", "9");
```

```csharp
image.Settings.SetDefine(MagickFormat.Png, "exclude-chunk", "all");
```

```csharp
image.Settings.SetDefine(MagickFormat.WebP, "method", "6");
```

```csharp
image.Settings.SetDefine(MagickFormat.Pdf, "use-cropbox", "true");
```

### MagickReadSettings

```csharp
using ImageMagick;

var settings = new MagickReadSettings
{
    Format = MagickFormat.Pdf,
    Density = new Density(300),
    BackgroundColor = MagickColors.White,
    FrameIndex = 0,
    FrameCount = 1
};

settings.SetDefine(MagickFormat.Pdf, "use-cropbox", "true");

using var image = new MagickImage("file.pdf", settings);
```

### جدول چند Define مفید

| Define | کاربرد |
|---|---|
| `jpeg:extent` | محدود کردن حجم خروجی JPEG |
| `png:compression-level` | سطح فشرده‌سازی PNG |
| `png:exclude-chunk` | حذف chunkهای اضافی PNG |
| `webp:method` | سرعت/کیفیت encode WebP |
| `pdf:use-cropbox` | استفاده از crop box در PDF |

---

<a id="20-محدودیت-منابع-امنیت-و-پایداری"></a>

## 20. محدودیت منابع، امنیت و پایداری

مستند Resource Limits:

- https://github.com/dlemstra/Magick.NET/blob/main/docs/ResourceLimits.md

وقتی تصویرهای بزرگ یا ورودی‌های نامعتبر پردازش می‌کنید، محدود کردن منابع بسیار مهم است.

### تنظیم محدودیت منابع

```csharp
using ImageMagick;

MagickNET.SetTempDirectory(Path.Combine(Path.GetTempPath(), "magicknet-temp"));

ResourceLimits.Memory = 512 * 1024 * 1024;       // 512 MB
ResourceLimits.Disk = 2L * 1024 * 1024 * 1024;   // 2 GB
ResourceLimits.Thread = 2;
ResourceLimits.Time = 30;                        // seconds
ResourceLimits.Width = 20000;
ResourceLimits.Height = 20000;
```

### نکات امنیتی مهم

1. **ورودی نامعتبر را جدی بگیرید**  
   پردازش تصویرهای آپلودشده توسط کاربر می‌تواند خطرناک باشد.

2. **فرمت را محدود کنید**  
   اگر فقط JPEG/PNG/WebP نیاز دارید، اجازه پردازش SVG، PDF، MVG یا فرمت‌های خطرناک‌تر را ندهید.

3. **از Stream استفاده کنید**  
   تا حد امکان فایل موقت نسازید و از Stream استفاده کنید.

4. **ResourceLimits تنظیم کنید**  
   برای جلوگیری از مصرف بیش از حد RAM یا CPU.

5. **کتابخانه را به‌روز نگه دارید**  
   ImageMagick و Magick.NET را به‌روز نگه دارید، چون مشکلات امنیتی ممکن است در نسخه‌های جدید رفع شوند.

6. **از اجرای delegateهای غیرضروری پرهیز کنید**  
   بعضی فرمت‌ها به ابزارهای خارجی مثل Ghostscript وابسته هستند. اگر نیاز ندارید، آن‌ها را در production نصب یا فعال نکنید.

7. **SVG و MVG را با احتیاط پردازش کنید**  
   این فرمت‌ها می‌توانند برداری و قابل تفسیر باشند؛ پردازش آن‌ها از منابع نامعتبر می‌تواند ریسک امنیتی داشته باشد.

مستندات امنیت ImageMagick:

- https://imagemagick.org/script/security-policy.php

---

<a id="21-مدیریت-خطاها"></a>

## 21. مدیریت خطاها

خطاهای Magick.NET معمولاً از نوع `MagickException` یا زیرمجموعه‌های آن هستند.

```csharp
using ImageMagick;

try
{
    using var image = new MagickImage("input.jpg");

    image.Resize(new MagickGeometry(800, 600));
    image.Write("output.jpg");
}
catch (MagickMissingDelegateErrorException ex)
{
    Console.WriteLine("Missing delegate error:");
    Console.WriteLine(ex.Message);
}
catch (MagickCorruptImageErrorException ex)
{
    Console.WriteLine("Corrupt image error:");
    Console.WriteLine(ex.Message);
}
catch (MagickResourceLimitErrorException ex)
{
    Console.WriteLine("Resource limit error:");
    Console.WriteLine(ex.Message);
}
catch (MagickException ex)
{
    Console.WriteLine("General ImageMagick error:");
    Console.WriteLine(ex.Message);
}
```

### خطاهای رایج

| خطا | علت احتمالی |
|---|---|
| Missing delegate | نبود Ghostscript یا delegate لازم |
| Corrupt image | فایل خراب یا ناقص |
| Resource limit exceeded | عبور از محدودیت RAM، disk یا زمان |
| Blob error | مشکل در دسترسی به فایل یا stream |
| Option error | پارامتر نادرست |

---

<a id="22-best-practices-و-چکلیست-پروژه-واقعی"></a>

## 22. Best Practices و چک‌لیست پروژه واقعی

### نکات عمومی

1. همیشه از `using` برای `MagickImage` و `MagickImageCollection` استفاده کنید.
2. بعد از `Crop` معمولاً `RePage()` را صدا بزنید.
3. قبل از ذخیره در stream، `Format` را تنظیم کنید.
4. برای آپلود تصاویر، `AutoOrient()` را فراموش نکنید.
5. برای کاهش حجم، `Strip()` را در نظر بگیرید.
6. برای thumbnail، از `Thumbnail` استفاده کنید.
7. برای پروژه‌های وب، WebP یا AVIF را در صورت پشتیبانی مرورگر/محیط در نظر بگیرید.
8. از پردازش تصویرهای خیلی بزرگ بدون `ResourceLimits` خودداری کنید.
9. برای PDF، نیاز بودن Ghostscript را در نظر بگیرید.
10. در سرورهای Linux، فونت‌ها را جداگانه نصب یا همراه برنامه deploy کنید.

### چک‌لیست production

- [ ] پکیج مناسب انتخاب شده است: Q8/Q16/HDRI/OpenMP
- [ ] `ResourceLimits` تنظیم شده است
- [ ] temp directory مشخص شده است
- [ ] خطاهای MagickException مدیریت می‌شوند
- [ ] فرمت‌های مجاز آپلود محدود شده‌اند
- [ ] متادیتای حساس حذف می‌شود
- [ ] تصاویر قبل از ذخیره `AutoOrient` شده‌اند
- [ ] کیفیت خروجی مشخص است
- [ ] در Linux فونت نصب شده است
- [ ] در صورت نیاز به PDF، Ghostscript نصب شده است
- [ ] لاگ‌گیری مناسب برای خطاها وجود دارد
- [ ] تست با فایل خراب، فایل بزرگ و فرمت غیرمجاز انجام شده است

---

<a id="23-چند-مثال-کاربردی"></a>

## 23. چند مثال کاربردی

### مثال 1: تغییر اندازه تصویر در ASP.NET Core

```csharp
using ImageMagick;
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapPost("/resize", async (IFormFile file) =>
{
    if (file is null || file.Length == 0)
        return Results.BadRequest("File is empty.");

    using var inputStream = file.OpenReadStream();
    using var image = new MagickImage(inputStream);

    image.AutoOrient();
    image.Strip();
    image.Resize(new MagickGeometry(1024, 1024));

    image.Format = MagickFormat.WebP;
    image.Quality = 80;

    using var outputStream = new MemoryStream();
    image.Write(outputStream);

    outputStream.Position = 0;

    return Results.File(outputStream, "image/webp");
});

app.Run();
```

---

### مثال 2: تبدیل دسته‌ای PNG به WebP

```csharp
using ImageMagick;

foreach (string file in Directory.EnumerateFiles("images", "*.png"))
{
    using var image = new MagickImage(file);

    image.Format = MagickFormat.WebP;
    image.Quality = 80;

    string outputPath = Path.ChangeExtension(file, ".webp");
    image.Write(outputPath);

    Console.WriteLine($"Converted: {outputPath}");
}
```

---

### مثال 3: ساخت thumbnail

```csharp
using ImageMagick;

public static void CreateThumbnail(string inputPath, string outputPath, int size = 300)
{
    using var image = new MagickImage(inputPath);

    image.AutoOrient();
    image.Thumbnail(new MagickGeometry(size, size));
    image.Strip();

    image.Write(outputPath);
}
```

---

### مثال 4: واترمارک لوگو روی همه تصاویر یک پوشه

```csharp
using ImageMagick;

string watermarkPath = "logo.png";

foreach (string file in Directory.EnumerateFiles("photos", "*.jpg"))
{
    using var image = new MagickImage(file);
    using var watermark = new MagickImage(watermarkPath);

    watermark.Resize(new Percentage(10));
    watermark.Alpha(AlphaOption.On);
    watermark.Evaluate(Channels.Alpha, EvaluateOperator.Multiply, 0.7);

    image.Composite(watermark, Gravity.Southeast, CompositeOperator.Over);

    string outputPath = Path.Combine("watermarked", Path.GetFileName(file));
    image.Write(outputPath);

    Console.WriteLine($"Watermarked: {outputPath}");
}
```

---

### مثال 5: مقایسه دو تصویر

```csharp
using ImageMagick;

using var imageA = new MagickImage("a.png");
using var imageB = new MagickImage("b.png");

double difference = imageA.Compare(imageB, ErrorMetric.Absolute);

Console.WriteLine($"Difference: {difference}");
```

---

### مثال 6: ایجاد تصویر خالی

```csharp
using ImageMagick;

using var canvas = new MagickImage(MagickColors.Transparent, 800, 600);

canvas.Write("blank.png");
```

---

### مثال 7: ترکیب چند تصویر به‌صورت افقی

```csharp
using ImageMagick;

using var collection = new MagickImageCollection();

collection.Add(new MagickImage("1.png"));
collection.Add(new MagickImage("2.png"));
collection.Add(new MagickImage("3.png"));

using var combined = collection.AppendHorizontally();
combined.Write("combined.png");
```

---

### مثال 8: ترکیب چند تصویر به‌صورت عمودی

```csharp
using ImageMagick;

using var collection = new MagickImageCollection();

collection.Add(new MagickImage("1.png"));
collection.Add(new MagickImage("2.png"));
collection.Add(new MagickImage("3.png"));

using var combined = collection.AppendVertically();
combined.Write("stacked.png");
```

---

### مثال 9: Flatten کردن تصاویر شفاف

```csharp
using ImageMagick;

using var collection = new MagickImageCollection();

collection.Add(new MagickImage("background.jpg"));
collection.Add(new MagickImage("overlay.png"));

using var flattened = collection.Flatten(MagickColors.White);
flattened.Write("flattened.jpg");
```

---

### مثال 10: عملیات Morphology و Convolution

این بخش پیشرفته‌تر است و برای پردازش‌های خاص مثل edge detection، blur سفارشی، dilate و erode استفاده می‌شود.

```csharp
using ImageMagick;

using var image = new MagickImage("input.png");

// Convolution با kernel سفارشی
var sharpenKernel = new ConvolveMatrix(3, new double[]
{
     0, -1,  0,
    -1,  5, -1,
     0, -1,  0
});

image.Convolve(sharpenKernel);

image.Write("convolved.png");
```

```csharp
using ImageMagick;

using var image = new MagickImage("input.png");

image.Morphology(MorphologyMethod.Dilate, Kernel.Diamond, 1);

image.Write("dilated.png");
```

```csharp
using ImageMagick;

using var image = new MagickImage("input.png");

image.Threshold(new Percentage(50));

image.Write("threshold.png");
```

---

<a id="24-منابع-معتبر"></a>

## 24. منابع معتبر

### منابع رسمی ImageMagick

- سایت رسمی ImageMagick  
  https://imagemagick.org

- مستندات خط فرمان ImageMagick  
  https://imagemagick.org/script/command-line-processing.php

- گزینه‌های خط فرمان ImageMagick  
  https://imagemagick.org/script/command-line-options.php

- فرمت‌های پشتیبانی‌شده  
  https://imagemagick.org/script/formats.php

- امنیت و Policy در ImageMagick  
  https://imagemagick.org/script/security-policy.php

- لایسنس ImageMagick  
  https://imagemagick.org/script/license.php

---

### منابع رسمی Magick.NET

- ریپازیتوری رسمی Magick.NET  
  https://github.com/dlemstra/Magick.NET

- مستندات Magick.NET  
  https://github.com/dlemstra/Magick.NET/tree/main/docs

- نصب Magick.NET  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/InstallMagickNet.md

- خواندن و نوشتن تصاویر  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/ReadingAndWritingImages.md

- تغییر اندازه تصاویر  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/ResizingImages.md

- ترکیب تصاویر  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/CombiningImages.md

- رسم متن و شکل  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/Drawing.md

- انیمیشن و GIF  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/Animation.md

- PDF  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/ConvertPDF.md

- EXIF  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/ExifProfile.md

- IPTC  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/IptcProfile.md

- XMP  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/XmpProfile.md

- Pixel Collection  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/PixelCollection.md

- Define  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/Define.md

- Default Defines  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/DefaultDefines.md

- Quantum  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/Quantum.md

- Resource Limits  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/ResourceLimits.md

- Thread Safety  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/ThreadSafety.md

- Windows Fonts  
  https://github.com/dlemstra/Magick.NET/blob/main/docs/WindowsFonts.md

---

### NuGet

- Magick.NET-Q16-AnyCPU  
  https://www.nuget.org/packages/Magick.NET-Q16-AnyCPU

- Magick.NET-Q8-AnyCPU  
  https://www.nuget.org/packages/Magick.NET-Q8-AnyCPU

- Magick.NET-Q16-HDRI-AnyCPU  
  https://www.nuget.org/packages/Magick.NET-Q16-HDRI-AnyCPU

---

### سایر منابع مفید

- Ghostscript  
  https://ghostscript.com

- Stack Overflow تگ Magick.NET  
  https://stackoverflow.com/questions/tagged/magick.net

- Stack Overflow تگ ImageMagick  
  https://stackoverflow.com/questions/tagged/imagemagick

- Issues و Discussions ریپازیتوری Magick.NET  
  https://github.com/dlemstra/Magick.NET/issues  
  https://github.com/dlemstra/Magick.NET/discussions

---

## جمع‌بندی

اگر می‌خواهید در C# با ImageMagick کار کنید، بهترین مسیر استفاده از **Magick.NET** است. این کتابخانه به شما امکان می‌دهد:

- تصاویر را بخوانید، تبدیل کنید و ذخیره کنید
- تغییر اندازه، برش، چرخش و افکت‌ها را اعمال کنید
- واترمارک و متن اضافه کنید
- متادیتا را بخوانید یا حذف کنید
- با GIF، PDF و فرمت‌های مختلف کار کنید
- پردازش‌های پیشرفته تصویر انجام دهید
- برای پروژه‌های وب و backend بهینه‌سازی کنید

برای شروع، پیشنهاد می‌کنم با این مسیر پیش بروید:

1. نصب `Magick.NET-Q16-AnyCPU`
2. خواندن و تبدیل فرمت ساده
3. Resize و Crop
4. Watermark
5. Metadata stripping
6. Optimization
7. کار با Stream در وب
8. GIF و PDF
9. تنظیم ResourceLimits و امنیت
10. استفاده از Defineها و سناریوهای پیشرفته
