---
name: release-notes-export
description: >
  Generates a formatted Excel release notes file for a customer patch by querying Azure
  DevOps. Use this skill whenever a user asks to generate release notes, create a release
  notes export, produce a hotfix summary, or says things like: "generate release notes for
  Daylight", "create an excel export for release 2023.04 with release date on or after
  1/1/2025", "what was included in Daylight Patch 8", "release notes for Upstream Suite
  2023.04 since January", "patch export for [customer]", "client-facing release notes",
  "internal release notes". Always use this skill before manually querying ADO work items
  for release note purposes — it handles the full workflow: ADO querying by Release Date,
  HTML stripping, Patch label mapping, product sheet splitting, and Excel generation.
  NEVER ask the user for specific release version numbers (e.g. 2023.04.1.28); always
  derive the version range from the Release Date filter instead.
---

# Release Notes Export Skill

Produces a formatted Excel file (.xlsx) containing all Bugs and Features included in a
customer's hotfix/patch, grouped by Release Item, split by product tab, with clean
plain-text release note fields.

**Key design principle:** PMs don't know exact version numbers. The version range is
always derived automatically by querying Release Items filtered by `Custom.ReleaseDate`.
Never ask for specific version numbers like `2023.04.1.28` — use the date filter instead.

---

## Inputs Required from User

Collect only these inputs. Never ask for specific version numbers:

| Input | Example | Notes |
|---|---|---|
| **Customer name** (or "internal") | `Daylight`, `MEW`, `internal` | Used to look up Patch WI for label. Use `internal` to skip. |
| **Product suite** | `Upstream Suite`, `eSuite` | Determines Release Item title prefix and sheet split |
| **Major version** | `2023.04`, `2024.04` | The hotfix line (first two segments only) |
| **Release Date on or after** | `1/1/2025`, `January 2025` | Inclusive lower bound on `Custom.ReleaseDate` |

**Common customer abbreviations in ADO:** `DAY` = Daylight, `MEW` = MEW Energy,
`APH` = Apache, `UTG` = UTG, `GEC` = GEC. Full names also work.

---

## Step 0 — ADO Connection

Always follow the `ado-connection` skill first. Verify connection with `wit_my_work_items`
before proceeding.

---

## Step 1 — Resolve Patch Label (skip if customer = "internal")

The Patch label (e.g. `"Patch 8 - 1/30/2026"`) is stored in the customer's Patch WI in
the `QuorumServices` project.

### 1a. Search for the Patch WI

```wiql
SELECT [System.Id], [System.Title], [Microsoft.VSTS.Common.ClosedDate],
       [System.ChangedDate], [Custom.ToCoreHotfixVersion]
FROM WorkItems
WHERE [System.TeamProject] = 'QuorumServices'
  AND [System.WorkItemType] = 'Patch'
  AND [System.Title] CONTAINS '{CUSTOMER_NAME}'
  AND [Custom.SourceRelease] = '{MAJOR_VERSION}'
  AND [System.State] <> 'Cancelled'
ORDER BY [System.CreatedDate] DESC
```

If multiple Patch WIs are returned, pick the most recent one whose
`Custom.ToCoreHotfixVersion` field (HTML) includes a version with a Release Date on or
after the user's requested date.

### 1b. Parse the Patch Label

Fetch the matching Patch WI with `expand: all`. Parse `Custom.ToCoreHotfixVersion` HTML
to extract version → Release WI ID mappings like:

```
UPS on 2023.04.1.33 hotfix - #1757597
ESuite on 2023.04.1.28 hotfix - #1765116
```

Derive the Patch label from the WI title:
- `"DAY - Patch 8 on 2023.04 - Upstream"` → extract `"Patch 8"`
- Append `ClosedDate` (or `ChangedDate` if no ClosedDate) formatted as `M/D/YYYY`
- Result: `"Patch 8 - 1/30/2026"`

---

## Step 2 — Fetch Release Items by Release Date

Use a `WorkItemLinks` WIQL query modeled on Sarah's ADO query. Filter by
`Custom.ReleaseDate` on the Source Release Item — do not filter by version number.

### Product title prefix mapping

| Product suite | Use in WIQL `CONTAINS` |
|---|---|
| `Upstream Suite` | `'UPS'` |
| `eSuite` | `'ESuite'` |
| Both / unknown | run two queries, one for each prefix, and merge |

