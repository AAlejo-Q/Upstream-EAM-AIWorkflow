---
name: eam-user-story-template
description: "Template for generating EAM User Story drafts in Azure DevOps. Use when asked to generate, draft, write, or create user stories, story maps, or work items for any EAM screen migration. Also triggers on: 'draft stories for [screen]', 'create a story map', 'write user stories'. v6.1 standard: -01 Foundation+Nav+CRUD, -02 BRs with standard validation patterns, -03+ Special, -QA last. Satellites: -Security (Feature 1786977), -API (Feature 1806481). No security in numbered stories. Successor links chain -01 to -QA. Mandatory -01 ACs: navigation+menu (§16); query+results+empty (§1+§4); record view & selection+retrieve/more/all (§26) standalone; pagination+grid defaults (§3+§21/§22/§23/§25) as last AC with 100-record default, dropdown to 200. Mandatory -02 patterns: required fields, entity existence, duplicate prevention, pick list integrity. Always presented for review before ADO creation. No SP estimation — sizing is the team's responsibility."
---

# EAM User Story Authoring Template

> **Version:** 6.1 | **Last Updated:** 2026-05-14
> **Project:** Enterprise Accounting Modernization (EAM)
> **ADO Project:** Quorum (dev.azure.com/QuorumSoftware)
> **Exemplars:** VL031 (Redesign, 10 stories), VL025 (Lift and Shift, 8 stories — revised), JE100 (Lift and Shift, 8 stories), JE102 (Lift and Shift, 5 stories), JE005 (Lift and Shift, 6 stories), JE010 (v3 standard, 5 stories — Feature 1786978) — all under Features 1786972, 1792329, 1786978. **Note:** These exemplars predate v6 structural changes and use the v5 story sequence. Forward-looking screens use the v6 standard below.
> **v6.0 changes (2026-05-14):** (1) **Security split** — Security & Access Control is now a standalone satellite story (`[ScreenID]-Security`) parented to Feature 1786977, with its own `eam-phase-security` tag. No security content in any numbered story — all numbered stories assume the user has complete access. (2) **Foundation + CRUD merged** — Screen Foundation, Navigation, Query & Record CRUD consolidated into `-01` as a single story. (3) **Sequence renumbered** — BRs move to `-02`, special stories start at `-03`. (4) **Satellite story naming** — Security uses `[ScreenID]-Security:` suffix; API uses `[ScreenID]-API:` suffix. Neither uses a numeric sequence number. (5) **Successor links** — numbered stories (`-01` through `-QA`) are chained via ADO Successor relationship in a third creation step. Security and API stories are excluded from the chain. (6) **Tag update** — `eam-phase-query-ui` and `eam-phase-crud` replaced by `eam-phase-ui-crud`; new `eam-phase-security` tag added. (7) **AC consolidation** — Query filter + results grid + empty results state combined into one AC; Pagination + Retrieve/More/All combined into one AC. (8) **Universal standard** — Track A / Track B distinction removed; all screens (L&S, Redesign, Combine) follow the same decomposition structure.
> **v6.1 changes (2026-05-14):** (1) **-01 title shortened** — removed "Query" since Read is already part of CRUD; title is now `Screen Foundation, Navigation & Record CRUD`. (2) **Navigation AC mandatory** — -01 must include an explicit AC specifying which module group the screen appears under in the application navigation menu, referencing §16. (3) **Record View & Selection + Retrieve/More/All is a standalone AC** (§26) — covers Record View and Selection field, Retrieve 5,000 batch, More appends, All loads full set, record count indicator. (4) **Pagination merged with Grid Interaction Defaults** (§3 + §21/§22/§23/§25) — combined into the last functional AC in -01 stories; covers 100-record default page size with dropdown up to 200, export, highlight, copy, bulk view/edit. This replaces the standalone grid defaults AC in -01 stories. (5) **-02 standard validation patterns added** — Business Rules stories must cover four standard validation categories as applicable: required field validation, entity/reference existence, duplicate prevention, and pick list value integrity (valid code table values). Each with paired positive/negative ACs.
> **Related skill:** `safe-popm-best-practices` — SAFe framework principles underlying this template
> **ADO connectivity:** Uses direct REST API via PAT token (per `/ado` skill). Falls back from MCP if unavailable.

---

## ⛔ MANDATORY PRE-FLIGHT — READ THIS FIRST

> **This section exists because Claude has repeatedly skipped the templates below when creating stories.** Every instruction in this box is non-negotiable. Violations produce wrong output that must be manually corrected in ADO.

### File Read Requirements

**This file is ~1100 lines.** A single `view` call will truncate it. You MUST read it in at least two paginated calls:
- **Call 1:** Lines 1–550 (covers Pre-Flight, Workflow, Decomposition, ADO Fields, Description Template, AC Template)
- **Call 2:** Lines 550–1104 (covers Estimation, Splitting, Examples, Appendices)

**If your view was truncated and you did not see Section 5 (Description Template) or Section 6 (AC Template), you have NOT read the template. Stop and re-read before writing any HTML.**

Before drafting ANY story — whether a full story map or a single ad-hoc story — also read:
- `/mnt/project/CLAUDE_INSTRUCTIONS_EAM_WORKFLOW.md` (5-phase workflow)
- `/mnt/project/EAM-Story-Authoring-Quick-Reference.md` (PO-voice checklist)

### Description HTML Skeleton (Mandatory Structure)

Every EAM story Description MUST use this exact HTML structure. No exceptions:

```html
<h2>User Story</h2>
<p>As a <strong>[role]</strong>, I want [capability] so that [business value].</p>

<h2>Description</h2>
<p>[2-3 sentences of business context.]</p>
<p><strong>Legacy reference:</strong> [Screen name (ScreenID)]</p>

<h3>Functional Scope</h3>
<ul>
  <li>[User-facing behavior 1]</li>
  <li>[User-facing behavior 2]</li>
</ul>

<h3>Out of Scope</h3>
<ul>
  <li>[What this story does NOT cover (covered in [ScreenID]-[Seq])]</li>
</ul>
```

**Roles:** `revenue accountant` (all functional stories), `QA analyst` (QA/Parity story only), `system administrator` (Security story only).
**DO NOT** include `<h2>Story Points: [N]</h2>` — SP goes only in the dedicated ADO fields.
**DO NOT** include a Business Rules section unless the story is explicitly a rules/validation story.
**DO NOT** include security content (roles, permissions, access restrictions, unauthorized user blocking) in any numbered story — security analysis lives exclusively in the `-Security` satellite story. All numbered stories assume the user has complete access.

### ADO Defaults (Verify Before Every Creation)

| Field | Value | Rule |
|---|---|---|
| Project | `Quorum` | Never use `QuorumSoftware` for stories |
| Work Item Type | `User Story` | Never use `Requirement` |
| Area Path | `Quorum\North America\Upstream\myQ Accounting Modernization` | **ALWAYS verify against a live sibling story in the same Feature before setting.** Never trust a memorized value. |
| Iteration Path | **Do NOT set** | Sprint assignment is Scrum Master's responsibility |
| Assigned To | The user creating the stories (do not hardcode; assign to whoever is running the session) | |

### Pre-Creation Validation Gate

Before calling `wit_add_child_work_items` or any ADO write API, verify ALL of the following:

- [ ] Description HTML starts with `<h2>User Story</h2>`
- [ ] "As a [role], I want... so that..." statement is present
- [ ] `<h2>Description</h2>` heading is present before the business context paragraph
- [ ] `<h3>Functional Scope</h3>` section is present
- [ ] `<h3>Out of Scope</h3>` section is present with cross-story references
- [ ] No `<h2>Story Points: [N]</h2>` in the Description
- [ ] No class names, table names, stored procedures, or method names anywhere
- [ ] **No security content in numbered stories** — all numbered stories (-01 through -QA) assume the user has complete access; security analysis lives exclusively in the `-Security` satellite story
- [ ] Area path verified against a live story in the target Feature (not from memory)
- [ ] ACs use Given/When/Then with nested `<ul>` inside `<li>` for "Then" clauses
- [ ] Every validation rule has BOTH a positive case (valid input accepted) AND a negative case (invalid input blocked) — paired ACs are never separated or dropped
- [ ] AC count reflects complete coverage — no AC has been omitted due to a count ceiling; completeness takes precedence
- [ ] **Input grid stories (-01 or child grid stories):** Include the "empty editable row available" AC referencing Section 6 of the EAM wiki (see Input Grid AC Standard below)
- [ ] **-01 stories:** Include a navigation AC specifying the module group and menu placement, referencing Section 16 of the EAM wiki
- [ ] **-01 stories:** Include a standalone Record View & Selection + Retrieve/More/All AC (§26) with explicit 5,000-record batch, append behavior, record count indicator
- [ ] **-01 stories:** Include the combined Pagination + Grid Interaction Defaults AC (§3 + §21/§22/§23/§25) as the **last** functional AC — 100-record default, dropdown to 200, export, highlight, copy, bulk view/edit. This replaces the standalone grid defaults AC in -01 stories
- [ ] **-02 stories:** Include standard validation patterns as applicable — required field validation, entity/reference existence, duplicate prevention, and pick list value integrity (valid code table values) — each with paired positive/negative ACs
- [ ] **Tags set on every story** — `eam-phase-*` tag populated in `System.Tags` during batch update (see Tag Reference in Section 3)
- [ ] **Security stories** are parented to Feature 1786977, not the screen's functional Feature
- [ ] **API stories** are parented to Feature 1806481, not the screen's functional Feature
- [ ] SP fields are NOT set — do not pass `Custom.SAFeStoryPoints` or `Microsoft.VSTS.Scheduling.StoryPoints`
- [ ] **Successor links** — after all numbered stories (-01 through -QA) are created, chain them via Successor relationship (see Section 12, Step 3)

**If any item fails, fix it before creating. Do NOT proceed with a known template violation.**

### Ad-Hoc Story Creation

When creating a **single story** in response to a developer question, stakeholder request, or gap analysis (not a full story map), the same template and validation rules apply. Abbreviated workflow:

1. Read this SKILL.md (full, paginated) + Quick Reference
2. Identify the parent Feature and verify area path against a sibling story
3. Draft the story using the Description and AC templates from Sections 5 and 6
4. Run the Pre-Creation Validation Gate above
5. Present the draft for approval
6. Create in ADO using the four-step pattern (Section 12)

**There is no shortcut for "just one story." Template compliance is mandatory regardless of story count.**

---

## Purpose

This skill is a **PO/PM-voice generation template** — when invoked, it produces a complete set of EAM User Story drafts for a given screen migration from the Product Owner's perspective. Stories define **WHAT** the user needs and **WHY** — never HOW the developer should implement it. Technical decisions (architecture, framework, patterns, code structure) are left to the development team during refinement and sprint planning.

