# Export Cash Recovery

A working prototype: a student submission to Razorpay. It tracks the realisation deadline for export invoices, matches bank credits by reference, drafts a reminder for any shortfall, and queues that reminder only after consent is on record and the merchant approves it. **The app never sends anything to a buyer.** Queued means stored and waiting.

The visual design follows [DESIGN_GUIDE.md](DESIGN_GUIDE.md). All business rules live on the server in `server/rules/`; the front end only displays what the API returns.

## Run it

Needs Node.js 18.15 or later (written for Node 20; tested on 18.15).

```
npm install
npm run seed
npm start
npm test
```

1. `npm install` installs Express and Vitest.
2. `npm run seed` creates `data/store.json` with the seeded transaction history. Run it again to start over.
3. `npm start` starts the server. Open <http://localhost:3000>. Do not open `public/index.html` from the file system: the styles, logo and API are served by the server, so the page will look broken.
4. `npm test` runs the Vitest tests in `tests/rules.test.js`.

## What is in the app

**Seeded history.** `npm run seed` (and "Reset demo data") creates the same data every time, using a fixed random seed: 8 buyers in the US, UK, Germany, Singapore and UAE (2 without consent), 45 export invoices exported between October 2025 and October 2026 in USD, EUR and GBP (10 in INR), their bank credits and a backfilled audit log. Most invoices are realised; some are partially paid, a few are unpaid and on track, 2 are breached, 1 is on the Caution List, one was over-paid, and 3 credits arrived with no reference. "Today" is **13 October 2026**.

**Invoices.** A summary (total outstanding, shortfall recovered, realised this month, deadlines within 60 days, breached, unmatched credits), then every invoice with search by reference or buyer, filters (status, buyer, currency, deadline state), column sorting and 15 per page. Open an invoice to see its deadline, see when and how the buyer's consent arrived, record a credit, or unmatch one. "New invoice" creates one for an existing or a new buyer, and is where the INR and Caution List flags are set.

**Transactions.** The ledger of bank credits, matched and unmatched, with search, currency and date filters and an Unmatched badge. "Record a credit" takes an amount, currency, optional invoice reference, received date and note. Unmatched credits can be matched by hand to an open invoice; matched credits can be unmatched.

**Reminders.** Opens on the list of buyers who need a reminder (any invoice with a balance), soonest deadline first, with a **Remind** button on the right. Remind opens the reminder already drafted, in a text area you can edit, with **Approve and queue**. Approval needs the buyer's consent and only queues the reminder: queued reminders are listed below as "Queued, not sent". Nothing is sent. If no payment has arrived, the wording says so; it only mentions a short payment when one has been received.

**Consent.** Consent is the buyer's to give, so the app only records it: "Consent recorded on 9 Oct 2025 via WhatsApp reply" (or "No consent on record", or "withdrawn"). Nothing lets the merchant tick it. Because there is no real buyer channel, a clearly labelled **Demo only: Simulate buyer response** box (Buyer agrees / Buyer opts out) stands in for the buyer's reply on the reminder, invoice and buyer pages, and the record then reads "via buyer link (demo)". A new buyer added with "The buyer's consent is already on file" is recorded as a merchant entry.

**Audit log.** Coloured count chips show how many events of each kind there are (click one to filter, click again to clear). Each row shows the time, the event, the buyer and the invoice it is about, and an **Open** link to the invoice, reminder, buyer or agent page it relates to. Filter by event type, invoice and date range; hover over a row for the details; "Export CSV" downloads the filtered log (columns `time`, `type`, `invoice`, `message`).

**Profile card** (top right): the exporter's details and "Reset demo data", which asks for confirmation and restores the seeded history exactly.

## Riri, the Recovery Agent (the AI part)

An AI-native workflow for **revenue recovery**, in the direction of Razorpay's Agentic Dashboard and Agent Studio: an agent that does the legwork, a merchant who stays in control, and a record of what it did and why. Open the **Agent** tab.

**What it does.** "Run the agent" reads every open invoice and every unmatched credit and proposes the next best actions, ranked:

| Proposal | What it is | What applying it does |
|---|---|---|
| Match credit | An unmatched bank credit that probably pays an open invoice (same currency, amount fits the balance, arrived after the export). Shown with a confidence score. | Matches the credit and recalculates the invoice. Reversible by unmatching. |
| Draft reminder | An open invoice whose buyer has consented. | Stores a draft. It is not approved, queued or sent. |
| Review draft | A draft already waiting. | Opens it. Approving is yours. |
| Consent needed | A balance is owed but the buyer has not consented, so no reminder can be approved. | Marks it as handled. |
| Needs you | The deadline has passed, or the exporter is on the Caution List. | Marks it as handled. It needs a decision, not an automatic step. |

