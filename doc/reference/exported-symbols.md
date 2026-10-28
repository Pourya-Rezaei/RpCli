# رابط‌ها و نمادهای مهم

این صفحه راهنمای پیدا کردن نقاط اتصال معماری است، نه فهرست همهٔ متغیرهای خصوصی. نسخه `0.10.1`، تاریخ `2026-10-28`؛ شواهد از کد استخراج شده‌اند.

## وضعیت API عمومی

[package.json](../../package.json) executable به نام `rc` و files برابر dist دارد؛ `exports` برای SDK پایدار تعریف نشده است. export شدن یک تابع در TypeScript به معنی تضمین سازگاری برای مصرف‌کنندهٔ بیرونی نیست. import مستقیم مسیرهای داخلی، قرارداد عمومی مستندشدهٔ پکیج محسوب نمی‌شود.

## عامل و API client

| ماژول | رابط | مسئولیت و نکته |
| --- | --- | --- |
| [agent.ts](../../source/actions/agent.ts) | `ChatMode` | normal، yolo، plan |
| همان ماژول | `AIResponseOptions` | token، prompt، callbackها، mode و گزینه‌های مدل/ابزار |
| همان ماژول | `getChatSystemPrompt` | آماده‌سازی prompt زمینهٔ عامل |
| همان ماژول | `getAIResponse` | نسل جدید، حلقهٔ ابزار و نتیجهٔ نهایی |
| [apiClient.ts](../../source/core/apiClient.ts) | `SendMessageOptions`, default `sendMessage` | ارسال بالادستی، PoW و پیشبرد parent |
| همان ماژول | `beginGeneration`, `isGenerationStopped`, `stopCurrentGeneration` | کنترل یک نسل و علامت توقف |
| همان ماژول | `resetChatSession`, `getCurrentSessionId`, `setCurrentSessionId` | state مشترک سطح module |
| همان ماژول | `chatSessionExists` | بررسی نشست در سرویس؛ متکی به شبکه |

state API client متعلق به یک instance گفتگو به‌صورت کلاس مستقل نیست. ایجاد چند caller در همان پردازش به‌تنهایی جداسازی ایجاد نمی‌کند. مدل‌های server، CLI و transcript را نباید یک نوع دادهٔ واحد فرض کرد.

## transport پایین‌دستی

نمایهٔ exportها: [core-lib/index.ts](../../core-lib/index.ts).

| export | ماژول مالک | مسئولیت |
| --- | --- | --- |
| `chat`, `ChatResult`, `ChatStreamChunk` | [Chat.ts](../../core-lib/Chat.ts) | completion، SSE، metadata و ادامهٔ پاسخ |
| `continueChat` | [ContinueChat.ts](../../core-lib/ContinueChat.ts) | درخواست continuation و بدنهٔ stream |
| `stopStream` | [StopStream.ts](../../core-lib/StopStream.ts) | درخواست توقف پاسخ remote |
| `createSessions` | [CreateSessions.ts](../../core-lib/CreateSessions.ts) | ایجاد نشست |
| `chatSessions`, `ChatSession` | [ChatSessions.ts](../../core-lib/ChatSessions.ts) | فهرست نشست‌های حساب |
| `deleteSession` | [DeleteSession.ts](../../core-lib/DeleteSession.ts) | ارسال درخواست حذف |
| `createPowChallenge` | [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) | دریافت چالش پیش از completion |
| `InvalidTokenError`, `assertValidTokenResponse`, `isInvalidTokenError` | [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) | طبقه‌بندی خطای اعتبارنامه |

این رابط‌ها به API وب DeepSeek وابسته‌اند و API رسمی مبتنی بر کلید نیستند. جزئیات در [انتقال DeepSeek](../explanation/deepseek-transport.md).

## ابزارها و فرمان‌ها

