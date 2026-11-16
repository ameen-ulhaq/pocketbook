# Pocketbook spec

## Product

A simple, complete personal expense app for everyone, in any region. Works offline, no account, data stays on the phone. Free tier with core tracking; one-time lifetime Pro.

## Decisions so far

- No bank sync. No receipt photos. Email parsing is out.
- Easiest storage: local SQLite on the device, with one-tap backup and restore to a file the user keeps (Drive, iCloud, email, WhatsApp).
- Cloud sync and live shared groups are not in v1.
- Desktop (NativePHP Desktop) comes later; phone and desktop would exchange data through the backup file.
- Reducing manual entry: share-to-add, one-line quick entry ("lunch 4.50 cash"), one-tap repeats, recurring auto-posting, CSV statement import, home widget, daily reminder. Android only: read SMS and notifications from allowed senders into a review list. iPhone: Shortcuts automation that passes message text to the app. Nothing is saved without the user tapping Add.
- Free vs Pro: free has tracking, budgets, reports, backup and up to 3 splits and 3 debts. Pro (one purchase) unlocks unlimited splits and debts, auto-capture, multi-currency, CSV and PDF export, home widget. Every Pro feature runs on the phone so a lifetime price is sustainable. Price is a placeholder.

## Features

**Transactions:** expense, income, transfer. Fields: amount, category, account, date, note, optional tags. Edit, delete, search, filter by type, month, account, category.

**Accounts:** cash, bank, card, mobile wallet. Live balance. Transfers between accounts are not spending.

**Categories:** sensible defaults, custom, optional subcategories, icon from a fixed set (no per-category colors).

**Budgets:** monthly overall and per category. Alerts at 80% and 100%. Status always shown in words ("Over by $12").

**Recurring:** rent, salary, subscriptions, installments. Posted when due; if the app was closed, missed ones are added next open. Optional reminder.

**Reports:** where it went (donut + list), trends (income vs spending bars for six months, daily spending), by period (week, month, year, custom). Export CSV/PDF (Pro).

**Debts:** money owed to me and money I owe. Person or lender, amount, due date, partial payments, reminders. Repayment is a payment against the debt, never income or expense.

**Shared expenses and bill splitting:** a private ledger; friends do not need the app. Add a bill, choose who paid and which account, pick people, split Equal, Exact, Percent or Shares. The user's own share is the only expense; the rest becomes "owed to you" per person. Record payments (partial allowed). Share a text summary through WhatsApp/SMS.

**Paid on behalf:** (a) I paid for someone: not my expense, shows as owed to me. (b) Holding money for someone: money received in my account that I must pay out is not income and clears when paid out.

**Settings:** language (RTL ready), currency, month start day (payday), theme, app lock (PIN + biometric), recurring, budgets, categories, accounts, backup and export, allowed senders.

## Money rules

- Amounts are integer minor units + currency code. Display with the device locale; always show the sign (−$4.50, +$1,400).
- Account balance = opening balance + income − expense ± transfers ± debt payments that touch the account.
- Spending reports and budgets count only expense transactions (own share for splits). Transfers, repayments and held funds are excluded.
- A "month" starts on the user's month start day.

## Data model (draft)

- `settings(key, value)`
- `accounts(id, name, type, currency, opening_balance_minor, archived)`
- `categories(id, name, kind, icon, parent_id, sort, archived)`
- `transactions(id, type, amount_minor, currency, account_id, to_account_id, category_id, occurred_at, note, tags, split_id, debt_id, created_at)`
- `budgets(id, category_id nullable, limit_minor, period)`
- `recurring(id, type, amount_minor, account_id, category_id, note, frequency, day, next_run_at, active, remind_days)`
- `people(id, name)`
- `debts(id, person_id, direction, kind, principal_minor, currency, due_on, note, status)`  — direction: owed_to_me or i_owe; kind: loan, on_behalf, held, split_share, installment
- `debt_payments(id, debt_id, amount_minor, account_id, occurred_at, note)`
- `splits(id, title, total_minor, paid_account_id, method, occurred_at, transaction_id)`
- `split_shares(id, split_id, person_id nullable for the user, share_minor, debt_id)`

## Screens (21)

Row 1: Welcome and setup, App lock, Home, Add transaction, Transactions (history), Transaction detail.
Row 2: Accounts, Categories, Budgets, Recurring, Review auto-captured entries.
Row 3: Reports overview, Reports trends, Debts, Debt detail with record payment, Paid on behalf.
Row 4: Split a bill, Split detail, Settings (More), Backup and export, Pro upgrade.

Navigation: bottom bar with Home, History, a raised Add button, Reports, More. 78px high, labels always shown.

## Look

Calm and simple. Off-white background, white cards with a 1px border, one deep-green accent, green for money in, red for debts and overspending, amber for warnings. Neutral icon tiles; no per-category colors. One font (Inter). The only gold is on the Pro banner. Tokens are in `docs/design/tokens.json`; map each token to a CSS variable of the same name.

## Phase 1 done means

- Fresh install runs the setup flow, then lands on Home with default accounts and categories.
- Add, edit, delete expense/income/transfer; balances update correctly.
- History search and filters work; detail screen shows and edits an entry.
- PIN and biometric lock work.
- Currency and month start day affect all screens.
- Backup writes one file; restore replaces data from it and returns to the same state.
- Feature tests cover balances, transfers, month boundaries, and backup round trip.
- Runs on the iOS simulator and an Android emulator.
