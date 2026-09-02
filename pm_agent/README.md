# pm-agent

AVH Contentful migration toolkit for Claude Code — an Azure DevOps work-item
agent centered on turning the Figma design file into a working backlog, plus a
small set of supporting ticketing skills.

> This is an AVH-specific fork of Mapleton Hill's general-purpose `pm-agent`.
> It's scoped down to Figma-driven ticket creation for this engagement; the
> feature-spec/prioritize-backlog/SOW/status-update/actuals-report skills from
> the general template were archived out of this copy rather than deleted —
> see [`_archived-pm-agent-skills/`](../_archived-pm-agent-skills) at the repo
> root if this engagement needs them back.

## What's inside

| Component | Type | What it does |
|---|---|---|
| `pm-agent` | agent | Orchestrates the skills below, end to end, and posts/updates the AVH Azure DevOps board. |
| `figma-to-tickets` | skill | The main entry point. Walks the AVH Figma file's pages/frames, confirms ticket granularity with you, and drafts one ticket per screen/component — grounded in what's actually in the design, never guessed. |
| `review-current-site` | skill | Audits the live aspenvalleyhealth.org site page by page — content, components, third-party integrations — so migration tickets account for what has to survive the move, not just what's in the new design. |
| `draft-ticket` | skill | Turns one slice or rough idea into a fully-specified story, interviewing you about the gaps first. |
| `triage-ticket` | skill | Grades an existing ticket for clarity/AI-readiness and rewrites it. |
| `estimate-ticket` | skill | Estimates engineering hours, split into Planning & Coding buckets. |
| `ticket-standards` | skill | The house quality bar — Cohn template, INVEST, vertical slicing, Definition of Ready. |

## How the pieces fit together

Two sources of truth feed the backlog: the Figma designs (what the new site
should be) and the live site (what has to survive the move). `figma-to-tickets`
and `review-current-site` each ground tickets in one of those; `draft-ticket`,
`triage-ticket`, and `estimate-ticket` handle everything that happens to an
individual ticket after that — refining one that needs more detail, grading a
pasted one, or sizing it.

```
  (Figma file/frame) ─► figma-to-tickets ─┐
                                          ├─► draft-ticket ─► estimate-ticket
  (live site page)   ─► review-current-site ┘   more detail    Planning +
                                                 on one ticket  Coding hours

  figma-to-tickets: one ticket per screen/component, granularity confirmed
  with you first so shared components never get ticketed twice.
  review-current-site: flags what's on the live page today that the Figma
  frame is missing (or vice versa) before tickets are drafted.

  pm-agent posts & updates all of these in Azure DevOps (mapleton / AVH).
  triage-ticket grades any ticket pasted in from outside this flow.
```

Entry points are flexible: a Figma link → `figma-to-tickets`; "what's actually
on this page today" → `review-current-site`; a rough single idea →
`draft-ticket`; a pasted ticket → `triage-ticket`; "how long / how big" →
`estimate-ticket`. Just describe the goal and `pm-agent` routes to the right
skill and offers the next one.

## Setup: Azure DevOps token

The bundled MCP server (`.mcp.json`) authenticates to Azure DevOps with a
Personal Access Token (PAT). The token is **not** stored in this repo — it is
read at runtime from the `AZURE_DEVOPS_PAT` environment variable, so every
person supplies their own and no secret is ever committed.

### 1. Create the PAT

In Azure DevOps: **User settings (top-right avatar) → Personal access tokens →
New Token**, then set:

- **Organization:** `mapleton` (not "All accessible organizations").
- **Expiration:** the shortest window that's practical — you'll rotate it when
  it lapses rather than leaving a long-lived credential around.

### 2. Scope it to the minimum required

Grant **only** the scope the agent needs to read, create, update, and delete
tickets — nothing more. Do **not** use the "Full access" option.

- Under **Scopes**, choose **Custom defined**, then enable:
  - **Work Items → Read, write, & manage**

That single scope covers all four operations the agent performs:

| Operation | Covered by |
|---|---|
| Read tickets | Work Items · Read |
| Create tickets | Work Items · Write |
| Update tickets | Work Items · Write |
| Delete tickets | Work Items · Manage |

Because Azure DevOps bundles read + create + update + delete of work items into
the single **Read, write, & manage** level, that is the *minimal* scope here —
"Read & write" alone cannot delete, and anything broader (Code, Build, Release,
Full access) grants access this agent must never have. Leave every other scope
category unchecked.

### 3. Store it as an environment variable