The template follows SAFe POPM best practices: lean story writing, INVEST quality criteria, Fibonacci estimation, and Feature-to-Story alignment.

Every EAM screen migration should produce stories that follow this structure. The output is always a **draft for review first**, then ADO creation after approval.

---

## Story Generation Workflow

> **IMPORTANT: Do NOT stop to ask questions mid-generation.** Capture all uncertainties, assumptions, and ambiguities in the generation log. Complete the full pipeline, then present everything for review.

When this skill is invoked to generate stories for a screen, follow these steps in order:

### Step A — Gather Context (Read, Don't Ask)

Gather context from all available sources. If the user provided a feature description or path, read it. If information is missing, note it in the generation log and proceed with reasonable assumptions.

**Sources to consult (in order):**
1. **User-provided input** — feature description, screen ID, ADO work item, or specification path
2. **DRI Footprint Screens (live from SharePoint)** — fetch from SharePoint using Microsoft 365 MCP tools (`mcp__claude_ai_Microsoft_365__read_resource` or `mcp__claude_ai_Microsoft_365__sharepoint_search`). SharePoint URL: `https://qbsol.sharepoint.com/:x:/r/sites/ProductManagement/_layouts/15/Doc.aspx?sourcedoc=%7B2CD5FF1A-5CD0-4546-B5E7-BBE0D6ECB045%7D&file=DRI%20Footprint%20Screens.xlsx`. If MCP fails, fall back to local copy at `EAM/DRI Footprint Screens.xlsx`
3. **Legacy ClassicGUI source** — search for `QFrm{ScreenID}*.cs` in the relevant ClassicGUI repo to discover business rules, child grids, and workflows
4. **Parent Feature in ADO** — look up via REST API if a Feature ID is known
5. **SAFe POPM Workbook** — `Best Practice docs/01 POPM Workbook (2025.09.16).pdf` for lean story writing principles
6. **`safe-popm-best-practices` skill** — for Feature/Story alignment guidance

