---
name: product-release
description: "Use when a teammate wants to generate a release brief for a specific initiative in the \"Cross-Team Work and Learning\" Notion database, or for the Linear project/ticket behind it — covering what marketing, support, and sales need before and at launch. Triggers: \"generate a release brief for X\", \"run product-release on [Notion URL]\", \"draft the release brief for the CalcsPlatform refactor\", \"brief this release\", pasting a Cross-Team Work and Learning card URL or a Linear project/ticket link and asking for a brief."
---


## Overview

**Workspace details:** see `references/workspace-config.md` for this install's two database
IDs/URLs and customer list. Read it before Step 1.

Generates the release brief defined in "Release Ways of Working" — Title, What it does, Messaging /
Positioning, Who asked for this / Availability, What did we build, screenshots/video, release size,
timing, channel, status, support/FAQ split, and co-marketing/white-label alignment — sourced from a
card in the **Cross-Team Work and Learning** Notion database, plus any related cards that belong to
the same release family (see Step 3). A linked Linear ticket or project, when one exists, is added as
a **reference link only** — this brief stays focused on customer- and marketing-facing information,
not engineering detail, so Linear content is never fetched or read into the brief's fields. Not every
card will have a Linear link, and its presence or absence is never itself an output field.

**Output target:** the brief is written as its own entry in the **Product Release templates**
database (`<PRODUCT_RELEASE_DB_ID>`, at
<PRODUCT_RELEASE_DB_URL>), not appended as content on the source
card. The entry's `Source Card` property is a two-way relation to the source card's data source
(`<CROSS_TEAM_DB_ID>`); creating that relation automatically adds a
reciprocal `Release Brief` property back on the source card, so the card always carries a live
reference to its brief without any manual back-linking step. When a release family has more than one
confirmed card (see Step 3), `Source Card` links to all of them, so every card in the family carries
that same reciprocal reference.

**Properties vs. page content:** the data source keeps only six properties, chosen because they're
the fields worth filtering/sorting a database view on — `Title`, `Category/Area`, `Release Size`,
`Go-live Date`, `Source Card`, `Channel`, and `Status`. Everything else is written as page
**content** under `##` headings, not as separate properties — these are narrative fields that read
better as a document than as table cells, and cramming them into properties was reversed for exactly
that reason. Do not re-add content-only fields as properties without the user asking for that back.

**No Region field.** Region was dropped from the brief entirely (not just demoted to content) — it
isn't a relevant axis for Calcs.com releases, so don't ask about it and don't include a "Region"
heading in the page content.

**Content order and grouping (fixed — do not reorder, re-split, or rename without the user asking):**
1. What it does / Problem it solves
2. Messaging / Positioning — the one-line hook / why a partner should care right now, distinct from
   the factual "what it does." Always human-authored; default to "Not available" with the suggested
   prompt below (see Step 4).
3. Who asked for this / Availability — **one merged section**, not two. Covers who requested/is
   using the release (`Customer`) and who it is/isn't available to. Never split or rename this.
4. What did we build
5. Screenshots/video
6. Support articles
7. Internal FAQ (support/sales) — **separate from External FAQ.** Internal FAQ is for support/sales
   fielding questions; it can be more technical/candid.
8. External FAQ (customer-facing) — written for the customer to read directly; never merge this back
   into a single "FAQ" heading.
9. Co-marketing/white-label

Core principle: **only state what's actually recorded.** Never infer, estimate, or guess a field's
value. Anything not present on the source card(s) is marked "Not available" and flagged for a human
to fill in — this skill does not write marketing copy from assumptions. Recorded-but-scattered still
counts as recorded: a fact written on any confirmed card in the release family (Step 3) is a valid
source, the same as if it were on a single card. The one exception to "never draft" is "What it does
/ problem it solves": when the family has enough recorded material to work from, this skill drafts it
itself, sourced only from already-recorded facts, and marks the draft **"Drafted by Claude — validation
required"** rather than presenting it as a confirmed fact (see Step 4b). Messaging / Positioning and
both FAQ sections are always left as "Not available" for a human — this skill does not draft
positioning angles or FAQ answers itself, even on request; the auto-draft exception is limited to
"What it does / problem it solves" only.

