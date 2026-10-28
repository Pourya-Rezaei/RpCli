# نقشهٔ پوشش مخزن و شواهد

نسخه `0.10.1`؛ تاریخ `2026-10-28`. این فهرست، موجودی ۸۵ فایل تحت Git در شروع مستندسازی را به مسئولیت و محل توضیح مرتبط می‌کند. فایل‌های جدید `doc/` در این عدد نیستند. «پوشش» در اینجا پوشش مستندات است، نه test coverage.

برای هر تغییر، فایل مالک را در جدول پیدا کنید و صفحهٔ مرتبط را بازبینی کنید. ارجاع فایل برای اثبات اجرای موفق آن کافی نیست؛ وضعیت اجرا در [گزارش بررسی](../development/verification.md) ثبت شده است.

## پیکربندی، بسته‌بندی و راهنماهای ریشه

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [.editorconfig](../../.editorconfig) | whitespace، LF و encoding | [محیط توسعه](../development/setup-and-scripts.md) |
| [.gitattributes](../../.gitattributes) | قواعد Git برای نوع/پایان خط فایل‌ها | [محیط توسعه](../development/setup-and-scripts.md) |
| [.gitignore](../../.gitignore) | موارد خارج از tracking | [کشف و امنیت](../explanation/security-and-privacy.md) |
| [.prettierignore](../../.prettierignore) | استثناهای formatter، از جمله Markdown | [اعتبارسنجی](../development/verification.md) |
| [AGENTS.md](../../AGENTS.md) | راهنمای عامل repository و زمینهٔ prompt | [اختلاف‌ها](known-limitations.md) |
| [README.md](../../README.md) | معرفی فعلی و دستورهای کاربر | [اختلاف‌ها](known-limitations.md) |
| [package.json](../../package.json) | نسخه، dependency، bin، scripts و config ابزارها | [محیط توسعه](../development/setup-and-scripts.md) |
| [pnpm-lock.yaml](../../pnpm-lock.yaml) | نسخه‌های resolveشده و engine/peer dependency | [محدودیت محیط](known-limitations.md) |
| [pnpm-workspace.yaml](../../pnpm-workspace.yaml) | تنظیم workspace و سیاست release age | [محیط توسعه](../development/setup-and-scripts.md) |
| [tsconfig.json](../../tsconfig.json) | ورودی source/core-lib و خروجی dist | [محیط توسعه](../development/setup-and-scripts.md) |
| [copy-prompts.mjs](../../scripts/copy-prompts.mjs) | کپی assetهای prompt پس از compile | [پرامپت‌ها](prompts.md) |

## ورودی و orchestration

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [cli.tsx](../../source/cli.tsx) | parse، انتخاب serve/resume/commit/prompt/interactive | [CLI](cli.md) |
| [app.tsx](../../source/app.tsx) | gate توکن، نمای مناسب و cleanup خروج | [جریان اجرا](../explanation/execution-flows.md) |
| [agent.ts](../../source/actions/agent.ts) | حلقهٔ ابزار و پاسخ | [پروتکل](tool-protocol.md) |
| [commitMessage.ts](../../source/actions/commitMessage.ts) | diff و تولید پیام commit | [راهنمای کامیت](../guides/commit-messages.md) |
| [commands/index.ts](../../source/commands/index.ts) | registry فرمان‌های slash | [فرمان‌ها](keyboard-and-slash-commands.md) |

