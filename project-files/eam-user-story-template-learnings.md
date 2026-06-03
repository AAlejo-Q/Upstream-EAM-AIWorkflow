# Learnings: eam-user-story-template
> Target skill: `/mnt/skills/user/eam-user-story-template/SKILL.md`
> Created: 2026-04-21
> Last promoted: Never (initial seed)
> Last updated: 2026-05-21

---

## Pending Learnings
*(Applied as overlays each session until promoted into the skill)*

### L001 — 2026-04-21 — [PENDING]
**Type:** CONSOLIDATION
**Context:** VL025 revision — Notes & Contextual Navigation story was originally standalone at 3 SP
**Learning:** When a Notes or Contextual Navigation story is ≤3 SP AND shares row-selection context with the CRUD story on the same screen, merge it into the CRUD story rather than leaving it standalone. The combined story must not exceed 8 SP. If the merge would push past 8 SP, keep it separate.
**Example:** VL025 Story 09 (Notes + nav link to VL031, 3 SP) merged into Story 03 CRUD → stays at 8 SP
**Source:** VL025 revision session
**Applies to:** Section 10 (Consolidation Heuristic)

---

### L002 — 2026-04-21 — [PENDING]
**Type:** CONSOLIDATION
**Context:** VL025 revision — Auto-Population (5 SP) and Core Validation (8 SP) were separate stories
**Learning:** When Auto-Population and Core Business Rules/Validation fire in the same service-layer pipeline (Save / AfterInsert / AfterUpdate), merge them into a single Business Rules story. Re-estimate auto-pop contribution; combined total should not exceed 8 SP. If it would, keep Advanced Rules separate with a 🔴 High risk flag.
**Example:** VL025 Stories 04+05 merged → single Business Rules story at 8 SP
**Source:** VL025 revision session
**Applies to:** Section 10 (Consolidation Heuristic), Section 3 (Decomposition Guide)

---

### L003 — 2026-04-21 — [PENDING]
**Type:** DECOMPOSITION
**Context:** VL025 revision — Dynamic UX story was generated but not applicable
**Learning:** Do NOT generate a Dynamic Read-Only / Dynamic UX story for screens that have no product-code-driven or state-driven conditional field enabling. Before drafting this story, verify in the legacy ClassicGUI source that conditional field enabling actually exists. If not present, skip this story type entirely.
**Example:** VL025 has no product-code-driven field enabling → Dynamic UX story eliminated
**Source:** VL025 revision session
**Applies to:** Section 3, Track A sequence — add conditional check before including this story type

---

### L004 — 2026-04-21 — [PENDING]
**Type:** DECOMPOSITION
**Context:** VL025 revision — Detail View story generated but not applicable for pure inline-edit grids
**Learning:** Do NOT generate a Detail View story for screens that are pure inline-edit grid screens with no separate form or panel. Before drafting this story, verify whether the legacy screen has a dedicated detail form/panel. If the screen only edits rows inline in the grid, skip Detail View.
**Example:** VL025 is a pure inline-edit grid — no Detail View story
**Source:** VL025 revision session
**Applies to:** Section 3, Track A — add applicability check

---

### L005 — 2026-04-21 — [PENDING]
**Type:** AC_QUALITY
**Context:** QA/Parity stories — ACs were too generic in early drafts
**Learning:** QA/Parity story Acceptance Criteria must include screen-specific functional verifications, not generic "verify the screen works" language. Each AC should name a specific business function, field behavior, or workflow that can be directly traced to a prior story in the same screen's story map. Reference the exemplars VL031-09 (1792455), VL025-10 (1793069), JE100-08 (1800111), JE102-05 (1800117) for the correct specificity level.
**Example:** BAD: "Verify CRUD operations work as expected." GOOD: "Verify that saving a DRI Link record with an overlapping effective date triggers the date-range validation and prevents save."
**Source:** Multiple story review sessions
**Applies to:** Section 6 (AC Template), QA Story pattern

---

