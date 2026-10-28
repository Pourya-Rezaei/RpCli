# معماری و مالکیت ماژول‌ها

تاریخ بررسی: 2026-10-28 | نسخه: 0.10.1
مبنای این توضیح خواندن کد است؛ رفتار برنامه در زمان اجرا آزموده نشده است.

## مدل ذهنی

RpCli یک رابط ترمینالی بر فراز نشست وب DeepSeek است، نه یک موتور مدل محلی.
هماهنگ‌کنندهٔ عامل، خروجی متنی مدل را به درخواست ابزار تبدیل می‌کند و نتیجه را دوباره به همان گفتگو می‌فرستد.
رابط HTTP نیز از همین هماهنگ‌کننده استفاده می‌کند، اما ابزارهای اعلام‌شدهٔ مشتری را محلی اجرا نمی‌کند.
پس «ابزار مدل»، «عملیات فایل محلی» و «فراخوانی تابع مشتری HTTP» یک مرز اجرایی مشترک ندارند.

```text
cli -> Ink / app -> views -> getAIResponse -> sendMessage -> core-lib -> DeepSeek
cli -> startServer -> request / session / stream -> getAIResponse
getAIResponse -> parseToolCalls -> executeToolCalls -> local tools
```

## ورود و انتخاب رابط

[source/cli.tsx](../../source/cli.tsx) محل بارگذاری تنظیمات و تفسیر آرگومان‌ها با Meow است.
شاخهٔ `serve` پیش از ورود به Ink انتخاب می‌شود؛ سرور را نباید زیرمجموعهٔ مسیریابی `App` فرض کرد.
در شاخهٔ ترمینال، گزینهٔ commit، ورودی تک‌پرسشی و گفتگوی تعاملی مسیرهای جدا دارند.
[source/app.tsx](../../source/app.tsx)، با مؤلفهٔ `App`، دسترسی به نماها را به وجود `DEEPSEEK_TOKEN` وابسته می‌کند.
توکن نامعتبر نیز از همین لایه به جریان تنظیم مجدد اعتبارنامه برمی‌گردد.
جزئیات نحو فرمان‌ها در [مرجع CLI](../reference/cli.md) قرار دارد.

## هماهنگ‌کننده و حالت مشترک

`getAIResponse` در [agent.ts](../../source/actions/agent.ts) صاحب حلقهٔ تولید، تجزیهٔ ابزار و بازخوراندن نتیجه است.
`getChatSystemPrompt` متن آغاز گفتگو را از promptها و راهنمای پروژه می‌سازد.
`ChatMode` و `AIResponseOptions` قرارداد میان نماها و عامل را مشخص می‌کنند.
این لایه با محدودیت دور ابزار، پایان‌دادن به چرخه را کنترل می‌کند؛ مسئول تفسیر SSE نیست.
`sendMessage` در [apiClient.ts](../../source/core/apiClient.ts) نشست، پیام والد و PoW را مدیریت می‌کند.
`beginGeneration` و `stopCurrentGeneration` نیز در همین فایل مالک وضعیت لغو هستند.
شناسهٔ نشست و کنترل‌کنندهٔ لغو در سطح ماژول نگهداری می‌شوند، نه در یک شیء مستقل به‌ازای هر مشتری.
این انتخاب برای درک صف سرور و محدودیت جداسازی گفتگوها اهمیت دارد.

## مالکیت تمام فایل‌های انتقال

| فایل | نماد مالک و مسئولیت |
| --- | --- |
| [index.ts](../../core-lib/index.ts) | بازصادرکردن عملیات و نوع‌های انتقال؛ وجود export به معنای SDK پایدار نیست. |
| [Chat.ts](../../core-lib/Chat.ts) | `chat`، `ChatResult` و `ChatStreamChunk`؛ completion، خواندن SSE و ادامهٔ پاسخ ناقص. |
| [ContinueChat.ts](../../core-lib/ContinueChat.ts) | `continueChat`؛ درخواست ادامهٔ پیام موجود. |
| [ChatSessions.ts](../../core-lib/ChatSessions.ts) | `chatSessions` و `ChatSession`؛ بازیابی نشست‌ها و شناسهٔ پیام جاری. |
| [CreateSessions.ts](../../core-lib/CreateSessions.ts) | `createSessions`؛ ایجاد شناسهٔ نشست دوردست. |
| [DeleteSession.ts](../../core-lib/DeleteSession.ts) | `deleteSession`؛ ارسال درخواست حذف، نه تضمین تأیید حذف. |
| [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) | `createPowChallenge`؛ دریافت چالش برای حل محلی. |
| [StopStream.ts](../../core-lib/StopStream.ts) | `stopStream`؛ درخواست توقف پیام در سرویس دوردست. |
| [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) | `InvalidTokenError`، `assertValidTokenResponse` و `isInvalidTokenError`؛ تشخیص خطای اعتبارنامه. |

## نماها و مؤلفه‌های ترمینال

