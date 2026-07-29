# job-tracker-setup

A Claude skill that builds a **live, self-updating job-application tracker** from your Gmail. Point it at your inbox and it scans for application confirmations, rejections (including deleted ones in Trash), interview invites, offers, referrals, and recruiter/HM correspondence, then publishes a live dashboard that stays current on a schedule you choose.

Created by [Jessica Higgs](https://github.com/jessicahiggs). Generalized from a workflow originally built for one real job search, so it can bootstrap a fresh tracker for anyone's inbox.

## What it looks like

A live dashboard, updated automatically from your inbox — every application with its status, date, and notes, plus quick filters and search across companies and roles. *(Sample below uses illustrative data.)*

<img width="743" height="605" alt="Job application tracker dashboard" src="https://github.com/user-attachments/assets/830a0704-ea39-4e03-b9ba-05fa354ec742" />

## What it does

- **Seeds** from an existing spreadsheet/CSV (optional) or builds your pipeline from scratch by scanning inbox history.
- **Classifies** each application into a small, ordered status set: `Applied → Warm → Interviewing → Offer`, with `Rejected` as a terminal state.
  - `Warm` = the specific role didn't proceed, but a recruiter is keeping you warm / referring you elsewhere — a state that otherwise gets mislabeled.
- **Maintains** the tracker on every run: adds new applications, moves statuses forward on real email evidence, and appends dated notes from recruiter threads — **without ever overwriting your existing notes**.
- **Publishes** a searchable, filterable live dashboard (see `job-tracker-setup/assets/tracker_template.html`) that you can reopen anytime.
- **Optionally schedules** itself to re-sync automatically (e.g. nightly).

## Requirements

- A connected **Gmail** MCP connector (to search/read mail).
- Cowork's **artifact** tool (`mcp__cowork__create_artifact` / `update_artifact`) to publish the live dashboard.
- *Optional:* a Drive/Sheets connector (only to seed from an existing spreadsheet) and the `schedule` skill (only for automatic recurring runs).

## Install

This is a Claude skill. You install it once, then talk to Claude normally — you never open or read the files yourself. Claude reads them for you.

### 1. Download the skill

On this page: click the green **Code** button → **Download ZIP**, then unzip it.

### 2. Zip the inner folder

⚠️ **This is the step people get wrong.** The download gives you two nested folders with almost the same name:

```
job-tracker-setup-main/        ← the repo. NOT this one.
└── job-tracker-setup/         ← the skill. THIS one.
    ├── SKILL.md
    ├── references/
    └── assets/
```

You want the **inner** `job-tracker-setup` folder — the one with `SKILL.md` sitting directly inside it. Right-click it → **Compress**.

If you zip the outer folder by mistake, `SKILL.md` ends up one level too deep and the upload will be rejected.

### 3. Upload it to Claude

Go to [claude.ai](https://claude.ai) → **Settings** → **Capabilities** → **Skills** → upload your zip.

### 4. Connect Gmail

**Settings** → **Connectors** → add **Gmail**, and sign in with the address you apply to jobs from.

The tracker only ever sees this one account. If you apply from a second address, those applications won't show up.

### 5. Ask for your tracker

Start a new chat and say:

> build me a job application tracker from my Gmail

Claude will ask you a few short questions — which account to scan, whether you have an existing spreadsheet to import, whether you use a Gmail label for job mail, and whether you want it to re-check automatically each night. Defaults are fine for all of them; answer "no" to anything you don't have.

Then it scans your inbox and builds your dashboard.

### Later

To refresh it any time:

> sync my job tracker

---

**Using Claude Code instead?** Drop the inner `job-tracker-setup` folder into `~/.claude/skills/` and the skill will load. Note that the live dashboard is published as a Claude artifact, so the final publishing step is designed for claude.ai.

## How it works (files)

- **`SKILL.md`** — the entry point: prerequisite checks, the setup interview, and the build/schedule flow.
- **`references/gmail_queries.md`** — the exact Gmail search patterns for each signal type (new application, rejection, interview, offer, sent-mail dialogue, referral) and the reasoning behind them.
- **`references/status_rules.md`** — the status/notes rules that keep the tracker trustworthy across many runs (progression, "Warm is sticky," re-applications, row matching).
- **`assets/tracker_template.html`** — the self-contained dashboard template with `{{LAST_SYNC}}`, `{{RECENT_CHANGES}}`, and `{{DATA}}` placeholders.

## Design notes / hard-won lessons

This skill has been stress-tested against a real, active job search. A few rules exist specifically because the naive version got them wrong:

- **Never trust a single phrase to detect an application.** Confirmation wording varies enormously; the skill matches on a wide phrase list *and* a sender heuristic *and* a direct label sweep.
- **Read every email's body for the exact role.** One email thread can contain confirmations for two *different* roles at the same company. Companies don't send duplicate confirmations, so two confirmations with different role text are two applications — never merge them.
- **Convert Gmail's UTC timestamps to the user's local date** before deciding what counts as "today."
- **A tracker only sees the one account it's connected to** — mail from a second address won't appear.

## Limitations

- **Single Gmail account** at a time.
- **English-language emails** — the bundled search phrases are English; other languages need translated patterns.
- **Validated end-to-end against one real inbox** — a very differently-shaped mailbox may surface untested edge cases.

## License

MIT — see [LICENSE](LICENSE).
