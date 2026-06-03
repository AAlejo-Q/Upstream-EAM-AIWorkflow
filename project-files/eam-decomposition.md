# EAM Story Decomposition Guide

> **Version:** 1.0 | **Last Updated:** 2026-05-14
> **Load this file:** When planning a new story map — before presenting the story plan in Phase 2. Not needed for single ad-hoc story fixes.
> **What this file covers:** Universal story sequence, special capability slots, consolidation rules, SP estimation heuristics, risk levels, open questions checklist.

---

## Universal Story Sequence (v6.1 Standard)

All screens — regardless of migration pattern (Lift and Shift, Redesign, or Combine) — follow this fixed structure.

| Seq | Category | Standardized Title Pattern | Tag | Parent Feature | Successor Chain? |
|-----|----------|---------------------------|-----|----------------|-----------------|
| 01 | Foundation + CRUD | `[ScreenID]-01: Screen Foundation, Navigation & Record CRUD` | `eam-phase-ui-crud` | Screen's functional Feature | ✅ Chain start |
| 02 | Business Rules | `[ScreenID]-02: Business Rules and Validations` | `eam-phase-validations` | Screen's functional Feature | ✅ |
| 03 | Notes | `[ScreenID]-03: Notes` | `eam-phase-special` | Screen's functional Feature | ✅ |
| 04 | Audit History | `[ScreenID]-04: Audit History — Change History & Deleted Records Viewer` | `eam-phase-special` | Screen's functional Feature | ✅ |
| 05 | PPN | `[ScreenID]-05: PPN Triggers and Creation` | `eam-phase-special` | Screen's functional Feature | ✅ |
| 06+ | Special | `[ScreenID]-06: [Special Functionality Name]` | `eam-phase-special` | Screen's functional Feature | ✅ |
| Last | QA | `[ScreenID]-[N]: Testing & QA Parity` | `eam-phase-qa` | Screen's functional Feature | ✅ Chain end |
| Security | Security satellite | `[ScreenID]-Security: Security & Access Control` | `eam-phase-security` | **Feature 1786977** | ❌ Excluded |

> Security satellite stories use a 4-AC structure (menu navigation, full access, read-only, no access = not visible in menu). See `eam-ac-library.md` Section 7 for the complete standard. Use "security group" not "role". Do not reference QARCH.

### Universal Rules

- **Foundation + Navigation + CRUD always merged** into -01. Title always: `[ScreenID]-01: Screen Foundation, Navigation & Record CRUD`.
- **No security content in numbered stories** — all numbered stories (-01 through -QA) assume the user has complete access. Security analysis lives exclusively in the `-Security` satellite.
- **Business Rules always -02, standalone** — never merged with CRUD. Allows parallel dev pickup.
- **Special stories follow fixed order** — Notes (-03), Audit History (-04), PPN (-05), Other (-06+). Skip slots that don't apply and continue numbering sequentially.
- **QA always last numbered story** — title: `[ScreenID]-[N]: Testing & QA Parity`.
- **Security satellite always under Feature 1786977** — never the screen's functional Feature.
- **Successor links mandatory** — chain numbered stories (-01 → -02 → ... → -QA) after creation. Satellites excluded.
- **No SP estimates** — story point sizing is the team's responsibility.
- **No Detail View story** for inline-edit grid screens — fold view-mode behavior into -01.

---

## Story Naming Convention

```
[ScreenID]-[Seq]: [Brief Action Description]     (numbered)
[ScreenID]-Security: Security & Access Control   (satellite)
```

- `ScreenID` = screen code from screen inventory (e.g., VL031, TS005, CW010)
- `Seq` = two-digit, zero-padded (01, 02, 03…)
- Description = concise action phrase; max ~80 characters (fits ADO board cards)

**Examples:**
```
JE020-01: Screen Foundation, Navigation & Record CRUD
JE020-02: Business Rules and Validations
JE020-03: Notes
JE020-03: Audit History — Change History & Deleted Records Viewer
JE020-05: PPN Triggers and Creation
JE020-06: Mass Changes
JE020-07: Testing & QA Parity
JE020-Security: Security & Access Control
```

**Enabler / Spike:**
```
[ScreenID]-[Seq]: Spike — [Research Objective]
```

---

## Special Capability Slots

Each slot is added only when the screen has that capability. Skip inapplicable slots; continue numbering sequentially.

| Slot | Capability | Trigger Condition |
|------|-----------|-------------------|
| -03 | Notes | Screen has a user notes/comments panel |
| -04 | Audit History | Screen records field-level change history viewable by users. See `eam-ac-library.md` Section 6 for the mandatory 8-AC two-tab standard (Change History tab = record-scoped; Delete History tab = entity-wide). |
| -05 | PPN | Screen generates Prior Period Notification records |
| -06+ | Other Special | Mass Changes dialog, Batch Integration, Child Grid management, Advanced Business Rules, File Upload, or any other distinct special capability |

