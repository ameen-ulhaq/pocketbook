# Pocketbook: user flows

Each flow lists the steps in order, the screen at each step, and the rule that matters. Screen names match `docs/SCREENS.md`.

## 1. First launch
1. Open app → **Welcome and setup**.
2. Choose Language, Currency, Month start day, Cash on hand.
3. Tap **Start tracking**. App seeds the Cash account and default categories.
4. Offer "Create a PIN" (optional, can skip). If created, ask whether to use Face ID or fingerprint.
5. Land on **Home** (empty state).

## 2. Lock and unlock
1. App launch or return after the auto-lock time → **App lock**.
2. Biometric prompt opens first; on failure or cancel use the PIN keypad.
3. Right PIN → previous screen. Wrong PIN → shake, message, delay after 5 tries.

## 3. Add an expense
1. Tap the raised **Add** button (or Add from Home empty state).
2. **Add** opens on Expense. Type the amount on the keypad.
3. Choose Category (picker), Account (default: last used), Date (default today), Note (optional).
4. **Save expense** → toast → back. Balance, Recent, budgets and reports update.

## 4. Quick add from a line of text
1. On **Home**, type "lunch 4.50 cash" in the quick add line and submit.
2. App reads amount (4.50), account ("cash"), and note ("lunch"); category comes from the last category used with that note, else asks.
3. **Add** opens prefilled; the user taps Save (or edits).
4. If no amount is found: "I couldn't find an amount" and Add opens empty.

## 5. Add income
1. Add → switch to **Income**.
2. Amount, Category (income categories), Account, Date, Note → **Save income**.

## 6. Transfer between accounts
1. Accounts → transfer icon, or Add → **Transfer**.
2. Amount, **From** account, **To** account (must differ), Date, Note → **Save transfer**.
3. Neither account's income or spending changes; only balances move.

## 7. Find, edit and delete an entry
1. **History** → search or filter (type chips, month) → tap a row → **Detail**.
2. **Edit** → Add prefilled → **Save changes**; or **Delete** → confirm → toast → back.
3. Entries made by a split or debt payment open their parent instead.

## 8. Set a budget
1. More → **Budgets** → **New budget**.
2. Choose Overall or a category, enter the monthly limit → Save.
3. As spending grows the bar goes accent → amber (80%) → red (over); local notifications fire at 80% and 100%.

## 9. Recurring item
1. More → **Recurring** → **New recurring item**.
2. Type (income/expense), amount, category, account, frequency, day, optional reminder → Save.
3. When due (checked on app open and by background task) the entry is posted automatically with the due date; the switch pauses it.

## 10. Lend money and get repaid
1. **Debts** → **Add debt** → I lent, person, amount, due date, account the money left from.
2. Debt appears under Owed to you. Later: tap it → **Debt detail**.
3. Enter the amount received (default = remaining) and the account it went into → **Record payment**.
4. At zero the debt shows Settled. **Remind** shares a message through the share sheet.

## 11. Borrow money and repay
Same as 10 with "I borrowed". The payment form says **From** and reduces the chosen account.

## 12. Split a bill and collect
1. Home or Debts → **Split a bill** (entry point: Debts → Add debt → Split, and a shortcut in More).
2. Enter total, What for, Paid by (you + account), method, pick people, adjust amounts.
3. **Save split**: one expense for the user's own share, one debt per other person.
4. Later open **Split detail**: **Record payment** per person (partial allowed), **Share summary** to message everyone.

## 13. Paid on behalf (two ways)
- **I paid for someone:** Add → On behalf → mode 1 → Person, Amount, Paid from, Note → Save. Account drops, owed to you rises, no expense.
- **Holding money for them:** mode 2 → Person, Amount, Received into, Note → Save. Account rises, you owe them, no income. Paying it out is a payment against that debt.

## 14. Review auto-captured messages
1. Bell on Home (badge) or More → Allowed senders → **Review entries**.
2. For each card: **Add**, **Edit** (opens Add prefilled), or **Ignore**.
3. Nothing is saved until Add. Android collects on its own; iPhone entries arrive through the Shortcuts automation.

## 15. Back up and restore
1. More → **Backup and export** → **Back up now** → share/save sheet.
2. On a new phone: install, skip setup, More → Backup and export → **Restore from a file** → confirm → app restarts at Home.

## 16. Export
1. Reports export icon or More → Backup and export.
2. Range, Format (CSV or PDF) → **Export report** → share sheet. (Pro.)

## 17. Reports
1. **Reports** → Where it went; pick Week / Month / Year / Custom.
2. Tap a legend row → History filtered to that category and period.
3. **Trends** tab for six-month income vs spending, daily spending, average per day, percent saved.

## 18. Upgrade to Pro and restore
1. More → Pro banner (or hitting a free limit) → **Pro**.
2. **Unlock Pro** → store purchase sheet → success state → features unlock at once.
3. New phone or reinstall: **Restore purchase**.

## 19. Change settings
- Language, Theme: apply immediately.
- Month starts on: reports, budgets and Home month totals recalculate; a note says history is unchanged.
- Currency: display change only (warn); real multi-currency is Pro.
- App lock: on → create PIN; off → ask PIN.
