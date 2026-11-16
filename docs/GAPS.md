# Pocketbook: what the design does not show yet

The 30 designed screens cover the main path and the setup, PIN, capture and add/edit forms. Everything below is needed for a complete app but is not drawn. For each: what it must do, and a recommended default so it can be built. Build these with the design system tokens and the existing patterns (Field rows, Chips, Buttons, Cards). Mark a design request in your report so the owner can have it designed.

Status key: **Decide** = needs the owner's call (a default is given so work is not blocked); **Default** = safe to build as described. **Design** = a drawn screen or change is needed before the owner would call it finished; the default is to build it from existing components in the meantime. Phase in brackets.

## Drawn since the first audit (no longer gaps)
Restore link on Welcome; Create PIN, Confirm PIN (error state), Forgot PIN; Forgot PIN? link on Lock; Allowed senders (Android list, iPhone steps); New account, New category, New budget, New recurring item, Add debt; Split a bill button and the Overdue pill on Debts; "Debts and splits" row in More; Allowed senders opens its screen. They are written up as SCREENS 22–30.

## Setup and security
| Item | Needed behavior | Status |
| --- | --- | --- |
| Normal Confirm PIN state [P1] | The static screen shows only the error state; build the normal state with accent dots and no message | Default |
| Biometric-off state on Forgot PIN [P1] | Hide the Face ID card and button; say it is not set up; only Erase remains | Default · Design |
| Erase confirm [P1] | "Erase everything? This deletes all data on this phone and cannot be undone." Cancel / Erase | Default · Design |
| Change PIN [P1] | A row in More → Security: verify with biometric if on, else the current PIN, then Create and Confirm | Default · Design |
| Auto-lock time [P1] | Immediately, 1 minute, 5 minutes (default 1 minute); a row in Security | Default · Design |
| Lockout schedule [P1] | 30 s after 5 wrong tries, 1 min after 10, 5 min after 15, then 15 min per 5; kept across app restarts | Default |
| App-switcher privacy cover [P1] | Hide balances in the app switcher; Android secure window | Default |
| Currency lock [P1] | Currency is read-only once any entry exists (see Resolved decisions) | Default |
| Language and currency pickers [P1] | Searchable lists | Default |
| Numerals for Arabic and Urdu [P1] | Show Western digits (0–9) or locale digits? Recommended: Western digits in v1 for input reliability | Decide |

## Pickers and sheets
Category picker (with subcategories, search, New category); Account picker (with balances and a "No account" option on Add debt); Date picker; Month picker; Custom date-range picker; Person picker (with Add person); Day-of-month picker (1–28 for month start; 1–31 for recurring); Repeats, On day and Remind pickers on New recurring item; Inside (parent) picker on New category; Filter sheet in History (account, category, amount range; lives in the month chip's sheet); Share sheet (system); Discard-changes confirm; Upcoming (future-dated) marker. All Default; none drawn.

## Forms and states of drawn forms not drawn
- Edit modes of New account, New category, New budget and New recurring item (titles "Edit …", Archive/Delete/Adjust balance, pause). Default
- Edit debt (due date, note, installment); edit or delete split; Add person; Adjust account balance. Default
- Add sender (name or number, account, optional default category), Android notification-access explainer and "access is off" state, iPhone-only version of Allowed senders (no list, "Open Shortcuts"). **Default · Design**
- Free-tier Currency row on New account (read-only, no chevron). Default
- Validation and error states of every form (empty name, duplicate name, zero amount, split not adding up). Default
- **Date row on Add debt** (Debt detail shows "Lent 12 Oct", the form has no date). **Design**
- **Date and Category rows on Split a bill.** **Design**
- **Tags on Add** (free-text chips). **Design**
- Full icon set for categories: only 8 icons are drawn. **Decide** the final list (recommended ~24 neutral glyphs covering the default categories).

## Detail screens not drawn
Account detail (balance, its transactions, edit, adjust, archive, delete when empty; the Accounts cards have no link yet); Category detail (month total, entries); CSV statement import (choose file, map columns once, preview, skip duplicates); Share-to-add (receive text, show parsed Add screen) [P4]; Recurring item detail; Person detail (all debts and splits with that person); Debt/Split "⋯" menus. Default.

## Elements the design does not show that the rules need
- **Duplicate** button on Detail (one-tap repeat, in SPEC but in no screen). **Design**
- Pills on Detail and History for debt movements, adjustments and upcoming entries. Default
- **Previous/next period arrows** on Reports. Without them past months cannot be seen. **Design**
- **Plain-file note** on Backup and export, and the **Pro tag** on Export report for free users. **Design**
- **Pro screen copy**: "SMS and Shortcuts" should read "notifications and Shortcuts". **Design**
- **Review entries link** on Allowed senders (the bell hides at 0 and Senders only links out). **Design**
- **Theme row** is still drawn on More; v1 is light-only, so omit it. **Design** (remove)
- **Auto-lock, Change PIN, Reminders, About and help, Delete all data** rows on More. **Design**
- Delete debt/split and Edit in a menu; Remind setting. Default
- **Installment** on Add debt: Debts shows "Installment $50 monthly" but no form creates one. **Decide** (recommended: an optional "Installment" row on Add debt that creates a recurring debt payment; or drop installments from v1)

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
Reminders (daily "log your day" time), Auto-lock, Change PIN, About and Help (version, privacy statement "all data stays on this phone", contact), Dark theme (later release: a `dark` theme in `tokens.json`, a contrast check, and the Theme row; not in v1), Delete all data (confirm twice).

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
8. Forgot PIN is designed as two options: biometric, or erase and start over (then restore from a backup). Erase asks for confirmation.

## Open decisions for the owner
1. App name and logo. "Pocketbook" is a placeholder.
2. Pro price and store product ID.
3. Languages to ship first. The setup screen lists English, Urdu, Arabic, Spanish and French; recommended v1 is English plus Urdu and Arabic to test right-to-left early. All text goes through language files from day one either way.
4. Numerals for Arabic and Urdu (default: Western digits).
5. Backup staleness: "Safe" for 14 days, then amber "Old" (default; change if wanted).
6. Debt installments: add an Installment row to Add debt, or drop installments from v1.
7. The final category icon list.
