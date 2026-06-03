# EAM v2 Project — Setup Guide

Upload all files in this folder to the new Claude.ai project's knowledge base, then paste the system prompt below into the project instructions.

---

## System Prompt (paste into Project Instructions)

```
You are working with Aaron Alejo and Asher Maddox, Business Analyst and Product Owner on the EAM (Enterprise Accounting Modernization) project at Quorum Software — migrating QRA and QDO desktop screens to a modern web application.

At the start of every session, load `eam-project-instructions.md` for team, ADO, and domain context.

For story authoring, follow the File Loading Guide in `eam-project-instructions.md` to load the right file at the right phase. The minimum for any story work is `eam-story-core.md` — load it before writing any HTML.

Never create ADO work items without explicit approval. Never set story points — sizing is the team's responsibility.
```

---

## File Inventory — 23 Files + This README

### Core Workflow (load per session)

| File | When to Load |
|---|---|
| `eam-project-instructions.md` | Every session — team, ADO defaults, domain vocab, file loading guide |
| `eam-workflow.md` | When you need the 5-phase process sequence or approval gate language |
| `eam-story-core.md` | Before writing any story HTML — Description template, AC template, validation gate, ADO creation |
| `eam-decomposition.md` | When planning a new story map — decomposition, splitting, estimation, risk |
| `eam-ac-library.md` | When writing ACs — wiki anchors §1–§25, -01 mandatory AC templates, -02 validation pairs, grid defaults AC |

### Operational Skills (load per task)

| File | When to Load |
|---|---|
| `ado-connection.md` | Before any ADO write operation |
| `upstream-codebase-guide.md` | Before codebase search (Phase 1 analysis) |
| `batch-process-qa-review.md` | For Epic 5 batch process QA reviews |
| `skill-learnings.md` | At session end when a new pattern was discovered |
| `safe-popm-best-practices.md` | For SAFe work items, WSJF, PI planning, Feature creation |

### Domain Knowledge (used by Claude during analysis)

| File | Content |
|---|---|
| `project-context.md` | EAM project background, petroleum accounting context |
| `screen-inventory.md` | Screen list with migration patterns and Feature IDs |
| `dri-module.md` | DRI module — business rules and screen detail |
| `qra-modules.md` | QRA modules — VL, JE, CW, VA, MF, DO, TS |
| `qdo-qca-qcfs.md` | QDO, QCA, QCFS screen and domain context |
| `batch-and-qpec.md` | Batch process and QPEC reference |
| `shared-infrastructure.md` | Shared web platform context |
| `upstream-codebase-reference.md` | Deep technical reference — repos, module interconnections, table structures |
| `Effort-Priority-Matrix.md` | POC gap analysis — use Migration Pattern, BR count, Feature ID columns only |
| `DRI_Footprint_Screens.xlsx` | Screen inventory with design approach and priority |
| `eam-user-story-template-learnings.md` | Accumulated session learnings (L001+) — applied as overlays each session |

### SAFe Reference

| File | Content |
|---|---|
| `safe-templates.md` | Epic, Feature, Story templates in SAFe format |
| `project_personas.md` | Role guide — PM, PO, BA, Engineering Lead interaction styles |

---

## What Was Retired (do not upload)

These files from the old project are fully replaced and should not be added:

| Old File | Replaced By |
|---|---|
| `CLAUDE_INSTRUCTIONS_EAM_WORKFLOW-v9_0.md` | `eam-workflow.md` |
| `EAM-Project-Claude-Instructions.md` | `eam-project-instructions.md` |
| `eam-user-story-template-SKILL-v6_1.md` | `eam-story-core.md` + `eam-decomposition.md` + `eam-ac-library.md` |
| `EAM-Story-Authoring-Quick-Reference-v6_1.md` | Absorbed into core files |
| `SKILL.md` (safe-popm-best-practices) | `safe-popm-best-practices.md` |

---

## Verify After Upload

Once all files are uploaded and the system prompt is saved, open a new conversation in the project and run:

> "Load eam-project-instructions.md and confirm you can see the File Loading Guide."

If Claude returns the table from Section 7 of that file, the project knowledge is wired correctly.
