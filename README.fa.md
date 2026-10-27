# Manotohubir News Worker — OpenRouter Multi-Model Edition

نسخهٔ آماده برای GitHub + Cloudflare Workers. خبرهای چندزبانه درباره ایران، اسرائیل، آمریکا و تحولات مرتبط را هر ۵ دقیقه جمع می‌کند، خبرهای تکراری را حذف می‌کند، با OpenRouter به فارسی خلاصه می‌کند و خبرهای واجد شرایط را در `@Manotohubir` منتشر می‌کند. پنل ادمین تلگرام نیز از طریق polling هر دقیقه بررسی می‌شود.

## چه مدل‌هایی تنظیم شده‌اند؟

ترتیب پیش‌فرض fallback:
1. `nvidia/nemotron-3-ultra-550b-a55b:free`
2. `nvidia/nemotron-3.5-lightning:free`
3. `nvidia/nemotron-3.5-super-vl:free`
4. `poolside/laguna-s-2.1:free`

اگر مدل اول خطا بدهد، محدودیت بخورد یا پاسخ نامعتبر بدهد، مدل بعدی امتحان می‌شود. مدل Laguna S 2.1 در صفحهٔ OpenRouter اعلام شده که از ۳۱ اکتبر ۲۰۲۶ از حالت رایگان خارج می‌شود؛ پس آن را فقط fallback موقت در نظر بگیر. شناسه و وضعیت رایگان مدل‌ها ممکن است تغییر کند.

## راه‌اندازی اولیه

### 1) آپلود در GitHub
- ZIP را استخراج کن.
- محتویات پوشه `manotohubir-worker` را در ریشهٔ repository قرار بده؛ `wrangler.toml` باید در ریشه باشد.
- Branch اصلی را `main` نگه دار.

### 2) اتصال GitHub به Cloudflare
- Cloudflare Dashboard → Workers & Pages → Create application → Workers → Connect to Git.
- Repository را انتخاب کن و Root directory را ریشه بگذار.
- Build command: `npm install && npm run typecheck`
- Deploy command: `npx wrangler deploy`
- Save and Deploy.
- پس از آن Push روی `main` به‌طور خودکار Deploy می‌شود.

### 3) فقط سه Secret اولیه در Cloudflare
در Worker `manotohubir-news` → Settings → Variables and Secrets این Secretها را بساز:

- `TELEGRAM_BOT_TOKEN` — توکن ربات از `@BotFather`
- `ADMIN_TELEGRAM_ID` — شناسهٔ عددی حساب تلگرام خودت؛ می‌توانی آن را از `@userinfobot` بگیری
- `CONFIG_ENCRYPTION_KEY` — یک رمز تصادفی طولانی، حداقل ۳۲ کاراکتر. نمونهٔ تولید در ترمینال: `openssl rand -hex 32`

این سه مورد فقط یک‌بار لازم‌اند. هیچ کلیدی را در GitHub یا `wrangler.toml` نگذار. `OPENROUTER_API_KEY` در Secretهای Cloudflare لازم نیست؛ آن را بعد از استقرار، از چت خصوصی پنل ادمین ربات ثبت می‌کنی و برنامه آن را با AES-GCM رمزگذاری‌شده در Durable Object ذخیره می‌کند. اگر `CONFIG_ENCRYPTION_KEY` را بعداً عوض کنی، کلید ذخیره‌شده قابل رمزگشایی نخواهد بود؛ ابتدا کلید API را از ربات حذف کن، سپس رمز را عوض و دوباره API key را ثبت کن.

### 4) ادمین کردن ربات در کانال
- همان ربات را به `@Manotohubir` اضافه کن.
- به آن Admin و مجوز ارسال پیام بده.
- پیشنهاد می‌شود از یک ربات اختصاصی برای این پروژه استفاده کنی. سیستم از Telegram polling استفاده می‌کند و هنگام اولین اجرا، webhook قبلی همین ربات را حذف می‌کند تا `getUpdates` کار کند.

## ثبت API از داخل بات ادمین
پس از Deploy، به ربات در چت خصوصی پیام بده:

- `/start` — راهنمای پنل
- `/status` — وضعیت ثبت API، مدل فعال و آخرین مدل موفق
- `/models` — فهرست چهار مدل و ترتیب fallback
- `/setmodel 1` تا `/setmodel 4` — تغییر مدل اصلی؛ سه مدل دیگر پشتیبان می‌مانند
- `/setkey YOUR_OPENROUTER_KEY` — ثبت یا جایگزینی کلید OpenRouter
- `/delkey` — حذف کلید ذخیره‌شده
- `/test` — آزمایش مدل اصلی و API
- `/myid` — نمایش شناسهٔ عددی تلگرام (این دستور برای شناسایی اولیه هم کار می‌کند)

**ثبت API:** کلید OpenRouter را از پنل https://openrouter.ai/settings/keys بساز و در چت خصوصی با قالب `/setkey sk-or-v1-...` بفرست. برنامه تلاش می‌کند پیام حاوی کلید را حذف کند؛ با این حال، به‌دلیل اینکه کلید را در تلگرام ارسال می‌کنی، فقط در چت خصوصی ربات بفرست و اگر پیام حذف نشد، خودت آن را حذف کن. کلید در Durable Object با AES-GCM رمزگذاری می‌شود و فقط Worker می‌تواند آن را با `CONFIG_ENCRYPTION_KEY` باز کند.

## زمان‌بندی
- Cron هر دقیقه برای دریافت دستورات ادمین
- پایش فیدها و پردازش خبر هر ۵ دقیقه
- سقف هر اجرا: ۳۵ ورودی و ۴ پست
- وضعیت سرویس: URL Worker + `/health`
- گزارش خطاها: Cloudflare Dashboard → Worker → Logs / Observability

## محدودیت‌های مهم
- «قیمت صفر» مدل‌ها به معنی دسترسی نامحدود نیست؛ OpenRouter می‌تواند نرخ درخواست یا دسترسی هر مدل رایگان را تغییر دهد.
- هر چهار مدل از یک کلید OpenRouter استفاده می‌کنند؛ fallback یعنی تغییر مدل، نه افزایش سهمیهٔ کلی حساب.
- چند منبع بودن لزوماً استقلال منابع را ثابت نمی‌کند. خروجی مدل راستی‌آزمایی قطعی نیست؛ خبرهای جنگی، تلفات و ادعاهای نظامی ممکن است نیازمند بررسی انسانی باشند.
- Laguna S 2.1 در وضعیت فعلی موقت است و از ۳۱ اکتبر ۲۰۲۶ قرار است از حالت رایگان خارج شود.
- اگر ربات قبلاً webhook برای سرویس دیگری داشته، اجرای این Worker آن webhook را حذف می‌کند. از ربات جداگانه استفاده کن.


## رفع خطای نصب وابستگی‌ها (v1.2.1)

نسخهٔ `@cloudflare/workers-types` به نسخهٔ موجود `^5.20261027.1` به‌روزرسانی شد؛ نسخهٔ قبلی `^4.20261001.0` در رجیستری در دسترس نبود.


### اصلاحات نسخه 1.2.2
- تاریخ سازگاری Cloudflare به `2026-10-27` تغییر کرد تا تاریخ آینده نباشد.
- نام Worker در `wrangler.toml` با نام مورد انتظار Cloudflare Builds یعنی `manotohubnews` یکسان شد.
