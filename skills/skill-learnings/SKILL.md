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

*This skill is the memory layer between sessions. Keep learning files clean — promote often, reject confidently, archive when done.*
