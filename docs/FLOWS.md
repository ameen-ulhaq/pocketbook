# Pocketbook: user flows

Each flow lists the steps in order, the screen at each step, and the rule that matters. Screen names match `docs/SCREENS.md`.

## 1. First launch
1. Open app → **Welcome and setup**. (On a new phone: tap **Have a backup? Restore it** instead, choose the file, confirm → Home. Skip the rest.)
2. Choose Language, Currency, Month start day, Cash on hand.
3. Tap **Start tracking**. App seeds the Cash account and default categories.
4. **Create a PIN** opens. Enter 4 digits → **Confirm PIN** → enter them again. A mismatch shows "PINs did not match. Try again." and clears. On **Skip for now** App lock stays off.
5. On Confirm, "Use Face ID or fingerprint too" (shown only if the device has it) turns biometrics on after the system prompt.
6. Land on **Home** (empty state).

## 2. Lock and unlock
1. App launch or return after the auto-lock time → **App lock**.
2. Biometric prompt opens first; on failure or cancel use the PIN keypad.
3. Right PIN → previous screen. Wrong PIN → shake, message, waits from 5 tries on (30 s, 1 min, 5 min, 15 min).
4. **Forgot PIN?** → **Forgot PIN** screen with two choices: **Use Face ID or fingerprint** (only if on; unlocks to Home; then More → Change PIN needs no old PIN) or **Erase and start over** → confirm (Cancel / Erase) → all data, PIN and settings are removed → **Welcome**, where **Have a backup? Restore it** can load a file.

## 3. Add an expense
1. Tap the raised **Add** button (or Add from Home empty state).
2. **Add** opens on Expense. Type the amount on the keypad.
3. Choose Category (picker), Account (default: last used for this type), Date (default today), Note (optional), and Tags if wanted.
4. **Save expense** → toast → back. Balance, Recent, budgets and reports update.

## 4. Quick add from a line of text
1. On **Home**, type "lunch 4.50 cash" in the quick add line and submit.
2. App reads the line with these rules (Default):
   - Amount: the first number, using the locale's decimal mark and also accepting the other mark (4.50, 4,50, 1,200.50). A leading currency symbol or code is ignored. More decimals than the currency allows are rounded.
   - Type: Expense unless the line starts with "+" or contains "income", "salary" or "received" (then Income).
   - Account: the first word that equals an account name (case-insensitive, or a unique prefix of 3+ letters). Else the default account.
   - Date: "today" or "yesterday" if present; else today.
   - Note: the remaining words, trimmed ("lunch"). Category: the one last used with that note, else left empty.
3. **Add** opens prefilled; if the category is empty the picker opens first. The user taps Save (or edits). Nothing is saved without Save.
4. If no amount is found: "I couldn't find an amount" and Add opens empty.

## 5. Add income
1. Add → switch to **Income**.
2. Amount, Category (income categories), Account, Date, Note → **Save income**.

## 6. Transfer between accounts
1. Accounts → transfer icon, or Add → **Transfer**.
2. Amount, **From** account, **To** account (must differ), Date, Note → **Save transfer**.
3. Neither account's income or spending changes; only balances move.

## 7. Find, edit and delete an entry
1. **History** → search or filter (type chips, month, and account or category in the month sheet) → tap a row → **Detail**.
2. **Edit** → Add prefilled → **Save changes**; or **Duplicate** → Add prefilled with today's date → Save; or **Delete** → confirm → toast → back.
3. Entries made by a split or debt payment open their parent instead.

## 8. Set a budget
1. More → **Budgets** → **New budget**. (Tap an existing card to edit its limit or delete it.)
2. On **New budget** choose Whole month or One category, pick the category, enter the monthly limit, leave "Warn me at 80% and 100%" on or off → **Save budget**.
3. As spending grows the bar goes accent → amber (80%) → red (over); local notifications fire at 80% and 100%.

## 9. Recurring item
1. More → **Recurring** → **New recurring item**.
2. On **New recurring item** choose Expense or Income, amount, category, account, Repeats, On day, Remind, Note → **Save**. (Debt installments: GAPS.)
3. When due (checked on app open and by background task) the entry is posted automatically with the due date, once per due date; the switch pauses it. Tap a card to edit (future postings only) or delete (posted entries stay).

## 10. Lend money and get repaid
1. **Debts** (Home debt boxes, or More → Debts and splits) → **Add debt** → I lent, person, amount, due date, From account (or "No account"), note → **Save debt**.
2. Debt appears under Owed to you. Later: tap it → **Debt detail**.
3. Enter the amount received (default = remaining) and the account it went into → **Record payment**.
4. At zero the debt shows Settled. **Remind** shares a message through the share sheet.

## 11. Borrow money and repay
Same as 10 with "I borrowed". The payment form says **From** and reduces the chosen account.
Optional: set an **Installment** (amount and day of month) on Add debt. On each due date the payment is posted automatically from that account, a reminder fires the day before, and it stops at zero. It also shows in More → Recurring items.

## 12. Split a bill and collect
1. More → **Debts and splits** → **Split a bill** (the same button is on Debts).
2. Enter total, What for, Date, Category, Paid by (you + account, or another person), method, pick people, adjust amounts.
3. **Save split**: one expense for the user's own share (none if the user is not in the split), one debt per other person. If someone else paid, the expense has no account and one debt is owed to that person.
4. Later open **Split detail**: **Record payment** per person (partial allowed), **Share summary** to message everyone.

