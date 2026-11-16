# Pocketbook: screen-by-screen spec

Source of truth for what each of the 39 designed screens contains and does. Each entry lists every element, every option, every action, the states, and the rules behind it.

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
- **Phase gating:** several P1 screens draw elements from later phases. Build them in their own phase and hide them until then: Home quick add line (P2), Home debt boxes (P3), Home bell (P4), Add's On behalf link (P3), Detail's Duplicate (P1) and Repeats row (P2), Settings rows for Recurring and Budgets (P2), Debts and Splits (P3), Pro banner and Allowed senders (P4), Reports tab, period arrows and Reports export icon (P2), Pro tag on Export (P4). In P1 the Reports tab opens a plain "Reports arrive in a later update" placeholder; it is removed from release builds until P2 is done.
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
- **Have a backup? Restore it** text link under the button. Opens the file picker, then the same confirm and result as Backup and export, then Home (setup is skipped). Static design: the link opens the Backup screen; build it as a direct file picker, because there is no More screen to go Back to before setup.

**On Start tracking:** save settings, create the Cash account with the opening balance, seed default categories, then open Create a PIN (screen 22; it can be skipped), then go to Home.
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
- A **Forgot PIN?** link under the biometric link opens Forgot PIN (screen 24). It stays available during a lockout wait.
- While the app is in the app switcher, a plain cover hides balances (and on Android the window is marked secure).
- Biometric success unlocks; biometric failure falls back to the keypad.
- Auto-lock time is Immediately, After 1 minute (default, marked Recommended), After 5 minutes or After 15 minutes, chosen on screen 32.
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
- **Tags** row under Note (drawn): free-text chips, typed with no chevron, optional; shown on Detail. Several tags are separated by commas; tags are created on first use and suggested afterwards.
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
- Duplicate sits beside Edit in one row (drawn); Delete is full width below it. Not drawn: the pills for debt, adjustment and upcoming entries (GAPS).

## 7. Accounts (`Accounts.html`, route `/accounts`, P1)

**Elements**
- Title "Accounts" with a transfer icon button top right (opens Add in Transfer mode).
- "Total across accounts" with the big total.
- One card per account: type icon, name, "type · currency", current balance. (The static Cash card shows "Wallet" instead of "Cash · USD"; use "type · currency" on every card.)
- **Add account** button.
- Helper text about transfers.

**Actions**
- Tap an account → Account detail (see GAPS; the static design has no link from a card): its transactions, Edit (name, type, opening balance), Adjust balance (asks for the real balance, creates an `adjustment` entry for the difference; not income or spending), Archive (hides it and removes it from totals; confirm if the balance is not zero; history kept) and Delete (only when it has no entries).
- Add account form: screen 26 (name, type, currency, opening balance). Credit cards may go negative.
- Free tier: single currency. Multi-currency accounts are Pro (P4).

## 8. Categories (`Categories.html`, route `/categories`, P1)

**Elements**
- Title, segmented **Expense / Income**.
- Rows: icon tile, name, subcategories line ("Groceries, Dining, Tea") or "N subcategories", this month's total, chevron.
- **New category** button (opens screen 27).

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
- New budget form: screen 29 (Whole month or One category, monthly limit, 80%/100% warning switch). Edit and delete by tapping a card (edit mode and Delete not drawn).

## 10. Recurring (`Recurring.html`, route `/recurring`, P2)

**Elements**
- Back, title "Recurring", explanation: "Posted automatically when due. If the app was closed, missed ones are added next time you open it."
- The **New recurring item** button opens screen 28.
- Cards: icon, name, schedule ("Monthly on the 5th · reminder 1 day before"), amount (+ green for income), and an on/off switch (off shows "paused").
- **New recurring item** button.

