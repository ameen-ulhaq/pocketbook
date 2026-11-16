# Pocketbook: what the design does not show yet

The 46 designed screens cover the main path, setup, PIN, capture, add/edit forms, settings pages, detail screens, menus and states. Everything below is needed for a complete app but is not drawn. For each: what it must do, and a recommended default so it can be built. Build these with the design system tokens and the existing patterns (Field rows, Chips, Buttons, Cards). Mark a design request in your report so the owner can have it designed.

Status key: **Decide** = needs the owner's call (a default is given so work is not blocked); **Default** = safe to build as described. **Design** = a drawn screen or change is needed before the owner would call it finished; the default is to build it from existing components in the meantime. Phase in brackets.

## Drawn since the first audit (no longer gaps)
Restore link on Welcome; Create PIN, Confirm PIN (error state), Forgot PIN; Forgot PIN? link on Lock; Allowed senders (Android list, iPhone steps); New account, New category, New budget, New recurring item, Add debt; Split a bill button and the Overdue pill on Debts; "Debts and splits" row in More; Allowed senders opens its screen. They are written up as SCREENS 22–30.

Drawn in the fourth update: Account detail, Edit account, Debt menu, Split menu, Time picker, Allowed senders with access off, Help and tips; Account and Category rows on Add sender; Accounts cards open Account detail; "⋯" menus on Debt detail and Split detail; Time row opens the picker; Help link on About. Owner rule: edit forms for category, budget, recurring item, debt and sender reuse the New forms, filled in, with a red Delete.

Drawn in the third update: Choose an icon (24 icons), Auto-lock, Reminders, About and help, Erase everything (type ERASE), Add sender with a message test, Auto-capture on iPhone, Forgot PIN with no biometrics, Change PIN; More now has Reminders, Auto-lock, Change PIN, About and help and Delete all data; New category links to the icon picker; the Pro tag colors are now the tokens `gold-tint` and `gold-ink`.

Drawn in the second update: Tags row on Add; Duplicate on Detail; Date and Category rows on Split a bill; period arrows on Reports; the not-encrypted note and Pro tag on Backup and export; Theme row removed; Review entries button on Allowed senders; Date and Installment rows on Add debt; Pro copy "notifications and Shortcuts".

## Setup and security
| Item | Needed behavior | Status |
| --- | --- | --- |
| Normal Confirm PIN state [P1] | The static screen shows only the error state; build the normal state with accent dots and no message | Default |
| Biometric-off state on Forgot PIN [P1] | Drawn (screen 38) | Done |
| Erase confirm [P1] | Drawn (screen 35, type ERASE) | Done |
| Change PIN [P1] | Drawn (screen 39) | Done |
| Auto-lock time [P1] | Drawn (screen 32): Immediately, 1, 5 or 15 minutes; default 1 | Done |
| Lockout schedule [P1] | 30 s after 5 wrong tries, 1 min after 10, 5 min after 15, then 15 min per 5; kept across app restarts | Default |
| App-switcher privacy cover [P1] | Hide balances in the app switcher; Android secure window | Default |
| Currency lock [P1] | Currency is read-only once any entry exists (see Resolved decisions) | Default |
| Language and currency pickers [P1] | Searchable lists | Default |
| Numerals [P1] | Western digits (0–9) in every language for v1 (decided) | Default |

