# InvestTrack 📈
> Personal investment tracker and payday system — built for ENGLE

A personal finance web app for tracking investments, completing payday checklists, and building long-term wealth consistently. Built with React (CDN), Supabase, and hosted on GitHub Pages.

---

## 🌐 Live App
```
https://engmanmano.github.io/investment-tracker
```
Open on any device — phone, PC, tablet. Add to home screen for app-like experience.

---

## 🏗️ Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| Frontend | React 18 via CDN + Babel | Single `index.html` — no build step needed |
| Database | Supabase (PostgreSQL) | Project: `investment-tracker`, Asia-Pacific/Tokyo |
| Hosting | GitHub Pages | Free, auto-deploys from `main` branch |
| Auth | Supabase Email/Password | Single user — just you |

---

## 📁 Repository Structure
```
investment-tracker/
└── index.html        ← The entire app lives here
└── README.md         ← This file
```

---

## 🗄️ Database (Supabase)

**Project URL:** `https://fimxumrsegtkcgsnhjzy.supabase.co`

### Tables

| Table | Purpose |
|---|---|
| `accounts` | Your platforms — COL, DragonFi, IBKR, GCash etc. |
| `investments` | All 33 holdings with amounts, units, avg price |
| `payday_logs` | Each completed payday as a single record |
| `transactions` | Every individual transfer logged per payday |
| `settings` | Exchange rate, goals, PERA limit, preferences |

### Key Settings (editable in app)

| Key | Default | Notes |
|---|---|---|
| `usd_to_php_rate` | 56.00 | Update manually when doing IBKR transfers |
| `portfolio_goal_php` | 1,000,000 | Your ₱1M target |
| `pera_annual_contributed` | 102,000 | Update each year |
| `pera_annual_limit` | 200,000 | Raised from ₱100K in 2023 |
| `payday_budget_10th` | 10,000 | Budget per 10th payday |
| `payday_budget_20th` | 10,000 | Budget per 20th payday |

### Security
- Row Level Security (RLS) enabled on all tables
- Only authenticated users can access data
- Publishable key is safe to include in frontend code
- **Never expose the Secret key** (`sb_secret_...`) in any file

---

## 💰 Payday System

**Schedule:** 10th and 20th of every month
**Flow:** LBP → GoTyme (−₱15) → distribute

### 10th Payday — ₱10,000

| Investment | Amount | Platform | Notes |
|---|---|---|---|
| MP2 | ₱2,000 | Pag-IBIG | Tax-free ~7% |
| VWRA (stage in GoTyme) | ₱3,000 | GoTyme → WISE → IBKR | Deploy quarterly to minimize fees |
| MBT | ₱1,000 | DragonFi PERA | Avg ₱67.50 |
| COOP (Providers) | ₱1,000 | Maya Personal Goal | Accumulate toward ₱50K TD |
| Manulife Asia Pacific REITs | ₱1,000 | GCash GFunds | |
| BPI PERA MMF | ₱500 | DragonFi PERA | |
| Buffer | ₱1,500 | GoTyme | Deploy only when clear opportunity exists |

### 20th Payday — ₱10,000

| Investment | Amount | Platform | Notes |
|---|---|---|---|
| MP2 | ₱1,000 | Pag-IBIG | |
| OGP | ₱3,500 | DragonFi PERA | 1 board lot ~₱3,400 · hold until 2036 |
| SGP | ₱2,500 | DragonFi PERA | 1 board lot ~₱2,200 · hold until 2033 |
| DMC | ₱1,000 | DragonFi PERA | 1 board lot ~₱1,000 |
| Global Tech Feeder Fund | ₱1,000 | GCash GFunds | ATRAM |
| Crypto (5 × ₱200) | ₱1,000 | Bitget | Qubic, Starknet, Tidecoin, Neptune Cash, QRL |

---

## 🏦 Account Roles

| Account | Role |
|---|---|
| **LBP** | Salary receiving account |
| **GoTyme** | Hub — funneling + VWRA quarterly staging (3% interest, free transfers) |
| **Maya** | Daily spending — maximize 6% boost |
| **UnionBank Savings+** | Emergency fund (3-6 months) + free group life insurance (maintain ₱25K ADB) |

---

## 📊 PERA Notes

- **Annual limit:** ₱200,000 (raised from ₱100K in 2023)
- **Current annual contributions:** ₱102,000
- **Tax credit:** 5% of contributions = ₱5,100 back on taxes per year 🎁
- **Administrator:** DragonFi (first SEC-accredited PERA administrator)
- **Holdings inside PERA:** OGP, SGP, DMC, MBT, BPI MMF

---

## 🌍 IBKR / VWRA Strategy

