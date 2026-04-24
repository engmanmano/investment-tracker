# InvestTrack — Maintenance Guide
> SQL cheat sheet for managing your investment tracker database
> All queries run in: Supabase Dashboard → SQL Editor

---

## 📋 TABLE OF CONTENTS
1. [Viewing Your Data](#1-viewing-your-data)
2. [Updating Investment Values](#2-updating-investment-values)
3. [Adding New Investments](#3-adding-new-investments)
4. [Adding New Accounts / Platforms](#4-adding-new-accounts--platforms)
5. [Logging a Missed Payday](#5-logging-a-missed-payday)
6. [Recording Dividends](#6-recording-dividends)
7. [Recording a Stock Sale](#7-recording-a-stock-sale)
8. [Updating Settings](#8-updating-settings)
9. [Monthly Maintenance Checklist](#9-monthly-maintenance-checklist)
10. [Fixing Mistakes](#10-fixing-mistakes)

---

## 1. VIEWING YOUR DATA

### See all investments
```sql
select name, ticker, category, currency, amount_invested, current_value, is_pera
from investments
where is_active = true
order by category, name;
```

### See PERA holdings only
```sql
select name, ticker, amount_invested, current_value
from investments
where is_pera = true
order by name;
```

### See all accounts and cash balances
```sql
select name, platform_type, currency, cash_balance
from accounts
order by platform_type;
```

### See payday history
```sql
select payday_date, payday_type, total_invested, notes
from payday_logs
order by payday_date desc;
```

### See all transactions for a specific payday
```sql
select t.transaction_date, t.transaction_type, t.amount_php, t.notes
from transactions t
join payday_logs p on t.payday_log_id = p.id
where p.payday_date = '2026-04-20'  -- change date
order by t.created_at;
```

### See portfolio total by category
```sql
select 
  category,
  currency,
  count(*) as num_investments,
  sum(amount_invested) as total_invested,
  sum(current_value) as total_current
from investments
where is_active = true
group by category, currency
order by category;
```

### Check current settings
```sql
select key, value, notes from settings order by key;
```

---

## 2. UPDATING INVESTMENT VALUES

### Update a single investment's current value
```sql
-- Use this after checking the app or your broker
update investments
set current_value = 7500.00  -- enter new value
where ticker = 'OGP';
```

### Update MP2 after a contribution
```sql
-- Add your contribution amount to both fields
update investments
set 
  amount_invested = amount_invested + 2000,  -- how much you added
  current_value = current_value + 2000
where ticker = 'MP2-1';  -- or MP2-2 for rollover account
```

### Update a fund after contribution (GCash GFunds, BPI MMF, etc.)
```sql
update investments
set 
  amount_invested = amount_invested + 1000,
  current_value = amount_invested + 1000  -- approximate until next valuation
where ticker = 'MAN-APAC-REIT';  -- change ticker as needed
```

### Update stock after buying a board lot
```sql
-- Example: bought 100 more OGP shares at ₱35.00
update investments
set
  amount_invested = amount_invested + 3500,  -- 100 shares × ₱35
  units = units + 100,
  average_price = (amount_invested + 3500) / (units + 100),  -- recalculate avg
  current_value = (units + 100) * 35.00  -- units × current price
where ticker = 'OGP';
```

### Update stock current value only (price changed, no new purchase)
```sql
-- Example: OGP is now trading at ₱36.50
update investments
set current_value = units * 36.50
where ticker = 'OGP';
```

### Update VUAA/VWRA after IBKR purchase
```sql
-- Example: bought 0.5 more VWRA units at $130.00
update investments
set
  amount_invested = amount_invested + 65.00,  -- in USD
  units = units + 0.5,
  average_price = (amount_invested + 65.00) / (units + 0.5),
  current_value = (units + 0.5) * 130.00  -- current price per unit
where ticker = 'VUAA';
```

### Bulk update all stock current values (do this monthly)
```sql
-- Update each stock's current value based on latest price
-- Check prices on COL Financial or PSE website first
update investments set current_value = units * 68.00  where ticker = 'MBT' and account_id = (select id from accounts where name = 'COL Financial');
update investments set current_value = units * 68.00  where ticker = 'MBT' and account_id = (select id from accounts where name = 'DragonFi PERA');
update investments set current_value = units * 35.00  where ticker = 'OGP';
update investments set current_value = units * 23.00  where ticker = 'SGP';
update investments set current_value = units * 10.50  where ticker = 'DMC';
update investments set current_value = units * 162.00 where ticker = 'JFC';
```

---

## 3. ADDING NEW INVESTMENTS

### Add a new stock (PHP, inside PERA)
```sql
insert into investments (
  account_id, name, ticker, category, currency,
  amount_invested, current_value, units, average_price,
  first_buy_date, is_active, is_pera, notes
)
values (
  (select id from accounts where name = 'DragonFi PERA'),
  'Company Full Name',   -- e.g. 'Semirara Mining and Power Corp.'
  'SCC',                 -- ticker symbol
  'stock',               -- category
  'PHP',
  5000.00,               -- amount you invested
  5000.00,               -- current value (same as invested on first buy)
  200,                   -- number of shares
  25.00,                 -- average price per share
  '2026-05-10',          -- date of first buy
  true,                  -- is_active
  true,                  -- is_pera (true if inside DragonFi PERA)
  'Your notes here'
);
```

### Add a new stock (PHP, NOT in PERA — e.g. COL Financial)
```sql
insert into investments (
  account_id, name, ticker, category, currency,
  amount_invested, current_value, units, average_price,
  first_buy_date, is_active, is_pera, notes
)
values (
  (select id from accounts where name = 'COL Financial'),
  'Company Full Name',
  'TICKER',
  'stock',
  'PHP',
  10000.00,
  10000.00,
  100,
  100.00,
  '2026-05-10',
  true,
  false,  -- NOT PERA
  'Your notes here'
);
```

### Add a new crypto position
```sql
insert into investments (
  account_id, name, ticker, category, currency,
  amount_invested, first_buy_date, is_active, is_pera, notes
)
values (
  (select id from accounts where name = 'Bitget'),
  'Tidecoin',
  'TIDE',
  'crypto',
  'PHP',
  200.00,
  '2026-05-20',
  true,
  false,
  'Quantum-resistant thesis · small position'
);
```

### Add a new fund (GCash GFunds)
```sql
insert into investments (
  account_id, name, ticker, category, currency,
  amount_invested, current_value, units, average_price,
  first_buy_date, is_active, is_pera, notes
)
values (
  (select id from accounts where name = 'GCash GFunds'),
  'Fund Full Name',
  'FUND-TICKER',
  'fund',
  'PHP',
  1000.00,
  1000.00,
  10.0000,      -- units purchased
  100.0000,     -- NAVPU at time of purchase
  '2026-05-10',
  true,
  false,
  'Your notes here'
);
```

---

## 4. ADDING NEW ACCOUNTS / PLATFORMS

```sql
insert into accounts (name, platform_type, currency, notes)
values (
  'Platform Name',      -- e.g. 'Tonik Bank'
  'digital_bank',       -- type: bank / digital_bank / broker / pera / crypto_exchange / digital_wallet / government / cooperative
  'PHP',                -- PHP or USD
  'Your notes here'
);
```

### Platform type reference
| Type | Examples |
|---|---|
| `bank` | LBP, UnionBank, BDO, BPI |
| `digital_bank` | GoTyme, Maya, Tonik |
| `broker` | COL Financial, IBKR |
| `pera` | DragonFi PERA |
| `crypto_exchange` | Bitget, OKX, MEXC, Maya Crypto |
| `digital_wallet` | GCash GFunds |
| `government` | Pag-IBIG MP2, SSS, PhilHealth |
| `cooperative` | Providers COOP |

---

## 5. LOGGING A MISSED PAYDAY

### Log a payday you completed but forgot to log in the app
```sql
-- Step 1: Create the payday log
insert into payday_logs (payday_date, payday_type, total_invested, notes)
values (
  '2026-04-20',    -- actual date you did the payday
  '20th',          -- '10th' or '20th'
  10000,           -- total amount moved
  'Backdated entry — completed manually'
)
returning id;      -- copy the id from the result!
```

```sql
-- Step 2: Log individual transactions
-- Replace 'paste-id-here' with the id from Step 1
insert into transactions (payday_log_id, transaction_type, transaction_date, amount_php, notes)
values
  ('paste-id-here', 'deposit', '2026-04-20', 1000, 'MP2'),
  ('paste-id-here', 'deposit', '2026-04-20', 3500, 'OGP (DragonFi PERA)'),
  ('paste-id-here', 'deposit', '2026-04-20', 2500, 'SGP (DragonFi PERA)'),
  ('paste-id-here', 'deposit', '2026-04-20', 1000, 'DMC (DragonFi PERA)'),
  ('paste-id-here', 'deposit', '2026-04-20', 1000, 'Global Tech Feeder Fund'),
  ('paste-id-here', 'deposit', '2026-04-20', 1000, 'Crypto 5x200');
```

---

## 6. RECORDING DIVIDENDS

### Log a dividend received (e.g. MBT paid dividends to COL)
```sql
-- Step 1: Add to account cash balance
update accounts
set cash_balance = cash_balance + 1250.00  -- dividend amount received
where name = 'COL Financial';

-- Step 2: Log the transaction
insert into transactions (
  investment_id, account_id, transaction_type, transaction_date, amount_php, notes
)
values (
  (select id from investments where ticker = 'MBT' and account_id = (select id from accounts where name = 'COL Financial')),
  (select id from accounts where name = 'COL Financial'),
  'dividend',
  '2026-05-15',   -- date dividend was credited
  1250.00,        -- amount received
  'MBT cash dividend · ₱2.50/share × 500 shares'
);
```

### Log MP2 dividend (annual)
```sql
insert into transactions (
  investment_id, account_id, transaction_type, transaction_date, amount_php, notes
)
values (
  (select id from investments where ticker = 'MP2-1'),
  (select id from accounts where name = 'Pag-IBIG MP2'),
  'dividend',
  '2026-12-31',
  5000.00,   -- your annual MP2 dividend
  'MP2 annual dividend · ~7% p.a.'
);

-- Also update current value
update investments
set current_value = current_value + 5000.00
where ticker = 'MP2-1';
```

---

## 7. RECORDING A STOCK SALE

### Sell a stock (e.g. selling MBT from COL when price recovers)
```sql
-- Step 1: Log the sale transaction
insert into transactions (
  investment_id, account_id, transaction_type, transaction_date, amount_php, units, price_per_unit, notes
)
values (
  (select id from investments where ticker = 'MBT' and account_id = (select id from accounts where name = 'COL Financial')),
  (select id from accounts where name = 'COL Financial'),
  'sell',
  '2026-06-10',   -- date of sale
  38920.00,       -- total proceeds
  560,            -- shares sold
  69.50,          -- price per share
  'Sold MBT COL · moving position to DragonFi PERA'
);

-- Step 2: Mark investment as inactive (don't delete — keep history)
update investments
set is_active = false
where ticker = 'MBT'
  and account_id = (select id from accounts where name = 'COL Financial');

-- Step 3: Add proceeds to COL cash balance
update accounts
set cash_balance = cash_balance + 38920.00
where name = 'COL Financial';
```

---

## 8. UPDATING SETTINGS

### Update USD to PHP exchange rate
```sql
update settings
set value = '57.50', updated_at = now()  -- enter current rate
where key = 'usd_to_php_rate';
```

### Update PERA annual contributions (after each PERA transaction)
```sql
-- Add your latest PERA contribution to the running total
update settings
set value = (value::numeric + 3500)::text, updated_at = now()
where key = 'pera_annual_contributed';
```

### Reset PERA contributions every January 1
```sql
update settings
set value = '0', updated_at = now()
where key = 'pera_annual_contributed';
```

### Update portfolio goal
```sql
update settings
set value = '2000000', updated_at = now()  -- ₱2M new goal
where key = 'portfolio_goal_php';
```

---

## 9. MONTHLY MAINTENANCE CHECKLIST

Do this once a month (suggested: last day of the month):

```sql
-- 1. Update all stock current values (check prices on PSE/COL first)
update investments set current_value = units * [price] where ticker = 'MBT' and account_id = (select id from accounts where name = 'COL Financial');
update investments set current_value = units * [price] where ticker = 'MBT' and account_id = (select id from accounts where name = 'DragonFi PERA');
update investments set current_value = units * [price] where ticker = 'OGP';
update investments set current_value = units * [price] where ticker = 'SGP';
update investments set current_value = units * [price] where ticker = 'DMC';
update investments set current_value = units * [price] where ticker = 'JFC';

-- 2. Update VUAA/VWRA current value (check IBKR app)
update investments set current_value = units * [price_in_usd] where ticker = 'VUAA';

-- 3. Update exchange rate
update settings set value = '[current_rate]' where key = 'usd_to_php_rate';

-- 4. Verify portfolio total looks right
select 
  sum(case when currency = 'PHP' then current_value else 0 end) as php_total,
  sum(case when currency = 'USD' then current_value else 0 end) as usd_total
from investments
where is_active = true;
```

**Replace `[price]` with the actual current price you look up.**

---

## 10. FIXING MISTAKES

### Undo a wrong payday log (logged twice by accident)
```sql
-- Step 1: Find the log to delete
select id, payday_date, payday_type, total_invested from payday_logs order by created_at desc;

-- Step 2: Delete its transactions first
delete from transactions where payday_log_id = 'paste-id-here';

-- Step 3: Delete the log
delete from payday_logs where id = 'paste-id-here';
```

### Fix a wrong investment amount
```sql
update investments
set amount_invested = 6830.00  -- correct value
where ticker = 'OGP';
```

### Reactivate an archived investment
```sql
update investments
set is_active = true
where ticker = 'TICKER-HERE';
```

### See everything (including inactive investments)
```sql
select name, ticker, is_active, amount_invested
from investments
order by is_active desc, name;
```

---

## 🔑 QUICK REFERENCE — TICKERS

| Ticker | Investment | Platform |
|---|---|---|
| `MP2-1` | MP2 (2023-2028) | Pag-IBIG |
| `MP2-2` | MP2 (2026-2031) rollover | Pag-IBIG |
| `MBT` | Metropolitan Bank | COL Financial |
| `MBT` | Metropolitan Bank | DragonFi PERA |
| `OGP` | Oceana Gold Philippines | DragonFi PERA |
| `SGP` | Synergy Grid & Development | DragonFi PERA |
| `DMC` | DMCI Holdings Inc. | DragonFi PERA |
| `JFC` | Jollibee Foods Corp | DragonFi PERA |
| `BPI-PERA-MMF` | BPI PERA Money Market Fund | DragonFi PERA |
| `MAN-APAC-REIT` | Manulife Asia Pacific REIT | GCash GFunds |
| `ATRAM-TECH` | ATRAM Global Tech Feeder | GCash GFunds |
| `VUAA` | Vanguard S&P 500 UCITS ETF | IBKR |
| `BTC` | Bitcoin | Maya Crypto |
| `DOGE` | Dogecoin | Maya Crypto |
| `STRK` | Starknet | Bitget |
| `QUBIC` | Qubic | Bitget |

---

## 💡 TIPS

- **Never delete investments** — use `is_active = false` instead. History is valuable.
- **Always update `current_value` separately from `amount_invested`** — invested is what you put in, current is what it's worth now.
- **Run a SELECT before UPDATE** — confirm you're editing the right row before changing anything.
- **For stocks, always note units × price** — makes it easy to verify the math.
- **Update exchange rate before checking dashboard** — stale rate = inaccurate combined total.

---

*InvestTrack Maintenance Guide · engmanmano · Updated April 2026*
