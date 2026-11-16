# Changelog

## Design update 3: 39 screens, gold tokens (2026-11-16)
- **Tokens:** added `gold-tint` `#F6EBCF` and `gold-ink` `#7A5B12` to `tokens.json` and `tokens.css` (Pro tag only). Contrast checked with the WCAG formula: gold-ink on gold-tint 5.31:1, on white 6.30:1, on the page background 5.87:1. All pass 4.5:1 for body text.
- **New screens 31 to 39 in SCREENS:** Choose an icon, Auto-lock, Reminders, About and help, Erase everything, Add sender, Auto-capture on iPhone, Forgot PIN (no biometrics), Change PIN.
- **Changed screens:** More now has Reminders, Auto-lock, Change PIN, About and help and Delete all data; New category has a See all link to the icon picker; Forgot PIN and Allowed senders link to the new erase and Add sender screens. SCREENS 2, 11, 19, 24, 25 and 27 rewritten.
- **FLOWS:** Change PIN (flow 28), reminders (25), capture setup with the message test and the iPhone guide (27), forgot PIN with and without biometrics (2), new flows 29 (Delete all data), 30 (icons) and 31 (auto-lock); reachability updated.
- **SPEC:** settings list rewritten (reminders, auto-lock with a 15 minute option, change PIN, delete all data, no theme); reminder keys in `settings`; screen count 39.
- **Static HTML:** 9 new screens and 4 changed ones in `docs/design/screens/`; index shows all 39.
- **GAPS:** the matching design items removed; icon list replaced with the 24 actually drawn and a new default category mapping (the earlier proposed list is gone).
- **New findings:** Add sender has no Account or default category row; Help and tips has no page or text; Erase everything's Back and Cancel always go to More; the Weekly backup switch exists on both Reminders and Backup and export (one setting); the Reminders screen holds items from three phases, so rows appear as their features land.

## Design update 2: 10 screens refreshed (2026-11-16)
Re-read from the canvas; docs and `docs/design/screens/` now match.
- **Add:** Tags row under Note. **Detail:** Duplicate beside Edit. **Split a bill:** Date and Category rows. **Reports (both tabs):** period arrows around the period label, next dimmed at the current period.
- **Backup and export:** "Backups are not encrypted" note; gold Pro tag on Export report. **More:** Theme row removed. **Allowed senders:** Review entries button. **Add debt:** Date row and Installment row (the static screen shows I borrowed, with "Lender"). **Pro:** copy now "notifications and Shortcuts".
- SCREENS: those entries rewritten; phase gating and sample-slips lists updated; the Pro tag and Date rules added.
- SPEC: gold rule widened to the Pro banner and Pro tag.
- GAPS: the matching design items removed; new "Design still needed" short list (Pro tag colors as tokens, iPhone Allowed senders layout, More rows, Forgot PIN states, Add sender form, 16 icons).
- New finding: the Pro tag colors are not tokens.

## Owner decisions: v1 details (2026-11-16)
- Backup shows "Old" (amber) after 14 days, "Safe" before.
- Western digits (0–9) in all languages for v1.
- Language order: English first, then Urdu and Arabic.
- App name stays Pocketbook for now.
- Pro price is decided at the start of Phase 4.
- Category icons: about 24 from the existing icon set; the list and the default mapping are in GAPS.
- Allowed senders on iPhone shows only the Shortcuts guide.
- GAPS: these moved to Resolved decisions 9–15; the open decisions list is now empty.

## Decision: debt installments (2026-11-16)
- Kept. Add debt gets an optional Installment row (I borrowed, with an account). It creates a recurring debt payment that posts a `debt_out` payment each due date, capped at the remaining balance, and stops at zero. No installments for money the user lent. Updated SCREENS 14, 28 and 30, FLOWS 11, GAPS (moved to resolved; the row still needs drawing).

## Design update: 9 new screens, 4 changed screens (2026-11-16)

Read from the design canvas and re-audited every doc against all 30 screens.

**Added to the docs**
- `docs/SCREENS.md`: screens 22 to 30 (Create PIN, Confirm PIN, Forgot PIN, Allowed senders, New account, New category, New budget, New recurring item, Add debt), each with elements, behavior and rules; updated Welcome, Lock, Debts and More; screen count 21 → 30 in CLAUDE.md, SPEC, SCREENS, GAPS, KICKOFF and design README.
- `docs/FLOWS.md`: first launch now includes Create/Confirm PIN; Forgot PIN and Erase flow; budget, recurring, debt, account and category flows use the new forms; new flow 28 (App lock on/off and Change PIN); reachability check lists all 30.
- `docs/SPEC.md`: screen rows 5 and 6; `budgets.alerts_on`; account types.
- `docs/design/screens/`: static HTML for the 9 new screens plus the changed Welcome, Lock, Debts and More; index rebuilt with all 30.
- `docs/GAPS.md`: "Drawn since the first audit" list; items now designed removed from the needs-a-design lists; Forgot PIN moved to Resolved.

**New findings from the new screens (all in GAPS)**
- Installments: Debts shows one but no form can create it. Decision needed.
- Add debt has no Date row; Split a bill still has no Date or Category row.
- Allowed senders has no link to Review entries, no Add sender form, no iPhone-only layout.
- Forgot PIN has no biometric-off state and no erase confirmation; Change PIN is not in More and must accept biometric.
- Confirm PIN is drawn only in its error state.
- Category icon grid shows 8 icons; the final set must be listed.
- The Welcome restore link opens the Backup screen in the design; build a direct file picker.
- Not changed in the canvas and still needing design: Duplicate (Detail), Tags (Add), Date and Category (Split a bill), period arrows (Reports), backup note and Pro tag (Backup), Pro screen copy, removing the Theme row, and the Auto-lock, Change PIN, Reminders, About and Delete all data rows.

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