- **ETF:** VWRA (Vanguard FTSE All-World UCITS ETF) — switched from VUAA (S&P500 only)
- **Existing VUAA units:** Keep, let ride long term — no new contributions
- **Future contributions:** VWRA going forward
- **Transfer chain:** GoTyme → WISE → IBKR → buy VWRA
- **Frequency:** Accumulate ₱3,000/payday in GoTyme → deploy **quarterly** (minimizes IBKR trading fees)
- **Why WISE:** ~0.8-1% fee, much cheaper than bank wire ($30-50 flat)
- **IBKR access:** Browser requires VPN in PH (SEC/BSP directive) — use mobile app instead

---

## 🔑 Keys & Credentials

> ⚠️ Store these in Proton Pass — never commit the secret key to GitHub

| Item | Where to find |
|---|---|
| Supabase URL | Supabase Dashboard → Data API |
| Publishable key | Supabase Dashboard → Settings → API Keys |
| Secret key | Supabase Dashboard → Settings → API Keys (keep private!) |
| DB password | Proton Pass ← already saved ✅ |

---

## 🔧 How to Update the App

### Change an investment amount
1. Open the app → **Settings**
2. Edit the relevant value and save
3. For payday allocations (hardcoded) — edit `index.html` directly

### Add a new investment
Run this in Supabase SQL Editor:
```sql
insert into investments (account_id, name, ticker, category, currency, amount_invested, current_value, units, average_price, first_buy_date, is_active, is_pera, notes)
values (
  (select id from accounts where name='DragonFi PERA'),
  'Investment Name', 'TICKER', 'stock', 'PHP',
  0, 0, 0, 0,
  '2026-01-01', true, true, 'Your notes here'
);
```

### Update current values
Run this in Supabase SQL Editor:
```sql
update investments set current_value = 12345.00 where ticker = 'OGP';
```

### Update exchange rate
Open app → **Settings** → tap USD→PHP Rate → edit → save.

### After each payday
The checklist automatically logs to the database when you tap **"Log This Payday"** at the end.

---

## 🚀 Deployment

### First time setup
1. Create Supabase project (done ✅)
2. Run SQL for all 5 tables (done ✅)
3. Create GitHub repo `investment-tracker` (done ✅)
4. Upload `index.html` to repo (done ✅)
5. Enable GitHub Pages → Settings → Pages → Branch: main → Save

### Updating the app
```bash
# Edit index.html locally, then:
git add index.html
git commit -m "Update: describe what you changed"
git push
# GitHub Pages auto-deploys in ~1-2 minutes
```

### If you lose the index.html
Rebuild from this repo — the Supabase database is separate and your data is safe.

---

## 📱 Adding to Phone Home Screen

**Android (Chrome):**
1. Open the app URL in Chrome
2. Tap ⋮ menu → "Add to Home Screen"
3. Name it "InvestTrack" → Add

**iPhone (Safari):**
1. Open the app URL in Safari
2. Tap Share button → "Add to Home Screen"
3. Name it "InvestTrack" → Add

---

## ⏰ Reminders

Set a **recurring alarm** on your phone:
- 📅 Every 10th of the month → label: "Payday — invest now 💰"
- 📅 Every 20th of the month → label: "Payday — invest now 💰"

---

## 📋 Pending Action Items

- [ ] MBT (COL) → consolidate to DragonFi PERA when price approaches ₱69.50 avg (currently at loss — actually no CGT if sold now, consider it)
- [ ] Check UnionBank group life insurance coverage amount
- [ ] COOP: build toward ₱50K threshold for Time Deposit
- [ ] OKX/MEXC: consolidate micro crypto positions eventually
- [ ] IBKR: monitor PH regulatory situation — consider alternatives if access worsens
- [ ] PERA: ₱98,000 room left annually — consider increasing contributions
- [ ] Update `current_value` for all investments monthly (manually in Supabase or via Settings)

---

## 💡 Investment Philosophy

> The boring strategy wins.

- **Consistent** — invest every payday without fail, regardless of market conditions
- **Automated** — same amounts, same platforms, same order every time
- **Diversified** — PH + global, equities + fixed income + REITs, tax-advantaged + regular
- **Long-term** — 20+ year horizon, growth-focused, retirement-level thinking
- **Low-cost** — VWRA (0.22% ER), PERA tax credit (5%), MP2 tax-free returns

**Monthly totals:**
- PERA equities: ₱7,000 (35%)
- Global index VWRA: ₱6,000 (30%)
- Government MP2: ₱3,000 (15%)
- REITs: ₱1,000 (5%)
- Stocks: ₱1,000 (5%)
- COOP: ₱1,000 (5%)
- Crypto: ₱1,000 (5%)

---

*Built April 2026 · engmanmano · Powered by boring consistency 🌱*