| مؤلفه | مرز مسئولیت |
| --- | --- |
| [InteractiveChatView](../../source/components/InteractiveChatView.tsx) | ورودی، تاریخچه، فرمان‌ها، مدل، حالت کار و اتصال جریان به نمایش. |
| [SinglePromptView](../../source/components/SinglePromptView.tsx) | یک پرسش، تأیید ابزارهای آن و خروج پس از پاسخ. |
| [CommitView](../../source/components/CommitView.tsx) | نمایش پیام پیشنهادی و تأیید پیش از اجرای Git. |
| [TokenSetupView](../../source/components/TokenSetupView.tsx) | دریافت و ذخیرهٔ توکن نشست وب با ورودی پوشیده. |
| [MarkdownText](../../source/components/MarkdownText.tsx) | تبدیل زیرمجموعه‌ای از Markdown به مؤلفه‌های Ink. |
| [FzfFilePicker](../../source/components/FzfFilePicker.tsx) | انتخاب مسیر با تطبیق fuzzy داخلی، نه اجرای برنامهٔ خارجی fzf. |
| [SlashCommandPicker](../../source/components/SlashCommandPicker.tsx) | پیشنهاد و انتخاب فرمان اسلش. |
| [ToolConfirmation](../../source/components/ToolConfirmation.tsx) | گرفتن تصمیم کاربر دربارهٔ اجرای ابزار. |
| [Spinner](../../source/components/Spinner.tsx) | نشانگر انتظار، نه منبع وضعیت شبکه. |
| [RpCliLogo](../../source/components/RpCliLogo.tsx) | نمایش هویت برنامه در تاریخچه. |
| [MessageRow](../../source/components/chat/MessageRow.tsx) | نمایش یک پیام با توجه به نقش و محتوای آن. |
| [ChatStatusBar](../../source/components/chat/ChatStatusBar.tsx) | نمایش وضعیت گفتگو و گزینه‌های فعال. |
| [ModelConfirmation](../../source/components/chat/ModelConfirmation.tsx) | تصمیم رابط برای انتخاب مدل. |
| [PlanConfirmation](../../source/components/chat/PlanConfirmation.tsx) | تصمیم اجرای طرح پس از پاسخ حالت plan. |

`useChatSession` در [useChatSession.ts](../../source/components/chat/useChatSession.ts) آغاز، تعویض مدل و پاک‌سازی نشست بلااستفاده را هماهنگ می‌کند.
`useComposerKeys` در [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) تصمیم‌های صفحه‌کلید را از نمای اصلی جدا می‌کند.
`createLiveStream` در [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) به‌روزرسانی نمایش را تقریباً هر 50 میلی‌ثانیه دسته‌بندی می‌کند.
نوع `ChatMessage` در [types.ts](../../source/components/chat/types.ts) قرارداد پیام‌های رابط است، نه قالب پاسخ DeepSeek.

## ابزار، prompt و دادهٔ پایدار

[registry.ts](../../source/tools/registry.ts) ابزارها را معرفی می‌کند و `executeToolCalls` در [execute.ts](../../source/tools/execute.ts) سیاست دستهٔ اجرا را اعمال می‌کند.
نوع‌های `Tool` و `ToolCall` در [types.ts](../../source/tools/types.ts) قرارداد ابزار محلی‌اند.
`SystemPrompt` و `readAgentsMarkdown` در [prompts/index.ts](../../source/prompts/index.ts) از فایل‌های prompt و فقط `AGENTS.md` ریشهٔ cwd استفاده می‌کنند.
متن prompt خواستهٔ رفتاری مدل است؛ جایگزین اعتبارسنجی آرگومان یا کنترل مسیر نیست.
[TokenConfig.ts](../../source/core/TokenConfig.ts)، [SessionTranscript.ts](../../source/core/SessionTranscript.ts) و [LogChat.ts](../../source/core/LogChat.ts) به‌ترتیب تنظیمات، رونوشت ادامهٔ گفتگو و ثبت اختیاری رویداد را نگه می‌دارند.
این سه نوع داده با تاریخچهٔ دوردست DeepSeek یکی نیستند.

## مرز سرور و ادامهٔ مطالعه

`startServer` در [server/index.ts](../../source/server/index.ts) مسیرها را ثبت می‌کند؛ `runCompletion` و `enqueueCompletion` در [session.ts](../../source/server/session.ts) اجرای مشترک را صف‌بندی می‌کنند.
[request.ts](../../source/server/request.ts) ورودی مشتری و [stream.ts](../../source/server/stream.ts) قالب پاسخ HTTP را تفسیر می‌کنند.
برای زنجیرهٔ رخدادها [جریان‌های اجرا](execution-flows.md)، برای انتقال [DeepSeek](deepseek-transport.md) و برای مرز اعتماد [امنیت و حریم خصوصی](security-and-privacy.md) را بخوانید.
قرارداد دقیق‌تر در [نمادهای صادرشده](../reference/exported-symbols.md)، [ابزارهای محلی](../reference/local-tools.md) و [promptها](../reference/prompts.md) است.
فهرست مسیرهای مطالعه در [نمایهٔ مستندات](../README.md) قرار می‌گیرد.