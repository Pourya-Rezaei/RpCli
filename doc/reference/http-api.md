# مرجع HTTP API

نسخهٔ مبنا `0.10.1`، بررسی ایستا در `2026-10-28`. پاسخ‌ها و مثال‌ها از کد استخراج شده‌اند، نه از اجرای سرویس DeepSeek. این رابط adapter حالت‌دار است، نه پیاده‌سازی کامل OpenAI.

## منابع مسئول

[index.ts](../../source/server/index.ts): routeها و `startServer`؛ [request.ts](../../source/server/request.ts): ساخت prompt و function parsing؛ [session.ts](../../source/server/session.ts): صف و نشست؛ [stream.ts](../../source/server/stream.ts): SSE؛ [types.ts](../../source/server/types.ts): شکل‌های TypeScript؛ [http.ts](../../source/server/http.ts): CORS و توکن؛ [errors.ts](../../source/server/errors.ts): خطا.

## راه‌اندازی و endpointها

پیش‌فرض bind برابر `127.0.0.1:3000` است. port گزینه بر `PORT` اولویت دارد و فقط عدد صحیح ۱ تا ۶۵۵۳۵ پذیرفته می‌شود؛ مقدار نامعتبر به ۳۰۰۰ برمی‌گردد. host گزینه بر `HOST` اولویت دارد. اندازهٔ JSON body حداکثر `10mb` تنظیم شده است.

| متد و مسیر | رفتار |
| --- | --- |
| `GET /health` | status برابر ok، object برابر health و timestamp؛ سلامت اتصال DeepSeek را نمی‌سنجد |
| `GET /v1/models` | list با یک مدل `deepseek`، created برابر ۰ و owned_by برابر deepseek |
| `GET /v1/models/deepseek` | شیء همان مدل |
| `GET /v1/models/:model` برای نام دیگر | 404 با model_not_found |
| `POST /v1/chat/completions` | ادامهٔ نشست فعال یا ساخت نشست لازم |
| `POST /v1/chat/completions/:sessionId` | تلاش برای انتخاب نشست مشخص |
| `OPTIONS *` | پاسخ 204 پس از middleware مربوط به CORS |

## ورودی completion

| فیلد | رفتار واقعی |
| --- | --- |
| `messages` | آرایهٔ غیرخالی؛ فقط آخرین item برای ساخت prompt استفاده می‌شود |
| آخرین `content` | string یا محتوای متنی استخراج‌شده از array؛ متن trimشده نباید خالی باشد |
| آخرین `role: tool` | متن با name و tool_call_id در بلوک tool_result پیچیده می‌شود |
| `model` | برچسب پاسخ و پیش‌فرض thinking؛ انتخاب مستقیم Expert نیست |
| `thinking_enabled` | در صورت ارائه بر reasoning_effort و پیش‌فرض مدل اولویت دارد |
| `reasoning_effort` | none/minimal خاموش؛ low/medium/high/xhigh/max روشن؛ درجات واقعی بودجه اعمال نمی‌شوند |
| `search_enabled` | پیش‌فرض false؛ جستجوی بالادستی |
| `stream` | فقط مقدار دقیق true مسیر SSE را فعال می‌کند |
| `stream_options.include_usage` | در true، chunk پایانی usage با choices خالی |
| `n` | غایب یا ۱؛ سایر مقادیر 400 |
| `tools` | functionها با نام معتبر؛ غایب یعنی استفاده از cache نشست در صورت وجود |
| `tool_choice` | auto، none، required یا انتخاب function با نام |
| `temperature`, `max_tokens` | در type حضور دارند، ولی به transport ارسال نمی‌شوند |

اعتبارسنجی کامل JSON Schema برای body وجود ندارد؛ وجود یک field در type تضمین validation runtime نیست. content array فقط متن را استخراج می‌کند و پردازش تصویر پیاده نشده است. پیام‌های قبلی، حتی system، برای ساخت prompt replay نمی‌شوند.

`resolveModel` نام غایب یا `deepseek-chat` را بدون thinking می‌گیرد؛ `deepseek-reasoner` و نام‌های مطابق reasoner یا R1 پیش‌فرض thinking را روشن می‌کنند. نام‌های دیگر نیز به‌عنوان برچسب پذیرفته می‌شوند؛ فهرست مدل‌ها وعدهٔ وجود همهٔ این مدل‌ها نیست.

## پاسخ JSON

```json
{
  "id": "chatcmpl-<generated>",
  "object": "chat.completion",
  "created": 0,
  "model": "deepseek-chat",
  "system_fingerprint": null,
  "choices": [{
    "index": 0,
    "message": {"role": "assistant", "content": "<answer>", "refusal": null},
    "logprobs": null,
    "finish_reason": "stop"
  }],
  "usage": {"prompt_tokens": 0, "completion_tokens": 0, "total_tokens": 0}
}
```

