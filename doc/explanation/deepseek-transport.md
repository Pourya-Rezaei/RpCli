# انتقال DeepSeek، جریان و ادامهٔ پاسخ

تاریخ بررسی: 2026-10-28 | نسخه: 0.10.1
شواهد این صفحه از خواندن منبع است؛ ارتباط زنده، اعتبار توکن و سازگاری فعلی سرویس آزموده نشده‌اند.

## سرویس مورد استفاده

`chat` در [Chat.ts](../../core-lib/Chat.ts) به `https://chat.deepseek.com/api/v0/chat/completion` درخواست می‌فرستد.
این مسیر بخشی از جریان نشست وب است؛ نباید آن را همان سرویس رسمی API با کلید API معرفی کرد.
اعتبارنامه در header با طرح Bearer ارسال می‌شود و نوع مورد انتظار، توکن نشست وب است.
`TokenSetupView` در [TokenSetupView.tsx](../../source/components/TokenSetupView.tsx) مسیر دریافت همین توکن را ارائه می‌کند.
ارسال توکن در این لایه معادل اعتبارسنجی جامع همهٔ پاسخ‌های upstream نیست.
headerهای نسخه و پلتفرم در کد ثابت‌اند؛ مستندات تضمینی برای پایداری قرارداد خصوصی سرویس نمی‌دهند.

## دو سطح هماهنگی

`sendMessage` در [apiClient.ts](../../source/core/apiClient.ts) تصمیم می‌گیرد پیام به کدام نشست و والد وصل شود.
`chat` در [Chat.ts](../../core-lib/Chat.ts) درخواست completion و خواندن بدنهٔ پاسخ را کنترل می‌کند.
اولی همچنین PoW و نرمال‌سازی markup ابزار را انجام می‌دهد؛ دومی نوع‌های `ChatResult` و `ChatStreamChunk` را تعریف می‌کند.
این تفکیک توضیح می‌دهد چرا خطای شبکه، خطای تجزیهٔ ابزار و خطای اجرای ابزار هم‌معنی نیستند.

## کشف والد و ساخت نشست

وقتی `parentMessageId` ناشناخته است، `sendMessage` از `chatSessions` فهرست نشست‌ها را می‌گیرد.
مالک این فراخوانی [ChatSessions.ts](../../core-lib/ChatSessions.ts) و نوع توصیف نشست `ChatSession` است.
اگر شناسهٔ فعلی پیدا شود، `current_message_id` به‌عنوان والد انتخاب می‌شود.
اگر پیدا نشود، `createSessions` در [CreateSessions.ts](../../core-lib/CreateSessions.ts) نشست تازه می‌سازد.
پس شناسهٔ ورودی، قفل قطعی روی یک نشست موجود نیست.
`setCurrentSessionId` و `resetChatSession` تنها وضعیت محلی ماژول را تغییر می‌دهند، نه تاریخچهٔ دوردست را.
پس از پاسخ، شناسهٔ پیام جدید برای ارسال بعدی نگه داشته می‌شود.

## چالش اثبات کار

`createPowChallenge` در [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) چالش مورد نیاز را دریافت می‌کند.
در `sendMessage`، نمونهٔ `PowSolver` مقداردهی اولیه می‌شود و `solve` payload پاسخ را می‌سازد.
payload با `JSON.stringify` و `btoa` کدگذاری می‌شود و در header به نام `x-ds-pow-response` قرار می‌گیرد.
این حل محلی بخشی از پیش‌نیاز درخواست است، نه تولید متن مدل.
حل همگام با AbortSignal completion متوقف نمی‌شود؛ سرعت و هزینهٔ آن در این بررسی اندازه‌گیری نشده است.
نباید از وجود PoW نتیجه گرفت که درخواست‌ها محدودیت نرخ محلی یا تضمین مقابله با سوءاستفاده دارند.

```text
session lookup/create -> challenge -> solver -> completion SSE
                                              -> continuation when INCOMPLETE
```

## خواندن SSE

`chat` ابتدا نوع محتوای پاسخ را بررسی می‌کند و انتظار `text/event-stream` دارد.
پاسخ غیرجریانی به JSON خوانده می‌شود؛ خطای تجاری می‌تواند `bizCode` و متن خطا به نتیجه بدهد.
نبود بدنه نیز نتیجهٔ ناموفق مستقل می‌سازد.
در مسیر جریانی، `TextDecoder` و buffer خطوط، قطعه‌های بدنه را برای پردازش رویدادها کنار هم می‌گذارند.
پیاده‌سازی هم شکل محتوای تخت و هم fragmentها و patchهای upstream را می‌شناسد.
`ChatStreamChunk` کانال‌های `thinking` و `response` را با `messageId` از هم جدا می‌کند.
`ChatResult` علاوه بر متن، وضعیت، اطلاعات مدل و usage دریافتی را گردآوری می‌کند.
وجود فیلد usage در نوع به معنی تضمین حضور آن در همهٔ پاسخ‌ها نیست.
قراردادهای upstream در این کد به‌صورت عمومی و کامل اعتبارسنجی ساختاری نمی‌شوند.

## نرمال‌سازی و نمایش

