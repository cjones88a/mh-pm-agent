---
name: ticket-standards
description: The house standard for a well-formed agile work item — the Cohn user-story template, the INVEST quality check, vertical slicing, and the Definition of Ready checklist. Use when drafting, grooming, or reviewing a user story, bug, or task before it goes on the board.
---

# Ticket Standards

The shared quality bar for work items on the board. Apply it when drafting a new
ticket, grooming the backlog, or reviewing a story before it is picked up. It
pairs with [[draft-ticket]] (turn a rough idea into a story), [[triage-ticket]]
(grade an existing story), and [[estimate-ticket]] (size it).

## 1. The Cohn user-story template

Write user stories in the Mike Cohn form so intent, actor, and value are all
explicit:

> **As a** \<role/persona\> **I want** \<capability\> **so that** \<benefit\>.

- The **role** is a concrete persona, not "the user" — name who actually does this.
- The **capability** is a single user-facing outcome, not an implementation step.
- The **benefit** is the reason it is worth doing — if you can't state it, question
  whether the story belongs on the board.

Bugs and tasks don't need the Cohn form, but still need a clear title, the
observed-vs-expected behavior (bugs), and explicit acceptance criteria.

## 2. INVEST quality check

A good story is:

- **I**ndependent — can be built and shipped without being blocked by a sibling
  story; minimal ordering dependencies.
- **N**egotiable — describes the outcome, not a frozen implementation contract;
  leaves room for a conversation about *how*.
- **V**aluable — delivers observable value to a user or the business on its own.
- **E**stimable — clear enough that the team can size it; if it can't be
  estimated, it's under-specified or too big.
- **S**mall — fits comfortably inside a single iteration; if it doesn't, split it.
- **T**estable — has acceptance criteria a test (or a tester) can pass/fail
  objectively.

If a story fails an INVEST letter, name which one and fix it before it's Ready.

## 3. Vertical slicing

Slice stories **vertically** — each one delivers a thin, end-to-end piece of
user-visible value through every layer it touches (UI → API → data), not a
horizontal layer on its own.

- ✅ "As a member I can cancel a single upcoming booking" (touches the whole
  stack, shippable, demoable).
- ❌ "Build the bookings database table" / "Add the cancel button (no wiring)"
  — horizontal slices that deliver nothing usable alone.

If a story is really a horizontal slice, either fold it into the vertical story
it serves or re-cut the work so each ticket stands up on its own.

## 4. Definition of Ready checklist

A story is **Ready** to be pulled into a sprint only when all of these hold. If
any fail, the story stays in grooming — say which ones and what's missing rather
than filling the gap with an assumption.

- [ ] **Persona & value** — the Cohn "as a / so that" names a real actor and a
      real benefit.
- [ ] **Acceptance criteria** — present, and each one is objectively pass/fail
      (Given/When/Then or a concrete checklist), not "works well".
- [ ] **Example** — at least one concrete input → expected output, including a
      realistic edge case, for any non-trivial behavior.
- [ ] **Constraints** — hard rules stated: formats, limits, uniqueness,
      permissions, what must *not* happen.
- [ ] **Scope boundary** — what's explicitly out of scope is written down.
- [ ] **Dependencies** — known blocking dependencies are identified (or "none").
- [ ] **INVEST** — passes all six letters; if not, the failing letter is
      addressed.
- [ ] **Sized** — small enough to fit one iteration; larger stories are split
      first.

## Rules

- Describe **what** should happen, not **how** to build it — keep tables,
  libraries, and component names out of the story body.
- Don't invent business rules to fill a gap. If only the author knows the
  answer, list it as an open question and ask rather than assuming a default.
- If a story bundles several unrelated outcomes, split it into separate stories
  before it's Ready.
- Never mark a story Ready with untestable acceptance criteria — that's the most
  common cause of wrong implementations.
