# Export Cash Recovery: Design Guide

Use this guide when building the prototype in Claude Code. The prototype is a working application: a small Node server with real business rules, a JSON file for storage, and a web front end served by the same server. It matches the look and feel of the Razorpay agent landing page in the reference screenshot. Values marked **(estimated)** were read from the screenshot, not from source code. Sample exact hex codes with a colour picker before you finalise them.

## 1. Brand and logo

- **Logo:** `assets/razorpay-logo-white.png`, the white variant with the blue mark. Use it only on the dark hero and nav, at 24px height in the header. Do not recolour or stretch it.

## 2. Colours

| Token | Value | Use |
|---|---|---|
| `--bg-hero` | `#000000` | Top navigation and hero background |
| `--bg-glow` | `#0b1f5c` to `#2b5ce6` radial gradient, centred low | Blue glow behind the hero and cards **(estimated)** |
| `--bg-app` | `#0a0f1e` | Application background (dark mode) |
| `--surface` | `#0f1a33` at 85% opacity | Cards |
| `--border` | `#ffffff` at 12% opacity, 1px | Card and button borders |
| `--text-primary` | `#ffffff` | Headings and body |
| `--text-muted` | `#a7b0c0` | Secondary text, timestamps |
| `--cta` | `#3b5bfd` **(estimated)** | Primary buttons, links |
| `--cta-hover` | `#2d49e6` | Button hover |
| `--chip-bg` | `#d9dde5` | Name chips (light grey on dark) |
| `--chip-text` | `#0a0f1e` | Text inside chips |
| `--ok` | `#22c55e` | Realised, approved, matched |
| `--warn` | `#f59e0b` | Partial payment, deadline within 60 days, Caution List |
| `--bad` | `#ef4444` | Deadline passed, consent missing, blocked |
| `--grid-line` | `#5b7bff` at 25% opacity | Faint background grid lines |

Contrast: `--text-muted` on `--bg-app` passes WCAG AA for normal text. Do not put muted text on the blue glow without a solid backing card.

## 3. Typography

**Font family:** Inter (free on Google Fonts). The reference uses a geometric grotesque very close to Inter. Fallback: `system-ui, -apple-system, Segoe UI, Roboto, sans-serif`.

**Minimum sizes (strict):** No text smaller than 13px anywhere. Body text is 16px.

| Role | Size | Weight | Line height | Notes |
|---|---|---|---|---|
| Hero headline | 56px desktop / 36px mobile | 500 | 1.05 | Second line in muted grey, as in the reference |
| Section heading (H2) | 32px / 26px mobile | 600 | 1.2 | |
| Card title (H3) | 20px | 600 | 1.3 | |
| Body | 16px | 400 | 1.6 | |
| Secondary text | 14px | 400 | 1.5 | Descriptions and metadata |
| Chip / tag label | 13px | 600 | 1.2 | UPPERCASE, letter-spacing 0.04em |
| Button label | 16px | 600 | 1 | |
| KPI numbers | 32px | 600 | 1.1 | Tabular numerals (`font-variant-numeric: tabular-nums`) |

Write the hero headline as two lines. Set the first line in white and the second in muted grey, as the reference does with "protect your revenue".

## 4. Layout and spacing

- **Spacing scale:** 4, 8, 12, 16, 24, 32, 48, 64px. Use only these values.
- **Container:** max 1200px wide, centred. Side padding 24px on desktop, 16px on mobile.
- **Nav bar:** 76px tall. Logo on the left, product name next to it. The demo has no login, so the nav shows only the logo and product name.
- **Hero:** Full width, about 620px tall on desktop on the first screen. Compact (about 200px) on the working screens so the data sits above the fold. Headline top-left.
- **Application grid:** 12 columns on desktop, 1 column below 900px. Gutter 16px.
- **Radius:** Cards 12px, buttons 8px, chips 4px.

## 5. Components

**Primary button.** Background `--cta`, white 16px text, 14px vertical and 24px horizontal padding, 8px radius, no border. Hover: `--cta-hover`. Focus: 2px white outline with 2px offset. Disabled: 45% opacity, not clickable.

**Secondary (ghost) button.** Transparent background, 1px white border at 40%, white text, same padding.

