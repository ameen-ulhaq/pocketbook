# Pocketbook: project guide for Claude Code

Pocketbook is a simple personal expense manager for iPhone and Android, for any region. It works fully offline, stores everything on the phone, and has a one-time lifetime Pro purchase. Built with Laravel and NativePHP Mobile.

## Read these first, in this order

1. `docs/SPEC.md`: product decisions, features, money rules, data model, phases.
2. `docs/SCREENS.md`: every one of the 30 designed screens with every element, option, action, state and rule.
3. `docs/FLOWS.md`: step-by-step user journeys.
4. `docs/GAPS.md`: what the design does not draw yet, with recommended defaults and open decisions.
5. `docs/design/`: tokens (`tokens.json`, `tokens.css`), the 30 screens as static HTML (`screens/index.html`), and the component guidelines and previews (`components/`).

Online (private to the owner, may not open for you): screens https://claude.ai/artifact/McKtWDuTK1kgQKSnffzJrs and design system https://claude.ai/artifact/JkVyy5F7v7a6TFdGPV2Tnc. The files in `docs/design/` are the portable copy and win if they differ.

## Stack

- PHP 8.4, latest stable Laravel, SQLite on device, Blade + Livewire (or what the current NativePHP Mobile docs recommend for web-view UI), Tailwind with the tokens as CSS variables.
- `nativephp/mobile`. Check the current docs (https://nativephp.com/docs/mobile) and licence requirements before installing; v3 was stable and v4 in beta when this was written.
- No server, no accounts, no network calls in v1.

## Rules

1. **Simplicity is the product.** One job per screen, big amounts, few colors. Do not add features or settings that are not in the spec.
2. **Money is integers.** Store amounts as integer minor units plus a currency code. Never use floats for money. Format with the device locale.
3. **Tokens, not hex.** Every color, size, radius and spacing comes from the tokens as CSS variables. The static screens use raw hex only because they are exports.
4. **Equal side padding.** Screens use 20px on both sides; full-width buttons fill that width.
5. **Offline only.** Nothing in v1 may need the network.
6. **Tests for the money logic.** Balances, transfers, budgets, debts and splits need feature tests. Run `php artisan test` before every commit.
7. **Small commits**, one feature each, clear messages.
8. **Ask before** adding a dependency that is not in this file or the spec.
9. **Docs stay true.** If you change behavior, update the matching doc in the same commit and note it in `docs/CHANGELOG.md`.

## Phases

1. **Core:** setup, PIN/biometric lock, accounts, categories, add/edit/delete transactions, history with search and filter, home, detail, settings, backup and restore.
2. **Insights:** budgets with alerts, recurring items, reports and charts, CSV/PDF export, CSV statement import, one-line quick entry.
3. **People and money:** debts, shared expenses and bill splitting, paid-on-behalf entries.
4. **Advanced:** lifetime Pro purchase, share-to-add, home-screen widget, multi-currency, Android auto-capture of SMS/notifications, iPhone Shortcuts action.

Finish and test each phase before starting the next.

## Task 0: audit before building

Before writing any code, check that the docs are complete and correct, and fix them:

1. Open each file in `docs/design/screens/` and compare it with its entry in `docs/SCREENS.md`. For every screen confirm that every visible element, option, button, link and state is described, and that the described behavior is consistent with `docs/SPEC.md` and `docs/FLOWS.md`.
2. Check that every screen is reachable in `docs/FLOWS.md` and every flow ends on a real screen.
3. Check the money rules and data model in `docs/SPEC.md` against the flows (balances, splits, debts, on-behalf, transfers, budgets, month start day). Look for contradictions and cases that do not work.
4. Check `docs/GAPS.md` is complete for a shippable app. Add anything missing; mark each item Default or Decide.
5. Update the docs directly where they are wrong, missing or unclear. Keep the same structure. Improve them wherever you judge it better, and explain why.
6. Write `docs/CHANGELOG.md` listing every change, and give the owner a summary: what you fixed, what you added, what is still undecided, and what needs a design.

Then wait for the owner to confirm before starting Task 1.

## Task 1: set up and build Phase 1

Set up the project (Laravel + NativePHP + SQLite + Tailwind with tokens), get a blank screen running on the iOS simulator and an Android emulator, then build Phase 1 in this order: schema and seeders, setup flow, accounts and categories, add transaction, history and detail, home, lock, settings, backup and restore. After each step run the tests and tell the owner what to look at in the simulator.
