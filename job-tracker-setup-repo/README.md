# job-tracker-setup

A [Claude Skill](https://code.claude.com/docs/en/skills) that builds a live, self-updating "Job Application Tracker" dashboard by scanning your Gmail for application confirmations, rejections, interview invites, and offers — no manual spreadsheet upkeep required.

Created by [Jessica Higgs](https://github.com/jessicahiggs).

## What it does

- Interviews you briefly (which inbox, any existing seed spreadsheet, status categories, whether you want it to run nightly)
- Scans your mailbox using a set of battle-tested Gmail search patterns (see `job-tracker-setup/references/gmail_queries.md`)
- Builds a live dashboard artifact — filterable, sortable, searchable — that never fabricates a row or silently overwrites your notes
- Optionally sets up a recurring nightly sync via the `schedule` skill

## Requirements

- A connected Gmail MCP connector
- Cowork's artifact tool (`create_artifact` / `update_artifact`) to publish the live dashboard
- Optional: a Drive/Sheets connector, only if you want to seed from an existing spreadsheet
- Optional: the `schedule` skill, only if you want automatic nightly runs

**Note:** built and tested for English-language inboxes. See `job-tracker-setup/SKILL.md` for the full known-limitations list.

## Install

Download `job-tracker-setup/` from this repo and drop it into your `.claude/skills/` directory (Claude Code) or install it via Cowork's "Save skill" option if you have the packaged `.skill` file.

## Structure

```
job-tracker-setup/
├── SKILL.md                       — main instructions
├── references/
│   ├── gmail_queries.md           — the search patterns for each signal type
│   └── status_rules.md            — status/notes rules that keep the tracker trustworthy
├── assets/
│   └── tracker_template.html      — the dashboard template
└── evals/
    └── evals.json                 — test prompts used to validate this skill
```

## License

MIT — see LICENSE, or add your preferred license before publishing.
