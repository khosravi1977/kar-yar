# کار یار — نصب PWA روی VPS و گوشی

این بسته، نسخهٔ PWA برنامهٔ «کار یار» است: با اجرا روی یک سرور با **HTTPS**، می‌توانید آن را مثل یک اپلیکیشن واقعی روی صفحهٔ اصلی گوشی نصب کنید. همهٔ داده‌ها (کارها، یادداشت‌ها، سوابق) در حافظهٔ محلی **همان گوشی** (localStorage) ذخیره می‌شود و برنامه پس از نصب **آفلاین** هم باز می‌شود.

## محتویات بسته
| فایل | نقش |
|---|---|
| `index.html` | خود برنامه (بدون تغییر نسبت به نسخهٔ ۳.۱، فقط افزودن PWA) |
| `manifest.webmanifest` | شناسنامهٔ اپ (نام، آیکون، رنگ، حالت standalone) |
| `sw.js` | service worker؛ کش آفلاین |
| `icons/` | آیکون‌های برنامه (عادی، maskable، favicon، آیکون iOS) |

---

## گام ۱ — آپلود روی VPS

فایل‌ها را در یک پوشه روی سرور بگذارید، مثلاً `/var/www/kar-yar`:

```bash
# از کامپیوتر خودتان (یا هر مسیر دلخواه روی سرور آپلود کنید):
scp -r kar-yar-pwa/* root@IP_SERVER:/var/www/kar-yar/
```

یا اگر پنل مدیریت (مثلaaPanel/DirectAdmin) دارید، محتویات zip را در پوشهٔ سایت آپلود و Extract کنید.

## گام ۲ — تنظیم nginx

فایل `/etc/nginx/sites-available/kar-yar` را بسازید:

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN.com;      # یا IP سرور
    root /var/www/kar-yar;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # نوع فایل مانیفست
    include mime.types;
    types {
        application/manifest+json webmanifest;
    }
}
```

```bash
ln -s /etc/nginx/sites-available/kar-yar /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

## گام ۳ — فعال‌سازی HTTPS (الزامی)

مرورگرها اجازهٔ نصب PWA و ثبت service worker را **فقط روی HTTPS** (یا localhost) می‌دهند. با Certbot رایگان است:

```bash
apt install certbot python3-certbot-nginx    # اوبونتو/دبیان
certbot --nginx -d YOUR_DOMAIN.com
```

پس از این مرحله سایت روی `https://YOUR_DOMAIN.com` بالا است و آمادهٔ نصب PWA می‌شود.

> اگر هاست‌تان https آماده دارد (مثلاً با Cloudflare)، فقط فایل‌ها را در پوشهٔ سایت بگذارید — همین کافی است.

## گام ۴ — نصب روی گوشی

**اندروید (Chrome):**
1. آدرس سایت را در Chrome باز کنید.
2. منوی ⋮ → **Add to Home screen** (افزودن به صفحهٔ اصلی).
3. تأیید کنید؛ آیکون «کار یار» مثل یک اپ واقعی روی صفحه می‌نشیند و تمام‌صفحه باز می‌شود.

**آیفون / آیپد (Safari):**
1. آدرس را در Safari باز کنید (فقط Safari!).
2. دکمهٔ Share ⬆️ → **Add to Home Screen**.
3. تأیید کنید.

## نکته‌ها
- داده‌های هر دستگاه مستقل است (روی گوشی جدا ذخیره می‌شود، روی کامپیوتر جدا). برای انتقال، از دکمه‌های «خروجی پشتیبان / بازیابی» داخل برنامه استفاده کنید.
- یادآوری ساعت‌دار هم کار می‌کند، اما اعلان در پس‌زمینهٔ کاملاً بسته به مجوز اعلان مرورگر نیاز دارد؛ بعد از نصب، دکمهٔ «فعال‌سازی یادآوری» را بزنید و اجازه بدهید.
- بعد از هر تغییر در فایل‌ها، شمارهٔ `CACHE_NAME` در `sw.js` را عوض کنید (مثلاً v2) تا نسخهٔ جدید روی گوشی‌ها اعمال شود.
