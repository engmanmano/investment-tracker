# InvestTrack — Conversation Handoff
> Context summary for continuing development in Cowork
> Original conversation: Claude.ai · April 24, 2026

---

## 👤 WHO I AM

**Name:** Engle (engmanmano)
**GitHub:** https://github.com/engmanmano
**Investment philosophy:** Boring, consistent, long-term. Boglehead-inspired.
**Payday schedule:** 10th and 20th of every month · ₱10,000 each
**Goal:** ₱1,000,000 portfolio · 20+ year horizon · growth-focused

---

## 🌐 THE APP

**Live URL:** https://engmanmano.github.io/investment-tracker
**Repo:** https://github.com/engmanmano/investment-tracker
**Stack:** Single `index.html` · React 18 CDN · Babel standalone · Supabase JS v2 · GitHub Pages

**Files in repo:**
- `index.html` — the entire app
- `README.md` — full system documentation
- `MAINTENANCE.md` — SQL cheat sheet for all common tasks

---

## 🗄️ DATABASE

**Supabase Project:** investment-tracker
**URL:** https://fimxumrsegtkcgsnhjzy.supabase.co
**Region:** Asia-Pacific (Tokyo)
**Publishable key:** sb_publishable_SISPn15jUjWY_J5gyihlIA_gfz4vFo1
**DB password:** stored in Proton Pass

### Tables
| Table | Rows | Purpose |
|---|---|---|
| `accounts` | 14 | Platforms and brokerages |
| `investments` | 33 | All holdings |
| `payday_logs` | varies | Completed payday records |
| `transactions` | varies | Individual transfer records |
| `settings` | 8 | Exchange rate, goals, preferences |

### Key settings values
| Key | Value |
|---|---|
| `usd_to_php_rate` | 56.00 |
| `portfolio_goal_php` | 1000000 |
| `pera_annual_limit` | 200000 |
| `pera_annual_contributed` | 102000 (planned annual, not actual balance) |
| `payday_budget_10th` | 10000 |
| `payday_budget_20th` | 10000 |

---

## 💰 PAYDAY ALLOCATION

### 10th — ₱10,000
| Investment | Amount | Platform |
|---|---|---|
| MP2 | ₱2,000 | Pag-IBIG |
| VWRA (stage in GoTyme) | ₱3,000 | GoTyme → WISE → IBKR quarterly |
| MBT | ₱1,000 | DragonFi PERA |
| COOP Providers | ₱1,000 | Stage in Maya Personal Goal |
| Manulife Asia Pacific REITs | ₱1,000 | GCash GFunds |
| BPI PERA MMF | ₱500 | DragonFi PERA |
| Buffer | ₱1,500 | GoTyme |

### 20th — ₱10,000
| Investment | Amount | Platform |
|---|---|---|
| MP2 | ₱1,000 | Pag-IBIG |
| OGP | ₱3,500 | DragonFi PERA · hold until 2036 |
| SGP | ₱2,500 | DragonFi PERA · hold until 2033 |
| DMC | ₱1,000 | DragonFi PERA |
| Global Tech Feeder Fund | ₱1,000 | GCash GFunds |
| Crypto (5 × ₱200) | ₱1,000 | Bitget |

---

## 🏦 ACCOUNT ROLES
| Account | Role |
|---|---|
| LBP | Salary in |
| GoTyme | Hub + VWRA staging (3% interest, free transfers) |
| Maya | Daily spending · 6% boost |
| UnionBank Savings+ | Emergency fund + free life insurance (₱25K ADB) |

---

## 📱 APP SCREENS

### 1. Dashboard (Home)
- Portfolio total (PHP + USD + combined at manual exchange rate)
- PERA card: current balance (from DB) + 2026 contributions + tax credit
- ₱1M goal progress
- Last completed payday
- "Open Payday Checklist" button

### 2. Payday Checklist
- 10th / 20th tabs — hardcoded allocations (see above)
- Step 1: LBP → GoTyme checkboxes
- Step 2: Distribution items by category
- On complete: logs to `payday_logs` + `transactions` in Supabase
- ⚠️ Does NOT auto-update `amount_invested` / `current_value` in investments (v2 feature)

### 3. Portfolio
- Filter chips by category
- All investments from DB grouped by category
- Shows: name, ticker, platform, PERA badge, units, avg price, current value, gain/loss %
- Totals: PHP + USD + combined

