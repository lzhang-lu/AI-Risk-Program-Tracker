# Deployment & Architecture Guide

---

## 1. Architecture Decision

### Recommended: Airtable (system of record + team editing) + GitHub (backup + static reference)

```
┌─────────────────────────────────────────────────────────┐
│                    TEAM MEMBERS                          │
│              (view, edit, collaborate)                    │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│           AIRTABLE (System of Record)                    │
│                                                          │
│  Base: AI Risk Program Tracker                           │
│  Table: Work Items (~100 records, 14+ fields)            │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Executive    │  │  Team        │  │  By           │  │
│  │  Dashboard    │  │  Tracker     │  │  Component    │  │
│  │  (Interface)  │  │  (Interface) │  │  (Interface)  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  6 Grid Views for filtering/sorting                      │
│  Real-time multi-user editing                            │
│  Comments, activity log, revision history                │
└──────────────────────┬──────────────────────────────────┘
                       │ periodic CSV export (manual)
                       ▼
┌─────────────────────────────────────────────────────────┐
│           GITHUB (Backup & Reference)                    │
│                                                          │
│  Repo: AI-Risk-Program-Tracker                           │
│  - Original HTML app (source code)                       │
│  - Documentation (README, DEPLOYMENT, AIRTABLE_SETUP)    │
│  - CSV exports in /exports                               │
│  - GitHub Pages: read-only static demo                   │
└─────────────────────────────────────────────────────────┘
```

### Why This Fits

| Constraint | How it's met |
|------------|-------------|
| Only GitHub + Airtable available | No other services required |
| No secrets in client-side code | Airtable PAT never touches HTML/JS — team edits through Airtable's own UI |
| Team needs shared visibility | Airtable provides real-time multi-user access natively |
| Team needs shared editing | Airtable grid + interface pages = native editing with no custom code |
| Preserve the HTML experience | Airtable Interfaces can replicate dashboard + tracker layout (see UI Preservation Plan) |
| Keep it simple | Zero middleware, zero servers, zero build steps |

### Why NOT a Custom Shared Web App

A fully shared editable version of the current HTML app would require:
- A **backend server** to proxy Airtable API calls (so the PAT stays secret)
- **Hosting infrastructure** you don't have (Heroku, Vercel, AWS, etc.)
- **User authentication** to control who can edit
- **Conflict resolution** for simultaneous edits

**You do not have access to any of these.** Airtable Interfaces give you all of this out of the box — authentication, multi-user editing, conflict handling, permissions — with zero infrastructure.

---

## 2. Fastest Safe Path

**Total time to working shared tracker: ~45 minutes**

| Step | Time | What |
|------|------|------|
| 1 | 2 min | Export CSV from the HTML app (click Export CSV) |
| 2 | 5 min | Create Airtable base and table with schema |
| 3 | 5 min | Import CSV into Airtable |
| 4 | 5 min | Fix column mappings and add formula fields |
| 5 | 5 min | Create 6 grid views |
| 6 | 15 min | Build 3 interface pages (Dashboard, Tracker, By Component) |
| 7 | 3 min | Share base with team members |
| 8 | 5 min | Push docs to GitHub, enable Pages |

**After step 7, the team is live.** Step 8 is for documentation and backup.

---

## 3. Migration Plan (Step-by-Step)

### Step 1: Export from HTML

1. Open `AI_Risk_Program_Tracker.html` in your browser
2. If you've made edits that are in localStorage, they'll be included
3. Click **Export CSV** in the header
4. Save the file — this is your migration payload (~100 rows)

### Step 2: Create Airtable Base