مثال شکل پاسخ است؛ created در واقع timestamp ثانیه‌ای و id تصادفی است. reasoning_content در صورت thinking یا وجود reasoning افزوده می‌شود. در function call، message شامل tool_calls است؛ content بدون متن می‌تواند null و finish_reason برابر tool_calls باشد. صفرهای usage ممکن است fallback نبود اطلاعات باشند، نه مصرف واقعی صفر.

هدر `X-RP-Session-Id` در صورت دسترسی پیش از ارسال header، شناسهٔ نشست را برمی‌گرداند. برای ادامه، مسیر دارای sessionId از تکیه بر نشست global صریح‌تر است؛ با این حال جداسازی tenant ایجاد نمی‌کند.

## SSE

رویدادها به‌شکل `data: <JSON>` با object برابر `chat.completion.chunk` ارسال می‌شوند؛ ابتدا delta نقش assistant و سپس content یا reasoning_content می‌آید. chunk پایانی finish_reason دارد و stream با `data: [DONE]` بسته می‌شود. در include_usage، پیش از DONE یک chunk با choices خالی می‌آید.

اگر ابزار کلاینت فعال باشد، stream تا پایان parsing بافر می‌شود؛ این مسیر جریانیِ توکن‌به‌توکن واقعی نیست. خطای پس از آغاز header در بدنهٔ SSE به‌صورت error و سپس DONE می‌آید و دیگر امکان تغییر HTTP status اولیه وجود ندارد.

## functionها و state

نام function باید با `[A-Za-z0-9_-]{1,64}` مطابق باشد. تعریف پارامترها به مدل داده می‌شود اما schema validator اجرا نمی‌شود. call نام ناشناخته خطای server ایجاد می‌کند. required بدون هیچ call خطا می‌دهد؛ انتخاب named function، call با نام متفاوت را رد می‌کند ولی نبود همهٔ callها را الزاماً رد نمی‌کند.

سرور هیچ‌یک از این functionها را با registry محلی اجرا نمی‌کند. کلاینت مسئول اجرا و ارسال نتیجه است. تعریف tools در Map حافظه‌ای به ازای نشست cache می‌شود؛ با restart از دست می‌رود. برای جزئیات قالب به [tool-protocol.md](tool-protocol.md) مراجعه کنید.

تمام completionها از یک صف عبور می‌کنند و از state API مشترک استفاده می‌کنند. تغییر token state فعال را reset می‌کند؛ انتخاب و ساخت prompt/cache پیش از اجرای صف انجام می‌شود. این معماری برای سرویس چندکاربره و ایزوله طراحی نشده است. قطع اتصال کلاینت ارسال خروجی را متوقف می‌کند؛ تلاش توقف upstream در finally پس از await نتیجه انجام می‌شود، نه الزاماً همان لحظهٔ disconnect.

## سازگاری و خطا

| قابلیت مورد انتظار OpenAI | وضعیت در این پروژه |
| --- | --- |
| envelope completion و SSE | پشتیبانی محدود |
| reasoning_content | افزونهٔ غیرهمسان با همهٔ کلاینت‌ها |
| replay همهٔ messages و نقش system | پشتیبانی نمی‌شود |
| انتخاب واقعی مدل با model | تقریبی؛ بیشتر برچسب و thinking |
| function calling | تبدیل متنی و validation محدود، اجرای سمت کلاینت |
| چند choice، تصویر، کنترل sampling | پشتیبانی نشده یا ارسال نمی‌شود |
| نشست مستقل هر درخواست/کاربر | تضمین نشده |
| احراز هویت جداگانهٔ API محلی | پیاده نشده |

envelope خطا شامل `error.message`, `type`, `param`, `code` است. ورودی messages یا JSON نامعتبر 400، نبود مدل/مسیر 404، token نامعتبر 401 و خطاهای عمومی معمولاً 500 می‌شوند. وقفهٔ نتیجه در پاسخ non-stream می‌تواند 504 شود؛ این timeout به معنی وجود deadline عمومی درخواست نیست. متن خطای token با عبارت API key سازگار با OpenAI تولید می‌شود، اما اعتبارنامهٔ واقعی پروژه توکن وب است.

توکن محیط بر Bearer اولویت دارد؛ CORS wildcard و allowance شبکهٔ خصوصی به معنی حفاظت دسترسی نیست. پیش از تغییر bind از loopback، [امنیت](../explanation/security-and-privacy.md) و [راهنمای سرور](../guides/http-server.md) را بخوانید.