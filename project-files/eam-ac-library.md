# EAM AC Library — Wiki Sections, Validation Patterns & Mandatory ACs

> **Version:** 1.3 | **Last Updated:** 2026-05-21
> **Load this file:** When writing ACs that reference EAM wiki sections, when drafting -01 or -02 mandatory ACs, or when you need the standard validation pair templates.
> **What this file covers:** Full wiki anchor reference table (§1–§25), -01 mandatory AC templates, -02 standard validation pair templates, combined grid defaults AC, QA/Parity story structure, business rules phrasing guide.
> **v1.2 changes (2026-05-20):** Section 4 fully rewritten — (1) Classic screen retirement language removed throughout: no retirement bullet in Functional Scope, no retirement or sign-off language in the SME confirmation AC, no "Classic GUI is no longer required" framing anywhere in the template (originated from Asher Maddox, 2026-05-19). (2) QA story framing shifted from Classic parity comparison to web screen behavior verification: "behaves correctly across all functional dimensions", Classic GUI repositioned as an optional side-by-side reference not required for testing. (3) AC persona convention: all AC GWT blocks use "I" (first person); the SME confirmation AC is the only exception, referring to "a revenue accountant SME" as a distinct named role. (4) Standard SME confirmation AC template added as the mandatory closing AC for all QA/Parity stories.

---

## Wiki Section Reference Table

All references link to the **EAM Functionality Reference Wiki** (ADO wiki page 11638).

**Base URL (substitute `{ANCHOR}` at the end):**
```
https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&pageId=11638&anchor={ANCHOR}
```

| § | Section Name | Anchor Value | Use For |
|---|---|---|---|
| 1 | Filter & Query Execution | `1.-filter-%26-query-execution` | All query/filter ACs |
| 2 | Inquiry Date | `2.-inquiry-date` | As-of date filtering |
| 3 | Pagination | `3.-pagination` | Pagination controls |
| 4 | Empty Results State | `4.-empty-results-state` | No-results behavior |
| 5 | Master-Detail Grid (Parent-Child) | `5.-master-detail-grid-(parent-child)` | Parent-child grid ACs |
| 6 | Add Row — New Record Creation | `6.-add-row-%E2%80%94-new-record-creation` | New record creation; **input grid empty editable row** |
| 7 | Delete with Confirmation Dialog | `7.-delete-with-confirmation-dialog` | Delete ACs |
| 8 | Required Field Validation | `8.-required-field-validation` | Required field save-block ACs |
| 9 | Blocking Validation Error | `9.-blocking-validation-error` | Business rule blocking errors |
| 10 | Non-Blocking Warning | `10.-non-blocking-warning` | Warning message ACs |
| 11 | PPN Audit Trail | `11.-ppn-audit-trail` | PPN/audit trail creation |
| 12 | Notes Panel | `12.-notes-panel` | Notes/comments ACs |
| 13 | Confirmation Dialog Before Bulk or Destructive Action | `13.-confirmation-dialog-before-bulk-or-destructive-action` | Bulk/destructive action confirmation |
| 14 | Batch Process Submission | `14.-batch-process-submission` | Batch submission ACs |
| 15 | Action Button State | `15.-action-button-state` | Toolbar button enable/disable |
| 16 | Screen Navigation & Menu Placement | `16.-screen-navigation-%26-menu-placement` | -01 navigation ACs — **mandatory** |
| 17 | Security — Screen-Level Access Control | `17.-security-%E2%80%94-screen-level-access-control` | **Security satellite story only** — never in numbered stories |
| 18 | Context Navigation | `18.-context-navigation` | Cross-screen navigation with context passing |
| 19 | User ID & Timestamp on Every Change | `19.-user-id-%26-timestamp-on-every-change` | Audit timestamp ACs (CRUD stories) |
| 20 | Effective Date Overlap Validation | `20.-effective-date-overlap-validation` | Effective date overlap blocking errors |
| 21 | Export to Excel (All Grids) | `21.-export-to-excel-(all-grids)` | Export button — **mandatory on all grid stories** |
| 22 | Cell Highlight (All Grids) | `22.-cell-highlight-(all-grids)` | Row/cell highlight — **mandatory on all grid stories** |
| 23 | Copy to Clipboard (All Grids) | `23.-copy-to-clipboard-(all-grids)` | Ctrl+C clipboard copy — **mandatory on all grid stories** |
| 24 | Import from Excel Template (Input Grids) | `24.-import-from-excel-template-(input-grids)` | Excel import — **PO-flagged input grids only** |
| 25 | Bulk View/Edit (All Grids) | `25.-bulk-view%2fedit-(all-grids)` | Bulk View/Edit mode — **mandatory on all grid stories** |
| 26 | Retrieve / More / All | `26.-retrieve-%2F-more-%2F-all` | Retrieve/More/All progressive loading — **mandatory -01 AC** |

