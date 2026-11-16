# Pocketbook: screen-by-screen spec

Source of truth for what each of the 21 designed screens contains and does. Each entry lists every element, every option, every action, the states, and the rules behind it.

- **Design file:** the matching static page in `docs/design/screens/<Name>.html` (open `docs/design/screens/index.html`). Sample amounts there are examples only (US dollars); real values come from the user's data and currency.
- **Online (clickable):** https://claude.ai/artifact/McKtWDuTK1kgQKSnffzJrs
- **Design system:** https://claude.ai/artifact/JkVyy5F7v7a6TFdGPV2Tnc (tokens in `docs/design/tokens.json`, components in `docs/design/components/`)
- Things the design does not draw but the app needs are in `docs/GAPS.md`. Step-by-step journeys are in `docs/FLOWS.md`.

Phase column: P1 = Core, P2 = Insights, P3 = People and money, P4 = Advanced (see `CLAUDE.md`).

## Global rules (apply to every screen)

- **Layout:** 390 wide design, 20px side padding on both sides, 52px full-width buttons, 44px minimum tap target. Screens with the bottom bar leave 78px at the bottom.
- **Bottom bar:** Home, History, raised Add button, Reports, More. Active tab is green with a soft pill behind the icon. It appears on Home, History, Reports (both tabs), More, and on the list screens Accounts, Categories, Budgets and Debts, where **More** is highlighted and there is no Back arrow. Setup, Lock, Add, Detail, Debt detail, Split a bill, Split detail, Paid on behalf, Recurring, Review entries, Backup and Pro have a Back arrow (none on Setup and Lock) and no bar.
- **Money:** integer minor units, always with sign on transactions (−$4.50, +$1,400), thousands separators and decimals by locale and currency (0 decimals for JPY, 3 for KWD).
- **Colors:** green only for money in or owed to the user; red for money the user owes, overspending, delete; amber for warnings; icons are neutral tiles.
- **Dates:** relative for today and yesterday ("Today", "Yesterday"), otherwise day then month ("13 Nov"); with year when not the current year. The Detail screen uses the long form with weekday ("Sat, 14 Nov 2026"). Use one order everywhere (the static Home sample shows "Nov 1"; that is a sample slip).
- **Destructive actions** always ask for confirmation with Cancel and a red action.
- **Success feedback:** a short toast ("Saved", "Deleted", "Backed up") and return to the previous screen.
- **Empty states:** one line that says what goes here and a primary button to add the first item.
- **Text size and RTL:** layouts must survive large system text and right-to-left languages.
- **Phase gating:** several P1 screens draw elements from later phases. Build them in their own phase and hide them until then: Home quick add line (P2), Home debt boxes (P3), Home bell (P4), Add's On behalf link (P3), Detail's Duplicate (P1) and Repeats row (P2), Settings rows for Recurring and Budgets (P2), Debts and Splits (P3), Pro banner and Allowed senders (P4), Reports tab and Reports export icon (P2). In P1 the Reports tab opens a plain "Reports arrive in a later update" placeholder; it is removed from release builds until P2 is done.
- **Free limits and Pro:** read through the `Entitlements` service (SPEC). A blocked action opens Pro with the reason line ("You have 3 open splits. Settle one or unlock Pro.").
- **Notifications:** the first time a feature needs them (budget alert, reminder, backup reminder) show a one-line explanation, then the system permission. If denied, features still work and Settings shows "Notifications are off" with a link to system settings.
- **Sample data in the static screens is illustrative only** and is not always internally consistent (see the "Known sample slips" list at the end).

---

## 1. Welcome and setup (`Main.html`, route `/setup`, P1)

First launch only. Gets the app usable in about 20 seconds.