**Context to extract (log assumptions for anything you can't find):**
- Screen ID (e.g., `VL032`, `TS005`)
- Parent Feature ID in ADO
- Design approach: Lift and Shift / Redesign / Combine
- Business rules: What validations, auto-population defaults, and cross-reference checks does the legacy screen enforce? (Use legacy source to understand *behavior*, not to prescribe implementation)
- Data relationships: Does the screen manage one entity or parent-child relationships?
- Special workflows: file upload, mass changes, batch integration, PPN audit trail
- User personas: Who uses this screen and for what business purpose?

> **Note on legacy source analysis:** Read legacy ClassicGUI source to understand *what the screen does* — its business rules, user workflows, and data relationships. Do NOT carry over technical details (class names, stored procedures, framework patterns) into the stories. The developer decides HOW to implement. The PO defines WHAT and WHY.

### Step B — Determine Story Decomposition (Fewer, Broader)

Using the Screen Decomposition Guide (Section 3):
- **All screens** (Lift and Shift, Redesign, Combine) → apply the v6.1 universal decomposition (Section 3): Foundation+Nav+CRUD merged into -01; BRs always -02 (with standard validation patterns); special stories start at -03; QA last numbered; Security satellite under Feature 1786977; API satellite under Feature 1806481
- **Bias toward consolidation** — aim for fewer, broader stories rather than many granular ones. Merge aggressively using the consolidation heuristic (Section 10)
- Apply consolidation rules (merge small stories, split rules at 10+)
- Assign preliminary SP estimates using the Estimation Guide (Section 7)

Document your decomposition as a **Story Map Summary Table** before writing individual stories.

### Step C — Draft Each Story

For each story in the decomposition, produce:
- **Title** following the naming convention (`[ScreenID]-[Seq]: [Brief Action]`)
- **Description** as HTML following the Description Template (Section 5) — MUST start with `<h2>User Story</h2>` and include `<h2>Description</h2>`, `<h3>Functional Scope</h3>`, `<h3>Out of Scope</h3>`
- **Acceptance Criteria** as HTML following the AC Template (Section 6) — write every AC needed to fully specify the story's behavior; no count ceiling; validation rules must always include both positive and negative cases as paired ACs
- **No Story Points** — Claude does not estimate SP. SP is omitted from all story drafts and ADO creation. Sizing is the team's responsibility during sprint planning.

> **If you have not read Section 5 and Section 6 of this file in the current session, STOP and read them now before writing any HTML.** Proceeding without reading the templates produces non-compliant output that requires manual correction.

### Step D — Self-Review & Validate

For each story, verify:
- [ ] Clear **persona** (who), **action** (what), **benefit** (why)
- [ ] **Testable** acceptance criteria (a QA analyst can write tests from the ACs alone)
- [ ] No significant **overlap** with another story — if overlap found, merge them
- [ ] Passes **INVEST** checklist (Section 11)
- [ ] No SP set — sizing is the team's responsibility

Flag and resolve issues before proceeding. Do not present stories that fail self-review.

### Step E — Write Output Files

Write two files (create `EAM/output/` if it doesn't exist):

1. **`EAM/output/[ScreenID]-user-stories.md`** — Complete story draft in the Draft Output Format (below), including:
   - Story Map Summary Table
   - Full Story Details (Description HTML + AC HTML for each)
   - Risk assessment for high-risk stories
   - Open Questions (Section 14)

2. **`EAM/output/generation-log.md`** — Generation log documenting:
   - Sources consulted and what was found/not found
   - Assumptions made when information was missing
   - Consolidation decisions (which stories were merged and why)
   - Ambiguities in the source spec that need human input
   - Any deviations from the template and rationale

End with a clear prompt: *"Story map written to `EAM/output/[ScreenID]-user-stories.md`. Review the draft and generation log. Confirm to proceed with ADO creation, or provide feedback to revise."*

### Step F — Create in ADO (After Explicit Approval Only)

> **GATE: Do NOT proceed to ADO creation until the user explicitly confirms.** Acceptable confirmations: "confirmed", "go ahead", "create them", "approved", "looks good, create". If the user provides feedback instead, return to Step C and revise.

**Pre-Creation Validation (mandatory before any API call):**
1. Run every item in the Pre-Creation Validation Gate checklist (top of this file)
2. Fetch the area path from an existing sibling story in the target Feature — use the returned value, not a memorized string
3. If any check fails, fix the draft before calling the API

Only after explicit user approval AND passing the validation gate, execute the Four-Step ADO creation pattern (Section 12) using direct REST API via the `/ado` skill. Never create work items without confirmation. Never assume silence is approval.

---

## Draft Output Format

When generating a story draft, produce output in this exact structure:

### 1. Story Map Summary Table

```markdown
## [ScreenID] Story Map — [Screen Name]
**Feature:** [Feature ID] | **Total Stories:** N | **Total SP:** XX

| # | Title | SP | Category | Risk |
|---|-------|----|----------|------|
| 01 | [ScreenID]-01: ... | 5 | Foundation | 🟢 |
| 02 | [ScreenID]-02: ... | 8 | Query | 🟡 |
| ... | ... | ... | ... | ... |

**Decomposition decisions:**
- [Explain any merge/split choices]
```

### 2. Full Story Details (repeat for each story)

````markdown
---
### [ScreenID]-[Seq]: [Title]
**Category:** [Foundation / Query / CRUD / etc.] | **SP:** [N] | **Risk:** [🟢/🟡/🔴]

**Description (System.Description — HTML):**
```html
[Full HTML content]
```

**Acceptance Criteria (Microsoft.VSTS.Common.AcceptanceCriteria — HTML):**
```html
[Full HTML content with AC1, AC2, ... ACN]
```
````

### 3. Open Questions

```markdown
## Open Questions for [ScreenID]
- [ ] [Question 1]
- [ ] [Question 2]
```

### 4. Review Prompt

```
Story map draft complete. X stories, XX total SP.
Review above and confirm to create in ADO, or provide feedback to revise.
```

---

## Table of Contents

1. [Before You Start — Pre-Authoring Checklist](#1-before-you-start--pre-authoring-checklist)
2. [Story Naming Convention](#2-story-naming-convention)
3. [Story Map — Screen Decomposition Guide](#3-story-map--screen-decomposition-guide)
4. [ADO Field Mapping Reference](#4-ado-field-mapping-reference)
5. [Description Template (System.Description)](#5-description-template-systemdescription)
6. [Acceptance Criteria Template (Dedicated ADO Field)](#6-acceptance-criteria-template-dedicated-ado-field)
7. [Story Point Estimation Guide](#7-story-point-estimation-guide)
8. [Definition of Done](#8-definition-of-done)
10. [Story Splitting Patterns for Screen Migrations](#10-story-splitting-patterns-for-screen-migrations)
11. [INVEST Quality Checklist](#11-invest-quality-checklist)
12. [ADO Automation — Four-Step Creation Pattern](#12-ado-automation--four-step-creation-pattern)
13. [Worked Examples — VL031 (Redesign) & VL025 (Lift and Shift)](#13-worked-examples)
14. [Open Questions Checklist](#14-open-questions-checklist)
15. [Complexity and Risk Notes](#15-complexity-and-risk-notes)

---

## 1. Before You Start — Pre-Authoring Checklist

Before writing stories for a screen, confirm the following:

- [ ] **Parent Feature exists in ADO** — identify the Feature ID (e.g., 1786972 for DRI screens)
- [ ] **Feature Description reviewed** — read the Benefit Hypothesis, Screen Inventory, and Acceptance Criteria from the parent Feature to ensure story alignment
- [ ] **Screen inventory reviewed** — confirm which screens are in scope, their priority (P1/P2/P3), and design approach (Lift and Shift / Redesign / Combine)
- [ ] **Legacy screen behavior understood** — what does the user do on this screen today? What data do they manage? What validations does the system enforce? (Use legacy source to understand behavior, not to prescribe implementation)
- [ ] **Business rules understood** — what validations, defaults, and cross-checks does the current screen enforce? State them in plain business language
- [ ] **POC gap assessment reviewed** — pull from `/mnt/project/Effort-Priority-Matrix.md` (not screen-inventory.md); note the following fields for the screen:
  - **Effort** (S / M / L / XL) — overall implementation size signal
  - **BRs** (Business Rules count) — informs how many validation ACs to expect in the -03 story
  - **Web Migration Approach** (Lift and Shift / Redesign / Combine) — informs story count and complexity expectations
  - **Key Complexity Signals** — specific flags (e.g., audit history, batch integration, child grids, mass changes) that trigger supplemental stories after -03
  Do NOT use Critical Gaps, Medium Gaps, or parity % columns to generate questions or affect sizing.
- [ ] **Foundation dependencies confirmed** — Features 1786989 (Web Framework), 1786990 (Data Access), 1786977 (Security), 1786983 (Metadata)

---

## 2. Story Naming Convention

```
[ScreenID]-[Seq]: [Brief Action Description]
```

**Rules:**
- `ScreenID` = the screen code from the screen inventory (e.g., VL031, TS005, CW010)
- `Seq` = two-digit sequence number, zero-padded (01, 02, 03…)
- Description = concise action phrase — what does the story deliver?
- Maximum title length: ~80 characters (fits ADO board cards)

**Examples:**
```
JE020-01: Screen Foundation, Navigation & Record CRUD
JE020-02: Business Rules and Validations
JE020-03: Notes
JE020-04: Audit History
JE020-05: PPN Triggers and Creation
JE020-06: [Special Functionality Name]
JE020-07: Testing, QA Parity & Classic Screen Sign-Off
JE020-Security: Security & Access Control
JE020-API: REST API Endpoints for Account/Account Group Cross-Reference
```

**Enabler / Spike naming:**
```
[ScreenID]-[Seq]: Spike — [Research Objective]
[ScreenID]-[Seq]: Enabler — [Technical Capability]
```

---

## 3. Story Map — Screen Decomposition Guide

EAM screen migrations use a **universal decomposition structure** regardless of design approach (Lift and Shift, Redesign, or Combine). The v6 standard defines a fixed story structure with two satellite stories (Security and API) that parent to dedicated Features outside the screen's functional Feature.

> **v6.1 Standard (effective 2026-05-14):** Security split into standalone satellite story for elaborate access analysis. Foundation, Navigation & CRUD merged into -01. BRs always -02. Track A / Track B distinction removed — all screens follow the same universal structure. Successor links chain numbered stories.

---

### Universal Story Sequence (v6 standard)

| Seq | Category | Standardized Story Title Pattern | Tag | Parent Feature | In Successor Chain? |
|-----|----------|----------------------------------|-----|----------------|---------------------|
| 01 | Foundation + CRUD | `[ScreenID]-01: Screen Foundation, Navigation & Record CRUD` | `eam-phase-ui-crud` | Screen's functional Feature | ✅ Chain start |
| 02 | Business Rules | `[ScreenID]-02: Business Rules and Validations` | `eam-phase-validations` | Screen's functional Feature | ✅ |
| 03 | Notes | `[ScreenID]-03: Notes` | `eam-phase-special` | Screen's functional Feature | ✅ |
| 04 | Audit History | `[ScreenID]-04: Audit History` | `eam-phase-special` | Screen's functional Feature | ✅ |
| 05 | PPN | `[ScreenID]-05: PPN Triggers and Creation` | `eam-phase-special` | Screen's functional Feature | ✅ |
| 06+ | Special | `[ScreenID]-06: [Special Functionality Name]` | `eam-phase-special` | Screen's functional Feature | ✅ |
| Last | QA | `[ScreenID]-[N]: Testing, QA Parity & Classic Screen Sign-Off` | `eam-phase-qa` | Screen's functional Feature | ✅ Chain end |
| Security | Security | `[ScreenID]-Security: Security & Access Control` | `eam-phase-security` | **Feature 1786977** (Security) | ❌ Excluded |
| API | API | `[ScreenID]-API: REST API Endpoints for [Screen Name]` | `eam-phase-api` | **Feature 1806481** (API) | ❌ Excluded |

> **Key principle: No security content in numbered stories.** All numbered stories (-01 through -QA) assume the user has complete access. Screen-level security analysis (roles, permissions, field-level access restrictions, unauthorized user blocking) lives exclusively in the `-Security` satellite story. This allows a deeper, more thorough security analysis per screen without cluttering the functional stories.

> **Key principle: CRUD and Business Rules are always separate stories.** The -01 story covers Foundation, Navigation, and Record CRUD (Create, Read, Update, Delete). The -02 story covers all save-time validations, including standard patterns (required fields, entity existence, duplicate prevention, pick list integrity) and screen-specific rules. This allows one developer to pick up CRUD while another picks up validations in parallel.

> **Rule:** Always include both positive and negative test cases for every validation rule as paired ACs. Never truncate at an AC count ceiling — completeness takes precedence. Positive/negative pairs for the same rule must never be separated across stories or dropped.

> **Satellite stories (Security and API):** Both use non-numeric suffixes (`-Security`, `-API`), both parent to dedicated Features (1786977 and 1806481 respectively), and both are excluded from the successor chain. They are created alongside the numbered stories but are independent of the build sequence.

#### Special Functionality Stories (Slots -03, -04, -05, -06+)

The following special capability stories are added in this fixed order when applicable. Each gets its own sequential slot only when that capability is present on the screen.

| Slot | Capability | Story Title Pattern | Trigger |
|------|-----------|-------------------|---------|
| -03 | Notes | `[ScreenID]-03: Notes` | Screen has a user notes/comments panel |
| -04 | Audit History | `[ScreenID]-04: Audit History` | Screen records field-level change history |
| -05 | PPN | `[ScreenID]-05: PPN Triggers and Creation` | Screen generates Prior Period Notification records |
| -06+ | Other Special | `[ScreenID]-06: [Special Functionality Name]` | Mass Changes, Batch Integration, Child Grids, Advanced Rules, or any other distinct special capability |

When a slot is not applicable, skip it and continue sequential numbering with the next applicable story.

#### Standard Story Map Reference

| Screen type | Numbered Stories | Satellite Stories |
|---|---|---|
| No special functionality | -01 Foundation+CRUD, -02 BRs, -[N] QA | -Security, -API |
| With Notes only | -01, -02, -03 Notes, -[N] QA | -Security, -API |
| With Audit History only | -01, -02, -04 Audit History, -[N] QA | -Security, -API |
| With Notes + Audit History | -01, -02, -03 Notes, -04 Audit History, -[N] QA | -Security, -API |
| With Notes + Audit + PPN | -01, -02, -03 Notes, -04 Audit History, -05 PPN, -[N] QA | -Security, -API |
| Full special stack | -01, -02, -03 Notes, -04 Audit History, -05 PPN, -06+ Other, -[N] QA | -Security, -API |

**Universal rules:**
- **Foundation + Navigation + CRUD always merged** into -01; title always: `[ScreenID]-01: Screen Foundation, Navigation & Record CRUD`
- **No security content in -01 or any numbered story** — security lives exclusively in the `-Security` satellite story
- **Business Rules always -02** — title always: `[ScreenID]-02: Business Rules and Validations`; standalone, never merged with CRUD
- **Special stories follow fixed order** — Notes (-03), Audit History (-04), PPN (-05), Other (-06+); omit slots that don't apply, continue numbering sequentially
- **QA always last numbered story** — title: `[ScreenID]-[N]: Testing, QA Parity & Classic Screen Sign-Off`
- **Security always satellite, under Feature 1786977** — title: `[ScreenID]-Security: Security & Access Control`; parent Feature is always 1786977
- **API always satellite, under Feature 1806481** — title: `[ScreenID]-API: REST API Endpoints for [Screen Name]`; parent Feature is always 1806481
- **Tags are mandatory** — set the appropriate `eam-phase-*` tag on every story during batch update (see Tag Reference below)
- **Successor links are mandatory** — chain numbered stories (`-01 → -02 → ... → -QA`) via ADO Successor relationship after creation (see Section 12, Step 3). Security and API stories are excluded from the chain.
- **No SP estimates** — Claude does not estimate story points; sizing belongs to the team during sprint planning
- **Skip the Detail View story** for grid-based inline-edit screens — no separate form panel exists
- **Merge contextual navigation** into -01 (Foundation+CRUD) when < 2 SP of effort

---

#### EAM Phase Tag Reference

Every EAM story must have its phase tag set in the `System.Tags` field during the batch update step. Tags power Asher's progress dashboard.

| Story Category | Tag |
|---|---|
| Foundation + CRUD (-01) | `eam-phase-ui-crud` |
| Business Rules (-02) | `eam-phase-validations` |
| Notes (-03), Audit History (-04), PPN (-05), Special (-06+) | `eam-phase-special` |
| QA (last numbered) | `eam-phase-qa` |
| Security (satellite, Feature 1786977) | `eam-phase-security` |
| API (satellite, Feature 1806481) | `eam-phase-api` |

**How to set tags via MCP:**
```
{ "op": "Replace", "path": "/fields/System.Tags", "value": "eam-phase-ui-crud" }
```

Multiple tags are separated by semicolons: `"eam-phase-ui-crud; eam"`

---

### Design Approach Guidance

All screens follow the universal decomposition structure above. The design approach affects **story count** and **complexity**, not the structure.

| Design Approach | Baseline Stories | Typical Story Count | Guidance |
|---|---|---|---|
| Lift and Shift | -01, -02, -QA + satellites | 4–6 numbered + 2 satellites | Fewest special stories; consolidate aggressively |
| Lift and Shift with minor redesign | -01, -02, special as needed, -QA + satellites | 5–8 numbered + 2 satellites | Add special stories only for capabilities that differ from Classic |
| Redesign | -01, -02, special as needed, -QA + satellites | 6–10 numbered + 2 satellites | May need child grid, advanced rules, or dynamic UX stories; split business rules into Core + Advanced only when 10+ rules |
| Combine (merging 2+ screens) | -01, -02, special as needed, -QA + satellites | 8–12 numbered + 2 satellites | Add one story per merged screen's unique workflow |

> **Note:** The Detail View story (for screens with a dedicated form/panel view, not inline-edit grids) is treated as a special functionality story when applicable. Inline-edit grid screens do NOT get a Detail View story.

---

## 4. ADO Field Mapping Reference

### Fields to Populate Per Story

| Story Section | ADO Field Reference | Format | Populated Via |
|---|---|---|---|
| Title | `System.Title` | Plain text | `wit_add_child_work_items` (Step 1) |
| Description | `System.Description` | Html | `wit_add_child_work_items` (Step 1) |
| Acceptance Criteria | `Microsoft.VSTS.Common.AcceptanceCriteria` | Html | `wit_update_work_items_batch` (Step 2) |
| SAFe Story Points | `Custom.SAFeStoryPoints` | Integer (picklist) | `wit_update_work_items_batch` (Step 2) |
| Story Points (built-in) | `Microsoft.VSTS.Scheduling.StoryPoints` | Decimal | `wit_update_work_items_batch` (Step 2) |
| Assigned To | `System.AssignedTo` | Identity string | `wit_update_work_items_batch` (Step 2) |
| Parent Feature | `System.Parent` | Link (auto) | `wit_add_child_work_items` via `parentId` |

### Critical ADO Constraints

> **⚠️ CONSTRAINT: SAFe Story Points is a Fibonacci picklist.**
> `Custom.SAFeStoryPoints` only accepts: **1, 2, 3, 5, 8, 13, 20**
> Values like 4, 6, 7, 9, 10 will be **rejected** by the API. Round to the nearest Fibonacci value.

> **⚠️ CONSTRAINT: Two Story Points fields must BOTH be populated.**
> - `Custom.SAFeStoryPoints` — the SAFe picklist (visible in the Story form's "Planning" section)
> - `Microsoft.VSTS.Scheduling.StoryPoints` — the built-in field (visible on backlog/board column totals)
> Set both to the same value. If only one is set, the points may not appear on all board views.

> **⚠️ CONSTRAINT: Acceptance Criteria go in the DEDICATED field — not in Description.**
> The `Microsoft.VSTS.Common.AcceptanceCriteria` field is a separate Html field on the User Story form.
> Never embed ACs in `System.Description`. The `wit_add_child_work_items` API only supports
> title + description, so ACs must be populated in a follow-up `wit_update_work_items_batch` call.

> **⚠️ CONSTRAINT: Work item type is "User Story" in the Quorum project.**
> Not "Requirement" — that type exists in the QuorumSoftware project.

> **⚠️ CONSTRAINT: AssignedTo format.**
> `'Display Name <email@domain.com>'` — e.g., `'Asher Maddox <asher.maddox@quorumsoftware.com>'`
> Assign to the user who is running the current session — do not hardcode a specific person.

> **⚠️ CONSTRAINT: NEVER set Iteration Path when creating EAM stories.**
> Do NOT pass `iterationPath` in `wit_add_child_work_items` or set `System.IterationPath` in any REST or MCP call.
> Leave the iteration path as the project default (`Quorum`). Sprint assignment is done by the Scrum Master during PI/sprint planning — it is never set during story creation.

---

## 5. Description Template (System.Description)

The Description field uses Html format and follows this standardized structure. **Focus on WHAT and WHY — never HOW.** No class names, stored procedures, framework patterns, or implementation guidance. The development team owns all technical decisions.

```html
<h2>User Story</h2>
<p>As a <strong>[role]</strong>, I want [capability] so that [business value].</p>

<h2>Description</h2>
<p>[2-3 sentences of business context. Reference the persona, the screen,
and the specific workflow step this story addresses. Mention the design approach
(Lift and Shift / Redesign) if relevant. Describe the business purpose and
user need — not the technical implementation.]</p>

<p><strong>Legacy reference:</strong> [Screen name in Classic GUI — e.g., "MI Link Maintenance (VL025)"]</p>

<h3>Functional Scope</h3>
<p>[Describe the user-facing behaviors this story delivers: filter fields,
grid columns, form fields, business rules in plain language,
or whatever is specific to this story's scope.]</p>

<h3>Business Rules (if applicable)</h3>
<ul>
  <li>[Rule in plain language — e.g., "Remitter must be an approved Business Associate"]</li>
  <li>[Rule — e.g., "Effective date ranges must not overlap for the same link"]</li>
</ul>

<h3>Out of Scope</h3>
<ul>
  <li>[Explicitly state what this story does NOT cover — helps prevent scope creep]</li>
  <li>[Use cross-story references: "(covered in [ScreenID]-[Seq])" to clarify story boundaries]</li>
</ul>
```

> **⚠️ The Description ends after Out of Scope.** Do NOT include `<h2>Story Points: [N]</h2>` or any trailing sections. Story Points go exclusively in the dedicated ADO fields (`Custom.SAFeStoryPoints` and `Microsoft.VSTS.Scheduling.StoryPoints`), set via `wit_update_work_items_batch` after creation.

### Description Content Guidelines

- **User Story** section: Always use the SAFe user-voice format: `As a [role], I want [activity] so that [business value]`
- **Roles** used in EAM: **revenue accountant** (all numbered stories except QA), **QA analyst** (QA/Parity story only), **system administrator** (Security satellite story only). Do NOT use tax analyst, land analyst, royalty reporting analyst, or developer as personas in EAM stories.
- **Legacy reference**: Name the Classic GUI screen for traceability — but do NOT include class names, file paths, table names, or stored procedures. The developer will find the code.
- **Business rules**: State rules in plain business language that a non-technical stakeholder could validate. Example: "Gas products must have a standard pressure value" — NOT "Call `QBusinessRuleMILinkSetStandardPressureForGas` in the AfterInsert pipeline"
- **No technical notes**: Do not include framework patterns, inheritance guidance, service layer specifics, or code-level references. The team decides the implementation approach during refinement.
- **No child story breakdowns**: Sub-task decomposition is a team activity during sprint planning — the PO defines the story, not the task split.

### CRITICAL: PO-Voice Anti-Patterns to Avoid

**Never include these in stories — they signal leaking implementation details:**

| ❌ Anti-Pattern | ✓ Correct PO Voice | Category |
|---|---|---|
| Class names: `QUIControllerVL031`, `QBusinessRuleMILink...` | "Users can add new records directly" | Implementation detail |
| Table/Column names: `RONL_MANL_INPUT`, `ManualInputSeqNo`, `UpdateUser` | "Sequence Number", "Header Record" | Database schema |
| Stored procedures: `SELECT_RONL_MANL_INPUT_VL`, `sp_CalculateAmounts` | "Query records by remitter number" | SQL details |
| Method/Event names: `InitializeBusinessRules`, `VLManualInputHTDRowChanged` | "Apply business rules when the record loads" | Code structure |
| ViewModel properties: `RonlManlInputVM.HeaderData` | "The form displays header information" | Framework pattern |
| Framework/code patterns: "UIControllerType = typeof(...)", "inheritance from BaseController" | "The screen renders securely with proper permissions" | Architecture |
| Internal flags: "MI_FLAG_LOCK is set", "UpdateDate field changed" | "The record is locked and cannot be edited" | System state |
| Vague outcomes: "The record is saved" | "The record is created and the grid refreshes showing the saved record with all values intact" | Observable behavior |

**Why:** Developers need flexibility to implement. The PO defines WHAT users need and WHY. HOW to achieve it belongs in sprint refinement and code review.

---

## 6. Acceptance Criteria Template (Dedicated ADO Field)

ACs are written in Gherkin (Given-When-Then) format and stored in the `Microsoft.VSTS.Common.AcceptanceCriteria` field — **never in Description**.

### When to Use Wiki Section References vs. Custom ACs

**MANDATORY FORMAT — Wiki-referenced ACs always follow this two-part structure:**

> **RULE:** Every AC — whether it references a wiki section or not — MUST include a full Given/When/Then block written specifically for the screen. The wiki reference line is ALWAYS appended *after* the GWT block. It is NEVER a replacement for the GWT block.

**Wiki-referenced AC** (GWT block first, then Reference line with anchored link):
```html
<p><strong>AC1 — Query filter & results:</strong></p>
<ul>
  <li>Given I am on the [Screen Name] screen</li>
  <li>When I enter one or more filter criteria and execute the query</li>
  <li>Then the grid displays matching records; if no records match, a 'No records found' message is shown</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=1.-filter-%26-query-execution">Section 1 — Filter &amp; Query Execution</a> in the EAM Functionality Reference Wiki.</p>
```

**Custom AC** (no wiki reference — story-specific logic):
```html
<p><strong>AC1 — [Short label]:</strong></p>
<ul>
  <li>Given [precondition / context]</li>
  <li>When [action / trigger]</li>
  <li>Then [expected outcome with specific, testable detail]</li>
</ul>
```

> **⚠️ ANTI-PATTERN — NEVER DO THIS:** A citation-only line with no GWT block is WRONG and must never be used:
> ```html
> <!-- WRONG — no GWT block, no anchor in URL -->
> <p><strong>AC1 — Screen Access & Navigation:</strong> See <a href="...wiki-root-url...">P01 — Screen Foundation & Navigation</a> in the EAM AC Patterns Wiki. Screen display name: 'X'. Module group: 'Y'.</p>
> ```

### Wiki Section Reference Table

Link ACs to these sections in the **EAM Functionality Reference Wiki** (page 11638). The `anchor=` value must be appended to the base URL so the link deep-links directly to the correct section.

**Base URL:**
```
https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&pageId=11638&anchor={SECTION-ANCHOR}
```

| Wiki Section | Anchor value | Use For |
|---|---|---|
| 1. Filter & Query Execution | `1.-filter-%26-query-execution` | All query/filter ACs |
| 2. Inquiry Date | `2.-inquiry-date` | ACs with as-of date filtering |
| 3. Pagination | `3.-pagination` | ACs with pagination controls |
| 4. Empty Results State | `4.-empty-results-state` | ACs for no-results behavior |
| 5. Master-Detail Grid (Parent-Child) | `5.-master-detail-grid-(parent-child)` | Parent-child grid ACs |
| 6. Add Row — New Record Creation | `6.-add-row-%E2%80%94-new-record-creation` | New record creation ACs |
| 7. Delete with Confirmation Dialog | `7.-delete-with-confirmation-dialog` | Delete ACs |
| 8. Required Field Validation | `8.-required-field-validation` | Required field save-block ACs |
| 9. Blocking Validation Error | `9.-blocking-validation-error` | Business rule blocking error ACs |
| 10. Non-Blocking Warning | `10.-non-blocking-warning` | Warning message ACs |
| 11. PPN Audit Trail | `11.-ppn-audit-trail` | PPN/audit trail creation ACs |
| 12. Notes Panel | `12.-notes-panel` | Notes/comments ACs |
| 13. Confirmation Dialog Before Bulk or Destructive Action | `13.-confirmation-dialog-before-bulk-or-destructive-action` | Bulk/destructive action confirmation ACs |
| 14. Batch Process Submission | `14.-batch-process-submission` | Batch submission ACs |
| 15. Action Button State | `15.-action-button-state` | Toolbar button enable/disable ACs |
| 16. Screen Navigation & Menu Placement | `16.-screen-navigation-%26-menu-placement` | -01 Foundation+CRUD story navigation ACs |
| 17. Security — Screen-Level Access Control | `17.-security-%E2%80%94-screen-level-access-control` | **Security satellite story only** — never in numbered stories |
| 18. Context Navigation | `18.-context-navigation` | Cross-screen navigation with context passing ACs |
| 19. User ID & Timestamp on Every Change | `19.-user-id-%26-timestamp-on-every-change` | Audit timestamp ACs (CRUD stories) |
| 20. Effective Date Overlap Validation | `20.-effective-date-overlap-validation` | Effective date overlap blocking error ACs |
| 21. Export to Excel (All Grids) | `21.-export-to-excel-(all-grids)` | Export button ACs — **mandatory on all grid stories** |
| 22. Cell Highlight (All Grids) | `22.-cell-highlight-(all-grids)` | Row/cell highlight ACs — **mandatory on all grid stories** |
| 23. Copy to Clipboard (All Grids) | `23.-copy-to-clipboard-(all-grids)` | Ctrl+C clipboard copy ACs — **mandatory on all grid stories** |
| 24. Import from Excel Template (Input Grids) | `24.-import-from-excel-template-(input-grids)` | Excel import ACs — input grids only, PO-flagged |
| 25. Bulk View/Edit (All Grids) | `25.-bulk-view%2fedit-(all-grids)` | Bulk View/Edit mode ACs — **mandatory on all grid stories** |

> **⚠️ SECTIONS 21–25 ARE MANDATORY ON ALL GRID STORIES.** Every story that includes a grid — query, CRUD, child grid, or batch selection — must include a combined AC covering Sections 21, 22, 23, and 25. Section 24 (Import) applies only to PO-flagged input grids. These are standard grid defaults, not optional additions.
>
> **Combined grid defaults AC format** (place as the last functional AC in grid stories):
> ```html
> <p><strong>AC[N] — Grid interaction defaults (export, highlight, copy, bulk view/edit):</strong></p>
> <ul>
>   <li>Given records are displayed in the [grid name] grid</li>
>   <li>When I interact with the grid</li>
>   <li>Then the following standard grid behaviors are available:
>     <ul>
>       <li>Export to Excel button downloads a .xlsx of all displayed rows</li>
>       <li>Clicking a row highlights it; multiple rows within the visible viewport can be highlighted</li>
>       <li>Ctrl+C copies highlighted rows to clipboard in tab-delimited format that pastes correctly into Excel with column headers</li>
>       <li>Bulk View/Edit button switches the grid into an Excel-like mode with flexible multi-row highlighting and horizontal scrolling; [Exit only — read-only grid / Apply, Apply and Exit, Exit — input grid] button returns to the default grid view</li>
>     </ul>
>   </li>
> </ul>
> <p>Reference: See <a href="...anchor=21.-export-to-excel-(all-grids)">Section 21 — Export to Excel</a>, <a href="...anchor=22.-cell-highlight-(all-grids)">Section 22 — Cell Highlight</a>, <a href="...anchor=23.-copy-to-clipboard-(all-grids)">Section 23 — Copy to Clipboard</a>, and <a href="...anchor=25.-bulk-view%2fedit-(all-grids)">Section 25 — Bulk View/Edit</a> in the EAM Functionality Reference Wiki.</p>
> ```

**When to add a Reference line:** When the AC covers a standard UI behavior documented in the wiki — the link gives dev and QA a screenshot and behavioral spec. Always write the full GWT block for the specific screen first, then append the Reference line.

**When NOT to add a Reference line:** For story-specific validations, edge cases, or business rules that have no corresponding wiki section.

**When to write a fully custom AC (no Reference line at all):** For story-specific logic not covered by any wiki section (e.g., affiliate identifier validation, account number length rule, change history content).

### AC Template

**AC with wiki section reference (GWT block first, Reference line after):**
```html
<p><strong>AC1 — [Short label]:</strong></p>
<ul>
  <li>Given [precondition / context specific to this screen]</li>
  <li>When [action / trigger]</li>
  <li>Then [expected outcome with specific, testable detail]</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor={SECTION-ANCHOR}">Section N — Section Name</a> in the EAM Functionality Reference Wiki.</p>
```

**AC without wiki reference (story-specific logic):**
```html
<p><strong>AC2 — [Short label]:</strong></p>
<ul>
  <li>Given [precondition]</li>
  <li>When [action]</li>
  <li>Then [expected outcome]</li>
</ul>
```

### AC Quality Rules

- Each AC tests **one behavior** — not a laundry list
- ACs cover **happy path AND edge cases** (empty results, validation failures, unauthorized access)
- ACs are **testable** — a QA analyst can read the AC and write a test without asking questions
- ACs reference **business-visible field names and screen labels** — NOT database columns, table names, or stored procedures
- ACs include **expected error messages** when testing validation rules ("Then an error states '…'")
- ACs describe **observable outcomes** — test WHAT the user sees, not HOW the system implements it
- **No AC count ceiling.** Write every AC needed to fully specify the story's behavior. Completeness always takes precedence over brevity. A story with 9 or 10 ACs because the behavior requires it is better than a story with 8 ACs with a coverage gap.
- **Validation pairs are mandatory.** Every validation rule must have BOTH a positive case (valid input → save succeeds) AND a negative case (invalid input → save blocked). These two ACs must appear together in the same story and must never be separated or omitted due to count concerns.

### Input Grid Story — Mandatory Empty Editable Row AC

Every story that includes an input grid (the -01 Foundation+CRUD story, or child grid stories with input capability) **must** include the following AC. This is non-negotiable. The exact GWT and wiki reference must be present — the screen name and wiki link are the only variables.

```html
<p><strong>AC[N] — New record creation:</strong></p>
<ul>
  <li>Given I am on the [Screen Name] screen</li>
  <li>When I need to create a new [entity] record</li>
  <li>Then an empty editable row is available with required fields highlighted and ready for input</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor=6.-add-row-%E2%80%94-new-record-creation">Section 6 — Add Row — New Record Creation</a> in the EAM Functionality Reference Wiki.</p>
```

This AC applies to ANY story containing an input grid — not just -01. If a child grid story (-06+) has editable rows, it also gets this AC.

### Consolidated AC Patterns for -01 Foundation+CRUD Story

The -01 story covers Foundation, Navigation, and Record CRUD. To keep ACs focused and avoid redundancy, the following standard behaviors are consolidated into combined ACs:

#### Navigation AC (§16) — Mandatory in every -01 story

Every -01 story must include an explicit AC specifying where the screen is placed in the application navigation menu:

```html
<p><strong>AC[N] — Screen navigation &amp; menu placement:</strong></p>
<ul>
  <li>Given I am a user with access to the application</li>
  <li>When I navigate to the [Module Group Name] module group in the application navigation menu</li>
  <li>Then I can see and open the [Screen Name] screen ([ScreenID])</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/EAM-Screen-Migration-Functionality-Reference?anchor=16.-screen-navigation-%26-menu-placement">Section 16 — Screen Navigation &amp; Menu Placement</a> in the EAM Functionality Reference Wiki.</p>
```

#### Combined AC: Query Filter, Results Grid & Empty State (§1 + §4)

Consolidates query execution, results display, and empty results behavior into a single AC:

```html
<p><strong>AC[N] — Query filter, results grid &amp; empty state:</strong></p>
<ul>
  <li>Given I am on the [Screen Name] screen</li>
  <li>When I enter one or more filter criteria and execute the query</li>
  <li>Then:
    <ul>
      <li>Matching records are displayed in the results grid with all expected columns</li>
      <li>If no records match the filter criteria, a 'No records found' message is shown and the grid remains empty</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/EAM-Screen-Migration-Functionality-Reference?anchor=1.-filter-%26-query-execution">Section 1 — Filter &amp; Query Execution</a> and <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/EAM-Screen-Migration-Functionality-Reference?anchor=4.-empty-results-state">Section 4 — Empty Results State</a> in the EAM Functionality Reference Wiki.</p>
```

#### Record View & Selection and Retrieve Record Count (§26) — Standalone AC

Covers the Record View and Selection field and the Retrieve / More / All retrieval behavior. This is its own AC to keep retrieval mechanics clearly separated and testable.

```html
<p><strong>AC[N] — Record view &amp; selection and retrieve record count:</strong></p>
<ul>
  <li>Given I am on the [Screen Name] screen and query results are displayed</li>
  <li>When I interact with the results grid</li>
  <li>Then:
    <ul>
      <li>A Record View and Selection field is available for selecting and viewing individual records within the results</li>
      <li>When the total result set exceeds the loaded records, the Retrieve button loads the first 5,000 matching records into the results grid</li>
      <li>Clicking More appends the next 5,000 records to the grid — previously loaded rows remain and are not replaced</li>
      <li>Clicking All loads the complete matching result set into the grid</li>
      <li>A record count indicator displays the number of records currently loaded versus the total matching the query criteria (e.g., &quot;5,000 of 12,340 records loaded&quot;) and updates after each Retrieve, More, or All action</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/EAM-Screen-Migration-Functionality-Reference?anchor=26.-retrieve-record-count-%E2%80%94-retrieve-%2f-more-%2f-all">Section 26 — Retrieve Record Count — Retrieve / More / All</a> in the EAM Functionality Reference Wiki.</p>
```

#### Combined AC: Pagination & Grid Interaction Defaults (§3 + §21/§22/§23/§25) — Last Functional AC in -01

Consolidates page size controls with the standard grid interaction defaults. **In -01 stories, this combined AC replaces the standalone grid defaults AC (§21/§22/§23/§25)** — do not include both. Place as the **last functional AC** in the -01 story.

```html
<p><strong>AC[N] — Pagination &amp; grid interaction defaults:</strong></p>
<ul>
  <li>Given records are displayed in the [grid name] grid</li>
  <li>When I interact with the grid</li>
  <li>Then the following standard behaviors are available:
    <ul>
      <li>The grid displays a default of 100 records per page; a dropdown option is available to adjust the page size to view fewer or more records, up to 200 per page</li>
      <li>Export to Excel button downloads a .xlsx of all displayed rows</li>
      <li>Click / Shift+Click / Ctrl+Click highlights single, contiguous, or non-contiguous rows</li>
      <li>Ctrl+C copies highlighted rows to clipboard in tab-delimited format that pastes correctly into Excel with column headers</li>
      <li>Bulk View/Edit button switches the grid into an Excel-like mode with flexible multi-row highlighting and horizontal scrolling; [Apply, Apply and Exit, and Exit — input grid / Exit only — read-only grid] buttons are available to commit or discard changes and return to the default grid view</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="...anchor=3.-pagination">Section 3 — Pagination</a>, <a href="...anchor=21.-export-to-excel-(all-grids)">Section 21 — Export to Excel</a>, <a href="...anchor=22.-cell-highlight-(all-grids)">Section 22 — Cell Highlight</a>, <a href="...anchor=23.-copy-to-clipboard-(all-grids)">Section 23 — Copy to Clipboard</a>, and <a href="...anchor=25.-bulk-view%2fedit-(all-grids)">Section 25 — Bulk View/Edit</a> in the EAM Functionality Reference Wiki.</p>
```

> **Note for non-01 grid stories** (e.g., child grids, audit history): Use the standalone grid defaults AC (§21/§22/§23/§25) from the Grid AC Standard section above — do NOT include pagination details unless the grid has its own independent pagination controls.

> **When to use these consolidated ACs:** Always use the consolidated versions in -01 Foundation+CRUD stories. The -01 AC sequence is: (1) Navigation, (2) Query+Empty, (3) Record View & Selection + Retrieve/More/All, (4) CRUD ACs, (5) Pagination + Grid Defaults as the **last** functional AC. In -01 stories, the combined Pagination + Grid Defaults AC **replaces** the standalone grid defaults AC — do not include both. **Do not lose specificity when consolidating — every explicit detail (100-record default, dropdown up to 200, Record View and Selection field, 5,000-record batch size, append behavior, record count indicator) must be present.**

### Standard Business Rule Validation Patterns for -02 Stories

The -02 Business Rules story must include ACs for these four standard validation categories **when applicable to the screen**. Each category requires paired positive (valid → save succeeds) and negative (invalid → save blocked) ACs. These are in addition to any screen-specific business rules discovered during codebase analysis.

#### 1. Required Field Validation

```html
<p><strong>AC[N] — Required fields validated on save (negative):</strong></p>
<ul>
  <li>Given I attempt to save a record with one or more required fields left empty (e.g., [list key required fields for this screen])</li>
  <li>When I click Save</li>
  <li>Then the save is blocked and an error message identifies the missing required field(s)</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/EAM-Screen-Migration-Functionality-Reference?anchor=8.-required-field-validation">Section 8 — Required Field Validation</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — All required fields present allows save (positive):</strong></p>
<ul>
  <li>Given I have populated all required fields on a record</li>
  <li>When I save</li>
  <li>Then the record is saved successfully with no required-field error</li>
</ul>
```

#### 2. Entity/Reference Existence Validation

When a field references another entity (e.g., Account Number must exist in Account Setup, Remitter must exist as a Business Associate), include:

```html
<p><strong>AC[N] — [Entity] existence validated on save (negative):</strong></p>
<ul>
  <li>Given I enter a [Entity Name] that does not exist in [Source Screen/Table]</li>
  <li>When I save the record</li>
  <li>Then the save is blocked and an error message states that the [Entity Name] is not recognized and must exist in [Source Screen/Table]</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/EAM-Screen-Migration-Functionality-Reference?anchor=9.-blocking-validation-error">Section 9 — Blocking Validation Error</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — Valid [Entity] is accepted (positive):</strong></p>
<ul>
  <li>Given I enter a [Entity Name] that exists in [Source Screen/Table]</li>
  <li>When I save the record with all other required fields populated</li>
  <li>Then the record is saved successfully with no existence error</li>
</ul>
```

#### 3. Duplicate Prevention

When a screen enforces uniqueness on a combination of fields:

```html
<p><strong>AC[N] — Duplicate [entity/combination] is blocked (negative):</strong></p>
<ul>
  <li>Given a record already exists for the [describe the unique key combination]</li>
  <li>When I attempt to create a second record with the same [key combination]</li>
  <li>Then the save is blocked and an error message states that a [entity] for this combination already exists</li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/EAM-Screen-Migration-Functionality-Reference?anchor=9.-blocking-validation-error">Section 9 — Blocking Validation Error</a> in the EAM Functionality Reference Wiki.</p>

<p><strong>AC[N+1] — Unique [entity/combination] is accepted (positive):</strong></p>
<ul>
  <li>Given no record exists for the entered [key combination]</li>
  <li>When I save the record with all required fields populated</li>
  <li>Then the record is saved successfully with no duplicate-prevention error</li>
</ul>
```

#### 4. Pick List Value Integrity

When a screen has fields backed by code table pick lists:

```html
<p><strong>AC[N] — Invalid pick list value is blocked (negative):</strong></p>
<ul>
  <li>Given I enter a value in a pick list field that is not a valid code table option</li>
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

> **When to include these patterns:** Always analyze the screen's codebase for each of these four categories. If the screen enforces the validation, include the paired ACs. If a category does not apply (e.g., no pick list fields), omit it. These standard patterns are **in addition to** any screen-specific business rules (e.g., duplicate default prevention, effective date overlap, auto-population logic).

### AC Testability Checklist

**Testable ACs have:**
- [ ] Explicit precondition in "Given" (what is the starting state?)
- [ ] Clear action in "When" (what does the user do?)
- [ ] Specific, observable outcome in "Then" (what does the user see or experience?)

**Not testable:**
- ✗ "Then the record is saved" (vague outcome)
- ✓ "Then the record is created and the grid refreshes showing the saved record with all values intact"

**Not testable:**
- ✗ "Then validation fires" (which validation? what does it say?)
- ✓ "Then an error states 'Remitter must be approved before records can be created' and the save is blocked"

**Not testable:**
- ✗ "Given a record exists" (which record? in what state?)
- ✓ "Given an existing SOD DRI Link record with a status of 'New' that has not been approved"

---

## 7. Story Point Estimation Guide

### Scale (Fibonacci — ADO Enforced)

| SP | Complexity | EAM Example |
|----|-----------|-------------|
| **1** | Trivial — single config or setting change | Toggle a screen-level permission |
| **2** | Simple — one clear behavior, minimal unknowns | Add a single lookup dropdown to an existing screen |
| **3** | Small — well-understood, narrow scope | Security story for a simple single-entity screen |
| **5** | Medium — multiple behaviors, some complexity | Foundation+CRUD for a simple filter-detail screen |
| **8** | Large — broad scope, multiple related behaviors | Foundation+CRUD with filter, pagination, full record management and contextual navigation |

### Estimation Heuristics for Business Rule Stories

Business rule stories often vary widely in complexity. Use these heuristics to estimate:

| Scope | Rule Count | Cross-Checks | SP |
|-------|-----------|---|---|
| Simple defaults | 1–2 rules (auto-population only) | None | 3 |
| Core rules | 3–5 rules (required fields, basic validation) | 1–2 simple lookups | 5 |
| Complex rules | 6–10 rules with cross-dependencies | 3–5 cross-checks or overlap detection | 8 |
| Multiple stories | 11+ rules OR complex date overlap logic | Multiple record-level guard conditions | Split into two stories (Core + Advanced) |

**Examples:**
- **3 SP:** "Auto-populate Operating Business Segment when a contract type is selected"
- **5 SP:** "Validate that remitter is approved, contract exists, and effective dates don't overlap" (3 rules, 2 lookups)
- **8 SP:** "Check approved status, validate contract type, detect effective date overlaps across child records, prevent edit if in 'Final Lock' status" (5+ rules, cross-dependencies)

### Estimation for Foundation+CRUD Stories (-01)

| Scope | Key Fields | Behaviors | SP |
|-------|-----------|-----------|---|
| Simple screen, read-only grid | None | Foundation, query, results display only | 5 |
| Standard CRUD screen | 2–3 fields | Foundation, query, add, modify, remove records | 8 |
| Complex CRUD + effective date logic | 2–3 fields | Foundation, query, CRUD PLUS auto-create history rows when end-dating | 8–13 (consider splitting child grids out) |

**Example:** ML006-03 (SOD DRI Link CRUD & Effective Date Logic) = 8 SP because:
- Create new records (inline add + paste)
- Edit non-key fields
- Delete with confirmation
- **Plus** effective date auto-creation (when user end-dates an open-ended record, system creates new row)

This complexity warrants 8 SP, not 5.
| **13** | Very Large — **split recommended** | Should be decomposed into smaller stories |
| **20** | Epic-sized — **must split** | Not appropriate for a single story |

### Estimation Factors for Screen Migrations

- **Business rule count**: Business Rules are always a standalone -02 story (v6 standard) — never merged with Foundation+CRUD regardless of rule count. If rules involve complex date overlap or cascade guard conditions that push scope significantly, add a supplemental Advanced Rules story after -02.
- **Data relationships**: Parent-child screens are more complex than single-entity screens
- **Lookup fields**: Screens with many searchable pick fields add complexity to the -01 story
- **Mass change dialogs**: Add a separate 5–8 SP story
- **Batch process integration**: Add a separate 5–8 SP story
- **File upload/import**: Add a separate 8 SP story (highest risk pattern)
- **Audit trail (PPN)**: Add a separate 3–5 SP story when the screen writes PPN records
- **Security**: Typically 3–5 SP; higher for screens with complex role-based field-level access restrictions

---

## 8. Definition of Done

Apply to **all** EAM screen migration stories:

- [ ] All acceptance criteria met and verified against legacy Classic GUI behavior
- [ ] Tests written and passing (scope and approach determined by the development team)
- [ ] Code reviewed and merged to development branch
- [ ] Screen accessible via navigation menu with security enforced
- [ ] Deployed to staging environment
- [ ] Accepted by Product Owner

**Additional for QA/Parity stories:**
- [ ] Structured parity verification executed against both Classic and Web
- [ ] Sign-off document produced confirming functional equivalence
- [ ] Classic screen retirement ticket raised in ADO

---

## 10. Story Splitting Patterns for Screen Migrations

### Pattern 1: Workflow Steps (Primary Pattern)
Split by the natural user workflow: Foundation+CRUD → Business Rules → Special capabilities → QA (+ Security and API satellites)
This is the default decomposition for all EAM screens.

### Pattern 2: Business Rule Variations
When a screen has 10+ business rules, split into:
- Core Rules (field-level validation, required fields, range checks)
- Advanced Rules (cross-table lookups, cascade operations, guard conditions)
- Audit/PPN Rules (post-DML audit trail creation)

### Pattern 3: Simple → Complex
- Story 1: Read-only query with grid display
- Story 2: Add inline editing (add/modify/delete)
- Story 3: Add child grid relationships
- Story 4: Add mass changes or special operations

### Pattern 4: Parent-Child Tables
When a screen manages multiple related tables:
- Story A: Header/parent table CRUD
- Story B: Child table(s) CRUD (can merge multiple children if they share the same pattern)
- Story C: Parent-child consistency rules

### Pattern 5: Spike First
When uncertainty is high (e.g., file upload in web framework, batch process integration):
- Spike story: "Investigate [approach] — time-boxed to [N] SP"
- Implementation stories: Created after spike findings are documented

### Pattern 6: Consolidation Guidelines
Aggressively consolidate before expanding beyond the baseline:

1. **No separate Detail View** — inline-edit grid screens don't need it. Fold any view-mode behavior into the -01 Foundation+CRUD story.
2. **One Business Rules story** (not two or three) — merge auto-population, field validation, and cross-reference checks into a single -02 story unless combined SP > 8 or rule count > 12.
3. **No Conditional Field Behavior story** unless the screen has product-type or status-driven field enabling. For most screens this does not apply.
4. **Notes & Comments merge into -01** — unless comments require behavior beyond the standard pattern (e.g., separate notes panel with its own add/edit/delete workflow).
5. **Contextual Navigation merges into -01** — nav links to related screens (< 2 SP) do not need a dedicated story.

**Result:** A typical screen should produce 4–6 numbered stories + 2 satellites. Reserve 8+ numbered stories for screens with mass changes, batch integration, child grids, or a high rule count.

---

### Consolidation Heuristic

Merge two candidate stories into one when **all** of these conditions are met:
1. **Combined SP <= 8** (must fit in a single iteration)
2. **Same user workflow or screen area** (both relate to the same entity or user action)
3. **One story has only 1-3 SP** (too small to track independently, or natural pair)
4. **Would be tested together** in a parity session or user story validation

**Decision tree for common consolidation scenarios:**

| Scenario | Merge? | Rationale | Example |
|----------|--------|-----------|---------|
| Auto-population defaults + Core Validation | ✓ Yes | Both part of "applying rules when user saves" | "Operating Business Segment auto-populates" + "Remitter must be approved" → one 5 SP story |
| Detail View + CRUD on same record | ✓ Yes | Inline editing consolidates both in -01 | Grid query + inline edit (no separate form) → part of -01 |
| Multiple child grid types with same pattern | ✓ Yes | Same add/edit/delete behavior across different grids | Severance Tax grid + Marketing Adjustment grid → one special story |
| Contextual Navigation (< 2 SP) + Foundation+CRUD | ✓ Yes | Navigation is incidental; no dedicated story | "Pass remitter number from VL031 to ML006" → part of -01 |
| Query + Pagination | ✓ Yes | Pagination is inherent to Query, always in -01 | Results display with 500-row limit → part of -01 |
| Core Rules + Advanced Rules | ✗ No | Split if combined > 8 SP or > 10 rules | 6 core rules (5 SP) + 5 advanced rules (5 SP) → split into two stories |
| Field Behavior (status-driven) + Foundation+CRUD | ✓ Yes if SP < 3 | If behavior adds < 1 SP, include in -01 | Read-only fields when status='Locked' → merge into -01 if simple |
| Field Behavior (status-driven) + Foundation+CRUD | ✗ No if SP >= 5 | If behavior is complex, dedicate a story | Multiple conditional field enables based on product type → separate special story |
| Notes/Comments + Foundation+CRUD | ✓ Yes | Comments are UI embellishment to CRUD | User can attach notes to record → part of -01 |
| PPN Audit Trail + Business Rules | ✗ No | Audit trail is separate system responsibility | Rules validation ≠ downstream audit; keep separate |

> **Security is never merged.** Security is always a standalone satellite story parented to Feature 1786977. It is never consolidated into any numbered story.

---

## 10a. Business Rules Phrasing Guide

When writing business rules in story descriptions and ACs, use plain business language that a revenue accountant could validate. Never phrase rules in terms of internal systems, database state, or code behavior.

### Phras­ing Anti-Patterns and Corrections

| ❌ Anti-Pattern | ✓ Correct Business Language | Category |
|---|---|---|
| "When `MI_FLAG_LOCK` is set, prevent editing" | "When a record is locked for final approval, users cannot edit it" | Internal flag |
| "Call `QBusinessRuleSODValidateContract()` to check contract" | "Verify that the SOD contract exists and is in an approvable status" | Method names |
| "Query `RXRF_MI_SOD` filtered by ManualInputSeqNo" | "Find all linkages for this record" | Table/column names |
| "UpdateDate must be > CreateDate AND < GETDATE()" | "Effective dates must be in logical order and cannot be in the future" | SQL-ish logic |
| "Cascade delete child rows from TPOLN_... table" | "When a master link is deleted, all child records associated with it are also removed" | Technical cascade |
| "Check the 'APPROVED' flag in the header before allowing insert" | "Only create new records when the parent contract has been approved" | Internal state |
| "Effective date ranges must not overlap per RXRF_MI_SOD.ContractGroup + DRI" | "For the same SOD contract and DRI combination, effective date ranges cannot overlap" | Technical constraint |
| "`UpdateUser` and `UpdateDate` are populated" | "The system records who made the change and when" | System fields |

### How to Extract Business Rules from Legacy Code

When analyzing legacy ClassicGUI source code to understand business rules, translate code logic into business language:

**Legacy code:**
```csharp
if (!Remitter.IsApproved()) 
    throw new ValidationException("Remitter must have ApprovedFlag = Y");
```

**Business rule (for story description):**
> "Remitter must be an approved Business Associate"

**How to write in AC:**
> Given a remitter that is not approved
> When I attempt to create a new record with that remitter
> Then an error states 'Remitter must be approved' and the save is blocked

**Legacy code:**
```csharp
if (Record.EffectiveFromDate >= Record.EffectiveToDate)
    throw new ValidationException("Invalid date range");
```

**Business rule:**
> "Effective To Date must be after Effective From Date"

**Legacy code (cross-table check):**
```csharp
if (Contract.ApprovalStatus != "APPROVED") 
    return false;
```

**Business rule:**
> "The SOD contract must be approved before records can be linked to it"

### When to List Business Rules in Story Description

Include a "Business Rules" section in the Description when:
- The story is explicitly a business rules story (e.g., VL031-06)
- The screen has validation/guard conditions that constrain how data is entered
- Users need to understand constraints before attempting to use the feature

**Don't include:**
- Implementation details (how the rule is checked)
- Database field names or column references
- Technical constraint names (e.g., "CHECK constraints")
- System-generated fields like UpdateUser/UpdateDate (these are implicit)

---

## 11. INVEST Quality Checklist

Run this checklist against every story before creating it in ADO:

| Criterion | Check | Pass? |
|-----------|-------|-------|
| **I**ndependent | Can this story be developed without waiting for another story in progress? (Dependencies on completed stories are OK) | |
| **N**egotiable | Does the story describe WHAT, not HOW? Can the team choose the implementation approach? | |
| **V**aluable | Does the story deliver value to a user or enable future user value (enabler)? | |
| **E**stimable | Can the team estimate this with reasonable confidence? (If not → spike first) | |
| **S**mall | Is the story ≤ 8 SP? Can it be completed in a single iteration? | |
| **T**estable | Are the acceptance criteria specific enough to write automated tests? | |

---

## 12. ADO Automation — Four-Step Creation Pattern

> **IMPORTANT: Present ALL stories for user review before any ADO API calls. Only create work items after explicit user approval ("confirmed", "go ahead", "create them"). Never auto-create.**

ADO work item creation uses a four-step pattern: (1) create stories with title + description + parent link, (2) populate remaining fields via batch update, (3) create successor links between numbered stories, (4) verify. Use direct REST API calls via the `/ado` skill (PAT-authenticated). If MCP tools are available they may be used, but never depend on MCP — always fall back to REST.

### Step 1: Create Stories with Title + Description + Parent Link

For each story, create a User Story work item linked to the appropriate parent Feature. **Do NOT set `iterationPath` — leave it unset so stories land in the default `Quorum` iteration. Sprint assignment is the Scrum Master's responsibility during PI planning.**

**Parent Feature routing:**
- **Numbered stories** (-01 through -QA) → screen's functional Feature (e.g., 1786972)
- **Security story** (`-Security`) → Feature **1786977**
- **API story** (`-API`) → Feature **1806481**

```bash
# Create a single User Story as child of Feature
curl -s -u ":$ADO_PAT" \
  -H "Content-Type: application/json-patch+json" \
  -X POST "https://dev.azure.com/QuorumSoftware/Quorum/_apis/wit/workitems/\$User%20Story?api-version=7.1" \
  -d '[
    { "op": "add", "path": "/fields/System.Title", "value": "[ScreenID]-01: ..." },
    { "op": "add", "path": "/fields/System.Description", "value": "<h2>User Story</h2>..." },
    { "op": "add", "path": "/relations/-", "value": {
        "rel": "System.LinkTypes.Hierarchy-Reverse",
        "url": "https://dev.azure.com/QuorumSoftware/_apis/wit/workItems/[FEATURE_ID]"
    }}
  ]'
```

Create stories one at a time (or use `$batch` endpoint for bulk). Record the returned `id` for each.

### Step 2: Populate Remaining Fields via PATCH

For each created story, update the fields that weren't set in Step 1:

```bash
# Update AC and AssignedTo — do NOT set Story Points
curl -s -u ":$ADO_PAT" \
  -H "Content-Type: application/json-patch+json" \
  -X PATCH "https://dev.azure.com/QuorumSoftware/Quorum/_apis/wit/workitems/[STORY_ID]?api-version=7.1" \
  -d '[
    { "op": "add", "path": "/fields/Microsoft.VSTS.Common.AcceptanceCriteria",
      "value": "<p><strong>AC1...</strong></p>..." },
    { "op": "add", "path": "/fields/System.AssignedTo",
      "value": "[Creator's display name <email>]" },
    { "op": "add", "path": "/fields/System.Tags",
      "value": "eam-phase-ui-crud" }
  ]'
```

> **SP fields are NOT set during story creation.** Do not pass `Custom.SAFeStoryPoints` or `Microsoft.VSTS.Scheduling.StoryPoints`. Story point estimation is the team's responsibility during sprint planning.

> **Tags are mandatory.** Every story must have its `eam-phase-*` tag set in Step 2. Use the Tag Reference in Section 3 to select the correct tag per story category. Security stories get `eam-phase-security`. API stories get `eam-phase-api`.

> **Satellite story parent Features.** When creating Security stories in Step 1, the parent `FEATURE_ID` must be **1786977**. When creating API stories in Step 1, the parent `FEATURE_ID` must be **1806481**. Neither uses the screen's functional Feature ID.

### Step 3: Create Successor Links (Numbered Stories Only)

After all stories are created and their IDs are known, chain the **numbered stories** (-01 through -QA) via ADO Successor relationship. Security and API satellite stories are **excluded** from the chain.

The successor chain establishes the intended build order: `-01 → -02 → -03 → ... → -QA`

Using MCP (`wit_work_items_link`):
```
For each consecutive pair (storyA, storyB) in the numbered story sequence:
  wit_work_items_link(
    ids: [storyA_ID],
    linkToId: storyB_ID,
    type: "Successor"
  )
```

Using REST API:
```bash
# Link storyA → storyB as Successor
curl -s -u ":$ADO_PAT" \
  -H "Content-Type: application/json-patch+json" \
  -X PATCH "https://dev.azure.com/QuorumSoftware/Quorum/_apis/wit/workitems/[STORY_A_ID]?api-version=7.1" \
  -d '[
    { "op": "add", "path": "/relations/-", "value": {
        "rel": "System.LinkTypes.Dependency-Forward",
        "url": "https://dev.azure.com/QuorumSoftware/_apis/wit/workItems/[STORY_B_ID]",
        "attributes": { "comment": "Successor in screen story sequence" }
    }}
  ]'
```

> **Successor chain scope:** Only numbered stories under the screen's functional Feature participate. The chain runs from -01 (first) to -QA (last). Security (`-Security`) and API (`-API`) stories are excluded — they are independent satellite stories.

### Step 4 (Verification): Confirm All Fields Populated

```bash
# Fetch created stories and verify fields
curl -s -u ":$ADO_PAT" \
  "https://dev.azure.com/QuorumSoftware/Quorum/_apis/wit/workitems?ids=[ID1],[ID2],...&fields=System.Id,System.Title,Microsoft.VSTS.Common.AcceptanceCriteria&api-version=7.1"
```

Verify every story has ACs populated, is linked to the correct parent Feature, and has successor links in the correct sequence.

### MCP Fallback Note

If MCP tools (`wit_add_child_work_items`, `wit_update_work_items_batch`, `wit_get_work_items_batch_by_ids`) are available and working, they may be used instead of raw REST calls. But per workspace policy: if MCP fails to connect, immediately fall back to REST — do not suggest restarting or reconfiguring the MCP server.

---

## 13. Worked Examples

> **Note:** The examples below predate the v6 structural changes and use the v5 story sequence (separate Foundation, CRUD, and Security). They remain as historical reference for how those screens were decomposed. **Forward-looking screens** use the v6 universal structure: merged Foundation+CRUD in -01, Security as a satellite story, successor links between numbered stories.

### Example A — VL031 Story Map (Redesign / Combine — v5 structure)

VL031 (Direct Revenue Input) was a **Redesign** that incorporated the legacy VL032 upload workflow into a unified screen. 10 stories, Feature 1786972.

| # | ADO ID | Title | SP | Category |
|---|--------|-------|-----|----------|
| 01 | 1792434 | Technical Foundation, Security & Navigation | 5 | Foundation |
| 02 | 1792435 | Data Services & Query Screen (Header Grid) | 8 | Query |
| 03 | 1792436 | Read-Only Detail View & Code Table Loading | 3 | Detail View |
| 04 | 1792437 | Header Record CRUD — Create, Update, Delete | 8 | CRUD |
| 05 | 1792438 | Child Grids — Severance Tax & Marketing Adjustment CRUD | 8 | Child Grids |
| 06 | 1792451 | Core Field Validation Business Rules | 8 | Rules (Core) |
| 07 | 1792452 | Advanced Rules — VLA Guard, Cascade Delete, VL Final Lock | 8 | Rules (Advanced) |
| 08 | 1792453 | PPN Audit Trail Creation (RV36) | 5 | Audit |
| 09 | 1792454 | Dynamic Read-Only Behavior by Major Product Code | 3 | Dynamic UX |
| 10 | 1792455 | Testing, QA Parity & Classic Screen Sign-Off | 8 | QA |

**Key decisions:**
- **Story 03 (Detail View) included** — VL031 has a dedicated form panel that opens from the grid; it is not purely inline-edit
- **Merged** Tax child grid + Marketing child grid into Story 05 (same parent, same pattern)
- **Split** business rules into Core (Story 06, 13 rules) and Advanced (Story 07, VLA/cascade/VL-Final) — justified by 13+ rule count
- **Separated** PPN audit trail (Story 08) — runs in AfterUpdate container, distinct lifecycle
- **Separated** Dynamic read-only (Story 09) — product-code-driven field enabling is purely UX, no data changes

---

### Example B — VL025 Story Map (Lift and Shift — v5 structure, Revised)

VL025 (DRI Link Maintenance) is a **Lift and Shift** of a grid-based maintenance screen. Originally authored as 10 stories / 60 SP, then revised to **8 stories / 52 SP** by applying Track A consolidation rules. The revised map is the current exemplar for Lean L&S decomposition.

> **Revision draft:** `EAM/drafts/VL025-revised-story-map.md`

| # | Title | SP | Category | Risk | vs. Original |
|---|-------|----|----------|------|--------------|
| 01 | VL025-01: Screen Foundation, Security & Navigation | 5 | Foundation | 🟢 | Unchanged |
| 02 | VL025-02: DRI Link Query & Results Grid | 5 | Query | 🟢 | Unchanged |
| 03 | VL025-03: DRI Link CRUD, Notes & Contextual Navigation | 8 | CRUD | 🟡 | **Merged** original 03 + 09 |
| 04 | VL025-04: Business Rules — Auto-Population, Validation & Cross-Reference Checks | 8 | Rules | 🟡 | **Merged** original 04 + 05 |
| 05 | VL025-05: Advanced Rules — Effective Date Overlap & MI Processed Checks | 5 | Rules (Advanced) | 🔴 | Unchanged (renumbered from 06) |
| 06 | VL025-06: PPN Audit Trail | 5 | Audit | 🟡 | Unchanged (renumbered from 07) |
| 07 | VL025-07: Mass Changes — End Date, Override & Clone | 8 | Mass Changes | 🟡 | Unchanged (renumbered from 08) |
| 08 | VL025-08: Testing, QA Parity & Classic Screen Sign-Off | 8 | QA | 🟢 | Updated dependencies (renumbered from 10) |

**Total: 8 stories | 52 SP** (vs. original 10 stories / 60 SP)

**Key consolidation decisions applied:**
- **Eliminated Read-Only Detail View** — VL025 is a pure inline-edit grid screen; no separate form/panel exists. Per Track A rule: no dedicated Detail View story
- **Eliminated Dynamic UX story** — VL025 has no product-code-driven or state-driven conditional field enabling. Not applicable
- **Merged Notes & Contextual Navigation into CRUD (Story 03)** — Original Story 09 was 3 SP covering notes + a single nav link to VL031. Too thin to stand alone; both features touch the CRUD-layer row selection context. Combined story stays at 8 SP
- **Merged Auto-Population into Business Rules (Story 04)** — Original Stories 04 (Auto-Population, 5 SP) and 05 (Core Validation, 8 SP) share the same service layer: both fire in the `Save` / `AfterInsert` / `AfterUpdate` pipeline. Auto-pop re-estimated to 3 SP. Combined into one validation container at 8 SP
- **Kept Advanced Rules separate (Story 05)** — Effective date overlap and delete-with-MIs-check are high-risk, date-range-intersection logic with distinct 🔴 risk flag
- **Kept PPN separate (Story 06)** — Runs in the AfterUpdate container with a distinct lifecycle from save-time validation

**Net result: 2 stories fewer, 8 SP saved, same business value coverage, no SAFe violations.** Each merged story is independently deliverable and testable within a sprint. This was the target pattern for Lean L&S screens under the v5 standard; v6 further consolidates Foundation+Query+CRUD into -01 and separates Security into a satellite story.

---

## 14. Open Questions Checklist

Capture these questions during story authoring. They flag refinement needs before development begins.

- [ ] **Security:** What roles/permissions does this screen require? Are there field-level access restrictions? Any changes from the legacy security model? (Document in the `-Security` satellite story — do not include security content in numbered stories)
- [ ] **Audit trail:** Does this screen need to generate PPN (Prior Period Notification) records? For which operations (insert, update, delete)?
- [ ] **Batch process:** Does this screen trigger any background processing jobs? If so, what are the business trigger conditions?
- [ ] **Configuration:** Are there any system settings that change this screen's behavior per deployment (e.g., whether delete is allowed, whether certain fields are required)?
- [ ] **Field scope:** Are all legacy fields in scope for launch, or can rarely-used fields be deferred to a later PI?
- [ ] **Business rule changes:** Are any legacy business rules being modified, added, or removed as part of this migration? Or is it strict parity?
- [ ] **User workflow changes:** Does the migration introduce any workflow changes (e.g., combining screens, new navigation paths), or is it a pure lift-and-shift of existing behavior?

---

## 15. Complexity and Risk Notes

Include a brief risk assessment in the story map markdown output (not in ADO — this is for team review):

### Risk Levels

| Level | Description | Action |
|-------|------------|--------|
| 🟢 Low | Standard pattern, well-understood, < 5 SP | Proceed |
| 🟡 Medium | Cross-table lookups, 10+ rules, moderate SP | Review in refinement |
| 🔴 High | Redesign, file parsing, batch integration, > 8 SP | Spike recommended; consider splitting |

### Common High-Risk Patterns in EAM

- **File upload/parsing** (e.g., VL032 merge into VL031) — Blazor Server streaming, column mapping
- **Mass Changes dialogs** (e.g., VL025 Mass End Date) — three modes, field override logic
- **Revenue Selection/Submittal** (VL100) — 6 tabs, 8 DB views, 4 staging tables, batch integration
- **Cascade delete with VLA guards** — cross-table integrity, user confirmation dialogs
- **Effective date overlap validation** — date-range intersection logic with existing records

---

## Appendix A: Feature Alignment Checklist

Before finalizing stories, verify alignment with the parent Feature:

- [ ] Every Feature acceptance criterion is covered by at least one story's ACs
- [ ] The story set covers all screens listed in the Feature's Screen Inventory
- [ ] Dependencies listed in the Feature are reflected as "Depends On" notes in the story map
- [ ] The total SP estimate is realistic for the Feature's PI capacity allocation
- [ ] No story exceeds 8 SP (if it does, split it)
- [ ] A QA/Parity story exists as the last numbered story in the sequence
- [ ] A Security satellite story exists parented to Feature 1786977
- [ ] An API satellite story exists parented to Feature 1806481
- [ ] Successor links chain all numbered stories (-01 through -QA)

---

## Appendix B: Personas Reference

All EAM screen migration stories use **one of three personas only**:

| Persona | When to Use |
|---------|-------------|
| Revenue Accountant | Every numbered story except the QA/Parity story — foundation, query, CRUD, business rules, audit trail, mass changes, batch integration, etc. |
| QA Analyst | QA/Parity story (last numbered story) only |
| System Administrator | Security story (`[ScreenID]-Security`) only |

> **Do NOT use** tax analyst, land analyst, royalty reporting analyst, or developer as personas in EAM user stories. The Revenue Accountant is the primary user of all migrated screens and provides the consistent PO voice across the functional story set. The System Administrator persona is used only for the Security satellite story to frame access control from the administrator's perspective.

---

## Appendix C: ADO Environment Quick Reference

| Item | Value |
|------|-------|
| ADO Organization | QuorumSoftware |
| ADO Project (new work) | Quorum |
| ADO Project (repos) | QuorumSoftware |
| Project GUID | `ecaedfc6-005f-4ee9-aa66-6da8c71a6ad1` |
| Work Item Type | `User Story` |
| Area Path | `Quorum\North America\Upstream\myQ Accounting Modernization` |
| Iteration Path | **Do NOT set** — leave as default `Quorum` |
| Parent Epic (QRA MVP) | 1786991 |
| Security Feature (all Security stories) | 1786977 |
| API Feature (all API stories) | 1806481 |
| Aaron's User ID | `062b23a1-7b0c-6b7e-88c4-74164dac8460` |
| AssignedTo format | `'Display Name <email@quorumsoftware.com>'` — assign to the session creator, not a hardcoded person |
| SAFe SP Field | `Custom.SAFeStoryPoints` (Fibonacci picklist) |
| Built-in SP Field | `Microsoft.VSTS.Scheduling.StoryPoints` (decimal) |
| AC Field | `Microsoft.VSTS.Common.AcceptanceCriteria` (Html) |
| Description Field | `System.Description` (Html) |
| Html-capable fields | Description, AcceptanceCriteria, NonFunctionalRequirements, BusinessOutcome, Objective, InScope |

> **⚠️ AREA PATH VERIFICATION RULE:** Before setting `System.AreaPath` on any story, fetch the area path from an existing sibling story in the same Feature using `wit_get_work_items_batch_by_ids`. Use the returned value verbatim. Area paths have changed historically (`myQ UPS Modernization` → `myQ Accounting Modernization`) and memorized values may be stale.

---

*This template is a living document. Update it as new patterns emerge from subsequent screen migrations.*