### WIQL template

```wiql
SELECT [System.Id], [System.Title]
FROM WorkItemLinks
WHERE (
    Source.[System.TeamProject] = @project
    AND Source.[System.WorkItemType] = 'Release'
    AND Source.[System.Title] CONTAINS '{TITLE_PREFIX}'
    AND Source.[System.Title] CONTAINS '{MAJOR_VERSION}'
    AND Source.[Custom.ReleaseDate] >= '{ISO_DATE}'
)
AND ([System.Links.LinkType] = 'System.LinkTypes.Related-Forward')
AND (
    Target.[System.TeamProject] = @project
    AND Target.[System.State] <> ''
    AND Target.[System.WorkItemType] IN ('Bug', 'Feature')
)
MODE (MustContain)
```

Replace:
- `{TITLE_PREFIX}` → `UPS` or `ESuite`
- `{MAJOR_VERSION}` → e.g. `2023.04`
- `{ISO_DATE}` → ISO 8601, e.g. `2025-01-01T00:00:00.0000000`

The query returns `workItemRelations`. Parse the structure:
- `rel = null` → source Release Item ID is in `target.id`
- `rel != null` → `source.id` = Release Item, `target.id` = linked Bug/Feature

### Extract and announce Release Items

Batch-fetch all unique source IDs with fields:
`System.Id`, `System.Title`, `EngineeringCMMI.ReleaseVersion`, `Custom.ReleaseDate`

Sort by `Custom.ReleaseDate` ascending. Before continuing, **report to the user** which
Release Items were found, e.g.:
> Found 6 Release Items from 6/6/2025 to 11/7/2025:
> UPS 2023.04.1.28, UPS 2023.04.1.29, UPS 2023.04.1.30, UPS 2023.04.1.31, UPS 2023.04.1.32, UPS 2023.04.1.33

---

## Step 3 — Fetch Linked Bug/Feature Fields

Batch-fetch all target WI IDs with `wit_get_work_items_batch_by_ids` (max 200 per call):

```
System.Id, Custom.SalesforceCase, EngineeringCMMI.Product, EngineeringCMMI.Module,
System.WorkItemType, System.Title, Custom.ReleaseNoteApproved,
EngineeringCMMI.ReleaseNoteIndicator, EngineeringCMMI.ReleaseNoteTitle,
EngineeringCMMI.ReleaseNote, EngineeringCMMI.ReleaseNoteScope,
EngineeringCMMI.ReleaseNoteImplementationDetails, Custom.TopItem,
EngineeringCMMI.HighAwareness
```

Filter results to only `System.WorkItemType IN ('Bug', 'Feature')`.

**HTML stripping** — apply to `ReleaseNote`, `ReleaseNoteScope`,
`ReleaseNoteImplementationDetails`:

```python
import html, re

def strip_html(text):
    if not text:
        return ""
    text = re.sub(r"<[^>]+>", " ", str(text))
    text = html.unescape(text)
    text = re.sub(r"[ \t]+", " ", text)
    text = re.sub(r"\s*\n\s*", "\n", text)
    return text.strip()
```

**Boolean normalization** — `True/False` → `1/0` for:
`Custom.ReleaseNoteApproved`, `EngineeringCMMI.ReleaseNoteIndicator`,
`Custom.TopItem`, `EngineeringCMMI.HighAwareness`

---

## Step 4 — Build Output Rows

One row per Bug/Feature. Sort by Release Item `Custom.ReleaseDate` ascending, then
Work Item ID ascending within each release.

| Column | Source |
|---|---|
| Release Item Title | `System.Title` from Release Item |
| Patch Label | From Step 1 (e.g. `"Patch 8 - 1/30/2026"`). Empty if internal mode. |
| Work Item ID | `System.Id` |
| Salesforce Case | `Custom.SalesforceCase` |
| Product | `EngineeringCMMI.Product` |
| Module | `EngineeringCMMI.Module` |
| Work Item Type | `System.WorkItemType` |
| Release Note Title | `EngineeringCMMI.ReleaseNoteTitle` |
| Release Note | `EngineeringCMMI.ReleaseNote` (HTML stripped) |
| Release Note Scope | `EngineeringCMMI.ReleaseNoteScope` (HTML stripped) |
| Release Note Implementation Details | `EngineeringCMMI.ReleaseNoteImplementationDetails` (HTML stripped) |
| Top Item | `Custom.TopItem` (0/1) |
| High Awareness | `EngineeringCMMI.HighAwareness` (0/1) |
| Release Note Approved *(internal only)* | `Custom.ReleaseNoteApproved` (0/1) |
| Release Note Indicator *(internal only)* | `EngineeringCMMI.ReleaseNoteIndicator` (0/1) |