## 13. Paid on behalf (two ways)
- **I paid for someone:** Add → On behalf → mode 1 → Person, Amount, Paid from, Note → Save. Account drops, owed to you rises, no expense.
- **Holding money for them:** mode 2 → Person, Amount, Received into, Note → Save. Account rises, you owe them, no income. Paying it out is a payment against that debt.

## 14. Review auto-captured messages
1. Bell on Home (badge), or More → Allowed senders → **Review entries** (the link from Allowed senders is not drawn yet; Review entries has "Manage allowed senders" the other way).
2. For each card: **Add** (opens Add prefilled when no category was found), **Edit** (opens Add prefilled), or **Ignore**.
3. Nothing is saved until Add. Android collects on its own; iPhone entries arrive through the Shortcuts automation.

## 15. Back up and restore
1. More → **Backup and export** → **Back up now** → share/save sheet. The Last backup card updates.
2. On a new phone: install → Welcome → **Have a backup? Restore it** (or, after setup, More → Backup and export → **Restore from a file**) → confirm → app restarts at Home.
3. Restore of a file from a newer app version is refused with a message to update the app.

## 16. Export
1. Reports export icon or More → Backup and export.
2. Range, Format (CSV or PDF) → **Export report** → share sheet. (Pro; a free user is sent to Pro.)

## 17. Reports
1. **Reports** → Where it went; pick Week / Month / Year / Custom; use the arrows to move to earlier periods.
2. Tap a legend row → History filtered to that category and period.
3. **Trends** tab for six-month income vs spending, daily spending, average per day, percent saved.

## 18. Upgrade to Pro and restore
1. More → Pro banner (or hitting a free limit) → **Pro**.
2. **Unlock Pro** → store purchase sheet → success state → features unlock at once.
3. New phone or reinstall: **Restore purchase**.

## 19. Change settings
- Language: applies immediately.
- Month starts on: reports, budgets and Home month totals recalculate; a note says history is unchanged.
- Currency: changeable only until the first entry exists; after that the row is read-only (Delete all data and set up again to change it). Real multi-currency is Pro.
- Auto-lock, Change PIN, Daily reminder: in More (see GAPS for the unplanned rows).
- App lock: on → create PIN; off → ask PIN.

## 20. Hit a free limit
1. Add the 4th open loan or the 4th open split → **Pro** opens with the reason ("You have 3 open splits. Settle one or unlock Pro.").
2. Unlock, or go back; nothing is lost. Settling or deleting one frees a slot.

## 21. Manage accounts
1. More → **Accounts** → **Add account** → **New account**: pick Type (Cash, Bank, Card, Wallet), Name, Currency, Opening → **Save account**.
2. Tap an account → **Account detail**: Edit, **Adjust balance**, Archive, Delete (only with no entries).
3. Adjust balance: enter the real balance; the difference is saved as an Adjustment entry (not income or spending).

## 22. Manage categories
1. More → **Categories** → Expense or Income → **New category**: Expense/Income, Name, Inside (optional parent), Icon → **Save category**.
2. Tap a row → edit, archive, or delete. Deleting a category in use asks which category receives its entries first.

## 23. Edit or delete a debt or split
1. Debt detail or Split detail → "⋯" menu → Edit or Delete → confirm.
2. Deleting a split removes its own-share expense and debts; deleting a debt with payments asks whether to keep or remove the account entries.

## 24. Reset after a forgotten PIN
See flow 2 step 4. Erase removes the database, settings and PIN; a backup file can then be restored from Welcome.

## 25. Daily reminder (P2)
1. More → Reminders → switch on, choose time (default 8:00 pm) → permission explainer → system permission.
2. A local notification "Anything to log today?" opens Add.

## 26. Share to add (P4)
1. In another app share text to Pocketbook → the lock screen if on → **Add** opens prefilled using the quick-add rules; the user taps Save.

## 27. Capture setup (P4, Pro)
- Android: More → Allowed senders → **Add sender** → permission explainer → notification access (SMS only if added later) → sender name or number, account, optional default category → messages appear in **Review entries**. A sender's switch pauses reading.
- iPhone: More → Allowed senders shows the **On iPhone** steps (Shortcuts → New automation → Message → Run "Add to Pocketbook"); each message creates a review card.

## 28. Turn App lock on, off, or change the PIN
1. More → Security → **App lock** on → **Create a PIN** → **Confirm PIN** (no Skip here; Back cancels and the switch returns to off).
2. App lock off → ask for the PIN (or biometric) → lock removed and the stored PIN deleted.
3. Change PIN (row not drawn): verify with biometric if on, else the current PIN → Create a PIN → Confirm PIN.
4. **Face ID or fingerprint** switch: only enabled while App lock is on; turning it on shows the system prompt.

## Reachability check (every screen)
Welcome 1 · Lock 2 · Home (tab) · Add 3, 5, 6 · History (tab) 7 · Detail 7 · Accounts 6, 21 · Categories 22 · Budgets 8 · Recurring 9 · Review entries 14 · Reports 17 · Trends 17 · Debts 10, 11 · Debt detail 10 · Paid on behalf 13 · Split a bill 12 · Split detail 12 · More (tab) 19 · Backup and export 15, 16 · Pro 18, 20 · Create a PIN 1, 28 · Confirm PIN 1, 28 · Forgot PIN 2 · Allowed senders 14, 27 · New account 21 · New category 22 · New budget 8 · New recurring item 9 · Add debt 10, 11.
