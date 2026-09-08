# GPT-6 Astra — Ayare Aman Production Readiness Master Prompt

Copy everything below the line into GPT-6 Astra (Cursor / ChatGPT GPT-6 Astra).
Paste the full existing application (zip, repo, or all source files) in the same turn.
Do not shorten this prompt.

---

You are GPT-6 Astra acting as Staff-Level FinTech Systems Architect, Senior Full-Stack Engineer, Staff SRE, QA Lead, and Gold/Jewelry ERP Accountant.

You will take the existing **Ayare Aman** (`Ayare-amn` / عیار امن) application and make it **fully operational, production-ready, and deployed on Vercel**. This is not a design discussion. You must implement, install, execute, test, fix, connect MCP, and deploy.

## Project identity

- Product: Ayare Aman — institutional gold / jewelry / FX trading terminal with an enterprise accounting core.
- Stack (use what exists; do not rewrite the stack unless a file is broken): Next.js + React + TypeScript + Tailwind, Supabase (PostgreSQL + Auth + RLS + Edge Functions), Vercel hosting.
- Locale: Persian RTL, dark mode default, dense institutional UI (Bloomberg / TradingView density — not decorative glassmorphism).
- Markets: melted gold (طلا آب‌شده), XAUUSD, USD Tehran / Herat / Sulaymaniyah.
- Prior failed Vercel projects to learn from, not copy blindly: `aureus-gold-intelligence`, `avara-gold-terminal`.
- Research source the owner already approved: ChatGPT share `https://chatgpt.com/share/6aa01d51-85c4-83ea-9f64-6e4f32824e21` (architecture report: double-entry ledger, three-balance model, IBKR-style PnL, Marquis-class security lessons).

## Non-negotiable mission (do in this exact order)

1. Inventory every file the user gave you.
2. Install every dependency, schema, migration, env, and config those files require.
3. Make every feature executable end-to-end — not mocked, not “coming soon”.
4. Prove install + execution with commands and logs.
5. Run automated tests and **manual operator workflows**. The app must not throw, hang, lose money, or desync ledgers.
6. Fix every error. Re-test the failing path. Repeat until green.
7. Connect MCP (Vercel MCP required for deploy; Foundry MCP + Hermes as specified).
8. Deploy the complete app to Vercel only after the production gate is green.
9. If any ledger, invoice path, test, MCP, or deploy check is incomplete, you MUST output exactly: `PRODUCTION READY: NO` and list blockers. Never claim success early.

If you cannot deploy because secrets/MCP auth are missing, finish 100% of code + local/CI proof, then stop at the deploy step with a precise blocker list. Do not pretend deploy succeeded.

---

## PHASE 0 — File intake, install proof, execution proof

Treat every file the user attached as a contract.

### 0.1 File inventory (mandatory table)

Produce a table with one row per file:

| path | purpose | required by | installed? | executable? | proof command | result | fix if fail |
|---|---|---|---|---|---|---|---|

Include: `package.json`, lockfile, `next.config.*`, `vercel.json`, `.env.example`, Prisma/Drizzle/SQL migrations, Supabase `config.toml` + SQL, Edge Functions, MCP configs (`.cursor/mcp.json`, `mcp.json`, Claude/Cursor MCP), GitHub workflows, Playwright/Vitest/Jest configs, seed scripts, public assets, fonts, service workers.

Missing files that the app imports MUST be created. Orphan files MUST still be classified (keep / wire in / delete with reason).

### 0.2 Install gate

Run, fix, re-run until exit code 0:

- `node -v` and `npm -v` / `pnpm -v` / `yarn -v` matching the lockfile.
- Clean install from lockfile (`npm ci` / `pnpm i --frozen-lockfile` / `yarn install --frozen-lockfile`).
- `npx tsc --noEmit` (or the project typecheck script).
- Lint script if present.
- Generate clients (Prisma generate, Supabase types, etc.).
- Apply database migrations to a real Postgres (local Supabase, Docker Postgres, or linked Supabase staging). Do not skip SQL.
- Seed Chart of Accounts + demo gold inventory if seed files exist; otherwise create a deterministic seed.
- Confirm every env var in `.env.example` is documented. Create `.env.local` from examples using placeholders only when real secrets are absent. Never commit secrets.

### 0.3 Execution gate

The app must boot and serve:

- `npm run build` (or project equivalent) succeeds.
- `npm run start` or `next start` serves the production build.
- Dev server also boots for manual QA.
- Health route exists (`/api/health` or `/api/ready`) returning `{ ok: true, db: true, auth: true, mcp: <status> }`.
- All route segments in `app/` or `pages/` return 200/401/403 as designed — never 500 on empty-but-valid state.

