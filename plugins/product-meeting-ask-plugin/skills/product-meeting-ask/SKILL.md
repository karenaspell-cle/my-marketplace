---
name: product-meeting-ask
description: >
  Handles the Sales → Product Meeting Requests workflow end to end: (1) SUBMIT mode — a
  sales/CS/AM rep describes an upcoming customer meeting and what they need from product/dev,
  and this skill turns that into a properly-formed entry in the Sales → Product Meeting
  Requests Notion database; (2) TRIAGE mode — a product/dev owner processes the queue of open
  requests, checks the 7-day notice flag, and drafts a starting response grounded in real
  research via research-repository. Use for "/product-meeting-ask", "I need something for my
  [customer] meeting", "customer wants a roadmap/status update", "submit a sales request",
  "what's in the request queue", "triage the sales requests", or "handle the product asks from
  sales". NOT for general research with no meeting attached (use research-repository directly),
  NOT for client-facing Atlas slides (use atlas-session-slides).
---

# Sales Request Intake

Runs the Sales → Product Meeting Requests process from either side: the person asking
(SUBMIT) or the person answering (TRIAGE). Both modes write to the same Notion database.

**Slash command:** `/product-meeting-ask` — triggers this skill regardless of which mode is
needed; ask which mode if it isn't clear from what follows the command.

**Workspace details:** see `references/workspace-config.md` for the database URL, data
source ID, and Slack channel for this install. Read it before Step 5 (SUBMIT) or Step 1
(TRIAGE) if you haven't already this session.

---

## Which mode?

- Someone describing an upcoming customer meeting and asking for material/status/roadmap input
  → **SUBMIT**
- Someone asking to process, review, or answer what's sitting in the queue → **TRIAGE**
- Unclear → ask which one before proceeding.

---

## SUBMIT mode

### Step 1 — Extract what's already there

Read the person's message for what it already tells you: customer, meeting date, requestor
(usually the person themselves), what they're asking for. Don't re-ask for anything already
stated.

### Step 2 — Push on "The Ask" specifically, and keep pushing until it's actually there

This is the field that determines whether the request is usable without a follow-up. If what
they've given you is "give me a roadmap update" or "customer wants to know what's coming" —
that's not enough. Ask them directly: *what exact question is the customer asking, and why
does it matter for this meeting?*

**One round of clarification is not automatically enough.** Check the answer you get back
against two things, separately:
- Is the question itself concrete? "A specific feature" is not concrete — it needs to name the
  feature. "Wants to know what's coming" answered with "wants to know if X ships" is concrete.
- Is the "why it matters" actually present? An answer that describes the *feature's status*
  (shipping, in progress, uncertain) is not the same as answering *why the customer cares* —
  don't let a status update substitute for the stakes.

If either piece is still missing or still generic, ask again, specifically for the piece that's
missing — don't move on because *an* answer arrived, and don't let a multiple-choice-shaped
follow-up (yours or `ask_user_input_v0`'s) quietly narrow what counts as answered. Only proceed
once the ask names the actual thing and the actual stakes. Don't submit a vague ask hoping
product will figure it out; that's the exact back-and-forth this system exists to prevent.

### Step 3 — Fill in the rest, and don't skip fields to move faster

Gather, in one pass if possible (use `ask_user_input_v0` for the missing pieces rather than
serial questions):
- Meeting date
- Date submitted (default: today, unless they say otherwise)
- Who's in the room (Technical / Exec sponsor / Mixed)
- Prior commitments (a real "none" is fine — don't leave it blank without asking)

All four are required before creating the page. If the conversation moves quickly and one gets
missed, go back for it rather than submitting with a gap — a fast submission with a blank field
just relocates the back-and-forth to whoever triages it later.

### Step 4 — Surface the notice flag before submitting, not after

Check today's date (or stated submission date) against the meeting date. If it's under 7 days,
tell the person up front: *"Heads up — this is inside the 7-day window, so it'll be flagged
best-effort. Still going in, just so you know product may need to simplify scope given the
timeline."* Don't let them find out after the fact.

### Step 5 — Create the page and confirm

Create the page in the data source named in `references/workspace-config.md`. Title format:
`"[Customer] - [Meeting date] [short ask type]"` (e.g. "Acme - Aug 12 roadmap ask"). Confirm
the fields with the person before writing if anything was inferred rather than stated;
otherwise just create it and share the link back.

### Step 6 — Post a Slack notification

After the page is created, post a per-submission notification to the channel named in
`references/workspace-config.md`:

```
📥 New product request submitted

Customer: [Customer / Account]
Meeting: [Meeting Date] ([X] days out — on time / ⚠️ best-effort)
Requestor: [Requestor]
The ask: [The Ask]
Who's in the room: [Who's in the Room]
→ [Notion page link]
```

If the request is best-effort, keep the ⚠️ marker in the "Meeting" line so it's visible without
opening the link.

---

## TRIAGE mode

### Step 1 — Query the queue

Query the data source named in `references/workspace-config.md` for Status = "Not started"
(and "In progress" if the person wants a full picture, not just new items).

### Step 2 — Surface the notice flag per item

For each request, check the Notice Flag. Lead with it if best-effort — the owner should know
the timeline constraint before deciding how much to invest in the response.

### Step 3 — Draft a starting response using research-repository

For each request, use the `research-repository` skill to pull what's actually known relevant to
"The Ask" — don't answer from memory. Ground the draft in Dovetail/Notion/Linear (and Fellow/
Slack if opted in per that skill's own rules).

Classify what you find the same way research-repository does — **Strong / Thin / None** — and
say so explicitly in the draft. A confident-sounding answer built on a thin research base is
worse than a short answer that states the gap plainly. If it's None, say what's needed to
answer it (e.g. "needs a check with engineering on X") rather than guessing.

### Step 4 — Confirm before writing back to Notion

Show the drafted response to the owner before updating the page. On confirmation:
- Set **Owner** to the person handling it (needs a real Notion user — look them up if unsure,
  don't guess)
- Update **Status** (In progress while drafting, Done once delivered)
- Optionally append the response into the page content so it's linked to the original ask

Never silently mark something Done without the owner confirming the response is actually what's
going out.

---

## Rules

- "The Ask" must always be a real question, not a category. If a submission arrives from
  elsewhere (e.g. someone pastes an ask directly into Notion) and it reads like "give me an
  update," flag it back to the requestor rather than triaging it as-is.
- An answer arriving is not the same as the gap being closed. Before creating the page, check
  each required field against what it's actually supposed to capture (the concrete question,
  the real "why," who's in the room, prior commitments) — not just whether some text landed in
  it. Re-ask the specific missing piece rather than accepting an adjacent answer or moving on
  because the conversation is flowing quickly.
- The 7-day rule is a **soft flag, not a hard block** — never refuse to submit or process a
  late request. Just make the tradeoff visible to both sides.
- Never fabricate roadmap, timeline, or status detail to fill a Thin/None research gap. State
  the gap.
- Confirm before writing to Notion in both modes — this is a shared team database, not a
  private scratchpad.
- Re-fetch the data source schema before writing if it's been more than a few turns since you
  last checked it; the schema can change independently of this skill.

---

## Required permissions

```json
{
  "permissions": {
    "allow": [
      "mcp__notion__notion-search",
      "mcp__notion__notion-fetch",
      "mcp__notion__notion-create-pages",
      "mcp__notion__notion-update-page",
      "mcp__notion__notion-query-data-sources",
      "mcp__slack__slack_send_message"
    ]
  }
}
```
