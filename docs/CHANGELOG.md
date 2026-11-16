# Changelog

## Owner decisions applied (2026-11-16)
- Free limits count open items only (was Decide; now written as final in SPEC and GAPS).
- Recurring transfers are not allowed in v1 (final).
- Backup is unencrypted in v1. Added a "Backups are not encrypted" note to the Backup and export spec (needs drawing).
- Android capture: notifications first, SMS optional and later. SPEC, SCREENS (Review entries), FLOWS 27 and GAPS updated; Pro screen copy to change to "notifications and Shortcuts" (needs design).
- Light-only v1: removed the Theme row from the Settings spec (SCREENS 19, phase gating, slips list), the `theme` setting from SPEC, and the Theme item from FLOWS 19. Dark tokens moved to a later release in GAPS.
- Currency is locked once any entry exists (SCREENS 19, FLOWS 19, SPEC). Replaces "display change with a warning".
- `CLAUDE.md` Phase 2 now lists CSV statement import.
- `docs/GAPS.md`: added "Resolved decisions" and cut the open list from 12 to 6 (Forgot PIN, app name, Pro price, languages, numerals, backup staleness).

## Task 0 audit (2026-11-16)

Method: read every doc, extracted the visible text of all 21 static screens and compared each with `docs/SCREENS.md`, checked the sample numbers in the designs for arithmetic, then traced every screen and rule through `docs/FLOWS.md` and the data model in `docs/SPEC.md`. The design files were not changed (they are exports); their slips are listed at the end of `docs/SCREENS.md`.

## Contradictions fixed
- **Debt payments were modelled twice** (a `debt_payments` table and a Debt-payment entry in History/balances). Now one model: each payment with an account owns one `debt_out`/`debt_in` transaction; balances read only from transactions. (SPEC)
- **Balance formula** ignored adjustments, which Accounts creates. Added the `adjustment` type and rewrote the formula. (SPEC)
- **Free limits** were "3 splits and 3 debts" (SPEC), "active debts" (SCREENS) and "used 3 free splits" (GAPS), and the Debts design shows 6 debts. Now: 3 open loans and 3 open splits; split shares, on-behalf and held debts do not count. Owner decision flagged. (SPEC, SCREENS, GAPS)
- **Phases vs Pro:** export is P2 and limits are P3 but Pro is P4. Added an `Entitlements` service that returns Pro in dev builds. (SPEC)
- **Phase 1 screens draw later-phase elements** (quick add, bell, debt boxes, On behalf link, Reports tab). Added a phase-gating rule. (SCREENS)
- **"Add saves exactly as parsed"** vs required category when a capture says "Uncategorized". Add now opens prefilled in that case. (SCREENS)
- **Month start day:** month naming was undefined. Now named by the month it starts in. (SPEC)
- **Future dates** were allowed but no rule said when they count. Now: shown as Upcoming, counted from their date. (SPEC, SCREENS)
- **"Skip setup" on a new phone** had no screen to do it. Added the Restore link on Welcome. (SCREENS, FLOWS, GAPS)
- **Split paid by someone else** had an expense but no account. Defined (expense with no account plus debt to the payer). (SPEC)
- **Home Debts reachable** only from Home boxes; **Split a bill** had no entry on any drawn screen (FLOWS invented "Debts → Add debt → Split"). Added More rows and a Debts button, both flagged for design. (SCREENS, FLOWS)
- **Review entries unreachable at zero** because the bell hides. Added the Allowed senders path. (SCREENS, FLOWS)
- **"Saved this month"** label vs period chips driving it. Label now follows the period. (SCREENS)
- **Default categories:** the design shows "Rent and home, 2 subcategories" but the defaults had none. Added Rent and Maintenance. (SCREENS)
- **Navigation:** the bar/Back rules were wrong for Accounts, Categories, Budgets and Debts (bar with More highlighted, no Back). Rewritten. (SCREENS, SPEC)

## Data model changes (SPEC)
- `occurred_at` → `occurred_on` (local date, no time zone drift).
- `transactions`: added `source`, `recurring_id`, `capture_id`, `updated_at`; `account_id` nullable; types extended; unique `(recurring_id, occurred_on)` so recurring never double posts.
- Added `tags`/`transaction_tags`, `captures`, `allowed_senders`, `exchange_rates`.
- `recurring`: added `debt_id`, `month`, `start_on`; `next_run_at` → `next_run_on`. `debts`: added `installment_minor`. `splits`: added `category_id`, `paid_by_person_id`; `split_shares`: added `input_value` so splits can be edited.
- `debt_payments` redefined (see above).
- Security settings and entitlement are kept out of the database and the backup.

## Rules added
- Rounding, largest remainder for splits and legend percentages, percent display (SPEC).
- Week and year definitions; rollup of subcategories into parents for reports and budgets; one budget per category; no rollover (SPEC, SCREENS).
- Quick-add parsing rules (FLOWS 4, the old text said "see FLOWS" but had none).
- PIN lockout schedule, auto-lock options, app-switcher cover (SCREENS 2).
- Backup staleness (Safe under 14 days), unencrypted warning, error cases, rescheduling after restore (SCREENS 20).
- Archive and delete rules for accounts; category move-on-delete; debt reminders on the due date; debt Overdue state; Duplicate for one-tap repeat (in SPEC, absent in all screens).
- Notification permission handling; phase gating list.

## Added
- `docs/FLOWS.md`: flows 20–27 (free limit, accounts, categories, edit/delete debt or split, forgot PIN, daily reminder, share-to-add, capture setup) and a reachability check of all 21 screens.
- `docs/SCREENS.md`: "Known sample slips" list.
- `docs/GAPS.md`: reorganised; added design-needed markers, missing-element list, rules-now-written list, technical feasibility checklist, store/release items and 12 open decisions.
- `CSV statement import` placed in P2 (was in SPEC with no phase).

## Not changed
`CLAUDE.md`, the design files, tokens and component guidelines are untouched. The 21 screen files each match their SCREENS entry for every visible element; differences found are in the "slips" list.
