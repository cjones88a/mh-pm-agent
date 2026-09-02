---
name: figma-to-tickets
description: Turn a Figma file, page, or frame into a batch of fully-specified Linear issues, one per screen/component/pattern. Use when the user shares a Figma link, or asks to turn designs into tickets, a backlog, or a sprint's worth of work.
---

# figma-to-tickets

The AVH Contentful migration is design-driven: the Figma file is the source of
truth for what gets built, and this skill is how it becomes a working backlog.
Given a Figma file, page, or frame, walk its structure, agree on ticket
granularity with the human, and draft one well-formed ticket per unit — grounded
in what is actually in the file, never in a guess about what a screen probably
contains.

## When to use this

- The user shares a Figma link (file, page, or single frame).
- The user asks to "turn the designs into tickets," "build the backlog from
  Figma," or similar.
- A previously-created ticket set needs a delta pass after the designs changed
  (see **Step 5a: re-syncing after a design change** below).

## Step 1: Read the file

If a Figma MCP tool or connector is available in this session, use it to
enumerate the file's pages and frames, and to read each frame's content,
component instances, and comments. If no such tool is available, ask the user
to paste the page/frame outline (Figma's left-hand layers/pages panel), share
exports or screenshots of the frames in scope, or grant access another way.

Either way: **never invent a frame's name, content, or behavior you have not
actually seen.** If something is ambiguous or not visible in what you were
given, say so and ask rather than filling the gap with a plausible-sounding
guess. This matters more here than almost anywhere else in the pipeline —
every ticket's acceptance criteria trace back to what's actually on the canvas.

## Step 2: Decide granularity with the human

Before drafting anything, confirm the unit of work with the user via
AskUserQuestion (or, if unavailable, ask directly in plain language). The
options are typically:

- **Per page/screen** — one ticket per distinct page template (e.g. "Homepage,"
  "Service Detail").
- **Per reusable component/pattern** — one ticket per component that appears
  across multiple screens (e.g. "Hero," "Card Grid," "CTA Banner"), built once
  and reused.
- **Both, but not double-counted** — screens get tickets for page-level
  assembly and content, while shared components get their own separate tickets;
  a screen's ticket should reference the component tickets it depends on rather
  than re-describing that component's build work.

Default to "both, but not double counted" when the user has no strong
preference, since it mirrors how a component-based CMS build (Contentful, in
this case) actually gets built. Whatever is chosen, hold to it for the whole
batch — don't let some screens get component-level tickets and others not
without saying so.

**Watch for the double-counting failure mode:** if a component (say, a "Team
Member Card") appears on both the "Who We Are" and "Leadership" screens, it
gets exactly one component ticket, not one per screen it appears on. Before
drafting, group frames by the components they share and flag any overlap to
the user.

## Step 3: Check for existing tickets

Before drafting anything new, check Linear for issues that already cover
a screen or component in scope, so the same design element never gets a
duplicate ticket. Surface any matches to the user and ask whether to update the
existing ticket instead of creating a new one.

## Step 4: Draft each ticket

For each screen or component the human confirmed in Step 2, draft a ticket to
the same bar as **[[draft-ticket]]** and **[[ticket-standards]]** — Cohn
template, INVEST check, vertical slicing, Definition of Ready. Ground every
element of the ticket in the Figma file:

- **Title** — the screen/component name as it appears in Figma (or a clear,
  consistent rename if the Figma name is unhelpful — note the rename).
- **Intent** — what this screen/component does for the user, inferred only from
  what's visible (layout, copy, evident interaction states) — not assumed
  business logic.
- **Acceptance criteria** — content blocks, states (default/hover/empty/error
  if shown), responsive behavior if multiple frame sizes are given, and any
  Contentful content-model implications (what becomes an editable field vs.
  fixed markup).
- **Design reference** — the direct Figma link (file + node id) for this
  screen/component, so the ticket always points back to its source of truth.
- **Open questions** — anything ambiguous in the design, plus any actual design
  comments found in Figma (quote them, don't paraphrase into something more
  definite than they are).

Apply **[[estimate-ticket]]** to each drafted ticket only if the user asks for
sizing at this stage; otherwise leave estimation for a follow-up pass so the
batch review in Step 5 isn't overloaded.

## Step 5: Batch review checkpoint

Present the whole batch of drafted tickets together — not one at a time — so
the user can review consistency across the set (naming, granularity, any
missed overlap) before anything is created. Explicitly ask whether the batch
needs changes; silence is not approval. Only after the user confirms the batch
does pm-agent create or update the items in Linear, using the Linear MCP
tools (`save_issue`) — never fabricate an issue ID or claim an item was
created if the tool call didn't succeed.

### Step 5a: re-syncing after a design change

When the user comes back after the Figma file has changed, don't re-draft the
whole batch. Re-read the affected pages/frames, diff against what the existing
tickets describe, and propose only the deltas: new tickets for new
screens/components, updates for changed ones, and a flagged (never
auto-closed) list for anything that appears to have been removed from the
design.

## Output format

Present each ticket in the batch as:

```
### <Title>
**Type:** User Story | Task
**Design reference:** <Figma link with node id>
**Intent:** <1–2 sentences>
**Acceptance criteria:**
- ...
**Open questions:**
- ...
```

Followed by a short summary line: how many tickets, how many are net-new vs.
duplicates-avoided, and any granularity overlap that was flagged in Step 2.

## Rules

- Never invent frame content, copy, or behavior that isn't visible in what you
  were given — ask instead.
- Never double-count a shared component across multiple screen tickets.
- Never create or update anything in Linear before the batch-level human
  confirmation in Step 5.
- Always check for an existing ticket before drafting a new one for the same
  screen/component.
- Keep every ticket traceable to a specific Figma node, not just "the designs"
  generally.
