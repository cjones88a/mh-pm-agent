---
name: estimate-ticket
description: Estimate engineering hours for a drafted ticket/issue, split into Planning & Architecture and Coding & Verification, with a short rationale and a summary table. Use when the user asks to estimate a ticket, size an issue, put hours on a story, or scope out effort for a piece of work.
metadata:
  type: skill
---

# Estimate Ticket Hours

Turns a drafted ticket/issue into an hour estimate, split into two buckets —
**Planning & Architecture** and **Coding & Verification** — with a short
rationale and a summary table. General-purpose: works on any ticket in any
project, not tied to a specific tracker or codebase.

## Pipeline

- **Inputs:** a drafted ticket ([[draft-ticket]] output, a pasted ticket, or an
  ADO id fetched via `pm-agent`) — read it in full first.
- **Outputs:** hours split into Planning & Architecture and Coding &
  Verification, with a Total.
- **Next step:** the Total is what the human uses to size and sequence work
  input. `pm-agent` can write the estimate back to the ADO ticket.

## When to use

The user asks to estimate, size, or put hours on a ticket/issue/story, or
asks "how long would this take."

## Inputs needed

- The ticket itself: pasted text, a link/ID to fetch (ADO, Jira, GitHub
  Issues, etc. — use whatever tool is available in the environment), or a
  local file. If only an ID/reference is given, fetch the full ticket before
  estimating — never estimate from a title alone or guess at unstated
  content.
- Repo/project context, if any exists: architecture docs, a style guide, or
  prior similar tickets with known actual hours. These are for calibration,
  not required — proceed without them if none exist, but say so in the
  reasoning rather than silently estimating as if they were consulted.

## The two buckets

- **Planning & Architecture** — time to resolve ambiguity, decide on an
  approach, design data models/interfaces, and get alignment before writing
  code. Scales with: how novel the work is (new pattern vs. reusing an
  established one), how many systems/teams it touches, and how many
  genuinely open questions the ticket still has.
- **Coding & Verification** — time to implement, test, and get the change
  reviewed and merged. Scales with: volume of change (files/layers touched),
  number of distinct behaviors/acceptance criteria, edge-case and
  concurrency surface, and how much needs new test infrastructure vs.
  reusing existing test patterns.

Don't force a balanced split. A ticket that's mechanical to build but needs
real design work (e.g. a new data model, a cross-team dependency) can be
Planning-heavy; a ticket that's well-specified but touches a lot of surface
area can be Coding-heavy.

## How to estimate

1. Read the full ticket — requirements/acceptance criteria, constraints,
   out-of-scope, and any open questions. Unresolved open questions inflate
   Planning & Architecture (time to resolve them) and should be named in the
   reasoning, not silently estimated around.
2. Assess complexity signals:
   - **Novelty** — does this reuse an existing, established pattern in the
     codebase, or invent a new one?
   - **Surface area** — how many files/layers/systems does this touch?
   - **Behavior count** — how many distinct requirements or acceptance
     criteria are there, and how independent are they?
   - **Risk surface** — concurrency, data migration, external integrations,
     security-sensitive paths add Coding & Verification time
     disproportionately (more edge cases, harder to test).
   - **Ambiguity** — how many open questions remain unresolved?
3. If prior similar tickets with known actual hours are available (in the
   repo, in conversation history, or supplied by the user), anchor to them
   explicitly and name the reference ticket in the reasoning, rather than
   estimating from first principles alone.
4. Produce the output (below). If estimating several tickets together (e.g.
   a whole epic), estimate each individually, then add a rollup table
   summing both buckets and the total across all tickets.

## Output format

Two sentences of reasoning — what drives the estimate (novelty, surface
area, ambiguity, risk) — followed by a table:

| Category | Hours |
|---|---|
| Planning & Architecture | X |
| Coding & Verification | Y |
| **Total** | **X+Y** |

For multiple tickets, repeat per-ticket, then close with a rollup:

| Ticket | Planning & Architecture | Coding & Verification | Total |
|---|---|---|---|
| ... | ... | ... | ... |
| **Total** | | | |

## Review checkpoint

**Estimates are the least certain output in this pipeline** — they're a judgment
call, not a measurement, and the human's domain knowledge will often move the
number. Treat every estimate as a draft to be verified, never a settled fact:

1. Present the estimate **with the assumptions and open questions that drive it**
   spelled out — the reader can only sanity-check a number if they can see what's
   behind it.
2. Explicitly ask the human to confirm or correct it before it's used anywhere
   (anywhere it's used to sequence work or quoted to a client).
3. Fold in their adjustments and show the revised number.
4. Don't let an unreviewed estimate flow downstream — flag it as unverified if it
   must be passed on before the human has weighed in.

## Rules

- Never estimate a ticket that hasn't actually been read in full — fetch it
  first if only an ID/reference was given.
- State assumptions and open questions explicitly in the reasoning, rather
  than picking a number that silently assumes they're resolved favorably.
- Don't force categories to look balanced — let the actual work drive the
  split.
- If there's genuinely insufficient information to estimate with any
  confidence, say so and ask what's missing rather than producing a number
  anyway.