## Pickers and sheets
Category picker (with subcategories, search, New category); Account picker (with balances and a "No account" option on Add debt); Date picker; Month picker; Custom date-range picker; Person picker (with Add person); Day-of-month picker (1–28 for month start; 1–31 for recurring); Repeats, On day and Remind pickers on New recurring item; Inside (parent) picker on New category; Filter sheet in History (account, category, amount range; lives in the month chip's sheet); Share sheet (system); Discard-changes confirm; Upcoming (future-dated) marker. All Default; none drawn.

## Forms and states of drawn forms not drawn
- Edit modes: owner rule, no separate designs. Category, budget, recurring item, debt and sender reuse the New forms filled in with a red Delete; accounts use Edit account. Done
- Add person and the Person picker. Default
- Free-tier Currency row on New account (read-only, no chevron). Default
- Validation and error states of every form (empty name, duplicate name, zero amount, split not adding up). Default

## Design still needed (short list)
- **Debt-settled state:** how Debt detail looks after "Mark as settled" (a "Settled without payment" history line). **Design**
- **Delete account** button for an account with no entries (same place as Archive). Default
- **Time picker minutes:** the wheel steps by 10; confirm this is intended or use 5. **Decide**
- **Back and Cancel on Erase everything** return to the caller (More or Forgot PIN); the static links all go to More. Default
- The Debt, Split and time sheets are drawn over blank cards; build them over the real screen. Default

## Detail screens not drawn
Category detail (month total, entries); CSV statement import (choose file, map columns once, preview, skip duplicates); Share-to-add (receive text, show parsed Add screen) [P4]; Recurring item detail; Person detail (all debts and splits with that person). Default.

## Elements the design does not show that the rules need
- Pills on Detail and History for debt movements, adjustments and upcoming entries. Default
- Delete debt/split and Edit in a menu; Remind setting. Default

## Rules that were missing or contradictory (now written)
- Month naming when the start day is not 1 (named by the month it starts in). Default
- Future-dated entries count from their date and show as Upcoming. Default
- Free limits count open loans and open splits only, not split shares, on-behalf or held debts. Resolved.
- Split paid by someone else: the user's share is an expense with no account plus a debt to the payer. Default
- A capture with no category opens Add instead of saving as Uncategorized. Default
- Recurring transfers are out of v1. Resolved.
- Budget percent shows 99% until the limit is reached; legend percentages add to 100. Default
- Adjust balance creates an `adjustment` entry that is neither income nor spending. Default
- Skip for now on Create PIN exists only during setup. Default
- Change PIN accepts biometric instead of the current PIN, so a user who forgot the PIN can still set a new one. Default

## States
- **Empty states** for every list (History, Accounts beyond Cash, Budgets, Recurring, Debts, Review entries, Reports with no data, Splits).
- **Loading:** local data is fast; no spinners except export and restore.
- **Errors:** clear one-line messages with a retry for backup, restore (bad file, newer version), export, purchase, and file access.
- **Success states** for Pro unlock, restore, backup, export.
- **Free limit reached:** when adding the 4th loan or split, show Pro with the reason ("You have 3 open splits. Settle one or unlock Pro.").
- **Notification permission** request (budgets, recurring reminders, backup reminder, due dates), with a gentle pre-explanation and a "Notifications are off" row if denied.
- **Large numbers:** amounts that do not fit shrink one step, then scroll; never truncate digits.
- **Date change at midnight:** "Today" labels update when the app returns to the foreground.

## Settings additions
Dark theme (later release: a `dark` theme in `tokens.json`, a contrast check, and the Theme row; not in v1). Help and tips content and the Help page. The settings pages are drawn (SCREENS 32 to 35, 39).

## Pro and advanced [P4]
Pro success state; Restore result messages; Multi-currency flow (account currency, manual exchange rates, base currency for totals, rates screen, transfers between currencies, per-currency debts); Home-screen widget (Add expense, balance); iPhone Shortcuts setup guide (in-app steps with a ready template); Android notification-listener permission explainer; CSV statement import (listed in SPEC but in no phase: it is now P2, after export).

## Store and release items [P4 end]
App name and icon, privacy policy page and store privacy answers ("no data collected"), store screenshots, age rating, product IDs and price, a support contact. All Decide/owner.

## Technical feasibility to confirm in Task 1 (before building the phase that needs them)
These depend on what NativePHP Mobile supports today; check the docs and licence first and tell the owner if any is not possible.
- Secure storage for the PIN hash, biometrics, local notifications and scheduled reminders, background task for recurring, file save/share/pick, in-app purchases (both stores), share-to-app (Android share intent, iOS share extension), home-screen widget (iOS WidgetKit is native Swift), Shortcuts action (iOS App Intents), Android SMS/notification listener.
- Google Play restricts the SMS permission; an expense app usually needs a policy declaration and may be refused. Resolved: notifications first; SMS is optional and added only if approved.
- iOS cannot read SMS or other apps' notifications, so iPhone auto-capture stays Shortcuts-only.
- Local notifications are limited on iOS (64 pending); schedule the next ones and reschedule on app open.

## Resolved decisions (owner, 2026-11-16)
1. Free limits count open (not settled) loans and splits only.
2. Recurring transfers are not allowed in v1.
3. Backup is an unencrypted file in v1, with a note on the Backup screen (**Design**: the note is not drawn).
4. Android capture reads notifications first; SMS is optional and later, only if Google Play allows it. The Pro screen copy "SMS and Shortcuts" should read "notifications and Shortcuts" (**Design**).
5. v1 is light-only. The Theme row is removed from the Settings spec (the static screen still draws it, omit it); dark tokens wait for a later release.
6. Currency cannot be changed once any entry exists; it is set at setup. Changing it means Delete all data and starting again.
7. CSV statement import is Phase 2 (also written in `CLAUDE.md`).
8. Debt installments are kept: an optional Installment row on Add debt, for money the user owes only.
9. Forgot PIN is designed as two options: biometric, or erase and start over (then restore from a backup). Erase asks for confirmation.

9. Backup status: "Safe" for 14 days, then an amber "Old" pill.
10. Digits: Western digits (0–9) in every language for v1, on screens, keypads and backups.
11. Languages: ship English first, then Urdu and Arabic. All text goes through language files from day one so right-to-left can be tested early; Spanish and French from the setup list wait until after those.
12. App name: Pocketbook for now. The name lives in one config value so it can change.
13. Pro price: decided at the start of Phase 4; until then the Pro screen keeps the placeholder.
14. Category icons: 24, picked from the existing icon set and now drawn on Choose an icon. The list and the default mapping are under "Category icon set".
15. Allowed senders on iPhone shows only the Shortcuts guide: no sender list and no Add sender button (the rules in SCREENS 25 already say this).

## Category icon set (24, drawn on "Choose an icon")
Stored by name in `categories.icon`. Row by row, as drawn: 1 fork-knife · 2 car · 3 bag · 4 bolt · 5 heart · 6 book · 7 ticket · 8 gift · 9 home · 10 phone · 11 card · 12 banknote · 13 bank · 14 wallet · 15 target (ring) · 16 repeat · 17 people · 18 person · 19 globe · 20 tag · 21 calendar · 22 clock · 23 chat · 24 star.
Default categories use these: Rent and home → home, Food → fork-knife, Groceries → bag, Dining → fork-knife, Tea → fork-knife, Bills → bolt, Electricity → bolt, Internet → globe, Mobile → phone, Transport → car, Fuel → car, Rides → car, Shopping → bag, Clothes → tag, Household → home, Health → heart, Doctor → heart, Medicine → heart, Education → book, Fees → card, Books → book, Fun → ticket, Outings → ticket, Subscriptions → repeat, Salary → banknote, Freelance → wallet, Gifts → gift, Other → star. A custom category defaults to the star.

## Open decisions for the owner
None. The next decisions come with the work: the Pro price at the start of Phase 4, and a later check of Urdu and Arabic layouts when those languages are added.