> **§21–§25 are mandatory on ALL grid stories.** Every story that includes a grid (query, CRUD, child grid, or batch selection) must include the combined §21/§22/§23/§25 AC as the last functional AC. §24 (Import) applies only to PO-flagged input grids.

---

## Section 1: -01 Mandatory AC Templates

Every -01 Foundation + CRUD story must include these four ACs (in this order, before the grid defaults AC).

### AC — Navigation & Menu Placement (§16)

```html
<p><strong>AC1 — Screen navigation and menu placement:</strong></p>
<ul>
  <li>Given I am logged in with access to the [Module Name] module</li>
  <li>When I navigate to [Module Group] in the application navigation menu</li>
  <li>Then the [Screen Display Name] screen is available as a menu item and loads correctly when selected</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=16.-screen-navigation-%2526-menu-placement">Section 16 — Screen Navigation &amp; Menu Placement</a> in the EAM Functionality Reference Wiki.</p>
```

### AC — Query Filter, Results & Empty State (§1 + §4)

```html
<p><strong>AC[N] — Query filter, results grid, and empty state:</strong></p>
<ul>
  <li>Given I am on the [Screen Name] screen</li>
  <li>When I enter one or more filter criteria and execute the query</li>
  <li>Then
    <ul>
      <li>Matching records are displayed in the results grid with all expected columns</li>
      <li>If no records match the filter criteria, a "No records found" message is displayed in the grid area</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=1.-filter-%2526-query-execution">Section 1 — Filter &amp; Query Execution</a> and <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=4.-empty-results-state">Section 4 — Empty Results State</a> in the EAM Functionality Reference Wiki.</p>
```

### AC — Retrieve/More/All (§26)

```html
<p><strong>AC[N] — Retrieve/More/All:</strong></p>
<ul>
  <li>Given the results grid is populated with records</li>
  <li>When I use the Retrieve, More, or All buttons</li>
  <li>Then
    <ul>
      <li>Retrieve loads the first 5,000 matching records into the background cache; the grid displays the first 100 per the default page size (see AC[pagination])</li>
      <li>More appends the next 5,000 records to the background cache (existing rows are not replaced); paging or changing the page size uses the cached records without re-querying</li>
      <li>All loads the complete result set into the background cache regardless of total count</li>
      <li>A record count indicator shows the number of loaded records vs. the total (e.g., "5,000 of 12,340 records loaded")</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=26.-retrieve-%252F-more-%252F-all">Section 26 — Retrieve / More / All</a> in the EAM Functionality Reference Wiki.</p>
```

> **Rules:** (1) Label is "AC[N] — Retrieve/More/All:" — never "Record view, selection, and Retrieve/More/All". (2) When clause is "When I use the Retrieve, More, or All buttons" — never "Record View and Selection controls". (3) No "Selecting a record displays its full detail" bullet — record selection is covered by grid defaults. (4) The "(see AC[N])" reference must point to the pagination AC number for that specific story (AC9 in most -01 stories; adjust if the pagination AC has a different number).

### AC — Pagination & Grid Defaults (§3 + §21/§22/§23/§25) — LAST FUNCTIONAL AC