**Rules**
- Frequencies: daily, weekly, monthly, yearly; day of month (1–31; on short months it posts on the last day) or weekday; optional reminder N days before. Types: expense or income. A debt installment is a third, internal type (payment on a debt). It is created from Add debt (screen 30), not from this form, and appears in this list with the debt's person as its name.
- Transfers cannot be recurring in v1.
- On app open (and by background task where available) every due occurrence is posted, oldest first, with date = due date, each exactly once (a due date can never post twice). If the account was archived the item is paused with a note. The catch-up also runs after a restore.
- Item form: screen 28 (New recurring item). Tapping a card opens the same form in edit mode.
- Switch off pauses without deleting. Tap a card to edit; delete asks for confirmation and keeps already-posted entries.

## 11. Review entries (`Pending.html`, route `/captures`, P4, Pro)

**Elements**
- Back, title "Review entries", note: "Nothing is saved until you tap Add. Only messages from senders you allow are read, on this phone."
- Cards: sender with icon, received time, the raw message in a shaded box, parsed amount, "merchant · category", a "Found by sender rule" pill, and buttons **Add**, **Edit**, **Ignore**.
- **Manage allowed senders** button (opens screen 25).

**Actions**
- Add saves exactly as parsed. If no category was found ("Uncategorized") Add opens Add prefilled instead, because a category is required. Edit opens Add prefilled. Ignore removes it from the list (kept hidden so the same message is not offered again).
- The same message is never offered twice. The drawn Add sender form has no account or category row, so a card uses the account last used and no category (GAPS: add an Account row).
- Android reads notifications from allowed senders (SMS is an optional later addition, only if the store allows it); iPhone gets entries from a Shortcuts automation. See `docs/SPEC.md`.
- Empty state: "No messages waiting." Reached from the bell when the badge shows, and from More → Allowed senders → Review entries (the bell is hidden at 0).

## 12. Reports: where it went (`ReportsOverview.html`, route `/reports`, P2)

**Elements**
- Title "Reports" with an export icon (opens Backup and export; Pro).
- Tabs: **Where it went** | **Trends**.
- Period chips: Week, Month, Year, Custom (Custom opens a date-range picker). Below them a row with previous and next arrows around the period label ("November 2026"; Week shows the date range, Year shows "2026", Custom shows its range with no arrows). The next arrow is disabled (dimmed) at the current period. The donut centre keeps "Spent in Nov".
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
- Rows are one per debt, not per person. Sections hide when empty. A debt past its due date shows a red "Overdue" pill instead of the amber due pill, and the detail line reads "Borrowed 2 Nov · was due 10 Nov" (drawn).
- Section **Owed to you**: avatar, name, detail ("Loan · paid back $30 of $80", "Paid on his behalf · phone repair", "Dinner split · 12 Nov"), remaining amount, optional due pill ("Due 20 Nov", amber).
- Section **You owe**: same pattern (personal loan with installment, borrowed from a person).
- **Add debt** button.