`createToolCallMarkupNormalizer` و `normalizeToolCallMarkup` در [toolCallMarkup.ts](../../source/core/toolCallMarkup.ts) میان DSML و قالب ابزار محلی پل می‌زنند.
`sendMessage` برای response و thinking نرمال‌ساز جدا نگه می‌دارد و باقی‌ماندهٔ buffer را در پایان تخلیه می‌کند.
این نرمال‌سازی با parse و اعتبارسنجی آرگومان‌ها در حلقهٔ عامل یکی نیست.
به‌روزرسانی‌های ترمینال بعداً در `createLiveStream` دسته‌بندی می‌شوند؛ مرز قطعهٔ شبکه الزاماً مرز یک پیام رابط نیست.
شرح متن پروتکل در [مرجع پروتکل ابزار](../reference/tool-protocol.md) است.

## سه سقف مستقل

| سقف | مالک | معنای دقیق |
| --- | --- | --- |
| 10 ادامه | `MAX_CONTINUATIONS` در [Chat.ts](../../core-lib/Chat.ts) | ادامهٔ پاسخ با وضعیت `INCOMPLETE` در یک فراخوانی chat. |
| 2 تلاش اضافی | `MAX_EMPTY_RESPONSE_RETRIES` در [apiClient.ts](../../source/core/apiClient.ts) | درخواست پاسخ نهایی وقتی فقط thinking غیرخالی رسیده است. |
| 10 دور ابزار | `getAIResponse` در [agent.ts](../../source/actions/agent.ts) | چرخهٔ مدل، ابزار و نتیجه؛ جدا از ادامهٔ انتقال. |

`continueChat` در [ContinueChat.ts](../../core-lib/ContinueChat.ts) درخواست ادامهٔ پیام را می‌فرستد.
ادامهٔ پاسخ ناقص با درخواست مجدد کاربر برای ادامهٔ یک کار یا فرمان `/continue` یک مفهوم نیست.
در تلاش مخصوص پاسخ فقط-thinking، `modelType` دوباره به `sendMessage` پاس داده نمی‌شود.
بنابراین در همین شاخه، حفظ انتخاب Expert تضمین نشده و مقدار پیش‌فرض می‌تواند اعمال شود.
هیچ‌کدام از این سقف‌ها deadline عمومی برای کل کار نیستند.

## خطا و لغو

`assertValidTokenResponse` در [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) پاسخ 401 و پیام‌های واجد شرایط خطای API را بررسی می‌کند.
پاسخ 403 به‌تنهایی در این تشخیص، مدرک کافی برای توکن نامعتبر نیست.
`isInvalidTokenError` امکان واکنش متفاوت رابط به خطای اعتبارنامه را فراهم می‌کند.
`sendMessage` نتیجهٔ `ok: false` را به خطای قابل پرتاب تبدیل می‌کند؛ این با توقف عمدی فرق دارد.
`chat` در مسیر AbortError نتیجهٔ متوقف‌شده با `stopped` می‌دهد.
`stopCurrentGeneration` ابتدا controller را abort می‌کند و اگر شناسهٔ پیام موجود باشد `stopStream` را فرا می‌خواند.
مالک درخواست توقف [StopStream.ts](../../core-lib/StopStream.ts) است.
فراخوانی‌های completion و continuation سیگنال لغو دارند؛ کشف نشست و دریافت چالش همان پوشش را ندارند.
در این لایه timeout عمومی برای درخواست‌ها تعریف نشده است.

## حذف و محدودیت مدل

`deleteSession` در [DeleteSession.ts](../../core-lib/DeleteSession.ts) پاسخ fetch را برمی‌گرداند و موفقیت HTTP حذف را الزاماً assert نمی‌کند.
`stopStream` نیز پاسخ fetch را بدون تضمین پذیرش توقف برمی‌گرداند؛ resolve شدن Promise اثبات نتیجهٔ دوردست نیست.
در `chat`، جستجو برای مدل غیر-default صریحاً رد می‌شود و thinking همراه vision نیز رد می‌شود.
نوع پایین‌دستی vision را می‌شناسد، اما این امر به معنی وجود گردش‌کار تصویری در CLI یا HTTP نیست.
`ref_file_ids` در درخواست completion خالی است؛ اشارهٔ متنی به مسیر فایل، بارگذاری فایل در DeepSeek نیست.

## ارتباط با HTTP محلی

[stream.ts](../../source/server/stream.ts) SSE مشتری HTTP را تولید می‌کند؛ این جریان با SSE دریافتی DeepSeek یک اتصال یکسان نیست.
نام‌هایی مثل `deepseek-reasoner` در [request.ts](../../source/server/request.ts) می‌توانند پیش‌فرض thinking را تغییر دهند، نه لزوماً مدل Expert upstream را.
در حالت ابزار مشتری، خروجی برای parse نهایی buffer می‌شود و جریان تدریجی معمول را ندارد.
برای جزئیات [مرجع HTTP](../reference/http-api.md)، [نمادها](../reference/exported-symbols.md) و [محدودیت‌ها](../reference/known-limitations.md) را ببینید.
برای جایگاه انتقال در کل برنامه به [معماری](architecture.md)، [جریان‌ها](execution-flows.md) و [نمایه](../README.md) مراجعه کنید.