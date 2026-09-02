---
name: triage-ticket
description: Scrutinize a user story / Jira ticket for quality and AI-readiness. Use when the user pastes a ticket, user story, or requirements text and asks to triage, review, scrutinize, grade, or check it before implementation. Input is free-form text (Jira-style), NOT JSON.
---

# Triage Ticket

Scrutinize a plain-text user story (e.g. a Jira ticket) and judge whether it is
clear, complete, and unambiguous enough to implement correctly — especially by an
AI agent that cannot infer missing details. Ambiguous stories cause hallucinated
business rules, generic design patterns, and wrong edge cases.

The input is **free-form text**, not JSON. Do not demand a JSON schema. Evaluate
the prose against the six elements below and rewrite it into a sharper ticket.

## Pipeline

- **Inputs:** a pasted/existing ticket, **or** the output of [[draft-ticket]]
  handed here for a quality gate.
- **Outputs:** a verdict (Ready / Needs work / Not implementable), a scorecard,
  and a sharpened rewrite.
- **Next step:** once Ready, offer [[estimate-ticket]] to size it and `pm-agent`
  to post/update it. If it fails on scope (several stories in one), send it back
  through [[draft-ticket]] to re-slice.

## How to run

1. Read the pasted ticket. If nothing was pasted, ask the user for the ticket text.
2. Score each of the six elements (below) as ✅ present / ⚠️ weak / ❌ missing.
3. Flag every anti-pattern you find, quoting the offending phrase.
4. Give a verdict: **Ready**, **Needs work**, or **Not implementable**.
5. Produce a rewritten ticket that fixes the gaps. Where a fact is genuinely
   unknown (a business rule only the author knows), list it as an open question
   rather than inventing it.

## The six elements every story should cover

1. **Intent** — the user-facing outcome with zero ambiguity. Must name explicit
   inputs and outputs, validation rules, and data-flow expectations.
2. **Acceptance criteria** — verifiable conditions, not prose. Each should be
   checkable (a tester or test could pass/fail it). Prefer Given/When/Then or a
   bulleted list of concrete pass conditions.
3. **Examples** — concrete input values and expected outputs / realistic cases.
   Examples disambiguate everything; most hallucinations disappear once examples
   are added. A story with no examples is suspect.
4. **Constraints** — what must NOT happen and the hard rules: allowed formats,
   uniqueness rules, size/length limits, character restrictions, permissions.
5. **NFRs** — non-functional requirements: latency, performance, security,
   privacy (e.g. "must not log PII"), payload limits, cost. Without these the
   implementation defaults to generic, possibly slow/insecure choices.
6. **Metadata** — id/title, author, and created/updated context so the story is
   traceable. (Often supplied by Jira itself — note if absent.)

## Quality rules

- **Intent must define inputs and outputs.** Reject "user can save profile
  easily" → require "user can update name, email, and avatar, with validation
  and persistence."
- **Acceptance criteria must be checkable.** Each line should be objectively
  pass/fail, not a feeling. "Works well" is not acceptance criteria.
- **Demand at least one concrete example** for any non-trivial behavior.
- **Constraints are non-negotiable.** If formats, limits, or uniqueness aren't
  stated, flag them as missing — do not assume defaults.
- **Say what should happen, not how.** Implementation details (specific tables,
  libraries, component names) in a story are a smell — flag them.
- **Edge cases must be named.** Empty input, max size, duplicate, unauthorized,
  concurrent action, failure/error path.

## Anti-patterns to flag (quote the phrase)

- ❌ **Vague prose / weasel words** — "easily", "simply", "just", "etc.",
  "and so on", "should work", "user-friendly". AI cannot infer the missing detail.
- ❌ **Missing examples** — no concrete grounding for the logic.
- ❌ **Mixing implementation into the story** — tells the AI *how* instead of
  *what should happen*.
- ❌ **Unstated constraints** — no formats, limits, uniqueness, or permissions.
- ❌ **Untestable acceptance criteria** — cannot be turned into a pass/fail test.
- ❌ **Multiple stories in one** — several unrelated outcomes bundled together;
  recommend splitting.

## Output format

```
## Triage: <ticket title>

**Verdict:** Ready | Needs work | Not implementable

### Element scorecard
- Intent: ✅/⚠️/❌ — <note>
- Acceptance criteria: ✅/⚠️/❌ — <note>
- Examples: ✅/⚠️/❌ — <note>
- Constraints: ✅/⚠️/❌ — <note>
- NFRs: ✅/⚠️/❌ — <note>
- Metadata: ✅/⚠️/❌ — <note>

### Issues found
1. <issue> — quote: "<offending phrase>"

### Open questions for the author
- <fact that only the author can supply>

### Suggested rewrite
<the improved ticket text, prose with bulleted acceptance criteria and examples>
```

## Review checkpoint

The verdict and rewrite are a **draft for the human to review**.

1. Present the verdict, scorecard, and rewrite.
2. Explicitly ask the human to confirm or correct it — particularly any open
   questions you raised for the author, which only they can answer.
3. Fold in their input and show the revised version.
4. Only advance (estimate, or post/update in Azure DevOps via `pm-agent`) once the
   human confirms.
