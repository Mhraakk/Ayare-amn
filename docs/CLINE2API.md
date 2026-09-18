# cline2api-workers و عیار امن

منبع: [pingmike2/cline2api-workers](https://github.com/pingmike2/cline2api-workers)

این پروژه مدل‌های رایگان Cline را به API سازگار با OpenAI روی Cloudflare Workers (یا Vercel) تبدیل می‌کند.

## استقرار یک‌باره

1. با `cline_oauth.py` یا Action آپ‌استریم یک `refreshToken` بگیر.
2. `worker.js` را روی Cloudflare Workers دیپلوی کن (راهنمای README آپ‌استریم).
3. Secretها: `CLINE_REFRESH_TOKEN` و `API_KEY`.
4. Base URL را یادداشت کن: `https://cline2api.<subdomain>.workers.dev`

## استفاده با Astra / کلاینت OpenAI

```text
Base URL:  https://cline2api.<subdomain>.workers.dev/v1
API Key:   همان API_KEY ورکر
Model:     cline-free/deepseek-v4.1-flash
```

```bash
curl https://cline2api.<subdomain>.workers.dev/v1/chat/completions \
  -H "Authorization: Bearer sk-..." \
  -H "Content-Type: application/json" \
  -d '{"model":"cline-free/deepseek-v4.1-flash","messages":[{"role":"user","content":"سلام"}]}'
```

برای عیار امن: در تنظیمات Astra / هر SDK سازگار با OpenAI همین Base URL و Key را بگذار. زرین (`gold-terminal-pro`) هم با envهای `CLINE2API_*` به همین ورکر وصل می‌شود.