---

## Standard Story Map Reference

| Screen Type | Numbered Stories | Satellites |
|---|---|---|
| No special functionality | -01, -02, -[N] QA | -Security |
| With Notes only | -01, -02, -03 Notes, -[N] QA | -Security |
| With Audit History only | -01, -02, -04 Audit History, -[N] QA | -Security |
| With Notes + Audit History | -01, -02, -03 Notes, -04 Audit, -[N] QA | -Security |
| With Notes + Audit + PPN | -01, -02, -03, -04, -05 PPN, -[N] QA | -Security |
| Full special stack | -01, -02, -03, -04, -05, -06+ Other, -[N] QA | -Security |

---

## Design Approach — Story Count Guidance

The migration pattern affects story count and complexity, not structure.

| Pattern | Baseline | Typical Count | Guidance |
|---|---|---|---|
| Lift and Shift | -01, -02, -QA + satellites | 3–4 numbered + 1 satellite | Fewest special stories; consolidate aggressively |
| Lift and Shift with minor redesign | -01, -02, specials as needed, -QA + satellites | 4–7 numbered + 1 satellite | Add special stories only for capabilities that differ from Classic |
| Redesign | -01, -02, specials as needed, -QA + satellites | 5–9 numbered + 1 satellite | May need child grid, advanced rules, or dynamic UX stories |
| Combine (2+ screens) | -01, -02, specials as needed, -QA + satellites | 7–11 numbered + 1 satellite | Add one story per merged screen's unique workflow |

---

## Consolidation Rules

Bias toward consolidation — fewer, broader stories are easier to plan, pick up, and test.

### Merge When ALL of These Are True

1. Combined SP ≤ 8 (can fit in a single iteration)
2. Same user workflow or screen area (same entity or user action)
3. One story has only 1–3 SP (too small to track independently, or a natural pair)
4. Would be tested together in a parity session or story validation

### Consolidation Decision Table

| Scenario | Merge? | Rationale |
|---|---|---|
| Auto-population defaults + core validation | ✅ Yes | Both part of "applying rules when user saves" |
| Detail View + CRUD on same record (inline-edit grid) | ✅ Yes | Inline editing consolidates both in -01 |
| Multiple child grid types with same add/edit/delete pattern | ✅ Yes | Same behavior, different data |
| Contextual Navigation (< 2 SP) + Foundation+CRUD | ✅ Yes | Navigation is incidental; no dedicated story |
| Query + Pagination | ✅ Yes | Pagination is inherent to Query, always in -01 |
| Notes/Comments + Foundation+CRUD (standard notes pattern) | ✅ Yes | If notes use standard wiki §12 pattern with no extra behavior |
| Core Rules + Advanced Rules (combined > 8 SP or > 10 rules) | ❌ No | Split into -02 Core + -06 Advanced |
| Field Behavior (status-driven, complex) + Foundation+CRUD | ❌ No | Complex field enable/disable logic needs a dedicated story |
| PPN Audit Trail + Business Rules | ❌ No | Audit trail is separate system responsibility |
| Security + any numbered story | ❌ Never | Security always standalone satellite under Feature 1786977 |

### Specific Consolidation Patterns

- **No separate Detail View** — inline-edit grids don't need a Detail View story. Fold into -01.
- **One Business Rules story (-02)** — merge auto-population, field validation, and cross-reference checks into -02 unless combined SP > 8 or rule count > 12.
- **Notes & Comments merge into -01** when they use the standard §12 pattern with no special behavior beyond add/edit/delete.
- **Contextual Navigation merges into -01** when the nav links cost < 2 SP.
- **No Conditional Field Behavior story** unless the screen has product-type or status-driven field enabling that adds ≥ 5 SP.

---

## Story Splitting Patterns

### Pattern 1: Workflow Steps (Default)

Split by natural user workflow. This is the standard v6.1 decomposition: Foundation+CRUD → Business Rules → Special capabilities → QA.

### Pattern 2: Business Rule Overload

When a screen has 10+ rules and combined SP > 8, split into:
- -02: Core Rules (required fields, basic validation, auto-population)
- -06: Advanced Rules (cross-table lookups, cascade operations, effective date overlap, guard conditions)

### Pattern 3: Parent-Child Tables

When a screen manages multiple related tables:
- -01: Header/parent table CRUD
- -03 (or -06): Child table(s) CRUD — can merge multiple children if they share the same add/edit/delete pattern

### Pattern 4: Spike First

