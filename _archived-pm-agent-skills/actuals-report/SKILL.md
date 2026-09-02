---
name: actuals-report
description: Turn a Basecamp (or similar) time-tracking export into an actuals hours report — every logged hour bucketed per ADO ticket and split into Dev/QA/PM columns, actuals only. Use when asked to build an actuals report, turn a time log / time-tracking export into "actuals", tally logged hours per feature, or reconcile hours against tickets.
---

# Actuals Report

Turn a raw time-tracking export (Basecamp "Time tracking report", or any log with
one `date · person · hours · description` entry per line) into an **actuals**
table like the delivery team keeps per SOW: one row per ADO ticket, hours split
into **Actual Dev / Actual QA / Actual PM / Actual Total** columns, plus a General
Hours row and a totals row.

**Actuals only.** This report never contains estimates. The estimate columns from
the historical spreadsheet are out of scope here — you are not given the estimates
and don't need them at this stage. Produce only what was actually logged.

## Pipeline

- **Inputs:** a time-tracking export (PDF/CSV/pasted text) and a role for **each**
  person in it (asked from the caller — see below).
- **Outputs:** an `.xlsx` (and `.csv`) actuals table, hours per ADO ticket × role,
  reconciled to the export's total.
- **Next step:** the numbers feed a delta-vs-estimate analysis elsewhere; this
  skill stops at actuals. `pm-agent` can pull ticket titles from ADO if the caller
  wants the Feature column filled in.

## The two things every hour needs

Each logged entry is bucketed on two axes:

1. **Which row** — parsed from the description. A 3–4 digit number (`1509`,
   `1515 uat`, `1515, standup`) → that ADO ticket. No ticket number → General
   Hours, unless it's future-SOW scoping work (see estimation handling).
2. **Which column** — the person's **role** (Dev / QA / PM). This is the input you
   must get from the caller.

## NEVER assume roles — ask the caller

**Do not infer who is Dev, QA, or PM from names or from what they logged**, however
obvious it looks (someone logging only "PM", someone logging only ticket numbers).
The caller assigns roles; you ask. Getting this wrong silently misattributes every
hour that person logged into the wrong column.

- List the distinct people in the export and **ask the caller for each one's role**
  (AskUserQuestion, one sub-question per person, options Dev / QA / PM).
- One role per person — their whole logged total goes in that column. If the caller
  says a person genuinely splits across roles, ask how they want it handled rather
  than guessing a split.
- The helper script **refuses to run** if any person is unmapped — that's
  intentional; resolve it by asking, not by filling in a guess.

**Red flags (stop and ask instead):** "Matt only logs PM so he's obviously the PM";
"Sean logs ticket numbers so he must be Dev"; "I'll just infer roles from the
descriptions." All of these are the assumption this skill forbids.

## How to run

1. **Get the export into text.** For a PDF, `pdftotext -layout export.pdf log.txt`
   preserves the columns. Pasted text or a CSV works too — one entry per line as
   `date  person  hours  description`.
2. **List the people** and **ask the caller each person's role** (see above). Also
   confirm two things while you're asking:
   - **Estimation / other-SOW work** — lines like "sow# 5 review and estimation",
     "eval and estimations review" are usually scoping for a *future* SOW, not
     delivery of the tickets in this report. Ask whether to keep them as their own
     **SOW Review & Estimation** row (`--estimation separate`, default), **exclude**
     them (`exclude`), or **fold into General Hours** (`general`).
   - **General Hours label** — e.g. "General Hours SOW #3".
3. **Run the helper** to do the aggregation deterministically (never hand-total the
   hours):
   ```bash
   python3 build_actuals.py --log log.txt \
     --roles '{"Elena Dotsenko":"QA","Matt McClain":"PM","Sean Dillon":"Dev"}' \
     --estimation separate --general-label "General Hours" \
     --out actuals.xlsx
   ```
   It writes `actuals.csv` (always) and `actuals.xlsx` (if `openpyxl` is installed —
   `pip install openpyxl`), and prints a reconciliation line. **Confirm the reported
   total equals the export's stated total** before handing it over.
4. Optionally pass `--features '{"1509":"Auto Membership Transfer", ...}'` to fill
   the Feature column (ask `pm-agent` to fetch titles from ADO if wanted).

## Handling notes

- **Multi-activity entries** — an entry that names a ticket *and* something general
  (`"1515, standup"`, 7.5 h) is attributed wholly to the ticket; the incidental
  standup mention isn't split out. Flag any such line to the caller so they know.
- **Non-ticket delivery work** (documentation, video recording, post-prod testing
  without a ticket) rolls into General Hours, in the logging person's role column.
- **Reconciliation is mandatory.** Ticket hours + general + estimation (+ anything
  excluded) must equal the export total. The script warns on a mismatch; don't ship
  a report that doesn't reconcile.

## Output format

Columns, in order: **ADO · Feature · Actual Dev · Actual QA · Actual PM · Actual
Total**. Ticket rows sorted by ADO ascending, then (if used) a SOW Review &
Estimation row, a General Hours row, and a bold TOTAL row. No estimate columns.

In the `.xlsx`, **Excel owns the arithmetic**: each row's Actual Total is a
`=SUM(Dev:PM)` formula and the TOTAL row is `=SUM()` over each column's cells, so
the totals recalculate if anyone edits a cell. (The `.csv` fallback can't hold
formulas, so it carries the computed values instead.)

## Review checkpoint

Present the report as a draft to be verified before it's used or shared:

1. Show the table (or the reconciliation summary) and the **role mapping you used**
   — the whole result hinges on who you were told is Dev/QA/PM.
2. Explicitly confirm the reported total matches the export's stated total, and
   surface any judgment calls (multi-activity entries, estimation handling).
3. Ask the human whether the role mapping or estimation handling should change,
   and re-run if so.
4. Only treat the `.xlsx` as final once the human confirms.

## Rules

- **Actuals only** — never add, infer, or back-fill estimate columns.
- **Never assume roles** — the caller assigns Dev/QA/PM per person; you ask.
- **Never hand-total** — the aggregation runs through `build_actuals.py` so the
  math is deterministic and reconciles to the source total.
- Confirm estimation-handling and the General Hours label with the caller rather
  than deciding silently — the estimation choice can move 10%+ of the hours.
