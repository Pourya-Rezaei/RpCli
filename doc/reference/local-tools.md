# مرجع ابزارهای محلی

نسخهٔ مبنا `0.10.1`؛ تاریخ بررسی `2026-10-28`. قرارداد زیر حاصل خواندن کد است، نه اجرای عملیات روی فایل‌های کاربر.

## مالکیت و مدل اجرا

تعریف ابزارها در [registry.ts](../../source/tools/registry.ts)، قراردادها در [types.ts](../../source/tools/types.ts)، تبدیل ورودی در [arguments.ts](../../source/tools/arguments.ts) و هماهنگی اجرا در [execute.ts](../../source/tools/execute.ts) است. `tools` شامل ۱۲ ابزار است. `getToolDescriptions` توصیف متنی را برای prompt تولید می‌کند؛ JSON Schema مرکزی برای اعتبارسنجی همهٔ ابزارها وجود ندارد.

## فایل، جستجو و shell

| ابزار | پارامترها | رفتار و خروجی | تأیید در normal |
| --- | --- | --- | --- |
| `list_directory` | `path?` پیش‌فرض `.` | ورودی‌های مستقیم، با برچسب file یا directory؛ غیر بازگشتی | خیر |
| `read_file` | `path`, `offset?`, `limit?` | متن UTF-8؛ offset یک‌مبنا و پیش‌فرض ۱؛ limit تعداد خطوط | خیر |
| `write_file` | `path`, `content` | ایجاد یا بازنویسی کامل؛ خروجی تعداد بایت | بله |
| `edit_file` | `path`, `old_text`, `new_text` | جایگزینی فقط یک تطابق دقیق و یکتا | بله |
| `delete_file` | `path` | حذف فایل معمولی؛ نه پوشه | بله |
| `search_files` | `query`, `path?` پیش‌فرض `.` | حداکثر ۵۰ خط با قالب path:line:text | خیر |
| `run_command` | `command` | اجرای shell؛ نتیجهٔ stdout و stderr | بله |

`read_file` فایل غیرمعمولی و اندازهٔ بیش از `100 * 1024` بایت را رد می‌کند؛ بازه پس از خواندن کل فایل محاسبه می‌شود. offset و limit باید عدد صحیح مثبت باشند. درخواست بازه‌ای خارج از انتهای فایل ممکن است متن خالی برگرداند.

`edit_file` تطابق نداشتن یا چند تطابق را خطا می‌داند. `old_text` خالی معتبر نیست؛ `new_text` می‌تواند خالی باشد. `write_file` پوشهٔ والد گمشده را ایجاد نمی‌کند. محدودیت ۱۰۰ KiB خواندن به معنی وجود همان محدودیت برای نوشتن یا edit نیست.

`search_files` جستجوی literal و حساس به حروف است، نه regex. فایل‌های بزرگ‌تر از ۱۰۰ KiB کنار گذاشته می‌شوند. کشف فایل از [ListWorkspaceFiles.ts](../../source/core/ListWorkspaceFiles.ts) شروع می‌شود: ابتدا Git، سپس fallback مبتنی بر [Gitignore.ts](../../source/core/Gitignore.ts). این فیلتر، کنترل محرمانگی نیست؛ خواندن صریح فایل ignored همچنان ممکن است.

## فهرست کارها

مالک: [todo.ts](../../source/tools/todo.ts)، نمادهای `todoTools` و `getTodoList`.

| ابزار | پارامترها | اثر |
| --- | --- | --- |
| `todo_add` | `description`, `parent_id?` | ایجاد task با وضعیت pending؛ افزودن فرزند به والد done رد می‌شود |
| `todo_list` | ندارد | نمایش فهرست و درصد تکمیل |
| `todo_update` | `id`, `status` | status یکی از pending، in-progress، done |
| `todo_split` | `id`, `subtasks` | جایگزینی فرزندان مستقیم و بازگشت والد به pending |
| `todo_clear` | ندارد | پاک‌کردن Map و بازنشانی شناسهٔ بعدی به ۱ |

همهٔ خروجی‌های todo متن قالب‌بندی‌شده هستند؛ حتی اگر description از «بازگرداندن id» صحبت کند، قرارداد خروجی JSON ساخت‌یافته نیست. داده فقط در حافظهٔ پردازش نگهداری می‌شود و با transcript نشست بازیابی نمی‌شود.

`subtasks` آرایه یا رشته می‌پذیرد؛ رشته بر اساس خط و در حالت تک‌خطی دارای ویرگول بر اساس ویرگول تقسیم می‌شود. تک‌مورد بیشتر از ۲۰۰ کاراکتر رد می‌شود. تکمیل والد فقط به فرزندان مستقیم نگاه می‌کند و propagation بازگشتی عمومی نیست. نمایش نیز ریشه و یک سطح فرزند را نشان می‌دهد.

فقط `todo_clear` دارای `requiresConfirmation: true` است؛ بقیه با وجود تغییر حافظه در طبقهٔ ابزارهای nonmutating این executor قرار می‌گیرند. بنابراین در plan هم قابل اجرا هستند. مشکل فراخوانی ابزار بدون پارامتر از markup در [پروتکل ابزار](tool-protocol.md) توضیح داده شده است.

## سیاست batch و تأیید

- `executeToolCalls`: batch سراسر nonmutating را موازی اجرا می‌کند؛ ترتیب نتیجه‌ها حفظ می‌شود.
- وجود یک ابزار mutating باعث اجرای ترتیبی تمام batch می‌شود.
- در plan ابزارهای طبقه‌بندی‌شده به‌عنوان mutating مسدود می‌شوند.
- در yolo تأییدهای معمول حذف می‌شوند؛ elevation همچنان تأیید می‌خواهد.
- خطای یک ابزار معمولاً اجرای ابزار بعد را متوقف نمی‌کند؛ رد تأیید باعث skipped شدن ادامهٔ batch می‌شود.
- `executeTool` به‌تنهایی سیاست mode و تأیید را اعمال نمی‌کند؛ این مسئولیت caller است.

نتیجهٔ عمومی `ToolResult` شامل `ok`، `tool_name` و در موفقیت `result` یا در شکست `error` است. آرایه‌ها با newline و خروجی shell با بخش stderr به متن تبدیل می‌شوند. خطا exception خام را به مدل پرتاب نمی‌کند؛ به نتیجهٔ ناموفق تبدیل می‌شود.

## مرز ایمنی

[paths.ts](../../source/tools/paths.ts) cwd اولیه را ثبت می‌کند. `safePath` مسیر موجود و realpath را کنترل می‌کند؛ `safeTargetPath` برای مقصد نوشتن تنها والد را resolve می‌کند. symlink نهایی مقصد و مسیر جستجو محدودیت‌های جداگانه دارند؛ شرح دقیق در [امنیت](../explanation/security-and-privacy.md).

اجرای shell عادی با cwd اولیه، محیط ارث‌برده و buffer یک MiB انجام می‌شود؛ timeout و sandbox ندارد. مسیر elevation در [sudo.ts](../../source/tools/sudo.ts) با تشخیص متنی `sudo` و `@slosk/sudo-prompt` کار می‌کند، نه parser واقعی shell. تأیید کاربر، محدودیت سطح سیستم‌عامل ایجاد نمی‌کند.

برای تفاوت این ابزارها با functionهای HTTP، [مرجع API](http-api.md) را بخوانید.