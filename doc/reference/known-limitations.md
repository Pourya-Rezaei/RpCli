# محدودیت‌ها و اختلاف‌های شناخته‌شده

نسخهٔ بررسی `0.10.1`، تاریخ `2026-10-28`. این فهرست نتیجهٔ تحلیل ایستا است؛ گزارش بهره‌برداری یا نتیجهٔ تست runtime نیست. کد یا مستندات ریشه در این کار اصلاح نشده‌اند.

## شیوهٔ خواندن

«قطعی از کد» یعنی کنترل جریان یا تنظیم مربوط قابل مشاهده است؛ «ریسک ایستا» نیازمند آزمون اختصاصی است. اولویت بیانگر اهمیت تصمیم و استفاده است، نه امتیاز رسمی امنیت. هر ردیف شناسه دارد تا در issue یا بازبینی بعدی قابل ارجاع باشد.

## اجرا، ایمنی و داده

| شناسه | اهمیت | یافته | شاهد و نتیجه |
| --- | --- | --- | --- |
| SEC-01 | بالا | shell محصور نیست؛ قطعی از کد | [registry.ts](../../source/tools/registry.ts): exec بدون sandbox یا timeout؛ تأیید را دقیق بررسی کنید |
| SEC-02 | بالا | symlink نهایی مقصد نوشتن؛ ریسک ایستا | [paths.ts](../../source/tools/paths.ts): safeTargetPath فقط والد را realpath می‌کند؛ preview ممکن است در normal زودتر مانع شود |
| SEC-03 | بالا | search روی فایل Git-listed؛ ریسک ایستا | [registry.ts](../../source/tools/registry.ts): safePath مستقل برای هر فایل نیست و fs.stat/readFile می‌تواند symlink را دنبال کند |
| SEC-04 | بالا | سرور با توکن تنظیم‌شده auth مستقل مشتری ندارد | [http.ts](../../source/server/http.ts): محیط بر Bearer مقدم است؛ پورت عمومی بدون کنترل بیرونی مناسب نیست |
| SEC-05 | بالا | نشست و صف مشترک، بدون tenant isolation | [session.ts](../../source/server/session.ts): state global و cache حافظه‌ای؛ صف به معنی جداسازی نیست |
| SEC-06 | متوسط | تشخیص sudo shell-aware نیست | [sudo.ts](../../source/tools/sudo.ts): regex و بازنویسی متن می‌تواند معنای نقل‌قول را تغییر دهد |
| DATA-01 | متوسط | خاموشی logging مانع ذخیرهٔ transcript نیست | [SessionTranscript.ts](../../source/core/SessionTranscript.ts): مکان مستقل برای resume |
| DATA-02 | متوسط | پایان درخواست حذف اثبات حذف نیست | [DeleteSession.ts](../../core-lib/DeleteSession.ts): HTTP success assert نشده است |
| PATH-01 | پایین | برخی نام‌های داخل workspace بیش از حد رد می‌شوند | [paths.ts](../../source/tools/paths.ts): startsWith('..') فراتر از parent segment است |

شرح تهدیدها و حدود ادعاها در [امنیت و حریم خصوصی](../explanation/security-and-privacy.md) قرار دارد. این صفحه تنها نمایهٔ ریسک‌هاست.

## سازگاری و رفتار