## مؤلفه‌های رابط

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [CommitView.tsx](../../source/components/CommitView.tsx) | تأیید و اجرای git commit | [راهنمای کامیت](../guides/commit-messages.md) |
| [FzfFilePicker.tsx](../../source/components/FzfFilePicker.tsx) | fuzzy picker داخلی فایل | [رابط ترمینال](../explanation/terminal-and-persian-rendering.md) |
| [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx) | کنترل اصلی گفتگو و ورودی | [جریان اجرا](../explanation/execution-flows.md) |
| [MarkdownText.tsx](../../source/components/MarkdownText.tsx) | تفسیر زیرمجموعهٔ Markdown برای Ink | [نمایش فارسی](../explanation/terminal-and-persian-rendering.md) |
| [RpCliLogo.tsx](../../source/components/RpCliLogo.tsx) | هویت و اطلاعات شروع رابط | [معماری](../explanation/architecture.md) |
| [SinglePromptView.tsx](../../source/components/SinglePromptView.tsx) | چرخهٔ درخواست تکی و cleanup | [جریان اجرا](../explanation/execution-flows.md) |
| [SlashCommandPicker.tsx](../../source/components/SlashCommandPicker.tsx) | انتخاب فرمان ثبت‌شده | [فرمان‌ها](keyboard-and-slash-commands.md) |
| [Spinner.tsx](../../source/components/Spinner.tsx) | نمایش انتظار | [معماری](../explanation/architecture.md) |
| [TokenSetupView.tsx](../../source/components/TokenSetupView.tsx) | فرم ورود توکن وب | [نصب](../tutorials/installation.md) |
| [ToolConfirmation.tsx](../../source/components/ToolConfirmation.tsx) | نمایش و حل تأیید ابزار | [ابزارهای محلی](local-tools.md) |
| [ChatStatusBar.tsx](../../source/components/chat/ChatStatusBar.tsx) | mode، search، thinking و مدل | [رابط ترمینال](../explanation/terminal-and-persian-rendering.md) |
| [MessageRow.tsx](../../source/components/chat/MessageRow.tsx) | نمایش نقش‌های گفتگو | [رابط ترمینال](../explanation/terminal-and-persian-rendering.md) |
| [ModelConfirmation.tsx](../../source/components/chat/ModelConfirmation.tsx) | تأیید تعویض مدل | [ادامه و مدل](../guides/resume-and-model-switching.md) |
| [PlanConfirmation.tsx](../../source/components/chat/PlanConfirmation.tsx) | تصمیم شروع plan | [جریان اجرا](../explanation/execution-flows.md) |
| [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) | بافر و زمان‌بندی update نمایش | [جریان اجرا](../explanation/execution-flows.md) |
| [chat/types.ts](../../source/components/chat/types.ts) | نوع پیام و state نمایش | [رابط‌ها](exported-symbols.md) |
| [useChatSession.ts](../../source/components/chat/useChatSession.ts) | چرخهٔ عمر نشست | [جریان اجرا](../explanation/execution-flows.md) |
| [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) | ویرایش متن و TAB | [کلیدها](keyboard-and-slash-commands.md) |

## هستهٔ مشترک

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [apiClient.ts](../../source/core/apiClient.ts) | state API، PoW و ارسال | [انتقال](../explanation/deepseek-transport.md) |
| [bidiText.ts](../../source/core/bidiText.ts) | ترتیب بصری فارسی با تشخیص محیط | [نمایش فارسی](../explanation/terminal-and-persian-rendering.md) |
| [Gitignore.ts](../../source/core/Gitignore.ts) | fallback قواعد ignore | [امنیت](../explanation/security-and-privacy.md) |
| [ListWorkspaceFiles.ts](../../source/core/ListWorkspaceFiles.ts) | فهرست Git یا پیمایش fallback | [ابزارها](local-tools.md) |
| [LogChat.ts](../../source/core/LogChat.ts) | لاگ تشخیصی اختیاری | [ذخیره‌سازی](configuration-and-storage.md) |
| [OpenDeepSeek.ts](../../source/core/OpenDeepSeek.ts) | بازکردن مرورگر بر اساس پلتفرم | [نصب](../tutorials/installation.md) |
| [SessionTranscript.ts](../../source/core/SessionTranscript.ts) | رونوشت محلی و validation شناسه | [ذخیره‌سازی](configuration-and-storage.md) |
| [textCursor.ts](../../source/core/textCursor.ts) | grapheme، موقعیت و واژه | [نمایش فارسی](../explanation/terminal-and-persian-rendering.md) |
| [TokenConfig.ts](../../source/core/TokenConfig.ts) | توکن و preference | [تنظیمات](configuration-and-storage.md) |
| [toolCallMarkup.ts](../../source/core/toolCallMarkup.ts) | نرمال‌سازی DSML و markup ناقص | [پروتکل](tool-protocol.md) |
| [bidi-js.d.ts](../../source/types/bidi-js.d.ts) | declaration محلی dependency bidi | [نمایش فارسی](../explanation/terminal-and-persian-rendering.md) |

## اتصال وب DeepSeek

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [Chat.ts](../../core-lib/Chat.ts) | completion، SSE و continuation | [انتقال](../explanation/deepseek-transport.md) |
| [ChatSessions.ts](../../core-lib/ChatSessions.ts) | فهرست نشست | [انتقال](../explanation/deepseek-transport.md) |
| [ContinueChat.ts](../../core-lib/ContinueChat.ts) | ادامهٔ stream | [انتقال](../explanation/deepseek-transport.md) |
| [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) | challenge | [انتقال](../explanation/deepseek-transport.md) |
| [CreateSessions.ts](../../core-lib/CreateSessions.ts) | ایجاد نشست | [انتقال](../explanation/deepseek-transport.md) |
| [DeleteSession.ts](../../core-lib/DeleteSession.ts) | درخواست حذف | [انتقال](../explanation/deepseek-transport.md) |
| [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) | تشخیص token نامعتبر | [انتقال](../explanation/deepseek-transport.md) |
| [StopStream.ts](../../core-lib/StopStream.ts) | توقف پاسخ remote | [انتقال](../explanation/deepseek-transport.md) |
| [core-lib/index.ts](../../core-lib/index.ts) | barrel export | [رابط‌ها](exported-symbols.md) |