### L006 — 2026-04-21 — [PENDING]
**Type:** TEMPLATE
**Context:** Work Item Type was set incorrectly as "User Story" in early ADO creation attempts
**Learning:** The ADO Work Item Type for child stories in the Quorum project is **Requirement**, not "User Story". Always use `"User Story"` → `"Requirement"` when calling the ADO REST API for story-level items. Features use `wit_create_work_item`; Requirements (stories) use `wit_add_child_work_items` with the parent Feature ID.
**Source:** ADO creation session — type mismatch discovered
**Applies to:** Section 12 (ADO Creation), Appendix C

---

### L007 — 2026-04-21 — [PENDING]
**Type:** WORKFLOW
**Context:** Bias calibration — early story maps had too many granular stories
**Learning:** Default bias should be toward **fewer, broader stories**. When in doubt between splitting and consolidating, consolidate. A story map that is 6–8 stories for a Lift and Shift screen is better than 10–12. Only split when: (a) SP would exceed 8, (b) stories are not independently deliverable, or (c) risk levels differ significantly between the bundled concerns.
**Source:** Ongoing calibration across VL031, VL025, JE100, JE102
**Applies to:** Section B (Decomposition), Section 10 (Consolidation Heuristic)

---

### L019 — 2026-05-15 — [PENDING]
**Type:** TEMPLATE
**Context:** JE020-03 was a 3-AC skeleton. JE005-04 (#1804738) is the established story standard. Screenshots of the actual web Audit History screen revealed a two-tab structure and specific column names not previously captured.
**Learning:** Every Audit History story must follow this complete standard:

**Title:** `[ScreenID]-[Seq]: Audit History — Change History & Deleted Records Viewer`
(Consistent with JE005-04 — always use this exact subtitle regardless of screen)

**Persona:** `revenue accountant or accounting administrator`

**Two-tab structure (mandatory in all Audit History stories):**
The web Audit History screen always has two tabs:
1. **Change History tab** — scoped to the currently selected record; shows all field-level changes made to that record. User must have a record selected before navigating to Audit History.
2. **Delete History tab** — returns ALL deleted records for that entity type across all users; NOT scoped to a selected record. The user does not need to have a record selected.

In Classic GUI, these were two separate Links menu options ("Change History" and "Delete History" or "Deleted History"). In the web, they are unified as two tabs on the same Audit History screen.

**Actual grid columns (from web implementation):**
- **Update Date** — date and time the change or deletion occurred
- **Modification Item** — the field that was changed (Change History) or "Deleted [Field Name]" (Delete History)
- **Update User** — the user who made the change or deletion
- **Modified From** — the previous value (Change History) or the value at time of deletion (Delete History)
- **Modified To** — the new value (Change History); **empty** for Delete History (no new value after deletion)

**Description must include:**
1. Classic GUI access pattern: Links menu → Change History AND Links menu → Delete History (or Deleted History) — two separate options in Classic, consolidated in web
2. Web consolidation: single View Audit History action → dedicated screen with two tabs
3. Whether the screen is single-entity or multi-entity (affects whether a "Record / Entity" discriminator column is needed; most JE screens are single-entity)
4. The two-tab behavioral distinction: Change History is record-scoped; Delete History is entity-wide
5. History key label (from `DisplayKeyLabel` in `InitializeHistory()` in source)

**No Business Rules section in Description** — security access to the Audit History screen is covered by the -Security satellite story, same as all other numbered stories. Do not add a Business Rules section to Audit History story Descriptions.

**Out of Scope:** no modify/purge, no other screens, CRUD in -01, security in -Security. Do NOT include an "export deferred" bullet — export is covered by the grid defaults AC (AC8).

**8 ACs (standard structure):**
- AC1: View Audit History action available in Actions menu for a selected record
- AC2: Navigation to Audit History screen — confirms title (with key value) and both tabs (Change History and Delete History) are present
- AC3: Change History tab — record-scoped; grid shows field-level changes for the selected record; columns: Update Date, Modification Item, Update User, Modified From, Modified To
- AC4: Delete History tab — entity-wide (no record selection required); grid shows all deleted records; columns same as above; Modified To column is empty for all rows
- AC5: Grid filtering (both tabs) — filter by user, date range, modification item; grid refreshes to matching results. **Pagination is NOT included here.**
- AC6: Read-only enforcement (single Then — inline)
- AC7: No-records state — applies to both tabs individually (informational message, not blank)
- AC8: **Pagination and grid interaction defaults** (§3 + §21/§22/§23/§25) — always the last AC; use the exact standard template bullets verbatim:
  - "The grid displays 100 records per page by default, with a page size dropdown offering options up to 200"
  - "Export to Excel button downloads a .xlsx of all displayed rows"
  - "Clicking a row highlights it; multiple rows within the visible viewport can be highlighted"
  - "Ctrl+C copies highlighted rows to clipboard in tab-delimited format that pastes correctly into Excel with column headers"
  - Read-only grid: "Bulk View/Edit button switches the grid into an Excel-like mode with flexible multi-row highlighting and horizontal scrolling; Exit button returns to the default grid view"
  - Input grid: "...; Apply, Apply and Exit, and Exit buttons return to the default grid view"

**CRITICAL: Pagination is ALWAYS combined with grid defaults (§21/§22/§23/§25) into one AC. Never separate them. Never paraphrase or simplify the standard bullets — use them verbatim. Applies to all story types — Audit History, CRUD, and everything else. AC5 covers content-specific filtering only; AC8 covers pagination + all grid interaction defaults together.**

**No security AC** — security is always covered in the -Security satellite story. Never include a security AC in Audit History stories (or any other numbered story).

**§3 + §21/§22/§23/§25 combined grid defaults AC IS required** for Audit History stories. Use "Exit only" for the Bulk View/Edit closing option since both grids are read-only.

**Source details to look up in InitializeHistory():**
- `AuditObjectID` — confirms the audit trail is registered
- `HistoryStorageTable` — typically JHIST_AUDIT for JE screens
- `TableList` entries — one entry = single entity; multiple entries = multi-entity (may need Record/Entity column)
- `DisplayKeyLabel` — field label used in the Audit History screen title (e.g., "Account Group No.", "Event")
- `DisplayKeyBindDataMember` — confirms which field is the history key

**Example (single-entity):** JE020 — one table tracked (JXRF_ACCT_ACCT_GRP), keyed by Account Group No. Change History scoped to selected xref record; Delete History returns all deleted xref records.
**Example (multi-entity):** JE005-04 — header + column assignments tracked. Record/Entity column present to distinguish which sub-entity each row belongs to.

**Source:** JE020-03 revision + web screenshot analysis (two-tab structure, actual column names) — 2026-05-15
**Applies to:** All Audit History stories (-03 or equivalent special slot) across all EAM screens

---

### L017 — 2026-05-15 — [PENDING]
**Type:** AC_QUALITY
**Context:** JE020-01 revision — AC3 "Record view, selection, and Retrieve/More/All" had a "Selecting a record" bullet and a "Record View and Selection controls" When clause that were removed as redundant
**Learning:** The §26 Retrieve/More/All AC must NOT include a "Selecting a record displays its full detail" bullet or a "Record View and Selection controls" When clause. Record selection behavior is already covered by grid default behavior. The correct AC3 pattern is:
- Label: "AC[N] — Retrieve/More/All:"
- When: "When I use the Retrieve, More, or All buttons"
- Then bullets:
  1. Retrieve loads the first 5,000 matching records into the background cache; the grid displays the first 100 per the default page size (see AC[pagination])
  2. More appends the next 5,000 records to the background cache (existing rows are not replaced); paging or changing the page size uses the cached records without re-querying
  3. All loads the complete result set into the background cache regardless of total count
  4. A record count indicator shows the number of loaded records vs. the total (e.g., "5,000 of 12,340 records loaded")
The "see AC[N]" reference must point to the pagination AC for that story (AC9 in most -01 stories; adjust if pagination AC has a different number).
**Source:** Aaron's revision of JE020-01 AC3 — 2026-05-15
**Applies to:** eam-ac-library.md — §26 Retrieve/More/All AC template; eam-story-core.md — -01 mandatory AC list

---

### L018 — 2026-05-15 — [PENDING]
**Type:** TEMPLATE
**Context:** JE020 story set revision — all instances of "Cross-Reference" in screen names needed updating
**Learning:** For screens whose names include "Cross-Reference", use "Xref" in all story content (Description, ACs, titles). This is the project naming convention for screens with "cross-reference" in their name. Apply to: story User Story statement, Description context paragraph, Legacy reference line, Functional Scope bullets, Out of Scope bullets, and all AC Given/When/Then text. Case usage: "Xref" (capital X) when used as a proper noun or screen name (e.g., "Account/Account Group Xref screen"); "xref" (lowercase) when used as a common noun (e.g., "xref records", "an xref record", "xref associations").
**Example:** "Account/Account Group Cross-Reference screen" → "Account/Account Group Xref screen"; "cross-reference records" → "xref records"; "a cross-reference record" → "an xref record"
**Source:** Aaron's naming convention instruction — 2026-05-15
**Applies to:** All EAM story content for screens with "cross-reference" in the name (JE020, and any future screens with xref in their name)

---

### L016 — 2026-05-15 — [PENDING]
**Type:** AC_QUALITY
**Context:** JE020 v6→v7 revision — single-outcome Then clauses were being wrapped in a nested `<ul>` even when only one outcome existed
**Learning:** The nested `<ul>` inside a Then `<li>` is only justified when there are TWO OR MORE distinct Then outcomes. When a Then clause has exactly one outcome, write it inline: `<li>Then the record is saved successfully</li>`. Never wrap a single outcome in a nested list — it adds visual noise with no structural benefit.
**Example:**
BAD (single outcome, unnecessary nesting): `<li>Then<ul><li>The history panel displays all recorded change events</li></ul></li>`
GOOD (single outcome, inline): `<li>Then the history panel displays all recorded change events</li>`
GOOD (multiple outcomes, nested): `<li>Then<ul><li>The record is removed and the grid refreshes</li><li>If cancelled, the record is retained</li></ul></li>`
**Source:** User feedback — JE020 v7 revision session 2026-05-15
**Applies to:** AC Template section of `eam-story-core.md` — GWT nested structure rule

---

### L020 — 2026-05-15 — [PENDING]
**Type:** TEMPLATE
**Context:** JE020-Security revision — missing menu navigation AC, and AC2 incorrectly described the no-permission behavior as "access-denied message or redirect." The QARCH framework removes menu items for users without Execute permission — unauthorized users never see the screen at all.
**Learning:** Every Security satellite story must follow this standard:

**Persona:** `system administrator`

**Description must explain the platform security model — NO "QARCH" references:**
The screen's access is controlled by the platform's security framework. Users are assigned to one or more security groups. Each security group is configured with specific privileges on the screen's registered security object. Those privileges determine: (1) whether the screen appears in the navigation menu at all, (2) whether the user can open it, and (3) which operations (query, modify, delete) are available once inside. Do NOT use the term "QARCH" anywhere in the story — it is an implementation detail that has no place in PO-voice content. Do NOT use the term "role" — the correct term is "security group."

**Wiki link format for §17 — SINGLE encoding (not double):**
- §16: `anchor=16.-screen-navigation-%2526-menu-placement` (double-encoded `%26` → `%2526`) — this is correct and tested
- §17: `anchor=17.-security-%E2%80%94-screen-level-access-control` — use this EXACTLY as written (single-encoded em dash); do NOT double-encode to `%25E2%2580%2594`

**Functional Scope must cover all four privilege tiers:**
- **Navigation visibility** — screen appears in the module navigation menu only for users assigned to a security group that grants the required screen access privilege. Users not assigned to any such security group do not see the screen as a menu option.
- **Screen access** — users assigned to a security group with screen access privilege can open and interact with the screen
- **Read-only access** — users assigned to a security group with view-only privilege can query and view records but cannot create, edit, or delete; CRUD toolbar controls are hidden or disabled
- **Full access** — users assigned to a security group with write and delete privileges can perform all CRUD operations

**4 ACs (mandatory — not 3):**
- AC1: **Menu navigation** — a user assigned to a security group that grants the required privilege navigates to the module group and sees the screen listed as a menu item; selecting it loads the screen correctly. Reference §16.
- AC2: **Screen loads with authorized functions** — user assigned to a security group with full access privileges opens the screen and all CRUD functions are available. Reference §17.
- AC3: **Read-only user** — user assigned to a security group with view-only privilege can query and view records; create, edit, and delete controls are hidden or disabled; no data changes can be saved. Reference §17.
- AC4: **No access = not visible in menu** — a user not assigned to any security group that grants access to this screen does NOT see the screen as a menu option. There is no error message, no redirect — the screen simply does not appear. Reference §17.

**CRITICAL: AC4 must say the screen does NOT appear in the navigation menu. Never say "access-denied message" or "redirect" for the no-permission scenario. The platform suppresses the menu item entirely before the user can navigate to it.**

**Source:** JE020 source analysis (`QFrmAcctAcctGroupXref` extends `QFrmBaseFilterDetailQra`, no custom `InitializeSecurity()`) + Aaron's corrections on no-permission behavior, security group terminology, and no QARCH references — 2026-05-15
**Applies to:** All Security satellite stories across all EAM screens

---


### L021 — 2026-05-20 — [PENDING]
**Type:** CODEBASE_ANALYSIS
**Context:** JE030 story set review — Contextual Navigation (-03) and Audit History (-03) stories were both hallucinated and had to be retired after codebase validation
**Learning:** Every special story slot beyond -01 and -02 must be backed by a specific, confirmed code artifact before it can be proposed. Three hallucination patterns were identified and must be actively guarded against:

1. **Cross-screen generalization** — A feature present on similar screens (e.g., Audit History on JE020/JE025) was assumed to exist on JE030 without verifying `BuildLinkContextMenuAdditionalScreenLinks` was overridden in the JE030 source file. It was not. Never carry a feature forward from one screen to another without independently confirming it in that screen's own source.

2. **One-sided contract** — `UpdateContext` being implemented on a destination screen was treated as proof that Contextual Navigation existed. The source side was never verified (no `LaunchScreen` call to JE030 exists anywhere in the codebase). Both sides of any two-party interaction must be confirmed.

3. **Identifier semantic drift** — `ValidateReqPurgedFields` (JE011) was misread as a delete-time purge check. The trigger event was `BeforeUpdate`, not `BeforeDelete`. Method names are not evidence of behavior — the trigger event and method body must both be read.

**Evidence standards by feature type:**
- Business Rule: registered in `InitializeBusinessRules()` + trigger event (`AddEvaluationTime`) confirmed + method body read
- Contextual Navigation: `UpdateContext` handles key AND at least one `LaunchScreen(...)` in the codebase targets this screen's product ID
- Audit History: `BuildLinkContextMenuAdditionalScreenLinks` is overridden AND contains a `LaunchScreen` call to an audit history screen
- Delete validation: rule registered with `BeforeDelete` trigger — `BeforeUpdate` rules are never delete-time checks

**Source:** JE030 story set validation session — 2026-05-20
**Applies to:** `eam-workflow.md` Phase 1 (Analysis) — evidence standards and two-sided contract verification

---

### L022 — 2026-05-20 — [PENDING]
**Type:** TEMPLATE
**Context:** JE030 QA story and all prior QA stories — Functional Scope contained "Sign-off document confirming functional equivalence" and "Classic screen retirement ticket raised in ADO upon sign-off"
**Learning:** The EAM project is NOT retiring Classic GUI screens as part of this migration. QA/Parity stories cover verification that the web screen works correctly — nothing more. All of the following must be removed from QA story content wherever they appear:
- "Classic screen retirement ticket raised in ADO upon sign-off"
- "Sign-off document confirming functional equivalence"  
- "provides formal sign-off that Classic GUI is no longer required for [screen] workflows"
- Any language implying the Classic screen will be decommissioned, deprecated, or retired as an outcome of this work

The QA story Functional Scope should contain only:
- Structured verification of web screen behavior across all functional areas covered by this screen's story sequence; Classic GUI may be used as a side-by-side reference but is not required to be executed as part of testing

The QA story title convention is: `[ScreenID]-[N]: Testing & QA Parity` (not "Classic Screen Sign-Off")

**Source:** Aaron's correction — 2026-05-20
**Applies to:** `eam-ac-library.md` QA story template; `eam-decomposition.md` QA title pattern; `eam-workflow.md` plan template; all future QA stories

---

### L023 — 2026-05-20 — [PENDING]
**Type:** AC_QUALITY
**Context:** JE030 QA story review — ACs were structured as "in both the web screen and Classic GUI... both environments behave the same." Classic GUI is not under test.
**Learning:** QA/Parity story ACs must describe the expected behavior of the web screen only. Classic GUI is not a required test environment — it may be used informally as a side-by-side reference guide during testing, but it must never appear as a required step or environment in any AC.

The following patterns are prohibited in QA ACs:
- "When a QA analyst performs X in both the web screen and Classic GUI"
- "Then both environments behave the same / return matching records / display equivalent messages"
- "Given the same [input] in both environments"
- Any AC step that requires the tester to execute an action in Classic GUI

The correct pattern is:
- Given/When: describes a user action in the web screen only
- Then: describes the expected web screen outcome — specific, testable, not "matches Classic"
- Classic GUI may appear as an optional parenthetical note (e.g., "consistent with Classic GUI behavior") but never as a required test step

**Example:**
BAD: "When a QA analyst creates a record in both the web screen and Classic GUI — Then the resulting data state matches between environments"
GOOD: "When a QA analyst creates a new record on the web screen with valid data and saves — Then the record appears in the results grid and its field values are persisted correctly"

**Source:** Aaron's correction — 2026-05-20
**Applies to:** `eam-ac-library.md` QA AC guidance; all future QA story ACs

---
### L024 — 2026-05-21 — [PENDING]
**Type:** WORKFLOW
**Context:** VL025-05 and VL025-06 review — developer identified five business rules missing from the original -02 story. Root cause was that `InitializeBusinessRules()` was not fully read and reconciled against the story's validation list before authoring.
**Learning:** Before drafting any -02 or validation story, search the source for ALL registered business rules in `InitializeBusinessRules()` for the screen's form file(s) — including any companion detail forms (e.g., both `QFrmMILinkMaintenance.cs` and `QFrmMILinkDetail.cs` for VL025). Produce a complete rule inventory and classify every rule before writing a single AC. Also check `Controller_OverrideUpdateData` for cascade operations not registered as formal business rules, and look for `DataAccessHelper.ExecuteNonQuery(...)` calls inside rule delegates — these are post-save sync operations that belong in the story even though they are not blocking validations. Do not begin drafting until every rule in the inventory is assigned to a story.
**Example:** VL025 `InitializeBusinessRules()` registers 22+ rules. Five were not assigned to any story: `VALIDATE_MI_EXISTS` (delete guard), `VALIDATE_DO_PROP_OR_MP_NO_REQUIRED` (conditional required fields), `VALIDATE_WELL_NO_REQUIRED` (conditional required fields), `UPDATE_RXRF_MI_SOD_FROM_MI_EFF_DT` (post-save sync), and `DELETE_RXRF_MI_SOD_FROM_MI_EFF_DT` (post-delete sync). All five were caught by the developer post-creation rather than pre-authoring.
**Source:** VL025-05/VL025-06 developer feedback — 2026-05-21
**Applies to:** `eam-project-instructions.md` — Pre-Draft Checklist for -02 and Validation Stories; `eam-workflow.md` Phase 1 (Analysis)

---

### L025 — 2026-05-21 — [PENDING]
**Type:** AC_QUALITY
**Context:** VL025-06 — the conditional required field rules (`VALIDATE_DO_PROP_OR_MP_NO_REQUIRED`, `VALIDATE_WELL_NO_REQUIRED`) were not in the original story. The generic Required Field template in eam-ac-library.md Section 2 Category 1 does not accommodate rules where the requirement is activated by a flag or configuration key rather than being always-on.
**Learning:** When a business rule enforces a field requirement only under a specific condition — a flag set to a particular value, a configuration key, or the presence/absence of another field — use the Conditional Required Field template (Section 2 Category 5), not the generic Required Field template (Section 2 Category 1). The conditional trigger must appear explicitly in the Given clause of the negative case. The positive case must confirm the field is NOT required when the condition is absent. Never collapse conditional rules into the generic template — the condition is load-bearing for QA test writability. Common triggers in QRA: `Calculate Severance Tax = Yes`, `Value Allocation flag = Y`, `BOOK_REV_WELL_COMPL` config key, `ALLOW_DELETION_MI_LINK` config key.
**Example:**
BAD (collapsed into generic): "Given I attempt to save a record with Well Number left empty — When I click Save — Then the save is blocked"
GOOD (conditional): "Given Calculate Severance Tax is set to Yes and Well Number is not provided — When I attempt to save — Then a validation error states that a Well Completion is required when Calculate Severance Tax is Yes, and the save is blocked"
**Source:** VL025-06 authoring session — 2026-05-21
**Applies to:** `eam-ac-library.md` Section 2 Category 5 (Conditional Required Field Validation)

---

### L026 — 2026-05-21 — [PENDING]
**Type:** AC_QUALITY
**Context:** VL025-06 — the SOD sync operations (`UPDATE_RXRF_MI_SOD_FROM_MI_EFF_DT`, `DELETE_RXRF_MI_SOD_FROM_MI_EFF_DT`) were entirely absent from both original VL025 tickets. Post-save sync operations have no home in the current AC library and are structurally invisible — they produce no UI feedback on success and are registered as business rules that fire silently after `AllEvaluatedRulesSuccessful` passes.
**Learning:** After completing the business rule inventory (L024), explicitly scan for post-save and post-delete sync operations by looking for `DataAccessHelper.ExecuteNonQuery(...)` calls guarded by `if (!businessRuleContext.AllEvaluatedRulesSuccessful ...) return;`. These are not blocking validations — they are silent side effects that fire only when all prior rules pass. They must be included in the same story as the user action that triggers them (save or delete), never deferred to a separate technical story. Write one AC set per sync operation using the Section 2b template: (1) sync fires when trigger condition is met, (2) sync does not fire when condition is absent (if conditional), (3) sync removes records on delete (if applicable), (4) sync failure surfaces as a user-visible error. Missing these in stories means they will not be implemented in the web migration, causing silent data corruption that is only detected at month-end close.
**Example:** VL025 `UpdateRxrfMISod()` fires `UPDATE_RXRF_MI_SOD_FROM_MI_EFF_DT` when either effective date changes and `DELETE_RXRF_MI_SOD_FROM_MI_EFF_DT` when rows are deleted. Neither appeared in any VL025 story until a developer flagged the gap. Both required their own AC pairs (trigger fires / trigger does not fire / delete path / failure handling).
**Source:** VL025-06 authoring session — 2026-05-21
**Applies to:** `eam-ac-library.md` Section 2b (Post-Save System Sync Operations)

---
## Promoted Learnings
*(Already incorporated into the skill SKILL.md — kept for audit trail)*

*(None yet — this is the initial seed)*

---

## Rejected Learnings
*(Decided not to promote — reason noted)*

*(None yet)*