**Elements, top to bottom**
- Logo tile, title "Set up Pocketbook in 20 seconds", subtitle "Everything stays on your phone. No account needed."
- **Language** field: opens a list of supported languages (English, Urdu, Arabic, Spanish, French and more as added). Default: device language.
- **Currency** field: opens a searchable list of ISO currencies with symbol (default: from device region). Shown as "$ · USD".
- **Month starts** field: day 1 to 28. Default 1. Helper text explains it is for people paid on another day.
- **Cash on hand** field: typed amount, becomes the opening balance of the default Cash account. Default 0.
- Helper text: "Set the month start to your payday if it is not the 1st. You can change all of this later in Settings."
- **Start tracking** (primary).
- **Have a backup? Restore it** text link under the button (not yet drawn, see GAPS). Opens the file picker, then the same confirm and result as Backup and export. After a successful restore, setup is skipped and the app opens on Home.

**On Start tracking:** save settings, create the Cash account with the opening balance, seed default categories, then offer "Create a PIN" (see GAPS), then go to Home.
**Validation:** opening balance must be zero or positive and fit the currency's decimals. Default Cash account name is "Cash", type Cash. Main Bank and Mobile Wallet in the designs are samples, not seeded.
**States:** none after completion; the screen never shows again.

## 2. App lock (`Lock.html`, route `/lock`, P1)

Shown on launch and when returning to the app after the auto-lock time, if App lock is on.

**Elements**
- Logo tile, "Enter your PIN", four dots (filled as digits are typed).
- Keypad 1 to 9, 0, and a delete key.
- "Use Face ID or fingerprint" link. The biometric prompt opens automatically first when biometrics are on.

**Behavior**
- PIN is 4 digits. On the 4th digit it checks automatically.
- Right: go to the screen the user was on (or Home on cold start).
- Wrong: dots clear with a shake and "Wrong PIN. Try again." Lockout waits are 30 seconds after 5 wrong tries, 1 minute after 10, 5 minutes after 15, then 15 minutes for every further 5. The count and the wait survive killing the app and reset on a correct PIN. During a wait the keypad is disabled and shows a countdown.
- A **Forgot PIN?** link under the keypad (not drawn, see GAPS) offers biometric if on, otherwise Reset app.
- While the app is in the app switcher, a plain cover hides balances (and on Android the window is marked secure).
- Biometric success unlocks; biometric failure falls back to the keypad.
- Auto-lock time is Immediately, 1 minute or 5 minutes (default 1 minute), set in More.
- The PIN is stored only as a salted hash in the device keystore; never in the database or backup.

## 3. Home (`Home.html`, route `/`, P1)

**Elements**
- Greeting by time of day (Good morning, afternoon, evening) and the current month and year.
- **Bell button** with a red count badge = number of entries waiting in Review entries; hidden when 0. Tap goes to Review entries. (P4)
- **Quick add** line (P2): type something like "lunch 4.50 cash". It finds the amount, the account by name and the category by remembered note, then opens Add prefilled for one-tap confirmation. If it cannot read an amount it says so and opens Add empty. (Detailed parsing in FLOWS.)
- **Balance card:** Total balance (sum of all non-archived accounts, in the base currency), and two boxes: Income and Spent for the current month (month respects the month start day). Not tappable. Income is `income` entries only and Spent is `expense` entries only, so transfers, opening balances, adjustments and debt movements never move them.
- **Debt boxes** (P3): "Owed to you" (green) and "You owe" (red) totals, both tap to Debts.
- **Recent:** the 3 newest entries dated today or earlier (by date, then time added), with "See all" to History. Each row opens Detail or the parent split/debt.
- Bottom bar.

**States**
- No transactions yet: Recent shows "Nothing yet. Add your first expense." with an Add button.
- Zero debts: the boxes show $0 and still tap to Debts.

## 4. Add transaction (`Add.html`, route `/transactions/create`, P1)

Also used to edit (prefilled; title "Edit transaction"; button "Save changes").