```html
<p><strong>AC[N] — Pagination and grid interaction defaults (export, highlight, copy, bulk view/edit):</strong></p>
<ul>
  <li>Given records are displayed in the [grid name] grid</li>
  <li>When I interact with the grid and pagination controls</li>
  <li>Then
    <ul>
      <li>The grid displays 100 records per page by default, with a page size dropdown offering options up to 200</li>
      <li>Export to Excel button downloads a .xlsx of all displayed rows</li>
      <li>Clicking a row highlights it; multiple rows within the visible viewport can be highlighted</li>
      <li>Ctrl+C copies highlighted rows to clipboard in tab-delimited format that pastes correctly into Excel with column headers</li>
      <li>Bulk View/Edit button switches the grid into an Excel-like mode with flexible multi-row highlighting and horizontal scrolling; [Exit only — read-only grid / Apply, Apply and Exit, Exit — input grid] button returns to the default grid view</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=3.-pagination">Section 3 — Pagination</a>, <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=21.-export-to-excel-(all-grids)">Section 21 — Export to Excel</a>, <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=22.-cell-highlight-(all-grids)">Section 22 — Cell Highlight</a>, <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=23.-copy-to-clipboard-(all-grids)">Section 23 — Copy to Clipboard</a>, and <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=25.-bulk-view%252fedit-(all-grids)">Section 25 — Bulk View/Edit</a> in the EAM Functionality Reference Wiki.</p>
```

> Substitute `[Exit only — read-only grid]` OR `[Apply, Apply and Exit, Exit — input grid]` based on the grid type. Read-only grids: Exit only. Input grids: Apply / Apply and Exit / Exit.

---

## Section 2: -02 Standard Validation Pair Templates

For -02 Business Rules stories, analyze the screen's `InitializeBusinessRules()` method and produce a complete rule inventory before drafting. Check every registered rule against the six categories below. Include the paired ACs for each category the screen enforces. Omit any category that does not apply.

Each validation category requires BOTH a negative case (invalid input blocked) AND a positive case (valid input accepted). Never include one without the other.

### 1. Required Field Validation

```html
<p><strong>AC[N] — Required fields validated on save (negative):</strong></p>
<ul>
  <li>Given I attempt to save a record with one or more required fields left empty (e.g., [list key required fields for this screen])</li>
  <li>When I click Save</li>
  <li>Then the save is blocked and an error message identifies the missing required field(s)</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=8.-required-field-validation">Section 8 — Required Field Validation</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — All required fields present allows save (positive):</strong></p>
<ul>
  <li>Given I have populated all required fields on a new record</li>
  <li>When I save</li>
  <li>Then the record is saved successfully and the grid refreshes showing the new record with all values intact</li>
</ul>
```

### 2. Entity / Reference Existence Validation

When a field references another entity that must exist (e.g., Account Number must exist in Account Setup, Remitter must exist as a Business Associate):

```html
<p><strong>AC[N] — [Entity] existence validated on save (negative):</strong></p>
<ul>
  <li>Given I enter a [Entity Name] that does not exist in [Source Screen/Setup]</li>
  <li>When I save the record</li>
  <li>Then the save is blocked and an error message states that the [Entity Name] is not recognized and must exist in [Source Screen/Setup]</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=9.-blocking-validation-error">Section 9 — Blocking Validation Error</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — Valid [Entity] is accepted (positive):</strong></p>
<ul>
  <li>Given I enter a [Entity Name] that exists in [Source Screen/Setup]</li>
  <li>When I save the record with all other required fields populated</li>
  <li>Then the record is saved successfully with no existence error</li>
</ul>
```

### 3. Duplicate Prevention

When a screen enforces uniqueness on a combination of fields:

```html
<p><strong>AC[N] — Duplicate [entity/combination] is blocked (negative):</strong></p>
<ul>
  <li>Given a record already exists for [describe the unique key combination — e.g., "the same Property, Owner, and Effective Date"]</li>
  <li>When I attempt to create a second record with the same [key combination]</li>
  <li>Then the save is blocked and an error message states that a [entity] for this combination already exists</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=9.-blocking-validation-error">Section 9 — Blocking Validation Error</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — Unique [entity/combination] is accepted (positive):</strong></p>
<ul>
  <li>Given no record exists for the entered [key combination]</li>
  <li>When I save the record with all required fields populated</li>
  <li>Then the record is saved successfully with no duplicate-prevention error</li>
</ul>
```

### 4. Pick List Value Integrity

When a screen has fields backed by code table pick lists:

```html
<p><strong>AC[N] — Invalid pick list value is blocked (negative):</strong></p>
<ul>
  <li>Given I enter a value in a pick list field (e.g., [field name]) that is not a valid code table option</li>
  <li>When I save the record</li>
  <li>Then the save is blocked and an error message identifies the field and states that the value must be selected from the available options</li>
</ul>

<p><strong>AC[N+1] — Valid pick list value is accepted (positive):</strong></p>
<ul>
  <li>Given all pick list fields contain valid code table values</li>
  <li>When I save the record</li>
  <li>Then the record is saved successfully with no pick list validation error</li>
</ul>
```

