---
name: skill-learnings
description: "Continuous learning system for capturing, storing, and promoting feedback-driven improvements across Claude skills. Use this skill whenever: (1) a user provides feedback that corrects or refines how Claude previously followed a skill (e.g., 'that story needs revision', 'merge those ACs', 'the QA story is missing X'); (2) a session produces new patterns or standards not yet in a skill; (3) the user says 'remember this for next time', 'add this to the template', 'update the skill', or similar; (4) Claude performs a self-review and identifies a gap in an existing skill; (5) at the end of any EAM story authoring session where revisions were made. Also triggers on: 'capture this learning', 'save this pattern', 'update the EAM template', 'what have you learned', 'apply past learnings'."
---

# Skill Learnings — Continuous Improvement System

> **Purpose:** Capture session-level learnings and feedback corrections, then promote them into the canonical skills they belong to.
> **Learning Store:** `learnings/` directory — one file per target skill
> **Last Merge:** See individual learning files for merge history

---

## How This Works

```
Session feedback → Capture Learning → learnings/{skill-name}.md
                                              ↓ (when promoted)
                                      Target SKILL.md updated
```

1. **Capture** — During or at end of a session, learnings are appended to `learnings/{skill-name}.md`
2. **Review** — Learnings accumulate with status: `[PENDING]`, `[PROMOTED]`, or `[REJECTED]`
3. **Promote** — When triggered, pending learnings are merged into the target skill and marked `[PROMOTED]`
4. **Prune** — After promotion, promoted entries can be archived to keep the file clean

---

## Step-by-Step Procedures

### Procedure A — Capture a Learning

Triggered when: user provides corrective feedback, a revision is made to a generated artifact, or a new pattern is identified.

