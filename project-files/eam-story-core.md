# EAM Story Core — Templates & ADO Creation

> **Version:** 1.2 | **Last Updated:** 2026-07-07
> **Load this file:** Before writing any story HTML in any session. This is the minimum file needed for all story work.
> **What this file covers:** Description template, AC template, pre-creation validation gate, ADO defaults, two-step creation pattern, PO voice rules, tags, personas.
> **For AC wiki patterns and validation pair templates:** See `eam-ac-library.md`.
> **For decomposition and splitting rules:** See `eam-decomposition.md`.
> **v1.2 changes (2026-07-07):** `System.AssignedTo` is now hardcoded to Asher Maddox (`asher.maddox@quorumsoftware.com`) for every newly created story, regardless of who is running the session. Previously assigned to the session creator.

---

## Pre-Creation Validation Gate

Run this before every ADO write — full story map or single ad-hoc story. No exceptions.

### Critical 5 (Highest-Frequency Violations — Fix First)

- [ ] Description starts with `<h2>User Story</h2>` and includes "As a [role], I want... so that..."
- [ ] No security content in any numbered story (security lives exclusively in `-Security` satellite)
- [ ] Area path fetched live from a sibling story in the target Feature — never memorized
- [ ] No `Story Points` in Description HTML — SP goes only in dedicated ADO fields
- [ ] Every grid story has the combined §21/§22/§23/§25 AC as the last functional AC
- [ ] Parent link will be set via `wit_add_child_work_items` or explicit `wit_work_items_link type: "parent"` — **never via `System.Parent` field** (silently ignored by ADO)

### Full Checklist

- [ ] `<h2>Description</h2>` heading present before business context paragraph
- [ ] `<h3>Functional Scope</h3>` section present
- [ ] `<h3>Out of Scope</h3>` section present with cross-story references (`"covered in [ScreenID]-[Seq]"`)
- [ ] No class names, table names, stored procedures, or method names anywhere in Description or ACs
- [ ] ACs use Given/When/Then with nested `<ul>` inside `<li>` for "Then" clauses **only when there are 2+ distinct outcomes**; single-outcome Then clauses are written inline
- [ ] Every validation rule has BOTH a positive case (valid input accepted) AND a negative case (invalid input blocked) — paired ACs never separated or dropped
- [ ] -01 stories include: navigation AC (§16), query+results+empty AC (§1+§4), Retrieve/More/All AC (§26) — background cache language, no Record View bullet, When = "When I use the Retrieve, More, or All buttons", pagination + grid defaults AC (§3+§21/§22/§23/§25) as last AC
- [ ] -02 stories include all applicable standard validation patterns (see `eam-ac-library.md` Section 2)
- [ ] `eam-phase-*` tag set on every story (see Tag Reference below)
- [ ] Security stories parented to Feature **1786977**, tagged `eam-phase-security`
- [ ] SP fields NOT set — do not pass `Custom.SAFeStoryPoints` or `Microsoft.VSTS.Scheduling.StoryPoints`

**Fix all failures before creating. Do not proceed with known violations.**

---

## ADO Defaults

| Field | Value | Rule |
|---|---|---|
| Project | `Quorum` | Never use `QuorumSoftware` for stories |
| Work Item Type | `User Story` | Never use `Requirement` |
| Area Path | `Quorum\North America\Upstream\myQ Accounting Modernization` | **Fetch from a live sibling story before every creation** — never trust a memorized value |
| Iteration Path | **Do NOT set** | Defaults to root `Quorum`; sprint assignment is the Scrum Master's responsibility |
| Assigned To | `Asher Maddox <asher.maddox@quorumsoftware.com>` | **Hardcoded** — every newly created story is assigned to Asher Maddox regardless of who is running the session |

---

## EAM Phase Tag Reference

Set `System.Tags` during the Step 2 batch update. Tags power the progress dashboard.

| Story Category | Tag |
|---|---|
| Foundation + CRUD (-01) | `eam-phase-ui-crud` |
| Business Rules (-02) | `eam-phase-validations` |
| Notes (-03), Audit History (-04), PPN (-05), Special (-06+) | `eam-phase-special` |
| QA (last numbered story) | `eam-phase-qa` |
| Security satellite | `eam-phase-security` |

Multiple tags separated by semicolons — syntax example: `"eam-phase-tag-one; eam"`

> **Never add tags beyond what the table above specifies for that story category.** Do not add "eam", "EAM", or any other tag not listed in this table. The only valid tags on EAM stories are the `eam-phase-*` values in the Tag column above.
---

## Personas Reference

Use exactly one of these three personas per story. No others.

| Persona | When to Use |
|---|---|
| `revenue accountant` | Every numbered story (-01 through the last numbered story before QA) |
| `QA analyst` | QA/Parity story (last numbered story) only |
| `system administrator` | Security satellite story only |

