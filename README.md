# Upstream EAM AI Workflow

Source of truth for all Claude AI workflow files supporting the **EAM (Enterprise Accounting Modernization)** project at Quorum Software.

This repo stores the Claude project knowledge files and skill files that power AI-assisted user story authoring, codebase analysis, and SAFe backlog management for the QRA/QDO desktop-to-web screen migration.

---

## Repository Structure

```
├── project-files/          Claude project knowledge — uploaded to the Claude.ai EAM Project
└── skills/                 Claude skill files — installed in the Claude skill system
```

### project-files/

| File | Purpose |
|---|---|
| `eam-project-instructions.md` | **Load every session.** Team context, ADO defaults, domain vocab, file loading guide |
| `eam-workflow.md` | 5-phase story authoring process + approval gate language |
| `eam-story-core.md` | Description/AC HTML templates, ADO creation pattern, validation gate |
| `eam-ac-library.md` | Full AC library §1–§26+, -01 mandatory AC templates, grid defaults |
| `eam-decomposition.md` | Story map planning, splitting rules, estimation, risk |
| `eam-user-story-template-learnings.md` | Accumulated session learnings applied as overlays |
| `project-context.md` | EAM project background + petroleum accounting context |
| `project_personas.md` | Team role guide — PM, PO, BA, Engineering Lead interaction styles |
| `screen-inventory.md` | Screen list with migration patterns and Feature IDs |
| `Effort-Priority-Matrix.md` | **Authoritative** screen technical profiles — BR counts, effort tier, complexity signals |
| `DRI_Footprint_Screens.xlsx` | Screen inventory with design approach and priority |
| `dri-module.md` | DRI module business rules and screen detail |
| `qra-modules.md` | QRA module reference (VL, JE, CW, VA, MF, DO, TS) |
| `qdo-qca-qcfs.md` | QDO, QCA, QCFS screen and domain context |
| `batch-and-qpec.md` | Batch process and QPEC reference |
| `shared-infrastructure.md` | Shared web platform context |
| `upstream-codebase-guide.md` | Codebase navigation — repos, modules, search patterns |
| `upstream-codebase-reference.md` | Deep technical reference — tables, interconnections, patterns |
| `batch-process-qa-review.md` | Epic 5 batch process QA review workflow |
| `ado-connection.md` | Azure DevOps auth bootstrap |
| `skill-learnings.md` | Learning capture and promotion system |
| `safe-popm-best-practices.md` | SAFe PO/PM best practices |
| `safe-templates.md` | SAFe Epic, Feature, Story templates |
| `SETUP.md` | Claude project setup guide — file inventory and upload instructions |

### skills/

| Skill | Purpose |
|---|---|
| `ado-connection/` | ADO connection and auth bootstrap |
| `batch-process-qa-review/` | Epic 5 batch QA review and analysis |
| `eam-user-story-template/` | EAM story authoring (superseded by project files — kept for reference) |
| `release-notes-export/` | Release notes generation from ADO |
| `safe-popm-best-practices/` | SAFe work item creation guidance |
| `skill-learnings/` | Continuous learning system + `learnings/` captured entries |
| `triage-low-medium-bugs/` | Bug triage workflow |
| `upstream-codebase-guide/` | Upstream codebase navigation skill |

---

## How to Use This Repo

### Setting Up a New Claude Project

1. Create a new project in Claude.ai
2. Download all files from `project-files/`
3. Upload them to the project's knowledge base
4. Paste the system prompt from `SETUP.md` into the project instructions
5. Verify: open a conversation and run `"Load eam-project-instructions.md and confirm you can see the File Loading Guide"`

### Updating Files

When a project file changes:
1. Update the file here in GitHub (this is the source of truth)
2. Download the updated file
3. In the Claude project, delete the old version and upload the new one

> Claude's project knowledge does **not** auto-sync — files must be manually re-uploaded after any change in this repo.

### Installing Skills

Skill files in `skills/` are installed via the Claude skill system (Claude Desktop). Each folder contains a `SKILL.md` that defines the skill's behavior.

---

## Team

| Role | Name |
|---|---|
| Business Analyst | Aaron Alejo |
| Product Owner | Asher Maddox |
| Product Manager | Sarah Paul |
| Engineering Lead | Jimmy Bidwell |
| Developers | Kelsey Pryor, Prasanna Joshi, Connor Morris |

**ADO Organization:** `QuorumSoftware` | **ADO Project (stories):** `Quorum`

---

## Active Feature IDs

| Feature | ID |
|---|---|
| DRI screen stories | 1786972 |
| JE screen stories | 1786978 |
| Security satellite (all screens) | 1786977 |
| API satellite (all screens) | 1806481 |
