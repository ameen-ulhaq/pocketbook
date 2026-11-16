# Pocketbook: screen-by-screen spec

Source of truth for what each of the 21 designed screens contains and does. Each entry lists every element, every option, every action, the states, and the rules behind it.

- **Design file:** the matching static page in `docs/design/screens/<Name>.html` (open `docs/design/screens/index.html`). Sample amounts there are examples only (US dollars); real values come from the user's data and currency.
- **Online (clickable):** https://claude.ai/artifact/McKtWDuTK1kgQKSnffzJrs
- **Design system:** https://claude.ai/artifact/JkVyy5F7v7a6TFdGPV2Tnc (tokens in `docs/design/tokens.json`, components in `docs/design/components/`)
- Things the design does not draw but the app needs are in `docs/GAPS.md`. Step-by-step journeys are in `docs/FLOWS.md`.

Phase column: P1 = Core, P2 = Insights, P3 = People and money, P4 = Advanced (see `CLAUDE.md`).

## Global rules (apply to every screen)

- **Layout:** 390 wide design, 20px side padding on both sides, 52px full-width buttons, 44px minimum tap target. Screens with the bottom bar leave 78px at the bottom.
- **Bottom bar** (Home, History, Accounts, Categories, Budgets, Reports x2, Debts, More): Home, History, raised Add button, Reports, More. Active tab is green with a soft pill behind the icon. Sub-screens (Add, Detail, Debt detail, Splits, Backup, Recurring, Review entries, Pro, On behalf) have a Back arrow and no bar.
- **Money:** integer minor units, always with sign on transactions (−$4.50, +$1,400), thousands separators and decimals by locale and currency (0 decimals for JPY, 3 for KWD).
- **Colors:** green only for money in or owed to the user; red for money the user owes, overspending, delete; amber for warnings; icons are neutral tiles.
- **Dates:** relative for today and yesterday ("Today", "Yesterday"), otherwise "13 Nov"; with year when not the current year.
- **Destructive actions** always ask for confirmation with Cancel and a red action.
- **Success feedback:** a short toast ("Saved", "Deleted", "Backed up") and return to the previous screen.
- **Empty states:** one line that says what goes here and a primary button to add the first item.
- **Text size and RTL:** layouts must survive large system text and right-to-left languages.

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

**On Start tracking:** save settings, create the Cash account with the opening balance, seed default categories, then offer "Create a PIN" (see GAPS), then go to Home.
**Validation:** opening balance must be zero or positive and fit the currency's decimals.
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
- Wrong: dots clear with a shake and "Wrong PIN. Try again." After 5 wrong tries wait 30 seconds, then longer each round.
- Biometric success unlocks; biometric failure falls back to the keypad.
- The PIN is stored only as a salted hash in the device keystore; never in the database or backup.

## 3. Home (`Home.html`, route `/`, P1)

**Elements**
- Greeting by time of day (Good morning, afternoon, evening) and the current month and year.
- **Bell button** with a red count badge = number of entries waiting in Review entries; hidden when 0. Tap goes to Review entries.
- **Quick add** line: type something like "lunch 4.50 cash". It finds the amount, the account by name and the category by remembered note, then opens Add prefilled for one-tap confirmation. If it cannot read an amount it says so and opens Add empty. (Detailed parsing in FLOWS.)
- **Balance card:** Total balance (sum of all non-archived accounts, in the base currency), and two boxes: Income and Spent for the current month (month respects the month start day). Not tappable.
- **Debt boxes:** "Owed to you" (green) and "You owe" (red) totals, both tap to Debts.
- **Recent:** the latest 3 transactions with "See all" to History. Each row opens Detail.
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
- **Account** field: picker of accounts with balances. For Transfer this becomes **From** and a second **To** field appears; they must differ.
- **Date** field: date picker, default today; future dates allowed (they count toward the month they fall in).
- **Note** field: free text, one line.
- Save button text follows the type: "Save expense", "Save income", "Save transfer".

**Rules**
- Amount must be greater than 0; category is required for expense and income; accounts must differ for transfer.
- Expense lowers the account balance, income raises it, transfer moves it between accounts and is never counted as spending or income.
- After save: toast, back to the previous screen, balances update.
- Remembers the last category used for each note text to prefill next time.

## 5. Transactions / History (`History.html`, route `/transactions`, P1)

**Elements**
- Title "Transactions".
- **Search** field: matches notes, category names, people names and amounts.
- **Filter chips:** All, Expense, Income, Transfer, and a month chip ("Nov 2026") that opens a month picker. The chip row scrolls sideways and keeps 20px at the end. More filters (account, category) are in the month chip's sheet (see GAPS).
- Grouped list by day. Each day header shows the date label and that day's net total (e.g. "−$34.50").
- Rows (icon tile, name, "Category · Account", signed amount) tap to Detail.

**States**
- No results: "No transactions match." with a Clear filters button.
- Empty app: same as Home empty state.
- Lists load in pages of 50 as the user scrolls.

## 6. Transaction detail (`Detail.html`, route `/transactions/{id}`, P1)

