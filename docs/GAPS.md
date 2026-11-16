# Pocketbook: what the design does not show yet

The 21 designed screens cover the main path. Everything below is needed for a complete app but is not drawn. For each: what it must do, and a recommended default so it can be built. Build these with the design system tokens and the existing patterns (Field rows, Chips, Buttons, Cards). Mark a design request in your report so the owner can have it designed.

Status key: **Decide** = needs the owner's call; **Default** = safe to build as described.

## Setup and security
| Item | Needed behavior | Status |
| --- | --- | --- |
| Create PIN | After setup (optional) or when turning on App lock: enter 4 digits, confirm, then "Use Face ID or fingerprint?" | Default |
| Change PIN | More → Security; ask current PIN, then new twice | Default |
| Forgot PIN | No server, so no recovery. Offer: biometric if on; otherwise "Reset app" which erases data and offers restore from a backup file | Decide |
| Auto-lock time | Immediately, 1 minute, 5 minutes (default 1 minute) | Default |
| Language and currency pickers | Searchable lists | Default |

## Pickers and sheets
Category picker (with subcategories, search, New category); Account picker (with balances); Date picker; Month picker; Custom date-range picker; Person picker (with Add person); Day-of-month picker (1–28); Filter sheet in History (account, category, amount range); Share sheet (system).

## Forms not drawn
Add or edit account; Add or edit category (name, icon from fixed set, parent, archive); Add or edit budget; Add or edit recurring item; Add debt; Edit debt (due date, note); Edit or delete split; Add person; Tags on a transaction (free text chips); Adjust account balance.

## Detail screens not drawn
Account detail (balance, its transactions, edit, adjust, archive); Category detail (month total, entries); Allowed senders (list, add sender name or number, per-sender rule preview, enable/disable); CSV statement import (choose file, map columns once, preview, skip duplicates); Share-to-add (receive text, show parsed Add screen); Recurring item detail; Person detail (all debts and splits with that person).

## States
- **Empty states** for every list (History, Accounts beyond Cash, Budgets, Recurring, Debts, Review entries, Reports with no data).
- **Loading:** local data is fast; no spinners except export and restore.
- **Errors:** clear one-line messages with a retry for backup, restore, export, purchase, and file access.
- **Success states** for Pro unlock, restore, backup, export.
- **Free limit reached:** when adding the 4th split or debt, show Pro with the reason ("You've used your 3 free splits").
- **Notification permission** request (budgets, recurring reminders, backup reminder), with a gentle pre-explanation.

## Settings additions
Reminders (daily "log your day" time), Debts and Splits shortcuts, About and Help (version, privacy statement "all data stays on this phone", contact), Dark theme tokens (a `dark` theme in `tokens.json` and a contrast check), Data: Delete all data (confirm twice).

## Pro and advanced
Pro success state; Restore result messages; Multi-currency flow (account currency, manual exchange rates, base currency for totals, rates screen); Home-screen widget (Add expense, balance); iPhone Shortcuts setup guide (in-app steps with a ready template); Android notification-listener permission explainer.

## Open decisions for the owner
1. Forgot PIN policy (see above).
2. Whether changing the currency in Settings should be blocked once data exists (recommended: allow display change with a clear warning; real conversion only in Pro multi-currency).
3. App name and logo. "Pocketbook" is a placeholder.
4. Pro price and store product ID.
5. Languages to ship first (English plus Urdu and Arabic would test right-to-left early).
6. Whether to ship a light-only v1 and add Dark later (recommended).
