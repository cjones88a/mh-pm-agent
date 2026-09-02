---
name: draft-ticket
description: Turn a rough feature idea into a fully-defined user story / Jira ticket. Use when the user describes something they want built in loose terms and asks to write, draft, define, or flesh out a ticket or user story. Interviews the user about ambiguities before producing the ticket. Input is free-form text, output is prose (Jira-style), NOT JSON.
---

# Draft Ticket

Take a rough, under-specified feature idea and turn it into a complete, unambiguous
user story by **interviewing the user about the gaps first**, then writing the
ticket. The bar is the same six-element model used by [[triage-ticket]]: a finished
ticket must be clear enough to implement correctly without guessing.

The output is **prose** (a Jira-style ticket), not JSON. Do not demand a schema
from the user.

## Pipeline

- **Inputs:** a rough idea from the user, **or** a single ticket handed down
  from [[figma-to-tickets]] that needs more detail (which already carries the
  design reference and whatever persona/scope is evident from the frame).
- **Outputs:** one finished, implementable story in the format below.
- **Next step:** offer [[triage-ticket]] to grade it, [[estimate-ticket]] to size
  it, and posting to the board via `pm-agent`.

## How to run

0. **Check where the input came from.** If this ticket was handed down from
   [[figma-to-tickets]], **inherit** the design reference and whatever
   persona/scope is already evident from the frame — do **not** re-ask what's
   plainly visible in the design. Only interview about gaps the design doesn't
   answer. For a standalone idea with no Figma source, treat every element as
   potentially open and continue below.
1. Read the user's rough description. Mentally map it to the six elements below
   and note which are missing, vague, or contradictory.
2. **Ask about the ambiguities** — use the AskUserQuestion tool. Bundle related
   gaps into a small batch of questions (max 4 per round) rather than a long
   interrogation. Offer concrete options with a recommended default first when you
   can infer a sensible one; the user can always pick "Other".
   - Only ask what you genuinely cannot infer from the description, the codebase,
     or sensible defaults. Don't ask for the sake of asking.
   - Run another round only if the answers open new ambiguities. Stop once the
     six elements are covered.
3. Write the finished ticket in the output format below, folding the answers in.
4. Anything still genuinely unknown after asking → list under **Open questions**,
   do not invent it.

## The six elements the finished ticket must cover

1. **Intent** — user-facing outcome with zero ambiguity: explicit inputs,
   outputs, validation, and data flow. ("As a … I want … so that …" is fine, but
   it must still name the concrete inputs/outputs.)
2. **Acceptance criteria** — verifiable pass/fail conditions, ideally
   Given/When/Then or a bulleted checklist. Each must be objectively testable.
3. **Examples** — at least one concrete input → expected output, including a
   realistic edge case.
4. **Constraints** — hard rules and what must NOT happen: allowed formats,
   uniqueness, size/length limits, character restrictions, permissions.
5. **NFRs** — latency, performance, security, privacy (e.g. "must not log PII"),
   payload limits, cost — whichever apply.
6. **Metadata** — title, and any author/priority/labels the user provides.

## What to interview the user about

Ask whenever the rough input leaves these unclear:

- **Who** is the user/actor and **what outcome** do they want?
- **Inputs and outputs** — what data goes in, what comes back / changes?
- **Validation rules** — what makes input valid vs. rejected?
- **Constraints** — limits, allowed formats, uniqueness, permissions.
- **Edge / failure cases** — empty input, max size, duplicate, unauthorized,
  concurrent action, error path. How should each behave?
- **NFRs** — does it have performance, security, or privacy requirements?
- **Scope boundary** — what is explicitly out of scope for this ticket?

## Rules (same quality bar as triage-ticket)

- Define inputs and outputs explicitly; never leave them implied.
- Every acceptance criterion must be pass/fail testable — no "works well".
- Include at least one concrete example for any non-trivial behavior.
- State constraints rather than assuming defaults.
- Describe **what** should happen, not **how** to build it — keep
  implementation choices (tables, libraries, components) out of the ticket.
- Name edge cases explicitly; don't leave them to the implementer's imagination.
- If the idea bundles several unrelated outcomes, propose splitting it into
  separate tickets before drafting.

## Output format

```
## <Ticket title>

**As a** <actor> **I want** <capability> **so that** <benefit>.

### Intent
<concrete description: inputs, outputs, validation, data flow>

### Acceptance criteria
- [ ] Given <context>, when <action>, then <result>
- [ ] ...

### Examples
- Input: <concrete value> → Output: <concrete result>
- Edge case: <value> → <result>

### Constraints
- <hard rule / what must not happen>

### Non-functional requirements
- <latency / security / privacy / limit, or "None identified">

### Out of scope
- <explicitly excluded>

### Open questions
- <anything still unresolved after the interview, or "None">

**Metadata:** <title / priority / labels / author as provided>
```

## Review checkpoint

The finished story is a **draft for the human to review**, not something to post
straight to the board.

1. Present the story.
2. Explicitly ask whether anything needs changing — don't treat silence as
   approval.
3. Fold in corrections and show the revised version.
4. Only advance (triage, estimate, or post to Linear via `pm-agent`) once
   the human confirms.
