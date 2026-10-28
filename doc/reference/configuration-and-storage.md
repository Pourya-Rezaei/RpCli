# تنظیمات و ذخیره‌سازی

مبنای بررسی: نسخه `0.10.1` در `2026-10-28`؛ خواندن کد، بدون بررسی حساب یا فایل شخصی کاربر.

## منابع تنظیمات

[cli.tsx](../../source/cli.tsx) و [server/index.ts](../../source/server/index.ts) فایل تنظیمات را با dotenv بارگذاری می‌کنند؛ `override` درخواست نشده است. در نتیجه محیط موجود معمولاً بر مقدار همان نام در فایل اولویت دارد. این قاعده برای preference مدل استثنا دارد، چون مستقیماً از فایل خوانده می‌شود.

| نام | مصرف‌کننده | معنا و پیش‌فرض |
| --- | --- | --- |
| `DEEPSEEK_TOKEN` | App، API client، سرور | توکن نشست وب DeepSeek؛ پیش‌نیاز ارسال |
| `RC_TOKEN` | `getToken` سرور | fallback برای توکن بالادستی؛ نه رمز مستقل کلاینت |
| `DEEPSEEK_SESSION_ID` | API client هنگام import | شناسهٔ اولیهٔ نشست؛ initialization تعاملی جدید reset می‌کند |
| `DEEPSEEK_MESSAGE_ID` | API client هنگام import | شناسهٔ عددی اولیهٔ پیام والد |
| `RP_CLI_LOG_CHAT` | LogChat | فعال با `1`، `true` یا `yes`؛ پیش‌فرض خاموش |
| `RP_CLI_MODEL` | TokenConfig | مقدار ذخیره‌شدهٔ `default` یا `expert`؛ سایر مقادیر default |
| `PORT` | startServer | پیش‌فرض 3000؛ گزینهٔ CLI اولویت دارد |
| `HOST` | startServer | پیش‌فرض 127.0.0.1؛ گزینهٔ CLI اولویت دارد |

`DEEPSEEK_API_KEY` در این پیاده‌سازی جایگزین `DEEPSEEK_TOKEN` نیست. نام انتخابی مدل HTTP نیز با preference مدل تعاملی یکسان نیست.

## فایل تنظیمات

مالک: [TokenConfig.ts](../../source/core/TokenConfig.ts)، نمادهای `tokenConfigDirectory` و `tokenConfigPath`.

مسیر ثابت نسبت به home، روی همهٔ پلتفرم‌ها:

```text
~/.config/rp-cli/.env
```

متغیر `XDG_CONFIG_HOME` برای جابه‌جایی این مسیر استفاده نشده است. `saveDeepSeekToken`، `clearDeepSeekToken`، `saveChatLoggingPreference` و `saveModelPreference` مقادیر را در Map بازنویسی می‌کنند؛ نظرها و قالب‌بندی قبلی فایل حفظ نمی‌شوند. حذف آخرین entry باعث حذف فایل می‌شود.

`getModelPreference` فایل را مستقیم می‌خواند؛ set کردن صرف `RP_CLI_MODEL` در محیط الزاماً انتخاب مدل رابط را تغییر نمی‌دهد. `expert` تنها مقدار غیراستاندارد پذیرفته‌شده در این preference است.

دایرکتوری جدید با mode برابر `0700` و فایل با `0600` درخواست می‌شود. این درخواست برای ایجاد، به معنی تعمیر permission فایل یا پوشهٔ ازقبل‌موجود یا رمزنگاری داده نیست.

## سه مخزن مستقل

| مخزن | مکان/مالک | محتوای نگهداری‌شده | وابستگی به logging |
| --- | --- | --- | --- |
| لاگ تشخیصی | زیر `logs/<session-id>/` در config؛ LogChat | prompt، event و نتیجهٔ خام | بله |
| رونوشت تعاملی | `sessions/<session-id>.json` در config؛ SessionTranscript | user، thinking، assistant و console؛ بدون logo | خیر |
| نشست سرویس | حساب DeepSeek؛ core-lib | تاریخچه و وضعیت remote | خیر |

شواهد: [LogChat.ts](../../source/core/LogChat.ts)، [SessionTranscript.ts](../../source/core/SessionTranscript.ts) و [useChatSession.ts](../../source/components/chat/useChatSession.ts).

در خاموشی `/logging`، ذخیرهٔ transcript تعاملی لزوماً متوقف نمی‌شود. لاگ خام و transcript برای دو هدف مختلف‌اند و سیاست retention خودکار یکسانی ندارند. حذف محلی، حذف remote را اثبات نمی‌کند.

`SessionTranscript` شناسه را به حروف و ارقام، `_` و `-` محدود می‌کند. متن ذخیره‌شده ترتیب منطقی اصلی را دارد؛ تبدیل RTL مربوط به نمایش است. خطاهای logging عمدتاً بدون متوقف‌کردن گفتگو نادیده گرفته می‌شوند؛ نبود فایل لاگ همیشه نشانهٔ نبود درخواست نیست.

## چرخهٔ عمر و اولویت توکن

در سرور: `DEEPSEEK_TOKEN` سپس `RC_TOKEN` سپس Bearer درخواست انتخاب می‌شود. اگر توکن محیط موجود باشد، Bearer کلاینت برای احراز هویت مستقل مقایسه نمی‌شود. جزئیات در [http.ts](../../source/server/http.ts).

نشست تعاملی استفاده‌نشده در خروج به‌صورت best effort برای حذف remote فرستاده می‌شود؛ نشست استفاده‌شده برای resume حفظ می‌شود. `/clear` transcript قدیمی را حذف می‌کند و نشست گفتگو را reset می‌کند، نه همهٔ مخازن و نه todoها را. تعویض مدل پس از گفتگو نشست تازه‌ای با خلاصه می‌سازد.

`deleteSession` صرفاً دریافت پاسخ fetch را تضمین می‌کند؛ موفقیت HTTP را در wrapper assert نمی‌کند. از دید نگهداری داده نباید «await تمام شد» را معادل «اطلاعات از سرویس حذف شد» دانست.

## مدیریت امن

توکن واقعی، پوشهٔ config و رونوشت‌ها را در repository یا گزارش اشکال منتشر نکنید. مسیرهای workspace ابزار، فایل home تنظیمات خود برنامه را از دسترس shell جدا نمی‌کنند. برای مرزهای دقیق به [امنیت و حریم خصوصی](../explanation/security-and-privacy.md) و برای رفع مشکل به [عیب‌یابی](../guides/troubleshooting.md) رجوع کنید.