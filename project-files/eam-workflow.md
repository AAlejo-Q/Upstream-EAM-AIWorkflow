# EAM Story Authoring Workflow

> **Version:** 2.0 | **Last Updated:** 2026-06-03
> **Load this file:** When starting a story authoring session or when you need to understand the process sequence.
> **What this file covers:** Process phases only — approval gates, what to do when, what to load. Templates and AC patterns live in `eam-story-core.md` and `eam-ac-library.md`.
> **What changed in v2.0:** Phase 1 restructured to consume the dev team's analysis and Screen-Spec before doing independent analysis. Metadata table queries added. Validation tier classification added to Phase 3 for -02 stories. Gap check added to Phase 4.5 against Screen-Spec. Feedback loop established from dev team's ADO Story Analysis reports.

---

## Critical Principles (Non-Negotiable)

1. **PO Voice Only** — Stories define WHAT users need and WHY. Never include HOW (class names, table names, stored procedures, framework patterns).
2. **Approval Gates** — Never create ADO work items without explicit user confirmation.
3. **Template Compliance** — Every story must use the Description and AC templates from `eam-story-core.md`. Read it before writing any HTML. No exceptions. No shortcuts.
4. **Validate Before Creating** — Run the Pre-Creation Validation Gate (Phase 4.5) before every ADO write, whether a full story map or a single ad-hoc story.
5. **Consolidate Aggressively** — Aim for 4–10 numbered stories per screen. Merge small stories. Split only when combined SP > 8 or rule count > 12.
6. **No SP Estimates** — Story point sizing is the team's responsibility during sprint planning. Claude does not estimate or set SP fields.
7. **Consume Before Re-Deriving** — Always check for existing dev team analysis and Screen-Spec artifacts before performing independent analysis. Do not duplicate work that already exists.
8. **Classify Validations by Tier** — Every validation rule in a -02 story must include its tier classification (Tier 1–4) so the dev team can implement without re-deriving.

---

## Phase 1: Analysis

**Goal:** Understand the full scope before generating stories. Every story type proposed in Phase 2 must be grounded in a specific, verified artifact found during this phase. If it cannot be grounded, it cannot be proposed.

**Load:** `upstream-codebase-guide.md` before any codebase search.

### Step 1 — Identify the Screen

Confirm: screen ID, parent Feature ID, migration pattern (Lift and Shift / Redesign / Combine), and any customer-requested enhancements beyond parity.

### Step 2 — Review Existing Context

* `eam-project-instructions.md` — domain background, ADO defaults, active Features
* `Effort-Priority-Matrix.md` — **primary source for screen technical profile.** Extract: New CodeGen (count + class names), Tables, BRs, Screen Type, Effort tier, and Key Complexity Signals. Do NOT use `screen-inventory.md` for gap counts or POC % — that data is outdated. The matrix is authoritative.
* ADO parent Feature — read Benefit Hypothesis, Screen Inventory, and Acceptance Criteria

### Step 3 — Check for Dev Team's Analysis and Screen-Spec

Before performing any independent source analysis, check whether the dev team has already produced artifacts for this screen. These live in the `Quorum-Enterprise/Quorum.Upstream.AITools` GitHub repo.

**Step 3a — Check for analysis files:**

```
documentation/Analysis/ClassicGUI/[ScreenID]-*/
```

Use `github:get_file_contents` to check for a folder starting with the screen ID. If found, it contains up to 5 files: `[ScreenID]-Overview.md`, `[ScreenID]-Technical-Analysis.md`, `[ScreenID]-SQL-Analysis.md`, `[ScreenID]-CodeGen-Analysis.md`, and `[ScreenID]-Test-Scenarios.md`.

**Step 3b — Check for Screen-Spec:**

```
documentation/SpecKit/modules/[Module]/[ScreenID]-*-Screen-Spec.md
```

If found, this is the dev team's canonical migration blueprint — it covers database tables, data objects, service layer, screen layout (filters, grid columns, toolbar actions), every business rule individually, lookups/dropdowns, context data dependencies, special behaviors, cross-screen navigation, Notes, and open questions.

**Step 3c — Check for existing ADO Story Analysis:**

```
documentation/SpecKit/modules/[Module]/[ScreenID]-ADO-Story-Analysis.md
```

If found, this contains the dev team's gap analysis of our existing stories against their spec. Extract any contradictions, spec-only items, or open questions — these need to be addressed.

**Step 3d — Route based on what was found:**

