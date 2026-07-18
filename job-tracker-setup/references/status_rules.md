# Status and notes rules

These rules exist because a tracker that quietly loses information, contradicts itself, or regresses status is worse than no tracker at all — the whole point is that the user can trust it without re-checking their inbox.

## Status set and progression

Default set (ordered): `Applied < Warm < Interviewing < Offer`, with `Rejected` as a separate terminal state reachable from any earlier state. Keep the set small (≤5) so "never regress" stays easy to reason about.

**What each means:**
- `Applied` — application submitted/received; no human contact yet.
- `Warm` — this specific role didn't proceed **or** there's no live interview, **but** a recruiter/HM is actively keeping the person warm: referring them to other roles, passing their resume to other hiring managers, or holding them for upcoming openings. This is a real state that otherwise gets mislabeled as `Rejected` (too pessimistic) or `Interviewing` (too optimistic). If you don't use `Warm`, fold these cases into whichever of your statuses is the least wrong and say so.
- `Interviewing` — a live interview/screen is scheduled or underway for this role.
- `Offer` — an offer was extended.
- `Rejected` — a genuine dead-end for this role, with no ongoing advocacy.

**Progression rules:**
- Only change a status when an email clearly supports the change — never infer a status from silence or absence of a reply.
- Never regress along the order. `Offer` is not downgraded by interview-stage language (an offer email may still say "let's schedule a follow-up call").
- `Rejected` is terminal **for that application**. Don't move it back on the strength of ordinary new correspondence. Two documented exceptions:
  1. A recruiter reaching out about a **different** role at the same company → a **new row**, not a resurrection of the old one.
  2. A genuine **re-application** to the *same* role (a fresh "application received" confirmation dated after the rejection) → set the row back to `Applied` and note `" | [MM/DD] Re-applied"`. Re-applying is a real, deliberate new attempt; reflect it.

## Warm is sticky

Once a row is `Warm`, do **not** bump it to `Interviewing` on the strength of an *old* email (e.g. a stale "schedule your interview" message from before the role went cold). Only move a `Warm` row when a genuinely **new** email — dated after the row's date and after the last sync — clearly shows, for **that exact role**, a live interview (→`Interviewing`), an offer (→`Offer`), or an explicit dead-end with no further advocacy (→`Rejected`). This prevents an automated sync from silently undoing a deliberate `Warm` classification.

## Notes

- Never delete or overwrite existing note text — treat it as an append-only log. This especially includes the user's own manual notes (referrals, context, reminders); a sync must preserve them verbatim and only *append*.
- Append new developments in a compact, dated format: `" | [MM/DD] <specific summary>"`. Specific means: names, roles, dates, and what happens next — not "had a call" but "30-min call with Priya (TA) booked for 7/22, next round is a panel."
- Check existing notes for the same `[MM/DD]` tag (or the same underlying fact under a different date) before appending, to avoid duplicating something already captured — this matters most when a sync runs more than once in the same day or the scan window overlaps a previous run.
- Only genuine human correspondence (or a referral, per `gmail_queries.md` §6) earns a note. Skip generic auto-replies ("Thanks, we've received your message") — they're not informative and clutter the tracker.
- When a company has multiple open roles and it's unclear which one a note belongs to, prefer role keywords from the email; if still ambiguous, attach to whichever row is furthest along (`Interviewing`/`Warm` before `Applied`), and failing that, the most recently touched row.

## Row matching (read this carefully — it's where the naive version broke)

- Match on **company + role**. Ignore trailing job/requisition numbers or bracketed codes for the *same* title (e.g. treat "Principal PM [19249]" and "Principal PM" as the same role if the company matches).
- **But descriptive role text is significant.** "Sr Lead Product Manager - Risk Decisioning Platform" and "Sr Lead Product Manager" are **different roles** — different applications — even at the same company, even if the confirmation emails share a subject line or a thread. Only treat two confirmations as the same application when the meaningful role title is the same; a differing descriptive suffix means a different role.
- **One thread can hold multiple distinct applications.** Do not let email grouping decide the count. Open the thread, read each message's body, and reconcile each distinct role separately.
- **Companies do not send duplicate confirmations for one application.** So if you're tempted to dismiss a second confirmation as a "duplicate," check the role first — it's much more likely a second application. When in doubt, keep them separate and let the user merge, rather than silently collapsing.
- Never fabricate a row. If an email is genuinely ambiguous about which company or role it refers to, don't guess — leave it out and flag the ambiguity to the user rather than silently placing it somewhere it might not belong.
