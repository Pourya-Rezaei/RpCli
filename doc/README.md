# مستندات تحلیلی RpCli

این مجموعه برای فهم، استفاده و توسعهٔ RpCli تهیه شده است: یک CLI گفتگو با DeepSeek، رابط ترمینال، عامل ابزارهای محلی و adapter HTTP. هدف، توضیح رفتار واقعی و قابل پیگیری کد است، نه تکرار معرفی محصول.

**نسخهٔ مبنا:** `0.10.1` از [package.json](../package.json)؛ **تاریخ تحلیل:** `2026-10-28`.

## اعتبار اطلاعات

تحلیل کد از آزمون اجرایی جداست. توضیح رفتارها بر پایهٔ فایل‌ها و نمادهای لینک‌شده است؛ مثال‌های دارای ارتباط DeepSeek یا تغییر فایل در این کار اجرا نشده‌اند. نتیجهٔ کنترل اسناد و وضعیت ابزارهای build در [گزارش بررسی](development/verification.md) ثبت می‌شود. محدودیت‌ها و اختلاف مستندات قدیمی در [فهرست موارد شناخته‌شده](reference/known-limitations.md) آمده‌اند.

تغییرات این مجموعه محدود به `doc/` است. کد برنامه، [README اصلی](../README.md)، [AGENTS.md](../AGENTS.md)، تنظیمات شخصی، توکن و خروجی build اصلاح نشده‌اند. در اختلاف میان معرفی قدیمی و رفتار پیاده‌سازی، برای فهم نسخهٔ مبنا به شاهد کد توجه کنید.

## مسیر مطالعه

| نیاز خواننده | مسیر پیشنهادی |
| --- | --- |
| اولین استفاده | نصب ← اولین نشست ← گردش کار تعاملی |
| فهم داخلی برنامه | معماری ← جریان اجرا ← انتقال DeepSeek ← نقشهٔ پوشش |
| ساخت کلاینت HTTP | راهنمای سرور ← مرجع API ← محدودیت‌ها و امنیت |
| تغییر کد | محیط توسعه ← رابط‌های مهم ← توسعهٔ ابزار/فرمان ← تست و نگهداری |
| ارزیابی نگهداری داده | تنظیمات و ذخیره‌سازی ← امنیت ← ادامهٔ نشست |

## آموزش‌های شروع

- [نصب و پیش‌نیازها](tutorials/installation.md)
- [اولین نشست](tutorials/first-session.md)

## راهنماهای انجام کار

- [گردش کار تعاملی](guides/interactive-workflows.md)
- [ادامهٔ نشست و تعویض مدل](guides/resume-and-model-switching.md)
- [تولید پیام کامیت](guides/commit-messages.md)
- [استفاده از سرور HTTP](guides/http-server.md)
- [عیب‌یابی](guides/troubleshooting.md)

## مرجع‌های فنی

- [خط فرمان و تقدم حالت‌ها](reference/cli.md)
- [تنظیمات و مخازن داده](reference/configuration-and-storage.md)
- [فرمان‌ها و صفحه‌کلید](reference/keyboard-and-slash-commands.md)
- [ابزارهای محلی و سیاست اجرا](reference/local-tools.md)
- [پروتکل فراخوانی ابزار](reference/tool-protocol.md)
- [قرارداد HTTP و ماتریس سازگاری](reference/http-api.md)
- [پرامپت‌ها و زمینهٔ پروژه](reference/prompts.md)
- [رابط‌ها و نمادهای مهم](reference/exported-symbols.md)
- [محدودیت‌ها و اختلاف‌های شناخته‌شده](reference/known-limitations.md)
- [نقشهٔ پوشش فایل‌های مخزن](reference/source-map.md)

## توضیح معماری

- [لایه‌ها و مسئولیت ماژول‌ها](explanation/architecture.md)
- [جریان‌های اجرای برنامه](explanation/execution-flows.md)
- [ارتباط، PoW و SSE در DeepSeek](explanation/deepseek-transport.md)
- [مرزهای اعتماد و حریم خصوصی](explanation/security-and-privacy.md)
- [رابط ترمینال و نمایش فارسی](explanation/terminal-and-persian-rendering.md)

## توسعه و کنترل کیفیت

- [راه‌اندازی و scriptها](development/setup-and-scripts.md)
- [تست‌های موجود و شکاف پوشش](development/testing.md)
- [افزودن ابزار و فرمان](development/extending-tools-and-commands.md)
- [نگهداری و کنترل تغییر مستندات](development/documentation-maintenance.md)
- [گزارش اعتبارسنجی این مجموعه](development/verification.md)

## واژه‌نامهٔ مشترک

| واژه | معنا در این پروژه |
| --- | --- |
| workspace / cwd | پوشه‌ای که فرایند از آن شروع می‌شود؛ مبنای ابزار فایل و AGENTS |
| نشست remote | گفتگو در حساب DeepSeek، با id و parent message |
| transcript / رونوشت | پیام‌های محلی رابط برای بازنمایی resume؛ مستقل از logging |
| log / لاگ | ثبت تشخیصی اختیاری prompt، event و نتیجه |
| generation / نسل پاسخ | یک نوبت تولید با state توقف مشترک |
| tool round / دور ابزار | دریافت call، اجرای آن و بازگرداندن نتیجه به مدل |
| continuation / ادامهٔ stream | ادامهٔ پاسخ ناقص upstream؛ متفاوت با دور ابزار و /continue |
| normal / yolo / plan | سیاست اجرای ابزار در رابط؛ نه مدل DeepSeek |
| default / expert | انتخاب مدل داخلی رابط؛ متفاوت با label مدل HTTP |
| client function | ابزار معرفی‌شده در HTTP که کلاینت اجرا می‌کند، نه registry محلی |
| prompt | دستور متنی به مدل؛ به معنی enforceشدن سیاست نیست |

## قواعد خوانایی و نگهداری

هر صفحه یک هدف دارد. جدول‌های قرارداد در reference نگهداری می‌شوند؛ آموزش و راهنما از آن‌ها استفاده می‌کنند. نام فایل‌ها و شناسه‌های کد ثابت و انگلیسی‌اند، اما توضیح‌ها فارسی‌اند. نمونه‌های چندخطی در code fence آمده‌اند تا ترتیب متن فارسی و فرمان‌ها کمتر مخدوش شود.

«تأیید از روی کد» با «تست‌شده» و «ریسک ایستا» با «آسیب‌پذیری اثبات‌شده» یکی نیست. تغییرات بعدی باید تاریخ، شاهد، قرارداد و محدودیت مرتبط را با هم بازبینی کنند؛ [دستور نگهداری](development/documentation-maintenance.md) و [نقشهٔ پوشش](reference/source-map.md) ابزار کنترل این فرایند هستند.