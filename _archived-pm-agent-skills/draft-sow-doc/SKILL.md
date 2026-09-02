---
name: draft-sow-doc
description: Compile a client-facing "Estimates and Documentation" attachment for a SOW from an already-created set of Azure DevOps tickets, matching the Portal SOW #4 template. Use when a PM asks to produce full requirements documentation for a new SOW attachment, or references "the SOW 4 process" for a new set of tickets. Human drives section-by-section; Claude drafts and asks when it can't infer.
metadata:
  type: skill
---

# Draft SOW Estimate Documentation

Turns an already-created set of Azure DevOps tickets into the client-facing
"Features and Enhancements — Estimates and Documentation" PDF attachment that
ships alongside a signed SOW, matching the structure of the Portal SOW #4 doc.

**This is a collaboration, not an autonomous write.** The human (PM/dev) has
context this skill cannot infer on its own — what was actually said to the
client, which assumptions are already confirmed vs. still open, and how
"Business Intent" should be framed for a non-technical reader. Draft one
ticket's section at a time (or a small batch), show it, and let the human
correct before moving on. Never generate the whole document unattended and
present it as final.

## Pipeline

- **Inputs:** an already-created set of ADO tickets (epic/Feature id or explicit
  ids), fetched via `pm-agent`; often the tickets produced upstream by
  [[feature-spec]] → [[draft-ticket]] → [[estimate-ticket]].
- **Outputs:** the client-facing "Estimates and Documentation" attachment
  (Markdown, optionally rendered to docx/PDF).
- **Next step:** this is the *pre-work scoping* deliverable; [[status-update]] is
  its *delivery* counterpart once the same tickets ship.

## When to use

The user says something like "let's do the SOW doc for X the way we did SOW 4"
or hands off a reference PDF and a target ticket set (an epic, a Feature, or
an explicit list of ADO ticket numbers).

## Inputs needed before starting

- The set of ADO tickets to document (epic/Feature id, or explicit ticket
  numbers). If not given, ask.
- The reference doc for this program, if one exists (comment on the parent
  scoping ticket, a design doc, prior SOW docs) — reread it before drafting;
  don't rely on ticket text alone if a richer source exists.
- Confirm the output format: Markdown source is the working format; ask
  whether the human wants it also rendered as `.docx`/PDF at the end (the
  `docx`/`pdf` skills in this environment can do that conversion once the
  Markdown is final — don't reach for them before content is approved).

## The section template

This is derived from reading all 15 ticket writeups (all 70 pages) of the
SOW 4 reference doc, not a sample. The source doc itself is **inconsistent**
in several places — heading wording drifts ticket to ticket, "Dependencies"
sometimes means "what this needs" and sometimes "what needs this," and the
Requirements section nests at different depths depending on who wrote that
particular entry. Reproducing that inconsistency in a new document would
just carry someone else's inconsistency forward. Where the source varies,
this template **picks one fixed convention** instead — noted explicitly
below wherever that happened.

Each ticket becomes one document section, title `{ADO#} - {Title}` exactly
matching the ADO ticket. Not every section applies to every ticket — match
weight to the ticket's actual complexity, the same way SOW 4 gave a one-page
treatment to a simple UI tweak and an eight-page treatment to a rules-heavy
feature with pending UI. Sections marked **(always, even if empty)** must
appear with an explicit "None." rather than being silently omitted — an
absent section reads as "forgotten," an explicit "None." reads as
"considered, nothing here."

1. **Opening summary** (always) — 1-3 short paragraphs in plain English:
   what this does and why. If the ticket has a `**Scope decision confirmed
   with the client:**` line or similar, surface it here.
2. **Source quote** (only when an actual client quote exists) — heading
   `Per client specification:`. **Never fabricate this quote.** If the ADO
   ticket doesn't contain one and the human doesn't supply one, omit this
   section entirely rather than paraphrase something as if it were a direct
   quote.
3. **Business Intent** (always) — bullets on business rationale (why this
   matters to the operator/client), not technical rationale. If the ADO
   ticket's "Context" section only argues the technical case, ask the human
   to supply the business framing rather than inventing it. **Stay neutral
   about the system being replaced** — state what the new system does
   (e.g. "computes availability from Portal's own data") without editorializing
   about the old one's shortcomings (e.g. don't write "so a slow/unreliable
   Wix API can no longer degrade the experience" — that's a jab, not a fact
   the client asked to see in writing). This is a professional client
   document, not an internal gripe session.
