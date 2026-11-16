# Pocketbook spec

## Product

A simple, complete personal expense app for everyone, in any region. Works offline, no account, data stays on the phone. Free tier with core tracking; one-time lifetime Pro.

## Decisions so far

- No bank sync. No receipt photos. Email parsing is out.
- Easiest storage: local SQLite on the device, with one-tap backup and restore to a file the user keeps (Drive, iCloud, email, WhatsApp).
- Cloud sync and live shared groups are not in v1.
- Desktop (NativePHP Desktop) comes later; phone and desktop would exchange data through the backup file.
- Reducing manual entry: share-to-add, one-line quick entry ("lunch 4.50 cash"), one-tap repeats, recurring auto-posting, CSV statement import, home widget, daily reminder. Android only: read notifications from allowed senders into a review list first; reading SMS is an optional later addition, only if Google Play allows it. iPhone: Shortcuts automation that passes message text to the app. Nothing is saved without the user tapping Add. CSV statement import is built in Phase 2 after export (free).
- Free vs Pro: free has tracking, budgets, reports, backup and up to 3 splits and 3 debts. Pro (one purchase) unlocks unlimited splits and debts, auto-capture, multi-currency, CSV and PDF export, home widget. Every Pro feature runs on the phone so a lifetime price is sustainable. Price is a placeholder.
- **Free limits count open items.** The free tier allows 3 open (not settled) splits and 3 open loans. Loans means debts of kind `loan` or `installment` only; debts created by a split (`split_share`), paid on behalf (`on_behalf`) and held money (`held`) never count toward the debt limit. Settling or deleting one frees a slot. Existing data is never removed or locked (for example after a restore); only adding the 4th is blocked.
- **Building before Pro exists (phases).** Export (P2) and the limits (P3) are built behind one `Entitlements` service that returns "Pro" in development builds until the purchase is built in P4. No feature code checks the store directly.

## Owner decisions (v1)

- Free limits count open (not settled) items only.
- Recurring transfers are not allowed in v1.
- Backups are plain, unencrypted files; the Backup screen says so.
- Android capture reads notifications first; SMS is optional and later.
- v1 is light-only: there is no Theme setting.
- The currency cannot be changed once any entry exists. It is chosen at setup. (Pro, P4, adds accounts in other currencies; the base currency stays the one chosen at setup.)

## Features

**Transactions:** expense, income, transfer. Fields: amount, category, account, date, note, optional tags. Edit, delete, search, filter by type, month, account, category. One-tap repeat: Detail offers Duplicate, which opens Add prefilled with today's date. Adjustments, debt movements and split shares also appear in History (see Transaction types).

**Accounts:** cash, bank, card, mobile wallet. Live balance. Transfers between accounts are not spending.

**Categories:** sensible defaults, custom, optional subcategories (one level), icon from a fixed set (no per-category colors). Reports and budgets roll subcategories up into their parent.

**Budgets:** monthly overall and per category (top-level categories; a category budget includes its subcategories). No rollover: each month starts fresh. Alerts at 80% and 100%. Status always shown in words ("Over by $12").

**Recurring:** rent, salary, subscriptions, and debt installments. Posted when due; if the app was closed, missed ones are added next open, each exactly once. Optional reminder. A debt installment posts a debt payment (not an expense).

**Reports:** where it went (donut + list), trends (income vs spending bars for six months, daily spending), by period (week, month, year, custom). Export CSV/PDF (Pro).

**Debts:** money owed to me and money I owe. Person or lender, amount, due date, partial payments, reminders. Repayment is a payment against the debt, never income or expense.

**Shared expenses and bill splitting:** a private ledger; friends do not need the app. Add a bill, choose who paid and which account, pick people, split Equal, Exact, Percent or Shares. The user's own share is the only expense; the rest becomes "owed to you" per person. Record payments (partial allowed). Share a text summary through WhatsApp/SMS.

**Paid on behalf:** (a) I paid for someone: not my expense, shows as owed to me. (b) Holding money for someone: money received in my account that I must pay out is not income and clears when paid out.

