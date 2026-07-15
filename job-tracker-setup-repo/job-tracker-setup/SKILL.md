---
name: job-tracker-setup
description: Build a brand-new, self-updating "Job Application Tracker" artifact for someone's own job search by scanning their Gmail for application confirmations, rejections, interview invites, and offers. Use this whenever a user wants a live dashboard of their job search built from their inbox — for example "build me a job application tracker from my email", "I want a live dashboard of everything I've applied to", "scan my Gmail for job rejections and interviews and make me something like a spreadsheet but automatic", "set up a tracker like Jessica's for my own search", or "track my applications automatically instead of a manual spreadsheet." Runs a short interview to collect the Gmail account/label to scan and any seed spreadsheet, then builds and maintains a live artifact using repeatable Gmail-search-and-rebuild logic, with an optional nightly schedule.
metadata:
  author: Jessica Higgs (@jessicahiggs)
  repository: https://github.com/jessicahiggs/job-tracker-setup
---

# Job Tracker Setup (generalized)

Created by [Jessica Higgs](https://github.com/jessicahiggs).

This skill sets up a live, Gmail-driven job-application tracker for anyone — not tied to a specific inbox or company list. It's the reusable version of a workflow originally built for one person's job search (Jessica's), generalized so it can bootstrap a fresh tracker for someone else's inbox and pipeline.

**Requires:** a connected Gmail MCP connector, and Cowork's artifact tool to publish the live dashboard. Optional: a Drive/Sheets connector (only if seeding from an existing spreadsheet) and the `schedule` skill (only if the user wants automatic recurring runs). See Step 0 below — check these before interviewing the user, don't assume they're present.

**Built and tested for English-language inboxes.** The search phrases in `references/gmail_queries.md` ("thank you for applying," "unfortunately," etc.) are English phrases — they won't reliably catch application/rejection/interview emails written in another language. If the user's job-search correspondence is mostly in a different language, say so upfront and either translate the query patterns together before running them, or set expectations that recall will be weaker than what this skill was built for.

Read `references/gmail_queries.md` before running any searches — it has the exact query patterns for each signal type (new application, rejection, interview, offer, active dialogue) along with the reasoning behind each one, so you're not reinventing search strings from scratch. Read `references/status_rules.md` for the status/notes rules that keep the tracker trustworthy over many runs.

## Step 0 — Check prerequisites before promising anything

This skill only works if two things are actually connected: a Gmail MCP connector (to search/read mail) and Cowork's artifact tool (`mcp__cowork__create_artifact` / `update_artifact`, to publish the live dashboard). A Drive/Sheets connector is a third, optional dependency — only needed if the user wants to seed from an existing spreadsheet.

Before you say anything about timelines or start the interview in Step 1, check what's actually available in this session:

1. **Gmail.** Look for a Gmail MCP connector (tool names typically start `mcp__<id>__search_threads` / `get_thread` / etc.). If none is connected, don't proceed as if it exists — tell the user plainly that this needs a Gmail connector, and help them get one: search the connector registry (`mcp__mcp-registry__search_mcp_registry` with keywords like "gmail", "email") and suggest connecting it via `mcp__mcp-registry__suggest_connectors`, or point them to Settings → Connectors if you're not able to search the registry from here. Don't guess at Gmail data or fabricate what you'd find — wait until the connector is actually live.
2. **Cowork artifacts.** Confirm `mcp__cowork__create_artifact` (or `update_artifact`) is available. This is what turns the tracker into something the user can reopen and see refreshed automatically — without it you can still build the underlying data, but you'd be handing back a static file instead of a living dashboard, so say so explicitly rather than silently downgrading the deliverable.
3. **Drive/Sheets (only if they mention a seed spreadsheet).** If the user wants to seed from an existing sheet, check for a Drive/Sheets connector before promising to read it. If it's missing, ask them to export/paste the data instead, or help them connect the right connector.
4. **The `schedule` skill (only if they want automatic recurring runs).** This isn't an MCP connector, just confirm it's available in the skills list before promising a nightly cadence in Step 1's cadence question.

If something essential (Gmail, at minimum) isn't connected, stop here and help the user connect it before running any part of the interview — don't ask all the Step 1 questions first and then discover halfway through that you can't actually act on the answers.

## Step 1 — Interview the user

Before touching Gmail, confirm the essentials (skip any the user has already answered):

1. **Which Gmail account/connector** should be scanned? If more than one Gmail MCP connector is available, ask which one.
2. **Seed data** — do they have an existing spreadsheet, CSV, or list of applications to start from? If yes, get the file (Drive file id, or an uploaded file) and confirm the column mapping (company, role, status, date, notes are the defaults — ask if theirs differs). If no, start from an empty pipeline and build it entirely from scanning inbox history.
3. **Scope** — is there a Gmail label they use for job-search mail (many people don't have one)? If so, get its label id via the Gmail connector's label-listing tool. If not, that's fine — searches just run across the whole mailbox (`in:anywhere`) instead of being scoped to a label.
4. **Status categories** — confirm the default four (`Applied`, `Interviewing`, `Offer`, `Rejected`) work for them, or learn their preferred set. Keep it to a small fixed set; more than 4-5 statuses makes the "never regress" rule hard to reason about.
5. **Cadence** — do they want this to also run automatically (nightly, or some other interval)? If yes, after finishing the initial build, hand off to the `schedule` skill to set up a recurring run of this same workflow. If they only want on-demand runs, skip scheduling.
6. **Where should state live?** You need one JSON file that persists between runs (shape below) — ask where they'd like it saved (a connected folder is ideal so it survives between sessions; otherwise use your own working directory and tell them plainly that it won't persist across sessions unless they have a connected folder).

Don't proceed to Step 2 until you have answers to 1–3 at minimum; the rest can use sensible defaults if the user doesn't care.

## Step 2 — Seed the pipeline

If seeding from a spreadsheet: read it, parse company/role/status/date/notes (skip header/separator rows, unescape any escaped characters), and normalize every status to the agreed set. If a source status doesn't map cleanly (e.g. "Warm", "On hold"), don't silently force it into a bucket — pick the closest fit based on what actually happened (a recruiter call already occurred vs. no contact yet) and say what you chose and why, so the user can correct you.

If starting empty: don't fabricate history. Either start with zero rows and let the first sync populate it from "application received" type emails going forward, or — if the user wants historical data — run the "new applications" search from `references/gmail_queries.md` across their full mail history once, as a one-time backfill.

Write the initial `state.json`:
```json
{"lastSync": "<today's date>", "rows": [{"company": "", "role": "", "status": "", "date": "", "notes": ""}]}
```

## Step 3 — Scan and build

Run the searches and status/notes logic from `references/gmail_queries.md` and `references/status_rules.md` against the window since `lastSync`. This is identical logic whether it's the first run or the hundredth — the only thing that changes is the date window.

## Step 4 — Build the artifact

Use `assets/tracker_template.html` as the base. It expects three substitutions:
- `{{LAST_SYNC}}` → today's date
- `{{RECENT_CHANGES}}` → a JS array literal of change strings (or `[]`)
- `{{DATA}}` → a JS array literal of `{company, role, status, date, notes, changed}` objects

Call `mcp__cowork__create_artifact` (first time) or `mcp__cowork__update_artifact` (subsequent runs) with the built HTML. Pick a clear artifact id, e.g. `<name>-job-tracker`.

## Step 5 — Offer scheduling

If the user wanted a recurring sync (Step 1.5), invoke the `schedule` skill to set up a scheduled task that repeats Steps 3–4 (scan since last sync, rebuild artifact) using the same state file and artifact id. Otherwise, tell the user how to re-trigger a sync on demand (e.g. "just ask me to sync your tracker again").

## Guardrails

- Never fabricate a row, a status, or a note — if the evidence in an email is ambiguous, leave it out and mention the ambiguity rather than guessing.
- Never regress a status once it's reached `Rejected` or `Offer` from real evidence.
- Never delete existing note text — always append.
- Be transparent about where state is being saved and any limitations (e.g., "this won't survive past this session unless you connect a folder").

## Known limitations

- **English-language emails only.** The bundled search patterns are English phrases; a mailbox where job-search correspondence arrives in another language will get much weaker recall until the patterns are translated for that user.
- **Tested against one real inbox.** This skill's search logic is written generically, not tied to any one person's companies or wording, but it has only been validated end-to-end against one real, English-language mailbox with an existing job-search label and a rich, well-structured application history. A mailbox with a very different shape (little existing structure, an unusual ATS mix, mixed personal/professional mail) may surface edge cases this hasn't been tested against — treat early results from a new user with a bit more scrutiny until you've seen how it performs on their actual mail.