**Agent card.** The signature component from the reference.
- Dark surface with a faint 1px border, 12px radius, 24px padding.
- Top-left: a chip with the agent name in uppercase, for example `EXPORT CASH RECOVERY AGENT`.
- Below the chip: the outcome in 18px white, for example "Invoice EX-1042: USD 11,760 received, USD 240 short".
- Small status dot in the corner: green for realised, amber for partial payment, red for deadline breached.
- Show one number per card where possible, such as the shortfall or days left.

**Chip / tag.** Light grey background, dark uppercase text, 13px, 4px radius, 4px 8px padding. Variants: `ok`, `warn`, `bad` use the status colours at 18% opacity with coloured text.

**KPI tile.** Surface card, label in 14px muted, value in 32px white with tabular numerals, optional secondary line in 14px muted.

**Deadline countdown.** Shows days remaining in 32px tabular numerals. Use `--warn` when 60 days or fewer remain and `--bad` when the deadline has passed. The rule used (for example "9 months from export") is shown in the Deadline tile on an invoice, not in the countdown box.

**Stepper.** Five steps across the top. Current step has a blue border and a white label. Completed steps are clickable. Steps not yet reached are disabled at 40% opacity.

**Toggle (checkbox row).** 18px checkbox with `--cta` accent, 16px label, 8px gap. Changing a toggle changes server state, not only the view.

**Input and select.** Surface background, 1px border at 20% white, 16px text, 8px radius, 12px padding. Focus: border becomes `--cta`.

**Table.** Header row 13px uppercase muted. Body rows 16px. Row separators at 8% white. Status always shown as a chip, never colour alone.

**Message preview.** Surface panel, 16px text, preserved line breaks, 8px radius. The text is what the buyer would receive, nothing added.

## 6. Background and decoration

- Use the hero's blue radial glow as the main accent. Keep it behind the headline, not behind tables or the audit log.
- Faint grid lines (1px, `--grid-line`) can sit behind the hero. Keep them below 25% opacity.
- No photos of people in the application screens, with two exceptions: the photo under the Buyer and Invoice card on a reminder (`assets/reminder-meeting.jpg`) and the image in the Recovery Agent box (`assets/agent-overview.jpg`).
- Riri, the agent, has an illustrated avatar (`assets/riri.webp`). It is cropped to a rounded square and shown beside her name: 28px in the Riri tab, 56px in the top of her box and 40px on the Ask Riri and bank statement boxes. It is decorative (empty alt text) because her name is always next to it.

## 7. Motion

- Hover transitions 150ms ease. Panel and step changes 300ms ease.
- Cards fade in with a 40ms stagger on load. Nothing bounces or spins.
- Respect `prefers-reduced-motion` and turn all animation off.

## 8. Accessibility

- Every interactive element has a visible focus ring.
- Never convey status by colour alone. Pair colour with a text label.
- Minimum touch target 44 × 44px on mobile.
- Form controls have labels. Buttons describe their action ("Approve and queue", not "OK").
- Check the glow background against text with a contrast checker, and put text on solid surfaces.

## 9. Responsive rules

- **Below 900px:** The stepper wraps to two rows. The application grid becomes one column. The hero headline drops to 36px.
- **Below 600px:** Buttons go full width. KPI tiles go to two per row.

## 10. Working demo path

The prototype is a working application. Each screen reads from and writes to the server. Nothing is a static mock-up. The five steps are the same as before, but every number and status comes from the rules on the server and is stored in a JSON file, so it survives a page reload.

1. **Invoices (start state).** Lists the seeded export invoice EX-1042, status Unpaid, with the deadline countdown and the rule used. Toggles for "Invoice is in INR" and "Exporter is on the Caution List" update the stored invoice and recalculate the deadline.
2. **Simulate payment.** A button posts a bank credit of 98% of the invoice. The matcher links it to EX-1042 by invoice reference, records the shortfall, and sets status to Partially paid.
3. **Reminder.** The agent drafts a reminder for the balance, stored as a draft. A consent toggle reads and writes the buyer's consent record. Without consent, the server refuses to queue the reminder. "Approve and queue" records the merchant's approval and sets status to Queued (not sent). The app does not send anything.
4. **Simulate balance received.** Posts a second bank credit for the shortfall. The matcher marks the invoice Realised and stops the countdown.
5. **Audit log.** Lists every stored event with its timestamp, newest first, in a 14px table with tabular numerals.