Do NOT use: tax analyst, land analyst, royalty reporting analyst, developer.

---

## Naming Conventions

**Xref:** For screens whose name includes "Cross-Reference", use **Xref** throughout all story content (title, Description, ACs). Capital X as a proper noun or screen name (e.g., "Account/Account Group Xref screen"); lowercase when used as a common noun (e.g., "xref records", "an xref record").

---

## Description Template

Every EAM story Description MUST use this exact HTML structure. Start with `<h2>User Story</h2>` — no preamble.

```html
<h2>User Story</h2>
<p>As a <strong>[role]</strong>, I want [capability] so that [business value].</p>

<h2>Description</h2>
<p>[2–3 sentences of business context. Reference the screen, the user workflow step this story
addresses, and the business purpose. State WHY this matters — not HOW it works technically.]</p>
<p><strong>Legacy reference:</strong> [Screen name in Classic GUI — e.g., "MI Link Maintenance (VL025)"]</p>

<h3>Functional Scope</h3>
<ul>
  <li>[User-facing behavior this story delivers]</li>
  <li>[Another user-facing behavior]</li>
</ul>

<h3>Out of Scope</h3>
<ul>
  <li>[What this story does NOT cover — always include cross-story reference: "(covered in [ScreenID]-[Seq])"]</li>
</ul>
```

### Description Rules

- Description ends after `</ul>` closing the Out of Scope section. No trailing sections.
- Do NOT include `<h2>Story Points: [N]</h2>` — SP goes only in the dedicated ADO fields.
- Do NOT include a Business Rules section in the Description unless the story is explicitly a business rules validation story (-02 or a BR-focused special story).
- Do NOT include security content (roles, permissions, access restrictions) in any numbered story. All numbered stories assume the user has complete access.
- No class names, table names, stored procedures, method names, ViewModel properties, or framework patterns anywhere in the Description.

---

## AC Template

ACs are written in Given/When/Then (Gherkin) format and stored in the `Microsoft.VSTS.Common.AcceptanceCriteria` field — **never in Description**.

### Standard AC (custom — no wiki reference)

```html
<p><strong>AC[N] — [Short label]:</strong></p>
<ul>
  <li>Given [precondition — the specific starting state for this screen]</li>
  <li>When [action — what the user does]</li>
  <li>Then
    <ul>
      <li>[Specific observable outcome 1]</li>
      <li>[Specific observable outcome 2 — if applicable]</li>
    </ul>
  </li>
</ul>
```

### Wiki-Referenced AC (GWT block first, Reference line after)

```html
<p><strong>AC[N] — [Short label]:</strong></p>
<ul>
  <li>Given [precondition specific to this screen]</li>
  <li>When [action]</li>
  <li>Then
    <ul>
      <li>[Expected outcome with specific, testable detail]</li>
    </ul>
  </li>
</ul>
<p>Reference: See <a href="https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&amp;pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&amp;pageId=11638&amp;anchor={SECTION-ANCHOR}">Section N — Section Name</a> in the EAM Functionality Reference Wiki.</p>
```

> **Rule:** The wiki Reference line is ALWAYS appended AFTER a full GWT block written for this specific screen. A citation-only line with no GWT block is WRONG. The GWT block is NEVER replaced by a wiki reference.

### AC Rules

- "Then" clauses use nested `<ul>` inside `<li>` **only when there are two or more distinct outcomes**. When a Then clause has exactly one outcome, write it inline — never wrap a single outcome in a nested list (e.g., `<li>Then the record is saved successfully</li>`)
- Every AC must be testable: a QA analyst must be able to write a test from it without asking the dev team
- No ceiling on AC count — write every AC needed to fully specify the story's behavior
- Every validation rule requires BOTH a positive case (valid input accepted) AND a negative case (invalid input blocked) — paired ACs are never separated or dropped
- No wiki references for self-explanatory ACs (navigation, QA parity, user ID/timestamp)
- No class names, table names, stored procedures, method names, internal flags anywhere in ACs

### Testability Test

Before finalizing any AC, verify:
- "Given" has an explicit, specific precondition (not "Given a record exists")
- "When" has a clear action (not "When I use the screen")
- "Then" has a specific, observable outcome (not "the record is saved")

**Not testable:** "Then the record is saved"
**Testable:** "Then the record is created and the results grid refreshes showing the new record with all values intact"

**Not testable:** "Then validation fires"
**Testable:** "Then an error message states 'Remitter must be approved' and the save is blocked until a valid remitter is selected"

---

## PO Voice — Hard Rules

Stories define **WHAT** users need and **WHY**. Never include **HOW**.

### Anti-Pattern Quick Reference

