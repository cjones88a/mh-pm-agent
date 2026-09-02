---
name: feature-spec
description: Turn an epic or feature idea into a product-facing spec — intent, success metrics, scope boundary — and decompose it into candidate vertical-slice stories ready for drafting. Use when the input is bigger than a single story: "spec out this feature", "break this epic down", "what stories does this need".
---

# Feature Spec

Take a feature or epic (bigger than one story) and produce a product-facing
specification: what it's for, how success is measured, what's in and out of
scope, and a **decomposition into candidate vertical-slice stories**. It sits one
level above [[draft-ticket]] — this skill defines the whole feature and its
slices; `draft-ticket` then fleshes out each slice into a full story.

This is the front of the PM pipeline. Get the spec right and every downstream
skill inherits clean context instead of re-deriving it.

## Pipeline

- **Inputs:** a feature/epic description from the user; optionally a parent ADO
  Feature/Epic id (fetch it via `pm-agent`) and the local codebase for grounding.
- **Outputs:** a spec (intent, metrics, scope) **plus** a numbered list of
  candidate story slices, each carrying the inherited persona, scope boundary,
  and NFRs — the exact shape [[draft-ticket]] consumes.
- **Next step:** offer to run [[draft-ticket]] on each slice (it will inherit the
  context below and not re-ask it), then [[estimate-ticket]] and
  [[prioritize-backlog]] once slices exist. Post the epic + stories via `pm-agent`.

## When to use

The input describes a *capability or initiative*, not a single change — an epic,
a feature, "a whole flow", or something that clearly becomes several stories. If
it's already one story, skip straight to [[draft-ticket]].

## How to run

1. Read the feature description. Ground it in the codebase where possible (which
   systems/components it touches) so scope and slicing are realistic, not
   invented.
2. **Interview the user about feature-level gaps** with the AskUserQuestion tool
   (max 4 per round): who the primary persona is, the business outcome, the
   success metric, and the hard scope boundary. Only ask what you can't infer.
3. Draft the spec sections below.
4. Decompose into vertical-slice stories (method below). Show the slice list and
   let the user add/remove/merge before anything is drafted — **do not
   auto-draft all slices**; decomposition is a decision point.
5. Hand approved slices to [[draft-ticket]] one at a time.

## Spec sections

1. **Intent** — 1-3 plain sentences: what this feature lets a user do and why it
   matters to the business. Name the primary persona.
2. **Success metrics** — how you'll know it worked: the observable metric(s) or
   outcome(s), not "users are happy". At least one measurable signal.
3. **Scope boundary** — what this feature *is* and, explicitly, what it is
   **not**. This boundary is inherited by every slice, so state it once, here.
4. **Non-functional requirements** — feature-wide latency/security/privacy/cost
   constraints that apply across the slices (or "None identified").
5. **Open questions** — feature-level unknowns only the stakeholder can answer;
   don't invent answers.

## Decomposition method

Cut the feature into **vertical slices** per the [[ticket-standards]] rule — each
candidate story delivers a thin, end-to-end, shippable piece of user-visible
value, not a horizontal layer. Aim for slices that individually pass INVEST.

- Prefer the smallest slice that's still demoable on its own.
- If two slices can't ship independently, note the dependency rather than
  pretending it away.
- Don't decompose by technical layer ("build the table", "add the endpoint") —
  that's the anti-pattern `ticket-standards` warns about.

## Output format

```
# Feature: <name>

## Intent
<persona + what it enables + why it matters>

## Success metrics
- <measurable signal>

## Scope
**In:** <what this feature covers>
**Out:** <explicit exclusions — inherited by every slice>

## Non-functional requirements
- <feature-wide NFR, or "None identified">

## Candidate story slices
Each is a vertical slice ready for `draft-ticket`. Persona and scope-out are
inherited from above unless overridden.

1. **<slice title>** — <one-line outcome>. [depends on: #N | none]
2. **<slice title>** — <one-line outcome>. [depends on: none]
...

## Open questions
- <feature-level unknown, or "None">
```

## Review checkpoint

The spec and slice list are a **draft for the human to review**, not a finished
plan. Before any slice is drafted or anything is written to Azure DevOps:

1. Present the spec and the candidate slices.
2. Explicitly ask what to change — add, remove, merge, or re-scope slices; correct
   the metrics or scope. Don't treat silence as approval.
3. Fold in the corrections and show the revised version.
4. Only hand slices to `draft-ticket` once the human confirms the list.

## Rules

- Describe **what** the feature does, not **how** to build it — keep
  architecture out of the spec (that's a `dev-team` design-doc's job).
- Every slice must be a vertical slice; if you catch yourself writing a
  layer-only slice, re-cut it.
- Don't fabricate a success metric or a "confirmed" scope decision — if the
  stakeholder hasn't said it, it's an open question.
- Stop at the slice list and get approval. Drafting the slices is the *next*
  skill's job, run per-slice with the user in the loop.