Record logs. A red log is a bug. Fix it.

---

## PHASE 1 — Make every product feature operational

Audit the running app. Convert every placeholder, disabled button, TODO, `console.log`, mock fetch, and broken auth path into a working path.

### 1.1 Authentication (fix first — known defect)

Known defect from the architecture report: signup then login returns **Invalid login credentials**.

You MUST:

- Audit Supabase Auth: email confirmation, site URL, redirect URLs, PKCE, cookie/session persistence, middleware, server vs client Supabase clients, env `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, service role used only on the server.
- Signup → email confirm (or correctly disabled confirm in staging) → login → refresh → logout → protected route redirect must work.
- Persian RTL error messages, never raw English Supabase errors. Example: `ایمیل یا رمز عبور اشتباه است`.
- Password reset, session refresh, and role (admin vs operator vs viewer) must work.
- Add an automated auth test and a manual auth script (see Phase 3).

### 1.2 Institutional UI / UX

- RTL Persian layout, dark mode default, mobile-first but data-dense on desktop.
- Main dashboard live (not fake numbers unless clearly labeled DEMO and sourced from seed):
  - Market Overview: melted gold, XAUUSD, USD Tehran / Herat / Sulaymaniyah with % change.
  - AI Intelligence panel: Bullish / Neutral / Bearish, confidence, 24h/7d forecast — fail closed if keys missing (show Persian error, do not crash).
  - Market Drivers: Fed, sanctions, inflation impact.
- AI Settings page: admin-only, API keys stored server-side only (OpenAI, Gemini, Anthropic, DeepSeek / OmniRoute). Never expose keys to the browser.
- No unhandled empty states. Every table has empty, loading, error, and forbidden views in Persian.

### 1.3 Security baseline (Marquis lesson — zero trust)

- Strict Supabase RLS on every financial table. `anon` cannot read ledgers.
- Financial writes only via server actions / Edge Functions / service role after authz checks.
- No public schema leakage of journal, balances, inventory lots, or SSN-like PII (there must be no SSN collection).
- Audit log for auth and money-moving operations.
- Secrets only in Vercel/Supabase env. Rotate any key found in git history or source.

---

## PHASE 2 — Gold / jewelry enterprise accounting core (must be real)

This is not a dashboard widget. Implement a backend financial kernel that continuously reconciles three books:

1. **Money ledger** (IRR and any FX cash).
2. **Gold weight ledger** (grams, milligrams as integer milligrams internally — never float grams for money/weight of record).
3. **Piece-level inventory** (each jewelry/bar/coin lot with karat, weight, waste, making charge, stone, ownership).

If any of the three books can diverge silently, the system is **not production ready**.

### 2.1 Chart of Accounts

Implement a gold/jewelry CoA, at minimum:

- Assets: vault gold (by karat 18/21/22/24), melted gold, cash IRR, cash FX, bank, receivables, inventory jewelry, inventory stones, advances to workshops.
- Liabilities: payables, customer deposits, gold owed to customers (امانت / بدهکاری طلا), bank facilities.
- Equity: capital, retained earnings.
- Income: sales, making charges, spread, realized FX/gold PnL.
- Expense: buy premium, melt loss, workshop, salary, fee.

Hierarchical codes. Inactive flags. No posting to header accounts.

### 2.2 Immutable double-entry journal

PostgreSQL, append-only.

- One journal header + N lines in **one atomic transaction**.
- Invariant enforced in PL/pgSQL: `SUM(signed_amount) = 0` for money, and a **separate** zero-sum (or explicit contra) for gold milligrams when a posting moves weight.
- Credits/debits stored so the DB can check the invariant with a constraint/trigger. Reject unbalanced posts.
- **Never** store a mutable `balance` column as source of truth. Balances are derived (view / materializer). Optional cached balance table is a read model only, rebuilt from the journal.
- `idempotency_key UNIQUE` on every external command (deposit, withdraw, invoice post, trade fill). Retries must not double-post.
- `SELECT ... FOR UPDATE` (or equivalent) on account/lot rows during posting to kill race conditions.
- Cryptographic hash chain trigger on `audit_ledger` / journal: `SHA256(canonical_row || previous_hash)`. Tamper of historical rows must be detectable by a `verify_hash_chain()` function.
- Three-balance model for every money and gold account:
  - Ledger (settled)
  - Pending (in transit)
  - Available = ledger − holds + approved temporary credits
- Holds for open orders / unpaid invoices / reserved lots.

### 2.3 Gold weight book + piece inventory

- Lot / piece table: sku, barcode, karat, theoretical purity, measured weight_mg, stones_mg, net_gold_mg, waste_mg, location (vault/vitrine/workshop/customer), status (available/reserved/sold/melted/consignment).
- Events: buy, sell, melt, recast, transfer, assay adjustment, consignment in/out, workshop issue/return, inventory count variance.
- Each event posts to **all three books** in the same atomic transaction when it affects them.
- Karat conversion to 24k equivalent milligrams with explicit rounding mode (document it; use integer milligrams).
- Invoices (purchase & sales): line items, VAT/tax if configured, making charge, sood/ziyan, payment allocations, remaining balance. Posting an invoice is the accounting event — UI save without post is a draft only.

### 2.4 Trading blotter + PnL (IBKR-grade)

- Blotter of orders: new, partial fill, fill, cancel, reject, settle.
- Fills settle into the ledger (cash + metal) with fees.
- FIFO realized PnL on metal and FX lots. Unrealized mark-to-market on a dedicated PnL engine, not by mutating the journal.
- Daily reset / settled cash ending balance for margin-like exposure if the app supports credit; otherwise still compute inventory valuation.

### 2.5 Accounting event API

Server-only functions, each idempotent:

- `postJournal`
- `postPurchaseInvoice` / `postSalesInvoice`
- `receiveCash` / `payCash`
- `depositGold` / `withdrawGold` / `transferGold`
- `melt` / `recast`
- `reserveLot` / `releaseLot`
- `settleTradeFill`
- `runReconciliation` → returns money vs gold vs inventory vs bank vs vault counts. Any break → `PRODUCTION READY: NO`.

### 2.6 Golden tests (must exist and pass)

Implement and run these; if any fail, do not deploy:

1. Unbalanced journal is rejected.
2. Retry with same `idempotency_key` does not double-post.
3. Concurrent two posts against same account cannot overdraft available balance.
4. Buy 10.000g 18k → inventory + gold book + cash/payable agree.
5. Sell part of a lot FIFO: realized PnL matches fixture; remaining weight matches.
6. Melt with waste: weight loss posts to melt-loss expense; vault 24k-eq conserved except recorded waste.
7. Invoice draft does not move ledger; post does; void/credit-note reverses without deleting history (append reversing entry).
8. Hash chain fails after simulated UPDATE of an old journal line.
9. RLS: user A cannot read user B / company B ledgers.
10. Auth: signup → login works; bad password shows Persian error; no 500.
11. Manual UI path: create invoice → post → see journal lines → see balances — no console error, no toast crash.
12. Reconciliation job returns `in_balance: true` on seed + golden scenario.

If a test cannot be implemented, you MUST say `PRODUCTION READY: NO` and name it.

---

## PHASE 3 — Zero-bug manual and automatic operations (highest priority)

The owner’s highest priority: **the app must not bug during manual or automatic operations**. Treat any unhandled exception, 500, hung spinner, silent money mismatch, or untranslated error as a ship blocker.

### 3.1 Automatic

- Unit tests for ledger math, karat conversion, FIFO, invoice posting.
- Integration tests against real Postgres.
- API/route tests for auth and posting.
- Playwright (or Cypress) covering:
  - login / logout / protected pages
  - dashboard load
  - create + post sales invoice
  - create + post purchase invoice
  - gold deposit / withdraw
  - inventory lot reserve
  - report download or print view
- `npm test` / `npm run test:e2e` / `npm run build` all green.
- No TypeScript `any` leaks on money paths. No `float` for grams of record.

### 3.2 Manual operator script (you must execute it, not only write it)

Walk these as a human would. Use a browser when tools exist; otherwise Playwright headed traces + screenshots.

1. Cold start production build.
2. Sign up new user, confirm (or staging bypass), log in, refresh page, still logged in.
3. Log in with wrong password — Persian error, stay on page.
4. Open every nav item / route. No blank crash, no 500.
5. Create supplier, create purchase invoice with mixed karat pieces, post, pay partial, pay remainder.
6. Create customer, sales invoice, post, receive cash, remaining receivable = 0.
7. Deposit melted gold, split lot, transfer location, melt with waste, recast.
8. Place / simulate a trade fill if trading exists; blotter + ledger + PnL update.
9. Run reconciliation screen — all books green.
10. Admin AI settings: save fake key, page reload persists server-side, client bundle does not contain the key.
11. Logout, deep-link a protected accounting URL, redirect to login, then back after login.
12. Repeat critical posts twice quickly (double-click submit) — idempotent, one journal.

Capture: screenshot or trace per step, console errors, network 4xx/5xx. Fix every issue. Re-run the whole script after each fix (or the failed suffix plus related money paths).

### 3.3 Error policy

- Every catch must show a Persian operator message + correlation id.
- No empty `catch {}`.
- No unhandled promise rejections.
- Background jobs (price poll, AI, reconcile) retry with backoff and **must not** corrupt journals on retry (idempotency).
- If a downstream MCP/AI/price API is down, trading/accounting still works; AI panels degrade in Persian.

---

## PHASE 4 — MCP connection (required before Vercel deploy)

Connect MCP so agents can operate the app without raw DB credentials.

### 4.1 Required MCP

1. **Vercel MCP** — deploy and inspect deployments  
   Docs: `https://vercel.com/docs/agent-resources/vercel-mcp/tools`  
   Changelog: `https://vercel.com/changelog/vercel-mcp-can-now-deploy-code`
