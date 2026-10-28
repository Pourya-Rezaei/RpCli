# پروتکل فراخوانی ابزار

نسخه `0.10.1`، بررسی ایستا در `2026-10-28`. این صفحه قالب پیام را توضیح می‌دهد؛ دستور اجرای واقعی نیست.

## دو پروتکل مستقل

| مسیر | قالب مدل | اجراکننده |
| --- | --- | --- |
| عامل محلی | `tool_calls / invoke / parameter` | registry محلی از طریق executor |
| functionهای HTTP | `function_call / argument` | کلاینت HTTP؛ سرور فقط ترجمه می‌کند |

این دو قالب قابل جایگزینی نیستند. مرجع نام‌ها و ورودی ابزار محلی در [local-tools.md](local-tools.md) است.

## قالب محلی

مالک: `parseToolCalls` در [parse.ts](../../source/tools/parse.ts)، تعریف نوع در [types.ts](../../source/tools/types.ts).

```xml
<tool_calls>
<invoke name="read_file">
<parameter name="path">README.md</parameter>
<parameter name="offset">1</parameter>
<parameter name="limit">20</parameter>
</invoke>
</tool_calls>
```

parser مبتنی بر regex است؛ XML parser عمومی با پشتیبانی کامل escaping، nesting و schema نیست. مقدار parameter ابتدا رشته است؛ helperهای `stringArgument`، `textArgument` و `positiveIntegerArgument` در [arguments.ts](../../source/tools/arguments.ts) تفسیر و اعتبارسنجی متناسب با ابزار را انجام می‌دهند. whitespace و newline ورودی باید با قواعد helper مربوط مقایسه شود؛ همهٔ ورودی‌ها trim یکسان ندارند.

در این نسخه invoke بدون parameter به‌عنوان فراخوانی معتبر پذیرفته نمی‌شود. بنابراین ابزارهایی مانند `todo_list()` و `todo_clear()` با وجود تعریف بدون پارامتر در registry، از این مسیر markup ناسازگاری دارند. این محدودیت parser است، نه نبود ابزار در registry.

## نرمال‌سازی و حلقهٔ عامل

[toolCallMarkup.ts](../../source/core/toolCallMarkup.ts) قالب DSML بالادستی را به فرم محلی تبدیل و تکه‌های ناقص را برای نمایش کنترل می‌کند. این نرمال‌سازی با parsing و اجرای مجاز یکسان نیست.

`getAIResponse` در [agent.ts](../../source/actions/agent.ts) ابتدا answer و در نبود call در answer، thinking را برای ابزار بررسی می‌کند. خروجی مخفی‌شده در UI ممکن است در این مسیر همچنان ورودی تصمیم عامل باشد.

1. پاسخ مدل دریافت و markup نرمال می‌شود.
2. callها استخراج یا خطای parse شناسایی می‌شود.
3. executor با mode و callback تأیید اجرا می‌شود.
4. موفقیت و خطا در بلوک‌های `tool_result` به مدل فرستاده می‌شوند.
5. چرخه تا پاسخ نهایی، توقف یا سقف ۱۰ دور ابزار ادامه پیدا می‌کند.

خطای parse نیز از بودجهٔ دور ابزار مصرف می‌کند و پیام اصلاح قالب به مدل می‌دهد. در سقف، عامل پاسخ خلاصهٔ بدون ابزار درخواست می‌کند؛ این سقف با سقف ادامهٔ پاسخ ناقص در لایهٔ transport یکی نیست. نبود callback تأیید، مجوز پیش‌فرض برای عملیات ایجاد نمی‌کند.

## قالب function در HTTP

مالک: `withToolInstructions` و `parseClientToolCalls` در [request.ts](../../source/server/request.ts).

```xml
<function_call name="lookup_document">
<argument name="document_id">"example"</argument>
<argument name="limit">3</argument>
</function_call>
```

مقدار argument ابتدا با `JSON.parse` امتحان و در صورت شکست به رشته تبدیل می‌شود. قالب JSON object نیز برای بدنهٔ function پذیرفته می‌شود؛ parser مسیر fallback برای رشتهٔ triple-quoted دارد. خروجی بیرونی، call با id تولیدشده و `function.arguments` به‌شکل رشتهٔ JSON است.

نام function باید در تعریف ابزارهای درخواست یا cache نشست باشد. JSON Schema پارامترهای ارسال‌شده به‌عنوان قرارداد اعتبارسنجی runtime اعمال نمی‌شود. `tool_call_id` نتیجهٔ کلاینت نیز به call ثبت‌شده تطبیق داده نمی‌شود. محدودیت انتخاب named tool در [known-limitations.md](known-limitations.md) ثبت شده است.

در پاسخ بعدی کلاینت، آخرین پیام با role برابر `tool` توسط `buildCompletionPrompt` در `tool_result` پیچیده می‌شود. ارسال چند نتیجه در messages به معنی پردازش همهٔ آن‌ها نیست؛ تنها آخرین مورد مبنای prompt است.