4. **Where this data will live** (dashboard/reporting features only) — a
   short description of UI placement. Only include if the human can
   describe placement, or the ADO ticket already specifies it.
5. **Requirements** (always) — **fixed nesting, applied uniformly, unlike
   the source:**
   - `## Requirements` (H2)
   - `### <Named Facet>` (H3) for each distinct rule-area the ticket covers
     — always H3, never an italic-only or bold-only top-level grouping.
   - Within an H3, if the facet decomposes into several named items (e.g. a
     glossary of metrics, a set of trigger types), use a **bold inline
     label** per item followed by its own bullet list — that is the one
     permitted second tier, not italics, not a further heading level.
   - Rewrite ADO Acceptance Criteria/Constraints as declarative
     "System must..."/"UI must..." bullets, not Given/When/Then — a register
     shift for a client audience, not new content.
   - If real business-rule ambiguity exists that needs client sign-off
     before dev starts (e.g. revenue classification rules, edge-case
     handling), give it its own `### Decisions Requiring Confirmation`
     subsection under Requirements rather than a separate top-level section
     — keeps confirmed-vs-open in one place instead of scattered.
6. **Expected Behavior** (usually) — bullets stating resulting behavior as
   plain outcomes. Derive from the Acceptance Criteria's "then" clauses.
7. **QA / Scope Notes** (always) — this exact heading, the one string the
   source used with zero variation. "QA must verify:"/"QA must test:"
   bullet lists, subdivided by named area for complex tickets.
8. **Dependencies** (always, even if empty) — what this ticket needs from
   other tickets/systems to work. Bold ticket number + title, one-line
   relationship note. Write "None." if there are none — don't omit the
   heading.
9. **Downstream Dependencies** (always, even if empty) — the inverse: what
   other tickets/features depend on *this* one (relevant for foundational/
   infrastructure tickets, e.g. a data-pipeline ticket other features read
   from). Write "None." if nothing downstream depends on it yet.
10. **Out of Scope** (always) — this exact heading (the source drifted
    between "Out of Scope"/"Out of scope" — always capitalize both words).
    Bullets of explicit exclusions, mapped from the ADO ticket's "Out of
    scope" section.
11. **Future Opportunity** (only if something was deliberately deferred,
    not just excluded) — bullets naming functionality that's plausible
    later but explicitly not this ticket. Distinct from Out of Scope:
    Out of Scope says "not this," Future Opportunity says "not yet, and
    here's the shape of it." Omit entirely if nothing qualifies — don't
    force it.
12. **UI / UX Scope (Defined at High Level for Estimation)** (only when
    the ticket has UI work with no final design yet, e.g. Figma-pending)
    — this exact heading. State plainly that layout/interaction isn't
    defined here, list required *capabilities* only, close with a
    **Scope Note** reiterating that.
13. **Clarifications Needed** — **do NOT auto-populate this from the ADO
    ticket's "Open questions" and dump it into the doc.** This is the #1
    thing the human has repeatedly stripped out. The process is
    **collaborative**: when a ticket has open questions, **ASK THE HUMAN
    directly in chat** (AskUserQuestion or a plain numbered list) and fold
    their answers into the relevant section — usually Requirements. Only
    include a written "Clarifications Needed" section in the doc if, after
    asking, the human explicitly says a specific item should stay open for
    the client to decide. Default to NOT having this section at all. Never
    tag questions with `(Client)`/`(Dev Team)` attributions in the doc —
    the human rejected that too. Same rule applies to the "Decisions
    Requiring Confirmation" subsection under Requirements: don't manufacture
    it from ticket open-questions; ask first, then state the resolved
    answer as fact.