When uncertainty is high (file upload, batch integration, novel UI pattern):
- Spike: `[ScreenID]-[Seq]: Spike — [Research Objective]` — time-boxed, produces a findings doc
- Implementation stories: Created after spike findings are reviewed

---

## Story Point Estimation Heuristics

> **Note:** Claude does not set SP in ADO. These heuristics are for planning discussions and team calibration. Sizing is always the team's responsibility.

### Fibonacci Scale

| SP | Complexity | EAM Example |
|----|-----------|-------------|
| 1 | Trivial | Toggle a screen-level permission |
| 2 | Simple — single clear behavior | Add one lookup dropdown to an existing screen |
| 3 | Small — narrow, well-understood scope | Security satellite for a simple single-entity screen |
| 5 | Medium — multiple behaviors, manageable complexity | -01 for a simple filter-detail screen |
| 8 | Large — broad scope, multiple related behaviors | -01 with filter, pagination, full CRUD + effective date logic |
| 13 | Very Large — split recommended | Candidate for decomposition |
| 20 | Epic-sized — must split | Not appropriate for a single story |

### -01 Foundation + CRUD Sizing

| Scope | SP |
|---|---|
| Simple screen, read-only grid only | 5 |
| Standard filter-detail screen with CRUD | 8 |
| Complex CRUD with effective date history rows | 8–13 (consider splitting child grids) |

### -02 Business Rules Sizing

| Rule Count | Cross-Checks | SP |
|---|---|---|
| 1–2 rules (auto-population only) | None | 3 |
| 3–5 rules (required fields, basic validation) | 1–2 simple lookups | 5 |
| 6–10 rules with cross-dependencies | 3–5 cross-checks or overlap detection | 8 |
| 11+ rules OR complex date overlap | Multiple guard conditions | Split into Core + Advanced |

### Estimation Factors

- **Mass change dialogs** — add a separate 5–8 SP special story
- **Batch process integration** — add a separate 5–8 SP special story
- **File upload/import** — add a separate 8 SP story (highest risk pattern)
- **PPN audit trail** — add a separate 3–5 SP story when the screen writes PPN records
- **Security satellite** — typically 3–5 SP; higher for screens with complex security group configurations or field-level access restrictions

---

## Risk Level Reference

| Level | Description | Action |
|---|---|---|
| 🟢 Low | Standard pattern, well-understood | Proceed |
| 🟡 Medium | Cross-table lookups, 10+ rules, moderate SP | Review in refinement before sprint |
| 🔴 High | Redesign, file parsing, batch integration, > 8 SP | Spike recommended; consider splitting |

### Common High-Risk Patterns in EAM

| Pattern | Risk | Notes |
|---|---|---|
| File upload/parsing (e.g., VL032 merge into VL031) | 🔴 | Blazor Server streaming, column mapping |
| Mass Changes dialogs (e.g., VL025 Mass End Date) | 🔴 | Three modes, field override logic |
| Revenue Selection/Submittal (VL100) | 🔴 | 6 tabs, 8 DB views, 4 staging tables, batch integration |
| Cascade delete with guard conditions | 🔴 | Cross-table integrity, user confirmation dialogs |
| Effective date overlap validation | 🟡 | Date-range intersection logic with existing records |
| Child grid management | 🟡 | Depends on whether child follows same pattern as parent |

---

## Open Questions Checklist

Capture these before development begins. Flag anything unanswered.

- [ ] **Security:** What roles/permissions does this screen require? Any field-level access restrictions? Changes from the legacy security model?
- [ ] **Audit trail:** Does this screen generate PPN records? For which operations (insert, update, delete)?
- [ ] **Batch process:** Does this screen trigger any background jobs? What are the business trigger conditions?
- [ ] **Configuration:** Any system settings that change screen behavior per deployment (e.g., delete allowed, fields required)?
- [ ] **Field scope:** Are all legacy fields in scope for launch, or can rarely-used fields be deferred?
- [ ] **Business rule changes:** Any legacy rules being modified, added, or removed? Or strict parity?
- [ ] **Workflow changes:** Does the migration introduce workflow changes (combined screens, new nav paths), or is it pure lift-and-shift?

---

## Feature Alignment Checklist

Before finalizing the story map:

- [ ] Every Feature acceptance criterion is covered by at least one story's ACs
- [ ] The story set covers all screens listed in the Feature's Screen Inventory
- [ ] Dependencies listed in the Feature are reflected as risk notes in the story map
- [ ] No story exceeds 8 SP (if it does, split it)
- [ ] A QA/Parity story exists as the last numbered story
- [ ] A Security satellite story exists parented to Feature 1786977
- [ ] Successor links chain all numbered stories (-01 through -QA)