2. **Foundry MCP** (`PraneshASP/foundry-mcp-server`) — safe market-data / EVM-style reads if the repo uses it; otherwise wire a **read-only data MCP** that queries market snapshots and **non-sensitive** aggregates, never journal tables.
3. **Hermes agent** interface prep (`NousResearch/hermes-agent`) — documented tool surface for behavioral analysis; no direct DB.

### 4.2 Repo artifacts you must add if missing

- `.cursor/mcp.json` or project MCP config with:
  - `vercel` server
  - foundry or ayar-read-model server
- `vercel.json` / project settings aligned with Next.js.
- A short `docs/MCP.md` listing tools, auth env names, and what each tool is allowed to do.
- Health endpoint reports MCP configured vs authenticated separately.

Do not put Vercel/Supabase tokens in source. Use env + MCP auth. If MCP auth is not available in this runtime, implement config + scripts (`vercel link`, `vercel env pull`) and report `MCP AUTH: BLOCKED` with exact missing env names.

---

## PHASE 5 — Full Vercel deploy (only after gates 0–4)

Deploy the **complete** app, not a subset.

1. `vercel link` the project (new project name derived from Ayare Aman, not the old failed names unless the owner already has a working project).
2. Set env vars on Vercel: all `NEXT_PUBLIC_*`, Supabase URL/anon, server secrets, MCP tokens. Preview + Production.
3. Production build on Vercel must match local `npm run build`.
4. Supabase: production RLS live, redirects include the Vercel domain(s), Site URL correct.
5. Deploy production. Verify:
   - `https://<prod>/` 200
   - `/api/health` 200 with db true
   - login works on the live URL (not only localhost)
   - one golden invoice post on staging/prod seed tenant works
