# AI Risk Program Project Tracker

## System Overview

This project tracks work items for the AI Risk Management Program across 12+ program components. It originated as a single-page HTML app with browser-local storage. **The system of record is now Airtable** for shared team collaboration.

---

## Where Things Live

| What | Where | Purpose |
|------|-------|---------|
| **System of record** | Airtable Base: `AI Risk Program Tracker` | All edits happen here. This is the source of truth. |
| **Primary team interface** | Airtable Interfaces (3 pages) | Mirrors the HTML dashboard + tracker experience for shared use. |
| **Source code & backups** | This GitHub repo | Stores the original HTML app, documentation, and periodic CSV exports. |
| **Reference/demo site** | GitHub Pages | Read-only static view of the original tracker layout. Not for live team editing. |

---

## How to Edit Data (For Team Members)

1. Open the **Airtable Base** (link shared by your team lead)
2. For day-to-day work: use the **Team Tracker** interface page
3. For status reviews: use the **Executive Dashboard** interface page
4. For component deep-dives: use the **By Component** interface page
5. You can also edit directly in the grid views if you prefer spreadsheet-style editing

**Do NOT edit the HTML file and expect changes to be shared.** The HTML file uses `localStorage`, which is private to your individual browser on your individual machine. No one else can see your edits.

---

## How to Update Backups

Periodically (weekly or monthly), export data from Airtable and commit it here:

```bash
# 1. In Airtable: go to "All Work Items" view > ... menu > Download CSV
# 2. Save the CSV to the exports/ folder in this repo
cd AI-Risk-Program-Tracker
cp ~/Downloads/Work\ Items-All\ Work\ Items.csv exports/backup-$(date +%Y-%m-%d).csv
git add exports/
git commit -m "Backup: Airtable export $(date +%Y-%m-%d)"
git push
```

---

## GitHub Pages (Read-Only Reference)

The original HTML tracker is published as a static reference at:

```
https://lzhang-lu.github.io/AI-Risk-Program-Tracker/
```

This is a **read-only demo** showing the original layout. Edits made here are stored only in your local browser. Use Airtable for shared work.

---

## Repository Structure

```
AI-Risk-Program-Tracker/
  AI_Risk_Program_Tracker.html   # Original HTML app (reference/backup)
  index.html                      # GitHub Pages entry point (copy of HTML app)
  README.md                       # This file
  DEPLOYMENT.md                   # Architecture + deployment + rollout guide
  AIRTABLE_SETUP.md              # Airtable schema, views, interfaces, sharing
  .gitignore                      # Git ignore rules
  exports/                        # Periodic CSV backups from Airtable
```

---

## Key Rules

1. **System of record = Airtable.** Always.
2. **GitHub = backup, documentation, and static reference.**
3. **Never put Airtable tokens or API keys in any HTML/JS file.**
4. **If you need to edit data, go to Airtable. Not the HTML file. Not GitHub.**
