# Gmail search patterns for job-tracker skills

These patterns were battle-tested on a real job search and cover the five signal types a tracker needs. `SINCE` = `lastSync` minus 3 days (safety overlap for late-arriving mail), formatted `YYYY/MM/DD`.

**These are English phrases.** Every quoted string below ("thank you for applying," "unfortunately," etc.) only matches English-language mail. If a user's job-search correspondence arrives in another language, these patterns will miss most of it — translate the equivalent phrases for that language before relying on this reference, and say so to the user rather than quietly under-reporting their pipeline.

A critical tool behavior to know: `search_threads` returns only a partial message list per thread (often just the first ~5), so a thread can look stale when it actually has newer messages. Whenever a thread is relevant to a pipeline role, call `get_thread` with `messageFormat MINIMAL` to enumerate every message and its date before drawing conclusions or writing a note. Reach for `FULL_CONTENT` only when a snippet genuinely isn't enough — it can be very large, so prefer `MINIMAL` by default.

Search the entire account, not just the inbox: `in:anywhere` covers Trash and Spam too, and don't forget Sent — recruiters' replies often live in a Sent thread's conversation rather than a fresh inbound email.

## 1. New applications

```
to:<user's address> after:SINCE ("thank you for applying" OR "application received" OR "we received your application" OR "thanks for applying" OR "your application" OR "thank you for your interest")
```
Extract company + role from the subject/snippet. If company+role isn't already a row (case-insensitive match, ignoring trailing job numbers or brackets like `[19249]` or `(200035972)`), append a new row: status `Applied`, date = email date, notes `""`.

## 2. Rejections (including deleted mail)

```
in:anywhere after:SINCE ("unfortunately" OR "not be moving forward" OR "not moving forward" OR "other candidates" OR "decided not to" OR "position has been filled" OR "role has been filled" OR "will not be progressing" OR "pursue other candidates" OR "not to move forward")
```
Also check any job-search label the user has, since rejections sometimes get auto-labeled and archived:
```
label:<label id> after:SINCE unfortunately
```
Match to a pipeline row by company (and role keywords if the company has multiple open rows). If the matching row is `Applied` or `Interviewing`, move it to `Rejected`. This is a one-way street — once rejected from real evidence, a row never moves back.

## 3. Interview / progress

```
in:anywhere after:SINCE ("schedule your interview" OR "schedule an interview" OR "set up some time" OR "30 minute interview" OR "invite you to interview" OR "phone screen" OR "recruiter screen" OR "initial call" OR "like to set up" OR "move forward to the interview")
```
If the matching row is `Applied`, move it to `Interviewing`. Never downgrade an `Offer` row based on this signal.

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

- Genuine two-way dialogue with a recruiter or hiring manager on an `Applied` row → move to `Interviewing`.
- A one-way cold outreach with no reply stays `Applied` — but it still deserves a note (see below), since "I emailed the hiring manager and haven't heard back" is meaningfully different from having done nothing.

## Recruiter/HM notes (applies across all signal types)

For every pipeline company with any human correspondence — recruiter or hiring manager, inbound or the user's own replies — open the full thread and read the latest messages. Summarize the newest development in a short, specific phrase: names, roles, dates, and what happens next. Vague notes like "recruiter followed up" are much less useful later than "Zoe Lloyd (Intuit TA) reconnected with HM Diego on 07/08, more openings likely ~September."

Before appending, always check whether that same development is already captured (same date, same fact) — this logic runs repeatedly over time, and duplicate notes make the tracker noisy and eventually untrustworthy.
