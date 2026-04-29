# نسخه Netlify پروژه Vercel

این پروژه معادل Netlify فایل‌های Vercel آپلودشده است. از Netlify Edge Functions استفاده می‌کند و همه مسیرها را با `path = "/*"` به فایل `netlify/edge-functions/relay.js` می‌فرستد.

> فقط برای سرویس‌ها، دامنه‌ها و upstreamهایی استفاده کن که مالکشان هستی یا مجوز استفاده از آن‌ها را داری.

## ساختار پروژه

```txt
netlify-xhttp-relay/
├─ netlify.toml
├─ package.json
├─ netlify/
│  └─ edge-functions/
│     └─ relay.js
└─ public/
   └─ index.html
```

## معادل‌سازی فایل‌های Vercel به Netlify

### Vercel

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/api/index" }
  ]
}
```

### Netlify

```toml
[[edge_functions]]
  function = "relay"
  path = "/*"
```

یعنی هر مسیری که روی دامنه Netlify باز شود، اول وارد Edge Function به نام `relay` می‌شود.

---

# آموزش ۰ تا ۱۰۰ دیپلوی روی Netlify

## روش ساده با GitHub

### 1) فایل zip را باز کن

فایل `netlify-xhttp-relay.zip` را unzip کن.

### 2) یک ریپوی GitHub بساز

در GitHub یک repository جدید بساز، مثلاً:

```txt
netlify-xhttp-relay
```

### 3) پروژه را push کن

داخل پوشه پروژه این دستورها را بزن:

```bash
git init
git add .
git commit -m "Convert Vercel relay to Netlify Edge Function"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/netlify-xhttp-relay.git
git push -u origin main
```

`YOUR_USERNAME` را با یوزرنیم GitHub خودت عوض کن.

### 4) پروژه را در Netlify بساز

در Netlify:

```txt
Add new site → Import an existing project → GitHub → انتخاب repo
```

Netlify خودش `netlify.toml` را می‌خواند. تنظیمات باید این باشد:

```txt
Build command: npm run build
Publish directory: public
```

### 5) Environment Variable را ست کن

در Netlify برو به:

```txt
Site configuration → Environment variables → Add variable
```

این متغیر را اضافه کن:

```txt
TARGET_DOMAIN=https://your-upstream-domain.com
```

نکته مهم: آخر دامنه `/` نگذار. درست:

```txt
https://example.com
```

غلط:

```txt
https://example.com/
```

البته کد خودش `/` آخر را حذف می‌کند، ولی بهتر است تمیز وارد شود.

### 6) Deploy کن

بعد از اضافه کردن env، برو به:

```txt
Deploys → Trigger deploy → Deploy site
```

### 7) تست کن

بعد از deploy، آدرس Netlify را باز کن:

```txt
https://YOUR-SITE.netlify.app/
```

اگر `TARGET_DOMAIN` را نگذاشته باشی، این خطا را می‌بینی:

```txt
Misconfigured: TARGET_DOMAIN is not set
```

اگر upstream در دسترس نباشد، این خطا می‌آید:

```txt
Bad Gateway: Relay Failed
```

---

# روش CLI بدون GitHub

اگر نمی‌خواهی GitHub استفاده کنی:

```bash
npm install
npx netlify login
npx netlify init
npx netlify env:set TARGET_DOMAIN https://your-upstream-domain.com
npx netlify deploy --prod
```

برای تست لوکال:

```bash
npx netlify dev
```

اگر لوکال تست می‌کنی، یک فایل `.env` بساز:

```env
TARGET_DOMAIN=https://your-upstream-domain.com
```

---

# فایل‌های مهم

## `netlify.toml`

```toml
[build]
  command = "npm run build"
  publish = "public"

[[edge_functions]]
  function = "relay"
  path = "/*"
```

## `netlify/edge-functions/relay.js`

این فایل همان منطق `api/index.js` در Vercel است، فقط برای Netlify Edge Function بازنویسی شده.

---

# عیب‌یابی سریع

## خطای `TARGET_DOMAIN is not set`

یعنی env را در Netlify اضافه نکردی یا بعد از اضافه کردنش redeploy نکردی.

## خطای 502

یعنی Netlify نتوانسته به `TARGET_DOMAIN` وصل شود، یا upstream جواب درست نداده.

## مسیرها درست منتقل نمی‌شوند

مطمئن شو در `netlify.toml` این بخش هست:

```toml
[[edge_functions]]
  function = "relay"
  path = "/*"
```

## بعد از تغییر env هنوز کار نمی‌کند

حتماً دوباره deploy بگیر:

```txt
Deploys → Trigger deploy → Deploy site
```
