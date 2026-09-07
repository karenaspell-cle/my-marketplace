---
name: meeting-recap
description: Turn raw meeting notes, a transcript, or a stream-of-consciousness recap into a structured summary with decisions, action items (owner + due date if stated), and open questions. Use this whenever the user pastes meeting notes and asks for a recap, summary, or "turn this into action items," even if they don't say the word "skill."
---

# Meeting Recap

Given raw notes or a transcript, produce three sections, in this order:

## Decisions
One line each. Only include things that were actually decided — not proposed, not discussed. If nothing was decided, write "No decisions recorded."

## Open Questions
Things raised but not resolved. Skip this section entirely if there aren't any — don't pad it.

## Action Items
Format: `- [ ] <task> — <owner> (<due date if stated>)`. If no owner was stated, write "unassigned" rather than guessing who it might be.

Keep the whole thing shorter than the source material. Don't editorialize or add commentary the notes didn't contain.
