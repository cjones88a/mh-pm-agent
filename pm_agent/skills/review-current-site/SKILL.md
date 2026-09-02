---
name: review-current-site
description: Audit the live aspenvalleyhealth.org static site — page by page — to document what actually exists today, as grounding for AVH Contentful migration tickets and implementation decisions. Use before drafting migration tickets for a page/section, when scoping a rebuild, or when cross-checking a Figma design against the current site.
---

# review-current-site

The AVH migration has two sources of truth that have to agree: the Figma
designs (what the new site should be) and the current live site at
**aspenvalleyhealth.org** (what actually exists today and has to survive the
move). This skill covers the second half — reading the live site directly so
nothing gets silently dropped, and nothing gets ticketed as "new" that's
actually a straight port of existing content.

## When to use this

- Before drafting migration tickets for a specific page or section, to know
  what content/behavior on the live page has to be accounted for.
- When scoping a page rebuild and the client asks "what's actually on this
  page today?"
- When cross-checking a Figma frame against the current site to catch content
  gaps (in either direction) before tickets are drafted.
- When compiling a full content inventory of the current site for planning.

## Step 1: Establish scope

Confirm with the user whether this is a full-site pass or a specific
page/section (e.g. "just Services" or "the homepage vs. the new Figma
homepage frame"). Don't audit the whole site when only one page is in scope —
it's slow and produces noise the user didn't ask for.

## Step 2: Read the live site — don't rely on memory

Fetch the actual current page(s) (WebFetch, a connected browser tool, or
whatever's available in the session) every time this skill runs. The site can
change between runs, so a prior audit or anything remembered from an earlier
session is a starting point to verify, never a substitute for re-reading the
live page.

As of the last time this skill's author checked, the site's rough shape was:

- **Top nav:** Services, Patients & Visitors, Giving, Who We Are, Healthy
  Journey, Events, News, MyChart, Careers, Contact.
- **Known page paths:** `/`, `/about`, `/contact`, `/services` (+ subpages:
  `/services/primary-care`, `/emergency`, `/surgery`, `/birth-center`,
  `/behavioral-health`, `/cardiology`, `/orthopaedics`, `/oncology`),
  `/doctors`, `/patients`, `/mychart`, `/news`, `/events`,
  `/healthy-journey`, `/careers`, `/giving`, `/privacy`, `/accessibility`.

Treat that list as a hint of where to look, not as fact — confirm each page
still exists and re-read its actual current content before documenting
anything about it. If `/sitemap.xml` is reachable, pull it fresh to catch
pages added or removed since.

## Step 3: Document each page

For every page in scope, capture:

- **URL and purpose** — what the page is for, in one sentence.
- **Content blocks present** — hero, intro copy, stat/card grids, staff or
  provider listings, location/map widgets, forms, embedded scheduling or
  MyChart links, PDFs or downloads, testimonials, FAQs, footers/banners —
  whatever is actually there.
- **Dynamic or third-party elements** — anything that isn't static content:
  a provider search/finder, an interactive map, a scheduling widget, a
  MyChart integration, an embedded form, a review widget. Flag these
  separately — they're the pieces least likely to be a clean drop-in to a
  Contentful content model and most likely to need their own ticket or a
  technical decision.
- **Metadata** — page title, meta description, and anything else visible that
  affects SEO, if readable from the fetch.

Never describe a page's content from assumption or from what a similar
healthcare site "probably" has — only report what was actually fetched and
read in this pass.

## Step 4: Cross-check against Figma, if a design exists for this page

When the corresponding Figma frame is available (see [[figma-to-tickets]]),
compare the two directions:

- **Content on the live site but not in the Figma frame** — a likely
  migration gap. Flag it; don't assume it was intentionally dropped.
- **Content in the Figma frame but not on the live site** — net-new content
  the client will need to supply, not a straight port.

Present this as an explicit list, not a paragraph — it's the part of the
audit most likely to get acted on directly.

## Step 5: Flag migration risks

Call out anything that won't be a simple "copy the text into a new component"
job:

- Forms and integrations (MyChart links, scheduling widgets, third-party
  embeds) — these need a technical decision about how (or whether) they're
  rebuilt, not just content migration.
- PDFs, downloads, or documents — decide whether they get migrated as-is,
  converted to page content, or replaced.
- Interactive elements (maps, provider finders, search) — likely out of scope
  for a content migration ticket and worth their own ticket.
- Legal/compliance pages (privacy, accessibility) — confirm whether these are
  in scope for redesign or should be ported verbatim.

## Output format

One entry per page:

```
### <Page title> — <URL>
**Purpose:** <1 sentence>
**Content blocks:** <list>
**Dynamic/third-party elements:** <list, or "none observed">
**Figma comparison:** <gaps in either direction, or "no corresponding frame reviewed">
**Migration risks:** <list, or "none observed">
```

Followed by a short summary: pages audited, gaps found, and risks flagged.

## Handing off

- A page's audit feeds directly into [[figma-to-tickets]] or
  [[draft-ticket]] as grounding — reference the specific content blocks and
  risks found here rather than re-describing the page from scratch when
  drafting its ticket.
- If the audit surfaces open questions (ambiguous ownership of a piece of
  content, an integration nobody has a plan for), surface them to the user
  directly rather than guessing at a resolution.

## Rules

- Always re-fetch the live site; never answer from a stale memory of what a
  page contains.
- Never fabricate a page's content, structure, or metadata.
- Keep the scope the user asked for — don't expand a single-page request into
  a full-site crawl unprompted.
- Separate "content" from "dynamic/third-party elements" — they get migrated
  in very different ways.
