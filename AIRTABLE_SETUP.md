# Airtable Setup Guide

This guide walks through creating the Airtable base that replaces the localStorage-based HTML tracker while preserving its layout and workflow as closely as possible.

---

## 1. Create the Base

1. Log in to [airtable.com](https://airtable.com)
2. Click **+ Create** > **Start from scratch**
3. Name the base: `AI Risk Program Tracker`
4. Rename the default table to `Work Items`

---

## 2. Table Schema: Work Items

Delete any default fields Airtable creates, then set up these fields in order.

### Core Fields (match the HTML app exactly)

| # | Field Name | Field Type | Configuration | HTML Source |
|---|-----------|------------|---------------|-------------|
| 1 | **Priority** | Single Select | Options: `CRITICAL` (red #dc2626), `HIGH` (orange #f59e0b), `MEDIUM` (blue #3b82f6) | `d.priority` — colored badge |
| 2 | **Program Component** | Single Select | See options list below | `d.component` — bold name in table |
| 3 | **Description** | Long Text | Component-level description | `d.desc` — gray subtitle under component name |
| 4 | **Key Activity** | Long Text | The specific work item | `d.activity` — editable textarea |
| 5 | **Deliverable / Output** | Long Text | What "done" looks like | `d.deliverable` — editable textarea |
| 6 | **SFS DRI** | Single Line Text | SFS directly responsible individual | `d.sfsDri` / `d.owner` — editable text input |
| 7 | **Block DRI** | Single Line Text | Block directly responsible individual | `d.blockDri` — editable text input |
| 8 | **Status** | Single Select | Options: `Not Started` (gray #94a3b8), `In Progress` (blue #3b82f6), `Blocked` (red #dc2626), `Complete` (green #16a34a) | `d.status` — color-coded dropdown |
| 9 | **Target Date** | Date | Date format: US or ISO | `d.date` — date picker |
| 10 | **Current State at Block** | Long Text | Current state assessment (read-only in HTML, fully editable in Airtable) | `d.currentState` — displayed as gray text |
| 11 | **Artifacts / Links** | URL or Long Text | Links to supporting documents | `d.artifacts` — text input |
| 12 | **Notes** | Long Text | General notes and context | `d.notes` — editable textarea |

### Additional Fields (requested in spec)

| # | Field Name | Field Type | Configuration |
|---|-----------|------------|---------------|
| 13 | **Industry Owners** | Single Line Text | External/industry owner if applicable |
| 14 | **Work We Need to Do** | Long Text | Detailed next steps beyond the Key Activity |
| 15 | **Comments** | Long Text | Dedicated comments field (in addition to Airtable's native per-record comment thread) |

### Formula Fields

| # | Field Name | Field Type | Formula |
|---|-----------|------------|---------|
| 16 | **Complete Flag** | Formula | `IF({Status} = 'Complete', '✅', '')` |
| 17 | **At Risk** | Formula | `IF(AND({Target Date} != BLANK(), {Status} != 'Complete', IS_BEFORE({Target Date}, TODAY())), '⚠️ OVERDUE', IF(AND({Target Date} != BLANK(), {Status} != 'Complete', IS_BEFORE({Target Date}, DATEADD(TODAY(), 14, 'days'))), '🔶 Due Soon', ''))` |

### Program Component Options

Create these as the Single Select options for the **Program Component** field, matching the 12 components in the HTML app:

1. AI Governance Structure
2. AI Policy Framework
3. AI Inventory & Classification
4. Risk Assessment
5. Testing & Validation (TEVV)
6. Monitoring & Oversight
7. Third-Party Risk Management (AI-Specific)
8. Data Governance for AI
9. Incident Response
10. Regulatory & Compliance
11. Training & Awareness
12. Stakeholder Engagement

---

## 3. Import Current Data

### Step 1: Export from HTML

1. Open `AI_Risk_Program_Tracker.html` in a browser
2. Click **Export CSV** in the header bar
3. Save the file (it will be named `AI_Risk_Program_Tracker.csv`)

### Step 2: Import into Airtable

1. In your `Work Items` table, click the `+` dropdown on the table tab > **Import data** > **CSV file**
2. Upload the CSV
3. Map columns using this reference:

| CSV Column Header | Map To Airtable Field |
|------------------|-----------------------|
| Priority | Priority |
| Program Component and Description | ⚠️ See note below |
| Key Activity | Key Activity |
| Deliverable / Output | Deliverable / Output |
| SFS DRI | SFS DRI |
| Block DRI | Block DRI |
| Status | Status |
| Target Date | Target Date |
| Current State at Block | Current State at Block |
| Artifacts / Links | Artifacts / Links |
| Notes | Notes |

**Important:** The CSV export combines component and description into one column (`"AI Governance Structure - Establish oversight bodies..."`). After import:
1. Import the combined column into a temporary text field
2. Split it: everything before ` - ` goes into **Program Component**, everything after goes into **Description**
3. You can do this with Airtable formulas or by editing in bulk
4. **Alternative:** Import, then manually set the Program Component single-select for each group (there are only 12 distinct components, so grouping makes this fast)

### Step 3: Verify

- Confirm ~100 records imported
- Spot-check Priority, Status, and Component values
- Verify Target Date fields parsed correctly
- Check that the single-select options for Priority and Status match the expected values

---

## 4. Views

Create these views in the `Work Items` table. These replicate the filtering capabilities of the HTML app.

### View 1: All Work Items (Grid)
- **Purpose:** Default view — everything visible
- **Sort:** Program Component (A→Z), then Priority (CRITICAL → HIGH → MEDIUM)
- **Fields shown:** All

### View 2: By Program Component (Grid)
- **Purpose:** Mirrors the "Progress by Program Component" section in HTML
- **Group by:** Program Component
- **Sort within groups:** Priority (CRITICAL first)
- **Collapse groups** by default for scanability
- **Summary row:** Show count and % complete per group

### View 3: High Priority (Grid)
- **Purpose:** Quick access to critical and high-priority items
- **Filter:** Priority is `CRITICAL` OR Priority is `HIGH`
- **Sort:** Priority (CRITICAL first), then Target Date (earliest first)

### View 4: Open Items (Grid)
- **Purpose:** Everything that's not done yet
- **Filter:** Status is NOT `Complete`
- **Sort:** Priority (CRITICAL first), then Target Date (earliest first)

### View 5: At Risk (Grid)
- **Purpose:** Mirrors the "Due Within 30 Days" section + overdue items
- **Filter:** At Risk field is NOT empty
- **Sort:** Target Date (earliest first)
- **Row coloring:** If available, color rows where At Risk contains "OVERDUE" in red

### View 6: Due Soon (Grid)
- **Purpose:** Items coming up in the next 30 days
- **Filter:** Target Date is within the next 30 days AND Status is NOT `Complete`
- **Sort:** Target Date (earliest first)

---

## 5. Interface Pages

Go to the **Interfaces** section in Airtable and create these three pages. Together, they replicate the full HTML experience.

---

### Page 1: Executive Dashboard

**HTML equivalent:** The top of the page — summary cards, status counts, priority bars, component progress.

**Layout (top to bottom):**

**Row 1 — Key Metrics (4 number elements across)**

| Element | Type | Config |
|---------|------|--------|
| Total Work Items | Number | Count all records in Work Items |
| Complete | Number | Count where Status = `Complete` |
| In Progress | Number | Count where Status = `In Progress` |
| Blocked / At Risk | Number | Count where Status = `Blocked` OR At Risk is not empty |

**Row 2 — Charts (2 charts side by side)**

| Element | Type | Config |
|---------|------|--------|
| Status Distribution | Pie/donut chart | Group by Status, colored by status colors |
| Priority Breakdown | Bar chart | Group by Priority, stack or color by Status to show completion within each priority level |

**Row 3 — Component Progress**

| Element | Type | Config |
|---------|------|--------|
| Progress by Component | Bar chart | Group by Program Component, show % where Status = Complete. This mirrors the HTML's 2-column progress bar grid. |

**Row 4 — At Risk Items**

| Element | Type | Config |
|---------|------|--------|
| Items At Risk | Grid (linked to At Risk view) | Shows overdue and due-soon items. Columns: Priority, Program Component, Key Activity, Status, Target Date, At Risk |

---

### Page 2: Team Tracker

**HTML equivalent:** The filter bar + main work item table. This is the daily operational view.

**Layout:**

**Top — Filter Bar**
- Add filter controls for: Priority, Status, Program Component, SFS DRI
- Airtable Interface grids support built-in filtering

**Main — Full Work Item Grid**

| Element | Type | Config |
|---------|------|--------|
| Work Items Grid | Grid element | Source: All Work Items view. Inline editing enabled. |

**Columns to show (in this order, matching the HTML table):**

1. Priority
2. Program Component
3. Key Activity
4. Deliverable / Output
5. SFS DRI
6. Block DRI
7. Status
8. Target Date
9. Current State at Block
10. Artifacts / Links
11. Notes
12. At Risk
13. Complete Flag

**Settings:**
- Enable inline editing for all editable fields
- Sort default: Priority (CRITICAL first), then Program Component
- Allow users to add new records from this grid (replaces the "+ Add Work Item" button)

---

### Page 3: By Component

**HTML equivalent:** The "Progress by Program Component" section, but interactive — you can drill into each one.

**Layout:**

**Top — Component Selector**
- Record list or filter element grouped by Program Component
- Let users click a component to see its items below

**Middle — Summary for Selected Component**

| Element | Type | Config |
|---------|------|--------|
| Total Items | Number | Count for selected component |
| Complete | Number | Count where Status = Complete for selected component |
| Completion % | Number | % complete for selected component |
| Blocked | Number | Count where Status = Blocked for selected component |

**Bottom — Items Grid**

| Element | Type | Config |
|---------|------|--------|
| Component Items | Grid element | Filtered to selected Program Component. All fields visible. Inline editing enabled. |

---

## 6. Sharing & Permissions

### Add Team Members

1. Click **Share** on the base (top right)
2. Add team members by email
3. Set permission level per person:

| Role | Permission Level | Who |
|------|-----------------|-----|
| Program leads / admin | **Creator** | You + co-leads who manage the tracker structure |
| DRIs who update status | **Editor** | Team members responsible for work items |
| Stakeholders who review | **Commenter** | Execs, reviewers who need to see + comment but not edit |
| External reviewers | **Read only** | Auditors, external stakeholders if needed |

### Share Interface Pages Directly

You can share interface page links directly:
1. Open an interface page
2. Click **Share** in the top right of the interface
3. Copy the link
4. Send to team members

This is the best way to share the **Executive Dashboard** with leadership — they get a clean view without seeing the raw grid.

---

## 7. Ongoing Maintenance

### Weekly
- DRIs update their assigned items directly in the Team Tracker interface
- Review the **At Risk** view and **Due Soon** view
- Use Airtable comments to discuss blockers on specific records

### Monthly
- Export CSV backup from the **All Work Items** view
- Save to `exports/` in the GitHub repo and push
- Review the **Executive Dashboard** interface for reporting

### Quarterly
- Review the **By Component** interface page for program-level assessment
- Archive completed items if the table gets large (move to an "Archive" table)
- Review and update permissions as team membership changes

---

## 8. Optional: Form View for Adding Items

If you want to replicate the HTML "Add Work Item" modal experience exactly:

1. In the `Work Items` table, click **Views** > **Form**
2. Name it: `Submit New Work Item`
3. Include fields: Program Component, Priority, Key Activity, Deliverable / Output, SFS DRI, Block DRI, Target Date, Notes
4. Share the form link with anyone who needs to submit items (even people who don't have base access)
5. New submissions appear automatically in the table

This is actually **better** than the HTML modal because:
- The form can be shared outside the team
- Submissions are validated
- You can add required fields
- No code changes needed to update the form
