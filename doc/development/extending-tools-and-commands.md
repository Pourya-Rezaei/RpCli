# افزودن ابزار محلی یا فرمان اسلش

این راهنما نقاط تغییر را معرفی می‌کند؛ قرارداد کامل را از مراجع پیوندشده بخوانید.
ابتدا مشخص کنید نیاز، ابزار قابل فراخوانی مدل است یا فرمان مستقیم رابط کاربر.

## ابزار محلی

قرارداد `Tool` شامل `name`، `description`، `execute` و `requiresConfirmation` اختیاری است.
پیاده‌سازی را در مرز [source/tools/types.ts](../../source/tools/types.ts) نگه دارید.
ابزار را به `tools` در [source/tools/registry.ts](../../source/tools/registry.ts) اضافه کنید.
`getToolDescriptions` از همین registry توضیح پرامپت می‌سازد.
این قرارداد schema ماشینی پارامترها ندارد؛ توضیح و اعتبارسنجی زمان اجرا جدا هستند.
نوع، مقدار پیش‌فرض، بازه و رفتار خطا را در هر دو هماهنگ نگه دارید.
از الگوی اعتبارسنجی [source/tools/arguments.ts](../../source/tools/arguments.ts) استفاده کنید.

برای عملیات فایل، الگوی مسیر [source/tools/paths.ts](../../source/tools/paths.ts) را بررسی کنید.
`safePath` و `safeTargetPath` هم‌ارز نیستند؛ دومی والد هدف را resolve می‌کند.
برای هدف موجود symlink، صرف بررسی والد را تضمین confinement ندانید.
برای ابزار تغییردهنده، metadata تأیید و پیش‌نمایش مناسب لازم است.
پیش‌نمایش‌ها در [source/tools/describe.ts](../../source/tools/describe.ts) ساخته می‌شوند.
کنترل mode و تأیید در `executeToolCalls` است، نه خود `executeTool`.
فراخوانی مستقیم `executeTool` این کنترل‌ها را دور می‌زند؛ آن را مسیر امن عمومی معرفی نکنید.

نام و پارامترها را با parser فعلی تطبیق دهید.
parser برای invoke حداقل یک parameter می‌خواهد، حتی اگر توضیح ابزار آرگومان نخواهد.
برای قالب دقیق، [پروتکل ابزار](../reference/tool-protocol.md) را بخوانید.
ابزارهای کلاینت HTTP از این registry جدا هستند و سرور آن‌ها را محلی اجرا نمی‌کند.

## فرمان اسلش

مالک تعریف [source/commands/index.ts](../../source/commands/index.ts) است.
`SlashCommand` نام، alias اختیاری، توضیح و تابع `execute` می‌گیرد.
وابستگی رابط را از `SlashCommandContext` عبور دهید؛ در صورت نیاز context را در مصرف‌کننده نیز وصل کنید.
فرمان را به `slashCommands` اضافه کنید تا help و resolver از همان تعریف استفاده کنند.
`resolveSlashCommand` کل متن trim‌شده و lowercase‌شده را تطبیق می‌دهد، نه یک parser آرگومان عمومی.
برای فرمان پارامتردار، این محدودیت را صریح طراحی و آزمایش کنید.
تغییر حالت‌های موجود را با فرمان ساختگی مستند نکنید؛ مسیر فعلی TAB است.

## بررسی پیش از تحویل

برای آرگومان نامعتبر، خطا و مسیر موفق آزمون محدود بنویسید.
برای ابزار تغییردهنده، رفتار `plan`، `normal`، `yolo` و رد تأیید را نیز بررسی کنید.
پس از تغییر TypeScript، build یا typecheck و سپس آزمون مرتبط را اجرا کنید.
توضیح registry، مرجع و نمونه مصرف‌کننده را همزمان به‌روز کنید.
دستورهای آماده در [آزمون](testing.md) و جدول رفتار در [ابزارهای محلی](../reference/local-tools.md) است.

## شواهد و اعتبار

مالک اجرا: [source/tools/execute.ts](../../source/tools/execute.ts)، `executeToolCalls` و `toolIsMutating`.
مالک parsing: [source/tools/parse.ts](../../source/tools/parse.ts)، `parseToolCalls`.
نقشه توسعه در [نمادها](../reference/exported-symbols.md) و [نمایه](../README.md) آمده است.
نسخه مبنا: **0.10.1**؛ بازبینی: **2026-10-28**؛ تحلیل ایستا، بدون آزمون اجرایی DeepSeek.