14. **Important Callout** (only for the highest-stakes or most easily
    misread tickets) — this exact heading (the source alternated
    "Important Callout (Client Alignment)" / "Final Note (Important
    Alignment)" — pick one). 1-2 closing paragraphs restating the scope
    boundary in plain terms, for client sign-off. Don't add this to every
    ticket — only where misalignment risk is real (e.g. an override
    mechanism that bypasses normal rules). **Don't invent an interim/staged-
    rollout risk** (e.g. "between this ticket and that one shipping, two
    systems could briefly disagree") unless the human has confirmed the
    tickets in this SOW actually ship on separate, independent timelines —
    when the whole ticket set ships together as one program/release, there
    is no live interim window for the client to be alerted to, even if the
    ADO ticket's own "known risk"/"sequencing" language talks about one
    (that language describes engineering sequencing *within* the build, not
    a state the client will ever observe).

Formatting to match: horizontal rule between major sections; `must`-phrased
declarative requirement bullets; monospace for literal values/formulas;
nested bullets (`o`, then `▪`) for sub-detail, not deep heading nesting.

## How to run

1. Confirm the ticket set and reread any richer source doc (step above).
2. Fetch each ADO ticket's title + full description (Acceptance Criteria,
   Constraints, Examples, Out of scope, Open questions, Related links).
3. For **one ticket at a time** (or a small batch the human requests):
   a. Decide which sections from the template above actually apply, based on
      the ticket's real complexity — don't force sections that don't fit.
   b. Draft the section using only what's derivable from the ADO ticket text.
   c. Explicitly flag anything you couldn't derive and are guessing at or
      omitting: a missing client quote, a Business Intent framing you're
      unsure reflects the actual sales conversation, a UI-placement
      description with no source. Ask the human directly rather than
      inventing plausible-sounding filler — a fabricated "per client
      request" quote in a document going in front of the client is a real
      problem, not a minor error.
   c2. **Any open question the ticket carries is a question FOR THE HUMAN,
      not a line item for the doc.** Ask it in chat, get the answer, fold
      it in. Do not park unresolved questions in the client-facing document
      and move on — that is the opposite of collaborative and the human
      finds it infuriating. Verify factual premises ("today the system does
      X") with the human too rather than asserting them from the ADO
      ticket's own context notes — those notes can be wrong.
   d. Show the drafted section(s) and wait for correction/approval before
      moving to the next ticket.
4. Once all requested tickets are approved, assemble the full document:
   title page (`{Product} / {SOW name} / Estimates and Documentation /
   {date}`), a Table of Contents listing each ticket's title and starting
   page/section, then the approved sections in ticket order.
5. If the human wants a rendered file (not just Markdown), use the
   available `docx`/`pdf` skill to convert — only after content is approved,
   never before.

## Review checkpoint

This skill is collaborative by design — the human reviews **each section as it's
drafted** (see How to run), not just at the end. Reinforce that:

1. Show each section and wait for correction/approval before the next.
2. Explicitly surface anything you couldn't derive or are guessing at, and ask
   rather than filling it in.
3. Assemble the full document only after every section is approved, then confirm
   whether the human wants it rendered to docx/PDF.

## Rules

- Never fabricate a "per client specification" quote, a business-intent
  claim, or a "confirmed with client" status that isn't actually backed by
  the ADO ticket or the human's direct input in this conversation.
- Don't silently promote an "Open question" into a confirmed requirement —
  those are different confidence levels and conflating them misrepresents
  what's actually settled to whoever reads this before signing.
- **Use the exact heading strings fixed above, every time** — `Requirements`,
  `QA / Scope Notes`, `Dependencies`, `Downstream Dependencies`,
  `Out of Scope`, `Future Opportunity`, `Clarifications Needed`,
  `Important Callout`. Don't let wording drift ticket to ticket the way the
  SOW 4 source did — a reader skimming 15+ sections should be able to
  pattern-match on identical headings, not reparse a slightly different
  phrase each time.
- **Dependencies and Downstream Dependencies are always present**, even if
  the answer is "None." — an omitted section and an empty one look
  identical to a reader unless you say so explicitly.
- **Requirements always nests the same way**: H2 → H3 named facets → bold
  inline label for named sub-items when needed. Don't drop to a flatter or
  deeper structure because one ticket's source material happened to be
  organized differently — normalize it into this shape when drafting.
- Match section *presence* to real complexity (a one-line ticket doesn't
  need Future Opportunity or UI/UX Scope) — but when a section is used,
  its heading and internal structure don't vary.
- Work in the increments the human wants (one ticket, a few, or the whole
  set) — don't default to drafting everything in one shot for a document
  this consequential.