**Excluded from the working demo:** local-rail payment links, live bank or accounting integrations, real WhatsApp or email sending, login, and multi-user accounts. Mention them as later steps in the pitch.

## 11. Architecture

**Stack**

- **Server:** Node.js 20 or later with Express. One process serves the API and the front end.
- **Front end:** plain HTML, CSS and JavaScript in `public/`. No build step, so nothing to compile.
- **Storage:** a single JSON file, `data/store.json`, created by the seed script. The server reads it at startup and writes it after every change. Writes go to a temporary file first, then replace the real file, so a crash can't leave it half-written.
- **Tests:** Vitest for the rules.
- **AI drafting (optional):** the Anthropic API, called from the server only. Read the key from `ANTHROPIC_API_KEY` and the model from `ANTHROPIC_MODEL`, defaulting to `claude-haiku-5-5`. If no key is set, the server uses a fixed template. The app works either way.

**Folder layout**

```
export-cash-recovery/
  DESIGN_GUIDE.md
  README.md                 Exact commands to run the app
  package.json
  assets/razorpay-logo-white.png
  server/
    index.js                Express app: API routes and static files
    store.js                Reads and writes data/store.json safely
    seed.js                 Creates the demo invoice, buyer and empty collections
    audit.js                Appends audit events
    rules/
      deadline.js           Realisation deadline and days left
      matcher.js            Matches bank credits to invoices, computes shortfall
      consent.js            Consent gate for reminders
      status.js             Allowed status transitions
    agent/
      draft.js              Drafts the reminder, via the API or the template
  public/
    index.html
    app.js                  Renders the five screens and calls the API
    styles.css              Tokens from section 2 as CSS variables
  data/
    store.json              Created by the seed script. Not committed.
  tests/
    rules.test.js
```

**Data shape (`data/store.json`)**

```
{
  "invoices":     [ { id, reference, buyer_id, amount, currency, export_date, is_inr, on_caution_list, status, created_at } ],
  "buyers":       [ { id, name, country, email, consent_recorded } ],
  "bank_credits": [ { id, invoice_reference, amount, currency, received_at, note } ],
  "reminders":    [ { id, invoice_id, body, amount_outstanding, status, approved_at, created_at } ],
  "audit_events": [ { id, invoice_id, type, message, created_at } ]
}
```

- `status` on invoices is one of `unpaid`, `partially_paid`, `queued`, `realised`.
- `status` on reminders is one of `draft`, `queued`.
- `audit_events` only ever gets new entries. Nothing edits or deletes them.

**API (JSON)**

- `GET /api/invoices`: list invoices with computed deadline, days left, rule used and shortfall.
- `PATCH /api/invoices/:id`: body `{ is_inr?, on_caution_list? }`. Recalculates the deadline and writes an audit event.
- `POST /api/invoices/:id/credits`: body `{ amount }`. Records a bank credit, runs the matcher, updates status, writes audit events.
- `PATCH /api/buyers/:id/consent`: body `{ consent_recorded }`. Writes an audit event.
- `POST /api/invoices/:id/reminder/draft`: creates or replaces the draft reminder for the outstanding balance.
- `POST /api/invoices/:id/reminder/approve`: refuses with `409` if the buyer has no consent. Otherwise records approval, sets status `queued`, and writes an audit event.
- `GET /api/audit`: all audit events, newest first.
- `POST /api/reset`: re-runs the seed. Development only.
- Later additions (ledger, browsing, reminders, audit export) are listed in `README.md`.

## 12. Business rules

These rules live in `server/rules/` and are tested in `tests/rules.test.js`. The front end never decides them.