---

## Step 5 — Product Sheet Splitting

Split rows by `EngineeringCMMI.Product` on the Bug/Feature WI:

| Sheet | Product values |
|---|---|
| `Upstream` | `Upstream Suite`, `QRA`, `QCA`, `QDO`, `QCFS`, `QPTM` |
| `Esuite` | `eSuite`, `ESuite`, `esuite` |

Unrecognized products go to an `Other` sheet. Omit empty sheets.

---

## Step 6 — Column Layout per Mode

### Client-facing (customer specified)
```
Release Item Title | Patch Label | Work Item ID | Salesforce Case | Product | Module |
Work Item Type | Release Note Title | Release Note | Release Note Scope |
Release Note Implementation Details | Top Item | High Awareness
```

### Internal (customer = "internal")
```
Release Item Title | Work Item ID | Salesforce Case | Product | Module |
Work Item Type | Release Note Approved | Release Note Indicator |
Release Note Title | Release Note | Release Note Scope |
Release Note Implementation Details | Top Item | High Awareness
```

---

## Step 7 — Write Excel File

Install openpyxl if needed: `pip install openpyxl --break-system-packages -q`

**Header**: `#1F3864` fill, white bold Calibri 11, centered, wrap text, row height 30
**Data rows**: top-aligned, wrap text OFF. **Freeze panes**: row 2.

### Column widths

| Column | Width | Column | Width |
|---|---|---|---|
| Release Item Title | 28 | Release Note Title | 45 |
| Patch Label | 22 | Release Note | 70 |
| Work Item ID | 12 | Release Note Scope | 30 |
| Salesforce Case | 20 | Release Note Implementation Details | 45 |
| Product | 16 | Release Note Approved | 10 |
| Module | 24 | Release Note Indicator | 10 |
| Work Item Type | 14 | Top Item | 10 |
| | | High Awareness | 14 |

### Filename

```python
from pathlib import Path
downloads = Path.home() / "Downloads"
date_label = release_date.strftime("%Y-%m-%d")  # e.g. 2025-01-01
filename = f"ReleaseNotes_{customer}_{product_safe}_{major_version}_from_{date_label}.xlsx"
output_path = downloads / filename
```

---

## Step 8 — Present the File

Use `present_files` to deliver the file. Then summarize:
- Release Items found and their date range
- Total items and per-release breakdown
- Sheets created
- Any items with `Release Note Approved = 0` — flag these before client delivery

---

## Error Handling

| Situation | Action |
|---|---|
| No Release Items found | Report the date and version queried; ask user to confirm |
| No Bugs/Features linked to a Release Item | Note in summary; skip in output |
| Patch WI not found | Warn; generate file without Patch Label |
| `openpyxl` not installed | Install in bash_tool first |
| Results span multiple major versions | Warn; proceed with what was found |

---

## ADO Reference

| Field | Value |
|---|---|
| Bugs/Features project | `QuorumSoftware` |
| Patch WI project | `QuorumServices` |
| Release date field on Release WI | `Custom.ReleaseDate` |
| Link type (Release → Bug/Feature) | `System.LinkTypes.Related-Forward` |
| Patch version map field | `Custom.ToCoreHotfixVersion` (HTML) |
| Patch source release field | `Custom.SourceRelease` |

---

## Trigger Examples

| User says | What Claude does |
|---|---|
| "Release notes for Daylight, 2023.04, on or after 1/1/2025" | Date-based query, client-facing mode |
| "Upstream Suite 2023.04 with release date on or after January 2025" | Same — `UPS` prefix, parse date |
| "What was in Daylight Patch 8?" | Look up DAY Patch WI, parse ToCoreHotfixVersion for dates, query |
| "Internal release notes for 2024.04 since March 2026" | Skip Patch label, all-columns mode |
| "Release notes for Daylight eSuite 2023.04 from 2/1/2025" | `ESuite` prefix, Esuite sheet |
