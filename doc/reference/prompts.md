# مرجع پرامپت‌ها

نسخه `0.10.1`، تاریخ بررسی `2026-10-28`. این فایل‌ها دستور به مدل‌اند؛ به‌تنهایی کنترل امنیتی یا تضمین کیفیت خروجی نیستند.

## سازنده‌ها و دارایی‌ها

مالک بارگذاری: [source/prompts/index.ts](../../source/prompts/index.ts). `loadPrompt` فایل مجاور module را همگام می‌خواند؛ missing asset می‌تواند هنگام فراخوانی builder خطا بدهد.

| builder | فایل | مصرف |
| --- | --- | --- |
| `SystemPrompt` | [system.md](../../source/prompts/system.md) و [tools.md](../../source/prompts/tools.md) | آغاز گفتگوی عامل محلی |
| `ServerSystemPrompt` | [server.md](../../source/prompts/server.md) | آغاز نشست سرور، بدون فعال‌سازی اجرای ابزار محلی |
| `InitPrompt` | [init.md](../../source/prompts/init.md) | درخواست تولید راهنمای AGENTS برای repository |
| `ContinuePrompt` | [continue.md](../../source/prompts/continue.md) | ادامهٔ وظیفه پس از توقف یا سقف ابزار |
| `GitCommitMessage` | [gitMessage.md](../../source/prompts/gitMessage.md) | دستور تولید پیام commit |
| `PlanPrompt(task)` | [plan.md](../../source/prompts/plan.md) | درج وظیفه در دستور برنامه‌ریزی |
| `ExecutePlanPrompt(mode)` | [executePlan.md](../../source/prompts/executePlan.md) | شروع برنامهٔ تأییدشده با حالت مقصد |

## تزریق زمینه

`SystemPrompt` مقدارهای platform و cwd و شرح ابزارها را جایگزین می‌کند. شرح ابزار از `getToolDescriptions` در registry می‌آید. راهنمای elevation متناسب با win32 یا سایر پلتفرم‌ها تولید می‌شود.

`readAgentsMarkdown` فقط فایل AGENTS در cwd را می‌خواند؛ پیمایش سلسله‌مراتبی والد یا راهنمای جدا در هر زیردایرکتوری ندارد. نبود فایل به رشتهٔ خالی تبدیل می‌شود؛ خطاهای خواندن فایل موجود الزاماً پنهان نمی‌شوند. SystemPrompt و ServerSystemPrompt محتوای موجود را به انتهای prompt اضافه می‌کنند.

ServerSystemPrompt فهرست registry ابزار محلی را inject نمی‌کند. functionهای درخواست HTTP در مسیر `withToolInstructions` به prompt درخواست اضافه می‌شوند و اجرای آن‌ها بر عهدهٔ کلاینت است.

## چه چیز enforce می‌شود؟

| موضوع | لایهٔ واقعی کنترل |
| --- | --- |
| توصیهٔ برنامه‌ریزی، پاسخ مناسب و Conventional Commits | متن prompt؛ تضمین ساختاری عمومی ندارد |
| مسدودکردن ابزار mutating در plan | executor، مستقل از اطاعت مدل |
| منع مسیر خارج cwd در ابزارهای مشخص | helper مسیر؛ با استثناهای مستندشده |
| تأیید عملیات | callback رابط و mode در executor |
| سقف دور ابزار | getAIResponse |

بنابراین prompt «فقط بخوان» را نباید sandbox دانست. همچنین تولید فایل AGENTS توسط `/init` یک عملیات وابسته به مدل و ابزار است، نه scaffold قطعی داخل فرمان slash.

## چرخهٔ build و تغییر

[scripts/copy-prompts.mjs](../../scripts/copy-prompts.mjs) فقط فایل‌های مستقیم `.md` را به `dist/source/prompts/` می‌برد. `pnpm build` ابتدا tsc سپس copy را اجرا می‌کند؛ `pnpm dev` یک‌بار copy و سپس watch TypeScript است. ویرایش Markdown لزوماً copy مجدد خودکار ایجاد نمی‌کند و خروجی stale پاک نمی‌شود.

برای تغییر، asset مبدا و builder و مصرف‌کننده را با هم بررسی کنید. پس از تغییر قرارداد ابزار، شرح registry و مثال قالب و parser نیز باید هماهنگ بمانند. [راهنمای نگهداری](../development/documentation-maintenance.md) فهرست کنترل مستندات را دارد.