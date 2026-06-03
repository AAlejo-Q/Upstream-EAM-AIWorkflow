# EAM Story Authoring Workflow

> **Version:** 1.1 | **Last Updated:** 2026-05-20
> **Load this file:** When starting a story authoring session or when you need to understand the process sequence.
> **What this file covers:** Process phases only — approval gates, what to do when, what to load. Templates and AC patterns live in `eam-story-core.md` and `eam-ac-library.md`.

---

## Critical Principles (Non-Negotiable)

1. **PO Voice Only** — Stories define WHAT users need and WHY. Never include HOW (class names, table names, stored procedures, framework patterns).
2. **Approval Gates** — Never create ADO work items without explicit user confirmation.
3. **Template Compliance** — Every story must use the Description and AC templates from `eam-story-core.md`. Read it before writing any HTML. No exceptions. No shortcuts.
4. **Validate Before Creating** — Run the Pre-Creation Validation Gate (Phase 4.5) before every ADO write, whether a full story map or a single ad-hoc story.
5. **Consolidate Aggressively** — Aim for 4–10 numbered stories per screen. Merge small stories. Split only when combined SP > 8 or rule count > 12.
6. **No SP Estimates** — Story point sizing is the team's responsibility during sprint planning. Claude does not estimate or set SP fields.

---

## Phase 1: Analysis

**Goal:** Understand the full scope before generating stories. Every story type proposed in Phase 2 must be grounded in a specific, verified artifact found during this phase. If it cannot be grounded, it cannot be proposed.

**Load:** `upstream-codebase-guide.md` before any codebase search.

### Steps

1. **Identify the screen** — Confirm: screen ID, parent Feature ID, migration pattern (Lift and Shift / Redesign / Combine), and any customer-requested enhancements beyond parity.

2. **Review existing context:**

   * `eam-project-instructions.md` — domain background, ADO defaults, active Features
   * `Effort-Priority-Matrix.md` — **primary source for screen technical profile.** Extract: New CodeGen (count + class names), Tables, BRs, Screen Type, Effort tier, and Key Complexity Signals. Do NOT use `screen-inventory.md` for gap counts or POC % — that data is outdated. The matrix is authoritative.
   * ADO parent Feature — read Benefit Hypothesis, Screen Inventory, and Acceptance Criteria

3. **Read the full source file** — Fetch and read `QFrm{ScreenID}.cs` (or the equivalent detail form) in its entirety. Do not rely on search snippets or cached results. Key things to extract and record verbatim:

   * All entries in `InitializeBusinessRules()` — rule name, trigger event (`AddEvaluationTime`), and method name
   * All entries in `InitializeData()` — table names and relationships
   * All entries in `InitializeControlState()` — read-only fields, grid behaviors, double-click events
   * `BuildLinkContextMenuAdditionalScreenLinks` — if overridden, list every `LaunchScreen` call and its target product ID
   * `FillContextDataMgrForPlugIn` and `UpdateContext` — if present, record which context keys are registered
   * `InitializeSecurity` — if overridden, note any tab-level security patterns

4. **Read each business rule method body** — For every rule registered in `InitializeBusinessRules()`, read the actual method implementation. Record:
   * What data the rule reads
   * What condition triggers a block
   * The trigger event (e.g., `BeforeUpdate`, `BeforeDelete`, `AfterQuery`) — **the trigger event is the only reliable indicator of when a rule fires; method names are not evidence**

5. **Apply two-sided contract verification** — For any feature that involves a two-party interaction, both sides must be confirmed in code before the feature can be proposed:

   | Feature Type | What Must Be Confirmed |
   |---|---|
   | **Business Rule** | Rule registered in `InitializeBusinessRules()` + trigger event confirmed + method body read |
   | **Contextual Navigation** | Destination: `UpdateContext` handles the key AND Source: at least one `LaunchScreen(...)` call in the codebase references this screen's fully-qualified product ID |
   | **Audit History** | `BuildLinkContextMenuAdditionalScreenLinks` is overridden AND contains a `LaunchScreen` call to an audit history screen |
   | **Custom Links Menu** | `BuildLinkContextMenuAdditionalScreenLinks` is overridden in this file — base class behavior alone is not sufficient |
   | **Delete validation** | A rule is registered with `BeforeDelete` trigger — `BeforeUpdate` rules cannot be described as delete-time checks |

6. **Admit uncertainty explicitly** — You have explicit permission from Aaron and Asher to say "I don't know" or "I cannot confirm this from the code." This is not a failure — it is the correct behavior. If after reading the full source file a feature cannot be confirmed with direct code evidence:
   * State clearly: *"I cannot confirm [feature] from the codebase. The evidence I found is [what was found]. I cannot determine [what is missing]."*
   * Do NOT fill the gap with a plausible inference, a pattern from another screen, or a general assumption about how the framework works
   * Proceed to Step 7