| شناسه | یافته | شاهد و اثر |
| --- | --- | --- |
| ENV-01 | engine اعلام‌شده و dependency ناسازگار | [package.json](../../package.json) Node >=18؛ [pnpm-lock.yaml](../../pnpm-lock.yaml) Ink 7 به Node >=22 نیاز دارد؛ پیشنهاد عملی 22+ حاصل dependency است، نه تست همهٔ نسخه‌ها |
| ENV-02 | peer/type mismatch بالقوه | React 19 و @types/react 18 و peerهای متفاوت بعضی Ink dependencyها؛ احتمال ناسازگاری، نه اثبات شکست build |
| TOOL-01 | invoke بدون parameter parse نمی‌شود | [parse.ts](../../source/tools/parse.ts): با ابزارهای todo بدون آرگومان ناسازگاری وجود دارد |
| TODO-01 | درخت todo عمومی و پایدار نیست | [todo.ts](../../source/tools/todo.ts): نمایش/تکمیل یک سطح و پاک‌سازی غیر بازگشتی فرزندان split |
| API-01 | تنها آخرین پیام درخواست پردازش می‌شود | [request.ts](../../source/server/request.ts): replay تاریخچه و system role کامل انجام نمی‌شود |
| API-02 | model انتخاب واقعی Expert نیست | [request.ts](../../source/server/request.ts) و [session.ts](../../source/server/session.ts): برچسب و پیش‌فرض thinking |
| API-03 | named tool الزام صفر call را کامل کنترل نمی‌کند | [server/index.ts](../../source/server/index.ts): some با آرایهٔ خالی false است؛ required رشته‌ای بررسی مستقل دارد |
| API-04 | schema function و tool-result id enforce نمی‌شود | [request.ts](../../source/server/request.ts): محدودیت اعتبارسنجی، نه sandbox برای نتیجه |
| API-05 | stream ابزار بافر می‌شود | [server/index.ts](../../source/server/index.ts): خروجی پس از parse، نه توکن‌به‌توکن |
| API-06 | disconnect لغو فوری upstream نیست | همان فایل: stop در finally پس از await؛ deadline عمومی نیز وجود ندارد |
| MODEL-01 | retry پاسخ thinking-only مدل را حفظ نمی‌کند | [apiClient.ts](../../source/core/apiClient.ts): درخواست‌های نهایی مجدد modelType را حذف می‌کنند |
| BUILD-01 | build clean نیست و watch prompt را دائمی copy نمی‌کند | [copy-prompts.mjs](../../scripts/copy-prompts.mjs) و scripts پکیج |
| UI-01 | rendering Markdown زیرمجموعه‌ای است | [MarkdownText.tsx](../../source/components/MarkdownText.tsx): خروجی مرورگر یا پشتیبانی همهٔ tokenها نیست |
| UI-02 | صفحه‌کلید و bidi وابسته به ترمینال‌اند | [bidiText.ts](../../source/core/bidiText.ts)، [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts)؛ inline code معافیت code block را ندارد |

## اختلاف با مستندات ریشه

| سند فعلی | ادعا یا کمبود | مبنای درست برای این مجموعه |
| --- | --- | --- |
| [README.md](../../README.md) | --plan و /plan و /normal و /yolo | در CLI/registry ثبت نشده‌اند؛ TAB حالت را عوض می‌کند |
| همان سند | /clear فهرست todo را پاک می‌کند | reset گفتگو و حذف transcript با todo_clear یکسان نیست |
| همان سند | PageUp/PageDown اسکرول صفحه‌ای | handler مقدار ±1 می‌فرستد |
| همان سند | redirect فقط پاسخ نهایی | quiet فقط نمایش thinking را کنترل می‌کند؛ stdout ساخت‌یافته تضمین نیست |
| همان سند | محدودیت مسیر و تأیید به‌شکل کلی | shell، symlink و mode استثناهای مهم دارند |
| [AGENTS.md](../../AGENTS.md) | نبود تست، استفاده از DEEPSEEK_API_KEY | ۱۲ test تعریف شده؛ برنامه DEEPSEEK_TOKEN وب می‌خواهد |
| همان سند | مقصد prompt و clean build | خروجی dist/source/prompts؛ مرحلهٔ clean وجود ندارد |
| همان سند | server routing در App و Conventional Commits enforceشده | server در CLI انتخاب می‌شود؛ قالب commit بیشتر دستور prompt است |

این اختلاف‌ها برای جلوگیری از سردرگمی ثبت شده‌اند؛ متن قدیمی بی‌اجازه بازنویسی نشده است. هنگام اصلاح هر مورد، شناسه، شاهد و صفحهٔ مرجع مرتبط باید هم‌زمان به‌روزرسانی شوند.

## مواردی که هنوز تأیید اجرایی ندارند

ارتباط با حساب واقعی DeepSeek، رفتار stream در تغییرات API وب، همهٔ ترمینال‌ها، elevation در پلتفرم‌های مختلف، cancellation، symlinkها و چندکلاینت هم‌زمان در این کار اجرا نشده‌اند. شمار تست‌های موجود محدود به bidi، markup و بخشی از ابزار است؛ [مرجع تست](../development/testing.md) معیار گسترش پوشش را توضیح می‌دهد.

نباید از وجود این فهرست نتیجه گرفت سایر بخش‌ها بدون اشکال‌اند. برای مقادیر و قراردادهای جاری به [CLI](cli.md)، [ابزارها](local-tools.md) و [HTTP API](http-api.md) رجوع کنید.