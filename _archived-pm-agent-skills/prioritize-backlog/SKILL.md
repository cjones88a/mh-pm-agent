---
name: prioritize-backlog
description: Score and rank a set of tickets into a prioritized roadmap using RICE (default), ICE, or MoSCoW. Reuses estimate-ticket hours as the Effort input. Use when the user asks to prioritize, rank, sequence, or build a roadmap from a backlog of tickets.
---

# Prioritize Backlog

Take a **set** of tickets and turn them into a ranked, defensible roadmap. Where
[[estimate-ticket]] answers "how much effort is this one?", this skill answers
"which of these do we do first?" — combining value and effort into a single
ordering.

Default framework is **RICE**; ICE and MoSCoW are offered when they fit better.

## Pipeline

- **Inputs:** a set of tickets (ADO ids fetched via `pm-agent`, pasted text, or
  the outputs of [[feature-spec]] → [[draft-ticket]]); **Effort** taken from
  [[estimate-ticket]] hours where available.
- **Outputs:** a scored table ranked by priority, plus a roadmap grouping (Now /
  Next / Later).
- **Next step:** offer to write the ranking back to ADO via `pm-agent` (order
  field, tags, or iteration assignment), and to communicate it via
  [[status-update]] or [[draft-sow-doc]].

## When to use

The user has more than one candidate ticket and needs an order — "what should we
build first", "prioritize these", "make a roadmap", "sequence the backlog".

## Choosing a framework

- **RICE** (default) — best when you can estimate value quantitatively. Score =
  **(Reach × Impact × Confidence) ÷ Effort**.
  - **Reach** — how many users/events per period this affects (a number).
  - **Impact** — per-user effect, on the standard scale: 3 = massive, 2 = high,
    1 = medium, 0.5 = low, 0.25 = minimal.
  - **Confidence** — how sure you are of Reach/Impact: 100% / 80% / 50%.
  - **Effort** — person-hours (or person-weeks). **Pull this from
    [[estimate-ticket]]** — its Total hours is the Effort input. If a ticket
    hasn't been estimated, say so and estimate it first rather than guessing.
- **ICE** — lighter, when Reach is hard to quantify. Score = Impact × Confidence
  × Ease, each 1-10.
- **MoSCoW** — when the ask is scope-cut for a fixed deadline, not scoring: sort
  into Must / Should / Could / Won't.

Ask which framework if it's ambiguous; otherwise default to RICE and say so.

## How to run

1. Assemble the ticket set. Fetch full text for each (via `pm-agent`) — don't
   score from titles alone.
2. For each ticket, gather the framework inputs. **Reach and Impact are business
   facts you usually can't infer** — ask the user (AskUserQuestion, batched)
   rather than inventing numbers. Confidence reflects how grounded those inputs
   are.
3. Take **Effort from the ticket's estimate**. If missing, run
   [[estimate-ticket]] first or flag it as unscored.
4. Compute the score, rank descending, and group into a roadmap.
5. Show your work — the per-ticket inputs, not just the final order — so the
   ranking is auditable and the user can challenge a number.

## Output format

RICE example:

```
## Prioritization (RICE)

| Rank | Ticket | Reach | Impact | Conf. | Effort (h) | Score |
|---|---|---|---|---|---|---|
| 1 | #123 <title> | 500 | 2 | 80% | 16 | 50.0 |
| 2 | #130 <title> | 200 | 3 | 100% | 24 | 25.0 |
| ... |

## Roadmap
**Now:**  #123, ...
**Next:** #130, ...
**Later:** ...

**Notes:** <assumptions, low-confidence inputs, unscored tickets>
```

## Review checkpoint

The ranking is only as good as its inputs, and several of them (Reach, Impact, and
the **Effort it inherits from `estimate-ticket`**) are uncertain. Present the
roadmap as a draft to be verified:

1. Show the scored table with the per-ticket inputs, not just the final order.
2. Explicitly ask the human to sanity-check the inputs — especially any effort
   estimates that haven't been human-verified, and low-confidence Reach/Impact.
3. Fold in corrections and re-rank.
4. Don't write the order back to Azure DevOps until the human confirms it.

## Rules

- **Never fabricate Reach or Impact.** They're business facts — ask the user, or
  mark the ticket unscored. A made-up number produces a confident-looking but
  meaningless ranking.
- **Effort comes from [[estimate-ticket]]**, not from a fresh guess inside this
  skill — keep a single source of truth for effort.
- Show the inputs behind every score; a ranking you can't defend line-by-line is
  worse than no ranking.
- State low-confidence inputs explicitly in the notes rather than smoothing them
  into the score.
- Don't write the order back to ADO until the user approves it.
