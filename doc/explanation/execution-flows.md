# جریان‌های اجرا و عمر نشست

تاریخ بررسی: 2026-10-28 | نسخه: 0.10.1
این متن از کد استخراج شده است؛ هیچ نشست، فرمان Git یا درخواست شبکه برای تأیید آن اجرا نشده است.

## سه مفهوم متفاوت

نشست دوردست، رشتهٔ پیام‌های نگهداری‌شده در DeepSeek است.
رونوشت محلی، پیام‌های قابل نمایش برای ادامهٔ رابط تعاملی است.
نسل جاری، اجرای قابل لغوی است که ممکن است چند درخواست تولید و چند دور ابزار داشته باشد.
مالک دو مفهوم آخر به‌ترتیب [SessionTranscript.ts](../../source/core/SessionTranscript.ts) و `beginGeneration` در [apiClient.ts](../../source/core/apiClient.ts) است.
تغییر یکی از این‌ها لزوماً دو مورد دیگر را حذف نمی‌کند.

## آغاز تازه

1. [cli.tsx](../../source/cli.tsx) تنظیمات و آرگومان‌ها را می‌خواند و مسیر رابط را برمی‌گزیند.
2. `App` در [app.tsx](../../source/app.tsx) نبود توکن را به `TokenSetupView` هدایت می‌کند.
3. [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx) ترجیح مدل را بارگذاری و وضعیت API را برای گفتگوی تازه بازنشانی می‌کند.
4. `ensureSession` و `startSession` در [useChatSession.ts](../../source/components/chat/useChatSession.ts) prompt سیستم را می‌فرستند.
5. `sendMessage` نشست را پیدا می‌کند یا می‌سازد و پیام را به والد مناسب متصل می‌کند.

پاسخ مقداردهی اولیه، پاسخ معمول به کاربر نمایش داده نمی‌شود.
ارسال ورودی کاربر منتظر پایان مقداردهی اولیه می‌ماند؛ صرف نمایش صفحه به معنای آماده‌بودن شبکه نیست.
شمارندهٔ `sessionGeneration` در hook، نتیجهٔ مقداردهی قدیمی را از نشست جدید جدا می‌کند.
اگر نتیجهٔ قدیمی پس از تعویض برسد، برای نشست آن درخواست حذف فرستاده می‌شود.

## ادامهٔ گفتگو

در `resume <session-id>`، مسیر رابط شناسهٔ درخواستی را به hook می‌رساند.
`setCurrentSessionId` در [apiClient.ts](../../source/core/apiClient.ts) پیام والد را ناشناخته می‌کند تا در ارسال بعد کشف شود.
رونوشت محلی در صورت وجود بارگذاری می‌شود؛ prompt سیستم دوباره ارسال نمی‌شود.
در ارسال بعدی، `chatSessions` شناسهٔ پیام جاری نشست را تأمین می‌کند.
اگر نشست در فهرست پیدا نشود، `sendMessage` می‌تواند نشست تازه بسازد؛ resume تضمین شکست صریح برای شناسهٔ ناموجود نیست.
نبود رونوشت محلی نیز با نبود تاریخچهٔ دوردست یکسان نیست.
قالب ذخیره و شناسه‌ها در [پیکربندی و ذخیره‌سازی](../reference/configuration-and-storage.md) توضیح داده می‌شود.

## یک نوبت و حلقهٔ ابزار

`getAIResponse` در [agent.ts](../../source/actions/agent.ts) صاحب این چرخه است:

```text
prompt -> sendMessage -> parse tool calls -> executeToolCalls
                       ^                         |
                       +------ tool_result ------+
```

ابتدا متن پاسخ برای فراخوانی ابزار بررسی می‌شود؛ فقط در نبود فراخوانی پاسخ، متن thinking نیز بررسی می‌شود.
`parseToolCalls` در [parse.ts](../../source/tools/parse.ts) متن را به فراخوانی‌های محلی تبدیل می‌کند.
خطای parse یک دور مصرف می‌کند و پیام اصلاح به مدل فرستاده می‌شود.
نتیجهٔ اجرا، حتی نتیجهٔ خطا، داخل بلوک `tool_result` به گفتگو برمی‌گردد.
حداکثر 10 دور ابزار مجاز است؛ در مرز سقف، عامل خلاصه‌ای بدون ابزار درخواست می‌کند.
این عدد، سقف ادامهٔ SSE یا شمار پیام‌های کل نشست نیست.
نمایش جریان با `createLiveStream` در [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) دسته‌بندی می‌شود و markup ابزار برای کاربر پنهان می‌ماند.

## حالت و تصمیم کاربر