### 4. History
- All `payday_logs` ordered by date desc
- Tap to expand and see individual transactions

### 5. Settings
- Edit: USD→PHP rate, portfolio goal, PERA contributions, payday budgets
- Shows: tax credit, PERA room left
- Sign out button

---

## 🐛 KNOWN ISSUES (V2 TODO)

1. **Checklist does not auto-update portfolio values** — after logging a payday, user must manually update `amount_invested` and `current_value` in Supabase SQL Editor
2. **`pera_annual_contributed` is manual** — does not auto-sum from transactions. User updates it in Settings.
3. **Auth redirect URL broken** — email confirmation link redirects to localhost. Fix: Supabase → Auth → URL Configuration → set Site URL to `https://engmanmano.github.io/investment-tracker`
4. **PERA dashboard label was confusing** — fixed in latest version (shows Current Balance separately from Annual Plan)

---

## 🚀 V2 PRIORITY FEATURES

### High Priority
- [ ] Auto-update fund balances (MP2, MMF, REITs) when payday is logged
- [ ] "Update Prices" prompt after payday for stocks/ETF/crypto
- [ ] Manual current value editor in Portfolio (tap to edit inline)

### Medium Priority
- [ ] Growth chart (line chart using payday_logs data)
- [ ] Per-investment goal progress bars
- [ ] PERA auto-calculate from transactions
- [ ] Add/remove investments via Settings UI (no SQL needed)

### Nice to Have
- [ ] Multi-user support (add user_id to tables, update RLS)
- [ ] VWRA quarterly deployment reminder (when GoTyme staging hits ₱9K+)
- [ ] MBT consolidation alert (when COL price approaches ₱69.50 avg)
- [ ] Dividend tracker
- [ ] Export to CSV

---

## 📊 CURRENT PORTFOLIO SNAPSHOT (April 2026)

**Total value:** ~₱174,806
**PHP holdings:** ₱161,813
**USD holdings:** $232.03
**Gain vs invested:** ₱5,780 (3.4%)
**PERA actual balance:** ₱21,614
**Investments:** 33 total

### Holdings breakdown
| Category | Key Holdings |
|---|---|
| Government | MP2-1 (₱76K), MP2-2 (₱500) |
| PERA Stocks | OGP, SGP, DMC, MBT, JFC |
| PERA Fund | BPI PERA MMF |
| Stocks | MBT (COL) — consolidating to PERA |
| International ETF | VUAA (IBKR, USD) — switching new contributions to VWRA |
| REITs | Manulife Asia Pacific REIT (GCash) |
| Funds | ATRAM Global Tech Feeder (GCash) |
| Cooperative | Providers COOP (₱2,900) |
| Crypto | BTC, DOGE (Maya) · STRK, QUBIC, BGB, ASTEROID (Bitget) · various (OKX, MEXC) |

---

## ⚠️ IMPORTANT NOTES FOR COWORK

1. **Edit `index.html` directly** — all React code is in one file, no build process needed
2. **After editing** → save → push to GitHub → auto-deploys in ~1-2 min
3. **Supabase publishable key is safe** to be in the file (RLS protects data)
4. **Never put the secret key** (`sb_secret_...`) in any file
5. **Test changes** by opening https://engmanmano.github.io/investment-tracker after deploy
6. **Hard refresh** (Ctrl+Shift+R) after deploy to clear browser cache
7. **SQL changes** go in Supabase Dashboard → SQL Editor
8. **App auth** is email/password (separate from Supabase dashboard GitHub login)

---

## 🔧 DEVELOPMENT WORKFLOW

```
Edit index.html locally
    ↓
git add index.html
git commit -m "describe change"
git push
    ↓
Wait ~1-2 minutes
    ↓
Hard refresh https://engmanmano.github.io/investment-tracker
    ↓
Verify change looks correct
```

---

## 📁 FILE LOCATIONS (local machine)
Update these paths based on where Engle cloned the repo:
```
[local repo path]/index.html       ← main app file
[local repo path]/README.md        ← system documentation  
[local repo path]/MAINTENANCE.md   ← SQL cheat sheet
[local repo path]/HANDOFF.md       ← this file
```

---

*Generated from Claude.ai conversation · April 24, 2026*
*Continue development in Cowork · engmanmano*