| Analysis Found | Screen-Spec Found | Action |
|---|---|---|
| Yes | Yes | **Fast path** — Consume both. Extract findings from the spec and analysis files. Proceed to Step 5 (metadata supplementation). Skip independent source analysis unless specific gaps exist. |
| Yes | No | Consume the analysis files. Perform targeted source reads for anything the analysis didn't cover. Proceed to Step 5. |
| No | No | **Full analysis path** — Perform independent source analysis (Step 4). |
| No | Yes | Consume the spec. Verify key claims against the source file (Step 4, targeted only). |

### Step 4 — Source Analysis (Full or Targeted)

> Skip this step if the fast path was taken in Step 3d AND no gaps were identified.

**When performing full analysis:** Read the full source file `QFrm{ScreenID}.cs` (or the equivalent detail form) in its entirety. Do not rely on search snippets. Key things to extract and record verbatim:

* All entries in `InitializeBusinessRules()` — rule name, trigger event (`AddEvaluationTime`), and method name
* All entries in `InitializeData()` — table names and relationships
* All entries in `InitializeControlState()` — read-only fields, grid behaviors, double-click events
* `BuildLinkContextMenuAdditionalScreenLinks` — if overridden, list every `LaunchScreen` call and its target product ID
* `FillContextDataMgrForPlugIn` and `UpdateContext` — if present, record which context keys are registered
* `InitializeSecurity` — if overridden, note any tab-level security patterns

**When performing targeted analysis:** Read only the specific methods or sections needed to fill gaps identified in Step 3d.

**Read each business rule method body** — For every rule registered in `InitializeBusinessRules()`, read the actual method implementation. Record what data the rule reads, what condition triggers a block, and the trigger event (e.g., `BeforeUpdate`, `BeforeDelete`, `AfterQuery`). The trigger event is the only reliable indicator of when a rule fires; method names are not evidence.

**Apply two-sided contract verification** — For any feature that involves a two-party interaction, both sides must be confirmed in code before the feature can be proposed:

| Feature Type | What Must Be Confirmed |
|---|---|
| **Business Rule** | Rule registered in `InitializeBusinessRules()` + trigger event confirmed + method body read |
| **Contextual Navigation** | Destination: `UpdateContext` handles the key AND Source: at least one `LaunchScreen(...)` call references this screen's product ID |
| **Audit History** | `BuildLinkContextMenuAdditionalScreenLinks` is overridden AND contains a `LaunchScreen` call to an audit history screen |
| **Custom Links Menu** | `BuildLinkContextMenuAdditionalScreenLinks` is overridden in this file — base class behavior alone is not sufficient |
| **Delete validation** | A rule is registered with `BeforeDelete` trigger — `BeforeUpdate` rules cannot be described as delete-time checks |

### Step 5 — Metadata Supplementation

Regardless of which path was taken, query the following metadata tables using the `quorum-metadata-dev` MCP server. These queries add precision that even the dev team's analysis may not fully capture for story authoring purposes.

**Step 5a — Grid configuration:**

```sql
SELECT GRID_ID, GRID_NM, SELECT_SQL_NM, TABLE_NM, READ_ONLY_IND
FROM QARCH_CNFG_GRIDCTRL_NET
WHERE FORM_ID = '[ENTITY_ID]'
```

Record: grid name, primary table, whether the grid is read-only. This determines whether -01 needs CRUD or is query-only.

**Step 5b — Grid column definitions (using GRID_ID from 5a):**

```sql
SELECT SEQ_NO, DB_COL_NM, DISPLAY_HDR, CTRL_TYPE_CD, CDTBL_ID, PKLIST_ID, READ_ONLY_IND, HIDDEN_IND, NULLABLE_IND
FROM QARCH_CNFG_GRIDCTRL_COL_NET
WHERE GRID_ID = [GRID_ID]
ORDER BY SEQ_NO
```

Extract: all `CDTBL_ID` values (code table dropdowns) and `PKLIST_ID` values (pick list dialogs). Count visible editable columns vs. read-only columns.

**Step 5c — Code table definitions (if CDTBL_IDs found):**

```sql
SELECT CDTBL_ID, CDTBL_DESCR, DB_TBL_NM
FROM QARCH_CDTBL_DEFINE
WHERE CDTBL_ID IN ([comma-separated IDs])
```

**Step 5d — Pick list definitions (if PKLIST_IDs found):**

```sql
SELECT PKLIST_ID, PKLIST_NM, PKLIST_DESCR
FROM QARCH_CTRL_PKLIST_NET
WHERE PKLIST_ID IN ([comma-separated IDs])
```