### 5. Conditional Required Field Validation

When a field is only required under a specific condition (e.g., Well/Completion required only when Calculate Severance Tax = Yes, a secondary field required only when a flag is set). Write one AC pair per condition. The negative case must state the condition that activates the requirement; the positive case must confirm the field is not required when the condition is absent.

```html
<p><strong>AC[N] — [Field] required when [condition] (negative):</strong></p>
<ul>
  <li>Given [the condition that activates the requirement — e.g., "Calculate Severance Tax is set to Yes"] and [the conditionally required field] is not provided</li>
  <li>When I attempt to save</li>
  <li>Then a validation error states that [field name] is required when [condition], and the save is blocked</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=9.-blocking-validation-error">Section 9 — Blocking Validation Error</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — [Field] not required when [condition is absent] (positive):</strong></p>
<ul>
  <li>Given [the condition is not present — e.g., "Calculate Severance Tax is set to No"] and [the conditionally required field] is not provided</li>
  <li>When I save the record with all unconditionally required fields populated</li>
  <li>Then the record is saved successfully without requiring [field name]</li>
</ul>
```

> **When to use:** Any time the source has a business rule that reads a flag or field value before enforcing a requirement — common patterns include flag-gated fields, type-specific required fields, and configuration-driven requirements (e.g., `BOOK_REV_WELL_COMPL` config key). Do NOT collapse these into the generic Required Field template — the conditional trigger is load-bearing for test writability.

### 6. Delete Guard — Referential Integrity

When a screen blocks deletion of a record that has dependent child records in another table (e.g., MI Links cannot be deleted while Manual Inputs exist, an Account Group cannot be deleted while Accounts reference it). Write one AC pair: delete blocked when dependents exist (negative), delete succeeds when no dependents exist (positive).

```html
<p><strong>AC[N] — Delete blocked when [dependent records] exist (negative):</strong></p>
<ul>
  <li>Given I select a [record type] that has one or more associated [dependent record type] records</li>
  <li>When I attempt to delete the record</li>
  <li>Then a validation error states that the [record type] cannot be deleted because [dependent record type] records still exist, and the deletion is blocked until those records are removed</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=9.-blocking-validation-error">Section 9 — Blocking Validation Error</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — Delete succeeds when no [dependent records] exist (positive):</strong></p>
<ul>
  <li>Given I select a [record type] that has no associated [dependent record type] records</li>
  <li>When I delete the record and confirm the deletion</li>
  <li>Then the record is deleted successfully</li>
</ul>
```

> **When to use:** Any time the source registers a rule against `QBusinessRuleTriggerEvent.StartingUpdate` or `BeforeDelete` that queries a child table before allowing the delete to proceed. Common in parent-child screens throughout QRA. Always check for this pattern during the business rule inventory — it is frequently implemented but easy to overlook because it does not fire on save.

---

## Section 2b: Post-Save System Sync Operations

Some screens trigger automatic system-managed operations after a successful save or delete — these are not blocking validations and produce no UI feedback unless they fail. They are invisible to the user during normal operation but represent real business logic that must be migrated. They are easy to miss because they don't appear in the UI and are not registered as standard business rules.

**How to identify them:** In the source, look for calls to `DataAccessHelper.ExecuteNonQuery(...)` inside business rule delegate methods, particularly those that fire on `BeforeUpdate` or `BeforeCommit` and check `AllEvaluatedRulesSuccessful` before executing. Also look for `Controller_OverrideUpdateData` overrides, which often contain cascade deletes on child tables.

**Common patterns in QRA:**
- SOD link sync when MI Link effective dates change (`UPDATE_RXRF_MI_SOD_FROM_MI_EFF_DT`)
- SOD link removal when MI Link records are deleted (`DELETE_RXRF_MI_SOD_FROM_MI_EFF_DT`)
- Cascade deletion of child VLA MI Link records when a parent is deleted (`DELETE_CHILD_VLA_MI_LINK_RECORDS`)
- PPN audit trail creation on insert, modify, and delete (`INSERT_PPN_VL025`)

