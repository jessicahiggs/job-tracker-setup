# Status and notes rules

These rules exist because a tracker that quietly loses information, contradicts itself, or regresses status is worse than no tracker at all — the whole point is that the user can trust it without re-checking their inbox.

## Status progression

Pick a small, ordered set of statuses (the default is `Applied < Interviewing < Offer`, with `Rejected` as a separate terminal state reachable from any of the three). A status only ever moves forward along that order:

- Only change a status when an email clearly supports the change — never infer a status from silence or absence of a reply.
- `Rejected` is terminal once set from a real rejection signal. Don't move a rejected row back to `Applied` or `Interviewing` even if new correspondence appears (e.g. a recruiter reaching out again about a *different* role at the same company should become a new row, not resurrect the old one).
- Don't downgrade `Offer` based on interview-stage language — an offer email might still contain phrases like "we'd like to schedule a follow-up call," which shouldn't undo the offer.

## Notes

- Never delete or overwrite existing note text — treat it as an append-only log.
- Append new developments in a compact, dated format: `" | [MM/DD] <specific summary>"`. Specific means: names, roles, dates, and what happens next — not "had a call" but "30-min call with Priya (TA) booked for 7/22, next round is a panel."
- Check existing notes for the same `[MM/DD]` tag (or the same underlying fact under a different date) before appending, to avoid duplicating something already captured — this matters most when a sync runs more than once in the same day or the scan window overlaps a previous run.
- Only genuine human correspondence earns a note. Skip generic auto-replies ("Thanks, we've received your message") — they're not informative and clutter the tracker.
- When a company has multiple open roles and it's unclear which one a note belongs to, prefer role keywords from the email; if still ambiguous, attach to whichever row is furthest along (`Interviewing` before `Applied`), and failing that, the most recently touched row.

## Row matching

- Match company + role case-insensitively, and ignore trailing job/requisition numbers or bracketed codes (e.g. treat "Principal PM [19249]" and "Principal PM" as the same role if company matches).
- Never fabricate a row. If an email is ambiguous about which company or role it refers to, don't guess — leave it out and flag the ambiguity to the user rather than silently placing it somewhere it might not belong.