**Elements**
- Back, title "Details".
- Hero: icon tile, "Category › Subcategory", signed amount, type pill (Expense / Income / Transfer).
- Detail card rows: **Account** (for a transfer: "From → To"), **Date**, **Note**, **Tags**, **Added** (how it got here: Typed, Quick add, Shared in, Recurring, Auto-capture, Split, Debt payment).
- **Edit** (opens Add prefilled) and **Delete** (confirm: "Delete this entry? Balances will update." Cancel / Delete).

**Rules**
- Entries created by a split or a debt payment open their parent (split or debt) for editing; Delete is offered there so the ledger stays consistent.
- Recurring entries show an extra row "Repeats monthly" linking to the recurring item.

## 7. Accounts (`Accounts.html`, route `/accounts`, P1)

**Elements**
- Title "Accounts" with a transfer icon button top right (opens Add in Transfer mode).
- "Total across accounts" with the big total.
- One card per account: type icon, name, "type · currency", current balance.
- **Add account** button.
- Helper text about transfers.

**Actions**
- Tap an account → Account detail (see GAPS): its transactions, Edit (name, type, opening balance), Adjust balance (creates an "Adjustment" entry), Archive (hides it but keeps history).
- Add account form: name, type (Cash, Bank, Card, Mobile wallet), currency, opening balance. Credit cards may go negative.
- Free tier: single currency. Multi-currency accounts are Pro (P4).

## 8. Categories (`Categories.html`, route `/categories`, P1)

**Elements**
- Title, segmented **Expense / Income**.
- Rows: icon tile, name, subcategories line ("Groceries, Dining, Tea") or "N subcategories", this month's total, chevron.
- **New category** button.

**Defaults seeded at setup**
- Expense: Rent and home, Food (Groceries, Dining, Tea), Bills (Electricity, Internet, Mobile), Transport (Fuel, Rides), Shopping (Clothes, Household), Health (Doctor, Medicine), Education (Fees, Books), Fun (Outings, Subscriptions), Other.
- Income: Salary, Freelance, Gifts, Other.

**Actions**
- Tap → Category edit (see GAPS): name, icon (from a fixed set), parent, archive. Deleting a category in use asks to move its entries to another category first.

## 9. Budgets (`Budgets.html`, route `/budgets`, P2)

**Elements**
- Title "Budgets".
- **Monthly budget card:** spent of limit ("$963.50 of $1,100"), "15 days left", progress bar, and a sentence: "88% used. $136.50 left, about $9.10 a day."
- **Category budget cards:** icon, name, "$243 of $250", a pill (percent used, or "Over by $12"), progress bar.
- **New budget** button.

**Rules**
- Bar color: accent under 80%, amber 80–100%, red over 100% (bar capped at full width); status is always also in words.
- A budget is monthly and follows the month start day. Overall budget is separate from category budgets.
- Alerts: local notification at 80% and 100% (once each per month).
- New budget form: Overall or a category, monthly limit. Edit and delete by tapping a card.

## 10. Recurring (`Recurring.html`, route `/recurring`, P2)

**Elements**
- Back, title "Recurring", explanation: "Posted automatically when due. If the app was closed, missed ones are added next time you open it."
- Cards: icon, name, schedule ("Monthly on the 5th · reminder 1 day before"), amount (+ green for income), and an on/off switch (off shows "paused").
- **New recurring item** button.

**Rules**
- Frequencies: daily, weekly, monthly, yearly; day of month or weekday; optional reminder N days before.
- On app open (and by background task where available) every due occurrence is posted, oldest first, with date = due date.
- Switch off pauses without deleting. Tap a card to edit; delete asks for confirmation and keeps already-posted entries.

## 11. Review entries (`Pending.html`, route `/captures`, P4, Pro)

**Elements**
- Back, title "Review entries", note: "Nothing is saved until you tap Add. Only messages from senders you allow are read, on this phone."
- Cards: sender with icon, received time, the raw message in a shaded box, parsed amount, "merchant · category", a "Found by sender rule" pill, and buttons **Add**, **Edit**, **Ignore**.
- **Manage allowed senders** button.

**Actions**
- Add saves exactly as parsed. Edit opens Add prefilled. Ignore removes it from the list.
- Android reads SMS and notifications from allowed senders; iPhone gets entries from a Shortcuts automation. See `docs/SPEC.md`.
- Empty state: "No messages waiting."

## 12. Reports: where it went (`ReportsOverview.html`, route `/reports`, P2)

**Elements**
- Title "Reports" with an export icon (opens Backup and export; Pro).
- Tabs: **Where it went** | **Trends**.
- Period chips: Week, Month, Year, Custom (Custom opens a date-range picker).
- Donut chart with the total spent in the centre, one segment per category.
- Legend list: color swatch, category, percent, amount. Top 5 categories plus "Other".

**Rules**
- Counts expenses only (own share for splits). Transfers, repayments and held funds are excluded.
- Tapping a legend row opens History filtered to that category and period.
- Chart has a text alternative (the legend). Empty period: "No spending in this period."

## 13. Reports: trends (`ReportsTrends.html`, route `/reports/trends`, P2)