7. **Resolve unknowns before proceeding** — For every item flagged as uncertain in Step 6, attempt one of the following before moving to Phase 2:
   * **Broader code search** — Search for the feature by a different approach (e.g., for contextual navigation: search for the screen's product ID string across the full repo)
   * **Base class check** — If the feature might be inherited, check the base class (`QFrmBaseQra`, `QFrmBaseFilterDetailFormQra`) for the relevant method
   * **SME escalation** — If code search is exhausted and the feature still cannot be confirmed, mark it as an **open SME question** in the Phase 2 plan. Do not create a story for it until the SME confirms it exists.

8. **Translate confirmed findings only** — Convert confirmed technical findings to PO-voice business language. Only findings with direct code evidence from Steps 3–7 are eligible for translation. Unconfirmed inferences are not translated — they become open SME questions.

---

## Phase 2: Planning (Approval Gate)

**Goal:** Present a structured plan and wait for approval before drafting stories.

**Load:** `eam-decomposition.md` before planning.

### Steps

1. **Select decomposition** — Using the Universal Story Sequence in `eam-decomposition.md`, identify which stories apply to this screen. Start with -01 and -02 (always present), then identify which special capability slots (-03, -04, -05, -06+) are triggered by this screen's features. **Every special story slot must be backed by a confirmed code artifact from Phase 1. If no artifact was found for a slot, that slot is not included — it becomes an open SME question instead.**

2. **Run the Evidence Check** — Before presenting the plan, verify each proposed story against Phase 1 findings:

   ```
   For each proposed story beyond -01 and -02:
   □ What is the specific code artifact that justifies this story?
   □ Was that artifact read and confirmed in Phase 1 (not inferred)?
   □ If the story involves a two-sided interaction — was BOTH sides confirmed?

   If any story fails this check:
   → Remove the story from the plan
   → Add it to the Open SME Questions section of the plan
   → Mark it: "Cannot confirm from codebase — SME input required before authoring"
   ```

3. **Present the plan** — Show:

```
   [ScreenID] — [Screen Name] — [Migration Pattern]
   
   Numbered stories:
   [ScreenID]-01: Screen Foundation, Navigation & Record CRUD
   [ScreenID]-02: Business Rules and Validations
   [ScreenID]-0N: [Special story — confirmed by: {specific artifact}]
   [ScreenID]-[Last]: Testing & QA Parity
   
   Satellite stories:
   [ScreenID]-Security: Security & Access Control (→ Feature 1786977)
   
   Numbered: N stories | Satellites: 1 | Total: N+1
   Successor chain: -01 → -02 → ... → -[Last]
   
   Open SME Questions (cannot confirm from codebase — do not author until resolved):
   Q1: [Feature or detail that could not be verified] — Evidence found: [what was found] — Missing: [what could not be confirmed]
   
   Risks: [any noted]
   ```

3. **Surface risks:**

   * Complex business rules (especially effective date overlap detection)
   * Design decisions that differ from Classic (document the rationale)
   * Dependencies on other screens or shared services
   * High-risk patterns: file upload, mass changes, batch integration (see risk table in `eam-decomposition.md`)
4. **Wait for explicit approval** before proceeding to Phase 3. Do not start drafting until the plan is confirmed.

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
4. **For -02 stories, mandatory validation patterns (if applicable to this screen):**

   * Required field validation (positive + negative)
   * Entity/reference existence validation (positive + negative)
   * Duplicate prevention (positive + negative)
   * Pick list value integrity (positive + negative)
Full HTML for these patterns is in `eam-ac-library.md`.
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

**Goal:** Verify every story is template-compliant before any ADO write.

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
* [ ] Tags set on every story — `eam-phase-*` tag correct per story category
* [ ] Security stories parented to Feature 1786977
* [ ] SP fields NOT set — do not pass `Custom.SAFeStoryPoints` or `Microsoft.VSTS.Scheduling.StoryPoints`

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

## Ad-Hoc Story Creation

When creating a single story (not a full story map), the same rules apply. There is no shortcut.

1. Load `eam-story-core.md` and confirm you have read the Description Template and AC Template
2. Identify the parent Feature and verify the area path against a sibling story
3. Draft using the templates
4. Run the Phase 4.5 Critical 5 check
5. Present for approval
6. Create using the two-step pattern

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
* Verify area path against a live sibling story before every creation
* Include paired positive/negative ACs for every validation rule
* Reference wiki sections (§1–§25) for standard behaviors; include clickable URLs
* Cross-reference Out of Scope items to the specific story that covers them
* State business rules in plain business language
* Read the **full source file** for every screen — do not rely on search snippets
* Read every business rule **method body**, not just its registered name
* Verify **both sides** of any two-party interaction (source + destination for navigation; trigger + handler for rules)
* Admit uncertainty explicitly when code evidence is absent — "I cannot confirm this from the codebase" is always the correct response when evidence is missing
* Exhaust code search before escalating to SME — but escalate rather than infer when search is exhausted

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
* Treat destination-side wiring (e.g., `UpdateContext`) as proof a feature is active — the source side must also be confirmed
* Fill an evidence gap with a plausible assumption — gaps become open SME questions, not draft stories