**When to include in a story:** Any time the codebase review reveals a sync operation tied to a user action on the screen. These belong in the same story as the action that triggers them (save or delete), not in a separate technical story. If the sync is tied to a delete action, it belongs alongside the delete guard AC pair.

**Standard AC template for sync operations (use one set per sync operation):**

```html
<p><strong>AC[N] — [Sync target] updated when [trigger condition]:</strong></p>
<ul>
  <li>Given [precondition — e.g., "an existing record with associated [sync target] records"]</li>
  <li>When [the triggering user action — e.g., "I modify the Effective Date From or Effective Date To and save the record"]</li>
  <li>Then
    <ul>
      <li>The system updates the corresponding [sync target] records to reflect the change</li>
      <li>[Sync target] records associated with other [parent record type] records are not affected</li>
    </ul>
  </li>
</ul>

<p><strong>AC[N+1] — [Sync target] NOT updated when trigger condition is absent:</strong></p>
<ul>
  <li>Given [precondition — e.g., "an existing record with associated [sync target] records"]</li>
  <li>When [a similar action that does NOT trigger the sync — e.g., "I save the record after modifying any field other than the effective dates"]</li>
  <li>Then the sync does not execute and the existing [sync target] records are unchanged</li>
</ul>

<p><strong>AC[N+2] — [Sync target] removed when [delete trigger]:</strong></p>
<ul>
  <li>Given [precondition — e.g., "an existing record with associated [sync target] records and no blocking dependents"]</li>
  <li>When I delete the record and confirm the deletion</li>
  <li>Then
    <ul>
      <li>All [sync target] records associated with the deleted record are also removed</li>
      <li>[Sync target] records belonging to other records are not affected</li>
    </ul>
  </li>
</ul>

<p><strong>AC[N+3] — Sync failure surfaces as a user-visible error:</strong></p>
<ul>
  <li>Given a sync operation fails due to a system error</li>
  <li>When the failure occurs during save or delete</li>
  <li>Then the system surfaces a clear error message and does not leave the record in a partially saved state</li>
</ul>
```

> **Note:** Omit AC[N+2] if the sync operation has no delete path. Omit AC[N+1] if the sync fires unconditionally on every save rather than only under a specific condition. Always include the failure AC — silent failures in sync operations are high-risk for downstream revenue processing.


---

## Section 3: Combined Grid Defaults AC (All Grid Stories)

Place this as the **last functional AC** in every story that contains a grid. Adjust the Bulk View/Edit closing options based on grid type.

```html
<p><strong>AC[N] — Grid interaction defaults (export, highlight, copy, bulk view/edit):</strong></p>
<ul>
  <li>Given records are displayed in the [grid name] grid</li>
  <li>When I interact with the grid</li>
  <li>Then the following standard grid behaviors are available:
    <ul>
      <li>Export to Excel button downloads a .xlsx of all displayed rows</li>
      <li>Clicking a row highlights it; multiple rows within the visible viewport can be highlighted</li>
      <li>Ctrl+C copies highlighted rows to clipboard in tab-delimited format that pastes correctly into Excel with column headers</li>
      <li>Bulk View/Edit button switches the grid into an Excel-like mode with flexible multi-row highlighting and horizontal scrolling; [Exit only — read-only grid / Apply, Apply and Exit, Exit — input grid] button returns to the default grid view</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=21.-export-to-excel-(all-grids)">Section 21 — Export to Excel</a>, <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=22.-cell-highlight-(all-grids)">Section 22 — Cell Highlight</a>, <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=23.-copy-to-clipboard-(all-grids)">Section 23 — Copy to Clipboard</a>, and <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=25.-bulk-view%252fedit-(all-grids)">Section 25 — Bulk View/Edit</a> in the EAM Functionality Reference Wiki.</p>
```

**Bulk View/Edit closing option rule:**
- **Read-only grid:** `Exit only` — the grid cannot be edited in Bulk View/Edit mode
- **Input grid:** `Apply, Apply and Exit, Exit` — changes can be applied inline

---

## Section 4: QA / Parity Story Structure

The last numbered story for every screen. Uses the `QA analyst` persona.

**Description template:**

