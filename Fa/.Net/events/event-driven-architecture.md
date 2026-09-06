
# Event-Driven Architecture و Event-Driven Integration

این مستند یک راهنمای جامع، مرحله‌به‌مرحله و مبتنی بر منابع معتبر (مانند Martin Fowler و مستندات رسمی Microsoft) برای درک و پیاده‌سازی معماری رویدادمحور (Event-Driven Architecture) و یکپارچه‌سازی رویدادمحور (Event-Driven Integration) است. این راهنما از مفاهیم پایه شروع شده و به الگوهای پیشرفته طراحی در سیستم‌های توزیع‌شده می‌رسد.

---

## فهرست مطالب

1. [مقدمه](#1-مقدمه)
2. [Event چیست؟](#2-event-چیست)
3. [Event در مقابل Command](#3-event-در-مقابل-command)
4. [Event Notification](#4-event-notification)
5. [Event-Carried State Transfer (ECST)](#5-event-carried-state-transfer-ecst)
6. [Domain Event](#6-domain-event)
7. [Domain Event در مقابل ECST](#7-domain-event-در-مقابل-ecst)
8. [Domain Event در مقابل Integration Event](#8-domain-event-در-مقابل-integration-event)
9. [Event-Driven Integration چیست؟](#9-event-driven-integration-چیست)
10. [Event به عنوان First-Class Design Element](#10-event-به-عنوان-first-class-design-element)
11. [Boundary و Event](#11-boundary-و-event)
12. [Distributed Big Ball of Mud](#12-distributed-big-ball-of-mud)
13. [Functional Coupling](#13-functional-coupling)
14. [Implementation Coupling](#14-implementation-coupling)
15. [Temporal Coupling](#15-temporal-coupling)
16. [Refactoring Event-Driven Integration](#16-refactoring-event-driven-integration)
17. [Integration Model](#17-integration-model)
18. [Published Language](#18-published-language)
19. [Consumer-Driven Contract](#19-consumer-driven-contract)
20. [انتقال Projection به Producer](#20-انتقال-projection-به-producer)
21. [Event Notification برای حل Temporal Coupling](#21-event-notification-برای-حل-temporal-coupling)
22. [معماری Refactored نهایی](#22-معماری-refactored-نهایی)
23. [مقایسه Before / After](#23-مقایسه-before--after)
24. [چه زمانی از کدام Event استفاده کنیم؟](#24-چه-زمانی-از-کدام-event-استفاده-کنیم)
25. [اشتباهات رایج (Common Mistakes)](#25-اشتباهات-رایج-common-mistakes)
26. [Idempotency](#26-idempotency)
27. [Eventual Consistency](#27-eventual-consistency)
28. [Reliability](#28-reliability)
29. [Outbox Pattern](#29-outbox-pattern)
30. [Versioning](#30-versioning)
31. [Testing](#31-testing)
32. [Observability](#32-observability)
33. [Production Considerations](#33-production-considerations)
34. [مثال کامل از یک سیستم واقعی (E-Commerce)](#34-مثال-کامل-از-یک-سیستم-واقعی-e-commerce)
35. [تحلیل عمیق مثال CRM](#35-تحلیل-عمیق-مثال-crm)
36. [سوالات مفهومی مهم (FAQ)](#36-سوالات-مفهومی-مهم-faq)
37. [Cheat Sheet](#37-cheat-sheet)
38. [منابع](#38-منابع)

---

## 1. مقدمه

### Event-Driven Architecture چیست؟
در یک سیستم Event-Driven (معماری رویدادمحور)، اجزای سیستم (Components) به جای فراخوانی مستقیم یکدیگر، بر اساس اتفاقاتی که رخ داده‌اند (Events) با یکدیگر ارتباط برقرار می‌کنند . 

- **Event چیست؟** گزارشی از یک تغییر وضعیت یا وقوع یک اتفاق مهم در گذشته است.
- **چرا به وجود آمد؟** برای حل مشکلات مقیاس‌پذیری (Scalability)، کاهش وابستگی (Decoupling) و افزایش تاب‌آوری (Resilience) در سیستم‌های توزیع‌شده.
- **چه مشکلاتی را حل می‌کند؟** وابستگی زمانی (Temporal Coupling) و وابستگی مستقیم بین سرویس‌ها را حذف می‌کند.
- **چه مشکلات جدیدی ایجاد می‌کند؟** پیچیدگی در ردیابی خطاها (Observability)، مدیریت سازگاری نهایی (Eventual Consistency) و اطمینان از تحویل پیام (Message Delivery Guarantees).
- **چه زمانی مناسب است؟** زمانی که سیستم‌های متعددی باید از یک تغییر وضعیت آگاه شوند بدون اینکه Producer بداند Consumerها چه کسانی هستند.
- **چه زمانی بیش از حد پیچیده است؟** برای فرآیندهای ساده، همگام (Synchronous) یا زمانی که Consistency قوی (Strong Consistency) الزامی است.

---

## 2. Event چیست؟

یک Event یک رکورد از یک اتفاق است که **قبلاً رخ داده است**. نام‌گذاری آن معمولاً به صورت فعل گذشته (Past Tense) است.

مثال:
```text
OrderCreated
PaymentCompleted
CustomerRegistered
OrderCancelled
```

ویژگی‌های کلیدی Event:
- اتفاقی است که در گذشته رخ داده و غیرقابل تغییر (Immutable) است.
- با **Command** (دستور انجام کار) تفاوت بنیادین دارد.
- نباید با **Request** (درخواست همگام) اشتباه گرفته شود، زیرا Event معمولاً به صورت ناهمگام (Asynchronous) پردازش می‌شود.

---

## 3. Event در مقابل Command

تفاوت این دو در "قصد" (Intent) آن‌هاست.

### Command
```text
CreateOrder
CancelOrder
PayOrder
```
معنی: **«این کار را انجام بده.»** (دستوری برای آینده). اگر Command شکست بخورد، فرستنده باید خطا را مدیریت کند.

### Event
```text
OrderCreated
OrderCancelled
PaymentCompleted
```
معنی: **«این اتفاق افتاد.»** (گزارشی از گذشته). فرستنده Event معمولاً اهمیتی نمی‌دهد که چه کسی آن را دریافت می‌کند یا آیا با شکست مواجه می‌شود (Fire and Forget).

| ویژگی | Command | Event |
|---|---|---|
| زمان | آینده (انجام بده) | گذشته (انجام شد) |
| هدف | درخواست تغییر وضعیت | اعلام تغییر وضعیت |
| گیرنده | معمولاً یک گیرنده مشخص | صفر، یک یا چندین گیرنده نامشخص |
| شکست | فرستنده باید خطا را مدیریت کند | فرستنده معمولاً از شکست گیرنده بی‌خبر است |

---

## 4. Event Notification

Event Notification ساده‌ترین الگوی رویداد است. در این الگو، Event فقط اعلام می‌کند که "چیزی اتفاق افتاده است" و حداقل داده ممکن را حمل می‌کند .

مثال:
```json
{
  "type": "marriage-recorded",
  "personId": "123",
  "timestamp": "2026-09-07T10:00:00Z"
}
```

- **چیست؟** فقط یک اعلان (Notification) از وقوع یک اتفاق.
- **چرا اطلاعات کمی دارد؟** برای جلوگیری از نشت Internal Model و کاهش حجم پیام.
- **Consumer چگونه اطلاعات بیشتر دریافت می‌کند؟** Consumer پس از دریافت Notification، باید یک درخواست (API Call) به Producer بزند تا جزئیات را دریافت کند (Callback/Fetch).
- **مزایا:** Coupling بسیار پایین، حجم پیام کم.
- **معایب:** ایجاد ترافیک شبکه اضافی (Fetch)، وابستگی به در دسترس بودن Producer برای Fetch.
- **نوع Coupling:** اگر Eventها بخشی از یک Workflow پیچیده شوند، دنبال کردن جریان سیستم دشوار می‌شود، اما از نظر داده‌ای Coupling پایینی دارد.

---

## 5. Event-Carried State Transfer یا ECST

در الگوی ECST، Event نه‌تنها وقوع اتفاق را اعلام می‌کند، بلکه **تغییرات State (وضعیت)** را نیز در خود حمل می‌کند .

مثال:
```json
{
  "type": "personal-details-changed",
  "personId": "123",
  "newLastName": "Williams",
  "timestamp": "2026-09-07T10:00:00Z"
}
```

- **چیست؟** انتقال تکه‌ای از State به همراه Event.
- **چرا State را داخل Event قرار می‌دهیم؟** تا Consumer بتواند یک کپی محلی (Local Cache / Local State) از داده‌های مورد نیاز خود بسازد.
- **Consumer چگونه Local State می‌سازد؟** با گوش دادن به تمام Eventهای مرتبط و به‌روزرسانی دیتابیس یا کش محلی خود.
- **مزایا:** کاهش Latency (چون نیازی به API Call نیست)، کاهش وابستگی به Producer در زمان Query، افزایش Resilience (اگر Producer دان شود، Consumer همچنان با داده‌های محلی کار می‌کند).
- **معایب:** نگهداری چند نسخه از State و هماهنگ نگه داشتن آن‌ها (Consistency) پیچیدگی (Complexity) ایجاد می‌کند.

---

## 6. Domain Event

Domain Event بیانگر یک اتفاق مهم و معنادار در **Business Domain** (حوزه کسب‌وکار) است .

مثال:
```json
{
  "type": "customer-married",
  "personId": "123",
  "assumedPartnerLastName": true,
  "timestamp": "2026-09-07T10:00:00Z"
}
```

- **چیست؟** اتفاقی که برای متخصصان حوزه کسب‌وکار (Domain Experts) معنی دارد.
- **چرا Business Meaning مهم است؟** چون نام و محتوای آن باید بخشی از Ubiquitous Language (زبان فراگیر) باشد.
- **چه اطلاعاتی باید داشته باشد؟** شناسه موجودیت (Entity ID)، نوع اتفاق، و داده‌های ضروری برای درک آن اتفاق در آن لحظه.
- **چه اطلاعاتی نباید حمل کند؟** داده‌هایی که صرفاً برای راحتی Consumer اضافه شده‌اند اما ربطی به آن اتفاق خاص ندارند (این کار باعث نشت Internal Model می‌شود).
- **تفاوت با State Snapshot:** Domain Event یک "اتفاق" است، نه یک "عکس فوری از کل وضعیت". مثلاً `CustomerAddressChanged` یک Domain Event است، اما ارسال کل پروفایل مشتری به عنوان Event، یک State Snapshot است که معمولاً توصیه نمی‌شود مگر در الگوی ECST.

---

## 7. Domain Event در مقابل ECST

این تمایز برای طراحی صحیح حیاتی است.

| ویژگی | Domain Event | ECST |
|---|---|---|
| **تمرکز** | Business Event (اتفاق تجاری) | State (وضعیت داده) |
| **هدف** | بیان وقوع یک اتفاق معنادار | انتقال تغییرات داده برای به‌روزرسانی Local State |
| **مناسب برای Cache** | معمولاً خیر | بله، هدف اصلی آن همین است |
| **Consumer** | واکنش به یک اتفاق تجاری (مثلاً ارسال ایمیل) | ساخت و به‌روزرسانی Local State (مثلاً به‌روزرسانی نمای مشتری) |
| **وابستگی** | Business-oriented | Data-oriented |

**مثال ازدواج:**
- **Event Notification:** `marriage-recorded` (فقط می‌گوید ازدواج ثبت شد).
- **ECST:** `personal-details-changed` (می‌گوید نام خانوادگی به X تغییر کرد تا Consumer کش خود را آپدیت کند).
- **Domain Event:** `customer-married` (بیانگر یک قانون یا اتفاق تجاری خاص است که ممکن است منجر به تغییر نام خانوادگی *یا* اقدامات دیگر شود).

---

## 8. Domain Event در مقابل Integration Event

بر اساس مستندات رسمی Microsoft، تمایز بین این دو بسیار حیاتی است .

### Domain Event
- **Scope:** داخل یک Bounded Context یا Microservice خاص.
- **هدف:** اطلاع‌رسانی به اجزای داخلی همان سرویس (مثلاً به‌روزرسانی یک Aggregate دیگر یا تریگر کردن یک Domain Service).
- **زمان ارسال:** درون همان Transaction دیتابیس محلی.
- **Event Bus:** معمولاً In-Memory (مثل MediatR در .NET).

### Integration Event
- **Scope:** عبور از مرز سرویس‌ها (Cross-Boundary) یا Bounded Contextها.
- **هدف:** هماهنگ کردن سیستم‌های مختلف و نگه‌داشتن Consistency در سطح سیستم توزیع‌شده.
- **زمان ارسال:** پس از موفقیت‌آمیز بودن Transaction محلی (معمولاً با الگوی Outbox).
- **Event Bus:** پیام‌رسان‌های خارجی (مثل RabbitMQ, Kafka, Azure Service Bus).

> **قانون طلایی:** یک Domain Event هرگز نباید مستقیماً به عنوان Integration Event منتشر شود. باید توسط یک Translator به Integration Event تبدیل شود .

---

## 9. Event-Driven Integration چیست؟

استفاده از Eventها برای هماهنگی و تبادل داده بین Bounded Contextها و Microserviceها بدون فراخوانی مستقیم (Direct API Call).

مثال ساده:
```text
CRM
 │
 │ (Integration Event)
 ▼
Marketing
```

مثال چند‌مصرف‌کننده (Pub/Sub):
```text
CRM
 │
 │ (Integration Event: CustomerUpdated)
 ├────────► Marketing
 │
 └────────► AdsOptimization
```
در اینجا CRM نیازی ندارد بداند Marketing یا AdsOptimization وجود دارند. این اوج Decoupling است.

---

## 10. Event به عنوان First-Class Design Element

Event فقط یک "پیام" (Message) فنی نیست؛ بلکه بخشی از **Design سیستم** است. طراحی Event بر موارد زیر تأثیر مستقیم دارد:
- **Boundary:** مشخص می‌کند چه چیزی از مرز سرویس عبور می‌کند.
- **Coupling:** تعیین می‌کند سرویس‌ها چقدر به هم وابسته هستند.
- **استقلال Bounded Context:** اجازه می‌دهد هر سرویس با سرعت خود تکامل یابد (Evolution).
اگر Eventها بد طراحی شوند، سیستم به جای Decoupled شدن، به صورت پنهانی (Implicitly) به شدت Coupled می‌شود.

---

## 11. Boundary و Event

در معماری نرم‌افزار باید مشخص کنیم چه چیزی داخل Boundary است و چه چیزی اجازه عبور از آن را دارد .

```text
CRM Bounded Context
-------------------
[Internal Domain Model]
[Domain Events]
[Business Rules]
[Repositories]
        │
        │ (Integration Contract / Published Language)
        ▼
[Marketing Bounded Context]
```
**چرا نباید Internal Model را مستقیماً نشان داد؟** چون Internal Model مدام در حال تغییر (Refactoring) است. اگر Consumerها به آن وابسته شوند، هر تغییر کوچک در CRM باعث شکستن (Breaking Change) سیستم‌های دیگر می‌شود.

---

## 12. Distributed Big Ball of Mud

حتی با استفاده از Eventها، می‌توان یک سیستم به شدت Coupled ساخت که به آن Distributed Big Ball of Mud می‌گویند.

معماری بد:
```text
CRM
 │
 ├────────► Marketing (گوش می‌دهد به تمام Eventهای CRM)
 │
 └────────► AdsOptimization (گوش می‌دهد به تمام Eventهای CRM)
                    │
                    ▼ (گوش می‌دهد به Eventهای AdsOptimization)
                Reporting
```
**چرا این سیستم Strongly Coupled است؟** چون اگرچه ارتباط فیزیکی غیرهمگام است، اما وابستگی منطقی (Logical Dependency) شدیدی وجود دارد. Reporting نمی‌تواند کار کند مگر اینکه AdsOptimization کارش را تمام کرده باشد.

---

## 13. Functional Coupling

این نوع Coupling زمانی رخ می‌دهد که چندین Consumer، **منطق تجاری یکسانی** را برای تفسیر Eventها پیاده‌سازی کنند.

مثال:
هم Marketing و هم AdsOptimization برای ساختن "نمای مشتری" (Customer Projection)، قوانین پیچیده‌ای را اجرا می‌کنند:
```text
CRM Domain Events (مثلاً AddressChanged, NameChanged, Merged)
        ↓
[Logic: ساخت Customer Projection] ← در Marketing
[Logic: ساخت Customer Projection] ← در AdsOptimization (تکراری!)
```
- **چرا ایجاد می‌شود؟** چون Producer فقط Eventهای خام (Low-level) را منتشر می‌کند و Consumerها مجبورند برای درک وضعیت نهایی، آن‌ها را تفسیر کنند.
- **چرا بد است؟** Duplicate Business Logic است. اگر قانون تغییر کند، باید در چندین جا اصلاح شود که منجر به ناهماهنگی می‌شود.

---

## 14. Implementation Coupling

زمانی رخ می‌دهد که Consumerها به **ساختار داخلی و جزئیات پیاده‌سازی** Producer وابسته شوند.

مثال:
```text
CRM
 ├── CustomerCreated
 ├── CustomerNameChanged
 ├── CustomerAddressChanged
 └── CustomerMerged
        │
        ├────► Marketing
        └────► AdsOptimization
```
> Consumerها به Implementation داخلی Producer وابسته شده‌اند.

- **تغییر Schema:** اگر CRM فیلد `newName` را به `name` تغییر دهد، تمام Consumerها می‌شکنند.
- **اضافه شدن Event جدید:** اگر `CustomerMerged` اضافه شود، Consumerهایی که Projection می‌سازند ممکن است فراموش کنند به آن گوش دهند و State نادرست (Inconsistent State) بسازند.

---

## 15. Temporal Coupling

وابستگی زمانی زمانی رخ می‌دهد که یک سرویس برای انجام کار خود، مجبور باشد منتظر بماند تا سرویس دیگری کارش را تمام کند.

سناریو:
```text
CRM → (Event) → AdsOptimization → (Event) → Reporting
```
اگر Reporting برای تولید گزارش روزانه نیاز به داده‌های AdsOptimization داشته باشد، مجبور است صبر کند (مثلاً 5 دقیقه Delay). 
- **چرا Delay نشانه مشکل است؟** چون سیستم‌ها باید مستقل باشند. وابستگی به زمان‌بندی یا ترتیب اجرای سیستم دیگر، مزیت اصلی Asynchronous بودن را از بین می‌برد و سیستم را شکننده (Fragile) می‌کند.

---

## 16. Refactoring Event-Driven Integration

**معماری بد (قبل از اصلاح):**
```text
CRM
 │
 │ (انتشار ALL Domain Events به صورت خام)
 ├────────► Marketing
 │
 └────────► AdsOptimization
```
**چرا باید اصلاح شود؟** چون باعث Functional، Implementation و Temporal Coupling می‌شود. ما باید مسئولیت ساخت State مناسب را از Consumerها بگیریم و به Producer بازگردانیم.

---

## 17. Integration Model

> Internal Domain Model نباید الزاماً همان Integration Model باشد.

معماری اصلاح‌شده:
```text
CRM
 │
 ├── Internal Domain Model (پیچیده، در حال تغییر)
 │
 └── Integration Model (ساده، پایدار، مخصوص مصرف‌کنندگان خارجی)
          │
          ├────► Marketing
          └────► AdsOptimization
```
- **Integration Model چیست؟** یک نمای (Projection) از داده‌ها که به‌طور خاص برای نیازهای سیستم‌های خارجی آماده شده است.
- **چرا باید جدا باشد؟** تا Internal Model بتواند بدون ترس از شکستن سیستم‌های دیگر Refactor شود.
- **چه چیزی در آن قرار می‌گیرد؟** فقط داده‌هایی که Consumerها برای انجام کارشان نیاز دارند (نه بیشتر، نه کمتر).

---

## 18. Published Language

Published Language یک مدل داده‌ای پایدار و مستندشده است که به عنوان قرارداد ارتباطی (Communication Contract) بین Bounded Contextها استفاده می‌شود .

مثال:
```json
{
  "customerId": "123",
  "fullName": "Ali Ahmadi",
  "email": "ali@example.com",
  "status": "Active",
  "segment": "VIP"
}
```
این مدل دیگر `CustomerNameChanged` یا `CustomerMerged` نیست. این یک **State Snapshot** پایدار است که CRM متعهد می‌شود آن را به‌روز نگه دارد. این مدل بخشی از Published Language سیستم CRM است.

---

## 19. Consumer-Driven Contract

در الگوی Consumer-Driven Contract (CDC)، این Consumer است که مشخص می‌کند چه اطلاعاتی برای انجام کارش نیاز دارد، و Producer متعهد می‌شود که آن قرارداد (Contract) را تأمین کند .

- **Consumer:** نیاز خود را اعلام می‌کند (مثلاً "من به fullName و segment نیاز دارم").
- **Producer:** Integration Model را بر اساس این نیازها می‌سازد و منتشر می‌کند.
- **Contract:** توافق‌نامه‌ای (مثلاً فایل Pact یا Schema) که تضمین می‌کند Producer آن ساختار را رعایت می‌کند.
- **Versioning & Backward Compatibility:** Producer باید تضمین کند که تغییرات جدید، Consumerهای قدیمی را خراب نمی‌کند.

---

## 20. انتقال Projection به Producer

**مشکل قبلی (Functional Coupling):**
```text
Marketing        AdsOptimization
    ↓                ↓
Projection Logic   Projection Logic
(تفسیر 10 Event)   (تفسیر 10 Event)
```

**راه‌حل (انتقال به Producer):**
```text
CRM
 │
 │ (CRM مسئول ساخت Projection است)
 ▼
Integration Model (Published Language)
 │
 ├────► Marketing (فقط دریافت می‌کند، بدون منطق پیچیده)
 │
 └────► AdsOptimization (فقط دریافت می‌کند)
```
**چرا این کار Coupling را کاهش می‌دهد؟** چون منطق تجاری (Business Logic) تفسیر Eventها فقط در یک جا (CRM) وجود دارد. اگر قانونی تغییر کند، فقط CRM تغییر می‌کند و Consumerها بدون تغییر کد، داده‌های صحیح را دریافت می‌کنند.

---

## 21. Event Notification برای حل Temporal Coupling

برای حل مشکل وابستگی زمانی در سناریوی Reporting و AdsOptimization:

```text
AdsOptimization
       │
       │ (Event Notification: CalculationCompleted)
       ▼
Reporting
       │
       │ (API Call: Fetch)
       ▼
AdsOptimization (داده‌های نهایی را برمی‌گرداند)
```
- **Event Notification چه می‌گوید؟** "محاسبات من تمام شد و داده‌ها آماده‌اند."
- **چرا Data کامل را داخل Event قرار نمی‌دهیم؟** چون ممکن است حجم داده زیاد باشد یا Reporting به همه جزئیات نیاز نداشته باشد.
- **چرا Reporting بعد از Notification اطلاعات را Fetch می‌کند؟** تا اطمینان حاصل شود که داده‌ها کاملاً پردازش و پایدار (Committed) شده‌اند.
- **کاهش Temporal Coupling:** Reporting دیگر نیازی به Polling (پرس‌وجوی مداوم) یا حدس زدن زمان اتمام کار AdsOptimization ندارد. خود AdsOptimization اعلام آمادگی می‌کند.

---

## 22. معماری Refactored نهایی

```text
                         CRM
                          │
                 Internal Domain Model
                          │
                          ▼
                 Integration Projection (ساخت State نهایی)
                          │
                   Published Language (قرارداد پایدار)
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
            Marketing       AdsOptimization
                                   │
                                   │ (Notification: CalculationCompleted)
                                   ▼
                              Reporting
                                   │
                                   │ (Fetch: دریافت داده‌های نهایی)
                                   ▼
                              Required Data
```

**تفاوت‌های کلیدی:**
1. CRM دیگر Eventهای خام داخلی را پخش نمی‌کند.
2. CRM مسئولیت ساخت یک Integration Model پایدار را بر عهده گرفته است.
3. Consumerها دیگر منطق Projection را تکرار نمی‌کنند.
4. Reporting به جای وابستگی زمانی کور، از الگوی Notification + Fetch استفاده می‌کند.

---

## 23. مقایسه Before / After

| مورد | قبل (Bad Architecture) | بعد (Refactored Architecture) |
|---|---|---|
| **Domain Events** | همه به صورت خام منتشر می‌شدند | محدود و تبدیل‌شده به Integration Event |
| **Projection** | در Consumer (تکراری و پیچیده) | در Producer (متمرکز و قابل مدیریت) |
| **Internal Model** | در معرض دید و وابستگی Consumer | کاملاً مخفی و محافظت‌شده |
| **Functional Coupling** | زیاد (تکرار منطق تجاری) | کمتر (منطق در Producer متمرکز است) |
| **Implementation Coupling** | زیاد (وابستگی به Schema داخلی) | کمتر (وابستگی به Published Language پایدار) |
| **Temporal Coupling** | وجود داشت (انتظار برای پردازش) | کاهش یافته (با استفاده از Notification) |
| **Integration Contract** | مبهم و شکننده | مشخص و مبتنی بر Consumer-Driven Contract |

---

## 24. چه زمانی از کدام Event استفاده کنیم؟

یک راهنمای تصمیم‌گیری (Decision Guide) ساده:

```text
آیا Consumer فقط باید بداند اتفاقی افتاده و خودش داده را بگیرد؟
        │
        └── Event Notification

آیا Consumer باید یک Local State / Cache از داده‌ها داشته باشد؟
        │
        └── Event-Carried State Transfer (ECST)

آیا می‌خواهیم یک اتفاق مهم تجاری را در داخل سرویس اعلام کنیم؟
        │
        └── Domain Event

آیا می‌خواهیم داده‌ای را از مرز یک Bounded Context به بیرون بفرستیم؟
        │
        └── Integration Event (که ممکن است از روی Domain Event ترجمه شود)
```
> **تذکر:** این‌ها همیشه جایگزین مستقیم یکدیگر نیستند. یک Integration Event ممکن است از نوع ECST باشد، یا یک Notification. انتخاب به نیاز Integration بستگی دارد.

---

## 25. اشتباهات رایج (Common Mistakes)

- **انتشار تمام Domain Events:** باعث نشت Internal Model و Implementation Coupling می‌شود.
- **استفاده از Domain Event به عنوان Public API:** Domain Event برای مصرف داخلی است، نه خارجی.
- **قرار دادن Internal Model در Event:** هر تغییر در دیتابیس Producer، سیستم‌های دیگر را می‌شکند.
- **Duplicate Projection Logic:** تکرار منطق تجاری در چندین Consumer (Functional Coupling).
- **Subscribe کردن همه Consumerها به همه Eventها:** ایجاد ترافیک غیرضروری و پیچیدگی.
- **استفاده از Event به جای Command:** اگر می‌خواهید کاری انجام شود، Command بفرستید. Event برای اعلام وقوع است.
- **قرار دادن Data بیش از حد در Event:** افزایش حجم پیام و نشت اطلاعات.
- **قرار دادن Data بسیار کم در Event:** مجبور کردن Consumer به Fetchهای متعدد و ناکارآمد.
- **ایجاد Temporal Coupling:** وابسته کردن اجرای یک سرویس به زمان‌بندی سرویس دیگر.
- **نادیده گرفتن Versioning:** تغییر Schema بدون پشتیبانی از نسخه‌های قدیمی.
- **نادیده گرفتن Idempotency:** پردازش تکراری یک Event که منجر به فساد داده می‌شود.
- **نادیده گرفتن Eventual Consistency:** انتظار داشتن داده‌های کاملاً همگام بلافاصله پس از ارسال Event.
- **نادیده گرفتن Failure Handling:** فرض اینکه Message Broker همیشه پیام‌ها را تحویل می‌دهد.

---

## 26. Idempotency

در سیستم‌های توزیع‌شده، تحویل "حداقل یک‌بار" (At-Least-Once Delivery) رایج است. یعنی ممکن است یک Event دو بار به Consumer برسد.

مثال:
```text
PaymentCompleted (Event ID: 999)
PaymentCompleted (Event ID: 999) <-- تکراری
```
**راه‌حل:** Consumer باید Idempotent (تکرارپذیر بدون عوارض جانبی) باشد.
1. ذخیره `Event ID` یا `Correlation ID` در دیتابیس.
2. قبل از پردازش، بررسی کند که آیا این `Event ID` قبلاً پردازش شده است یا خیر.
3. اگر تکراری بود، آن را نادیده بگیرد (یا فقط Acknowledge کند) بدون اینکه عملیات تجاری (مثلاً کسر موجودی) دوباره انجام شود.

---

## 27. Eventual Consistency

در معماری Event-Driven، پس از اینکه Service A یک Event منتشر کرد، مدتی طول می‌کشد تا Service B آن را دریافت و پردازش کند. در این فاصله، State دو سیستم متفاوت است.

**مثال واقعی:** کاربر در سایت محصولی را خریداری می‌کند (`OrderCreated`). سرویس Inventory هنوز موجودی را کسر نکرده است. اگر کاربر در همین لحظه صفحه پروفایل خود را باز کند، ممکن است سفارش را نبیند. این یک سازگاری نهایی (Eventual Consistency) است و برای دستیابی به مقیاس‌پذیری و Decoupling، یک مصالحه (Trade-off) پذیرفته‌شده است .

---

## 28. Reliability

برای اطمینان از قابلیت اطمینان در سیستم‌های رویدادمحور، مفاهیم زیر حیاتی هستند:
- **Retry:** تلاش مجدد خودکار برای پردازش‌های شکست‌خورده (با الگوی Exponential Backoff).
- **Duplicate Messages:** سیستم باید (همانطور که در Idempotency گفته شد) در برابر پیام‌های تکراری مقاوم باشد.
- **Message Ordering:** در برخی موارد (مثل ECST)، ترتیب Eventها مهم است. Kafka این را در سطح Partition تضمین می‌کند، اما RabbitMQ خیر.
- **Dead Letter Queue (DLQ):** صفی برای نگهداری پیام‌هایی که پس از چندین بار تلاش، همچنان شکست می‌خورند (برای بررسی دستی).
- **Poison Message:** پیامی که به دلیل فرمت خراب، باعث Crash مداوم Consumer می‌شود و باید سریعاً به DLQ منتقل شود.
- **Event Loss:** باید با تأییدیه (Acknowledgment) مناسب در Message Broker جلوگیری شود.

---

## 29. Outbox Pattern

**مشکل:** Dual Write Problem.
```text
1. ذخیره سفارش در Database (موفق)
2. انتشار Event در Message Broker (شکست به دلیل قطعی شبکه)
```
نتیجه: داده در دیتابیس هست، اما سیستم‌های دیگر هرگز مطلع نمی‌شوند (Inconsistency).

**راه‌حل: Transactional Outbox Pattern** .
```text
Application
    │
    ├── 1. Update Business DB (مثلاً جدول Orders)
    │
    └── 2. Save Event in Outbox Table (در همان Transaction دیتابیس)
             │
             ▼ (تضمین شده توسط ACID دیتابیس)
          Outbox Table
             │
             ▼ (یک پردازش جداگانه مثل Debezium یا Background Worker)
        Event Publisher
             │
             ▼
        Message Broker
```
با این الگو، اگر Transaction دیتابیس موفق شود، Event قطعاً در جدول Outbox ذخیره می‌شود. سپس یک فرآیند جداگانه (Relay) آن را به Broker می‌فرستد. اگر ارسال شکست بخورد، دوباره تلاش می‌کند، بدون اینکه داده تجاری از دست برود.

---

## 30. Versioning

وقتی Integration Event تغییر می‌کند، نباید Consumerهای قدیمی خراب شوند.
- **Backward Compatibility:** اضافه کردن فیلدهای جدید (Optional) مجاز است. حذف فیلدهای موجود یا تغییر نام آن‌ها ممنوع است.
- **Schema Evolution:** استفاده از فرمت‌هایی مثل Protobuf یا Avro که Schema Evolution را بهتر مدیریت می‌کنند، یا حداقل مستندسازی دقیق تغییرات JSON.
- **Event Versioning:** اگر تغییر Breaking است، یک Event جدید با نسخه جدید (مثلاً `CustomerUpdatedV2`) ایجاد کنید و برای مدتی هر دو را منتشر کنید تا Consumerها مهاجرت کنند.

---

## 31. Testing

تست سیستم‌های Event-Driven پیچیده‌تر از سیستم‌های همگام است:
- **Unit Testing:** تست منطق Domain Event Handlerها به صورت ایزوله.
- **Integration Testing:** تست ارتباط واقعی با Message Broker (مثلاً با استفاده از Testcontainers برای RabbitMQ/Kafka).
- **Contract Testing:** استفاده از ابزارهایی مثل **Pact** برای اطمینان از اینکه Producer و Consumer بر سر ساختار پیام توافق دارند، بدون نیاز به اجرای کامل هر دو سرویس .
- **Event Handler Testing:** تست اینکه آیا Consumer در برابر پیام‌های تکراری (Idempotency) و پیام‌های خراب (Poison) به درستی رفتار می‌کند.
- **End-to-End Testing:** تست جریان کامل تجاری در محیط شبیه‌سازی‌شده (Staging).

---

## 32. Observability

در سیستم‌های ناهمگام، ردیابی یک درخواست (Trace) دشوار است.
- **Correlation ID:** یک شناسه یکتا که با درخواست اولیه ایجاد شده و در **تمام** Eventهای بعدی کپی می‌شود تا تمام مراحل یک فرآیند تجاری به هم لینک شوند.
- **Distributed Tracing:** استفاده از ابزارهایی مثل OpenTelemetry، Jaeger یا Application Insights برای مشاهده مسیر حرکت Event بین سرویس‌ها .
- **Logging:** ثبت ورودی و خروجی Event Handlerها همراه با Correlation ID.
- **Metrics:** مانیتورینگ طول صف (Queue Length)، نرخ پردازش (Throughput)، و نرخ خطا (Error Rate).

---

## 33. Production Considerations

هنگام استقرار در محیط واقعی:
- **Reliability:** استفاده از Outbox Pattern و DLQ.
- **Scalability:** اطمینان از اینکه Consumerها می‌توانند به صورت افقی (Horizontal Scaling) مقیاس شوند (مثلاً با استفاده از Consumer Groups در Kafka).
- **Security:** رمزنگاری پیام‌ها در حالت انتقال (TLS) و در حالت استراحت (Encryption at Rest)، و احراز هویت Producer/Consumer.
- **Message Retention:** تنظیم سیاست نگهداری پیام‌ها در Broker (مثلاً 7 روز در Kafka) برای امکان Replay در صورت خرابی Consumer.

---

## 34. مثال کامل از یک سیستم واقعی (E-Commerce)

سناریو: پردازش یک سفارش جدید.

1. **OrderCreated**
   - **Producer:** Order Service
   - **Consumer:** Payment Service
   - **نوع:** Integration Event (Notification)
   - **داده:** `orderId`, `amount`, `customerId`
   - **Failure:** اگر Payment Service دان باشد، پیام در صف می‌ماند (با Outbox تضمین شده است).

2. **PaymentCompleted**
   - **Producer:** Payment Service
   - **Consumer:** Inventory Service, Notification Service
   - **نوع:** Integration Event (ECST/Notification)
   - **داده:** `orderId`, `paymentId`, `status: "Success"`
   - **Coupling:** پایین. Inventory و Notification مستقل از هم عمل می‌کنند.

3. **InventoryReserved**
   - **Producer:** Inventory Service
   - **Consumer:** Shipping Service
   - **نوع:** Integration Event
   - **داده:** `orderId`, `items: [...]`
   - **Idempotency:** اگر این Event دو بار بیاید، Inventory Service باید با بررسی `orderId` از رزرو تکراری جلوگیری کند.

4. **ShipmentCreated**
   - **Producer:** Shipping Service
   - **Consumer:** Notification Service
   - **نوع:** Integration Event
   - **داده:** `orderId`, `trackingCode`

---

## 35. تحلیل عمیق مثال CRM

فرض کنید سیستمی شامل `CRM`، `Marketing`، `AdsOptimization` و `Reporting` داریم.

### معماری بد (تحلیل Coupling)
- **Functional Coupling:** هم Marketing و هم AdsOptimization برای فهمیدن "وضعیت فعلی مشتری"، باید Eventهای `NameChanged`، `AddressChanged` و `Merged` را بگیرند و خودشان منطق Merge کردن را بنویسند.
- **Implementation Coupling:** اگر CRM فیلد `Address` را به `Street` و `City` بشکند، هر دو سرویس Consumer می‌شکنند.
- **Temporal Coupling:** Reporting می‌خواهد گزارش تبلیغات را بزند، اما باید صبر کند تا AdsOptimization Eventهای CRM را پردازش کند. اگر AdsOptimization عقب بیفتد، Reporting داده‌های ناقص می‌دهد.

### معماری Refactored (بهتر)
1. **CRM** یک **Integration Projection** می‌سازد (مثلاً یک نمای Materialized View در دیتابیس خود به نام `CustomerIntegrationView`).
2. هرگاه تغییری در CRM رخ دهد، این View به‌روز می‌شود.
3. CRM یک Event از نوع **ECST** منتشر می‌کند: `CustomerIntegrationViewUpdated` که حاوی کل State پایدار (Published Language) است.
4. **Marketing** و **AdsOptimization** فقط این Event را می‌گیرند و کش محلی خود را با آن جایگزین (Replace) می‌کنند (بدون هیچ منطق پیچیده‌ای).
5. **AdsOptimization** پس از اتمام محاسبات، یک `CalculationCompleted` (Notification) به **Reporting** می‌فرستد.
6. **Reporting** پس از دریافت Notification، یک API Call به AdsOptimization می‌زند و داده‌های دقیق را Fetch می‌کند.

---

## 36. سوالات مفهومی مهم (FAQ)

**آیا هر Domain Event باید Integration Event شود؟**
خیر. اکثر Domain Eventها فقط برای مصرف داخلی سرویس هستند. فقط آن‌هایی که برای Bounded Contextهای دیگر معنی دارند باید (پس از ترجمه) به Integration Event تبدیل شوند.

**آیا Domain Event همان Integration Event است؟**
خیر. Domain Event مربوط به یک Bounded Context خاص است. Integration Event یک قرارداد عمومی برای ارتباط بین سرویس‌هاست .

**آیا Event باید تمام State را داشته باشد؟**
بستگی دارد. برای ECST بله (یا حداقل تغییرات Delta). برای Notification خیر.

**چه زمانی Event Notification بهتر از ECST است؟**
وقتی Consumer به‌ندرت به داده‌ها نیاز دارد، یا حجم داده‌ها بسیار زیاد است، یا امنیت داده‌ها ایجاب می‌کند که Consumer هر بار برای دریافت آن‌ها احراز هویت (Fetch) کند.

**چرا نباید تمام Domain Eventها را منتشر کنیم؟**
باعث نشت Internal Model، افزایش ترافیک شبکه، و ایجاد Implementation Coupling شدید می‌شود.

**Functional Coupling چیست؟**
وقتی چندین سرویس، منطق تجاری یکسانی را برای تفسیر داده‌ها پیاده‌سازی کنند.

**Implementation Coupling چیست؟**
وقتی Consumerها به ساختار داخلی داده‌های Producer وابسته باشند.

**Temporal Coupling چیست؟**
وقتی اجرای صحیح یک سرویس، وابسته به زمان‌بندی یا اتمام کار سرویس دیگری باشد.

**Published Language چیست؟**
یک مدل داده‌ای پایدار و مستندشده که به عنوان قرارداد ارتباطی بین سرویس‌ها استفاده می‌شود .

**Consumer-Driven Contract چیست؟**
روشی که در آن Consumer نیازهای خود را تعریف می‌کند و Producer متعهد به تأمین آن ساختار داده‌ای می‌شود.

**چرا Internal Domain Model نباید Public Contract باشد؟**
چون Internal Model برای انعطاف‌پذیری و Refactoring طراحی شده است، در حالی که Public Contract نیاز به ثبات (Stability) دارد.

**آیا Event-Driven همیشه به معنی Loose Coupling است؟**
خیر. اگر Eventها بد طراحی شوند (مثل انتشار Internal Model)، می‌تواند منجر به Coupling پنهان و خطرناک‌تر شود.

**آیا استفاده از Kafka یا RabbitMQ به‌تنهایی سیستم را Decoupled می‌کند؟**
خیر. Message Broker فقط یک ابزار انتقال (Transport) است. Decoupling واقعی در **طراحی Eventها** و **مرزهای سرویس‌ها** (Boundaries) اتفاق می‌افتد.

---

## 37. Cheat Sheet

```text
Domain Event
→ Business Event (داخلی، برای تغییرات وضعیت در یک سرویس)

Event Notification
→ "Something happened" (اطلاع‌رسانی ساده، Consumer باید Fetch کند)

ECST (Event-Carried State Transfer)
→ "Here is the state/change" (همراه با داده برای به‌روزرسانی Local Cache)

Integration Event
→ Communication across boundaries (قرارداد پایدار بین سرویس‌ها)

Functional Coupling
→ Same business logic in multiple places (تکرار منطق تفسیر داده)

Implementation Coupling
→ Consumer depends on producer implementation (وابستگی به Schema داخلی)

Temporal Coupling
→ Consumer depends on producer timing (وابستگی به زمان‌بندی اجرا)

Published Language
→ Stable integration model (زبان مشترک و پایدار بین سرویس‌ها)

Consumer-Driven Contract
→ Contract based on consumer needs (تولید داده بر اساس نیاز مصرف‌کننده)
```

---

## 38. منابع

### منابع رسمی
- [Domain events: Design and implementation - Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)
- [Domain Events vs. Integration Events - .NET Architecture Blog](https://devblogs.microsoft.com/cesardelatorre/domain-events-vs-integration-events-in-domain-driven-design-and-microservices-architectures/)
- [Implement the Transactional Outbox Pattern - Microsoft Azure Architecture](https://learn.microsoft.com/en-us/azure/architecture/databases/guide/transactional-out-box-cosmos)
- [Transactional outbox pattern - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)

### منابع معماری
- [What do you mean by “Event-Driven”? - Martin Fowler](https://martinfowler.com/articles/201701-event-driven.html)
- [The Many Meanings of Event-Driven Architecture - Martin Fowler](https://martinfowler.com/articles/201701-event-driven.html) (بخش Event Notification و ECST)
- [Bounded Context - Martin Fowler](https://martinfowler.com/bliki/BoundedContext.html)

### منابع تکمیلی
- [Implementing Domain-Driven Design - Vaughn Vernon](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) (برای مفاهیم Published Language و CDC)
- [Building Microservices - Sam Newman](https://www.oreilly.com/library/view/building-microservices-2nd/9781492034018/) (برای مفاهیم Integration و Coupling)

---
*این مستند با رعایت اصول معماری نرم‌افزار و با استناد به منابع معتبر تا سال 2026 تهیه شده است. برای انتشار در GitHub، کافی است این متن را در یک فایل `README.md` یا `docs/event-driven-architecture.md` کپی کنید.*