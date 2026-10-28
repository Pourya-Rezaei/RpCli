# گزارش اعتبارسنجی مستندات

تاریخ بررسی: `2026-10-28`؛ نسخهٔ مبنا: `0.10.1`. این گزارش به تهیهٔ مجموعهٔ `doc/` مربوط است، نه تأیید انتشار یا آزمون کامل محصول.

## دامنه و روش

کد ورودی، عامل، ابزارها، transport وب، سرور، رابط ترمینال، تنظیمات، promptها و تست‌های موجود در تحلیل لحاظ شدند. [نقشهٔ پوشش](../reference/source-map.md) فایل‌های تحت Git را به محل توضیح وصل می‌کند. شواهد هر رفتار در صفحهٔ تخصصی لینک شده‌اند.

فقط فایل Markdown به `doc/` اضافه شده است؛ build output، کد برنامه، lockfile، اسناد ریشه و config شخصی ویرایش نشده‌اند. اجرای فرمان‌های نمونهٔ دارای اثر جانبی یا نیازمند حساب DeepSeek بخشی از اعتبارسنجی نبوده است.

## کنترل‌های اسناد

| کنترل | معیار |
| --- | --- |
| مقصد پیوندها | همهٔ لینک‌های نسبی فایل به مقصد موجود resolve شوند |
| حصار کد | بلوک‌های fenced بسته شوند |
| ناوبری | همهٔ صفحات Markdown مجموعه از نمایه قابل دسترس باشند |
| پوشش مخزن | هر فایل در موجودی اولیهٔ Git در source-map مرجع داشته باشد |
| محدودیت دامنه | diff کد/فایل‌های tracked قبلی خالی و تغییرات جدید محدود به doc باشد |
| خوانایی | هدف، ساختار موضوعی، مثال جدا و ارجاع به مرجع در بازبینی متنی کنترل شود |
| اعتبار ادعا | رفتار استخراج‌شده، ریسک ایستا و رفتار runtime آزمایش‌نشده مخلوط نشوند |

کنترل خودکار مقصد فایل، رندر بصری Markdown و درستی معنایی همهٔ جملات را اثبات نمی‌کند. پیوندهای این مجموعه عمداً به فایل‌اند و به heading یا line anchor شکننده تکیه ندارند. نمودارهای Mermaid متن‌محورند؛ رندر در تمام viewerها در این کار آزمایش نشده است.

## محیط و محدودیت build

Node موجود در محیط بررسی `v24.17.0` است. فرمان `pnpm exec tsc --noEmit` تلاش شد، اما shell با `pnpm: command not found` متوقف شد؛ بنابراین هیچ نتیجهٔ TypeScript از آن حاصل نشد.

بررسی بعدی نشان داد `node_modules`، compiler محلی TypeScript و AVA محلی نیز وجود ندارند. نصب dependency، دانلود package manager یا تماس شبکه‌ای برای رفع این پیش‌نیاز انجام نشد. نتیجه:

- build و typecheck اجراشده و موفق گزارش نمی‌شوند.
- lint، Prettier و AVA اجرا نشده‌اند.
- شمار ۱۲ case از تعریف تست‌ها استخراج شده؛ به معنی ۱۲ تست پاس‌شده نیست.
- از نبود خطای editor نمی‌توان موفقیت build را نتیجه گرفت.

[package.json](../../package.json) مسیر اجرای `pnpm build` و `pnpm test` را تعریف می‌کند. برای انجام این gateها در محیط آماده، [آماده‌سازی توسعه](setup-and-scripts.md) و [تست‌ها](testing.md) را دنبال کنید. [فایل ignore](../../.prettierignore) Markdown را کنار می‌گذارد؛ موفقیت pnpm test نیز جای کنترل پیوندها را نمی‌گیرد.

## رفتارهای اجرا نشده

توکن شخصی خوانده یا ثبت نشد؛ حساب remote، تولید پاسخ، سلامت واقعی DeepSeek، deletion نشست، function call، تغییر مدل، resume واقعی، اجرای shell، elevation و سناریوهای symlink آزمایش نشدند. سناریوهای امنیتی [فهرست محدودیت‌ها](../reference/known-limitations.md) استنتاج از کدند، نه exploit اجراشده.

