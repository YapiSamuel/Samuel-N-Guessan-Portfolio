Repo: https://github.com/YapiSamuel/BudgetBuddy---Finance-Tracker-App

# 💰 BudgetBuddy — Free Excel Budget Tracker Generator

**Free and open source. No account, no signup, no payment.**

BudgetBuddy answers a question most budget templates get wrong: *why should your finances fit someone else's spreadsheet?* A guided wizard asks about your real accounts, budgets and categories, then generates a working Excel tracker built around them — live formulas, dropdown validation, conditional formatting and charts, all wired together automatically. It is released free for anyone to use, fork or learn from, and remains a work in progress.

## 🔍 Features
- Generates a six-sheet Excel workbook from your answers — Dashboard, Transactions, Envelopes, Accounts, Subscriptions and a month-selectable Summary — with `SUMIF`/`SUMIFS` formulas, data-validation dropdowns and conditional formatting wired between sheets
- Multi-step React wizard with a live preview of the tracker as you answer, bilingual in English and French, with progress persisted to `localStorage` and a graceful in-memory fallback when storage is blocked
- Server-side validation bounds every request before work begins: caps on accounts, budgets and totals, a 256 KB body limit, and rejection of characters that would break Excel dropdown lists
- Per-IP rate limiting on the expensive endpoints, a CORS origin allowlist, and generic client-facing errors with real detail kept in the server log
- Degrades instead of failing — if the API is unreachable the browser builds a simpler workbook locally with SheetJS, so the user still gets a file
- Runs with zero configuration and costs nothing to use

## 🧰 Skills Demonstrated
- **Trust-boundary design:** the project was originally built to be sold, and the authorization model from that phase is its most valuable engineering. The client never decided whether it had access; that flag was written in exactly one place — a webhook whose signature was verified server-side — and the download route refused to build anything without it. Every value from the browser is treated as a suggestion, never a fact. The application is now free, so nothing is ever charged, but the model is intact and worth reading
- **Security review of my own code:** audited the application end to end and documented six findings with verified proof-of-concept, including an Excel formula-injection path where an unvalidated field is written by `openpyxl` as a live formula rather than text, and a set of malformed inputs that crash the validator instead of rejecting them. Findings are recorded with severity and remediation; fixes are in progress
- **Defensive input handling:** validating hostile input at the boundary, bounding CPU and memory per request, and treating a validation function as a security control rather than a convenience
- Full-stack development: Python (Flask, `openpyxl`, SQLite) on the back end, React and Vite on the front, with a REST contract between them
- Integration with a third-party payment provider — Stripe Checkout and signature-verified webhooks — including how to make such a flow inert without tearing it out
- Debugging at the file-format level — diagnosing broken charts by reading the generated OOXML directly and correcting axis positioning and category references that the library emitted incorrectly

## 🎯 Outcome
A complete product built solo, from a wizard someone can actually use through to a generated file that works when opened. The most valuable part was not the spreadsheet engine but everything defending it: deciding what the client is allowed to assert, bounding what a single request can cost, and then auditing my own work honestly enough to write down what I found wrong with it. Building it to a standard where it could have handled money forced a discipline I would not have reached otherwise — assume every request is lying, and make the server the only thing that decides. I have since released it free rather than selling it, because a tool that saves someone an afternoon of spreadsheet work is worth more shared than paywalled. It is open source and still under active development.