**Elements**
- Back arrow and **On behalf** link (opens Paid on behalf).
- **Segmented control:** Expense, Income, Transfer.
- **Amount:** large number with a cursor, built from the on-screen keypad (1 to 9, decimal point, 0, delete). Decimal point is hidden for zero-decimal currencies. Symbol follows the selected account's currency.
- **Category** field: picker listing categories of the chosen type with subcategories, a search box and "New category". Not shown for Transfer.
- **Account** field: picker of accounts with balances. Default is the account used last for this type, else Cash. For Transfer this becomes **From** and a second **To** field appears; they must differ.
- **Date** field: date picker, default today; future dates allowed (the entry is marked "Upcoming" and counts from its date).
- **Note** field: free text, one line.
- **Tags** (optional): free-text chips, shown on Detail; not drawn on Add yet (GAPS). Hidden behind a small "Add tags" row until used.
- Save button text follows the type: "Save expense", "Save income", "Save transfer".

**Rules**
- Amount must be greater than 0; category is required for expense and income; accounts must differ for transfer.
- Expense lowers the account balance, income raises it, transfer moves it between accounts and is never counted as spending or income.
- After save: toast, back to the previous screen, balances update.
- Remembers the last category used for each note text (case-insensitive, trimmed) to prefill next time.
- A foreign-currency account (Pro) changes the amount symbol; nothing is converted.
- In edit mode the On behalf link is hidden. Changing the type of an existing entry between expense, income and transfer is allowed; entries owned by a split or debt are not edited here.
- Leaving with unsaved input asks "Discard this entry?" (Keep editing / Discard).

## 5. Transactions / History (`History.html`, route `/transactions`, P1)

**Elements**
- Title "Transactions".
- **Search** field (placeholder "Search notes, people, amounts"; also matches category names and tags): matches notes, category names, tags, people names and amounts (typing 4.5 matches 4.50).
- **Filter chips:** All, Expense, Income, Transfer, and a month chip ("Nov 2026") that opens a month picker. The chip row scrolls sideways and keeps 20px at the end. More filters (account, category) are in the month chip's sheet (see GAPS).
- Grouped list by day. Each day header shows the date label and that day's net total = income − expense (e.g. "−$34.50"); transfers, adjustments and debt movements are left out of the total but their rows still appear. Entries dated in the future are grouped under an "Upcoming" header at the top.
- A month chip selects one budget month; the sheet it opens also holds the account and category filters, and "All time".
- Rows (icon tile, name, "Category · Account", signed amount) tap to Detail. The row shows the top-level category. Transfers read "Transfer · Cash → Main Bank" and show no sign colour. Debt rows read "Loan · Ali Raza" or "Paid back · Ali Raza". A split share paid by someone else reads "Category · Paid by Ali".

**States**
- No results: "No transactions match." with a Clear filters button.
- Empty app: same as Home empty state.
- Lists load in pages of 50 as the user scrolls.

## 6. Transaction detail (`Detail.html`, route `/transactions/{id}`, P1)

