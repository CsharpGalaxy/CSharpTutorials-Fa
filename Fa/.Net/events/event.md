
# راهنمای جامع معماری رویداد-محور: Event، Message، Command و انواع Event

این یک راهنمای جامع، ساختاریافته و گام‌به‌گام برای درک مفاهیم پایه تا پیشرفته در معماری نرم‌افزار توزیع‌شده و Event-Driven Architecture (EDA) است.

---

## 📑 فهرست مطالب (Table of Contents)

- [بخش 1 — مفاهیم پایه](#بخش-1--مفاهیم-پایه)
  - [1. Event چیست؟](#1-event-چیست)
  - [2. Message چیست؟](#2-message-چیست)
  - [3. Command چیست؟](#3-command-چیست)
  - [4. تفاوت Command و Event](#4-تفاوت-command-و-event)
- [بخش 2 — ساختار Message و Event](#بخش-2--ساختار-message-و-event)
  - [5. ساختار یک Event](#5-ساختار-یک-event)
  - [6. Metadata در Event](#6-metadata-در-event)
  - [7. Payload در Event](#7-payload-در-event)
- [بخش 3 — انواع Event (Event Notification)](#بخش-3--انواع-event)
  - [8. معرفی سه نوع اصلی Event](#8-معرفی-سه-نوع-اصلی-event)
  - [9. Event Notification](#9-event-notification)
  - [10. Event Notification و Query](#10-event-notification-و-query)
  - [11. Security در Event Notification](#11-security-در-event-notification)
  - [12. Concurrency و Event Notification](#12-concurrency-و-event-notification)
  - [13. Pessimistic Locking و Event Processing](#13-pessimistic-locking-و-event-processing)
- [بخش 4 — Event-Carried State Transfer (ECST)](#بخش-4--event-carried-state-transfer-ecst)
  - [14. ECST چیست؟](#14-ecst-چیست)
  - [15. طراحی Payload در ECST](#15-طراحی-payload-در-ecst)
  - [16. Local Cache با ECST](#16-local-cache-با-ecst)
  - [17. ECST و Data Replication](#17-ecst-و-data-replication)
  - [18. مشکلات ECST](#18-مشکلات-ecst)
- [بخش 5 — Domain Event](#بخش-5--domain-event)
  - [19. Domain Event چیست؟](#19-domain-event-چیست)
  - [20. Domain Event در DDD](#20-domain-event-در-ddd)
  - [21. Domain Event داخلی و Integration Event](#21-domain-event-داخلی-و-integration-event)
- [بخش 6 — Event Sourcing](#بخش-6--event-sourcing)
  - [22. Event Sourcing چیست؟](#22-event-sourcing-چیست)
  - [23. Domain Event و Event Sourcing](#23-domain-event-و-event-sourcing)
  - [24. Event Sourcing در برابر CRUD](#24-event-sourcing-در-برابر-crud)
- [بخش 7 — مباحث پیشرفته](#بخش-7--مباحث-پیشرفته)
  - [25. Event Ordering](#25-event-ordering)
  - [26. Duplicate Event و Idempotency](#26-duplicate-event-و-idempotency)
  - [27. Event Versioning و Schema Evolution](#27-event-versioning-و-schema-evolution)
  - [28. Event Delivery و Reliability](#28-event-delivery-و-reliability)
  - [29. مقایسه معماری و انتخاب نوع Event](#29-مقایسه-معماری-و-انتخاب-نوع-event)
- [بخش 8 — جمع‌بندی و طراحی یک سیستم واقعی](#بخش-8--جمع‌بندی-و-طراحی-یک-سیستم-واقعی)
- [پیوست‌ها](#پیوست‌ها)
  - [Cheat Sheet](#cheat-sheet)
  - [Comparison Table](#comparison-table)
  - [Real-World Architecture](#real-world-architecture)
  - [Common Mistakes](#common-mistakes)
  - [Exercises](#exercises)
  - [Interview Questions](#interview-questions)
  - [Final Summary](#final-summary)
  - [منابع](#منابع)

---

## بخش 1 — مفاهیم پایه

### 1. Event چیست؟
**تعریف ساده:** Event (رویداد) به معنای «یک اتفاق که در گذشته رخ داده است» می‌باشد.
**چرا وجود دارد؟** در سیستم‌های توزیع‌شده، سرویس‌ها نمی‌توانند دائماً وضعیت یکدیگر را چک کنند (Polling). Event به سرویس‌ها اجازه می‌دهد تا وقتی اتفاق مهمی افتاد، بقیه را مطلع کنند.

**تفاوت Event در معماری با `event` در C#:**
*   **در C#:** یک مکانیزم زبان (Language Feature) بر اساس Delegateها است برای ارتباط بین اشیاء در یک پروسس (In-Process).
*   **در معماری:** یک پیام (Message) است که از مرزهای سرویس عبور می‌کند، معمولاً از طریق Message Broker (مثل RabbitMQ یا Kafka) ارسال می‌شود و نشان‌دهنده یک تغییر وضعیت در Business است.

**مثال‌های واقعی:**
*   `OrderCreated` (سفارش ایجاد شد)
*   `PaymentCompleted` (پرداخت تکمیل شد)
*   `DeliveryConfirmed` (تحویل تایید شد)

### 2. Message چیست؟
**تعریف ساده:** Message (پیام) یک بسته داده (Data Package) است که بین دو سیستم یا سرویس رد و بدل می‌شود.
**رابطه Message، Command و Event:**
Message یک مفهوم عمومی (Superset) است. هر Command و هر Event، در واقع یک Message هستند. اما هر Message لزوماً Event یا Command نیست (ممکن است یک Document یا Request باشد).

```text
[ Message ]
   ├── Command (فرمان)
   ├── Event (رویداد)
   └── Document/Request (سند/درخواست)
```

### 3. Command چیست؟
**تعریف ساده:** Command (فرمان) پیامی است که به معنای «این کار را انجام بده» (Do this) است.
**ویژگی‌ها:**
*   یک **Request** (درخواست) است.
*   فقط باید توسط **یک** Consumer (مصرف‌کننده) پردازش شود.
*   **امکان Reject (رد) شدن دارد.**

**مثال:**
*   `CreateOrder` (سفارش را ایجاد کن)
*   `ProcessPayment` (پرداخت را پردازش کن)
*   **مثال خطا:** اگر `ProcessPayment` ارسال شود اما موجودی حساب کافی نباشد، Consumer آن را Reject کرده و یک خطا برمی‌گرداند.

### 4. تفاوت Command و Event

| ویژگی | Command (فرمان) | Event (رویداد) |
| :--- | :--- | :--- |
| **معنا** | انجام بده (Do this) | انجام شد (This happened) |
| **زمان** | آینده (Future) | گذشته (Past) |
| **مصرف‌کننده** | دقیقاً یک Consumer (1:1) | صفر، یک یا چند Consumer (1:N) |
| **قابلیت Reject** | بله، می‌تواند رد شود | خیر، گذشته را نمی‌توان تغییر داد |
| **جبران خطا** | برگشت خطا به Producer | نیاز به Compensating Action (رویداد جبرانی) |

**مفهوم Compensating Action:**
چون Event قابل Reject نیست، اگر اتفاقی افتاد که نباید می‌افتاد، باید یک Event جدید برای خنثی کردن آن منتشر کرد.
*   *مثال:* `PaymentCompleted` منتشر شده، اما کالا ناموجود است. سرویس پرداخت یک `RefundPayment` (Command) صادر می‌کند تا پول برگردد.

---

## بخش 2 — ساختار Message و Event

### 5. ساختار یک Event
یک Event استاندارد در معماری از دو بخش اصلی تشکیل شده است:
1.  **Metadata (فراداده):** اطلاعات درباره خود پیام.
2.  **Payload (داده اصلی):** اطلاعات مربوط به Business.

**اجزای Metadata:**
*   **Event Type:** نوع رویداد (مثلاً `OrderCreated`).
*   **Event ID / Message ID:** شناسه یکتای خود پیام (برای جلوگیری از پردازش تکراری).
*   **Correlation ID:** شناسه‌ای که کل جریان Business (از ابتدا تا انتها) را به هم متصل می‌کند.
*   **Causation ID:** شناسه پیامی که مستقیماً باعث تولید این Event شده است.
*   **Timestamp:** زمان دقیق رخ دادن اتفاق (نه زمان ارسال پیام).
*   **Version:** نسخه ساختار Event.
*   **Producer:** شناسه سرویس تولیدکننده.

### 6. Metadata در Event
**چرا Metadata لازم است؟** در Distributed Systems، یک درخواست ممکن است از 10 سرویس عبور کند. Metadata برای **Distributed Tracing** و **Debugging** حیاتی است.

*   **Correlation ID:** مثلاً در یک فرآیند خرید، از لحظه کلیک روی دکمه خرید تا تحویل کالا، یک Correlation ID ثابت است. اگر لاگ‌ها را با این ID فیلتر کنید، کل مسیر را می‌بینید.
*   **Causation ID:** اگر `OrderCreated` باعث تولید `ReserveInventory` (Command) شود و آن Command باعث تولید `InventoryReserved` (Event) شود، Causation ID در Event آخر، برابر با Message ID مربوط به Command است.

### 7. Payload در Event
**Payload چیست؟** داده‌های Business که در Event حمل می‌شوند.
**قانون طلایی:** فقط اطلاعاتی را در Payload بگذارید که Consumer برای پردازش Event به آن **نیاز فوری** دارد.

**مثال C# برای ساختار Event:**
```csharp
// Base Record برای Metadata
public record EventMetadata(
    string MessageId,
    string CorrelationId,
    string CausationId,
    DateTimeOffset Timestamp,
    int Version,
    string Producer
);

// Event Payload
public record OrderCreatedPayload(
    Guid OrderId,
    Guid CustomerId,
    decimal TotalAmount,
    DateTimeOffset OrderDate
);

// Event کامل
public record OrderCreatedEvent(
    EventMetadata Metadata,
    OrderCreatedPayload Payload
);
```

---

## بخش 3 — انواع Event

### 8. معرفی سه نوع اصلی Event
در معماری، Events را بر اساس **مقدار داده‌ای که حمل می‌کنند** و **هدفشان** به سه دسته تقسیم می‌کنیم:
1.  **Event Notification:** فقط خبر می‌دهد که اتفاقی افتاده است.
2.  **Event-Carried State Transfer (ECST):** علاوه بر خبر، State (وضعیت) را هم حمل می‌کند.
3.  **Domain Event:** مفهومی در DDD است و نشان‌دهنده یک تغییر وضعیت مهم در مدل Domain است.

### 9. Event Notification
**تعریف:** Eventی که فقط می‌گوید «اتفاقی افتاد» و حداقل اطلاعات ممکن را دارد.
**مثال:** `PaycheckGenerated` (فیش حقوقی تولید شد). Payload فقط شامل `EmployeeId` و `PaycheckId` است.
**نحوه دریافت اطلاعات بیشتر:** Consumer پس از دریافت Event، باید Producer را **Query** کند (مثلاً API صدای بزند) تا جزئیات فیش حقوقی را بگیرد.

**مزایا:** Payload بسیار کوچک، امنیت بالا.
**معایب:** وابستگی Runtime به Producer، افزایش ترافیک شبکه (Query).

### 10. Event Notification و Query
**الگوی Event → Query:**
1. سرویس A یک Event Notification کوچک منتشر می‌کند.
2. سرویس B Event را می‌گیرد.
3. سرویس B یک درخواست HTTP/gRPC به سرویس A می‌فرستد تا داده کامل را بگیرد.

**Consistency و Freshness:** چون بین دریافت Event و Query کردن فاصله زمانی وجود دارد، ممکن است داده‌ای که Query می‌کنید با داده‌ای که Event را تولید کرده متفاوت باشد (Stale Data).

### 11. Security در Event Notification
**مزیت امنیتی:** چون Payload کوچک است، داده‌های حساس (مثل شماره کارت، رمز عبور، اطلاعات شخصی) در Message Broker منتشر نمی‌شوند.
**Authorization هنگام Query:** وقتی Consumer برای گرفتن داده کامل به Producer درخواست می‌زند، Producer می‌تواند بررسی کند که آیا Consumer مجاز به دیدن این داده‌ها هست یا خیر.

### 12. Concurrency و Event Notification
**مشکل Stale Data و Race Condition:**
فرض کنید `CustomerUpdated` (Notification) ارسال می‌شود. قبل از اینکه Consumer آن را Query کند، یک آپدیت دیگر رخ می‌دهد. Consumer با Query کردن، داده جدیدتر را می‌گیرد و آپدیت قبلی را از دست می‌دهد.
**راه‌حل:** استفاده از Sequence Number در Event یا استفاده از ECST.

### 13. Pessimistic Locking و Event Processing
**Pessimistic Locking (قفل بدبینانه):** یعنی قبل از انجام کار، داده را قفل کن تا کس دیگری تغییرش ندهد.
**تفاوت Lock کردن داده با Lock کردن Message:**
*   در دیتابیس: `SELECT ... FOR UPDATE`
*   در Message Broker: وقتی یک Consumer یک پیام را از Queue برمی‌دارد، آن پیام برای بقیه Consumers مخفی می‌شود (Message Lock).

**مزایا:** جلوگیری از پردازش همزمان (Duplicate Processing).
**معایب:** کاهش Throughput و Performance. در سیستم‌های Event-Driven معمولاً از **Optimistic Concurrency** (استفاده از Version یا ETag) استفاده می‌شود.

---

## بخش 4 — Event-Carried State Transfer (ECST)

### 14. ECST چیست؟
**تعریف:** در ECST، Event فقط یک خبر نیست؛ بلکه **حامل State (وضعیت)** است. Consumer با دریافت Event، نیازی به Query کردن ندارد و می‌تواند State محلی (Local Cache) خود را آپدیت کند.
**مثال:** `CustomerUpdated` به همراه تمام اطلاعات جدید مشتری در Payload.

### 15. طراحی Payload در ECST
دو رویکرد وجود دارد:
1.  **Full State Event:** کل State ارسال می‌شود (مثلاً تمام فیلدهای Customer).
2.  **Partial State Event (Delta):** فقط فیلدهایی که تغییر کرده‌اند ارسال می‌شوند (مثلاً فقط `Email` و `PhoneNumber`).

**Versioning:** در ECST چون Payload بزرگ است، تغییر ساختار آن (Schema Evolution) بسیار مهم است و باید حتماً از `Version` در Metadata استفاده شود.

### 16. Local Cache با ECST
**نحوه کار:**
```text
[Producer DB] --> [Event] --> [Consumer] --> [Consumer Local DB / Cache]
```
Consumer با دریافت ECSTها، یک کپی از داده‌های Producer را در دیتابیس خود (Materialized View) نگه می‌دارد.
**مزایا:** حذف Queryهای مکرر، کاهش Coupling در زمان Runtime (اگر Producer دان شود، Consumer هنوز داده‌های قبلی را دارد).

### 17. ECST و Data Replication
ECST در واقع یک **Asynchronous Data Replication** (همگام‌سازی غیرهمزمان داده) است.
**Eventual Consistency (سازگاری در نهایت):** داده در Producer آپدیت می‌شود، اما مدتی طول می‌کشد تا به Consumer برسد. در این فاصله، داده در Consumer قدیمی (Stale) است، اما در نهایت (Eventually) با Producer همگام می‌شود.
**Fault Tolerance:** اگر سرویس A دان شود، سرویس B به کار خود ادامه می‌دهد چون داده‌ها را به صورت محلی دارد.

### 18. مشکلات ECST
1.  **Data Duplication:** داده در چندین جا کپی می‌شود (افزایش هزینه ذخیره‌سازی).
2.  **Schema Evolution:** تغییر ساختار Payload بسیار سخت است.
3.  **Out-of-Order Events:** اگر Event شماره 2 قبل از Event شماره 1 برسد، State محلی خراب می‌شود.
4.  **Duplicate Events:** اگر یک Event دو بار برسد، State باید Idempotent باشد.

---

## بخش 5 — Domain Event

### 19. Domain Event چیست؟
**تعریف:** رویدادی که نشان‌دهنده یک تغییر وضعیت مهم در **Business (دامنه)** است. این رویداد برای Domain Expertها معنی دارد.
**تفاوت با Integration Event:**
*   **Domain Event:** مربوط به داخل یک Bounded Context است (مثلاً `OrderConfirmed`).
*   **Integration Event:** برای ارتباط بین Bounded Contextها یا سرویس‌های مختلف استفاده می‌شود و ساختار آن برای مصرف‌کننده بیرونی بهینه‌سازی شده است.

### 20. Domain Event در DDD
در Domain-Driven Design (DDD)، Domain Eventها بخشی از مدل Domain هستند.
*   **Aggregate:** تغییرات State در Aggregate رخ می‌دهد.
*   **ایجاد Event:** وقتی یک Aggregate تغییر می‌کند، یک Domain Event تولید و درون خود Aggregate نگه‌داری می‌شود.
*   **انتشار:** پس از اینکه Aggregate با موفقیت در دیتابیس ذخیره شد (Commit)، Domain Eventها منتشر می‌شوند.

### 21. Domain Event داخلی و Integration Event
**آیا هر Domain Event باید به Message Broker ارسال شود؟** خیر!
بسیاری از Domain Eventها فقط برای ارتباط بین Aggregates در همان سرویس هستند (Internal).
**تبدیل به Integration Event:** فقط Eventهایی که سرویس‌های دیگر به آن نیاز دارند، توسط یک **Anti-Corruption Layer** یا یک Mapper به Integration Event تبدیل شده و به Broker ارسال می‌شوند.

```csharp
// Domain Event (داخلی)
public class OrderConfirmedDomainEvent { ... }

// Integration Event (خروجی به Broker)
public record OrderConfirmedIntegrationEvent(Guid OrderId, decimal TotalAmount);
```

---

## بخش 6 — Event Sourcing

### 22. Event Sourcing چیست؟
**تعریف:** به جای ذخیره State نهایی (مثل `CurrentBalance = 100`)، تمام Eventهایی که منجر به این State شده‌اند را ذخیره می‌کنیم (مثل `Deposited(50)`, `Withdrew(20)`, `Deposited(70)`).
**Event Store:** دیتابیسی که فقط Eventها را به صورت Append-Only (فقط افزودنی) ذخیره می‌کند.
**بازسازی State (Rehydration):** برای فهمیدن State فعلی، تمام Eventها را از ابتدا تا انتها Replay (بازپخش) می‌کنیم.

### 23. Domain Event و Event Sourcing
در Event Sourcing، Domain Eventها هسته اصلی سیستم هستند.
*   `OrderCreated`
*   `ItemAddedToOrder`
*   `PaymentCompleted`
*   `OrderShipped`

**تفاوت با EDA:** در EDA معمولی، Eventها برای ارتباط بین سرویس‌ها هستند و State در دیتابیس ذخیره می‌شود. در Event Sourcing، Eventها خودِ دیتابیس هستند.

### 24. Event Sourcing در برابر CRUD

| ویژگی | CRUD سنتی (State-based) | Event Sourcing (Event-based) |
| :--- | :--- | :--- |
| **ذخیره‌سازی** | State نهایی (Update) | تاریخچه تغییرات (Append) |
| **Audit History** | نیاز به جدول لاگ جداگانه | خود Eventها تاریخچه هستند |
| **Debugging** | سخت (فقط State فعلی) | آسان (Replay Eventها) |
| **پیچیدگی** | کم | بسیار بالا |
| **Query** | آسان (Select from Table) | سخت (نیاز به CQRS و Read Models) |

**چه زمانی مناسب نیست؟** برای سیستم‌هایی که نیاز به Audit دقیق ندارند، یا تیم فنی تجربه کافی برای مدیریت پیچیدگی‌های آن (مثل Snapshotting) را ندارد.

---

## بخش 7 — مباحث پیشرفته

### 25. Event Ordering
**چرا ترتیب مهم است؟** اگر `ItemAdded` بعد از `OrderCreated` پردازش شود، سیستم خطا می‌دهد.
**راهکارها:**
1.  **Sequence Number:** اضافه کردن یک شماره ترتیب در Metadata.
2.  **Partitioning (در Kafka):** ارسال Eventهای مربوط به یک Entity (مثلاً یک Order) به یک Partition مشخص تا ترتیبشان حفظ شود.

### 26. Duplicate Event و Idempotency
**چرا Duplicate می‌شود؟** شبکه ممکن است ACK را گم کند، Broker پیام را دو بار تحویل دهد (At-Least-Once Delivery).
**Idempotency (توان‌تایی):** یعنی پردازش یک پیام برای 1 بار یا 100 بار، نتیجه یکسانی داشته باشد.
**راهکار:** استفاده از **Idempotency Key** (معمولاً همان `MessageId`). Consumer قبل از پردازش، چک می‌کند که آیا این `MessageId` قبلاً در دیتابیس پردازش شده است یا خیر.

### 27. Event Versioning و Schema Evolution
وقتی ساختار Event تغییر می‌کند (مثلاً فیلدی اضافه یا حذف می‌شود):
*   **Backward Compatibility:** نسخه جدید می‌تواند داده‌های نسخه قدیم را بخواند (اضافه کردن فیلد اختیاری).
*   **Forward Compatibility:** نسخه قدیم می‌تواند داده‌های نسخه جدید را بخواند (نادیده گرفتن فیلدهای جدید).
*   **Best Practice:** هرگز فیلدی را حذف یا تغییر نام ندهید. فقط فیلد جدید اضافه کنید و فیلد قدیم را Deprecate کنید.

### 28. Event Delivery و Reliability
*   **At-most-once:** پیام ممکن است گم شود (سریع اما غیرقابل اعتماد).
*   **At-least-once:** پیام گم نمی‌شود، اما ممکن است تکراری شود (رایج‌ترین حالت).
*   **Exactly-once:** بسیار سخت و پرهزینه در سیستم‌های توزیع‌شده. معمولاً با Idempotency در سمت Consumer شبیه‌سازی می‌شود.

**Transactional Outbox Pattern:**
برای اینکه مطمئن شویم اگر دیتابیس آپدیت شد، Event هم حتماً منتشر می‌شود:
1. تغییر State و نوشتن Event در یک جدول `Outbox` در **همان تراکنش دیتابیس** انجام می‌شود.
2. یک پروسه جداگانه (Outbox Reader) جدول Outbox را می‌خواند و Eventها را به Message Broker ارسال می‌کند.

### 29. مقایسه معماری و انتخاب نوع Event

| ویژگی | Event Notification | ECST | Domain Event | Integration Event | Event Sourcing |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **هدف** | اطلاع‌رسانی ساده | همگام‌سازی State | تغییر وضعیت در DDD | ارتباط بین سرویس‌ها | ذخیره تاریخچه کامل |
| **Payload** | بسیار کوچک (ID) | بزرگ (Full/Partial State) | مرتبط با Aggregate | بهینه برای Consumer | خودِ State است |
| **مصرف‌کننده** | Query می‌کند | State محلی را آپدیت می‌کند | داخل Bounded Context | سرویس‌های دیگر | Event Store |
| **Coupling** | بالا (Runtime) | متوسط (Schema) | پایین (داخل مرز) | پایین (قرارداد محور) | N/A |
| **Data Replication**| خیر | بله (Async) | خیر | خیر | بله (پایه سیستم) |
| **نیاز به Query** | بله | خیر | خیر | خیر | خیر (Replay) |
| **امنیت** | بالا (داده کم) | متوسط | بالا | متوسط | بالا |
| **Consistency** | Strong (با Query) | Eventual | Strong (در Aggregate)| Eventual | Eventual / Strong |

---

## بخش 8 — جمع‌بندی و طراحی یک سیستم واقعی

### سناریوی فروشگاه اینترنتی (E-Commerce)
سرویس‌ها: Customer, Order, Payment, Inventory, Shipping, Notification.

**جریان کار و تحلیل معماری:**

1.  **Command:** کاربر دکمه خرید را می‌زند. API Gateway یک `CreateOrderCommand` به **Order Service** می‌فرستد.
2.  **Domain Event:** Order Service سفارش را ایجاد می‌کند. یک `OrderCreatedDomainEvent` درون Aggregate تولید می‌شود.
3.  **Transactional Outbox:** این Event در جدول Outbox ذخیره و Commit می‌شود.
4.  **Integration Event:** یک Worker، Event را از Outbox خوانده، به `OrderCreatedIntegrationEvent` تبدیل و در Broker منتشر می‌کند.
    *   *Metadata:* شامل `CorrelationId` (شماره سفارش) و `CausationId` (آی‌دی Command).
5.  **Event Notification:** **Notification Service** یک `OrderCreatedNotification` (فقط شامل OrderId) دریافت می‌کند و برای آماده‌سازی ایمیل، Order Service را Query می‌کند.
6.  **ECST:** **Inventory Service** یک `OrderCreatedECST` (شامل لیست آیتم‌ها و تعداد) دریافت می‌کند و موجودی محلی (Local Cache) خود را کم می‌کند.
7.  **Idempotency:** اگر Broker این Event را دو بار بفرستد، Inventory Service با چک کردن `MessageId` در دیتابیس، از کم کردن مجدد موجودی جلوگیری می‌کند.
8.  **Compensating Action:** اگر **Payment Service** پول را کسر کند اما **Inventory Service** اعلام کند کالا ناموجود است، Payment Service یک `RefundCommand` دریافت می‌کند.

---

## پیوست‌ها

### Cheat Sheet
*   **Message:** بسته داده بین سرویس‌ها.
*   **Command:** "انجام بده" (Request, 1:1, قابل Reject).
*   **Event:** "انجام شد" (Notification, 1:N, غیرقابل Reject).
*   **Event Notification:** خبر خالی، نیاز به Query دارد.
*   **ECST:** خبر + State، برای ساخت Local Cache.
*   **Domain Event:** تغییر وضعیت در مدل DDD.
*   **Integration Event:** پیام برای ارتباط بین سرویس‌ها.
*   **Event Sourcing:** ذخیره Event به جای State.
*   **Outbox Pattern:** تضمین انتشار Event همگام با تغییر دیتابیس.
*   **Idempotency:** پردازش امن پیام‌های تکراری.

### Comparison Table
*(جدول مقایسه کامل در بخش 29 ارائه شده است.)*

### Real-World Architecture
```text
[Client] --> (Command) --> [API Gateway] --> [Order Service]
                                                  |
                                          (Transactional Outbox)
                                                  |
                                          [Message Broker (Kafka/RabbitMQ)]
                                                  |
            +----------------+--------------------+--------------------+
            |                |                    |                    |
      (ECST: Inventory) (Notification: Email) (Event: Payment)  (Event: Shipping)
            |                |                    |                    |
     [Inventory Svc]  [Notification Svc]   [Payment Svc]       [Shipping Svc]
     (Local Cache)      (Queries Order)    (Processes Pay)     (Schedules Ship)
```

### Common Mistakes
1.  **استفاده از Event برای درخواست کار:** هرگز `CreateOrder` را به عنوان Event منتشر نکنید. این یک Command است.
2.  **Fat Events (Eventهای چاق):** گذاشتن تمام اطلاعات دیتابیس در Payload. فقط چیزی را بفرستید که Consumer نیاز دارد.
3.  **نادیده گرفتن Idempotency:** فرض کردن اینکه پیام فقط یک بار می‌رسد. همیشه باید Duplicate را هندل کرد.
4.  **استفاده از Event Sourcing برای همه چیز:** این الگو پیچیدگی بالایی دارد. فقط برای جاهایی که Audit Trail یا Replay حیاتی است استفاده شود.
5.  **تغییر ساختار Event بدون Versioning:** شکستن Consumerهای قدیمی با حذف یا تغییر نام فیلدها.

### Exercises
1.  **سطح 1:** یک Record در C# برای `UserRegisteredEvent` با Metadata و Payload طراحی کنید.
2.  **سطح 1:** تفاوت Command و Event را در 3 جمله توضیح دهید.
3.  **سطح 2:** یک سناریو بنویسید که در آن استفاده از Event Notification بهتر از ECST است.
4.  **سطح 2:** چگونه یک Consumer را در C# نسبت به پیام‌های تکراری (Duplicate) Idempotent می‌کنید؟ (کد بزنید).
5.  **سطح 3:** الگوی Transactional Outbox را روی کاغذ طراحی کنید و جریان داده را توضیح دهید.
6.  **سطح 3:** تفاوت Correlation ID و Causation ID را با یک مثال از 3 سرویس متوالی نشان دهید.
7.  **سطح 4:** یک Event را از نسخه 1 به نسخه 2 ارتقا دهید به طوری که Backward و Forward Compatible باشد.
8.  **سطح 4:** چرا در ECST نباید از Pessimistic Locking روی داده‌های Local Cache استفاده کرد؟
9.  **سطح 5:** یک Aggregate در DDD طراحی کنید که هنگام تغییر State، یک Domain Event تولید کند.
10. **سطح 5:** معماری Event Sourcing را برای یک سیستم "بانکی" طراحی کنید و مفهوم Snapshot را توضیح دهید.

### Interview Questions
1.  **سؤال:** تفاوت اصلی Command و Event چیست؟
    *   **پاسخ:** Command یک درخواست برای انجام کار در آینده است و می‌تواند رد شود (1:1). Event گزارش کاری است که در گذشته انجام شده و نمی‌تواند رد شود (1:N).
2.  **سؤال:** Event Notification چیست و چه تفاوتی با ECST دارد؟
    *   **پاسخ:** Notification فقط خبر می‌دهد و Consumer باید برای داده Query بزند. ECST خودِ داده (State) را در Payload حمل می‌کند.
3.  **سؤال:** منظور از Eventual Consistency چیست؟
    *   **پاسخ:** یعنی داده در لحظه اول در همه جا یکسان نیست، اما پس از گذشت زمان و پردازش Eventها، در نهایت در همه سرویس‌ها همگام و یکسان می‌شود.
4.  **سؤال:** چگونه از پردازش تکراری یک Event جلوگیری می‌کنید؟
    *   **پاسخ:** با استفاده از Idempotency. Consumer باید `MessageId` را در یک جدول پردازش‌شده چک کند و اگر قبلاً پردازش شده بود، آن را نادیده بگیرد.
5.  **سؤال:** Transactional Outbox Pattern چه مشکلی را حل می‌کند؟
    *   **پاسخ:** مشکل Dual Write را حل می‌کند. تضمین می‌کند که اگر تغییر State در دیتابیس Commit شد، Event هم حتماً برای انتشار در Broker ثبت شود.
6.  **سؤال:** Correlation ID چه کاربردی دارد؟
    *   **پاسخ:** برای ردیابی یک جریان کامل Business در Distributed Tracing. تمام پیام‌های مرتبط با یک فرآیند (مثلاً یک خرید) یک Correlation ID یکسان دارند.
7.  **سؤال:** Domain Event با Integration Event چه فرقی دارد؟
    *   **پاسخ:** Domain Event برای ارتباطات داخلی یک Bounded Context است و به مدل Domain وابسته است. Integration Event برای ارتباط بین سرویس‌هاست و ساختاری مستقل و بهینه دارد.
8.  **سؤال:** چرا Eventها را نباید Reject کرد؟
    *   **پاسخ:** چون Event نشان‌دهنده یک واقعیت در گذشته است. گذشته قابل تغییر نیست. اگر اتفاقی اشتباه بوده، باید یک Event جبرانی (Compensating Action) منتشر شود.
9.  **سؤال:** Event Sourcing چه زمانی مناسب نیست؟
    *   **پاسخ:** زمانی که نیاز به Audit Trail نداریم، تیم فنی تجربه کافی ندارد، یا سیستم نیاز به Queryهای پیچیده و سریع روی State فعلی دارد (بدون پیاده‌سازی CQRS).
10. **سؤال:** اگر ساختار یک Event تغییر کند، چه باید کرد؟
    *   **پاسخ:** باید از Event Versioning استفاده کرد. فیلدهای قدیمی حذف نشوند، فقط فیلدهای جدید به صورت اختیاری (Optional) اضافه شوند تا Compatibility حفظ شود.
11. **سؤال:** تفاوت Pessimistic و Optimistic Concurrency در پردازش Event چیست؟
    *   **پاسخ:** Pessimistic داده را قبل از پردازش قفل می‌کند. Optimistic فرض می‌کند تداخلی رخ نمی‌دهد و در زمان Commit، Version یا Timestamp را چک می‌کند. در Event-Driven معمولاً از Optimistic یا Idempotency استفاده می‌شود.
12. **سؤال:** Dead Letter Queue (DLQ) چیست؟
    *   **پاسخ:** صفی برای نگهداری پیام‌هایی که پس از چندین بار تلاش (Retry) همچنان قابل پردازش نیستند (Poison Messages) تا بعداً توسط انسان یا سیستم دیگری بررسی شوند.
13. **سؤال:** چرا در ECST ممکن است Stale Data داشته باشیم؟
    *   **پاسخ:** چون ECST به صورت Async ارسال می‌شود. تا زمانی که Event به Consumer برسد و پردازش شود، ممکن است Producer داده را دوباره تغییر داده باشد.
14. **سؤال:** نقش Anti-Corruption Layer در تبدیل Eventها چیست؟
    *   **پاسخ:** این لایه Domain Eventهای داخلی را به Integration Eventهایی تبدیل می‌کند که با قرارداد (Contract) سرویس‌های بیرونی سازگار باشد و از نشت جزئیات داخلی Domain جلوگیری کند.
15. **سؤال:** At-least-once delivery یعنی چه و چگونه هندل می‌شود؟
    *   **پاسخ:** یعنی Broker تضمین می‌کند پیام گم نمی‌شود، اما ممکن است تکراری شود. با Idempotency در سمت Consumer هندل می‌شود.
16. **سؤال:** چگونه ترتیب Eventها را در Kafka حفظ می‌کنیم؟
    *   **پاسخ:** با استفاده از Partition Key. تمام Eventهای مربوط به یک Entity (مثلاً یک OrderId) به یک Partition مشخص ارسال می‌شوند تا ترتیبشان در آن Partition حفظ شود.
17. **سؤال:** Materialized View در معماری Event-Driven چیست؟
    *   **پاسخ:** یک نمای خواندنی (Read Model) که با دریافت ECSTها یا Eventها، به صورت Async در دیتابیس Consumer ساخته و آپدیت می‌شود تا Queryها سریع باشند.
18. **سؤال:** آیا می‌توانیم در یک سیستم هم از CRUD و هم از Event Sourcing استفاده کنیم؟
    *   **پاسخ:** بله. می‌توان از Event Sourcing برای نوشتن (Write) و Audit استفاده کرد و با استفاده از CQRS، State را در یک دیتابیس关系‌ای (CRUD) برای خواندن (Read) Materialize کرد.
19. **سؤال:** Poison Message چیست؟
    *   **پاسخ:** پیامی که به دلیل نقص در ساختار (Schema) یا باگ در کد Consumer، هرگز قابل پردازش نیست و باعث خطای مکرر می‌شود. باید به DLQ منتقل شود.
20. **سؤال:** بهترین روش برای لاگ‌گیری در سیستم‌های Event-Driven چیست؟
    *   **پاسخ:** استفاده از Structured Logging و حتماً شامل کردن `CorrelationId`، `MessageId` و `EventType` در تمام لاگ‌ها برای قابلیت ردیابی.

### Final Summary
معماری Event-Driven و استفاده از Messageها (Command و Event) قلب تپنده سیستم‌های توزیع‌شده مدرن و میکروسرویس‌هاست.
*   از **Command** برای درخواست تغییر وضعیت استفاده کنید.
*   از **Event** برای اطلاع‌رسانی تغییرات گذشته استفاده کنید.
*   **Event Notification** را برای اطلاع‌رسانی سبک و **ECST** را برای همگام‌سازی داده‌ها و کاهش کوپلینگ Runtime انتخاب کنید.
*   **Domain Event** را برای مدل‌سازی Business در DDD به کار ببرید و آن را به **Integration Event** برای ارتباط بین سرویس‌ها تبدیل کنید.
*   همیشه **Idempotency**، **Versioning** و **Transactional Outbox** را در طراحی خود لحاظ کنید تا سیستمی مقاوم، قابل ردیابی و مقیاس‌پذیر بسازید.

### منابع
1.  **Microsoft Learn: Event-driven architecture style**
    *   *لینک:* [Microsoft Docs - Event-driven architecture](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven)
2.  **Book: Implementing Domain-Driven Design**
    *   *نویسنده:* Vaughn Vernon
    *   *توضیح:* مرجع اصلی برای درک Domain Event و Aggregateها.
3.  **Book: Designing Event-Driven Systems**
    *   *نویسنده:* Ben Stopford (Confluent)
    *   *توضیح:* عالی برای درک Event Sourcing، ECST و Kafka.
    *   *لینک:* [Confluent - Designing Event-Driven Systems](https://www.confluent.io/designing-event-driven-systems/)
4.  **Book: Building Microservices**
    *   *نویسنده:* Sam Newman
    *   *توضیح:* فصل‌های مربوط به ارتباط بین سرویس‌ها و Orchestrations vs Choreography.
5.  **Article: Transactional Outbox Pattern**
    *   *منبع:* Microsoft Architecture Center
    *   *لینک:* [Transactional Outbox Pattern](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/enterprise-integration/queue-message-bridge)
6.  **Udi Dahan's Blog: Sagas, Commands, and Events**
    *   *لینک:* [Udi Dahan - Commands and Events](https://udidahan.com/2009/02/19/commands-and-events/)
    *   *توضیح:* یکی از بهترین منابع برای درک تفاوت عمیق Command و Event.