```bash
# Add to your shell profile (~/.zshrc, ~/.bashrc, etc.) — never commit this
export AZURE_DEVOPS_PAT="<paste-your-PAT-here>"
```

Restart Claude Code after setting it so the MCP server picks up the token.

> **Rotate on leak or expiry.** If a PAT is ever exposed, revoke it immediately
> in the same Personal access tokens screen and issue a new one — a scoped,
> expiring token limits the blast radius but does not eliminate it.

## Live site access

`review-current-site` reads aspenvalleyhealth.org directly (via WebFetch or
whatever browsing tool is available in your session) every time it runs — it
never answers from a cached memory of the site, since pages change. No setup
is needed beyond normal internet access.

## Figma access

`figma-to-tickets` reads the AVH design file directly if a Figma MCP tool or
connector is available in your session. If not, it falls back to whatever you
paste or share in the moment (the page/frame outline, exports, screenshots) —
it will never invent frame content it hasn't actually seen. The current AVH
file:

```
https://www.figma.com/design/haNEF3v9Zr1mF2fm6nWPya/AVH-IA---Content
```

## Run it with local code access for best results

For the sharpest output, **run Claude Code from inside the relevant repository
checkout** so the agent can read the actual codebase. The `pm-agent` agent and
the ticketing skills all have `Read`, `Grep`, and `Glob` access, and they use it
to ground their work in reality rather than guessing:

- **draft-ticket / triage-ticket** confirm which components, endpoints, and
  existing patterns a story touches — so acceptance criteria and constraints
  match how the system actually behaves instead of inventing plausible-sounding
  detail.
- **estimate-ticket** calibrates hours against real surface area (files/layers
  touched), established vs. novel patterns, and prior similar work in the repo —
  a far better estimate than sizing from the ticket text alone.
- **pm-agent** avoids fabricating context: it fills a story's technical detail
  from what it can actually read, not from assumption.

Without a local checkout the tools still work — they just fall back to what you
paste in, the Figma file, and Azure DevOps — but the results are weaker and more
assumption-driven. Point the session at the codebase the ticket concerns
whenever you can.

## Usage

The skills are **invoked by describing your goal in plain language**, not by
slash commands — `pm-agent` reads the request and routes to the right skill,
then offers the next stage in the pipeline. You don't name the skill; you say
what you want. Typical triggers:

| Say something like… | Runs | You get |
|---|---|---|
| *"turn this Figma page into tickets"* / *"build the backlog from the designs"* | `figma-to-tickets` | A batch of tickets, one per screen/component, grounded in the Figma file |
| *"what's actually on the current homepage?"* / *"audit the live site before we migrate this"* | `review-current-site` | A page-by-page inventory of the live site, plus gaps vs. the Figma design |
| *"draft a story for X"* / *"write a ticket for…"* | `draft-ticket` | One fully-specified, implementable story |
| *"triage this ticket"* (paste one) / *"is this ready?"* | `triage-ticket` | Verdict, element scorecard, and a sharpened rewrite |
| *"estimate this"* / *"how long would this take?"* | `estimate-ticket` | Planning & Coding hours with rationale |
| *"create this ticket in ADO"* / *"update #1509"* | `pm-agent` | The item posted/updated on the Azure DevOps board |

`ticket-standards` isn't invoked directly — the drafting and triage skills apply
it automatically as their quality bar.

### Chaining across the pipeline

You can also drive several stages in one go, staying in the loop between them:

> *"Turn the homepage and services frames into tickets, then estimate each
> one."*

`pm-agent` runs `figma-to-tickets`, shows you the batch, and — once you approve
it — sizes each ticket with `estimate-ticket`. It won't auto-run the whole
chain unattended; the batch review and any bulk creation are decision points it
checks with you.

## Verify the output — you're the reviewer

Every skill produces a **draft for you to review**, not a finished artifact, and
there's a review checkpoint at the end of each stage. Nothing advances to the
next stage — and nothing is written to Azure DevOps — until you confirm. Treat
each hand-off as your cue to check the work and prompt for changes; silence is
not approval.

**You can, and should, verify everything — especially estimates and the Figma
grounding.** `estimate-ticket` produces a judgment call, not a measurement: it
can misjudge surface area, miss a hidden dependency, or over/under-weight
novelty. `figma-to-tickets` can misread a frame or miss a shared component —
always check that a drafted ticket actually matches what's in the design before
it's created. The skills are there to do the legwork and show their reasoning;
the sign-off is yours.