**Elements**
- Back, title "Details".
- Hero: icon tile, "Category › Subcategory", signed amount, type pill (Expense / Income / Transfer).
- Detail card rows: **Account** (for a transfer: "From → To"), **Date**, **Note**, **Tags**, **Added** (how it got here, from `source`: Typed, Typed in quick add, Shared in, Recurring, Auto-capture, Split, Debt payment), and for a split-share-from-someone-else the **Paid by** row.
- **Edit** (opens Add prefilled), **Duplicate** (opens Add prefilled with today's date; the one-tap repeat) and **Delete** (confirm: "Delete this entry? Balances will update." Cancel / Delete).

**Rules**
- Entries created by a split or a debt payment open their parent (split or debt) for editing; Delete is offered there so the ledger stays consistent.
- Recurring entries show an extra row "Repeats monthly" linking to the recurring item. Editing one posted entry changes only that entry.
- Adjustment entries show "Adjustment" in the type pill and can only be deleted, not edited.
- Not drawn: Duplicate button and the pills for debt, adjustment and upcoming entries (GAPS).

## 7. Accounts (`Accounts.html`, route `/accounts`, P1)

**Elements**
- Title "Accounts" with a transfer icon button top right (opens Add in Transfer mode).
- "Total across accounts" with the big total.
- One card per account: type icon, name, "type · currency", current balance. (The static Cash card shows "Wallet" instead of "Cash · USD"; use "type · currency" on every card.)
- **Add account** button.
- Helper text about transfers.

**Actions**
- Tap an account → Account detail (see GAPS): its transactions, Edit (name, type, opening balance), Adjust balance (asks for the real balance, creates an `adjustment` entry for the difference; not income or spending), Archive (hides it and removes it from totals; confirm if the balance is not zero; history kept) and Delete (only when it has no entries).
- Add account form: name, type (Cash, Bank, Card, Mobile wallet), currency, opening balance. Credit cards may go negative.
- Free tier: single currency. Multi-currency accounts are Pro (P4).

## 8. Categories (`Categories.html`, route `/categories`, P1)

**Elements**
- Title, segmented **Expense / Income**.
- Rows: icon tile, name, subcategories line ("Groceries, Dining, Tea") or "N subcategories", this month's total, chevron.
- **New category** button.

**Defaults seeded at setup**
- Expense: Rent and home (Rent, Maintenance), Food (Groceries, Dining, Tea), Bills (Electricity, Internet, Mobile), Transport (Fuel, Rides), Shopping (Clothes, Household), Health (Doctor, Medicine), Education (Fees, Books), Fun (Outings, Subscriptions), Other.
- Income: Salary, Freelance, Gifts, Other.

**Actions**
- Tap → Category edit (see GAPS): name, icon (from a fixed set), parent (top-level only; a category with subcategories cannot become a subcategory), archive. The designed Rent and home row has 2 subcategories; the names above are the defaults. This month's total on a row includes its subcategories. Deleting a category in use asks to move its entries to another category first.

## 9. Budgets (`Budgets.html`, route `/budgets`, P2)

**Elements**
- Title "Budgets".
- **Monthly budget card:** spent of limit ("$963.50 of $1,100"), "15 days left", progress bar, and a sentence: "88% used. $136.50 left, about $9.10 a day."
- **Category budget cards:** icon, name, "$243 of $250", a pill (percent used, or "Over by $12"), progress bar.
- **New budget** button.

**Rules**
- Bar color: accent under 80%, amber 80–100%, red over 100% (bar capped at full width); status is always also in words.
- A budget is monthly and follows the month start day. Overall budget is separate from category budgets (it is not the sum of them). One budget per category; the form hides categories that already have one. A category budget includes its subcategories. No rollover.
- The percent is rounded to the nearest whole number but shows 99% until spending actually reaches the limit. At exactly 100% the bar is amber and the pill says "100% used"; "Over by" starts above the limit.
- Alerts: local notification at 80% and 100% (once each per month).
- New budget form: Overall or a category, monthly limit. Edit and delete by tapping a card.

## 10. Recurring (`Recurring.html`, route `/recurring`, P2)

**Elements**
- Back, title "Recurring", explanation: "Posted automatically when due. If the app was closed, missed ones are added next time you open it."
- Cards: icon, name, schedule ("Monthly on the 5th · reminder 1 day before"), amount (+ green for income), and an on/off switch (off shows "paused").
- **New recurring item** button.

**Rules**
- Frequencies: daily, weekly, monthly, yearly; day of month (1–31; on short months it posts on the last day) or weekday; optional reminder N days before. Types: expense, income, or a payment on one of my debts (installment).
- Transfers cannot be recurring in v1.
- On app open (and by background task where available) every due occurrence is posted, oldest first, with date = due date, each exactly once (a due date can never post twice). If the account was archived the item is paused with a note. The catch-up also runs after a restore.
- Item form (not drawn): type, amount, name/note, category, account, frequency, day, start date, optional reminder, and for installments the debt.
- Switch off pauses without deleting. Tap a card to edit; delete asks for confirmation and keeps already-posted entries.

## 11. Review entries (`Pending.html`, route `/captures`, P4, Pro)

**Elements**
- Back, title "Review entries", note: "Nothing is saved until you tap Add. Only messages from senders you allow are read, on this phone."
- Cards: sender with icon, received time, the raw message in a shaded box, parsed amount, "merchant · category", a "Found by sender rule" pill, and buttons **Add**, **Edit**, **Ignore**.
- **Manage allowed senders** button.

**Actions**
- Add saves exactly as parsed. If no category was found ("Uncategorized") Add opens Add prefilled instead, because a category is required. Edit opens Add prefilled. Ignore removes it from the list (kept hidden so the same message is not offered again).
- Each allowed sender is tied to one account (the account the message is about) and an optional default category; the card shows that account. The same message is never offered twice.
- Android reads notifications from allowed senders (SMS is an optional later addition, only if the store allows it); iPhone gets entries from a Shortcuts automation. See `docs/SPEC.md`.
- Empty state: "No messages waiting." Reached from the bell when the badge shows, and from More → Allowed senders → Review entries (the bell is hidden at 0).

## 12. Reports: where it went (`ReportsOverview.html`, route `/reports`, P2)

**Elements**
- Title "Reports" with an export icon (opens Backup and export; Pro).
- Tabs: **Where it went** | **Trends**.
- Period chips: Week, Month, Year, Custom (Custom opens a date-range picker). Previous/next arrows beside the heading ("Spent in Nov") move to the earlier or later period (not drawn, see GAPS); the next arrow is disabled at the current period.
- Donut chart with the total spent in the centre, one segment per category.
- Legend list: color swatch, category, percent, amount. Top 5 top-level categories plus a roll-up row for the rest. Name the roll-up "Everything else" so it is not confused with the category "Other" (the static design says "Other"). Percentages are adjusted so the legend adds to 100 (the static sample adds to 99).

**Rules**
- Counts expenses only (own share for splits). Transfers, repayments and held funds are excluded.
- Tapping a legend row opens History filtered to that category and period.
- Chart has a text alternative (the legend). Empty period: "No spending in this period."

## 13. Reports: trends (`ReportsTrends.html`, route `/reports/trends`, P2)

**Elements**
- Same header, tabs and period chips.
- Two stat boxes: **Average per day** and **Saved this month** (percent of income). The second label follows the period ("Saved this week/month/year", "Saved in this range"). Average per day divides by days elapsed so far in the current period, or all days in a past one.
- **Income vs spending** grouped bars for the last 6 months with a legend.
- **Daily spending** columns for the selected month.

**Rules**
- Saved percent = (income − spending) ÷ income; shown as 0% if income is 0.
- Period chips drive the stats and the daily chart; the 6-month chart always ends at the current month.

## 14. Debts (`Debts.html`, route `/debts`, P3)

**Elements**
- Title "Debts"; two summary boxes **Owed to you** (green) and **You owe** (red).
- Rows are one per debt, not per person. Sections hide when empty. A debt past its due date shows a red "Overdue" pill instead of the amber due pill.
- Section **Owed to you**: avatar, name, detail ("Loan · paid back $30 of $80", "Paid on his behalf · phone repair", "Dinner split · 12 Nov"), remaining amount, optional due pill ("Due 20 Nov", amber).
- Section **You owe**: same pattern (personal loan with installment, borrowed from a person).
- **Add debt** button.

**Actions**
- Tap a loan, installment, on-behalf or held row → Debt detail. Tap a split row → Split detail.
- A **Split a bill** secondary button sits next to Add debt (not drawn, see GAPS). It is the only entry to Split a bill besides the More shortcut.
- Add debt form: direction (I lent / I borrowed), person or lender, amount, optional due date, optional installment amount (a monthly payment; creates a recurring debt payment), note, and "Did the money move through an account?" with an account picker (if yes it creates a `debt_out`/`debt_in` entry that is not an expense or income; if no, only the debt is recorded).
- Free tier: 3 open loans and 3 open splits (SPEC defines what counts); the 4th shows Pro with the reason.
- Reminders: if a due date is set, a local notification on the due date at 9:00.

## 15. Debt detail (`DebtDetail.html`, route `/debts/{id}`, P3)

**Elements**
- Back, avatar, name, "Owes you $50" (or "You owe Ali $50").
- Progress card: "Paid back $30 of $80" with a bar.
- History card: "Lent 12 Oct · Cash −$80", "Paid back 28 Oct · Cash +$20", etc.
- **Record a payment** block: **Amount** (default = remaining) and **Into** account (for "I owe" this reads **From**).
- **Record payment** (primary) and **Remind** (shares a text through the share sheet, for example WhatsApp).

**Rules**
- A payment cannot exceed what remains. At zero the debt becomes "Settled" and the form is hidden.
- Payments move money in the chosen account and are not income or expense.
- A "⋯" menu (not drawn, see GAPS) offers Edit (due date, note, installment) and Delete debt. Deleting asks for confirmation and, if payments exist, whether to keep or remove their account entries.
- The History card also lists the opening movement ("Lent 12 Oct · Cash −$80"). A debt with no account shows no account in its rows.

## 16. Paid on behalf (`OnBehalf.html`, route `/behalf`, P3)

**Elements**
- Back, title, segmented: **I paid for someone** | **Holding money for them**.
- Fields: **Person** (pick or add), **Amount**, **Paid from** (Mode 2: **Received into**), **Note**.
- Call-out: "Not your expense. $49 leaves Cash and shows as owed to you by Bilal. When he pays you back, record it as a repayment into any account."
- Second note explains the other mode: money sent to you to pay out is not income and clears when you pay it out.
- **Save**.

**Rules**
- Both modes use the same Person picker with Add person. Amount must be greater than 0; the account must exist.
- Mode 1: account goes down, a debt "owed to you" of kind on_behalf is created, no expense.
- Mode 2: account goes up, a debt "you owe" of kind held is created, no income; paying it out is a payment from an account against that debt.

## 17. Split a bill (`SplitNew.html`, route `/splits/create`, P3)

**Elements**
- Back, "Bill total" large.
- Fields: **What for**, **Paid by** ("You · Cash" or another person; if another person, the user's share becomes a debt "you owe").
- Method segmented: **Equal, Exact, Percent, Shares**.
- People checklist (the user plus others) with each person's amount; a check box per person; **Add person**.
- Summary: "Your share is $15. The other three owe you $45, tracked until they pay."
- Missing in the design: **Date** (default today) and **Category** (default Fun › Outings; used for the user's own share). Add both as Field rows under What for. (GAPS)
- **Save split**.

**Rules**
- Equal: total ÷ people, remainder cents go to the first people. Exact: amounts must add up to the total. Percent: must add up to 100. Shares: weights.
- Save creates one expense for the user's own share only (the Category field) plus a debt per other person. If someone else paid, the expense has no account and the user gets one debt owed to that payer (SPEC). If the user's box is unticked there is no expense.
- Percent must total 100 and Exact must total the bill; Save stays disabled with the difference shown ("$2 left to assign"). At least one other person is required.
- Editing a split rebuilds its expense and debts; payments already recorded block changing the people list (confirm).
- Free tier limit of 3 splits.

## 18. Split detail (`SplitDetail.html`, route `/splits/{id}`, P3)

**Elements**
- Back, date and payer line, title.
- Progress card: "Collected from others $20 of $45", "$25 still owed. Your own share: $15."
- People card: avatar, name, status line ("Paid in full on 13 Nov", "Paid $5 owes $10", "Not paid yet"), amount, status pill (Paid green, Part paid amber, Owes red).
- **Record payment** (choose person, amount, account) and **Share summary** (share sheet text listing who owes what).

**Rules**
- Each payment updates that person's debt and the progress. Fully paid splits show "Settled".
- Edit or delete the split from a menu (see GAPS); deleting removes the linked expense and debts after confirmation.

## 19. More / Settings (`Settings.html`, route `/settings`, P1)

**Elements**
- Title "More".
- **Pocketbook Pro** banner (gold star on dark): "One payment, yours for life" → Pro.
- **General:** Language, Currency, Month starts on (day 1–28). There is no Theme row in v1 (light only); the static screen draws one, leave it out.
- **Security:** App lock switch, Face ID or fingerprint switch.
- **Money:** Recurring items, Budgets, Categories, Accounts.
- **Data and capture:** Backup and export, Allowed senders (count, P4).
- **Not drawn, to add (GAPS):** Debts and Splits shortcuts (P3), Reminders (daily "log your day"), Auto-lock time, Change PIN, About and help, Delete all data.

**Rules**
- **Currency is locked once any entry exists** (any transaction, debt or split). Until then it can be changed freely. When locked the row is read-only, shows no chevron, and a line under it says "Currency can't be changed once you have entries." The only way to change it is Delete all data and set up again (back up first). Pro (P4) adds accounts in other currencies; the base currency stays as chosen.
- Turning App lock on asks the user to create a PIN; turning it off asks for the PIN.

## 20. Backup and export (`Backup.html`, route `/backup`, P1 backup, P2 export)

**Elements**
- Back, title.
- **Last backup** card: time, size, a "Safe" pill (green) when the last backup is under 14 days old; an amber "Old" pill after that; an amber "No backup yet" pill if never. Time and size come from the last backup made on this phone.
- **Plain-file note** under the buttons (not yet drawn): "Backups are not encrypted. Anyone with the file can read your data, so keep it somewhere private."
- **Back up now**: creates one file (all data, no PIN) and opens the share/save sheet.
- **Restore from a file**: file picker, then confirm "This replaces everything on this phone with the backup." Cancel / Replace. After success the app restarts at Home.
- **Weekly backup reminder** switch (day and time, default Sunday 9 pm).
- **Export** section: **Range** (month picker or custom), **Format** (CSV spreadsheet or PDF report), **Export report** (Pro) which opens the share sheet.

**Rules**
- The file is not encrypted, so the card says "Anyone with this file can read your data." (optional password is a possible later release). File name `pocketbook-YYYY-MM-DD`. Backup is a SQLite copy with a `schema_version`; restoring an older file migrates it up; restore refuses files from a newer app version with a clear message.
- Restore never touches the PIN or lock setting. After restore, reminders and notifications are rescheduled and recurring catch-up runs.
- Free users see Export report with a small "Pro" tag; tapping it opens Pro (not drawn). Backup and restore are always free.
- Errors (cannot read file, not a Pocketbook file, file from a newer version, disk full) show one line with Try again.

## 21. Pocketbook Pro (`Pro.html`, route `/pro`, P4)

**Elements**
- Back, logo, "Pocketbook Pro", "Pay once. Keep it forever. No subscription."
- Feature list with checks (the static copy says "Auto-capture from SMS and Shortcuts"; change it to "notifications and Shortcuts" until SMS is approved): Unlimited splits and debts (free includes 3 of each); Auto-capture from SMS and Shortcuts; Multiple currencies; CSV and PDF export; Home-screen widget.
- Note: "Free keeps everything else: tracking, budgets, reports, backup."
- **Unlock Pro · [ONE-TIME PRICE]** and **Restore purchase**.

**Rules**
- One non-consumable purchase through the App Store and Google Play. Price comes from the store, shown in local currency.
- Restore purchase re-checks the store entitlement. After purchase show a success state and unlock features immediately (see GAPS).
- Pro is checked locally from the store receipt; the app never needs a server.
- Opened from a free limit it shows the reason line above the feature list. Restore purchase shows "Pro restored" or "No purchase found for this store account."

---

## Known sample slips in the static screens (do not copy)

- History: the "13 Nov" header says +$40 but its two entries (+$40 and −$12) net +$28.
- History: Groceries is shown under Shopping, but Groceries is a subcategory of Food by default.
- Home: Recent shows Salary from 1 Nov while newer entries exist in History; Recent is the 3 newest.
- Reports: legend percentages add to 99.
- Reports: "Other" in the legend is a roll-up, not the category Other.
- Accounts: the Cash card says "Wallet" instead of "Cash · USD".
- Debts: lists 6 debts, more than the 3 free loans; it is a Pro or sample view. Split shares do not count.
- Trends: stat label is "Saved this month" even when the chips can change the period.
- Settings: draws a Theme row; v1 is light only, so omit it.
