# فرمان‌ها و صفحه‌کلید

نسخه `0.10.1`، بررسی کد در `2026-10-28`؛ رفتار ترمینال واقعی در این مرحله آزمایش نشده است.

## فرمان‌های ثبت‌شده

مالک: `slashCommands` و `resolveSlashCommand` در [commands/index.ts](../../source/commands/index.ts). تطبیق پس از trim و lowercase و به‌صورت کامل است؛ parser آرگومان عمومی برای slash command ندارد.

| فرمان | نام دیگر | اثر |
| --- | --- | --- |
| `/init` | ندارد | ارسال prompt تولید AGENTS و تلاش بارگذاری مجدد فایل پس از پاسخ |
| `/continue` | ندارد | ارسال ContinuePrompt برای ادامهٔ کار قبلی |
| `/clear` | ندارد | reset گفتگو؛ حذف transcript قبلی، نه پاک‌کردن todoها |
| `/help` | ندارد | نمایش کمک تولیدشده از registry فرمان‌ها |
| `/search` | ندارد | تغییر وضعیت جستجوی وب |
| `/thinking` | ندارد | تغییر وضعیت thinking |
| `/logging` | `/log` | تغییر ذخیرهٔ لاگ تشخیصی و preference آن |
| `/todos` | ندارد | نمایش فهرست todo حافظه‌ای |
| `/model` | ندارد | جابه‌جایی default و expert با روند تأیید رابط |
| `/exit` | `/quit` | درخواست خروج برنامه |

`/plan`، `/normal` و `/yolo` ثبت نشده‌اند. `/clear` سابقهٔ نمایش را با پاک‌کردن کل messages یکی نمی‌گیرد؛ logo دیگری اضافه می‌شود و state نشست بازنشانی می‌شود. اثر فرمان‌ها در [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx)، داخل `runCommand` قرار دارد.

## ورود متن و حالت

مالک: [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) و [textCursor.ts](../../source/core/textCursor.ts).

| کلید | اثر در composer فعال |
| --- | --- |
| TAB | چرخهٔ normal → yolo → plan → normal |
| جهت چپ/راست | حرکت بر پایهٔ مرز grapheme |
| Ctrl یا Meta همراه چپ/راست | حرکت واژه‌ای |
| Home / End | ابتدا/انتهای خط |
| Ctrl+Up / Ctrl+Down | ابتدا/انتهای متن |
| Up / Down | خط مجاور، در محدودهٔ متن چندخطی |
| Backspace / Delete | حذف پیش/پس از مکان‌نما |
| Ctrl همراه حذف | حذف واژه‌ای؛ تشخیص Backspace به encoding ترمینال هم وابسته است |

TAB هنگام picker، loading یا تأیید فعال الزاماً چرخهٔ mode را اجرا نمی‌کند؛ hook composer در این وضعیت‌ها غیرفعال است. ورودی و submit همچنین از component textarea و Ink اثر می‌گیرند؛ این جدول ادعای سازگاری کامل با همهٔ terminal emulatorها نیست.

## اسکرول، توقف و خروج

مالک: handler عمومی `useInput` در [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx).

- PageUp و PageDown در نبود picker، `scrollBy(-1)` و `scrollBy(1)` می‌زنند؛ جهش یک صفحهٔ کامل پیاده نشده است.
- Ctrl+Home و Ctrl+End ابتدا و انتهای نمایش گفتگو را هدف می‌گیرند.
- Esc هنگام loading درخواست توقف می‌دهد؛ اگر initialization موفق نشده باشد، توقف فوری تمام کارهای upstream تضمین نمی‌شود.
- Ctrl+C هنگام loading توقف را درخواست می‌کند؛ در حالت idle با متن موجود، ابتدا متن را پاک می‌کند.
- Ctrl+C با ورودی خالی و نشست موجود ابتدا پیام resume را نمایش می‌دهد؛ فشار بعدی خروج با کد 130 را درخواست می‌کند.

## انتخاب‌گر و تأیید

[FzfFilePicker.tsx](../../source/components/FzfFilePicker.tsx) fuzzy matcher داخلی دارد؛ اجرای executable خارجی fzf لازم نیست. انتخاب فایل فقط متن `@path` را وارد composer می‌کند و محتوای فایل را خودکار attach نمی‌کند. [SlashCommandPicker.tsx](../../source/components/SlashCommandPicker.tsx) برای انتخاب فرمان ثبت‌شده است.

پنجره‌های [ToolConfirmation.tsx](../../source/components/ToolConfirmation.tsx)، [PlanConfirmation.tsx](../../source/components/chat/PlanConfirmation.tsx) و [ModelConfirmation.tsx](../../source/components/chat/ModelConfirmation.tsx) قراردادهای جدا دارند. تأیید ابزار با `y` یا Enter و رد با `n` انجام می‌شود؛ Enter در اینجا تأیید است، نه گزینهٔ پیش‌فرض امنِ رد. plan با Enter یا y آغاز و با Esc یا n رد می‌شود؛ مدل با Enter تأیید و Esc لغو می‌شود.

پذیرش plan، حالت قبلی غیرplan را برمی‌گرداند؛ اگر قبل از plan در yolo بوده‌اید، اجرای برنامه هم می‌تواند در yolo شروع شود. برای اثر mode بر ابزارها [local-tools.md](local-tools.md) را ببینید.