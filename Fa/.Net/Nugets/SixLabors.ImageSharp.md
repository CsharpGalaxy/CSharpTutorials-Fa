 **یک راهنمای کامل از مقدماتی تا پیشرفته برای پردازش تصویر در دات‌نت**

---

## 📑 فهرست مطالب

| # | عنوان | لینک |
|---|-------|------|
| ۱ | [ImageSharp چیست؟](#1-imagesharp-چیست) | 🔗 |
| ۲ | [چرا ImageSharp به‌جای System.Drawing؟](#2-چرا-imagesharp-بهجای-systemdrawing) | 🔗 |
| ۳ | [نصب و راه‌اندازی](#3-نصب-و-راهاندازی) | 🔗 |
| ۴ | [مفاهیم پایه و معماری](#4-مفاهیم-پایه-و-معماری) | 🔗 |
| ۵ | [بارگذاری و ذخیره تصاویر](#5-بارگذاری-و-ذخیره-تصاویر) | 🔗 |
| ۶ | [تغییر اندازه (Resize)](#6-تغییر-اندازه-resize) | 🔗 |
| ۷ | [برش، چرخش و آینه](#7-برش-چرخش-و-آینه) | 🔗 |
| ۸ | [فیلترها و افکت‌ها](#8-فیلترها-و-افکتها) | 🔗 |
| ۹ | [فرمت‌های تصویری و انکودرها](#9-فرمتهای-تصویری-و-انکودرها) | 🔗 |
| ۱۰ | [کار با متادیتا (EXIF / XMP / IPTC)](#10-کار-با-متادیتا) | 🔗 |
| ۱۱ | [کار با پیکسل‌ها به‌صورت مستقیم](#11-کار-با-پیکسلها-بهصورت-مستقیم) | 🔗 |
| ۱۲ | [Drawing API (نقاشی و شکل)](#12-drawing-api-نقاشی-و-شکل) | 🔗 |
| ۱۳ | [پردازش پیشرفته و سفارشی](#13-پردازش-پیشرفته-و-سفارشی) | 🔗 |
| ۱۴ | [نکات عملکردی و مدیریت حافظه](#14-نکات-عملکردی-و-مدیریت-حافظه) | 🔗 |
| ۱۵ | [الگوهای رایج در پروژه‌های واقعی](#15-الگوهای-رایج-در-پروژههای-واقعی) | 🔗 |
| ۱۶ | [عیب‌یابی و خطاهای رایج](#16-عیبیابی-و-خطاهای-رایج) | 🔗 |
| ۱۷ | [منابع و مراجع معتبر](#17-منابع-و-مراجع-معتبر) | 🔗 |

---

## ۱. ImageSharp چیست؟

**SixLabors.ImageSharp** یک کتابخانه پردازش تصویر **کراس‌پلتفرم** و **تمام‌مدنی (Fully Managed)** برای دات‌نت است که توسط گروه **Six Labors** توسعه داده می‌شود.

### ✨ ویژگی‌های کلیدی

| ویژگی | توضیح |
|--------|--------|
| 🌐 **کراس‌پلتفرم** | روی Windows، Linux، macOS و حتی موبایل کار می‌کند |
| 🚫 **بدون وابستگی Native** | هیچ وابستگی به GDI+ یا کتابخانه‌های سیستمی ندارد |
| ⚡ **مدرن و سریع** | با C# خالص و بهینه‌سازی‌های SIMD نوشته شده |
| 🔌 **پلاگین‌پذیر** | می‌توانید فرمت‌ها و پردازشگرهای سفارشی اضافه کنید |
| 📦 **API ساده** | طراحی Fluent و زنجیره‌ای برای خوانایی بالا |
| 🖼️ **پشتیبانی گسترده** | JPEG، PNG، GIF، WebP، TIFF، BMP و ... |

### 🎯 چه زمانی از ImageSharp استفاده کنیم؟

- ✅ وب‌اپلیکیشن‌های ASP.NET Core که نیاز به پردازش تصویر دارند
- ✅ ساخت API برای آپلود و بهینه‌سازی تصاویر
- ✅ ساخت Thumbnail و تغییر اندازه
- ✅ تبدیل فرمت تصاویر
- ✅ اعمال فیلتر و افکت
- ✅ خواندن/نوشتن متادیتا

> ⚠️ **نکته:** ImageSharp برای پردازش بلادرنگ ویدیو یا پردازش تصویر صنعتی سنگین (مثل OpenCV) طراحی نشده است.

---

## ۲. چرا ImageSharp به‌جای System.Drawing؟

مایکروسافت به‌طور رسمی `System.Drawing` را **منسوخ (Deprecated)** اعلام کرده و برای محیط‌های سرور توصیه **نمی‌کند**.

| معیار | System.Drawing (GDI+) | ImageSharp |
|-------|----------------------|------------|
| پلتفرم | فقط Windows (کامل) | همه پلتفرم‌ها |
| Thread Safety | ❌ ناامن | ✅ امن |
| وابستگی Native | ✅ دارد (GDI+) | ❌ ندارد |
| ASP.NET Core | ⚠️ توصیه نمی‌شود | ✅ توصیه می‌شود |
| فرمت‌های مدرن | ❌ محدود | ✅ WebP, TIFF, ... |
| نگهداری آینده | ❌ منسوخ | ✅ فعال |

> 📖 **منبع رسمی مایکروسافت:**
> [System.Drawing.Common cross-platform guidance](https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/6.0/system-drawing-common-windows-only)

---

## ۳. نصب و راه‌اندازی

### 📦 نصب پکیج اصلی

از طریق NuGet Package Manager Console:

```bash
Install-Package SixLabors.ImageSharp
```

یا با .NET CLI:

```bash
dotnet add package SixLabors.ImageSharp
```

### 📦 پکیج‌های مرتبط

| پکیج | کاربرد | دستور نصب |
|------|---------|-----------|
| `SixLabors.ImageSharp` | هسته اصلی (پردازش تصویر) | `dotnet add package SixLabors.ImageSharp` |
| `SixLabors.ImageSharp.Drawing` | نقاشی، متن، شکل‌ها | `dotnet add package SixLabors.ImageSharp.Drawing` |
| `SixLabors.ImageSharp.Web` | Middleware برای ASP.NET Core | `dotnet add package SixLabors.ImageSharp.Web` |

### 🔧 اولین برنامه

```csharp
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Processing;

// بارگذاری تصویر
using var image = Image.Load("input.jpg");

// تغییر اندازه به عرض ۸۰۰ پیکسل
image.Mutate(x => x.Resize(800, 0));

// ذخیره با فرمت جدید
image.Save("output_resized.jpg");

Console.WriteLine($"✅ تصویر ذخیره شد! ابعاد: {image.Width}x{image.Height}");
```

### ⚙️ ساختار using مورد نیاز

```csharp
// هسته اصلی
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Processing;

// فرمت‌ها و انکودرها
using SixLabors.ImageSharp.Formats;
using SixLabors.ImageSharp.Formats.Jpeg;
using SixLabors.ImageSharp.Formats.Png;
using SixLabors.ImageSharp.Formats.Webp;
using SixLabors.ImageSharp.Formats.Gif;

// متادیتا
using SixLabors.ImageSharp.Metadata;
using SixLabors.ImageSharp.Metadata.Profiles.Exif;

// Drawing (پکیج جداگانه)
using SixLabors.ImageSharp.Drawing;
using SixLabors.ImageSharp.Drawing.Processing;
```

---

## ۴. مفاهیم پایه و معماری

### 🏗️ ساختار کلی

```
SixLabors.ImageSharp
│
├── Image<TPixel>          ← کلاس اصلی تصویر
├── Image                  ← کلاس غیر‌جنریک (کمکی)
├── Configuration          ← تنظیمات سراسری
├── IImageDecoder          ← رابط رمزگشا
├── IImageEncoder          ← رابط رمزگذار
├── Pixel Formats          ← فرمت‌های پیکسلی
│   ├── Rgba32
│   ├── Bgr24
│   ├── L8
│   └── ...
└── Processing
    ├── IImageProcessor
    ├── ResizeProcessor
    ├── FilterProcessor
    └── ...
```

### 🖼️ کلاس `Image<TPixel>`

مهم‌ترین کلاس کتابخانه است:

```csharp
// تصویر با فرمت پیکسلی مشخص
Image<Rgba32> image = new Image<Rgba32>(width: 1920, height: 1080);

// تصویر با رنگ پس‌زمینه
Image<Rgba32> colored = new Image<Rgba32>(800, 600, Color.CornflowerBlue);
```

### 🎨 فرمت‌های پیکسلی رایج

| فرمت | توضیح | بیت بر پیکسل |
|------|--------|--------------|
| `Rgba32` | قرمز، سبز، آبی + آلفا (پیش‌فرض) | ۳۲ |
| `Bgr24` | آبی، سبز، قرمز (بدون آلفا) | ۲۴ |
| `Rgb24` | قرمز، سبز، آبی (بدون آلفا) | ۲۴ |
| `L8` | خاکستری ۸ بیتی | ۸ |
| `L16` | خاکستری ۱۶ بیتی | ۱۶ |
| `Rgba64` | RGBA با دقت ۱۶ بیت در هر کانال | ۶۴ |
| `A8` | فقط کانال آلفا | ۸ |

> 💡 **نکته:** اگر نیازی به شفافیت (Transparency) ندارید، از `Rgb24` استفاده کنید تا حافظه کمتری مصرف شود.

### ⚙️ کلاس `Configuration`

تنظیمات سراسری کتابخانه:

```csharp
// دسترسی به تنظیمات پیش‌فرض
Configuration config = Configuration.Default;

// ایجاد تنظیمات سفارشی
var customConfig = Configuration.Default.Clone();

// اضافه کردن یک فرمت سفارشی
customConfig.ImageFormatsManager.SetEncoder(JpegFormat.Instance, new JpegEncoder
{
    Quality = 90
});

// استفاده از تنظیمات سفارشی
using var image = Image.Load(customConfig, "photo.jpg");
```

---

## ۵. بارگذاری و ذخیره تصاویر

### 📂 بارگذاری (Load)

#### از فایل

```csharp
// ساده‌ترین حالت
using var image = Image.Load("photo.jpg");

// با تنظیمات سفارشی
using var image2 = Image.Load(Configuration.Default, "photo.png");

// بارگذاری فقط فرمت پیکسلی مشخص
using Image<Rgba32> rgbaImage = Image.Load<Rgba32>("photo.jpg");
```

#### از Stream

```csharp
using var stream = File.OpenRead("photo.jpg");
using var image = Image.Load(stream);
```

#### از آرایه بایت

```csharp
byte[] imageBytes = await File.ReadAllBytesAsync("photo.jpg");
using var image = Image.Load(imageBytes);
```

#### از Base64 (مثلاً در وب)

```csharp
string base64 = "iVBORw0KGgoAAAANS..."; // داده Base64
byte[] bytes = Convert.FromBase64String(base64);
using var image = Image.Load(bytes);
```

### 📝 ذخیره (Save)

```csharp
using var image = Image.Load("input.jpg");

// ذخیره ساده (فرمت از روی پسوند تشخیص داده می‌شود)
image.Save("output.png");

// ذخیره با انکودر مشخص
image.Save("output.webp", new WebpEncoder
{
    Quality = 85
});

// ذخیره در Stream
using var ms = new MemoryStream();
image.Save(ms, new JpegEncoder { Quality = 80 });
byte[] resultBytes = ms.ToArray();
```

### 🔍 شناسایی تصویر بدون بارگذاری کامل (Identify)

اگر فقط می‌خواهید اطلاعات تصویر را بخوانید بدون اینکه کل تصویر را در حافظه بارگذاری کنید:

```csharp
// فقط اطلاعات (ابعاد، فرمت، متادیتا)
ImageInfo info = Image.Identify("huge_photo.jpg");

Console.WriteLine($"ابعاد: {info.Width} x {info.Height}");
Console.WriteLine($"فرمت: {info.Metadata.DecodedImageFormat?.Name}");
Console.WriteLine($"رزولوشن: {info.HorizontalResolution} DPI");
```

> ⚡ **مزیت:** `Identify` بسیار سریع‌تر از `Load` است چون فقط هدر فایل را می‌خواند.

### 📋 بارگذاری با محدودیت حافظه

```csharp
var options = new DecoderOptions
{
    // محدودیت حافظه: حداکثر ۱۰۰ مگابایت
    MemoryAllocator = MemoryAllocator.Create(new MemoryAllocatorOptions
    {
        MemoryGroupBudget = 100 * 1024 * 1024
    })
};

using var image = Image.Load(options, "large_image.tiff");
```

---

## ۶. تغییر اندازه (Resize)

### 🔧 تغییر اندازه ساده

```csharp
using var image = Image.Load("photo.jpg");

// تغییر به عرض ۸۰۰ (ارتفاع خودکار محاسبه می‌شود)
image.Mutate(x => x.Resize(800, 0));

// تغییر به ارتفاع ۶۰۰ (عرض خودکار)
image.Mutate(x => x.Resize(0, 600));

// تغییر به ابعاد دقیق (ممکن است نسبت تصویر به هم بخورد)
image.Mutate(x => x.Resize(800, 600));
```

### 🎯 تغییر اندازه با گزینه‌های پیشرفته

```csharp
var resizeOptions = new ResizeOptions
{
    // حالت تغییر اندازه
    Mode = ResizeMode.Max,

    // ابعاد هدف
    Size = new Size(1920, 1080),

    // الگوریتم نمونه‌برداری
    Sampler = KnownResamplers.Lanczos3,

    // فقط اگر تصویر بزرگ‌تر از هدف باشد کوچک شود
    // (از بزرگ‌شدن جلوگیری می‌کند)
    // در نسخه‌های جدید از Pad یا Max استفاده کنید
};

image.Mutate(x => x.Resize(resizeOptions));
```

### 📐 حالت‌های ResizeMode

| حالت | توضیح |
|------|--------|
| `Max` | تصویر درون جعبه جا می‌شود، نسبت حفظ می‌شود |
| `Min` | تصویر جعبه را پر می‌کند، نسبت حفظ می‌شود |
| `Crop` | تصویر برش می‌خورد تا دقیقاً اندازه هدف شود |
| `Pad` | تصویر با حاشیه پر می‌شود (مناسب Thumbnail) |
| `Stretch` | تصویر دقیقاً کشیده می‌شود (نسبت ممکن است به هم بخورد) |
| `BoxPad` | تصویر در مرکز قرار می‌گیرد و اطراف پر می‌شود |

### 📌 مثال عملی: ساخت Thumbnail

```csharp
public static void CreateThumbnail(string inputPath, string outputPath, int size = 150)
{
    using var image = Image.Load(inputPath);

    var options = new ResizeOptions
    {
        Mode = ResizeMode.Crop,
        Size = new Size(size, size),
        Sampler = KnownResamplers.Lanczos3
    };

    image.Mutate(x => x.Resize(options));
    image.Save(outputPath, new JpegEncoder { Quality = 85 });
}
```

### 🔍 الگوریتم‌های نمونه‌برداری (Resamplers)

| الگوریتم | کیفیت | سرعت | کاربرد |
|-----------|--------|------|---------|
| `NearestNeighbour` | کم | بسیار سریع | پیکسل‌آرت |
| `Box` | متوسط | سریع | کوچک‌سازی |
| `Triangle` | متوسط | سریع | عمومی |
| `Lanczos2` | خوب | متوسط | عمومی |
| `Lanczos3` | عالی | کند | بهترین کیفیت |
| `Lanczos5` | بسیار عالی | خیلی کند | حرفه‌ای |
| `Bicubic` | خوب | متوسط | عمومی |
| `CatmullRom` | خوب | متوسط | عمومی |

```csharp
// کوچک‌سازی سریع
image.Mutate(x => x.Resize(new ResizeOptions
{
    Size = new Size(200, 200),
    Sampler = KnownResamplers.Box
}));

// بزرگ‌نمایی با کیفیت بالا
image.Mutate(x => x.Resize(new ResizeOptions
{
    Size = new Size(3000, 3000),
    Sampler = KnownResamplers.Lanczos5
}));
```

---

## ۷. برش، چرخش و آینه

### ✂️ برش (Crop)

```csharp
using var image = Image.Load("photo.jpg");

// برش از مختصات مشخص
image.Mutate(x => x.Crop(new Rectangle(100, 100, 500, 400)));

// برش از مرکز
image.Mutate(x => x.Crop(new Rectangle(
    x: image.Width / 4,
    y: image.Height / 4,
    width: image.Width / 2,
    height: image.Height / 2
)));
```

### 🔄 چرخش (Rotate)

```csharp
// چرخش ۹۰ درجه ساعتگرد
image.Mutate(x => x.Rotate(90));

// چرخش ۴۵ درجه (با پس‌زمینه شفاف)
image.Mutate(x => x.Rotate(45));

// چرخش با زاویه دلخواه و پر کردن پس‌زمینه
image.Mutate(x => x.BackgroundColor(Color.White));
image.Mutate(x => x.Rotate(30));
```

### 🪞 آینه (Flip)

```csharp
// آینه افقی
image.Mutate(x => x.Flip(FlipMode.Horizontal));

// آینه عمودی
image.Mutate(x => x.Flip(FlipMode.Vertical));
```

### 🔃 چرخش EXIF (Auto-Orient)

بسیاری از تصاویر موبایل دارای چرخش EXIF هستند:

```csharp
// اصلاح خودکار چرخش بر اساس متادیتا
image.Mutate(x => x.AutoOrient());
```

> 💡 **نکته مهم:** همیشه قبل از تغییر اندازه، `AutoOrient()` را صدا بزنید تا ابعاد صحیح باشند.

---

## ۸. فیلترها و افکت‌ها

ImageSharp مجموعه غنی از فیلترهای داخلی دارد:

### 🎨 فیلترهای رنگی

```csharp
using var image = Image.Load("photo.jpg");

// سیاه و سفید
image.Mutate(x => x.Grayscale());

// معکوس (نگاتیو)
image.Mutate(x => x.Invert());

// سپیا (قدیمی)
image.Mutate(x => x.Sepia());

// تنظیم روشنایی (-1 تا 1)
image.Mutate(x => x.Brightness(1.2f));  // روشن‌تر

// تنظیم کنتراست (-1 تا 1)
image.Mutate(x => x.Contrast(1.3f));    // کنتراست بیشتر

// تنظیم اشباع رنگ
image.Mutate(x => x.Saturate(0.5f));    // رنگ‌های کم‌رنگ‌تر

// تغییر رنگ (Hue)
image.Mutate(x => x.Hue(90));

// تغییر Gamma
image.Mutate(x => x.Gamma(1.5f));
```

### 🌫️ فیلترهای محو و شارپ

```csharp
// محو گاوسی (Gaussian Blur)
image.Mutate(x => x.GaussianBlur(5f));

// شارپ گاوسی
image.Mutate(x => x.GaussianSharpen(3f));

// محو ساده (Box Blur)
image.Mutate(x => x.BoxBlur(4));

// محو با شعاع مشخص
image.Mutate(x => x.Blur(10));
```

### 🖌️ فیلترهای هنری

```csharp
// پیکسلی شدن (Pixelate)
image.Mutate(x => x.Pixelate(8));

// روغن‌نقاشی (Oil Painting)
image.Mutate(x => x.OilPainting(10, 5));

// مداد (Sketch)
image.Mutate(x => x.Sketch());

// مداد رنگی
image.Mutate(x => x.SketchColor(Color.Black));
```

### 🔲 فیلترهای ماتریسی

```csharp
// اعمال ماتریس رنگی سفارشی
var matrix = new ColorMatrix(
    new float[] { 1, 0, 0, 0, 0 },
    new float[] { 0, 1, 0, 0, 0 },
    new float[] { 0, 0, 1, 0, 0 },
    new float[] { 0, 0, 0, 1, 0 }
);
image.Mutate(x => x.Filter(matrix));

// فیلتر سفارشی: حذف کانال آبی
var noBlue = new ColorMatrix(
    new float[] { 1, 0, 0, 0, 0 },
    new float[] { 0, 1, 0, 0, 0 },
    new float[] { 0, 0, 0, 0, 0 },  // آبی حذف شد
    new float[] { 0, 0, 0, 1, 0 }
);
image.Mutate(x => x.Filter(noBlue));
```

### 🔗 ترکیب فیلترها (زنجیره‌ای)

```csharp
image.Mutate(x => x
    .AutoOrient()
    .Brightness(1.1f)
    .Contrast(1.2f)
    .Saturate(1.1f)
    .GaussianSharpen(1.5f)
    .Resize(new ResizeOptions
    {
        Size = new Size(1200, 800),
        Mode = ResizeMode.Max
    })
);
```

### 🎭 فیلتر با ناحیه مشخص

```csharp
// اعمال فیلتر فقط روی بخشی از تصویر
var region = new Rectangle(100, 100, 300, 300);

image.Mutate(x => x.GaussianBlur(10f, region));
```

---

## ۹. فرمت‌های تصویری و انکودرها

### 📊 فرمت‌های پشتیبانی‌شده

| فرمت | خواندن | نوشتن | پکیج |
|------|--------|--------|------|
| JPEG / JPG | ✅ | ✅ | داخلی |
| PNG | ✅ | ✅ | داخلی |
| GIF (انیمیشن) | ✅ | ✅ | داخلی |
| WebP | ✅ | ✅ | داخلی |
| BMP | ✅ | ✅ | داخلی |
| TIFF | ✅ | ✅ | داخلی |
| TGA (Targa) | ✅ | ✅ | داخلی |
| QOI | ✅ | ✅ | داخلی |
| ICO / CUR | ✅ | ✅ | داخلی |
| PBM/PGM/PPM | ✅ | ✅ | داخلی |

### 🖼️ تنظیمات انکودرها

#### JPEG

```csharp
var jpegEncoder = new JpegEncoder
{
    Quality = 85,                    // کیفیت ۰ تا ۱۰۰
    ColorType = JpegEncodingColor.YCbCrRatio420, // فشرده‌سازی رنگ
    Interleaved = true,              // Progressive JPEG
};

image.Save("output.jpg", jpegEncoder);
```

#### PNG

```csharp
var pngEncoder = new PngEncoder
{
    ColorType = PngColorType.RgbWithAlpha,
    BitDepth = PngBitDepth.Bit8,
    CompressionLevel = PngCompressionLevel.BestSpeed, // سرعت یا اندازه
    FilterMethod = PngFilterMethod.Adaptive,
};

image.Save("output.png", pngEncoder);
```

#### WebP

```csharp
var webpEncoder = new WebpEncoder
{
    Quality = 80,
    Method = WebpEncodingMethod.BestQuality,
    FileFormat = WebpFileFormatType.Lossy, // یا Lossless
    NearLossless = false,
    UseAlphaCompression = true,
};

image.Save("output.webp", webpEncoder);
```

#### GIF (انیمیشن)

```csharp
var gifEncoder = new GifEncoder
{
    ColorTableMode = GifColorTableMode.Global,
    Quantizer = KnownQuantizers.WebSafe,
};

image.Save("animation.gif", gifEncoder);
```

### 🔍 تشخیص فرمت خودکار

```csharp
// ImageSharp فرمت را از روی امضای بایت تشخیص می‌دهد
using var image = Image.Load(unknownBytes);

// فرمت تشخیص‌داده‌شده
IImageFormat format = image.Metadata.DecodedImageFormat;
Console.WriteLine($"فرمت: {format?.Name}");
Console.WriteLine($"MIME: {format?.DefaultMimeType}");
```

### 📄 تبدیل فرمت

```csharp
public static void ConvertFormat(string input, string output)
{
    using var image = Image.Load(input);

    // فرمت خروجی از روی پسوند فایل تعیین می‌شود
    image.Save(output);
}

// استفاده
ConvertFormat("photo.jpg", "photo.png");
ConvertFormat("photo.png", "photo.webp");
ConvertFormat("photo.bmp", "photo.jpg");
```

### 🎞️ کار با GIF انیمیشنی

```csharp
using var gif = Image.Load("animation.gif");

// تعداد فریم‌ها
int frameCount = gif.Frames.Count;
Console.WriteLine($"تعداد فریم‌ها: {frameCount}");

// دسترسی به هر فریم
ImageFrame firstFrame = gif.Frames[0];

// تغییر زمان نمایش هر فریم (بر حسب ثانیه)
foreach (var frame in gif.Frames)
{
    var metadata = frame.Metadata.GetGifMetadata();
    metadata.FrameDelay = 100; // میلی‌ثانیه
}

// استخراج یک فریم به‌عنوان تصویر جدا
using var singleFrame = gif.Frames.CloneFrame(0);
singleFrame.Save("frame0.png");
```

---

## ۱۰. کار با متادیتا

### 📷 پروفایل EXIF

```csharp
using var image = Image.Load("photo.jpg");

// خواندن EXIF
ExifProfile exif = image.Metadata.ExifProfile;

if (exif != null)
{
    // خواندن مقادیر
    if (exif.TryGetValue(ExifTag.DateTime, out var dateTime))
        Console.WriteLine($"تاریخ عکس: {dateTime.Value}");

    if (exif.TryGetValue(ExifTag.Make, out var cameraMake))
        Console.WriteLine($"سازنده دوربین: {cameraMake.Value}");

    if (exif.TryGetValue(ExifTag.Model, out var cameraModel))
        Console.WriteLine($"مدل دوربین: {cameraModel.Value}");

    if (exif.TryGetValue(ExifTag.Orientation, out var orientation))
        Console.WriteLine($"جهت: {orientation.Value}");

    // لیست تمام تگ‌ها
    foreach (var value in exif.Values)
    {
        Console.WriteLine($"{value.Tag}: {value.Value}");
    }
}
```

### ✏️ نوشتن EXIF

```csharp
using var image = Image.Load("photo.jpg");

// ایجاد یا ویرایش پروفایل EXIF
var exif = image.Metadata.ExifProfile ?? new ExifProfile();

exif.SetValue(ExifTag.Artist, "نام عکاس");
exif.SetValue(ExifTag.Copyright, "© ۱۴۰۴");
exif.SetValue(ExifTag.ImageDescription, "توضیح تصویر");

image.Metadata.ExifProfile = exif;
image.Save("output.jpg");
```

### 🗑️ حذف متادیتا (برای حریم خصوصی)

```csharp
using var image = Image.Load("photo.jpg");

// حذف تمام متادیتا
image.Metadata.ExifProfile = null;
image.Metadata.XmpProfile = null;
image.Metadata.IptcProfile = null;

image.Save("clean_photo.jpg");
```

### 📐 رزولوشن و DPI

```csharp
using var image = Image.Load("photo.jpg");

// خواندن رزولوشن
Console.WriteLine($"افقی: {image.Metadata.HorizontalResolution} DPI");
Console.WriteLine($"عمودی: {image.Metadata.VerticalResolution} DPI");

// تنظیم رزولوشن
image.Metadata.HorizontalResolution = 300;
image.Metadata.VerticalResolution = 300;
image.Metadata.ResolutionUnits = PixelResolutionUnit.PixelsPerInch;
```

---

## ۱۱. کار با پیکسل‌ها به‌صورت مستقیم

### 🔬 دسترسی به پیکسل‌ها

```csharp
using var image = new Image<Rgba32>(100, 100);

// دسترسی مستقیم به یک پیکسل
Rgba32 pixel = image[50, 50];
Console.WriteLine($"R={pixel.R}, G={pixel.G}, B={pixel.B}, A={pixel.A}");

// تغییر یک پیکسل
image[50, 50] = new Rgba32(255, 0, 0, 255); // قرمز
```

### 🔄 پیمایش تمام پیکسل‌ها

```csharp
using var image = Image.Load<Rgba32>("photo.jpg");

// روش ۱: حلقه ساده
for (int y = 0; y < image.Height; y++)
{
    Span<Rgba32> row = image.DangerousGetPixelRow(y);
    for (int x = 0; x < row.Length; x++)
    {
        ref Rgba32 pixel = ref row[x];

        // مثال: معکوس کردن رنگ
        pixel.R = (byte)(255 - pixel.R);
        pixel.G = (byte)(255 - pixel.G);
        pixel.B = (byte)(255 - pixel.B);
    }
}
```

### ⚡ پیمایش سریع با `ProcessPixelRows`

```csharp
image.ProcessPixelRows(accessor =>
{
    for (int y = 0; y < accessor.Height; y++)
    {
        Span<Rgba32> row = accessor.GetRowSpan(y);

        for (int x = 0; x < row.Length; x++)
        {
            // تبدیل به خاکستری
            byte gray = (byte)(0.299 * row[x].R + 0.587 * row[x].G + 0.114 * row[x].B);
            row[x] = new Rgba32(gray, gray, gray, row[x].A);
        }
    }
});
```

### 🎨 ساخت تصویر از صفر

```csharp
// ساخت تصویر با پیکسل‌های سفارشی
var pixels = new Rgba32[200 * 200];

for (int y = 0; y < 200; y++)
{
    for (int x = 0; x < 200; x++)
    {
        // گرادیانت
        pixels[y * 200 + x] = new Rgba32(
            r: (byte)(x % 256),
            g: (byte)(y % 256),
            b: 128,
            a: 255
        );
    }
}

using var image = Image.LoadPixelData<Rgba32>(pixels, 200, 200);
image.Save("gradient.png");
```

### 🔍 کپی و Clone

```csharp
using var original = Image.Load("photo.jpg");

// کپی کامل
using var clone = original.Clone();

// کپی بخشی از تصویر
using var cropped = original.Clone(x => x.Crop(new Rectangle(0, 0, 200, 200)));

// کپی با تغییر اندازه
using var thumbnail = original.Clone(x => x.Resize(150, 150));
```

---

## ۱۲. Drawing API (نقاشی و شکل)

> ⚠️ **توجه:** Drawing API در پکیج جداگانه `SixLabors.ImageSharp.Drawing` قرار دارد.

### 📦 نصب

```bash
dotnet add package SixLabors.ImageSharp.Drawing
```

### 📝 نوشتن متن روی تصویر

```csharp
using SixLabors.Fonts;
using SixLabors.ImageSharp.Drawing;
using SixLabors.ImageSharp.Drawing.Processing;

using var image = Image.Load("photo.jpg");

// بارگذاری فونت
var fontCollection = new FontCollection();
var fontFamily = fontCollection.Add("arial.ttf");
var font = fontFamily.CreateFont(48, FontStyle.Bold);

// نوشتن متن
image.Mutate(x => x.DrawText(
    "Hello ImageSharp!",           // متن
    font,                           // فونت
    Color.White,                    // رنگ متن
    new PointF(50, 50)              // موقعیت
));

image.Save("with_text.jpg");
```

### 📝 متن با سایه و حاشیه

```csharp
var textOptions = new TextOptions(font)
{
    Origin = new PointF(50, 50),
    // تنظیمات پیشرفته متن
};

image.Mutate(x => x
    // سایه
    .DrawText(textOptions, "Hello!", new SolidBrush(Color.Black), new Pen(Color.Transparent, 1), new PointF(52, 52))
    // متن اصلی
    .DrawText(textOptions, "Hello!", new SolidBrush(Color.White), new Pen(Color.Black, 2), new PointF(50, 50))
);
```

### 🔷 رسم شکل‌ها

```csharp
using var image = new Image<Rgba32>(800, 600, Color.White);

// تعریف نقاط یک چندضلعی
var polygon = new Polygon(
    new LinearLineSegment(
        new PointF(100, 100),
        new PointF(400, 100),
        new PointF(250, 350)
    )
);

image.Mutate(x => x
    // پر کردن با رنگ
    .Fill(Color.CornflowerBlue, polygon)
    // حاشیه
    .Draw(Color.DarkBlue, 3f, polygon)
);
```

### ⭕ رسم دایره و بیضی

```csharp
var ellipse = new EllipsePolygon(
    center: new PointF(400, 300),
    radiusX: 150,
    radiusY: 100
);

image.Mutate(x => x
    .Fill(new Color(new Rgba32(255, 100, 100, 200)), ellipse)
    .Draw(Color.Red, 2f, ellipse)
);
```

### 📏 رسم خط

```csharp
var line = new LinearLineSegment(
    new PointF(0, 0),
    new PointF(800, 600)
);

var path = new Path(line);

image.Mutate(x => x.Draw(Color.Black, 2f, path));
```

### 🖼️ واترمارک (Watermark)

```csharp
public static void AddWatermark(string imagePath, string watermarkText, string outputPath)
{
    using var image = Image.Load(imagePath);

    var fontCollection = new FontCollection();
    var fontFamily = fontCollection.SystemFamilies.First(); // فونت سیستم
    var font = fontFamily.CreateFont(image.Width / 15f, FontStyle.Bold);

    var textLocation = new PointF(image.Width / 2f, image.Height / 2f);

    var textOptions = new TextOptions(font)
    {
        HorizontalAlignment = HorizontalAlignment.Center,
        VerticalAlignment = VerticalAlignment.Center,
        Origin = textLocation
    };

    image.Mutate(x => x.DrawText(
        textOptions,
        watermarkText,
        new SolidBrush(Color.White),
        new Pen(Color.Black, 2f)
    ));

    image.Save(outputPath);
}
```

### 🏷️ افزودن لوگو

```csharp
using var baseImage = Image.Load("photo.jpg");
using var logo = Image.Load<Rgba32>("logo.png");

// تغییر اندازه لوگو
logo.Mutate(x => x.Resize(100, 0));

// محاسبه موقعیت (گوشه پایین راست)
var location = new Point(
    baseImage.Width - logo.Width - 20,
    baseImage.Height - logo.Height - 20
);

// رسم لوگو روی تصویر
baseImage.Mutate(x => x.DrawImage(logo, location, 0.8f)); // opacity = 0.8

baseImage.Save("with_logo.jpg");
```

---

## ۱۳. پردازش پیشرفته و سفارشی

### 🛠️ ساخت پردازشگر سفارشی

```csharp
using SixLabors.ImageSharp.Processing.Processors;

// پردازشگر معکوس‌سازی سفارشی
public class NegativeProcessor : IImageProcessor
{
    public IImageProcessor<TPixel> CreatePixelSpecificProcessor<TPixel>(
        Configuration configuration,
        Image<TPixel> source,
        Rectangle sourceRectangle)
        where TPixel : unmanaged, IPixel<TPixel>
    {
        return new NegativeProcessor<TPixel>(configuration, source, sourceRectangle);
    }
}

public class NegativeProcessor<TPixel> : ImageProcessor<TPixel>
    where TPixel : unmanaged, IPixel<TPixel>
{
    public NegativeProcessor(
        Configuration configuration,
        Image<TPixel> source,
        Rectangle sourceRectangle)
        : base(configuration, source, sourceRectangle)
    {
    }

    protected override void OnFrameApply(ImageFrame<TPixel> source)
    {
        source.ProcessPixelRows(accessor =>
        {
            for (int y = 0; y < accessor.Height; y++)
            {
                Span<TPixel> row = accessor.GetRowSpan(y);
                for (int x = 0; x < row.Length; x++)
                {
                    var rgba = row[x].ToRgba32();
                    rgba.R = (byte)(255 - rgba.R);
                    rgba.G = (byte)(255 - rgba.G);
                    rgba.B = (byte)(255 - rgba.B);
                    row[x].FromRgba32(rgba);
                }
            }
        });
    }
}

// استفاده
image.Mutate(x => x.ApplyProcessor(new NegativeProcessor()));
```

### 🔌 ثبت فرمت سفارشی

```csharp
// ایجاد Configuration سفارشی با انکودر سفارشی
var config = Configuration.Default.Clone();

// مثال: تغییر کیفیت پیش‌فرض JPEG
config.ImageFormatsManager.SetEncoder(JpegFormat.Instance, new JpegEncoder
{
    Quality = 90
});

// استفاده در سراسر برنامه
using var image = Image.Load(config, "photo.jpg");
image.Save(config, "output.jpg");
```

### 📊 پردازش موازی (Parallel)

```csharp
public static void ProcessImagesParallel(string[] paths, string outputDir)
{
    Parallel.ForEach(paths, new ParallelOptions { MaxDegreeOfParallelism = 4 }, path =>
    {
        using var image = Image.Load(path);

        image.Mutate(x => x
            .AutoOrient()
            .Resize(new ResizeOptions
            {
                Size = new Size(1200, 800),
                Mode = ResizeMode.Max
            })
        );

        var outputPath = Path.Combine(outputDir, Path.GetFileName(path));
        image.Save(outputPath, new JpegEncoder { Quality = 80 });
    });
}
```

### 🖥️ استفاده در ASP.NET Core

```csharp
// Controller Action
[HttpPost("upload")]
public async Task<IActionResult> Upload(IFormFile file)
{
    if (file == null || file.Length == 0)
        return BadRequest("فایل معتبر نیست");

    using var stream = file.OpenReadStream();
    using var image = await Image.LoadAsync(stream);

    // اعتبارسنجی ابعاد
    if (image.Width > 5000 || image.Height > 5000)
        return BadRequest("تصویر خیلی بزرگ است");

    // پردازش
    image.Mutate(x => x
        .AutoOrient()
        .Resize(new ResizeOptions
        {
            Size = new Size(1920, 1080),
            Mode = ResizeMode.Max
        })
    );

    // ذخیره
    var fileName = $"{Guid.NewGuid()}.webp";
    var outputPath = Path.Combine(_webHostEnvironment.WebRootPath, "uploads", fileName);

    await using var fs = System.IO.File.Create(outputPath);
    await image.SaveAsync(fs, new WebpEncoder { Quality = 80 });

    return Ok(new { FileName = fileName, Width = image.Width, Height = image.Height });
}
```

### 🌐 استفاده با ImageSharp.Web (Middleware)

```csharp
// Program.cs / Startup.cs
builder.Services.AddImageSharp();

var app = builder.Build();

// فعال‌سازی middleware
app.UseImageSharp();

// حالا می‌توانید در URL پارامتر بدهید:
// /images/photo.jpg?width=800&height=600&rmode=max
// /images/photo.jpg?width=200&format=webp
// /images/photo.jpg?width=400&rmode=crop&rsampler=nearest
```

---

## ۱۴. نکات عملکردی و مدیریت حافظه

### ⚠️ حتماً از `using` استفاده کنید

```csharp
// ✅ صحیح
using var image = Image.Load("photo.jpg");
// ... پردازش ...
// تصویر به‌طور خودکار Dispose می‌شود

// ❌ غلط - نشت حافظه!
var image = Image.Load("photo.jpg");
// بدون Dispose، حافظه آزاد نمی‌شود
```

### 📏 ابعاد تصویر را قبل از Load بررسی کنید

```csharp
// ✅ خوب: اول Identify، بعد Load در صورت نیاز
var info = Image.Identify(stream);
if (info.Width > 10000 || info.Height > 10000)
    throw new InvalidOperationException("تصویر خیلی بزرگ است");

stream.Position = 0;
using var image = Image.Load(stream);
```

### 🔄 از Clone به‌جای Load مجدد استفاده کنید

```csharp
using var original = Image.Load("photo.jpg");

// ✅ خوب: Clone از تصویر موجود
using var thumb1 = original.Clone(x => x.Resize(150, 150));
using var thumb2 = original.Clone(x => x.Resize(300, 300));

// ❌ بد: Load مجدد از فایل
using var thumb1 = Image.Load("photo.jpg");
thumb1.Mutate(x => x.Resize(150, 150));
```

### 🗑️ GC و حافظه

```csharp
// برای پردازش تعداد زیادی تصویر
for (int i = 0; i < imagePaths.Length; i++)
{
    using var image = Image.Load(imagePaths[i]);
    // پردازش...
    image.Save(outputPaths[i]);

    // هر ۵۰ تصویر، GC را فراخوانی کنید (اختیاری)
    if (i % 50 == 0)
    {
        GC.Collect();
        GC.WaitForPendingFinalizers();
    }
}
```

### 📊 مقایسه عملکرد

| عملیات | ImageSharp | System.Drawing |
|--------|-----------|----------------|
| بارگذاری JPEG ۱۰MB | ~۸۰ms | ~۱۲۰ms |
| Resize به ۵۰٪ | ~۶۰ms | ~۹۰ms |
| ذخیره JPEG | ~۴۰ms | ~۷۰ms |
| تبدیل PNG → WebP | ~۱۰۰ms | ❌ پشتیبانی نمی‌شود |

> 📌 اعداد تقریبی و بستگی به سخت‌افزار دارند.

---

## ۱۵. الگوهای رایج در پروژه‌های واقعی

### 📁 ساختار پیشنهادی پروژه

```
MyProject/
├── Controllers/
│   └── ImageController.cs
├── Services/
│   ├── IImageService.cs
│   └── ImageService.cs
├── Models/
│   └── ImageUploadResult.cs
├── wwwroot/
│   ├── uploads/
│   └── thumbnails/
└── Program.cs
```

### 🏢 سرویس پردازش تصویر

```csharp
public interface IImageService
{
    Task<ImageUploadResult> ProcessUploadAsync(Stream stream, string fileName);
    Task<byte[]> ResizeAsync(byte[] imageBytes, int width, int height);
    Task<byte[]> ConvertToWebPAsync(byte[] imageBytes, int quality = 80);
}

public class ImageService : IImageService
{
    private readonly string _uploadPath;
    private readonly string _thumbnailPath;

    public ImageService(IWebHostEnvironment env)
    {
        _uploadPath = Path.Combine(env.WebRootPath, "uploads");
        _thumbnailPath = Path.Combine(env.WebRootPath, "thumbnails");
    }

    public async Task<ImageUploadResult> ProcessUploadAsync(Stream stream, string fileName)
    {
        using var image = await Image.LoadAsync(stream);

        // اعتبارسنجی
        if (image.Width > 8000 || image.Height > 8000)
            throw new ArgumentException("تصویر بیش از حد بزرگ است");

        // اصلاح چرخش
        image.Mutate(x => x.AutoOrient());

        // تغییر اندازه در صورت نیاز
        if (image.Width > 1920)
        {
            image.Mutate(x => x.Resize(new ResizeOptions
            {
                Size = new Size(1920, 0),
                Mode = ResizeMode.Max
            }));
        }

        // تولید نام یکتا
        var uniqueName = $"{Guid.NewGuid():N}.webp";
        var fullPath = Path.Combine(_uploadPath, uniqueName);

        // ذخیره
        await using var fs = File.Create(fullPath);
        await image.SaveAsync(fs, new WebpEncoder { Quality = 82 });

        // ساخت thumbnail
        using var thumb = image.Clone(x => x.Resize(200, 200));
        var thumbPath = Path.Combine(_thumbnailPath, uniqueName);
        await using var thumbFs = File.Create(thumbPath);
        await thumb.SaveAsync(thumbFs, new WebpEncoder { Quality = 75 });

        return new ImageUploadResult
        {
            FileName = uniqueName,
            Width = image.Width,
            Height = image.Height,
            SizeInBytes = new FileInfo(fullPath).Length
        };
    }

    public async Task<byte[]> ResizeAsync(byte[] imageBytes, int width, int height)
    {
        using var image = await Image.LoadAsync(imageBytes);
        image.Mutate(x => x.Resize(new ResizeOptions
        {
            Size = new Size(width, height),
            Mode = ResizeMode.Max
        }));

        using var ms = new MemoryStream();
        await image.SaveAsync(ms, new JpegEncoder { Quality = 85 });
        return ms.ToArray();
    }

    public async Task<byte[]> ConvertToWebPAsync(byte[] imageBytes, int quality = 80)
    {
        using var image = await Image.LoadAsync(imageBytes);
        using var ms = new MemoryStream();
        await image.SaveAsync(ms, new WebpEncoder { Quality = quality });
        return ms.ToArray();
    }
}
```

### 📋 مدل نتیجه

```csharp
public class ImageUploadResult
{
    public string FileName { get; set; } = string.Empty;
    public int Width { get; set; }
    public int Height { get; set; }
    public long SizeInBytes { get; set; }
}
```

### 🔌 ثبت سرویس در DI

```csharp
// Program.cs
builder.Services.AddScoped<IImageService, ImageService>();
```

---

## ۱۶. عیب‌یابی و خطاهای رایج

### ❌ خطا: `UnknownImageFormatException`

```
SixLabors.ImageSharp.UnknownImageFormatException: Image format could not be recognized
```

**علت:** فرمت فایل پشتیبانی نمی‌شود یا فایل خراب است.

**راه‌حل:**

```csharp
try
{
    using var image = Image.Load(bytes);
}
catch (UnknownImageFormatException)
{
    // فایل تصویر معتبر نیست
    return BadRequest("فرمت تصویر ناشناخته است");
}
```

### ❌ خطا: `OutOfMemoryException`

**علت:** تصویر خیلی بزرگ است.

**راه‌حل:**

```csharp
// بررسی ابعاد قبل از بارگذاری
var info = Image.Identify(stream);
if (info.Width * info.Height > 50_000_000) // ۵۰ مگاپیکسل
    throw new InvalidOperationException("تصویر خیلی بزرگ است");
```

### ❌ خطا: `InvalidOperationException` در ASP.NET

**علت:** استفاده از `System.Drawing` به‌جای ImageSharp.

**راه‌حل:** مطمئن شوید از `SixLabors.ImageSharp` استفاده می‌کنید نه `System.Drawing`.

### ❌ تصویر سیاه یا خالی ذخیره می‌شود

**علت:** فراموش کردن `using` یا Dispose زودهنگام.

**راه‌حل:**

```csharp
// ✅ مطمئن شوید تصویر قبل از Save زنده است
using var image = Image.Load(inputPath);
image.Mutate(x => x.Resize(800, 0));
image.Save(outputPath); // هنوز داخل using هستیم
```

### ❌ رنگ‌ها در تصاویر CMYK خراب هستند

**علت:** ImageSharp CMYK را به RGB تبدیل می‌کند.

**راه‌حل:**

```csharp
// اگر نیاز به CMYK دارید، از TIFF encoder استفاده کنید
image.Save("output.tiff", new TiffEncoder
{
    CompressionType = TiffCompressionType.Lzw
});
```

---

## ۱۷. منابع و مراجع معتبر

### 📚 منابع رسمی

| منبع | لینک |
|------|------|
| 🏠 **وب‌سایت رسمی SixLabors** | [https://sixlabors.com](https://sixlabors.com) |
| 📖 **مستندات رسمی ImageSharp** | [https://docs.sixlabors.com/articles/imagesharp/](https://docs.sixlabors.com/articles/imagesharp/) |
| 💻 **مخزن GitHub** | [https://github.com/SixLabors/ImageSharp](https://github.com/SixLabors/ImageSharp) |
| 🎨 **مخزن Drawing API** | [https://github.com/SixLabors/ImageSharp.Drawing](https://github.com/SixLabors/ImageSharp.Drawing) |
| 🌐 **ImageSharp.Web** | [https://github.com/SixLabors/ImageSharp.Web](https://github.com/SixLabors/ImageSharp.Web) |
| 📦 **NuGet - ImageSharp** | [https://www.nuget.org/packages/SixLabors.ImageSharp](https://www.nuget.org/packages/SixLabors.ImageSharp) |
| 📦 **NuGet - Drawing** | [https://www.nuget.org/packages/SixLabors.ImageSharp.Drawing](https://www.nuget.org/packages/SixLabors.ImageSharp.Drawing) |
| 🔍 **API Reference** | [https://docs.sixlabors.com/api/ImageSharp/](https://docs.sixlabors.com/api/ImageSharp/) |

### 📄 مستندات مایکروسافت

| موضوع | لینک |
|-------|------|
| ⚠️ **System.Drawing منسوخ شده** | [https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/6.0/system-drawing-common-windows-only](https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/6.0/system-drawing-common-windows-only) |
| 🖼️ **پردازش تصویر در ASP.NET Core** | [https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/file-uploads](https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/file-uploads) |

### 💬 انجمن و پشتیبانی

| منبع | لینک |
|------|------|
| 💬 **GitHub Discussions** | [https://github.com/SixLabors/ImageSharp/discussions](https://github.com/SixLabors/ImageSharp/discussions) |
| 🐛 **Issues** | [https://github.com/SixLabors/ImageSharp/issues](https://github.com/SixLabors/ImageSharp/issues) |
| 🗨️ **Stack Overflow (تگ imagesharp)** | [https://stackoverflow.com/questions/tagged/imagesharp](https://stackoverflow.com/questions/tagged/imagesharp) |

### 📝 مقالات و آموزش‌ها

| عنوان | لینک |
|-------|------|
| 📖 **ImageSharp Documentation - Getting Started** | [https://docs.sixlabors.com/articles/imagesharp/](https://docs.sixlabors.com/articles/imagesharp/) |
| 📖 **ImageSharp.Web Documentation** | [https://docs.sixlabors.com/articles/imagesharp.web/](https://docs.sixlabors.com/articles/imagesharp.web/) |
| 📖 **Drawing API Documentation** | [https://docs.sixlabors.com/articles/drawing/](https://docs.sixlabors.com/articles/drawing/) |

### 📌 لایسنس

| پکیج | لایسنس | توضیح |
|------|---------|--------|
| SixLabors.ImageSharp | **Apache 2.0** | رایگان برای استفاده تجاری و شخصی |
| SixLabors.ImageSharp.Drawing | **Apache 2.0** | رایگان |
| SixLabors.ImageSharp.Web | **Apache 2.0** | رایگان |

> 📄 متن کامل لایسنس: [https://github.com/SixLabors/ImageSharp/blob/main/LICENSE](https://github.com/SixLabors/ImageSharp/blob/main/LICENSE)

---

## 📋 خلاصه سریع دستورات

```csharp
// بارگذاری
using var img = Image.Load("input.jpg");

// تغییر اندازه
img.Mutate(x => x.Resize(800, 0));

// فیلتر
img.Mutate(x => x.Grayscale().Brightness(1.1f));

// برش
img.Mutate(x => x.Crop(new Rectangle(0, 0, 500, 500)));

// چرخش
img.Mutate(x => x.Rotate(90));

// ذخیره
img.Save("output.webp", new WebpEncoder { Quality = 80 });
```

---

> ✍️ **تهیه‌کننده:** این مستند برای آموزش در ریپازیتوری آموزشی گردآوری شده است.
>
> 📅 **آخرین به‌روزرسانی:** اوت ۲۰۲۶
>
> 📦 **نسخه ImageSharp مرجع:** 3.x

---

*اگر این مستند مفید بود، لطفاً ⭐ ستاره بدهید!*