---

## How to run

### Step 1 — Resolve the input

The user gives one of three things:
- **A Notion URL** → fetch it directly with `notion-fetch`.
- **A Linear URL or ticket reference** → search the data source for a row whose `Linear ` property
  matches it (`notion-query-data-sources`, SQL mode, `WHERE "Linear " LIKE '%...%'`).
- **Free text** (e.g. "CalcsPlatform refactor") → SQL search the `Learning` title column, using
  `Segment / Product Layer` and `Workstream` as secondary filters if the text hints at a product area.

Data source: `<CROSS_TEAM_DB_ID>` ("Cross-Team Work and Learning").

If more than one row matches, list each candidate with its `Learning` title, `Segment / Product Layer`,
and `Status`, and use **AskUserQuestion** to have the user pick one. Never guess between close matches.
This picks the **primary card** — the one the dedup gate (Step 2) checks and the one `Release Brief`
gets its main reciprocal link from. Step 3 finds the rest of the family around it.

### Step 2 — Check for an existing brief (dedup gate)

Before writing anything, check whether the matched primary card already has a populated `Release Brief`
relation property (i.e. it already links to an entry in Product Release templates).

- **If none exists:** proceed to Step 3.
- **If one exists:** fetch that entry and **always show its `Status` property to the user alongside
  the rest of the entry.** Then branch on `Status`:

  - **`Status: Published`** — Tell the user plainly that this is a published release note and cannot
    be changed through this skill. **Do not offer to update it, do not ask via AskUserQuestion, do
    not accept an override even if the user insists.** State the reason (it's already published) and
    stop there for this entry. If the user wants changes made, that has to happen outside this skill
    (e.g. direct edit in Notion by whoever owns the publish), not through product-release.
  - **`Status: Ready`** — Show the entry, then tell the user updates have already been made to this
    brief and it's marked ready, and recommend they speak with the entry's owner or marketing before
    changing it further. This is advisory, not a hard block: if the user still wants to proceed after
    hearing that, use AskUserQuestion to confirm before editing, same as the Draft path below.
  - **`Status: Draft`** — Show the entry and ask (via AskUserQuestion) whether to update it or leave
    it as-is, same as before. Never regenerate or overwrite it silently.

This mirrors the same no-duplicate convention used by `opportunity-page-creator` and
`project-page-creator`, applied here via the relation instead of an in-page section, with the
`Status`-gated behavior above layered on top.

### Step 3 — Identify the release family (related-card rollup)

Before populating anything, find other cards in the same "Cross-Team Work and Learning" data source
that are really part of the same release as the primary card, so a fast-moving initiative that's
scattered across several cards (a CLI tool card, a live-design-tool card, a beta-cohort card, etc.)
gets synthesized into one brief instead of leaving each card's version looking thin. This step is
purely additive sourcing — it never changes what gets written or how; it only widens where facts can
come from.

**If Step 2 found an existing brief, start from its recorded family.** An existing entry's `Source
Card` relation is already a human-confirmed release family from a previous run — treat every card
already listed there as confirmed, no re-confirmation needed. Then run the matching below only to
look for **new** candidates not already in that list, and confirm just those before adding them.
Re-deriving the whole family from tags on every update would force re-confirming already-correct
cards alongside any new false positives the tags happen to catch — the recorded list is the stronger
signal and takes precedence.

**If there's no existing brief (or Step 2's entry has no `Source Card` recorded yet), match fresh, in
order:**
1. **Relation match (preferred).** If the primary card has a populated `Parent item` or `Sub-item`
   relation, treat every card reachable through that relation (parent, siblings under the same
   parent, and children) as part of the family.
2. **Tag match (fallback).** Only when the primary card has no `Parent item`/`Sub-item` relation at
   all, search the data source for other cards sharing both the same `Workstream` and the same
   `Segment / Product Layer` as the primary card. This is a looser signal — it can pull in unrelated
   work under a broad tag — so treat it as candidates, not a confirmed set.

**Always confirm before merging any candidate that isn't already a confirmed family member.**
Whichever match fired, show the user the candidate set — each card's `Learning` title, `Status`, and
why it matched ("parent item" / "sub-item" / "tag match: Workstream + Segment") — via
**AskUserQuestion**, and let them add, remove, or accept the set as-is. **Never merge a card's content
into the brief without this confirmation**, even a relation-matched one — a Parent item link can exist
for reasons unrelated to release scope, and a tag match can just as easily catch a same-tagged card
that belongs to a different initiative entirely (e.g. two unrelated workstreams sharing a Segment). If
no candidates are found beyond an already-confirmed family (or, on a fresh match, no related cards are
found at all), say so plainly and proceed with what's already established (this is normal, not a gap).

**If a candidate card already has its own `Release Brief`** (a different one from the entry in Step
2): flag it in the same confirmation step ("[title] already has a brief: [link]") rather than silently
pulling it in or silently skipping it. The user decides whether to still use its recorded facts as
source material for this brief, treat it as the primary card instead, or leave it out.

The confirmed set (the existing family, if any, plus the primary card plus any newly confirmed
candidates) is the **release family** for the rest of this run. Step 4 sources from all of them
together.

### Step 4 — Populate the brief

Map source properties to brief fields, sourcing from every card in the confirmed release family
(Step 3), not the primary card alone. Fields marked (property) go on the Product Release templates
entry's properties; fields marked (content) become a `##` heading + body text in the entry's page
content, written **in the fixed order given in Overview**.

| Brief field | Where it lives | Source |
|---|---|---|
| Title | property | Primary card's `Learning` |
| Category/Area | property | Primary card's `Segment / Product Layer` + `Workstream` |
| Release Size | property | Primary card's `Release Size` |
| Go-live Date | property | Primary card's `Launch date` / `Completed At` (`Status` = `Live` confirms shipped) |
| Channel | property | Primary card's `Channel` multi-select (Email, In-app, Website, Ads, Sales, Product, Content/SEO, Social, Other, Events) — copy directly, don't infer a channel that isn't set on the card |
| Status | property | not sourced from any card. **Always `Draft` when an entry is first created by this skill**, regardless of how complete the brief looks at creation time — a human promotes it to `Ready` (confirmed and content-complete) or `Published` (marketing has actually shipped their content) later, outside this skill's normal flow. Never set `Ready` or `Published` at creation. On an update to an existing entry, leave `Status` untouched unless the user explicitly asks to change it — and never touch it at all if `Status` is currently `Published` (see Step 2). |
| Source Card | property | relation to every card in the confirmed release family (see Step 3 and Step 6) |
| What it does / problem it solves | content (1st) | Not tracked as a discrete property. Auto-drafted from recorded facts when the family has enough material (see Step 4b), marked "Drafted by Claude — validation required"; falls back to "Not available — required human input" when there isn't enough to draft from. |
| Messaging / Positioning | content (2nd) | Not tracked anywhere — always "Not available — required human input (owner: [X]). Suggested prompt: what's the one-line hook, and why should a partner care about this right now?" Never draft this yourself, even under Step 4b — Step 4b covers only "What it does." |
| Who asked for this / Availability | content (3rd, merged) | Who asked for this: `Customer` from any card in the family (see references/workspace-config.md for your customer list). Availability: not tracked on any card — mark "Not available" unless a card's free text names specific partners beyond `Customer`. Write both under one `## Who asked for this / Availability` heading. |
| **What did we build** | content (4th) | Each family card's own **"What will we try?"** section (the shipped/scope bullet list), combined. This is a factual extraction, not synthesis — copy the recorded bullets describing what was actually shipped, with no added claims, framing, or omissions; where more than one card contributes, attribute each bullet to its card's title **and link that title to the card's own URL**, so a reader can click through to any item's full detail without the brief itself carrying that detail. If any family card has a linked Linear ticket or project, add it as its own reference link too (e.g. "Linear: [link]") — a pointer for engineering-level detail, not a source to read from. If no card in the family has such a section, mark "Not available." |
| Screenshots/video, Support articles | content (5th–6th) | Not tracked as a discrete property, so there is nothing to synthesize or draft here — **but if any family card's content already contains an explicit link to one** (a screenshot, a Loom/video URL, an existing help-center or support-article link), include it as a plain reference link rather than marking "Not available." This is link-only, the same as the Linear rule — never go looking beyond what a card already links, and never draft or describe the screenshot/article yourself. Nothing found → "Not available" and list as a required human input, never inferred. |
| Internal FAQ (support/sales) | content (7th) | Same link-only rule as above — a family card may already link an internal FAQ doc or a relevant support thread; if so, link it. Nothing found → "Not available — required human input (owner: [X]/support)" |
| External FAQ (customer-facing) | content (8th) | Same link-only rule as above — a family card may already link a customer-facing FAQ or help article; if so, link it. Nothing found → "Not available — required human input (owner: [X])" — keep separate from Internal FAQ, never merge |
| Co-marketing/white-label | content (9th) | Not tracked here — always mark "Not available" and list as a required human input, never inferred |

There is no Region field — do not add one, even if a linked Linear ticket or project happens to
mention a region in passing.

**Combining more than one card.** When two or more cards in the family both record a value for the
same field (e.g. two cards each have a `Customer` or an `Assumption`), include both rather than
silently picking one. If they genuinely **conflict** rather than merely add detail (e.g. one card's
`Launch date` disagrees with another's, or one says a feature is in scope and another says it was
cut), do not resolve the conflict yourself — write it inline as a flagged conflict, naming the cards:
"Conflicting values recorded — [Card A] says X; [Card B] says Y (needs human resolution)." This applies
to content fields; for a property field that can only hold one value (like `Go-live Date`), use the
primary card's value and note the conflicting related-card value in the nearest content section
instead of silently dropping it.

**Linear tickets and projects stay links, not content.** For every card in the family whose `Linear `
property is set, resolve the ticket's title and URL (and its parent project's, if it belongs to one)
just far enough to build a link — do not read the ticket, project description, or comments for facts
to fill brief fields with. The brief is deliberately scoped to what marketing, support, and sales need;
engineering-level detail belongs in Linear itself, one click away, not summarized into the brief. Not
every card has a Linear link, and its absence is normal, not a gap to flag.