Each proposal shows its reasoning, its evidence and the guardrail checks it passed. Applying or dismissing it is logged in the audit log with the reason.

**Match a bank statement.** Paste lines copied from a bank statement and press "Read statement" (or "Use an example"). For each line the agent works out who paid, how much, in what currency, and which invoice or invoices it covers, then shows its reasons. You record each line yourself.

- **Messy lines.** It reads amounts such as `USD 12,000.00`, `12,000 usd` or `£432.50`, dates written several ways, and references typed carelessly (`EX1O42` with a letter O, `INV 1042`, `EX-1008, 1044 and EX-1045`). A buyer named with a typo (`NORTHWND`) is still found.
- **One credit, several invoices.** When a transfer equals what several of one buyer's open invoices add up to (or is a little short, the size of bank charges), it is proposed as a split. The oldest invoices are paid in full first, so one balance is left to chase. It is recorded as one credit per invoice that share a `split_group` and add up to the bank amount, so every other rule keeps working and each part can be unmatched on its own.
- **Buyer history.** It looks at how that buyer has paid before (how short, how many days after export). A shortfall in line with their usual one raises confidence, and the reason says so.
- **Guardrails.** Same currency; the invoice must be open; never more than an invoice owes in a split; a line below your confidence threshold, with no date, or that may already be in the ledger asks you to confirm; the server works the proposal out again when you record it and refuses one that has gone stale or been altered; every record writes an audit event with the reasons. Nothing is recorded by reading, and nothing is sent.
- **Where the model comes in.** With `ANTHROPIC_API_KEY` set, Claude also reads each line to help with messy text. It is sent only the pasted lines, never invoices or buyers, and every value it returns must be visible in the line (an amount that is a number in the line, a currency written in it, a reference whose digits are in it) or it is dropped. It never chooses where money goes: the rules do. Without a key the rules read the lines and the screen says "Read by rules". (Tested against a mocked API; not yet run against a live key.)

**Ask the agent.** Plain-language questions ("What should I do today?", "Which invoices are at risk?", "Any unmatched credits?", "What does Merlion owe me?") answered from the data, with links to the invoices named. Questions are read-only.

**Guardrails you set.** Agent on or paused. **Review first** (the agent only proposes) or **Auto within guardrails** (it applies the low-risk actions itself: a credit match at or above your confidence threshold, and reminder drafts). Also: the most proposals per run, and how old an invoice must be before it is chased.

**What it can never do, in any mode:** send anything to a buyer, approve or queue a reminder, contact a buyer without consent on record, act while paused, or add an action that the rules did not propose.

**Where the model comes in.** The *rules* decide what can be proposed, so results are repeatable and testable. With `ANTHROPIC_API_KEY` set, Claude then (1) puts the proposals in the order it thinks is most worth doing and re-words each reason, (2) writes the reminder drafts, and (3) answers questions. Everything it returns is checked: it cannot add, remove or change a proposal, cannot cite an invoice it was not given, and receives no buyer emails. If a check fails or the call errors, the rules answer is used and the run says so. With no key the whole agent works on rules alone, and the screen says "Reasoning by rules".

| Razorpay direction | Here |
|---|---|
| Agentic Dashboard: ask in plain language | Ask the agent |
| Intelligent reconciliation | Match-credit proposals with confidence and evidence; pasted bank statements read and matched, including one credit that pays several invoices |
| Active revenue recovery | Ranked recovery plan; reminder drafts |
| Autonomous guardrails | Mode, confidence threshold, limits, pause |
| Agent Studio: review-first or automated | Review first / Auto within guardrails |
| Consent verified before outreach | Consent gate; no consent means no reminder |
| Irreversible or customer-facing actions need explicit approval | The agent never approves or sends |
| Every action logged with what, when and why | Audit log, including the agent's reasons |
| Agent performance dashboard | "How the agent is doing": runs, applied, dismissed, acceptance, credits matched |

### A walkthrough

A five-step story works on the invoice **EX-1042** (Northwind Trading LLC, US, USD 12,000, exported 2 October 2026, unpaid, consent on record). Find it in the Invoices list:

1. **Invoices.** Open EX-1042: 262 days left, deadline 2 Jul 2027 (9 months from export). An INR invoice gets 12 months, and an invoice on the Caution List shows "Existing RBI order applies" instead of a date; both are set when the invoice is created.
2. **Transactions.** Choose "Record a credit" (or use the same button on the invoice page, which fills in the reference), enter USD 11,760 and save. It is matched by reference, the shortfall of USD 240 is recorded as "likely intermediary (SWIFT) charges", and the status becomes Partially paid.
3. **Reminders.** EX-1042 appears in "Buyers to remind". Click **Remind** to open the drafted reminder. If the buyer's consent is missing (Albion and Rhein in the seed), "Approve and queue" is disabled and the server answers `409` if called directly; the demo box records the buyer's reply. With consent on record, approving sets the status to Queued. Nothing is sent.
4. **Transactions again.** Record the USD 240 balance. The invoice becomes Realised and the countdown stops.
5. **Audit log.** Every change, newest first.

The **Agent** tab can do steps 2 to 3 for you: it proposes matching an unmatched credit, or drafting the reminder, and you apply or dismiss each proposal.

Everything is stored in `data/store.json`, so it survives a page reload and a server restart. To start over, use "Reset demo data" in the profile card, or stop the server, run `npm run seed` and start it again.

### Two sets of rules

The original order-by-order endpoints (listed first in the API tables) keep the strict order in DESIGN_GUIDE.md section 12 and are still tested; the screens no longer use them. The ledger, the Reminders view and the other screens work on any invoice in any order, so the invoice status there is derived from the facts (see section 12a of the guide): realised when the credits cover the invoice, queued when a reminder is approved, partially paid when something is matched, otherwise unpaid. Amounts in different currencies are never added together; totals are listed per currency. The guide has no rule for exports before 1 October 2026, so the same 9 or 12 month period is applied to them and labelled as such.

## Settings (environment variables)

| Variable | Default | Meaning |
|---|---|---|
| `DEMO_TODAY` | `2026-10-13` | The date the app treats as today (`YYYY-MM-DD`). The seed assumes the default. |
| `PORT` | `3000` | Server port. |
| `HOST` | `127.0.0.1` | Interface to listen on. |
| `ANTHROPIC_API_KEY` | not set | If set, reminders are drafted by the Anthropic API. If not, a fixed template is used. |
| `ANTHROPIC_MODEL` | `claude-haiku-5-5` | Model used for drafting. |
| `STORE_FILE` | `data/store.json` | Alternative store file (the seed script always writes the default). |

Examples:

```
# PowerShell
$env:ANTHROPIC_API_KEY = "sk-ant-..."; npm start
```

```
# macOS / Linux
ANTHROPIC_API_KEY=sk-ant-... npm start
```

If the API call fails, or the model's text breaks the tone rules (no discounts, no deadline or urgency language), the server falls back to the template and says so in the audit log.

## Layout

```
server/index.js          Express app: original order-by-order routes, error handling, static files
server/routes/           browse (summary, search, ledger, buyers), transactions, reminders, audit
server/transactions.js   Create invoice, record credit, match and unmatch (one audit event each)
server/query.js          Search, filter, sort and page for the lists
server/summary.js        The summary figures, computed from the data
server/csv.js            CSV generation for the audit export
server/store.js          Reads and writes data/store.json (temporary file, then replace)
server/seed.js           The deterministic transaction history (and the one-invoice seed the tests use)
server/audit.js          Appends audit events
server/rules/            deadline, matcher, consent, status, ledger, tone
server/agent/draft.js    Reminder drafting (Anthropic API or template)
server/agent/planner.js  The Recovery Agent: candidate actions, ranking, credit matching
server/agent/guardrails.js  Modes, limits and what may run without the merchant
server/agent/model.js    The checked use of the model: re-ranking, wording, answers
server/agent/ask.js      Questions: the data snapshot and the rules answers
server/agent/statement.js  Reads pasted bank lines (rules, plus the checked model reading)
server/agent/reconcile.js  Buyer by name, buyer history, invoice combinations, confidence
server/routes/agent.js   Agent routes
server/reminderDrafts.js Drafting shared by the Reminders view and the agent
server/view.js           Builds the invoice and credit views the UI renders
public/                  index.html, styles.css, app.js (router and shell) and one file per area
assets/                  Razorpay logo (header only, 24px high)
tests/rules.test.js      Vitest tests for every rule in DESIGN_GUIDE.md sections 12, 12a and 12b
```

## API

Original order-by-order routes (kept for the API and tests; the screens no longer use them):

| Method and path | Purpose |
|---|---|
| `GET /api/invoices` | All invoices with computed deadline, days left, rule, shortfall |
| `PATCH /api/invoices/:id` | `{ is_inr?, on_caution_list? }`, recalculates the deadline |
| `POST /api/invoices/:id/credits` | `{ amount }`, records a bank credit and runs the matcher |
| `POST /api/invoices/:id/reminder/draft` | Draft for the balance (only while partially paid) |
| `POST /api/invoices/:id/reminder/approve` | `409` without consent; otherwise records approval and queues |