| ماژول | نمادهای مهم | قرارداد |
| --- | --- | --- |
| [tools/types.ts](../../source/tools/types.ts) | `Tool`, `ToolCall`, `ToolResult`, `ToolMode`, `ToolConfirmationDetails`, `ConfirmationHandler` | توصیف، ورودی، خروجی و تأیید |
| [registry.ts](../../source/tools/registry.ts) | `tools`, `getToolDescriptions` | registry و متن ابزار برای prompt |
| [execute.ts](../../source/tools/execute.ts) | `executeTool`, `executeToolCalls`, `toolIsMutating`, `toolRequiresConfirmation` | dispatch و سیاست batch؛ سیاست در تک‌اجرا تضمین نمی‌شود |
| [parse.ts](../../source/tools/parse.ts) | `parseToolCalls` | تبدیل markup به call |
| [arguments.ts](../../source/tools/arguments.ts) | `stringArgument`, `textArgument`, `positiveIntegerArgument` | خواندن و تبدیل آرگومان |
| [paths.ts](../../source/tools/paths.ts) | `rootDirectory`, `safePath`, `safeTargetPath` | کنترل مسیرهای ابزار |
| [todo.ts](../../source/tools/todo.ts) | `todoTools`, `getTodoList` | state task در حافظه |
| [commands/index.ts](../../source/commands/index.ts) | `SlashCommand`, `SlashCommandContext`, `slashCommands`, `resolveSlashCommand`, `formatSlashCommandHelp` | dispatch فرمان رابط |

[tools/index.ts](../../source/tools/index.ts) barrel مصرف‌کنندگان است. منطق preview در [describe.ts](../../source/tools/describe.ts)، diff نمایشی در [diff.ts](../../source/tools/diff.ts) و elevation در [sudo.ts](../../source/tools/sudo.ts) قرار دارد. افزودن export بدون ثبت در registry برای در دسترس قرارگرفتن ابزار مدل کافی نیست.

## نشست، ذخیره‌سازی و رابط

| ماژول | نماد/خانواده | مسئولیت |
| --- | --- | --- |
| [TokenConfig.ts](../../source/core/TokenConfig.ts) | tokenConfigDirectory، tokenConfigPath، save/clear/get preferenceها | فایل تنظیمات home |
| [SessionTranscript.ts](../../source/core/SessionTranscript.ts) | `sessionTranscriptDirectory`, `loadSessionTranscript`, `saveSessionTranscript`, `deleteSessionTranscript` | رونوشت محلی با validation شناسه |
| [useChatSession.ts](../../source/components/chat/useChatSession.ts) | `useChatSession` | initialize، reset، switch و cleanup |
| [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) | `createLiveStream` | تجمیع updateهای زندهٔ UI |
| [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) | `useComposerKeys` | کنترل ورودی و mode |
| [chat/types.ts](../../source/components/chat/types.ts) | نوع‌های پیام و وضعیت رابط | قرارداد نمایش، نه payload OpenAI |

ابزارهای bidi، cursor، logging و discovery نیز internal utility هستند؛ جزئیات مالکیت فایل‌ها در [نقشهٔ پوشش](source-map.md) است.

## سرور

| ماژول | exportها/رابط‌های اصلی |
| --- | --- |
| [server/index.ts](../../source/server/index.ts) | `startServer`, `ServerOptions` |
| [session.ts](../../source/server/session.ts) | `enqueueCompletion`, `runCompletion`, `getSessionTools`, `rememberSessionTools` |
| [request.ts](../../source/server/request.ts) | `generateCompletionId`, `resolveModel`, `resolveThinkingEnabled`, `buildCompletionPrompt`, `validateTools`, `withToolInstructions`, `parseClientToolCalls`, `toUsage` |
| [errors.ts](../../source/server/errors.ts) | `ApiError`, `errorBody`, `sendError`, `toApiError` |
| [types.ts](../../source/server/types.ts) | `ChatCompletionRequest`, `ChatMessage`, `FunctionTool`, `ToolChoice`, `ReasoningEffort`, `CompletionUsage` و نوع‌های مرتبط |

برای استفادهٔ بیرونی، به‌جای import این توابع از [قرارداد HTTP](http-api.md) استفاده کنید. برای توسعهٔ داخل مخزن، [افزودن ابزار و فرمان](../development/extending-tools-and-commands.md) و [مرجع پرامپت](prompts.md) نقاط همراه را مشخص می‌کنند.