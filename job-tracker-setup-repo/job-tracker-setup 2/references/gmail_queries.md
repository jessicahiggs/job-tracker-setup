# Gmail search patterns for job-tracker skills

These patterns were battle-tested on a real job search and cover the signal types a tracker needs. `SINCE` = `lastSync` minus 3 days (safety overlap for late-arriving mail), formatted `YYYY/MM/DD`.

**These are English phrases.** Every quoted string below ("thank you for applying," "unfortunately," etc.) only matches English-language mail. If a user's job-search correspondence arrives in another language, these patterns will miss most of it — translate the equivalent phrases for that language before relying on this reference, and say so to the user rather than quietly under-reporting their pipeline.

**Dates are the user's local dates.** Gmail timestamps can be UTC. An email stamped just after midnight UTC is usually the *previous evening* in the user's timezone (e.g. US Pacific). Convert before you record a `date` or decide what counts as "today," or the batch of applications you added tonight will show up on the wrong day.

**`search_threads` returns only a partial message list per thread** (often just the first ~5 messages), so a thread can look stale — or look like a single event — when it actually holds several newer or distinct messages. Whenever a thread is relevant to a role, call `get_thread` with `messageFormat MINIMAL` to enumerate every message and its date, and read each message's body for its own facts before drawing conclusions or writing a note. Reach for `FULL_CONTENT` only when a snippet genuinely isn't enough — it can be very large, so prefer `MINIMAL` by default.

**Search the entire account, not just the inbox:** `in:anywhere` covers Trash and Spam too (people delete rejections — you still want them), and don't forget Sent — recruiters' replies often live in a Sent thread's conversation rather than a fresh inbound email.

---

## 1. New applications — cast a WIDE net

Confirmation wording varies enormously; **do not rely on a single phrase.** In practice, matching on only "thank you for applying" silently drops applications worded "thank you for your application" or "we have received your application." Run BOTH of these:

```
to:<user's address> after:SINCE (application OR applying OR applied OR "your interest")
```
```
label:<job-search label id> after:SINCE
```

Then treat a message as an **application confirmation** when it is an automated careers/ATS email **and** its subject or body says an application was submitted or received.

- **Sender heuristic** (any of): `no-reply@` / `donotreply@` / `notification@`, `ashbyhq`, `greenhouse`, `lever`, `smartrecruiters`, `icims`, `workday` / `myworkday`, `eightfold`, `talent.*`, `*careers*`, or a plausible company `no-reply` address.
- **Phrase list** (any of): "thank you for applying", "thanks for applying", "thank you for your application", "we have received your application", "we received your application", "we have received your", "application received", "thank you for your interest", "we are lucky that you applied", "thank you for submitting".

For each confirmation, **read the message body to get the exact role** (subject lines are often generic like "we have received your application"). If the role truly isn't stated anywhere, record it as `(role not named in confirmation email)` and move on — don't guess.

**Reconcile each confirmation individually:**
- If `company + role` is **not** already a row → append: status `Applied`, date = email's *local* date, notes `""`.
- **Re-application:** if `company + role` **is** already a row and this confirmation is newer than the row's date → update the row's date and append `" | [MM/DD] Re-applied"`. If that row was `Rejected`, set it back to `Applied` (a fresh application to a role you were previously rejected from is a real, new attempt — see `status_rules.md`).
- **Multiple roles, one thread / one company:** a single conversation can contain confirmations for two *different* roles, and one company routinely has several open roles. Match on company **and the specific role**, create one row per distinct role, and **never collapse two different role titles into one row** just because they share a thread or an identical subject line. Companies do not send duplicate confirmations for a single application — if two confirmations name different roles, they are two applications.

List every application email you found and reconcile each one explicitly; skipping any is a failure. If a user later says an application is missing, widen the net (new sender or phrase) rather than defending the matcher.

## 2. Rejections (including deleted mail)

```
in:anywhere after:SINCE ("unfortunately" OR "not be moving forward" OR "not moving forward" OR "other candidates" OR "decided not to" OR "position has been filled" OR "role has been filled" OR "will not be progressing" OR "pursue other candidates" OR "not to move forward")
```
Also check any job-search label the user has, since rejections sometimes get auto-labeled and archived:
```
label:<label id> after:SINCE unfortunately
```
Match to a pipeline row by company **and role** (a company often has multiple open rows — don't reject the wrong one). If the matching row is `Applied`, `Warm`, or `Interviewing`, move it to `Rejected`. This is a one-way street from real evidence — but if the same email also offers to refer the person elsewhere or keep them for future roles, prefer `Warm` (see `status_rules.md`).

## 3. Interview / progress

```
in:anywhere after:SINCE ("schedule your interview" OR "schedule an interview" OR "set up some time" OR "30 minute interview" OR "invite you to interview" OR "phone screen" OR "recruiter screen" OR "initial call" OR "like to set up" OR "move forward to the interview")
```
Promote **only** a currently-`Applied` row → `Interviewing`, and **only if the email is newer than that row's date** (an old interview email for a role that has since gone cold should not re-promote it). **Never** promote a `Warm` row off this signal — `Warm` is sticky (see `status_rules.md`). Never downgrade an `Offer`.

## 4. Offers

```
in:anywhere after:SINCE ("pleased to offer" OR "offer letter" OR "extend an offer" OR "we would like to offer" OR "we'd like to offer")
```
Matching row → `Offer`, regardless of its prior status (short of already being `Rejected`, which shouldn't happen for a real offer — if you see both signals on the same thread, trust the more recent message).

## 5. Sent mail & active dialogue

```
in:sent after:SINCE from:<user's address>
```
For each sent message, check the recipient: skip anything to an ATS or no-reply address (`ashbyhq`, `greenhouse`, `smartrecruiters`, `lever`, `donotreply`, `no-reply`, and similar automated domains). For a sent message to a **named human** whose domain matches a pipeline company, open the full thread (`get_thread`, `MINIMAL`) and read both sides.

- Genuine two-way dialogue with a recruiter or hiring manager on an `Applied` row → move to `Interviewing` (or `Warm` if it's clearly future-roles nurturing rather than a live process).
- A one-way cold outreach with no reply stays `Applied` — but it still deserves a note, since "I emailed the hiring manager and haven't heard back" is meaningfully different from having done nothing.

## 6. Referrals

```
in:anywhere after:SINCE ("has referred you" OR "you've been referred" OR "referred you for" OR "accepted your referral" OR "referral")
```
These are often automated (e.g. a company "talent hub" that emails "MC <Name> has referred you for a position"). They're automated, but the *fact* they carry — who referred the user, for which role — is useful. Extract the referrer's name and the role, match the pipeline row, and if it has no referral note yet, append `" | [MM/DD] Referral by <Name>"` (or set the note if empty). Don't duplicate a referral note that's already there. Don't treat a referral notification as a rejection or a status change on its own.

## Recruiter/HM notes (applies across all signal types)

For every pipeline company with any human correspondence — recruiter or hiring manager, inbound or the user's own replies — open the full thread and read the latest messages. Summarize the newest development in a short, specific phrase: names, roles, dates, and what happens next. Vague notes like "recruiter followed up" are much less useful later than "Zoe Lloyd (Intuit TA) reconnected with HM Diego on 07/08, resume sent to two other teams, more openings likely ~September."

Before appending, always check whether that same development is already captured (same date, same fact) — this logic runs repeatedly over time, and duplicate notes make the tracker noisy and eventually untrustworthy.