Ledger and browsing:

| Method and path | Purpose |
|---|---|
| `GET /api/summary` | Summary figures, computed from the data |
| `GET /api/invoices/search` | Query: `q`, `status`, `buyer`, `currency`, `deadline` (`on_track`, `within_60`, `breached`, `caution_list`), `open`, `reminder` (`none`, `draft`, `queued`, `pending`), `sort` (`reference`, `buyer`, `amount`, `export_date`, `status`, `deadline`, `outstanding`), `dir`, `page`, `page_size` (default 15). Amounts sort by face value, whatever the currency. |
| `GET /api/invoices/:id` | One invoice |
| `POST /api/invoices` | New invoice: `buyer_id` or `new_buyer { name, country, email?, consent_recorded }`, `amount`, `currency`, `export_date`, `is_inr?`, `on_caution_list?` |
| `GET /api/credits` | The ledger. Query: `q`, `matched` (`all`, `matched`, `unmatched`), `currency`, `from`, `to`, `sort`, `dir`, `page`, `page_size` |
| `POST /api/credits` | Record a credit: `amount`, `currency`, `received_at`, `invoice_reference?`, `note?`, `confirm_duplicate?`. `422 currency_mismatch`, `409 possible_duplicate` |
| `POST /api/credits/:id/match` | `{ invoice_id }`, matches an unmatched credit by hand |
| `POST /api/credits/:id/unmatch` | Returns a matched credit to the unmatched queue |
| `GET /api/buyers`, `GET /api/buyers/:id` | Buyers with outstanding totals; one buyer with invoices and credits |
| `PATCH /api/buyers/:id/consent` | `{ consent_recorded, channel?, simulated? }`. Records the buyer's reply with when (`consent_at`) and how (`consent_channel`: `buyer_link`, `whatsapp`, `email`, `merchant_entry`).. The screens only call it from the demo box, with `simulated: true` |

Reminders and audit:

| Method and path | Purpose |
|---|---|
| `GET /api/reminders` | Reminders, newest first. Query: `status` (`draft`, `queued`) |
| `POST /api/reminders/draft` | `{ invoice_id }`, creates or replaces the draft for any invoice with a balance |
| `PATCH /api/reminders/:id` | `{ body }`, edits a draft. `422 tone_rules` for discounts, deadline or urgency language |
| `POST /api/reminders/:id/approve` | `409 consent_missing` without consent; otherwise queues the reminder |
| `GET /api/audit` | Events, newest first, each with `invoice_reference` and `buyer` worked out on read. Query: `type`, `invoice_id`, `from`, `to` |
| `GET /api/audit.csv` | The same filtered log as a CSV download (`time`, `type`, `invoice`, `message`) |
| `GET /api/meta` | Demo date, exporter and drafting mode |
| `POST /api/reset` | Restores the seeded history exactly, and clears the agent's runs (development only) |

The Recovery Agent:

| Method and path | Purpose |
|---|---|
| `GET /api/agent` | Settings, how it reasons (rules or model), the latest run, pending proposals, recent decisions, performance |
| `POST /api/agent/run` | Scans the portfolio and proposes actions. `409 agent_paused` when paused. In auto mode it also applies the safe ones |
| `POST /api/agent/actions/:id/apply` | Carries out one proposal. `409 not_pending`, or a rule error if it has gone stale |
| `POST /api/agent/actions/:id/dismiss` | `{ note? }`, dismisses a proposal |
| `PATCH /api/agent/settings` | `{ enabled?, mode?, min_match_confidence?, max_actions_per_run?, min_days_since_export? }` |
| `POST /api/agent/ask` | `{ question }`, answers from the data. Read-only |
| `POST /api/agent/reconcile` | `{ text }` (up to 50 lines of 300 characters) reads a bank statement and proposes where each credit belongs; or `{ lines }` to match lines already read, again. Records nothing. `409 agent_paused` |
| `POST /api/agent/reconcile/apply` | `{ line, allocations: [{ invoice_id, amount }], confirm?, confirm_duplicate? }` records one line, as one credit or a split. An empty `allocations` records it unmatched. `409 proposal_changed` if it no longer agrees with the agent's own working, `409 confirmation_needed`, `409 possible_duplicate`, `422 unreadable_line` |

## Not included

Local-rail payment links, live bank or accounting integrations, real WhatsApp or email sending, login and multi-user accounts. These are later steps for the pitch.