## سرور و قرارداد HTTP

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [errors.ts](../../source/server/errors.ts) | ApiError و envelope خطا | [HTTP](http-api.md) |
| [http.ts](../../source/server/http.ts) | توکن و CORS | [HTTP](http-api.md) |
| [index.ts](../../source/server/index.ts) | endpointها و lifecycle listener | [HTTP](http-api.md) |
| [request.ts](../../source/server/request.ts) | prompt، مدل، function parsing | [HTTP](http-api.md) |
| [session.ts](../../source/server/session.ts) | صف، init و cache | [HTTP](http-api.md) |
| [stream.ts](../../source/server/stream.ts) | ساخت SSE | [HTTP](http-api.md) |
| [types.ts](../../source/server/types.ts) | نوع درخواست و پاسخ | [HTTP](http-api.md) |

## ابزارهای محلی

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [arguments.ts](../../source/tools/arguments.ts) | تبدیل و validation | [پروتکل](tool-protocol.md) |
| [describe.ts](../../source/tools/describe.ts) | آماده‌سازی پیش‌نمایش تأیید | [ابزارها](local-tools.md) |
| [diff.ts](../../source/tools/diff.ts) | diff نمایشی ساده | [امنیت](../explanation/security-and-privacy.md) |
| [execute.ts](../../source/tools/execute.ts) | dispatch، mode و batch | [ابزارها](local-tools.md) |
| [index.ts](../../source/tools/index.ts) | exportهای مصرف‌کنندگان | [رابط‌ها](exported-symbols.md) |
| [parse.ts](../../source/tools/parse.ts) | markup محلی | [پروتکل](tool-protocol.md) |
| [paths.ts](../../source/tools/paths.ts) | cwd و کنترل containment | [امنیت](../explanation/security-and-privacy.md) |
| [registry.ts](../../source/tools/registry.ts) | پیاده‌سازی ابزارها و شرح آن‌ها | [ابزارها](local-tools.md) |
| [sudo.ts](../../source/tools/sudo.ts) | elevation detection و بازنویسی فرمان | [امنیت](../explanation/security-and-privacy.md) |
| [todo.ts](../../source/tools/todo.ts) | taskهای حافظه‌ای | [ابزارها](local-tools.md) |
| [types.ts](../../source/tools/types.ts) | قرارداد ابزار و تأیید | [رابط‌ها](exported-symbols.md) |

## دارایی‌های prompt و آزمون‌ها

| فایل | مسئولیت | مرجع توضیح |
| --- | --- | --- |
| [continue.md](../../source/prompts/continue.md) | ادامهٔ کار | [پرامپت‌ها](prompts.md) |
| [executePlan.md](../../source/prompts/executePlan.md) | اجرای برنامهٔ تأییدشده | [پرامپت‌ها](prompts.md) |
| [gitMessage.md](../../source/prompts/gitMessage.md) | دستور پیام commit | [پرامپت‌ها](prompts.md) |
| [prompts/index.ts](../../source/prompts/index.ts) | builder و AGENTS | [پرامپت‌ها](prompts.md) |
| [init.md](../../source/prompts/init.md) | تولید AGENTS | [پرامپت‌ها](prompts.md) |
| [plan.md](../../source/prompts/plan.md) | برنامه‌ریزی | [پرامپت‌ها](prompts.md) |
| [server.md](../../source/prompts/server.md) | زمینهٔ adapter HTTP | [پرامپت‌ها](prompts.md) |
| [system.md](../../source/prompts/system.md) | زمینهٔ عامل | [پرامپت‌ها](prompts.md) |
| [tools.md](../../source/prompts/tools.md) | قالب و دستور ابزار | [پرامپت‌ها](prompts.md) |
| [bidiText.test.ts](../../source/core/bidiText.test.ts) | ۵ case برای bidi | [تست](../development/testing.md) |
| [toolCallMarkup.test.ts](../../source/core/toolCallMarkup.test.ts) | ۳ case برای markup | [تست](../development/testing.md) |
| [tools.test.ts](../../test/tools.test.ts) | ۴ case ابزار و markup | [تست](../development/testing.md) |
| [lines.txt](../../test/fixtures/lines.txt) | fixture خواندن بازه‌ای | [تست](../development/testing.md) |

## روش کنترل

کنترل موجودی با `git ls-files` و مقایسهٔ مقصد پیوندهای همین صفحه انجام می‌شود. اضافه‌شدن فایل source بدون ردیف، نشانهٔ نیاز به بازبینی است. پیوند داشتن یک فایل کمکی به صفحهٔ مالک، لزوماً به معنی نیاز به یک سند مستقل برای آن فایل نیست؛ جداسازی اسناد بر اساس مسئولیت است، نه تعداد فایل‌ها.