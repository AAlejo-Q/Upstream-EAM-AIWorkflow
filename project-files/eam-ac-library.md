# EAM AC Library — Wiki Sections, Validation Patterns & Mandatory ACs

> **Version:** 1.4 | **Last Updated:** 2026-06-03
> **Load this file:** When writing ACs that reference EAM wiki sections, when drafting -01 or -02 mandatory ACs, or when you need the standard validation pair templates.
> **What this file covers:** Full wiki anchor reference table (§1–§25), -01 mandatory AC templates, -02 standard validation pair templates, combined grid defaults AC, QA/Parity story structure, business rules phrasing guide.
> **v1.4 changes (2026-06-03):** §24 Import from Excel is now mandatory on all input grid stories (previously PO-flagged only). Updated Section 1 and Section 3 grid defaults AC templates to include Import bullet and §24 reference link. Omit §24 only for read-only grids.
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
| 24 | Import from Excel Template (Input Grids) | `24.-import-from-excel-template-(input-grids)` | Excel import — **mandatory on all input grid stories** |
| 25 | Bulk View/Edit (All Grids) | `25.-bulk-view%2fedit-(all-grids)` | Bulk View/Edit mode — **mandatory on all grid stories** |
| 26 | Retrieve / More / All | `26.-retrieve-%2F-more-%2F-all` | Retrieve/More/All progressive loading — **mandatory -01 AC** |

> **§21–§25 are mandatory on ALL grid stories.** Every story that includes a grid (query, CRUD, child grid, or batch selection) must include the combined §21/§22/§23/§24/§25 AC as the last functional AC. §24 (Import) is mandatory on all input grid stories; omit §24 only for read-only grids.