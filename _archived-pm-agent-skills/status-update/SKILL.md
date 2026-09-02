---
name: status-update
description: Compile a client-facing progress update or release notes from a set of recently closed/updated Azure DevOps tickets. Use when the user asks for a status update, progress summary, release notes, or "what shipped" for a client or stakeholder.
---

# Status Update

Turn a set of recently closed or in-progress Azure DevOps tickets into a
**client-facing** progress update or release notes. Where [[draft-sow-doc]]
handles the *pre-work* scoping document, this skill handles *delivery* comms —
what actually shipped and what's next — for the same client audience.

## Pipeline

- **Inputs:** a ticket set + date range (e.g. "everything closed this sprint",
  an iteration path, or an explicit id list), fetched via `pm-agent`.
- **Outputs:** a plain-English update grouped into Shipped / In progress / Next,
  written for a non-technical stakeholder.
- **Next step:** the user reviews and sends it (this skill never sends on its
  own). Pairs with [[draft-sow-doc]] for the scoping side of the same program.

## When to use

The user asks for a status update, progress summary, release notes, sprint
recap, or "what did we ship for <client>" — anything that reports delivered work
outward.

## What to gather

1. The **ticket set**: an iteration/sprint path, a date range of state changes,
   or an explicit list of ADO ids. If not given, ask.
2. Each ticket's **actual state** — fetch it; only report something as shipped if
   its ADO state is genuinely Done/Closed. Never infer completion from the title.
3. The **audience** — confirm it's client-facing (this skill's default) vs. an
   internal recap, since the register differs.

## Register (client-facing)

Same tone discipline as [[draft-sow-doc]]:

- **Business outcomes, not implementation.** "Members can now cancel a booking
  themselves" — not "added a DELETE endpoint and a confirm modal".
- **Plain English**, no ticket jargon, no internal system names the client
  doesn't use.
- **Stay neutral about any system being replaced** — state what the new thing
  does; don't editorialize about the old one.
- **Only claim what actually shipped.** A ticket that's still In Progress goes
  under "In progress", not "Shipped", regardless of how close it looks.

## Output format

```
# <Client / Product> — Progress Update
_<date range>_

## Shipped
- **<outcome title>** — <what the client/user can now do, one or two lines>.
- ...

## In progress
- **<outcome title>** — <what it is and roughly where it stands>.

## Coming next
- <what's queued next, at a plain-English level>

## Notes
- <anything the client should be aware of, or "None">
```

## Review checkpoint

This goes to a client, so the human must review it before it leaves. This skill
never sends.

1. Present the drafted update.
2. Explicitly ask the human to verify it — especially that every "shipped" item
   really shipped and the tone reads right for this client.
3. Fold in corrections and show the revised version.
4. Hand the final version to the human to send; do not send it yourself.

## Rules

- **Ground every "shipped" item in the ticket's real ADO state.** Reporting
  unfinished work as done to a client is the failure mode to avoid above all.
- Don't fabricate a "coming next" roadmap — base it on the actual backlog /
  next iteration, or ask; if unknown, omit the section rather than inventing it.
- Report outcomes faithfully — if a planned item slipped, it's fine (and better)
  to say it's still in progress than to quietly drop it.
- **Never send the update.** Produce it for the user to review and send; sending
  client communications is the user's call, not this skill's.
