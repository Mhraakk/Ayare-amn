# نمونه‌های typesafe-local برای عیار امن

JSONها را با سرور محلی تست کن:

```bash
curl -s localhost:8000/v1/systemone \
  -H 'content-type: application/json' \
  -d @prompts/typesafe-local/scam-signal.example.json
```

راهنما: [`docs/TYPESAFE_LOCAL.md`](../../docs/TYPESAFE_LOCAL.md)
