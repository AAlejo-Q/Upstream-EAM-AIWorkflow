# Learnings: eam-user-story-template
> Target skill: `/mnt/skills/user/eam-user-story-template/SKILL.md`
> Created: 2026-04-21
> Last promoted: 2026-05-05 (L008, L009 → SKILL.md v3.1)

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

### L006 — 2026-04-21 — [REJECTED 2026-05-05]
**Type:** TEMPLATE
**Context:** Work Item Type was set incorrectly as "User Story" in early ADO creation attempts
**Learning:** The ADO Work Item Type for child stories in the Quorum project is **Requirement**, not "User Story". Always use `"User Story"` → `"Requirement"` when calling the ADO REST API for story-level items. Features use `wit_create_work_item`; Requirements (stories) use `wit_add_child_work_items` with the parent Feature ID.
**Source:** ADO creation session — type mismatch discovered
**Applies to:** Section 12 (ADO Creation), Appendix C
**Rejected reason:** This learning is factually backwards. The correct type in the `Quorum` project IS `User Story`. `Requirement` is the type in the `QuorumSoftware` project. Applying this overlay actively pushes the wrong instruction. Confirmed by SKILL.md v3.1 Appendix C and all successful story creations (VL031, VL025, ML006, JE100, JE102, JE005).

---

### L007 — 2026-04-21 — [PENDING]
**Type:** WORKFLOW
**Context:** Bias calibration — early story maps had too many granular stories
**Learning:** Default bias should be toward **fewer, broader stories**. When in doubt between splitting and consolidating, consolidate. A story map that is 6–8 stories for a Lift and Shift screen is better than 10–12. Only split when: (a) SP would exceed 8, (b) stories are not independently deliverable, or (c) risk levels differ significantly between the bundled concerns.
**Source:** Ongoing calibration across VL031, VL025, JE100, JE102
**Applies to:** Section B (Decomposition), Section 10 (Consolidation Heuristic)

---

## Promoted Learnings
*(Already incorporated into the skill SKILL.md — kept for audit trail)*

### L008 — 2026-05-04 — [PROMOTED 2026-05-05 → SKILL.md v3.1]
**Type:** WORKFLOW
**Context:** JE005-06 creation — SKILL.md was read but truncated at line 125, missing the Description Template (Section 5) entirely. Story was created with a plain business context paragraph instead of the required `<h2>User Story</h2>` + "As a revenue accountant..." format.
**Learning:** The `eam-user-story-template/SKILL.md` is ~1100 lines. When `view` is called without a range, it truncates after ~125 lines — before Section 5 (Description Template) and Section 6 (AC Template). ALWAYS paginate through the full SKILL.md in at least two `view` calls (lines 1–550, then 550–end) before drafting any story. Do NOT proceed with story writing after reading only the first chunk.
**Example:** First read returned workflow steps only. Sections 5 and 6 — the actual templates — are at lines 350–500 and were missed entirely.
**Source:** User feedback — JE005-06 creation session 2026-05-04
**Applies to:** Step A (Gather Context) of the Story Generation Workflow
**Promoted into:** ⛔ MANDATORY PRE-FLIGHT section (File Read Requirements), Step C (Draft Each Story warning), Step F (Pre-Creation Validation)

---

### L009 — 2026-05-04 — [PROMOTED 2026-05-05 → SKILL.md v3.1]
**Type:** WORKFLOW
**Context:** JE005-06 creation — the draft in the previous message already had the wrong description format, but Claude carried it forward into ADO creation without re-validating against the template.
**Learning:** Before calling `wit_add_child_work_items`, perform an explicit self-check: does the description HTML start with `<h2>User Story</h2>` and include the "As a [role], I want... so that..." statement? If not, rewrite before creating. Never carry a draft forward from a prior message without verifying it against the Description Template (Section 5).
**Example:** JE005-06 draft was written before reading the template. When Aaron said "go ahead," Claude pushed the non-compliant draft to ADO instead of re-validating first.
**Source:** User feedback — JE005-06 creation session 2026-05-04
**Applies to:** Step F (Create in ADO) — pre-flight description format check
**Promoted into:** ⛔ MANDATORY PRE-FLIGHT section (Pre-Creation Validation Gate), Step F (Pre-Creation Validation block), Ad-Hoc Story Creation workflow

---

## Rejected Learnings
*(Decided not to promote — reason noted)*

- **L006** (2026-04-21) — Rejected 2026-05-05. Learning was factually backwards: stated work item type should be "Requirement" when it is actually "User Story" in the Quorum project. See L006 entry above for full details.
