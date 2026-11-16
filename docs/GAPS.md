# Pocketbook: what the design does not show yet

The 21 designed screens cover the main path. Everything below is needed for a complete app but is not drawn. For each: what it must do, and a recommended default so it can be built. Build these with the design system tokens and the existing patterns (Field rows, Chips, Buttons, Cards). Mark a design request in your report so the owner can have it designed.

Status key: **Decide** = needs the owner's call (a default is given so work is not blocked); **Default** = safe to build as described. **Design** = a drawn screen or change is needed before the owner would call it finished; the default is to build it from existing components in the meantime. Phase in brackets.

## Setup and security
| Item | Needed behavior | Status |
| --- | --- | --- |
| Restore from Welcome | "Have a backup? Restore it" link under Start tracking; file picker, confirm, then Home. Needed because a new phone has no other way in before setup | Default · Design |
| Create PIN [P1] | After setup (optional) or when turning on App lock: enter 4 digits, confirm, then "Use Face ID or fingerprint?" | Default · Design |
| Change PIN [P1] | More → Security; ask current PIN, then new twice | Default |
| Forgot PIN [P1] | No server, so no recovery. Offer biometric if on; otherwise "Reset app" (type RESET) which erases data and returns to Welcome where a backup can be restored | Decide |
| Auto-lock time [P1] | Immediately, 1 minute, 5 minutes (default 1 minute); a row in Security | Default · Design |
| Lockout schedule [P1] | 30 s after 5 wrong tries, 1 min after 10, 5 min after 15, then 15 min per 5; kept across app restarts | Default |
| App-switcher privacy cover [P1] | Hide balances in the app switcher; Android secure window | Default |
| Language and currency pickers [P1] | Searchable lists | Default |
| Numerals for Arabic and Urdu [P1] | Show Western digits (0–9) or locale digits? Recommended: Western digits in v1 for input reliability | Decide |

## Pickers and sheets
Category picker (with subcategories, search, New category); Account picker (with balances); Date picker; Month picker; Custom date-range picker; Person picker (with Add person); Day-of-month picker (1–28 for month start; 1–31 for recurring); Filter sheet in History (account, category, amount range; lives in the month chip's sheet); Share sheet (system); Discard-changes confirm; Upcoming (future-dated) marker. All Default; none drawn.

## Forms not drawn
Add or edit account; Add or edit category (name, icon from fixed set, parent, archive); Add or edit budget; Add or edit recurring item; Add debt (including optional installment); Edit debt (due date, note, installment); Edit or delete split; Add person; Tags on Add (free-text chips); Adjust account balance; Date and Category rows on Split a bill (the design has neither, the data needs both); Split-a-bill entry button on Debts. Status: Default, but **Design** for the last three: they change a drawn screen.

## Detail screens not drawn
Account detail (balance, its transactions, edit, adjust, archive, delete when empty); Category detail (month total, entries); Allowed senders (list, add sender name or number, choose the account and default category, per-sender rule preview, enable/disable) [P4]; CSV statement import (choose file, map columns once, preview, skip duplicates); Share-to-add (receive text, show parsed Add screen) [P4]; Recurring item detail; Person detail (all debts and splits with that person); Debt/Split "⋯" menus. Default.

## Elements the design does not show that the rules need
- Duplicate button on Detail (one-tap repeat, in SPEC but in no screen). **Default · Design**
- Pills on Detail and History for debt movements, adjustments and upcoming entries. Default
- Previous/next period arrows on Reports. Without them past months cannot be seen. **Default · Design**
- "Overdue" red pill on Debts. Default
- Pro tag on Export report for free users. Default
- Delete debt/split and Edit in a menu; Remind setting. Default
- Debts and Splits rows in More (Debts is otherwise reachable only from Home's debt boxes). Default
- "Allowed senders → Review entries" link, since the Home bell hides at 0. Default

## Rules that were missing or contradictory (now written, mark Decide where noted)
- Month naming when the start day is not 1 (named by the month it starts in). Default
- Future-dated entries count from their date and show as Upcoming. Default
- Free limits count open loans and open splits only, not split shares, on-behalf or held debts. **Decide** (alternative: lifetime counts)
- Split paid by someone else: the user's share is an expense with no account plus a debt to the payer. Default
- A capture with no category opens Add instead of saving as Uncategorized. Default
- Installments on a debt are a recurring debt payment; recurring transfers are out of v1. **Decide** (alternative: allow recurring transfers for "Savings circle"-style saving)
- Budget percent shows 99% until the limit is reached; legend percentages add to 100. Default
- Adjust balance creates an `adjustment` entry that is neither income nor spending. Default

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
Reminders (daily "log your day" time), Auto-lock, Change PIN, Debts and Splits shortcuts, About and Help (version, privacy statement "all data stays on this phone", contact), Dark theme tokens (a `dark` theme in `tokens.json` and a contrast check), Delete all data (confirm twice). Backup file encryption is a Decide below.

## Pro and advanced [P4]
Pro success state; Restore result messages; Multi-currency flow (account currency, manual exchange rates, base currency for totals, rates screen, transfers between currencies, per-currency debts); Home-screen widget (Add expense, balance); iPhone Shortcuts setup guide (in-app steps with a ready template); Android notification-listener permission explainer; CSV statement import (listed in SPEC but in no phase: it is now P2, after export).

## Store and release items [P4 end]
App name and icon, privacy policy page and store privacy answers ("no data collected"), store screenshots, age rating, product IDs and price, a support contact. All Decide/owner.

## Technical feasibility to confirm in Task 1 (before building the phase that needs them)
These depend on what NativePHP Mobile supports today; check the docs and licence first and tell the owner if any is not possible.
- Secure storage for the PIN hash, biometrics, local notifications and scheduled reminders, background task for recurring, file save/share/pick, in-app purchases (both stores), share-to-app (Android share intent, iOS share extension), home-screen widget (iOS WidgetKit is native Swift), Shortcuts action (iOS App Intents), Android SMS/notification listener.
- Google Play restricts the SMS permission; an expense app usually needs a policy declaration and may be refused. Safer default: read notifications only, SMS later if approved. **Decide** after the check.
- iOS cannot read SMS or other apps' notifications, so iPhone auto-capture stays Shortcuts-only.
- Local notifications are limited on iOS (64 pending); schedule the next ones and reschedule on app open.

## Open decisions for the owner
1. Forgot PIN policy (see above).
2. Whether changing the currency in Settings should be blocked once data exists (recommended: allow display change with a clear warning; real conversion only in Pro multi-currency).
3. App name and logo. "Pocketbook" is a placeholder.
4. Pro price and store product ID.
5. Languages to ship first. The setup screen lists English, Urdu, Arabic, Spanish and French; recommended v1 is English plus Urdu and Arabic to test right-to-left early. All text goes through language files from day one either way.
6. Whether to ship light-only v1 and add Dark later (recommended). If yes, remove the Theme row until Dark exists.
7. Free-limit counting (open vs lifetime) and what counts as a debt.
8. Backup encryption: recommended v1 is an unencrypted file with a warning; optional password later.
9. Whether recurring transfers are allowed.
10. Android capture route: notifications only, or SMS too (store policy risk).
11. Numerals for Arabic and Urdu.
12. Backup staleness: "Safe" for 14 days, then amber "Old" (default; change if wanted).