```html
<h2>User Story</h2>
<p>As a <strong>QA analyst</strong>, I want to verify that the web [Screen Name] screen behaves correctly across all functional dimensions so that revenue accountants can confidently adopt the new screen.</p>

<h2>Description</h2>
<p>[2–3 sentences describing the verification scope — which functional areas are being tested on the web screen. Classic GUI may be referenced as a comparison guide but is not itself under test.]</p>
<p><strong>Legacy reference:</strong> [Screen Name (ScreenID)]</p>

<h3>Functional Scope</h3>
<ul>
  <li>Structured verification of web screen behavior across all functional areas covered by this screen's story sequence; Classic GUI may be used as a side-by-side reference but is not required to be executed as part of testing</li>
</ul>

<h3>Out of Scope</h3>
<ul>
  <li>New capabilities not present in Classic GUI (covered in the relevant functional story)</li>
  <li>Performance benchmarking</li>
</ul>
```

**ACs for QA/Parity stories** must be screen-specific functional verifications of web screen behavior grouped by functional dimension — not generic boilerplate. Write one AC per major functional area (e.g., "Query and filter behavior", "Record CRUD behavior", "Business rule enforcement"). Each AC describes the expected behavior of the web screen — not a comparison to Classic GUI. Classic GUI may be referenced informally as a guide during testing but must not appear as a required test step or environment in any AC.

### AC Persona Convention (mandatory)

**All AC GWT blocks use first person ("I") — never a named persona.**

- ✅ `Given I am on the [Screen Name] web screen`
- ✅ `When I execute a query with valid filter criteria`
- ✅ `Given I intentionally submit an invalid configuration`
- ❌ `Given a QA analyst intentionally submits an invalid configuration`
- ❌ `When a revenue accountant enters a valid Subledger Number`

The persona is established by the User Story statement (`As a QA analyst…`) and does not need to be restated in each AC. Using "I" keeps ACs concise and consistent across all QA stories.

**Exception — SME confirmation AC:** The closing SME confirmation AC refers to `a revenue accountant SME` because it describes a distinct named role performing a sign-off action, not the tester executing the step. See the standard template below.

### Standard SME Confirmation AC (mandatory closing AC)

Every QA/Parity story must end with this AC as the final item in the acceptance criteria.

```html
<p><strong>AC[N] — SME confirmation:</strong></p>
<ul>
  <li>Given all functional dimensions above have passed QA verification</li>
  <li>When a revenue accountant SME performs a structured walkthrough of [ScreenID] in the web environment</li>
  <li>Then the SME confirms the screen behaves as expected</li>
</ul>
```

> **Rules:** (1) This is always the last AC — no functional ACs follow it. (2) The Then clause is intentionally brief — do not add "and provides formal sign-off that Classic GUI is no longer required" or any retirement language. (3) The "revenue accountant SME" reference is the one permitted exception to the "I" persona rule above.

---

## Section 5: Business Rules Phrasing Guide

When translating legacy codebase findings into story language:

### Anti-Pattern Corrections

| ❌ Technical (from code) | ✅ Business Language (for stories) |
|---|---|
| "When `MI_FLAG_LOCK` is set, prevent editing" | "When a record is locked for final approval, users cannot edit it" |
| "Call `QBusinessRuleSODValidateContract()` to check contract" | "Verify that the SOD contract exists and is in an approvable status" |
| "Query `RXRF_MI_SOD` filtered by ManualInputSeqNo" | "Find all linkages for this record" |
| "UpdateDate must be > CreateDate AND < GETDATE()" | "Effective dates must be in logical order and cannot be in the future" |
| "Cascade delete child rows from TPOLN_ table" | "When a master link is deleted, all child records associated with it are also removed" |
| "Check the 'APPROVED' flag in the header before allowing insert" | "Only create new records when the parent contract has been approved" |
| "Effective date ranges must not overlap per RXRF_MI_SOD.ContractGroup + DRI" | "For the same SOD contract and DRI combination, effective date ranges cannot overlap" |
| "`UpdateUser` and `UpdateDate` are populated" | "The system records who made the change and when" |

### Code-to-Business Translation Examples

**Code:**
```csharp
if (!Remitter.IsApproved())
    throw new ValidationException("Remitter must have ApprovedFlag = Y");
```
**Business rule:** "Remitter must be an approved Business Associate"
**AC phrasing:**
> Given a remitter that is not approved / When I attempt to create a new record with that remitter / Then an error states "Remitter must be approved" and the save is blocked

---

**Code:**
```csharp
if (Record.EffectiveFromDate >= Record.EffectiveToDate)
    throw new ValidationException("Invalid date range");
```
**Business rule:** "Effective To Date must be after Effective From Date"