- **Deadline.** The realisation period is 9 months from the export date. For invoices that are INR-invoiced or INR-settled, it is 12 months. The rule applies to exports made on or after 1 October 2026, under FEMA 23(R)/(1)/2026-RB. Exporters on the Caution List as of 30 September 2026 are not given a computed date. The server returns the flag "existing RBI order applies" instead.
- **Days left.** Days from today to the deadline. Today is 13 October 2026 in the demo, set by a `DEMO_TODAY` environment variable. Negative values mean the deadline has passed and the status is breached.
- **Matching.** A credit is matched to an invoice by reference. A matched total below the invoice amount is Partially paid. A matched total equal to or above it is Realised.
- **Shortfall.** Invoice amount minus matched total. The UI labels it "likely intermediary (SWIFT) charges". It does not claim the cause.
- **Consent gate.** A reminder can be approved only if the buyer's `consent_recorded` is true. The server enforces this, and the UI reflects it.
- **Merchant approval.** Nothing is queued without an approval event. Nothing is sent. Queued means stored and waiting.
- **No discounts in drafts.** The reminder template never includes a discount, deadline pressure or urgency language.
- **Status transitions.** `unpaid → partially_paid → queued → realised`, plus `unpaid → realised` if a full credit arrives directly. Any other transition is rejected.
- **Audit.** Every change writes one event. Events are never edited or deleted.

## 12a. Extensions for the transaction ledger

Section 12 stays as written for the five-step guided demo, which keeps its strict order (`POST /api/invoices/:id/credits`, `POST /api/invoices/:id/reminder/draft` and `.../approve`). The ledger, the Reminders view and the other new screens work on every invoice, in any order, so they follow these additional rules:

- **Status is derived.** `realised` when the matched total is equal to or above the invoice amount; otherwise `queued` if a reminder for the invoice has been approved; otherwise `partially_paid` if anything is matched; otherwise `unpaid`. This allows `partially_paid → realised`, `unpaid → queued`, and the reversals caused by unmatching a credit. Every change still writes one audit event.
- **Credits.** A credit in a different currency from the invoice it names is rejected. A credit that would take the matched total above the invoice amount is accepted and flagged as an over-payment for review. A credit with the same reference, amount and currency as one already saved is a possible duplicate and needs confirmation. A credit with no reference, or a reference that matches no invoice, goes to the unmatched queue.
- **Reminders.** Any invoice that is not realised and has a balance can have a reminder drafted, edited and approved. The consent gate and the tone rules (no discounts, deadline pressure or urgency language) apply to the merchant's edits as well as to drafts. Approval still only queues the reminder.
- **Exports before 1 October 2026.** The guide gives no rule for them, and the seeded history starts in October 2025, so the same 9 or 12 month period is applied and labelled "export before 1 October 2026". `computeDeadline` itself is unchanged.
- **Amounts in different currencies are never added together.** Totals are listed one currency per line.

## 12b. Riri, the Recovery Agent

The agent is a planner on top of the rules above. It proposes actions and the merchant decides. These rules are enforced on the server and tested:

- **Rules propose, the model advises.** The planner (`server/agent/planner.js`) decides which actions are possible and ranks them. With an API key the model may re-order them and re-word each reason. It cannot add, remove or change an action, cannot cite an invoice it was not given, and is not sent buyer emails or reminder text. Any reply that fails a check is discarded and the rules result is used.
- **Never, in any mode:** send anything, approve or queue a reminder, contact a buyer without consent on record, act while the agent is paused.
- **Modes.** Review first: nothing is applied until the merchant applies it. Auto within guardrails: only a credit match at or above the merchant's confidence threshold, and a reminder draft, are applied automatically.
- **Every run, application, dismissal and settings change** writes an audit event that says what was done and why.
- **Proposals can go stale.** Applying one re-checks the data and is refused if the facts have changed.

### Matching a bank statement

Pasted statement lines are read and matched on the Agent screen. These rules are enforced on the server and tested:

- **Reading is separate from deciding.** Rules read each line (amount and currency, date, invoice references including typing slips such as a letter O for a zero). With a key, the model may also read the lines, and is sent only the pasted text. A value the model returns is used only if it can be seen in the line; otherwise it is dropped and the rules value is used. The model never chooses an invoice or an amount to record.
- **Which invoices.** The buyer comes from a named invoice or from a name in the narration (one typing slip allowed in a word of six letters or more; two buyers fitting equally means no buyer). Only that buyer's open invoices, in the line's currency, exported on or before the credit date, are considered. One credit is proposed as a split when a group of up to six of them has an outstanding total that the credit equals, or falls short of by no more than the tolerance (3%, or 1.5 times the buyer's usual short payment, at most 6%). A credit is never more than the invoices it is split across owe. Two buyers' invoices are never combined.
- **How the split is made.** Invoices are paid in full oldest first; the newest is left short. Recording it creates one credit per invoice, sharing a `split_group`, whose amounts add up to the amount the bank paid. Every other rule (status, over-payment, unmatching) applies to each part as to any credit.
- **Confidence** is the sum of independent evidence: the invoice is named, or the buyer is named, plus an exact amount or one within the tolerance, plus the buyer's history, minus anything ambiguous. Below the merchant's threshold, a line with no date, one that may already be in the ledger, and an over-payment each need an explicit confirmation to record.
- **Nothing is recorded by reading.** Each line is recorded by the merchant. The server works the proposal out again at that moment and refuses a request that no longer agrees with it.
- **Audit.** Reading a statement writes one event. Recording a line writes `credit_matched` (one invoice) or `credit_split` (one event per invoice), each with the reasons.

## 13. Acceptance criteria

The prototype is finished when all of these pass:

1. `npm install`, `npm run seed` and `npm start` run with no errors. The app opens at `http://localhost:3000`.
2. The five steps can be completed in order using only the UI, and the status changes match section 12. The demo invoice EX-1042 is found in the Invoices list; its payments are recorded under Transactions, and Reminders lists it under "Buyers to remind".
3. An INR invoice gets a 12 month deadline instead of 9 months, and an invoice on the Caution List shows the existing-order flag instead of a date. Both flags are set on the New invoice form; the invoice page does not edit them (the `PATCH /api/invoices/:id` route still does).
4. With no consent on record, the approve button is disabled, and calling the approve endpoint directly returns `409`. Consent shows as a record ("Consent recorded on [date] via [channel]"), not a tick box; the only way to change it in the UI is the labelled demo action "Simulate buyer response".
5. Refreshing the page keeps the state. Restarting the server keeps it too, because it's stored in `data/store.json`. This includes new invoices, credits, matches, reminders and edits.
6. The audit log shows one event per action in order, with timestamps.
7. `npm test` passes. Each rule in sections 12, 12a and 12b has at least one test.
8. The logo loads from `assets/` in the header, at 24px height.
9. No text smaller than 13px. Check with the browser inspector on at least three screens.
10. The layout works at 390px width without horizontal scrolling.
11. With no `ANTHROPIC_API_KEY`, drafting uses the template and the app still works. With a key set, drafting calls the API and the draft is stored.
12. Nothing is sent to any real buyer, email, WhatsApp or bank.
13. **Seed.** `npm run seed` creates the same data every time: 8 buyers (US, UK, Germany, Singapore, UAE; 2 without consent), 45 invoices exported October 2025 to October 2026 (USD, EUR, GBP and 10 in INR), mostly realised, with partial payments, unpaid invoices, 2 breached, 1 on the Caution List, an over-payment, 3 unmatched credits and a backfilled audit log. EX-1042 is unchanged: unpaid, USD 12,000, Northwind Trading LLC.
14. **New invoice.** The form creates an invoice that appears in the list. It rejects an amount of zero or less, an export date in the future, and an INR invoice not in INR. A new buyer can be added with a consent toggle.
15. **Bank credits.** A credit with a reference that matches an open invoice is matched and the status changes. A credit with no reference goes to the unmatched queue. A currency mismatch is rejected with a clear message. An over-payment is accepted and flagged. A duplicate needs confirmation.
16. **Matching by hand.** An unmatched credit can be matched to an open invoice, and a matched credit can be unmatched. Both recalculate the status and write an audit event.
17. **Browsing.** The invoice list searches by reference or buyer, filters by status, buyer, currency and deadline state, sorts by column and pages 15 at a time. The Transactions ledger searches and filters by date, with a clear Unmatched badge. A buyer page shows invoices, credits, total outstanding and consent. The summary figures are computed from the data.
18. **Reminders.** Any invoice with a balance can have a reminder drafted, edited in a text area and approved. Edits that break the tone rules are refused. The Reminders view lists queued reminders as "Queued, not sent".
19. **Audit log.** It shows the buyer and invoice for each event with an Open link, has count chips that filter by event type, filters by event type, invoice and date range, and exports the filtered log as a CSV with the columns time, type, invoice and message.
20. **Reset.** "Reset demo data" in the profile card asks for confirmation and restores the seed exactly, including the audit events.
21. **Agent plan.** "Run the agent" proposes ranked actions, each with its reasons, evidence and guardrail checks. With no API key it says "Reasoning by rules" and works fully.
22. **Agent control.** In review-first mode nothing changes until the merchant applies a proposal. Applying a credit match updates the invoice; applying a reminder proposal stores a draft only. Dismissing records a note. A paused agent will not run.
23. **Auto mode.** It applies only credit matches at or above the confidence threshold and reminder drafts. It never approves a reminder, and leaves escalations, consent gaps and low-confidence matches for the merchant.
24. **Model checks.** With a key, a reply that adds or changes an action, cites an unknown invoice or is not valid JSON is discarded and the rules result is used. (Tested against a mocked API; not yet run against a live key.)
25. **Ask the agent.** Plain-language questions are answered from the data, name the invoices they mention as links, and change nothing.
26. **Agent log.** Every run, applied or dismissed proposal and settings change appears in the audit log with its reason, and the performance figures are computed from them.
27. **Reading statement lines.** Pasted lines are read into amount, currency, date and invoice references, including `EX1O42`, `INV 1042` and several references in one line. A line with no amount is marked "Could not read" and cannot be recorded. With no key the screen says "Read by rules" and works fully.
28. **Split matching.** One credit that equals (or is a little short of) several of one buyer's open invoices is proposed as a split with each part shown. Recording it creates credits that add up to the bank amount, updates every invoice, and writes an audit event for each. A part can be unmatched on its own.
29. **Statement guardrails.** Nothing is recorded by reading. A line below the confidence threshold, with no date, or that may already be in the ledger asks for confirmation. A stale or altered request is refused. Currency, an open invoice, and the one-buyer rule always hold.
30. **Model checks on statements.** With a key, a value that cannot be seen in the line, or a reply that is not valid JSON, is discarded and the rules reading is used. The model is never sent invoices or buyers. (Tested against a mocked API; not yet run against a live key.)

## 14. Starting prompt for Claude Code

Paste this into Claude Code at the start of the project:

> Build a working prototype called "Export Cash Recovery". It is a student submission to Razorpay. Follow DESIGN_GUIDE.md exactly for visuals (sections 1 to 9), business rules (section 12), architecture and data shape (section 11), and acceptance criteria (section 13). Use Node 20 with Express for the server, and plain HTML, CSS and JavaScript in `public/` for the front end, with no build step. Store all data in `data/store.json` through `server/store.js`, writing to a temporary file and then replacing the real one. Do not add a database. Put the Razorpay logo at `assets/razorpay-logo-white.png` in the header only, at 24px height. Do not recolour or stretch it.
>
> Build the five-step demo path in section 10 as a real application: invoices, simulated payment, reminder with consent gate and merchant approval, realised, and audit log. Every status, deadline and shortfall must come from the server rules in section 12, stored in `data/store.json`, and survive a page reload and a server restart. Seed one invoice EX-1042 for Northwind Trading LLC (US), USD 12,000, exported 2 October 2026, with a buyer who has consent recorded. Set today to 13 October 2026 through `DEMO_TODAY`.
>
> The app must never send anything to a buyer, and the approval endpoint must refuse when consent is missing. Drafting uses the Anthropic API only if `ANTHROPIC_API_KEY` is set, otherwise a fixed template. Use `ANTHROPIC_MODEL` with a default of `claude-haiku-5-5`. Do not add login, real integrations or multi-user features.
>
> Write Vitest tests in `tests/rules.test.js` for every rule in section 12.
>
> Write a README with the exact commands: `npm install`, `npm run seed`, `npm start`, `npm test`.
>
> Before you finish, run those four commands, open the app in a browser, walk through all five steps, check the acceptance criteria in section 13 one by one, and report which pass, which fail, and anything you could not verify.

## 15. Before you finalise

- Sample `--cta` and the background gradient colours with a colour picker and replace the estimated values.
- Confirm the realisation clock start date and the Caution List rule against the RBI circular text, since the deadline logic depends on them.
- Decide whether the demo runs on your laptop or on a host. The app needs a Node process running. If you host it, keep `data/store.json` out of the repository.