1. Log in to [airtable.com](https://airtable.com)
2. Click **+ Create** > **Start from scratch**
3. Name it: `AI Risk Program Tracker`
4. Rename the default table to `Work Items`
5. Follow the schema in `AIRTABLE_SETUP.md`

### Step 3: Import CSV

1. In the `Work Items` table, click **+** > **Import data** > **CSV file**
2. Upload your exported CSV
3. Map the columns (see mapping table in `AIRTABLE_SETUP.md`)
4. After import, verify record count matches (~100 items)

### Step 4: Build Views and Interfaces

Follow the detailed instructions in `AIRTABLE_SETUP.md` for:
- 6 grid views (All Work Items, By Component, High Priority, Open Items, At Risk, Due Soon)
- 3 interface pages (Executive Dashboard, Team Tracker, By Component)

### Step 5: Share with Team

1. Click **Share** on the base
2. Add team members by email
3. Set permissions: Creator for leads, Editor for DRIs, Commenter for reviewers

### Step 6: Push to GitHub

Run the terminal commands below.

---

## 4. GitHub Terminal Commands

Your repo already exists at `https://github.com/lzhang-lu/AI-Risk-Program-Tracker.git`.

```bash
cd /Users/lzhang/AI-Risk-Program-Tracker

# Copy HTML as index.html for GitHub Pages (if not already done)
cp AI_Risk_Program_Tracker.html index.html

# Create exports directory
mkdir -p exports
touch exports/.gitkeep

# Stage all documentation and setup files
git add README.md DEPLOYMENT.md AIRTABLE_SETUP.md .gitignore index.html exports/.gitkeep

# Commit
git commit -m "Add deployment docs, Airtable migration guide, and GitHub Pages setup"

# Push
git push origin main
```

### If You Were Starting a New Repo From Scratch

```bash
mkdir AI-Risk-Program-Tracker
cd AI-Risk-Program-Tracker
git init
git add .
git commit -m "Initial commit: AI Risk Program Tracker"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
git push -u origin main
```

### Enable GitHub Pages

1. Go to **https://github.com/lzhang-lu/AI-Risk-Program-Tracker/settings/pages**
2. Under **Source**, select `Deploy from a branch`
3. Set branch to `main`, folder to `/ (root)`
4. Click **Save**
5. Wait 1-2 minutes
6. Verify at: `https://lzhang-lu.github.io/AI-Risk-Program-Tracker/`

This publishes the original HTML as a **read-only reference/demo**. It is NOT the team's editing surface.

---

## 5. UI Preservation Plan

This section maps every section of the current HTML app to its Airtable equivalent.

### Section-by-Section Mapping

#### Header Bar
| HTML | Airtable Equivalent | Match Level |
|------|---------------------|-------------|
| Title: "AI Risk Program Project Tracker" | Interface page title (customizable) | Exact |
| "Last updated" timestamp | Airtable shows "Last modified" per record automatically | Close — per-record, not global |
| "Export CSV" button | Built-in: grid view > `...` > Download CSV | Exact |
| "Auto-saving enabled" indicator | Airtable auto-saves every edit in real-time | Better — truly real-time + shared |

#### Dashboard Row 1: Summary Cards (4 cards)
| HTML Card | Airtable Interface Element | Match Level |
|-----------|---------------------------|-------------|
| **Total Work Items** (count) | Number element: count all records | Exact |
| **Overall Completion** (% ring + "X of Y complete") | Number element: % where Status = Complete. Ring chart not available — use a progress bar or pie chart | Close — no donut ring, but % + chart works |
| **Progress by Priority** (CRITICAL/HIGH/MEDIUM bars with X/Y counts) | Chart element: stacked bar grouped by Priority, colored by Status. Counts shown in legend | Close — Airtable charts don't show "5/20" inline text on bars, but data is same |

#### Dashboard Row 2: Status Cards (4 clickable cards)
| HTML Card | Airtable Interface Element | Match Level |
|-----------|---------------------------|-------------|
| Not Started / In Progress / Blocked / Complete — count + click-to-filter | 4 Number elements showing counts per status. **Cannot** click to filter the grid below. | Partial — counts are exact, but click-to-filter behavior is not possible in Airtable Interfaces |
| **Workaround:** Use the filter bar on the grid element in the Team Tracker page instead | |

#### Due Within 30 Days Section
| HTML | Airtable Interface Element | Match Level |
|------|---------------------------|-------------|
| List of items due in next 30 days with color-coded urgency (overdue/this week/later), showing activity, component, priority, status | Grid element filtered to `Due Soon` view. Use conditional coloring on the At Risk formula field. | Close — grid rows vs. custom card layout, but same data and urgency visibility |
| **What's lost:** The custom card-style layout with "3 days overdue" / "In 5 days" labels. Airtable shows dates but not relative urgency text. The `At Risk` formula field partially compensates with "OVERDUE" / "Due Soon" flags. | |

#### Progress by Program Component
| HTML | Airtable Interface Element | Match Level |
|------|---------------------------|-------------|
| 2-column grid of all 12 components, each with a progress bar and % | Chart element: bar chart grouped by Program Component, showing % complete. Or use the "By Component" interface page with a record list. | Close — bar chart shows the data, but not the exact compact 2-column progress-bar layout |
| **Alternative:** The "By Component" interface page lets users click a component and see its summary + items — arguably more useful than the static bar grid. | |

#### Filter Bar
| HTML | Airtable Interface Element | Match Level |
|------|---------------------------|-------------|
| Priority dropdown | Built-in filter on grid view or interface grid element | Exact |
| Status dropdown | Built-in filter | Exact |
| Component dropdown | Built-in filter or group-by | Exact |
| Search text field | Built-in search in grid views and interface grid elements | Exact |
| "Clear all filters" | Built-in "clear filters" in Airtable | Exact |
| "+ Add Work Item" button | Built-in: click `+` on last row or use form view | Exact |

#### Work Item Table (Main Data Grid)
| HTML Column | Airtable Field | Inline Editable? | Match Level |
|-------------|---------------|-------------------|-------------|
| Priority (color badge) | Single Select with colors | Yes (click to change) | Exact |
| Program Component (name + description) | Single Select (name) + Long Text (description) | Yes | Close — two fields instead of one compound cell |
| Key Activity (textarea) | Long Text | Yes | Exact |
| Deliverable / Output (textarea) | Long Text | Yes | Exact |
| SFS DRI (text input) | Single Line Text | Yes | Exact |
| Block DRI (text input) | Single Line Text | Yes | Exact |
| Status (color dropdown) | Single Select with colors | Yes | Exact |
| Target Date (date picker) | Date field | Yes | Exact |
| Current State at Block (read-only text) | Long Text | Yes (now editable, which is better) | Better |
| Artifacts / Links (text input) | URL or Long Text | Yes | Exact |
| Notes (textarea) | Long Text | Yes | Exact |
| Delete button (🗙) | Right-click > Delete record, or select + delete | Close — no single-click trash icon, but delete is available |

#### Add Work Item Modal
| HTML | Airtable Equivalent | Match Level |
|------|---------------------|-------------|
| Modal with fields for Component, Priority, Activity, Deliverable, SFS DRI, Block DRI, Date, Notes | Click `+` on last row to add inline, OR create a Form view for structured entry | Close — Form view replicates the modal experience; inline add is even faster |

#### Sorting
| HTML | Airtable | Match Level |
|------|----------|-------------|
| Click column header to sort asc/desc | Click column header > Sort A→Z / Z→A, or configure sort in view settings | Exact |

#### Undo Delete
| HTML | Airtable | Match Level |
|------|----------|-------------|
| 10-second undo bar after delete | Cmd+Z / Ctrl+Z immediately after delete, or revision history to recover | Close — Airtable undo is keyboard-based, not a floating bar |

### What Cannot Be Replicated in Airtable

| Feature | Why | Impact | Mitigation |
|---------|-----|--------|------------|
| Canvas-drawn progress ring (donut chart) | Airtable Interfaces don't support canvas/custom JS | Low — cosmetic only | Use pie chart or number element with % |
| Click status card to filter the table below | Interface elements aren't linked interactively | Medium — convenience feature | Use the filter bar on the grid element instead |
| "X days overdue" / "In Y days" relative labels | Airtable shows dates but not computed relative text | Low | The `At Risk` formula shows "OVERDUE" / "Due Soon" |
| Compact 2-column component progress bars | Interface charts are full-width, not compact grid | Low — cosmetic | Bar chart or "By Component" page provides same info |
| Single compound cell (component + description) | Airtable fields are one-per-column | None — this is actually cleaner | Two separate fields is better for filtering/grouping |
| Custom color gradients and CSS styling | Airtable has its own visual system | Low — professional look, just different | Airtable's built-in styling is clean and consistent |

### What Gets BETTER in Airtable

| Feature | Improvement |
|---------|------------|
| Real-time collaboration | Multiple people editing simultaneously — impossible in localStorage |
| Revision history | See who changed what, when. Roll back any record. |
| Comments | Add comments to any record — discussion threads per work item |
| Permissions | Control who can edit vs. view vs. comment |
| Notifications | Get notified when records you own are changed |
| Mobile access | Airtable mobile app works natively |
| No data loss | Data is cloud-persisted, not trapped in one browser's localStorage |
| Row-level detail view | Click any record to see all fields in a clean expanded view |

---

## 6. DO NOT DO THIS

### Never expose Airtable PATs in frontend code

```javascript
// ⛔ NEVER DO THIS — anyone can view-source and steal the token
const AIRTABLE_PAT = 'patXXXXXXXXXXXXXX';
fetch('https://api.airtable.com/v0/appXXXX/Work%20Items', {
  headers: { 'Authorization': 'Bearer ' + AIRTABLE_PAT }
});
```

**Why:**
- Anyone who visits the page can open DevTools > Network tab and see the token
- The token gives **full read/write/delete access** to your entire Airtable base
- There is no way to scope an Airtable PAT to read-only from a browser
- A leaked token could allow complete data destruction

### Never pretend GitHub Pages can support shared editing

GitHub Pages serves static files. It cannot:
- Accept form submissions
- Store data server-side
- Authenticate users
- Proxy API calls to hide secrets

If someone suggests "just call the Airtable API from GitHub Pages," that requires embedding the PAT in client-side code. See above.

### The safe path

Use Airtable's own interface for editing. Use GitHub Pages only for read-only reference or documentation.

---

## 7. Team Rollout Steps

### Day 1: Setup (You)
1. Create Airtable base and import data
2. Set up views and interfaces per `AIRTABLE_SETUP.md`
3. Test by editing a few records

### Day 2: Pilot (You + 1-2 team members)
1. Share the base with 1-2 team members as Editors
2. Have them update their assigned items
3. Confirm everyone sees each other's edits in real-time

### Day 3: Full Rollout
1. Share with all team members
2. Send a message:
   > "The AI Risk Program Tracker is now in Airtable. Please use the Team Tracker interface for daily updates. The old HTML version is available as a read-only reference at [GitHub Pages URL]. All edits should happen in Airtable."
3. Push documentation to GitHub
4. Enable GitHub Pages for the read-only reference