1. Read `learnings/{target-skill-name}.md` (create if it doesn't exist using the template below)
2. Identify the **learning type** (see taxonomy)
3. Write a concise, actionable learning entry (see format below)
4. Append to the appropriate section in the learning file
5. Confirm to the user: *"Captured as a learning for `{skill-name}`. It will be applied next session and can be promoted into the skill when you're ready."*

**When to auto-capture without asking:**
- User explicitly edits or rejects something Claude generated per a skill
- User says the output needed revision after Claude marked it complete
- A generated work item was updated before ADO creation

**When to ask first:**
- A pattern only appeared once and may be context-specific
- The learning would conflict with an existing skill rule

---

### Procedure B — Apply Learnings at Session Start

When a skill is invoked that has a corresponding learning file:

1. Check `learnings/{skill-name}.md` for `[PENDING]` entries
2. Silently apply them as overlays on top of the base skill instructions
3. If a pending learning conflicts with a base skill rule, surface the conflict to the user

> **Note:** Learnings are additive by default. They extend, not replace, base skill behavior unless explicitly marked `[OVERRIDE]`.

---

### Procedure C — Promote Learnings into a Skill

Triggered when: user says "promote learnings", "update the skill", or when 5+ PENDING learnings exist for a skill.

1. Read `learnings/{skill-name}.md` — collect all `[PENDING]` entries
2. Read the target `SKILL.md`
3. **If the skill version has incremented since the learnings were captured**, review each PENDING entry against the new version — learnings already absorbed by the new standard should be marked `[PROMOTED]` (with note: "absorbed by vX.X") or `[REJECTED]` before proceeding
4. For each remaining pending learning, determine the best insertion point in the target skill
5. Present the proposed changes to the user as a diff/summary **before** writing
6. After user approval, copy the target skill to `/tmp/`, apply edits, and package as a `.skill` file for the user to install
7. Mark promoted entries as `[PROMOTED]` in the learning file with date
8. Update the skill version number and "Last Updated" date in the target SKILL.md

---

### Procedure D — Session-End Prompt

At the end of any session where a skill was invoked AND revisions were made to the output:

Surface this prompt:
> *"We made [N] revision(s) to the generated output this session. Should I capture any of these as learnings for the `{skill-name}` skill so they're applied automatically next time?"*

If yes, execute Procedure A for each revision.

---

## Learning Entry Format

```markdown
### L{NNN} — {YYYY-MM-DD} — [{STATUS}]
**Type:** {learning-type}
**Context:** {brief description of the situation that triggered this learning}
**Learning:** {the actionable rule or pattern, written as an instruction to Claude}
**Example:** {optional — show before/after or a concrete example}
**Source:** {Session summary | User feedback | Self-review}
**Applies to:** {Section N of target skill, or "General"}
```

**Status values:**
- `[PENDING]` — Captured, not yet in the skill. Will be applied as overlay each session.
- `[PROMOTED]` — Merged into the target SKILL.md. Kept for audit trail.
- `[REJECTED]` — Decided not to promote (too context-specific, superseded, etc.)

---

## Learning Type Taxonomy

| Type | Description | Examples |
|------|-------------|---------|
| `CONSOLIDATION` | When to merge stories | "Merge notes + nav into CRUD when they share row-selection context" |
| `AC_QUALITY` | Acceptance criteria standards | "QA story must list screen-specific functional checks, not generic patterns" |
| `VALIDATION_PAIR` | Paired positive/negative validation ACs | "Every validation rule must have both a positive case (valid input → save succeeds) AND a negative case (invalid input → save blocked) as mandatory paired ACs; omitting either half is a template violation" |
| `ESTIMATION` | SP calibration corrections | "Mass Changes dialogs are consistently 8 SP, not 5" |
| `NAMING` | Title/tag conventions | "Use 'CRUD' not 'Create/Update/Delete' in story titles" |
| `DECOMPOSITION` | Story splitting/sequencing | "Foundation+Query always merged into -01 (8 SP); CRUD+BRs merged when screen BR count ≤5 per Effort-Priority Matrix; Audit History always standalone at 3 SP" |
| `TEMPLATE` | Description or AC template changes | "Add 'Legacy screen reference' line to Description for all Redesign stories" |
| `WORKFLOW` | Generation process improvements | "Always check for PPN trigger before drafting the Audit Trail story" |
| `CONSTRAINT` | New rules or guardrails | "Never exceed 8 SP on a single story without flagging a spike" |
| `PERSONA` | New or updated user personas | "Add 'Royalty Reporting Analyst' as a persona for TR010 screens" |
| `PATTERN` | Reusable screen patterns | "VL032-style file upload pattern: always flag as 🔴 High risk" |

---

## Learning File Template

When creating a new learning file for a skill, use this template:

```markdown
# Learnings: {skill-name}
> Target skill: `/mnt/skills/user/{skill-name}/SKILL.md`
> Created: {YYYY-MM-DD}
> Last promoted: {YYYY-MM-DD or "Never"}

## Pending Learnings
*(Applied as overlays each session until promoted)*

## Promoted Learnings
*(Already in the skill — kept for audit trail)*

## Rejected Learnings
*(Decided not to promote — reason noted)*
```

---

## Current Learning Files

| Skill | File | Pending | Promoted | Last Activity |
|-------|------|---------|----------|---------------|
| eam-user-story-template | `learnings/eam-user-story-template.md` | See file | See file | — |
| safe-popm-best-practices | `learnings/safe-popm-best-practices.md` | See file | See file | — |

*(Update this table when new learning files are created)*

---

## Promotion Decision Guide

When deciding whether a learning should be promoted into the skill vs. staying as an overlay:

**Promote when:**
- The pattern has appeared in 2+ sessions
- The learning changes default behavior (not just a one-off exception)
- The user explicitly says "add this to the template permanently"
- The learning fills a gap in the skill (something not addressed at all)

**Keep as overlay when:**
- The pattern is project/screen-specific
- It's only appeared once and may be an edge case
- It modifies a rule rather than adding a new one (needs more validation)

**Reject when:**
- It contradicts SAFe principles without good reason
- It was a one-time user preference, not a durable standard
- It's superseded by a later learning

---

## Example: Full Learning Lifecycle

**Session:** JE010 decomposition review. Feedback: "The CRUD and Business Rules stories should be merged — there are only 3 BRs. The BR count from the Effort-Priority Matrix is what drives the split, not a judgment call."

**Captured:**
```markdown
### L001 — 2026-05-06 — [PENDING]
**Type:** DECOMPOSITION
**Context:** JE010 initial story map had CRUD and Business Rules as separate stories despite only 3 BRs
**Learning:** When BR count ≤5 (verified against Effort-Priority Matrix), CRUD and Business Rules
are always merged into a single story per the v3 Track A standard. This is not a per-screen
judgment call — it is a fixed decision rule. The merged story is titled
"[Entity] Record CRUD & Business Rule Validations" at 8 SP.
**Example:** JE010 original Story 02 (CRUD, 5 SP) + Story 03 (BRs, 3 SP) → merged into
Story 02 "Account Record CRUD & Business Rule Validations" at 8 SP
**Source:** User feedback — JE010 decomposition session
**Applies to:** Section 3 (Track A, Step 2) of eam-user-story-template
```

**Applied next session:** Before generating JE015 stories, Claude reads this learning, checks the Effort-Priority Matrix for BR count, and pre-applies the merge decision rather than defaulting to separate stories.

**Promoted:** After the v3.2 template update formalizes this rule, the entry is marked `[PROMOTED]` with note "absorbed by v3.2 Track A standard" — no manual insertion into the skill needed.

---
## L001 — `wit_work_items_link` with `type: parent` fails with TF201036 when item already has a parent

**Context:** EAM re-parenting session, May 2026.

**Error observed:** `TF201036: You cannot add a Parent link between work items X and Y because a work item can have only one Parent link.`

**Cause:** The item already has a `System.LinkTypes.Hierarchy-Reverse` (Parent) relation. The MCP tool does not accept duplicate parent links regardless of whether the target parent is different.

**Resolution:** Items must be manually unparented in the ADO UI first (open the work item → click the existing parent link → remove it). After that, `wit_work_items_link` with `type: parent` succeeds immediately.

---

## L002 — `wit_update_work_item` field patches on `System.Parent` are silently ignored

**Context:** EAM re-parenting session, May 2026.

**Observed behavior:** Calling `wit_update_work_item` with `op: replace` or `op: add` on `/fields/System.Parent` returns HTTP 200 with no error, but `System.Parent` in the response still shows the old parent ID. The hierarchy relation is not changed.

**Root cause:** `System.Parent` is a computed/read-only field in the REST API — it reflects the current `Hierarchy-Reverse` relation but cannot be set directly through field patches. The API silently accepts and discards the patch.

**Do NOT use this approach for re-parenting.** Use `wit_work_items_link` with `type: parent` after manual UI unparenting instead (see L001).

---

## L003 — `wit_update_work_item` remove operations on relations fail due to MCP non-null enforcement

**Context:** EAM re-parenting session, May 2026.

**Observed behavior:** Attempting to use `wit_update_work_item` with `op: remove` on a relation path (e.g., to remove the existing `Hierarchy-Reverse` link programmatically) fails. The MCP enforces non-null on relation operations and blocks the call.

**Consequence:** There is no MCP-supported path to programmatically remove an existing parent link. Manual UI unparenting is the only reliable method before re-parenting via `wit_work_items_link`.

---

## L004 — After manual UI unparenting, `wit_work_items_link` with `type: parent` works reliably

**Context:** EAM re-parenting session, May 2026. Confirmed across 97 work items over two sessions.

**Pattern confirmed:** After the user manually removes the parent link in the ADO UI, a single call to `wit_work_items_link` with `type: parent`, `id` = child, `linkToId` = new parent succeeds atomically. The response body shows the correct new `System.Parent` value and a `System.LinkTypes.Hierarchy-Reverse` relation pointing to the new parent.

**Verify success by:** checking `System.Parent` in the response fields — it will show the new parent ID if the link was set correctly.

---

## L005 — Successor/predecessor chains should be audited via spot-checking midpoint stories, not full per-story fetches

**Context:** EAM dependency chain audit, May 2026 — 10 screens, ~97 stories.

**Pattern confirmed:** To determine whether a dependency chain is wired for a screen, fetch one midpoint story with `expand: relations` and check for `System.LinkTypes.Dependency-Reverse` (Predecessor) and `System.LinkTypes.Dependency-Forward` (Successor) entries. If the midpoint story has no dependency links, the entire screen's chain is unwired. Fetching every story individually is unnecessary and expensive in tool calls.

**Efficient audit workflow:**
1. Spot-check one mid-sequence story per screen (e.g., story 03 of a 06-story screen).
2. If no dependency links are found → wire the full chain for that screen.
3. If dependency links are present → optionally spot-check the head and tail to confirm completeness.

**Note:** VL031 had a complete chain already wired from story creation; this was confirmed by checking story -02 (which showed both predecessor to -01 and successor to -03) and story -08 (which showed the chain ran through to -09).

---

## L006 — Predecessor links auto-create the reciprocal successor; only one call per pair is needed

**Context:** EAM dependency chain wiring, May 2026.

**Pattern confirmed:** When `wit_work_items_link` is called with `type: predecessor`, `id` = later story, `linkToId` = earlier story, ADO automatically creates the reciprocal `System.LinkTypes.Dependency-Forward` (Successor) link on the earlier story. There is no need to make a second call to wire the successor direction.

**Efficient wiring pattern:** For a chain 01→02→03→04, make three calls:
- `{id: 02, linkToId: 01, type: "predecessor"}`
- `{id: 03, linkToId: 02, type: "predecessor"}`
- `{id: 04, linkToId: 03, type: "predecessor"}`

Each call wires both directions simultaneously.

---

## L007 — `wit_work_items_link` with `type: parent` must be sent one item at a time; batching hierarchy links produces inconsistent results

**Context:** EAM re-parenting session, May 2026.

**Observed behavior:** The `updates` array on `wit_work_items_link` supports multiple entries, but batching parent-type links in a single call produced inconsistent results during the re-parenting session. Some links in a batch would succeed while others silently failed, with no clear error distinguishing which items were affected.

**Recommended practice:** Send one `{id, linkToId, type: "parent"}` object per call for hierarchy links. Predecessor/successor links batched in a single call behaved reliably and can be batched if needed, though single-pair calls are safer for traceability.

---

## L008 — Non-existent work item IDs return "Not Found" from `wit_work_items_link`, not a soft failure

**Context:** EAM re-parenting session, May 2026. Item 1809403 (expected JE030 story) did not exist.

**Observed behavior:** When `wit_work_items_link` is called with an `id` or `linkToId` that does not exist in ADO, the call returns `Error linking work items: Failed to update work items in batch: Not Found`. This is a hard error, not a silent skip.

**Diagnostic step:** Confirm item existence with `wit_get_work_item` before calling link tools when item IDs come from a list that may contain gaps (deleted, never-created, or retired items). A null response from `wit_get_work_item` confirms the item does not exist.

---

*This skill is the memory layer between sessions. Keep learning files clean — promote often, reject confidently, archive when done.*