**Settings (More):** language (RTL ready), currency (locked once any entry exists), month start day (payday), reminders (log your day, weekly backup, budget warnings, due dates), app lock (PIN + biometric), auto-lock (immediately, 1, 5 or 15 minutes; default 1), change PIN, recurring, debts and splits, budgets, categories, accounts, backup and export, allowed senders (Android list; iPhone Shortcuts guide), about and help, delete all data (type ERASE). No theme setting in v1.

## Money rules

- Amounts are integer minor units + currency code. Display with the device locale; always show the sign (−$4.50, +$1,400).
- **Dates are local calendar dates** (`occurred_on`, no time and no time zone), so travelling never moves an entry to another day. Same-day order is by `created_at`.
- **Transaction types:** `expense`, `income`, `transfer`, `adjustment`, `debt_out`, `debt_in`.
  - `debt_out`: money leaves an account because of a debt (I lent, I paid someone's bill on their behalf, I paid back a loan or paid out held money).
  - `debt_in`: money enters an account because of a debt (I borrowed, someone paid me back, someone sent me money to hold).
  - `adjustment`: a signed correction created by "Adjust balance".
  - Debt and adjustment entries show in History under All only (not under the Expense, Income or Transfer chips) and carry a type pill ("Debt payment", "Loan", "Adjustment").
- **Account balance** = opening balance + income − expense + adjustments + transfers in − transfers out + `debt_in` − `debt_out`, counting only entries dated today or earlier. An account with no transactions shows its opening balance.
- **Future-dated entries** are allowed. They show in History marked "Upcoming" and start counting in balances, Home totals, budgets and reports on their date.
- **Spending reports and budgets count only `expense` entries** (own share for splits), rolled up to top-level categories. Transfers, adjustments, debt movements and held funds are excluded. Income totals count only `income`.
- **Month:** a "month" (budget/report period) starts on the user's month start day (1–28) and ends the day before the next one. It is named after the month it starts in (start day 25: 25 Oct–24 Nov is "Oct"). With day 1 it is the calendar month. Changing the day recalculates periods and never edits entries. Week = the locale's first weekday. Year = twelve periods starting with the January period.
- **Percentages** are rounded to the nearest whole number; a legend of percentages is adjusted by largest remainder so it adds to 100. A budget never shows 100% before spending reaches the limit, and never shows 0% once something is spent.
- **Splits:** equal split divides in minor units; leftover minor units go one each to people in list order (the user first). Percent and Shares use largest remainder so shares add exactly to the total. If someone else paid, the user's own share is still an `expense` (spending counts on the bill date) but with no account, so no account balance changes; the user owes that person a debt (`i_owe`, `split_share`), later paid with a `debt_out` payment. If the user is unticked from the list, no expense is created and the whole bill is owed to the user.
- **Archived accounts** are hidden from pickers and left out of totals. Archiving a non-zero account asks for confirmation ("Its balance will not count in your total"). An account with no entries may be deleted; otherwise only archived.
- Backups never contain the PIN, biometric choice, auto-lock time or Pro entitlement (kept in device secure storage, not the database).

## Data model (draft)

- `settings(key, value)`: language, currency, month_start_day, last_backup_at, reminder switches and times (log_day_on, log_day_time default 21:00, weekly_backup_on, budget_warnings_on, due_reminders_on). Security settings are not stored here.
- `accounts(id, name, type, currency, opening_balance_minor, archived)`: `type` is cash, bank, card or wallet (shown as Cash, Bank, Card, Mobile wallet); name unique among open accounts.
- `categories(id, name, kind, icon, parent_id, sort, archived)`: `kind` is expense or income; a subcategory inherits its parent's kind; one level deep.
- `transactions(id, type, amount_minor, currency, account_id nullable, to_account_id, category_id, occurred_on, note, source, split_id, debt_id, recurring_id, capture_id, created_at, updated_at)`
  - `amount_minor` is always positive; direction comes from `type`. `account_id` is null only for the user's own share of a bill someone else paid.
  - `source`: typed, quick_add, shared_in, recurring, capture, split, debt (feeds the "Added" row on Detail).
  - Unique `(recurring_id, occurred_on)` where `recurring_id` is not null, so a due item can never post twice.
- `tags(id, name)` and `transaction_tags(transaction_id, tag_id)`
- `budgets(id, category_id nullable, limit_minor, period, alerts_on)`: `period` is always `month` in v1; `alerts_on` is the form's warning switch (default on); null category = overall budget; one budget per category.
- `recurring(id, type, amount_minor, account_id, category_id, debt_id nullable, note, frequency, day, month, start_on, next_run_on, active, remind_days)`: `type` is expense, income or debt_payment; `day` is day of month (1–31, clamped to the month's last day) or weekday; `month` is used by yearly.
- `people(id, name)`
- `debts(id, person_id, direction, kind, principal_minor, currency, due_on, note, status, installment_minor nullable, written_off_minor)`: `direction` owed_to_me or i_owe; `kind` loan, on_behalf, held, split_share, installment; `status` open or settled. Remaining = principal − sum of payments − written_off_minor. "Mark as settled" writes off the remainder: no entry, no balance change, and it is neither income nor expense. A settled debt (paid or written off) shows the settled look on Debt detail.
- `debt_payments(id, debt_id, amount_minor, account_id nullable, occurred_on, note)`: one row per payment. When `account_id` is set the payment also owns one `debt_out`/`debt_in` transaction (`transactions.debt_id`), which is what moves the balance. A debt's opening movement (lent or borrowed through an account) is the same kind of transaction, not a payment.
- `splits(id, title, total_minor, paid_account_id nullable, paid_by_person_id nullable, method, occurred_on, category_id, transaction_id)`
- `split_shares(id, split_id, person_id nullable for the user, input_value, share_minor, debt_id)`: `input_value` keeps the typed percent, weight or exact amount so the split can be edited.
- `captures(id, sender_id, received_at, raw_text, amount_minor, account_id, merchant, category_id, status)`: status new, added or ignored. (P4)
- `allowed_senders(id, name, sender, account_id, default_category_id nullable, enabled)` (P4): a null category means "Ask me each time".
- `exchange_rates(currency, rate_to_base, updated_on)` (P4, Pro)

## Screens (46)

Row 1: Welcome and setup, App lock, Home, Add transaction, Transactions (history), Transaction detail.
Row 2: Accounts, Categories, Budgets, Recurring, Review auto-captured entries.
Row 3: Reports overview, Reports trends, Debts, Debt detail with record payment, Paid on behalf.
Row 4: Split a bill, Split detail, Settings (More), Backup and export, Pro upgrade.

Row 5 (PIN and capture): Create a PIN, Confirm PIN, Forgot PIN, Allowed senders.
Row 6 (Add and edit forms): New account, New category, New budget, New recurring item, Add debt.
Row 7 (Settings pages and states): Choose an icon, Auto-lock, Reminders, About and help, Erase everything, Add sender, Auto-capture on iPhone, Forgot PIN (no biometrics), Change PIN.
Row 8 (Menus, details and states): Account detail, Edit account, Debt menu, Split menu, Time picker, Allowed senders (access off), Help and tips.
Edit forms for categories, budgets, recurring items, debts and senders reuse the New forms, filled in, with a red Delete; accounts have their own Edit account with Archive.

Navigation: bottom bar with Home, History, a raised Add button, Reports, More. 78px high, labels always shown. Accounts, Categories, Budgets and Debts are list screens under More: they keep the bar with More highlighted and have no Back arrow. Recurring, Review entries, Backup, Pro and every detail or form screen have a Back arrow and no bar.

## Look

Calm and simple. Off-white background, white cards with a 1px border, one deep-green accent, green for money in, red for debts and overspending, amber for warnings. Neutral icon tiles; no per-category colors. One font (Inter). Gold appears only on Pro things: the Pro banner and the small Pro tag (pale gold fill, dark gold text; both need tokens). Tokens are in `docs/design/tokens.json`; map each token to a CSS variable of the same name.

## Phase 1 done means

- Fresh install runs the setup flow, then lands on Home with default accounts and categories.
- Add, edit, delete expense/income/transfer; balances update correctly.
- History search and filters work; detail screen shows and edits an entry.
- PIN and biometric lock work.
- Currency and month start day affect all screens.
- Backup writes one file; restore replaces data from it and returns to the same state. A new phone can restore straight from the Welcome screen.
- Feature tests cover balances, transfers, month boundaries, and backup round trip.
- Runs on the iOS simulator and an Android emulator.
