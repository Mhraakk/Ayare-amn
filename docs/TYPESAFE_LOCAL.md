# typesafe-local و عیار امن

منبع: [aabolfazl/typesafe-local](https://github.com/aabolfazl/typesafe-local)

سرور محلی System One: روی یک سند چند سؤال **تایپ‌شده** می‌پرسد و به‌جای متن آزاد، **توزیع احتمال کالیبره‌شده** برمی‌گرداند.

## محدودیت

- اجرای مدل با **MLX** فقط روی **Apple Silicon**
- روی Vercel اجرا نمی‌شود؛ برای Astra/اپراتور روی مک لوکال یا تونل

## اجرا

```bash
gh repo clone aabolfazl/typesafe-local
cd typesafe-local
python3 -m venv .venv
.venv/bin/pip install "mlx-lm>=0.20" fastapi uvicorn numpy
.venv/bin/python -m ots.server
```

- `GET /health`
- `GET /v1/models`
- `POST /v1/systemone` با `{ "state": ..., "questions": { ... } }`

انواع سؤال: `noul` | `choice` | `score` (فیلد `instructions` + در صورت نیاز `criteria`).

نمونه‌های عیار امن: [`prompts/typesafe-local/`](../prompts/typesafe-local/).

همراه پرامپت Astra از این برای تصمیم‌های ساخت‌یافته (ریسک پیام، مسیر تیکت، فوریت) استفاده کن؛ خروجی احتمال را قبل از عمل انسانی چک کن (`raw_mass` نزدیک ۱ = قابل اعتمادتر).