**Actions**
- Tap a loan, installment, on-behalf or held row → Debt detail. Tap a split row → Split detail.
- A **Split a bill** secondary button sits next to Add debt (drawn) and opens Split a bill. The other entry is More → Debts and splits, which opens this screen.
- Add debt form: screen 30. Direction (I lent / I borrowed), person (labelled Lender for I borrowed), amount, date, due date, account (or "No account"), installment (borrowed only), note. If an account is chosen it creates a `debt_out`/`debt_in` entry that is not an expense or income; with "No account" only the debt is recorded.
- Free tier: 3 open loans and 3 open splits (SPEC defines what counts); the 4th shows Pro with the reason. Adding a debt (screen 30) or a split counts here.
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
- **Date** (default today) and **Category** (default Food › Dining in the sample; the user's own share uses it) are Field rows under Paid by (drawn).
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
- **General:** Language, Currency, Month starts on (day 1–28), **Reminders** (value "On" when any reminder is on, else "Off"; opens screen 33). There is no Theme row in v1 (light only).
- **Security:** App lock switch (on opens Create a PIN, screen 22; off asks for the PIN), Face ID or fingerprint switch (needs App lock on), **Auto-lock** (value, e.g. "1 minute"; opens screen 32), **Change PIN** (opens screen 39). Auto-lock, Change PIN and the Face ID switch are disabled while App lock is off.
- **Money:** Recurring items, Debts and splits (opens Debts), Budgets, Categories, Accounts.
- **Data and capture:** Backup and export, Allowed senders (opens screen 25 on Android and screen 37 on iPhone; the count shows on Android only, P4).
- **About:** About and help (opens screen 34).
- **Delete all data** (red text, last row; opens screen 35).


**Rules**
- **Currency is locked once any entry exists** (any transaction, debt or split). Until then it can be changed freely. When locked the row is read-only, shows no chevron, and a line under it says "Currency can't be changed once you have entries." The only way to change it is Delete all data and set up again (back up first). Pro (P4) adds accounts in other currencies; the base currency stays as chosen.
- Turning App lock on asks the user to create a PIN; turning it off asks for the PIN.

## 20. Backup and export (`Backup.html`, route `/backup`, P1 backup, P2 export)

**Elements**
- Back, title.
- **Last backup** card: time, size, a "Safe" pill (green) when the last backup is under 14 days old; an amber "Old" pill after that; an amber "No backup yet" pill if never. Time and size come from the last backup made on this phone.
- **Plain-file note** under the restore text (drawn): "**Backups are not encrypted.** Anyone with the file can read your data, so keep it somewhere private."
- **Back up now**: creates one file (all data, no PIN) and opens the share/save sheet.
- **Restore from a file**: file picker, then confirm "This replaces everything on this phone with the backup." Cancel / Replace. After success the app restarts at Home.
- **Weekly backup reminder** switch (day and time, default Sunday 9 pm).
- **Export** section: **Range** (month picker or custom), **Format** (CSV spreadsheet or PDF report), **Export report** (Pro) which opens the share sheet.

**Rules**
- The file is not encrypted, so the card says "Anyone with this file can read your data." (optional password is a possible later release). File name `pocketbook-YYYY-MM-DD`. Backup is a SQLite copy with a `schema_version`; restoring an older file migrates it up; restore refuses files from a newer app version with a clear message.
- Restore never touches the PIN or lock setting. After restore, reminders and notifications are rescheduled and recurring catch-up runs.
- **Export report** carries a small gold "Pro" tag (drawn); for free users tapping it opens Pro. Backup and restore are always free. The tag uses a pale gold fill and dark gold text that are not tokens yet (GAPS).
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

## 22. Create a PIN (`PinCreate.html`, route `/lock/pin/create`, P1)

Opened from Start tracking at the end of setup, and when App lock is switched on in More.

**Elements**
- Back arrow (to Welcome during setup, to More otherwise), lock logo tile, "Create a PIN", "Choose 4 digits. You will use it to open Pocketbook. It is stored only on this phone."
- Four dots (filled as digits are typed), keypad 1 to 9, 0, delete key.
- **Skip for now** link at the bottom. Shown only during setup; from More it is hidden (Back cancels and leaves App lock off).

**Behavior**
- On the 4th digit, go to Confirm PIN. Skip leaves App lock off and goes to Home. A 4-digit PIN such as 0000 or 1234 is allowed; there is no strength rule.
- The Back arrow on Confirm returns here to choose a different PIN.

## 23. Confirm PIN (`PinConfirm.html`, route `/lock/pin/confirm`, P1)

The static screen draws the **error state** (title "Confirm PIN (error state)"); the normal state is the same with accent-colored empty dots and no message.

**Elements**
- Back arrow, lock logo tile, "Confirm your PIN", "Enter the same 4 digits again.", four dots, keypad.
- Error: red dots and "PINs did not match. Try again." (announced as an alert). Dots clear and the user stays here; no limit on attempts.
- Link **Use Face ID or fingerprint too** (with a person icon), shown only when the device has biometrics enrolled.

**Behavior**
- Match: save the salted PIN hash to secure storage, turn App lock on, go to Home (or back to More when opened from there). The PIN link is optional: tapping it first turns on biometric unlock (system prompt) and then finishes. Skipping it leaves the biometric switch off; it can be turned on later in More → Security.
- The Home that follows is the first screen shown; App lock starts applying from the next launch or return.

## 24. Forgot PIN (`PinForgot.html`, route `/lock/forgot`, P1)

Opened from the Lock screen link.

**Elements**
- Back arrow (to Lock), title "Forgot your PIN?", "Pocketbook has no account or server, so your PIN cannot be reset by email. Pick one of these." When Face ID or fingerprint is off the app shows screen 38 instead.
- Option card **Use Face ID or fingerprint**: "Works if you turned it on. You can then set a new PIN in More."
- Option card **Erase and start over** (red): "Deletes everything on this phone. You can restore a backup file afterwards."
- Buttons at the bottom: primary **Use Face ID or fingerprint**, danger-style secondary **Erase and start over**.

**Behavior**
- Face ID or fingerprint: system prompt; success unlocks to Home. If biometrics are off or not enrolled this screen is replaced by Forgot PIN, no biometrics (screen 38).
- After a biometric unlock the user may change the PIN in More without entering the old one: Change PIN (screen 39) offers "Use Face ID or fingerprint instead".
- Erase and start over opens Erase everything (screen 35), which needs ERASE typed. Erasing removes the database, settings, PIN and lock settings and goes to Welcome, whose **Have a backup? Restore it** link restores a file. From here Back and Cancel on screen 35 return to Forgot PIN (the static screen links them to More).

## 25. Allowed senders (`Senders.html`, route `/senders`, P4)

Opened from More → Allowed senders and from "Manage allowed senders" on Review entries. Back goes to More.

**Elements**
- Back arrow, title "Allowed senders", note: "Only messages from these senders are read, and only on this phone. Nothing is saved until you tap Add in Review entries."
- One card per sender: icon tile, name ("Bank alert", "Mobile Wallet"), "Sender name or number · Android", and an on/off switch. The count in More ("2") is the number of senders.
- **How a message is read** card: an example message ("Debit $30 at FUEL STATION on 15-Nov.") and three pills showing what was found (Amount $30, Expense, Fuel station).
- **On iPhone** card: "Set up a Shortcuts automation for each bank message. Steps: 1. Open Shortcuts. 2. New automation, Message. 3. Run "Add to Pocketbook"."
- **Add sender** (primary, opens screen 36) and **Review entries** (secondary, bell icon, opens Review entries).

**Behavior and rules**
- Android: shows the sender list and Add sender. The first time, a permission explainer then the system screen for notification access (SMS only if added later). If access is off the list shows "Notification access is off" with a button to the system setting. Not drawn.
- iPhone (decided): this screen is replaced by Auto-capture on iPhone (screen 37), which has only the Shortcuts guide.
- Add sender form: screen 36. Tapping a card opens the same form in edit mode (Delete inside, not drawn). A switch off keeps the sender but stops reading.
- The "· Android" text is the platform note; it is not shown to iPhone users.

## 26. New account (`AddAccount.html`, route `/accounts/create`, P1)

Opened from Accounts → Add account. Edit mode (not drawn) has the title "Edit account" and the same fields, plus Adjust balance, Archive and Delete.

**Elements**
- Back arrow, title "New account".
- **Type** tiles in one row: Cash, Bank, Card, Wallet (one selected, Bank in the sample). "Wallet" is the tile label for the Mobile wallet type; lists show "Mobile wallet".
- Fields: **Name** (typed), **Currency** (picker, chevron), **Opening** (typed balance).
- Helper: "The opening balance is what the account holds today. Cards may go below zero."
- **Save account** (primary), then toast and back to Accounts.

**Rules**
- Name is required and unique among open accounts (case-insensitive), because quick add matches by name.
- Opening balance may be negative only for the Card type; it must fit the currency's decimals. Default 0.
- Free tier: the currency is the base currency and the row is read-only with no chevron; changing it is Pro (P4).
- Opening balance is not income. Editing it later changes the balance by the difference.

## 27. New category (`AddCategory.html`, route `/categories/create`, P1)

Opened from Categories → New category; tapping a category opens the same form in edit mode (title "Edit category", plus Archive and Delete, not drawn).

**Elements**
- Back arrow, title "New category", segmented **Expense / Income**.
- Fields: **Name**, **Inside** (parent picker; empty means top level; "Bills" in the sample).
- **Icon** row: eight icons in a grid with one selected, and a **See all** link that opens Choose an icon (screen 31) with the full set of 24.
- Helper: "Choose "Inside" to make it a subcategory. Deleting a category that has entries asks you to move them first."
- **Save category** (primary).

**Rules**
- Name required and unique within the same parent.
- Inside lists only top-level categories of the same kind; picking one forces the segmented control to that kind and hides the control. A category that already has subcategories cannot be put inside another.
- A subcategory uses its own icon; the picker defaults to the parent's icon.

## 28. New recurring item (`AddRecurring.html`, route `/recurring/create`, P2)

**Elements**
- Back arrow (to Recurring), title "New recurring item", segmented **Expense / Income**.
- Fields: **Amount** (typed), **Category**, **Account**, **Repeats** (Every day, week, month, year), **On day** (5th), **Remind** (None, on the day, 1 day before, 2 days, 3 days, 1 week), **Note**.
- Helper: "Posted on its day. If the app was closed, it is added next time you open it."
- **Save** (primary).

**Rules**
- On day follows Repeats: hidden for Every day; a weekday for Every week; a date and month for Every year; a day of month (1–31, the last day on short months) for Every month.
- The Note is the item's name in the Recurring list; if empty the category name is used.
- The first posting is the next date on or after today that matches the schedule; today's day is included. There is no start date field.
- The drawn form has no debt installment type; debt installments are in GAPS.
- Edit mode (not drawn): same form, a Delete button, and the pause switch stays on the list. Changes affect future postings only.

## 29. New budget (`AddBudget.html`, route `/budgets/create`, P2)

**Elements**
- Back arrow (to Budgets), title "New budget", segmented **Whole month / One category** (One category selected in the sample).
- Fields: **Category** (picker, hidden for Whole month), **Monthly limit** (typed).
- Switch card **Warn me at 80% and 100%** ("A notification on this phone"), on by default.
- Helper: "Budgets restart on your month start day. Spending from last month does not carry over."
- **Save budget** (primary).

**Rules**
- Category picker lists top-level expense categories that do not yet have a budget; Whole month is available only once (if it exists, the form opens it for editing).
- Limit must be greater than 0. Store the switch as `alerts_on` on the budget; alerts are once per threshold per month.
- Edit mode (not drawn): title "Edit budget", same fields, and a Delete button.

## 30. Add debt (`AddDebt.html`, route `/debts/create`, P3)

Opened from Debts → Add debt.

**Elements**
- Back arrow (to Debts), title "Add debt", segmented **I lent / I borrowed**.
- Fields: **Person** (picker with Add person), **Amount**, **Due date** (optional), **From account** (reads **Into account** for I borrowed), **Note**.
- **Date** row (drawn): when the money moved; default today, past dates allowed, future dates not. Debt detail's first history line uses it. **Installment** row (drawn): only for I borrowed with an account chosen. Shows the amount and day ("$50 · 1st of month"); empty means none. Shown on Debts as "Installment $50 monthly".
- Helper, I lent: "Money you lend leaves the account but is not an expense. Choose "No account" if the money did not pass through one." Helper, I borrowed (drawn): "Installment is optional and only for money you borrow. It is paid from the account on that day, with a reminder the day before. Money you borrow is not income." The static screen draws I borrowed; I lent shows From account and no Installment row.
- **Save debt** (primary).

**Rules**
- Amount greater than 0. Person required. The account picker has a "No account" option.
- With an account, a `debt_out` (lent) or `debt_in` (borrowed) entry is created on the chosen Date. With no account, only the debt exists and no balance changes.
- Free tier: the 4th open loan opens Pro instead.
- A due date adds a due pill on Debts and a reminder on the due date; it cannot be before the Date.
- **Installment rules:** saving creates a recurring item of type debt payment linked to the debt (`recurring.debt_id`), paid from the chosen account, reminder 1 day before. Each due date posts a `debt_out` payment for the installment amount, or the remaining balance if smaller; when the debt reaches zero the item stops. Editing or deleting the debt updates or removes it. Installments are only for money the user owes; for I lent there is no row, so money is never auto-recorded as received.

## 31. Choose an icon (`IconPicker.html`, route `/categories/icon`, P1)

Opened from New category → See all.

**Elements**
- Back arrow (to New category), title "Choose an icon", note: "One icon per category. Icons are the same neutral color in the app; only the shape differs."
- A 4-column grid of 24 icon tiles (62px high); one is selected (accent border, `accent-tint` fill). The set is listed in `docs/GAPS.md` under "Category icon set".
- **Use this icon** (primary): saves the choice and returns to New category with the icon selected.

## 32. Auto-lock (`AutoLock.html`, route `/settings/auto-lock`, P1)

**Elements**
- Back arrow (to More), title "Auto-lock", note: "How long Pocketbook can be in the background before it asks for your PIN again."
- Four radio rows, one selected: Immediately ("Every time you leave the app"), After 1 minute ("Recommended", default), After 5 minutes, After 15 minutes.
- **Done** (primary): saves and returns to More; the More row shows the choice.

**Rules**
- Only available while App lock is on. The time counts from when the app goes to the background; a cold start always asks for the PIN.

## 33. Reminders (`Reminders.html`, route `/settings/reminders`, P2)

**Elements**
- Back arrow, title "Reminders", note: "Reminders are notifications made on this phone. No account or internet is needed."
- Switch cards: **Log your day** ("A nudge to add today's spending"), **Weekly backup** ("Sundays at 9 pm"), **Budget warnings** ("At 80% and 100% of a budget"), **Recurring and debt due dates** ("The day before").
- **Time** row (9:00 pm, opens a time picker) belongs to Log your day.
- Note: "If notifications are off in your phone settings, reminders cannot appear. Turn them on there."
- **Done** (primary).

**Rules**
- Weekly backup is the same setting as the switch on Backup and export. Budget warnings is a master switch over each budget's own "Warn me" switch (both must be on). The due-date switch covers recurring items with a reminder and debts with a due date.
- Turning a switch on asks for notification permission the first time (with a one-line explanation). If denied the switch stays on and the note shows.
- Default time for Log your day is 9:00 pm. Tapping the reminder opens Add.
- Phases: the screen is built in P1 with only Weekly backup; Log your day and Budget warnings arrive with P2 and the due-date switch with P2/P3. Hide a row until its feature exists.

## 34. About and help (`About.html`, route `/about`, P1)

**Elements**
- Back arrow, title "About and help", logo tile, "Pocketbook", "Version 1.0.0".
- Card rows: **Privacy** "Stays on this phone", **Accounts** "None needed", **Internet** "Not used".
- Note: "Pocketbook has no server. Your money data is stored only on this phone and in backup files you make yourself."
- Link rows: **Help and tips**, **Contact support**, **Rate Pocketbook**.

**Rules**
- The version is read from the app. Help and tips opens a plain help page (not drawn; GAPS: write the content). Contact support opens the mail app with a prefilled subject, never with user data. Rate Pocketbook opens the store page; these two leave the app and are the only external hand-offs.

## 35. Erase everything (`EraseConfirm.html`, route `/settings/erase`, P1)

Opened from More → Delete all data and from Forgot PIN → Erase and start over.

**Elements**
- Back arrow, title "Erase everything?", red trash tile, "This deletes every account, entry, budget, debt and setting on this phone, and removes your PIN. It cannot be undone."
- **Last backup** card with a pill (Safe, Old or No backup yet).
- Note: "Pocketbook keeps nothing online. After erasing, you can set up again or restore a backup file."
- **Type ERASE** field.
- **Erase everything** (red, with trash icon) and **Cancel**.

**Rules**
- Erase everything stays disabled until the field reads ERASE exactly (case-insensitive). Then it deletes the database, settings, PIN and lock settings, cancels all scheduled notifications, and opens Welcome. Pro stays with the store account, so Restore purchase brings it back.
- Back and Cancel return to whichever screen opened it.

## 36. Add sender (`AddSender.html`, route `/senders/create`, P4)

**Elements**
- Back arrow (to Allowed senders), title "Add sender".
- Fields: **Name** (a label such as "Bank alert") and **Sender** (the name or number the messages come from, e.g. HBLBANK).
- Note: "Enter the name or number the messages arrive from, exactly as your phone shows it. Pocketbook only reads messages from senders you add."
- **Try it with a message**: a text box for pasting a sample message; below it pills show what was found (Amount $30, Expense, Fuel station). "Nothing is saved from a test."
- **Save sender** (primary).

**Rules**
- Name and Sender are required; Sender is matched exactly and case-insensitively. A duplicate Sender is refused.
- The test is parsed on the phone with the same rules as capture and never creates an entry.
- Not drawn: an Account row and an optional default category row (GAPS). Edit mode has the title "Edit sender" and a Delete button.

## 37. Auto-capture on iPhone (`SendersIphone.html`, route `/senders`, P4)

Replaces Allowed senders on iPhone.

**Elements**
- Back arrow (to More), title "Auto-capture on iPhone", heading "Auto-capture", "On iPhone, apps cannot read your messages. Instead, a Shortcut sends each bank message to Pocketbook, and you review it before it is saved."
- Three numbered steps: 1 **Open Shortcuts** ("Go to the Automation tab and tap New Automation."), 2 **Choose Message** ("Enter your bank's sender name and select Run Immediately."), 3 **Add "Add to Pocketbook"** ("Pass the message text. It appears in Review entries.").
- **Open Shortcuts** (primary) and **Review entries** (secondary).

**Rules**
- No sender list and no Add sender on iPhone. The More row shows no count.
- Each message the Shortcut passes becomes one card in Review entries; the sender name comes from the Shortcut. The user still taps Add to save.

## 38. Forgot PIN, no biometrics (`PinForgotNoBio.html`, route `/lock/forgot`, P1)

Shown instead of screen 24 when Face ID or fingerprint is off or not set up.

**Elements**
- Back arrow (to Lock), title "Forgot your PIN?", "Face ID and fingerprint are off, and Pocketbook has no server to reset your PIN. The only way in is to erase this phone's data and start again."
- Card **Erase and start over**: "Deletes everything on this phone. Restore a backup file afterwards to get your data back."
- **Erase and start over** (red, opens screen 35) and **Back** (to Lock).

## 39. Change PIN (`PinCurrent.html`, route `/settings/change-pin`, P1)

Opened from More → Change PIN.

**Elements**
- Back arrow (to More), title "Change PIN", "Enter your current PIN", "Then you will choose a new one.", four dots and keypad.
- Link **Use Face ID or fingerprint instead**, shown only when biometrics are on.

**Rules**
- A correct PIN, or a biometric success, opens Create a PIN (screen 22) and then Confirm PIN (screen 23); Create's Skip is hidden. Success returns to More with the toast "PIN changed".
- A wrong PIN uses the same attempt count and lockout waits as the Lock screen.


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
- Backup: the gold Pro tag uses colors that are not tokens (fill `#F6EBCF`, text `#7A5B12`).
- Add debt: the static screen draws I borrowed (a $200 personal loan); the I lent version is only described.
- Welcome's restore link opens the Backup screen; build a direct file picker.
- Erase everything links Back and Cancel to More; from Forgot PIN they return to Forgot PIN.
- Pin Confirm is drawn only in its error state.
- The canvas board for Confirm PIN is titled "Confirm PIN (error state)"; its page title is "Confirm PIN".