---

**Code:**
```csharp
if (Contract.ApprovalStatus != "APPROVED")
    return false;
```
**Business rule:** "The SOD contract must be approved before records can be linked to it"

### When to Include a Business Rules Section in Description

Include `<h3>Business Rules</h3>` in the Description only when:
- The story is explicitly a business rules story (-02 or a BR-focused special story)
- The screen has validation/guard conditions that constrain how data is entered
- Users need to understand constraints before attempting to use the feature

Do NOT include for Foundation+CRUD stories (-01), Notes, Audit History, or QA stories.

---

## Section 6: Audit History Story Structure

Every Audit History story follows this complete standard (established from JE005-04 #1804738 and refined from JE020-03 and JE025-03).

### Title Convention
```
[ScreenID]-[Seq]: Audit History — Change History & Deleted Records Viewer
```
Always use this exact subtitle regardless of screen. Do not vary it.

### Persona
`revenue accountant or accounting administrator`

### Two-Tab Structure (mandatory)
The web Audit History screen always has two tabs:
1. **Change History tab** — scoped to the currently selected record; shows all field-level changes made to that record
2. **Delete History tab** — entity-wide; returns ALL deleted records for that entity type across all users; NOT scoped to a selected record; user does not need to have a record selected

In Classic GUI these were two separate Links menu options. In the web, they are unified as two tabs on the same Audit History screen.

### Actual Grid Columns (from web implementation)
| Column | Change History | Delete History |
|---|---|---|
| **Update Date** | Date/time of change | Date/time of deletion |
| **Modification Item** | Field that was changed | "Deleted [Field Name]" |
| **Update User** | User who made the change | User who deleted the record |
| **Modified From** | Previous value | Value at time of deletion |
| **Modified To** | New value | **Empty** — no new value after deletion |

### Description Requirements
1. Classic GUI access pattern: Links menu → Change History AND Links menu → Delete History — two separate options in Classic, consolidated in web
2. Web consolidation: single View Audit History action → dedicated screen with two tabs
3. Whether the screen is single-entity (flat list) or multi-entity (parent + child, needs Record/Entity discriminator column)
4. The two-tab behavioral distinction: Change History is record-scoped; Delete History is entity-wide
5. History key label (from `DisplayKeyLabel` in `InitializeHistory()` in source)

**No Business Rules section** — security is covered by the -Security satellite story.

**Out of Scope bullets:** no modify/purge, no other screens, CRUD in -01, security in -Security. Do NOT include "export deferred" — export is covered by the grid defaults AC.

### Source Fields to Look Up (InitializeHistory)
- `AuditObjectID` — confirms the audit trail is registered
- `HistoryStorageTable` — typically JHIST_AUDIT for JE screens
- `TableList` entries — one entry = single entity; multiple entries = multi-entity (may need Record/Entity column)
- `DisplayKeyLabel` — field label used in the Audit History screen title
- `DisplayKeyBindDataMember` — confirms which field is the history key

### 8 ACs (mandatory structure)

**AC1 — View Audit History action:**
- Given: record selected on the screen
- When: open the Actions menu
- Then: View Audit History option is available, consistent with the audit history action pattern on other migrated web screens

**AC2 — Navigation to Audit History screen with two tabs:**
- Given: record selected
- When: select View Audit History
- Then (2 bullets): navigated to Audit History screen AND screen title confirms the key value (e.g., Account Group Number, Event) AND two tabs present: Change History and Delete History

**AC3 — Change History tab — record-scoped content:**
- Given: on Audit History screen, select Change History tab
- When: tab loads
- Then (single inline): read-only grid displays all field-level changes for selected record, with columns for: Update Date, Modification Item, Update User, Modified From, Modified To

**AC4 — Delete History tab — entity-wide content:**
- Given: on Audit History screen, select Delete History tab
- When: tab loads
- Then (4 bullets): grid shows all deleted records NOT scoped to selected record; grid columns named; Modification Item prefixed with "Deleted"; Modified To empty for all rows

**AC5 — Grid filtering (both tabs):**
- Given: viewing either grid
- When: apply filter (by user, date range, modification item)
- Then (single inline): grid refreshes to show only matching records

**AC6 — Read-only enforcement:**
- Given: on Audit History screen on either tab
- When: attempt to interact with any row
- Then (single inline): no add, edit, or delete actions available — both tabs are strictly read-only

**AC7 — No records state (both tabs):**
- Given: navigate to Audit History for a record with no change events, or Delete History has no deleted records
- When: relevant tab loads
- Then (single inline): clear informational message displayed indicating no history exists for that tab, rather than a blank or broken grid

**AC8 — Pagination and grid interaction defaults (export, highlight, copy, bulk view/edit):**
- Given: records displayed in either the Change History or Delete History grid
- When: interact with the grid and pagination controls
- Then (5 bullets — verbatim standard template):
  - "The grid displays 100 records per page by default, with a page size dropdown offering options up to 200"
  - "Export to Excel button downloads a .xlsx of all displayed rows"
  - "Clicking a row highlights it; multiple rows within the visible viewport can be highlighted"
  - "Ctrl+C copies highlighted rows to clipboard in tab-delimited format that pastes correctly into Excel with column headers"
  - "Bulk View/Edit button switches the grid into an Excel-like mode with flexible multi-row highlighting and horizontal scrolling; Exit button returns to the default grid view" *(Exit only — these are read-only grids)*
- Reference: §3, §21, §22, §23, §25

> **No security AC** in Audit History stories. Security is always in the -Security satellite.
> **No §17 reference** in Audit History stories.

---

## Section 7: Security Story Structure

Every Security satellite story follows this complete standard (established from JE020-Security).

### Persona
`system administrator`

### Security Model Language
Do NOT use the term "QARCH" anywhere — it is an implementation detail with no place in PO-voice content. Do NOT use the term "role" — the correct term is **security group**.

The correct model: **Users → assigned to security groups → security groups have privileges → privileges control access to the screen's registered security object**

### Description Requirements
Explain the platform's security framework in PO voice:
- Users are assigned to one or more security groups
- Each security group is configured with specific privileges on the screen's registered security object
- Those privileges determine: (1) whether the screen appears in the navigation menu at all, (2) whether the user can open and query it, and (3) which operations are available once inside
- Users not assigned to any security group that grants access will not see the screen in the navigation menu — the menu item is suppressed by the platform

### Functional Scope — 4 Privilege Tiers (mandatory)
- **Navigation visibility** — screen appears in the module navigation menu only for users assigned to a security group with the required screen access privilege; users not assigned to any such security group do not see the screen as a menu option
- **Screen-level access** — users assigned to a security group with screen access privilege can open and interact with the screen
- **Read-only access** — users assigned to a security group with view-only privilege can query and view records; CRUD toolbar controls are hidden or disabled
- **Full access** — users assigned to a security group with write and delete privileges can perform all CRUD operations
- **No access** — users not assigned to any security group that grants access do not see the screen as a menu option; the screen is not accessible to them in any way

### 4 ACs (mandatory — not 3)

**AC1 — Authorized user sees screen in navigation menu:**
- Given: logged in and assigned to a security group that grants the required access privilege for this screen
- When: navigate to the module group in the application navigation menu
- Then: screen is listed as a menu item and loads correctly when selected
- Reference: §16

**AC2 — Authorized user opens screen with full access:**
- Given: logged in and assigned to a security group with full access privileges
- When: open the screen
- Then: screen loads and all authorized functions are available, including query, create, edit, and delete
- Reference: §17

**AC3 — View-only user can view but not modify:**
- Given: logged in and assigned to a security group with view-only access
- When: open the screen
- Then (3 bullets): can query and view records; create/edit/delete controls hidden or disabled; any attempt to modify data is blocked
- Reference: §17

**AC4 — User with no access does not see screen in menu:**
- Given: logged in and NOT assigned to any security group that grants access to this screen
- When: navigate to the module group in the application navigation menu
- Then: screen does NOT appear as a menu option — the item is not visible and the screen is not accessible
- **NEVER say "access-denied message" or "redirect"** — the platform suppresses the menu item entirely before the user can navigate to it
- Reference: §17

### Wiki Link Formats for Security ACs
- **§16:** `anchor=16.-screen-navigation-%2526-menu-placement` (double-encoded — correct and tested)
- **§17:** `anchor=17.-security-%E2%80%94-screen-level-access-control` (single-encoded em dash — do NOT double-encode to `%25E2%2580%2594`)