For any content field with no direct source on a family card, write **"Not available — [who should
confirm]"** rather than leaving it blank, guessing, or reaching into Linear for it. This turns the
brief into an actionable checklist instead of a document with silent gaps.

### Step 4b — Draft "What it does / Problem it solves" as marketing copy

This is the one field this skill writes beyond straight extraction — draft it whenever there's enough
recorded material to work from, not only when the user asks. This does not extend to Messaging /
Positioning or either FAQ section, which always stay "Not available" for a human, even if asked.

**Whether there's enough to draft from:** check the confirmed release family's "What will we try?" /
"Why do we believe this?" sections (and the populated "What did we build" content) for actual
substance — a sentence or more describing what's being built and why, not just a card title with
nothing underneath. Enough → draft per below. Not enough → leave "Not available — required human
input" as usual; the user can still ask for a draft later once more is recorded, or explicitly request
one now even if this check says it's thin (they may know context the card doesn't fully capture — in
that case, note in the draft that source material was limited).

**How to draft it:**
1. Load the `calcs-voice-tone` skill (and any active channel sub-skill) before drafting.
2. Source the draft only from facts already recorded across the confirmed release family (Step 3).
   Do not pull from a linked Linear ticket or project (Step 4 links them, it doesn't read them), and
   do not invent outcomes, numbers, or customer reactions that aren't recorded.
3. Declare the audience assumption per the voice-tone skill's rules (e.g. "Writing for API partners"),
   using the `Customer` field(s) recorded across the family to infer who the copy is for.
4. Keep it to 1-3 short sentences: outcome first, mechanism second, calmly confident, no banned
   words/em-dashes/exclamation marks, proof flagged as [PROOF NEEDED] if referenced but not recorded.
5. Lead the section with the marker **"Drafted by Claude — validation required"** on its own line,
   directly above the drafted copy — never present a drafted line as if it were a fact taken straight
   from a card. The marker travels with the text: it stays in place through Step 5's review and into
   the written brief, so anyone reading the entry later sees it hasn't been signed off by a human yet.
6. This still goes through Step 5 like every other field — the user sees it in the full brief before
   anything is written and can edit or replace it there. A separate approval gate beyond that isn't
   needed, since the marker itself signals "unvalidated." If the user removes the marker during review,
   that is their signal they've validated it. This step never applies to an entry whose `Status` is
   `Published` (see Step 2) — that entry is not editable through this skill at all.

### Step 5 — Confirm before writing

Show the full drafted brief to the user — properties and the content sections, in the fixed order,
including which cards fed the release family, any flagged conflicts from Step 4, and any Step 4b
draft still carrying its "Drafted by Claude — validation required" marker — before writing anything
to Notion. Wait for explicit confirmation (or edits) — never write on the first pass. Confirmation
here still results in `Status: Draft` at creation (see Step 4) — confirming the content is accurate
is not the same as marking the brief `Ready`, and a confirmed brief can still carry an unvalidated
Step 4b draft into Notion, marker intact.

### Step 6 — Write the brief

Once confirmed:
1. If no entry exists yet, create a new page in `<PRODUCT_RELEASE_DB_ID>`
   (Product Release templates) via `notion-create-pages`, setting the six properties (`Title`,
   `Category/Area`, `Release Size`, `Go-live Date`, `Channel`, `Status: Draft`) plus `Source Card`,
   and the page `content` (the `##`-headed sections from Step 4, in order) in the same call. Setting
   `Source Card` to every confirmed release-family card's URL automatically populates the reciprocal
   `Release Brief` property on each of those cards — no separate write to any card is needed for the
   link itself.
2. If updating an existing entry, use `notion-update-page` on that entry's page id — `update_content`
   or `replace_content` for section edits (use `replace_content` when reordering/merging sections, as
   `update_content`'s exact-match search-and-replace is brittle against Notion's own markdown
   re-rendering), `update_properties` for the tracked properties, including adding any newly-confirmed
   family cards to `Source Card`. Leave `Status` as-is unless asked.
   **Never reach this step at all for an entry whose `Status` is `Published`** — Step 2 stops the
   flow before any write is considered.
3. Share all the URLs back to the user: the new/updated Product Release templates entry, and every
   source card in the confirmed family (each of which now shows the `Release Brief` relation).

Do not write brief content as a section/table on any source card itself, do not add any of the
content-only fields back as data source properties, do not add a Region field anywhere, do not
merge Internal FAQ and External FAQ back into one heading, do not split "Who asked for this /
Availability" back into two headings or revert its name, do not create a new entry with any `Status`
other than `Draft`, and do not edit a `Published` entry under any circumstance — all of these were
deliberately removed, merged, split, renamed, fixed, or locked.

### Step 7 — Optional slide deck (major releases only)

If the brief's `Release Size` is "Major (marketing effort)" or "Major (capabilities)," offer (via
AskUserQuestion) to also render it as a short slide deck — a genuine alternative to reading the
written brief for people who prefer that format, not a condensed teaser. Skip this step without
asking for minor releases.

