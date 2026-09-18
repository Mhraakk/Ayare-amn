# UI sketch — مسیر امن → وکیل

Markdown UX sketch for a Persian RTL product path: client uploads an encrypted package, shares a link, lawyer receives in an inbox, and every step is audited. Inspired by [farsi-ui/ui](https://github.com/farsi-ui/ui) layout patterns and SecuShare-style share links. **Not** a React app — docs only.

## Wireframe sections (RTL)

```
┌─────────────────────────────────────────────────────────┐
│  عیار امن · مسیر امن → وکیل                    [حساب] │
├─────────────────────────────────────────────────────────┤
│  ۱) بارگذاری رمزنگاری‌شده                              │
│     [ انتخاب فایل ]  الگوریتم: AES-GCM · کلید کلاینت  │
│     وضعیت: آماده / در حال رمز / آمادهٔ اشتراک           │
├─────────────────────────────────────────────────────────┤
│  ۲) لینک اشتراک امن                                     │
│     https://…/s/••••  [ کپی ]  انقضا: ۲۴س · یک‌بار مصرف│
├─────────────────────────────────────────────────────────┤
│  ۳) اینباکس وکیل                                        │
│     ● پروندهٔ جدید · ۱۴۰۵/۰۶/۲۷ · در انتظار پذیرش      │
│     ○ مشاهده‌شده · یادداشت داخلی                        │
├─────────────────────────────────────────────────────────┤
│  ۴) ممیزی (Audit)                                       │
│     زمان | کنش | نقش | IP/دستگاه | نتیجه                 │
│     …                                                   │
└─────────────────────────────────────────────────────────┘
```

## Acceptance criteria

1. Root document `lang="fa"` `dir="rtl"`; labels and primary CTA use logical spacing (`ps`/`pe`, `start`/`end`).
2. Upload encrypts **before** network leave; plaintext never stored server-side by default.
3. Share link is time-bounded and preferably single-use; revoke control for the client.
4. Lawyer inbox lists only packages the lawyer is authorized to open; open events write audit rows.
5. Audit trail is append-only (or equivalent) and visible to client + lawyer roles with least privilege.
6. UI copy stays Persian-first; optional English secondary strings later.
7. No AGPL jewellery ERP code is vendored into the product without legal review.

## References

- farsi-ui: https://github.com/farsi-ui/ui  
- SecuShare (pattern inspiration): search GitHub `SecuShare`  
- OSS table: [`OSS_INSPIRATION.md`](./OSS_INSPIRATION.md)