`useComposerKeys` در [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) با TAB میان `normal`، `yolo` و `plan` حرکت می‌کند.
`executeToolCalls` در [execute.ts](../../source/tools/execute.ts) حالت را هنگام اجرای ابزار اعمال می‌کند.
در normal، ابزار دارای `requiresConfirmation` تصمیم کاربر می‌خواهد؛ نبود callback به معنی پذیرش نیست.
در yolo، تأییدهای معمول کنار گذاشته می‌شوند، ولی تشخیص elevation همچنان تأیید می‌خواهد.
در plan، عملیات دسته‌بندی‌شده به‌عنوان تغییر مسدود می‌شوند؛ افزودن و تغییر بعضی todoهای حافظه‌ای همچنان ممکن است.
`PlanPrompt` در [prompts/index.ts](../../source/prompts/index.ts) خواستهٔ رفتاری تکمیلی است، نه عامل اجرای این ممنوعیت.
`PlanConfirmation` در [PlanConfirmation.tsx](../../source/components/chat/PlanConfirmation.tsx) پس از طرح، Enter یا `y` را برای اجرا و Esc یا `n` را برای ماندن در plan می‌گیرد.
پذیرش طرح، حالت غیر-plan قبلی را بازمی‌گرداند و `ExecutePlanPrompt` را ارسال می‌کند؛ طرح یک تراکنش قابل rollback نیست.

## لغو و پاک‌سازی

هنگام بارگذاری، Ctrl+C از مسیر `stopCurrentGeneration` لغو محلی و در صورت داشتن شناسه، درخواست توقف دوردست را آغاز می‌کند.
کشف نشست، دریافت چالش و حل همگام PoW همان پوشش AbortSignal درخواست completion را ندارند.
لغو پاسخ نمی‌تواند تغییر فایل یا فرمانی را که قبلاً اجرا شده بازگرداند.
خارج از بارگذاری، Ctrl+C ابتدا ورودی موجود را پاک می‌کند و سپس مسیر خروج مطرح می‌شود.
`deleteUnusedSession` در [useChatSession.ts](../../source/components/chat/useChatSession.ts) برای نشست تازهٔ بدون پیام کاربر پاک‌سازی best-effort انجام می‌دهد.
نشست استفاده‌شده نگه داشته می‌شود و رابط می‌تواند فرمان resume را هنگام خروج نشان دهد.
`resetConversation` رونوشت قبلی را حذف و وضعیت API را بازنشانی می‌کند؛ `/clear` الزاماً نشست استفاده‌شدهٔ دوردست را حذف نمی‌کند.
این فرمان todoها را پاک نمی‌کند و تاریخچهٔ نمایش قبلی را نیز صرفاً با یک نشست تازه جایگزین کامل نمی‌کند.

## تعویض مدل

`switchModel` در [useChatSession.ts](../../source/components/chat/useChatSession.ts) پیش از اولین پیام کاربر نیاز به ساخت نشست جایگزین ندارد.
پس از استفاده، خلاصه‌ای با مدل قبلی، بدون thinking و بدون ابزار می‌گیرد.
سپس وضعیت API را پاک می‌کند و prompt سیستم به‌همراه خلاصه را به نشست تازهٔ مدل مقصد می‌فرستد.
نشست استفاده‌شدهٔ قبلی حذف نمی‌شود؛ این انتقال خلاصه است، نه کپی دقیق تاریخچه و تمام زمینهٔ آن.
اگر خلاصه خالی باشد، prompt آغازین بدون خلاصه ساخته می‌شود.

## تک‌پرسش، commit و سرور

`SinglePromptView` در [SinglePromptView.tsx](../../source/components/SinglePromptView.tsx) ابتدا زمینهٔ سیستم و سپس پرسش را می‌فرستد و ابزارهای normal را مدیریت می‌کند.
در پایان، درخواست حذف نشست را بدون انتظار برای نتیجه آغاز می‌کند و خارج می‌شود؛ quiet فقط نمایش thinking را خاموش می‌کند.
[commitMessage.ts](../../source/actions/commitMessage.ts) از diff مرحله‌بندی‌شده یا `git diff HEAD` پیام می‌سازد و پیش از بازگرداندن متن، منتظر درخواست حذف می‌ماند.
`CommitView` در [CommitView.tsx](../../source/components/CommitView.tsx) تنها پس از پذیرش کاربر `git commit -F -` را اجرا می‌کند.
گزینهٔ `-a` ورودی تولید پیام را عوض می‌کند؛ به‌خودی‌خود stage نمی‌کند و `git commit -a` نمی‌سازد.
در سرور، `enqueueCompletion` و `runCompletion` در [session.ts](../../source/server/session.ts) یک صف و وضعیت نشست مشترک دارند.
[request.ts](../../source/server/request.ts) فقط متن آخرین پیام مشتری را ورودی می‌کند، نه بازپخش کامل آرایهٔ گفتگو.
ابزار مشتری به‌صورت پاسخ برگردانده می‌شود؛ این مسیر از اجرای محلی ابزار جدا است.
قطع اتصال خروجی را خاموش می‌کند، اما درخواست توقف upstream بعد از بازگشت completion تلاش می‌شود.

## مسیرهای مرتبط

[معماری](architecture.md)، [انتقال DeepSeek](deepseek-transport.md) و [امنیت](security-and-privacy.md) چرایی این مرزها را توضیح می‌دهند.
جزئیات در [کلیدها و فرمان‌ها](../reference/keyboard-and-slash-commands.md)، [پروتکل ابزار](../reference/tool-protocol.md) و [HTTP API](../reference/http-api.md) است.
[محدودیت‌های شناخته‌شده](../reference/known-limitations.md) و [نمایهٔ مستندات](../README.md) نقطهٔ پیگیری موارد تأییدنشده‌اند.