**Elements**
- Same header, tabs and period chips.
- Two stat boxes: **Average per day** and **Saved this month** (percent of income).
- **Income vs spending** grouped bars for the last 6 months with a legend.
- **Daily spending** columns for the selected month.

**Rules**
- Saved percent = (income − spending) ÷ income; shown as 0% if income is 0.
- Period chips drive the stats and the daily chart; the 6-month chart always ends at the current month.

## 14. Debts (`Debts.html`, route `/debts`, P3)

**Elements**
- Title "Debts"; two summary boxes **Owed to you** (green) and **You owe** (red).
- Section **Owed to you**: avatar, name, detail ("Loan · paid back $30 of $80", "Paid on his behalf · phone repair", "Dinner split · 12 Nov"), remaining amount, optional due pill ("Due 20 Nov", amber).
- Section **You owe**: same pattern (personal loan with installment, borrowed from a person).
- **Add debt** button.

**Actions**
- Tap a loan row → Debt detail. Tap a split row → Split detail.
- Add debt form: direction (I lent / I borrowed), person or lender, amount, optional due date, note, and "Did the money move through an account?" with an account picker (if yes it creates a transfer-like entry that is not an expense or income).
- Free tier: up to 3 active debts and 3 splits; the 4th shows the Pro upgrade.

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
- Edit due date, note or delete the debt from a menu (see GAPS); deleting asks for confirmation.

## 16. Paid on behalf (`OnBehalf.html`, route `/behalf`, P3)

**Elements**
- Back, title, segmented: **I paid for someone** | **Holding money for them**.
- Fields: **Person** (pick or add), **Amount**, **Paid from** (Mode 2: **Received into**), **Note**.
- Call-out: "Not your expense. $49 leaves Cash and shows as owed to you by Bilal. When he pays you back, record it as a repayment into any account."
- Second note explains the other mode: money sent to you to pay out is not income and clears when you pay it out.
- **Save**.

**Rules**
- Mode 1: account goes down, a debt "owed to you" of kind on_behalf is created, no expense.
- Mode 2: account goes up, a debt "you owe" of kind held is created, no income; paying it out is a payment from an account against that debt.

## 17. Split a bill (`SplitNew.html`, route `/splits/create`, P3)

**Elements**
- Back, "Bill total" large.
- Fields: **What for**, **Paid by** ("You · Cash" or another person; if another person, the user's share becomes a debt "you owe").
- Method segmented: **Equal, Exact, Percent, Shares**.
- People checklist (the user plus others) with each person's amount; a check box per person; **Add person**.
- Summary: "Your share is $15. The other three owe you $45, tracked until they pay."
- **Save split**.

**Rules**
- Equal: total ÷ people, remainder cents go to the first people. Exact: amounts must add up to the total. Percent: must add up to 100. Shares: weights.
- Save creates one expense for the user's own share only (category chosen on save or default) plus a debt per other person.
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
- **General:** Language, Currency, Month starts on (day 1–28), Theme (System, Light, Dark).
- **Security:** App lock switch, Face ID or fingerprint switch.
- **Money:** Recurring items, Budgets, Categories, Accounts.
- **Data and capture:** Backup and export, Allowed senders (count).

**Rules**
- Changing currency after data exists only changes how amounts display and new entries; it does not convert history (warn clearly). Real multi-currency is Pro (P4).
- Turning App lock on asks the user to create a PIN; turning it off asks for the PIN.
- Theme Dark requires a dark token set (see GAPS).
- Missing items to add: Debts and Splits shortcuts, Reminders, About and help (GAPS).

## 20. Backup and export (`Backup.html`, route `/backup`, P1 backup, P2 export)

**Elements**
- Back, title.
- **Last backup** card: time, size, a "Safe" pill (or "No backup yet" with an amber pill).
- **Back up now**: creates one file (all data, no PIN) and opens the share/save sheet.
- **Restore from a file**: file picker, then confirm "This replaces everything on this phone with the backup." Cancel / Replace. After success the app restarts at Home.
- **Weekly backup reminder** switch (day and time, default Sunday 9 pm).
- **Export** section: **Range** (month picker or custom), **Format** (CSV spreadsheet or PDF report), **Export report** (Pro) which opens the share sheet.

**Rules**
- Backup file is versioned JSON or SQLite copy; restore refuses files from a newer app version with a clear message.
- Restore never touches the PIN or lock setting.

## 21. Pocketbook Pro (`Pro.html`, route `/pro`, P4)

**Elements**
- Back, logo, "Pocketbook Pro", "Pay once. Keep it forever. No subscription."
- Feature list with checks: Unlimited splits and debts (free includes 3 of each); Auto-capture from SMS and Shortcuts; Multiple currencies; CSV and PDF export; Home-screen widget.
- Note: "Free keeps everything else: tracking, budgets, reports, backup."
- **Unlock Pro · [ONE-TIME PRICE]** and **Restore purchase**.

**Rules**
- One non-consumable purchase through the App Store and Google Play. Price comes from the store, shown in local currency.
- Restore purchase re-checks the store entitlement. After purchase show a success state and unlock features immediately (see GAPS).
- Pro is checked locally from the store receipt; the app never needs a server.