**Step 5e — SQL key inventory:**

Derive the key prefix from the `SELECT_SQL_NM` value (e.g., `ONRL_LINK_MAINT_LOAD` → prefix `ONRL_LINK_MAINT`):

```sql
SELECT REGISTERED_SQL_NM, SQL_STATEMENT, UPDT_TABLE_NM
FROM QARCH_SQL
WHERE REGISTERED_SQL_NM LIKE '[prefix]%'
ORDER BY REGISTERED_SQL_NM
```

Count: SELECT operations (read), INSERT/UPDATE/DELETE operations (write). This confirms CRUD scope.

### Step 6 — Classify Validation Rules by Tier

For every business rule identified (from the dev team's analysis, Screen-Spec, or independent source analysis), assign a tier classification using the dev team's 4-tier system:

| Tier | Type | Description | Story Implication |
|---|---|---|---|
| 1 | Pure in-memory | DO property checks, no DB call | Standard -02 validation AC |
| 2 | DB-lookup | Reads a lookup table to validate a value | -02 AC must name the entity being validated against |
| 3 | Commit-time / collection-level | Validates across all rows in the batch | -02 AC must describe the cross-record condition |
| 4 | Side-effects | Auto-populate, user dialogs, PPN writes, audit writes | Separate special story (-03+), not in -02 |

**Classification rules:**
* If the rule only checks properties already on the record being saved → **Tier 1**
* If the rule reads another table to validate a value → **Tier 2**
* If the rule checks conditions across multiple records in the same save batch → **Tier 3**
* If the rule writes to another table, triggers a dialog, auto-populates fields, or creates PPN records → **Tier 4** (do NOT include in -02; create a separate special story)

**Resource string resolution:** When reading business rule method bodies, note any `RES_MSG_*` resource string references. If the dev team's Technical Analysis file includes a "Error Messages (Resolved Text)" section, use the canonical English text from there. Include the resolved error message text in -02 ACs where applicable — this prevents message drift during implementation.

### Step 7 — Uncertainty and Escalation

**Admit uncertainty explicitly.** You have explicit permission from Aaron and Asher to say "I don't know" or "I cannot confirm this from the code." If after reading all available sources a feature cannot be confirmed with direct evidence:
* State clearly: *"I cannot confirm [feature] from the available sources. The evidence I found is [what was found]. I cannot determine [what is missing]."*
* Do NOT fill the gap with a plausible inference, a pattern from another screen, or a general assumption

**Resolve unknowns before proceeding.** For every item flagged as uncertain:
* **Broader code search** — search by a different approach
* **Base class check** — check the base class for inherited behavior
* **Cross-reference dev team artifacts** — the Screen-Spec's Open Questions section may already flag this
* **SME escalation** — if all sources are exhausted, mark as an open SME question

### Step 8 — Translate to PO Voice

Convert confirmed findings to PO-voice business language. Only findings with direct evidence from Steps 3–7 are eligible for translation. Include:
* The tier classification for each validation rule (Tier 1/2/3/4)
* Canonical error message text where available (from resource string resolution)
* References to dev team findings that informed the translation

---

## Phase 2: Planning (Approval Gate)

**Goal:** Present a structured plan and wait for approval before drafting stories.

**Load:** `eam-decomposition.md` before planning.

### Steps

1. **Select decomposition** — Using the Universal Story Sequence in `eam-decomposition.md`, identify which stories apply to this screen. Start with -01 and -02 (always present), then identify which special capability slots (-03, -04, -05, -06+) are triggered. **Every special story slot must be backed by a confirmed artifact from Phase 1. Tier 4 rules go into special stories, not -02.**

2. **Run the Evidence Check** — Before presenting the plan, verify each proposed story against Phase 1 findings:

   ```
   For each proposed story beyond -01 and -02:
   □ What is the specific artifact that justifies this story?
   □ Was that artifact confirmed in Phase 1 (not inferred)?
   □ If the story involves a two-sided interaction — were BOTH sides confirmed?
   □ If the story is for a Tier 4 rule — is it correctly excluded from -02?

   If any story fails this check:
   → Remove the story from the plan
   → Add it to the Open SME Questions section
   → Mark it: "Cannot confirm from available sources — SME input required before authoring"
   ```

3. **Present the plan** — Show:

   ```
   [ScreenID] — [Screen Name] — [Migration Pattern]
   
   Dev Team Artifacts Consumed:
   Analysis files: [Yes — path | No — independent analysis performed]
   Screen-Spec: [Yes — path | No — not available]
   ADO Story Analysis: [Yes — N contradictions, N gaps | No — not available]
   
   Numbered stories:
   [ScreenID]-01: Screen Foundation, Navigation & Record CRUD
   [ScreenID]-02: Business Rules and Validations (Tier 1: N, Tier 2: N, Tier 3: N rules)
   [ScreenID]-0N: [Special story — confirmed by: {specific artifact}]
   [ScreenID]-[Last]: Testing & QA Parity
   
   Satellite stories:
   [ScreenID]-Security: Security & Access Control (→ Feature 1786977)
   
   Tier 4 rules (in special stories, not -02):
   - [Rule description] → [Which special story covers it]
   
   Numbered: N stories | Satellites: 1 | Total: N+1
   Successor chain: -01 → -02 → ... → -[Last]
   
   Open SME Questions (cannot confirm — do not author until resolved):
   Q1: [Feature or detail that could not be verified]
   
   Risks: [any noted]
   ```

4. **Surface risks:**
   * Complex business rules (especially effective date overlap detection)
   * Design decisions that differ from Classic
   * Dependencies on other screens or shared services
   * High-risk patterns: file upload, mass changes, batch integration
   * Contradictions identified in existing ADO Story Analysis reports

5. **Wait for explicit approval** before proceeding to Phase 3.

---

## Phase 3: Story Generation

**Goal:** Draft complete, business-focused stories using the EAM templates.

**Load:** `eam-story-core.md` (mandatory before writing any HTML), `eam-ac-library.md` (for AC patterns and wiki section references).

### Steps

1. **Read `eam-story-core.md` now** — Do not write any HTML until you have read the Description Template and AC Template from that file in the current session.

2. **Draft each story:**
   * Title: `[ScreenID]-[Seq]: [Brief Action]`
   * Description: Full HTML following the skeleton in `eam-story-core.md`
   * ACs: Full HTML following the AC template in `eam-story-core.md`; reference `eam-ac-library.md` for wiki section links and standard validation patterns

3. **For -01 stories, mandatory ACs:**
   * Navigation & menu placement (§16)
   * Query filter, results, and empty state (§1 + §4) — combined into one AC
   * Record View and Selection + Retrieve/More/All (§26) — standalone AC
   * Pagination + grid defaults (§3 + §21/§22/§23/§25) — as last functional AC; 100-record default, dropdown to 200

4. **For -02 stories, mandatory validation patterns (if applicable):**
   * Required field validation (positive + negative)
   * Entity/reference existence validation (positive + negative)
   * Duplicate prevention (positive + negative)
   * Pick list value integrity (positive + negative)

   Full HTML for these patterns is in `eam-ac-library.md`.

   **Tier classification requirement:** Each validation rule AC in a -02 story must include a `<!-- Tier N -->` HTML comment at the start of the AC's `<li>` element, where N is 1, 2, or 3. Tier 4 rules are excluded from -02 stories entirely. Example:

   ```html
   <!-- Tier 2 — DB-lookup: validates against [entity] -->
   <li><strong>AC-02-03: [Rule Name] — Positive</strong>
   ```

   **Canonical error messages:** When a validation rule has a resolved `RES_MSG_*` error message from the dev team's analysis, include the exact expected error text in the negative (failure) AC's "Then" clause. This prevents message drift during implementation.

5. **For all grid stories:** Include the combined §21/§22/§23/§25 AC as the last functional AC. Full template in `eam-ac-library.md`.

6. **PO voice scan** — Before presenting, check every story for class names, table names, stored procedures, method names, ViewModel properties, and internal flags. Remove and replace with observable business outcomes.

7. **Present the full draft** — Show all stories with Description HTML and AC HTML. Do not create in ADO yet.

---

## Phase 4: Review & Refinement

**Goal:** Incorporate feedback before ADO creation.

### Steps

1. **Display the full story map** — All stories, complete.
2. **Explain reasoning** — Note consolidation decisions, assumption calls, and any places where the web design differs from Classic.
3. **Flag caveats** — Highlight stories with external dependencies, complex AC phrasing, or areas needing team clarification.
4. **Iterate** — Adjust titles, groupings, or AC language based on feedback. Reprioritize using WSJF if needed.

Do not proceed to Phase 4.5 until all feedback has been incorporated.

---

## Phase 4.5: Pre-Creation Validation (Mandatory Gate)

**Goal:** Verify every story is template-compliant AND spec-aligned before any ADO write.

> **This gate exists because template violations were found in production stories (JE005-06 was created without `<h2>User Story</h2>`, without the "As a..." statement, and with the wrong area path).** Run it every time — full map or single ad-hoc story.

### Critical 5 — Fix These First

Before anything else, verify these five for every story:

* [ ] Description starts with `<h2>User Story</h2>` and includes the "As a [role], I want... so that..." statement
* [ ] No security content in any numbered story (roles, permissions, access restrictions, unauthorized user blocking)
* [ ] Area path fetched from a live sibling story in the target Feature — not memorized or hardcoded
* [ ] No Story Points in the Description HTML
* [ ] Every grid story has the combined §21/§22/§23/§25 AC as the last functional AC

### Full Validation Checklist

* [ ] `<h2>Description</h2>` heading present before business context paragraph
* [ ] `<h3>Functional Scope</h3>` section present
* [ ] `<h3>Out of Scope</h3>` section present with cross-story references
* [ ] No class names, table names, stored procedures, or method names anywhere
* [ ] ACs use Given/When/Then with nested `<ul>` inside `<li>` for "Then" clauses
* [ ] Every validation rule has BOTH a positive and negative case — paired ACs never separated or dropped
* [ ] -01 stories include navigation AC (§16) and record view/selection AC (§26)
* [ ] -02 stories include all applicable standard validation patterns (from `eam-ac-library.md`)
* [ ] -02 stories include tier classification comments on every validation AC
* [ ] -02 stories exclude Tier 4 rules (those belong in special stories)
* [ ] Tags set on every story — `eam-phase-*` tag correct per story category
* [ ] Security stories parented to Feature 1786977
* [ ] SP fields NOT set — do not pass `Custom.SAFeStoryPoints` or `Microsoft.VSTS.Scheduling.StoryPoints`

### Screen-Spec Alignment Check (when Screen-Spec is available)

When a dev team Screen-Spec was consumed in Phase 1, perform this additional check before ADO creation:

For each section of the Screen-Spec, verify coverage:

| Spec Section | Covered By | Check |
|---|---|---|
| Business Rules table | -02 ACs | Every spec rule row has a corresponding AC (or is classified as Tier 4 and in a special story) |
| Special Behaviors table | Special stories (-03+) | Every spec behavior row is covered by a story |
| Grid Columns / Screen Layout | -01 ACs | Grid column count and filter controls are consistent |
| Notes section | -03 story (if notes present) | Notes story exists if spec says "Comments present: Yes" |
| Cross-Screen Navigation | -01 or special story | Navigation targets are covered |

If gaps are found:
* Missing business rule → add to -02 (Tier 1–3) or appropriate special story (Tier 4)
* Missing special behavior → add a story or AC
* Log the gap and the fix applied

**If any Critical 5 item fails, fix it before continuing. Do not proceed with known violations.**

---

## Phase 5: ADO Creation (After Explicit Approval Only)

**Goal:** Create stories in ADO using the two-step pattern.

> **Gate: Do not call any ADO write API until the user explicitly confirms.** Acceptable confirmations: "confirmed", "go ahead", "create them", "approved", "looks good, create". Silence is not approval.

**Load:** `ado-connection.md` before any ADO write.

### Creation Steps

**Step 1 — Create with title + description + parent link:**

* Numbered stories → screen's functional Feature (e.g., 1786972)
* Security story → Feature **1786977**
* Use `wit_add_child_work_items`; do NOT set Iteration Path
* System.Parent is NOT a settable field in wit_create_work_item. Parent links must be set via wit_work_items_link with type: "parent", or by using wit_add_child_work_items. If wit_add_child_work_items times out, create via wit_create_work_item and immediately wire the parent link as a separate call before proceeding to the next story.

**Step 2 — Batch update remaining fields:**

* AC, tags (`eam-phase-*`), AssignedTo
* Do NOT set SP fields
* Use `wit_update_work_items_batch`

**Step 3 — Create successor links:**

* Chain numbered stories only: `-01 → -02 → ... → -QA`
* Security satellites excluded from chain
* Use `wit_work_items_link` with `type: "Successor"`

**Step 4 — Verify:**

* Confirm correct parent Feature on each story
* Confirm successor chain is intact
* Confirm tags are set
* Log ADO IDs

### Re-Parenting Rules

When moving a story to a new parent Feature:

1. First: `wit_work_item_unlink` with `type: "parent"` on each child individually
2. Then: `wit_work_items_link` in a single batch with `linkToId` pointing to new parent

Attempting to set a new parent without removing the old one fails silently.

---

## Feedback Loop: Consuming Dev Team Gap Reports

When a dev team ADO Story Analysis report is found (Phase 1, Step 3c), or when the dev team runs `Analyze-ado-stories` against our stories after creation, a feedback loop is triggered.

### When to Trigger

* During Phase 1 (Step 3c) — if a gap report already exists before we author stories
* After Phase 5 — if the dev team produces a gap report against newly created stories
* On request — when Aaron or Asher asks to review alignment against the spec

### What to Do

1. **Read the gap report** from `documentation/SpecKit/modules/[Module]/[ScreenID]-ADO-Story-Analysis.md`

2. **For each "In Stories, Not in Spec" gap:**
   * Our story describes something the spec doesn't cover
   * Review: is this a legitimate requirement the spec missed, or is our story over-specifying?
   * If legitimate: note for the dev team to update their spec
   * If over-specified: flag for story amendment

3. **For each "Spec-Only" item (in spec, no story backing):**
   * The spec describes something we don't have a story for
   * Review: does this need a story, or is it an implementation detail the dev team handles without a story?
   * If it needs a story: draft an amendment or new story
   * If implementation detail: no action needed

4. **For each "Contradiction":**
   * Our story AC conflicts with the spec
   * **These block the dev pipeline.** The Audit-screen skill has a migration readiness gate that stops on unresolved contradictions.
   * Resolve immediately: update the story AC to match the spec, or escalate to SME if the spec is wrong

5. **For each "Open Question":**
   * Review and assign to the appropriate owner (BA, PO, or Dev)

6. **Present the amendment plan** for approval before modifying any ADO stories.

---

## Ad-Hoc Story Creation

When creating a single story (not a full story map), the same rules apply. There is no shortcut.

1. Load `eam-story-core.md` and confirm you have read the Description Template and AC Template
2. Identify the parent Feature and verify the area path against a sibling story
3. Check for dev team Screen-Spec — if available, verify the story aligns with spec content
4. Draft using the templates
5. For -02 stories: include tier classification on every validation AC
6. Run the Phase 4.5 Critical 5 check
7. Present for approval
8. Create using the two-step pattern

---

## Session Close

At the end of any session where revisions produced a new pattern:

1. Load `skill-learnings.md`
2. Capture the pattern with context, rationale, and before/after example
3. Flag which file(s) the pattern should be promoted to in the next template update

---

## Always / Never Quick Reference

### Always

* Read `eam-story-core.md` before writing any story HTML
* **Check for dev team analysis and Screen-Spec** before performing independent source analysis
* Verify area path against a live sibling story before every creation
* Include paired positive/negative ACs for every validation rule
* **Include tier classification (Tier 1/2/3) on every -02 validation AC**
* **Exclude Tier 4 rules from -02 stories** — they go in special stories
* Reference wiki sections (§1–§25) for standard behaviors; include clickable URLs
* Cross-reference Out of Scope items to the specific story that covers them
* State business rules in plain business language
* Read the **full source file** for every screen — do not rely on search snippets (unless consuming dev team analysis that already did this)
* Read every business rule **method body**, not just its registered name
* Verify **both sides** of any two-party interaction
* Admit uncertainty explicitly when evidence is absent
* Exhaust available sources before escalating to SME
* **Run metadata queries** (grid config, columns, code tables, pick lists, SQL keys) for every screen
* **Include canonical error message text** in negative validation ACs when available from dev team's resource string resolution
* **Check for and consume ADO Story Analysis gap reports** when they exist

### Never

* Create ADO work items without explicit approval
* Include class names, table names, stored procedures, or method names in stories
* Set Iteration Path when creating stories
* Set SP fields — sizing is the team's responsibility
* Include security content in numbered stories
* Use a memorized area path
* Skip the template for "just one story"
* Infer a feature exists because it exists on a similar screen — each screen must be verified independently
* Treat a method name as evidence of behavior — read the method body and trigger event
* Treat destination-side wiring as proof a feature is active — the source side must also be confirmed
* Fill an evidence gap with a plausible assumption — gaps become open SME questions, not draft stories
* **Perform redundant analysis when dev team artifacts already exist** — consume first, supplement only for gaps
* **Put Tier 4 rules in -02 stories** — they cause implementation confusion and belong in special stories
* **Ignore ADO Story Analysis contradictions** — they block the dev pipeline's migration readiness gate