## ثبت نتیجهٔ نهایی

| نتیجهٔ کنترل | مقدار در این بازبینی |
| --- | --- |
| صفحهٔ Markdown | ۲۸ |
| پیوند نسبی فایل کنترل‌شده | ۵۸۶؛ بدون مقصد شکسته |
| پوشش موجودی اولیهٔ Git | ۸۵ از ۸۵ فایل |
| صفحهٔ غیرقابل دسترس از نمایه | صفر |
| حصار کد بازمانده | صفر |
| تغییر فایل tracked پیشین | صفر |
| فایل جدید بیرون doc | صفر |
| خطای گزارش‌شدهٔ editor در doc | صفر؛ جایگزین compiler نیست |

این کنترل‌ها با Node و filesystem محلی انجام شدند، بدون نصب کتابخانه. بازبینی متنی نیز مرجع‌های حساس CLI، function parsing، تنظیمات، mode و محدودیت‌های انتقال را با منابع تطبیق داد؛ این بازبینی تضمین رسمی صحت تمام جمله‌ها نیست.

## بازاجرای کنترل ساختاری

از ریشهٔ repository، این بررسی فقط فایل‌ها را می‌خواند و در صورت خطای پیوند، حصار، ناوبری یا پوشش exception می‌دهد. به pnpm یا حساب DeepSeek نیاز ندارد:

```sh
node --input-type=module <<'NODE'
import fs from 'node:fs'
import path from 'node:path'
import {execFileSync} from 'node:child_process'

const pages = new Map()
function scan(directory) {
	for (const entry of fs.readdirSync(directory, {withFileTypes: true})) {
		const file = path.join(directory, entry.name)
		if (entry.isDirectory()) scan(file)
		else if (file.endsWith('.md')) pages.set(file, fs.readFileSync(file, 'utf8'))
	}
}
scan('doc')
const edges = new Map()
for (const [file, text] of pages) {
	if ((text.match(/^```/gm) ?? []).length % 2) throw Error(`Fence: ${file}`)
	const targets = []
	for (const match of text.matchAll(/\]\(([^)]+)\)/g)) {
		if (/^(https?:|mailto:)/.test(match[1])) continue
		const [relative, anchor] = match[1].split('#')
		if (anchor) throw Error(`Anchor needs separate validation: ${file}`)
		const target = path.normalize(path.join(path.dirname(file), decodeURI(relative)))
		if (!fs.existsSync(target)) throw Error(`Link: ${file} -> ${target}`)
		targets.push(target)
	}
	edges.set(file, targets)
}
const visited = new Set()
function visit(file) {
	if (visited.has(file)) return
	visited.add(file)
	for (const target of edges.get(file) ?? []) if (pages.has(target)) visit(target)
}
visit('doc/README.md')
for (const file of pages.keys()) if (!visited.has(file)) throw Error(`Orphan: ${file}`)
const tracked = execFileSync('git', ['ls-files', '-z'], {encoding: 'utf8'}).split('\0').filter(Boolean)
const inventory = new Set(edges.get('doc/reference/source-map.md'))
for (const file of tracked) {
	if (!file.startsWith('doc/') && !inventory.has(file)) throw Error(`Uncovered: ${file}`)
}
console.log({pages: pages.size, links: [...edges.values()].flat().length, status: 'OK'})
NODE
```

این checker برای لینک‌های فایل معمولی و حصارهای همین مجموعه طراحی شده، نه parser کامل Markdown. افزودن anchor، HTML، image یا نوع حصار تازه نیاز به گسترش بررسی متناسب دارد. کنترل diff دامنهٔ تغییرات و بازبینی معنایی متن همچنان جدا لازم است.

تاریخ و نسخه متعلق به snapshot تحلیل است؛ مستندات تضمین همگامی خودکار با نسخه‌های آینده نیستند. [راهنمای نگهداری](documentation-maintenance.md) معیار بازبینی بعدی را مشخص می‌کند.