| ❌ Never Include | ✅ Replace With | Category |
|---|---|---|
| `QUIControllerVLManualInputHTD` | "The screen loads and renders" | Class name |
| `RONL_MANL_INPUT table` | "header records" | Table name |
| `SELECT_RONL_MANL_INPUT_VL stored procedure` | "query records by filter criteria" | Stored procedure |
| `VLManualInputHTDRowChanged event fires` | "the child grid refreshes" | Event/method |
| `ManualInputSeqNo field is locked` | "Sequence Number is read-only after save" | Column name |
| `MI_FLAG_LOCK is set` | "the record is locked for final approval" | Internal flag |
| `RELAX_UPDATE_PERMISSIONS business rule` | "the system allows record creation during effective date management" | Internal rule name |
| "As a developer..." | "As a revenue accountant..." | Wrong persona |
| "solution compiles with zero errors" | "screen loads without errors" | Technical outcome |
| `RonlManlInputVM.HeaderData` | "The form displays header information" | ViewModel property |
| "MI_FLAG_LOCK is set AND ApprovedFlag = Y" | "The record is approved and locked for final processing" | DB state |
| "Effective date ranges must not overlap per RXRF_MI_SOD" | "For the same contract and DRI combination, effective date ranges cannot overlap" | Technical constraint |

### Business Rule Phrasing

State rules as **business constraints a revenue accountant could validate**:

| ❌ Technical | ✅ Business |
|---|---|
| "Check ApprovedFlag = Y before insert" | "Only create records when the parent contract is approved" |
| "UpdateDate > CreateDate AND < GETDATE()" | "Effective dates must be in logical order and cannot be in the future" |
| "Cascade delete child rows from TPOLN_ table" | "When a master link is deleted, all associated child records are also removed" |
| "`UpdateUser` and `UpdateDate` are populated" | "The system records who made the change and when" |

---

## Two-Step ADO Creation Pattern

### Step 1: `wit_add_child_work_items`

Creates story with title + description linked to parent Feature. **Do NOT set Iteration Path.**

Parent Feature routing:
- Numbered stories (-01 through -QA) → screen's **functional Feature** (e.g., 1786972)
- Security story (`-Security`) → Feature **1786977**

Record the returned ID for each story.

> **⚠️ CRITICAL — `System.Parent` is NOT a settable field.** Never pass `System.Parent` as a field in `wit_create_work_item` — ADO silently accepts and ignores it, resulting in stories with no parent link. Parent links must be set via `wit_add_child_work_items` (preferred) or via an explicit `wit_work_items_link` call with `type: "parent"` after creation.
>
> **Fallback if `wit_add_child_work_items` times out:** Use `wit_create_work_item` per story instead, then immediately call `wit_work_items_link` with `type: "parent"` and `linkToId` set to the correct Feature ID for each story. Do NOT proceed to Step 2 until every story has a confirmed parent link.

### Step 2: `wit_update_work_items_batch`

Updates all fields not set in Step 1:
- `Microsoft.VSTS.Common.AcceptanceCriteria` — AC HTML
- `System.AssignedTo` — **always** `'Asher Maddox <asher.maddox@quorumsoftware.com>'`, regardless of who is running the session
- `System.Tags` — correct `eam-phase-*` tag
- `System.AreaPath` — value fetched from live sibling story (not memorized)

> **Do NOT set SP fields.** Do not pass `Custom.SAFeStoryPoints` or `Microsoft.VSTS.Scheduling.StoryPoints`. Story point estimation is the team's responsibility during sprint planning.

### Step 3: Successor Links (Numbered Stories Only)

After all numbered stories are created and IDs are known, chain them via ADO Successor relationship:

```
For each consecutive pair (storyA_ID, storyB_ID) in the numbered sequence:
  wit_work_items_link(
    ids: [storyA_ID],
    linkToId: storyB_ID,
    type: "Successor"
  )
```

Chain runs from -01 (first) to -QA (last). Security satellite stories are **excluded** — they are independent of the build sequence.

### Step 4: Verify

Confirm via `wit_get_work_item` with `expand: relations` for each story:
- Parent Feature link correct — verify `System.LinkTypes.Hierarchy-Reverse` (name: "Parent") relation exists pointing to the correct Feature ID. A story with only Successor/Predecessor relations and no Parent relation has a missing parent link — fix before closing out.
- `System.Tags` set correctly
- `System.AssignedTo` is `Asher Maddox <asher.maddox@quorumsoftware.com>`
- Successor chain intact on numbered stories
- Log all ADO IDs

---

## Wiki Base URL (for AC reference lines)

```
https://dev.azure.com/QuorumSoftware/Quorum/_wiki/wikis/Quorum.wiki/11638/?wikiVersion=GBwikiMaster&pagePath=/Quorum/North%20America/Upstream/myQ%20Modernization/EAM%252DScreen%252DMigration%252DFunctionality%252DReference&pageId=11638&anchor={SECTION-ANCHOR}
```

For the full anchor reference table (§1–§25), see `eam-ac-library.md`.