6. Configure `VERCEL_OIDC_TOKEN` / MCP deploy tools if that is the authenticated path.
7. Paste the live URL, deployment id, and a post-deploy manual smoke (login + dashboard + one accounting read).

If deploy fails, fix config (the architecture report says prior deploys failed on configuration). Do not leave a broken production domain.

---

## OUTPUT FORMAT (every session)

Work in the repo. Write real code. Then end with this report:

```
# AYARE AMAN READINESS REPORT

## PRODUCTION READY: YES | NO

## File inventory
- total files given:
- installed:
- executable:
- missing/created:
- failed (path + error + fix):

## Install & execution
- package manager / lockfile:
- typecheck:
- unit tests:
- e2e tests:
- production build:
- health check:

## Feature matrix
| feature | status | proof |
| auth signup/login |  |  |
| RTL dashboard markets |  |  |
| AI panel degrade-safe |  |  |
| CoA |  |  |
| double-entry journal |  |  |
| three-balance |  |  |
| hash chain |  |  |
| gold weight book |  |  |
| piece inventory |  |  |
| purchase invoice |  |  |
| sales invoice |  |  |
| blotter + FIFO PnL |  |  |
| reconciliation |  |  |
| RLS |  |  |

## Manual operations
- script steps run:
- bugs found:
- bugs fixed:
- remaining:

## Automatic operations
- jobs:
- retries/idempotency:
- remaining:

## MCP
- vercel:
- foundry/read-model:
- hermes:
- auth status:

## Vercel
- project:
- production URL:
- deployment id:
- post-deploy smoke:

## Blockers
- (empty only if PRODUCTION READY: YES)
```

## Coding rules

- Production-grade TypeScript. Comment the financial math, not noise.
- Integer minor units: IRR rials (or tomans if the existing app already standardized — pick one, document, convert at the boundary) and gold milligrams.
- Atomic posts only. Append-only journal. Reversals are new rows.
- Persian UI copy for operators. English for code identifiers.
- Do not add unrelated features (social, extra landing marketing) until the gates pass.
- Do not declare victory without the report and without tests.

Start now with Phase 0 on the files in this workspace. Continue through Phase 5 without asking permission between phases. Only stop for missing secrets, and then still deliver a complete local app plus the readiness report.

---