If the user wants it, invoke the `pptx` skill to build a 4-5 slide deck, grouped and ordered the
same way the brief itself is grouped:
1. **What it does & why** — What it does / Problem it solves, Messaging / Positioning.
2. **Who asked for this & availability** — Who asked for this / Availability, Category/Area, Channel.
3. **What we built & timing** — What did we build, Release Size, Go-live Date (confirmed), Status.
4. **Support** — Screenshots/Video, Support Articles, Internal FAQ, External FAQ.
5. **Alignment** — Co-marketing/White-label.

Rules for the deck:
- Use **only** the fields already confirmed and written in Step 6 (properties and content sections
  alike). Never add new content, examples, or phrasing that wasn't already in the brief — this is a
  reformat, not a rewrite.
- The deck should carry the same information as the database entry, not a summary of it. If a slide
  would otherwise be empty (e.g. every field in a group is "Not available"), keep the slide and show
  that plainly rather than dropping it — a missing slide reads as "no information," not "not needed."
- If a field is "Not available," show it as such on the slide (with who owns filling it in) rather
  than omitting it silently or inventing a placeholder.
- This is a rendering step, not a second authoring pass — if the brief changes later, the deck is
  regenerated from it, never edited separately in a way that lets it drift out of sync. A `Published`
  entry can still be rendered as a read-only slide deck (rendering isn't editing) — this restriction
  is about writing to Notion, not about presenting what already exists.

---

## Rules

- Never infer or estimate a field's value from context, tone, or general knowledge of the release —
  only from what's actually recorded on a card in the confirmed release family (Step 3). Mark unknowns
  "Not available" instead. The single content-drafting exception is Step 4b's marketing-copy draft for
  "What it does / problem it solves," which is still sourced only from recorded facts and must always
  carry the "Drafted by Claude — validation required" marker. Messaging / Positioning and both FAQ
  sections are never drafted by this skill, even on request.
- Never present Step 4b's drafted copy as a confirmed fact. It is always labeled "Drafted by Claude —
  validation required," and that label stays with the text through Step 5 and into the written brief.
- Never read a linked Linear ticket or project for content — add it as a plain reference link under
  "What did we build" (see Step 4) and stop there. The brief stays customer/marketing-facing;
  engineering detail stays in Linear.
- Every "What did we build" bullet links its card title back to that card's own URL, so a reader can
  click through to any item's full detail rather than the brief carrying it inline. This applies
  whether the family is one card or many.
- Screenshots/video, Support articles, Internal FAQ, and External FAQ default to "Not available," but
  if a family card already links one of these directly (a screenshot, a video, a help-center article,
  an FAQ doc), link it rather than mark "Not available" — link-only, same as Linear: never draft,
  describe, or go looking beyond what a card already links.
- Never merge a related card's content into a brief without the user confirming the matched set in
  Step 3 — a Parent item/Sub-item relation or a Workstream+Segment tag match is a candidate, not an
  automatic inclusion.
- Never silently resolve a conflict between two cards in the release family. Flag it inline, naming
  both cards, and leave the resolution to a human.
- Never create or update a Product Release templates entry without explicit user confirmation of the
  drafted content first.
- Never skip the dedup check in Step 2, even if the user seems confident there's no existing brief.
- **Never edit an entry whose `Status` is `Published`, under any circumstance, even if the user
  insists or claims to be the owner.** Tell them it's published and can't be changed through this
  skill, and stop. This is a hard block, not advisory.
- **When an existing entry's `Status` is `Ready`,** this is advisory, not a hard block: tell the user
  updates have already been made and recommend checking with the owner or marketing first, but still
  allow the update to proceed if the user confirms they want to continue.
- Always surface an existing entry's `Status` to the user as part of showing them the entry in
  Step 2 — never silently proceed without stating it.
- Never write brief content back onto any source card as a page section — a card only ever carries
  the automatic `Release Brief` relation to its entry in Product Release templates.
- Never add narrative fields back as data source properties. They live in page content under `##`
  headings. Only Title, Category/Area, Release Size, Go-live Date, Source Card, Channel, and Status
  are properties.
- Never include a Region field, in properties or content. It was deliberately removed as not
  relevant to Calcs.com releases.
- Never split "Who asked for this / Availability" into two separate headings, and never rename it.
- Never merge "Internal FAQ (support/sales)" and "External FAQ (customer-facing)" back into a single
  "FAQ" heading — they serve different audiences and were deliberately split.
- Content sections always appear in the fixed order: What it does / Problem it solves; Messaging /
  Positioning; Who asked for this / Availability; What did we build; Screenshots/video; Support
  articles; Internal FAQ; External FAQ; Co-marketing/white-label.
- **`Status` always starts at `Draft`** for any newly created entry, no matter how complete the
  brief looks — this skill never sets `Ready` or `Published` itself. Those transitions happen later,
  driven by a human, outside this skill's normal write path. Updates to an existing `Draft` or
  `Ready` entry leave `Status` untouched unless the user explicitly asks to change it.
- Publishing to any public-facing release-notes system is explicitly out of scope for this skill —
  it stops at the internal brief.
- The Step 7 deck is offered, never automatic, and only for releases marked Major. It renders the
  already-confirmed brief content in full — it must never introduce a fact, phrase, or estimate that
  wasn't already written into the brief in Step 6, and must never drop a section just because that
  section is mostly "Not available."

---

## External system rendering

- **Target:** Notion, data source `<PRODUCT_RELEASE_DB_ID>` ("Product
  Release templates"), at <PRODUCT_RELEASE_DB_URL>.
  Properties: `Title` (title), `Category/Area` (text), `Release Size` (select: Major (marketing
  effort) / Minor / Bundled / Major (capabilities)), `Go-live Date` (date), `Channel` (multi-select:
  Email, In-app, Website, Ads, Sales, Product, Content/SEO, Social, Other, Events — mirrors the
  source card's option set), `Status` (select: Draft / Ready / Published — new entries always start
  `Draft`; `Published` entries are locked against edits via this skill; `Ready` entries get an
  advisory warning), `Source Card` (two-way relation to
  `<CROSS_TEAM_DB_ID>`, reciprocal property "Release Brief" — may hold
  more than one card when the release family has more than one confirmed member, see Step 3). Page
  content, in fixed order: What it does / problem it solves; Messaging / Positioning; Who asked for
  this / Availability (merged); What did we build; Screenshots/video; Support articles; Internal FAQ
  (support/sales); External FAQ (customer-facing); Co-marketing/white-label. No Region property and
  no Region section.
- **Source database:** `<CROSS_TEAM_DB_ID>` ("Cross-Team Work and
  Learning"). Do not attempt to set `Days in stage` or `Cycle days` on it — both are read-only
  formula properties and will reject writes. Related-card matching (Step 3) reads `Parent item`,
  `Sub-item`, `Workstream`, and `Segment / Product Layer` on this same data source.
- **Linear:** for a card's linked ticket, and its parent project when it has one — resolved only far
  enough to link (title + URL) under "What did we build" (see Step 4). Never read for content; never
  used for Messaging / Positioning or either FAQ section.
- **Step 7 output:** a 4-5 slide `.pptx` deck, produced by delegating to the `pptx` skill. Content
  is fixed to the confirmed brief fields (see Step 7's slide grouping) — the `pptx` skill handles
  layout/formatting only, it does not originate content. Rendering a `Published` entry to a deck is
  fine; writing changes to it through this skill is not.

---

## Required permissions

```json
{
  "permissions": {
    "allow": [
      "notion-fetch",
      "notion-query-data-sources(<CROSS_TEAM_DB_ID>)",
      "notion-query-data-sources(<PRODUCT_RELEASE_DB_ID>)",
      "notion-create-pages(<PRODUCT_RELEASE_DB_ID>)",
      "notion-update-page(<PRODUCT_RELEASE_DB_ID>)",
      "notion-update-page(<CROSS_TEAM_DB_ID>)",
      "mcp__Linear__get_issue",
      "mcp__Linear__get_project"
    ]
  }
}
```
