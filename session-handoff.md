# EAM Workflow Restructuring — Session Handoff

**Source session:** June 3, 2026 (Aaron Alejo + Claude, EAM Claude Desktop project)
**Purpose:** Full context transfer for the new EAM project — captures all decisions, analysis, and next steps so a new Claude session can continue without losing ground.

\---

## Who You're Working With

* **Aaron Alejo** — Business Analyst, EAM project at Quorum Software
* **Asher Maddox** — Product Owner, co-manages the EAM project and Claude workspace
* Both are on the Products side of the myQ Accounting Modernization (EAM) project — migrating QRA/QDO classic desktop screens to a modern Azure-based web application

\---

## The Three Goals

### Goal 1 — Version-Controlled Project Migration (DECIDED)

**Problem:** Today, Aaron and Asher manage \~24 project knowledge files and \~9 skill files through a manual cycle: edit locally → delete old version from Claude project → re-upload new version. This is error-prone, creates sync gaps, and scales badly.

**Decision:** GitHub is the **single source of truth**. All project files and skills live in a GitHub repo. The Claude project pulls files from GitHub via MCP rather than relying on static uploads.

**Why this works:** The EAM project is above the RAG retrieval threshold (\~450KB of project files + skill content). Above this threshold, Claude's project knowledge already does selective chunk retrieval internally — the same as a tool call. The token overhead difference between project knowledge retrieval and GitHub MCP fetch is \~200-500 tokens per call, which is marginal at this scale.

**Architecture:**

* Project instructions (system prompt) live in the Claude project itself — this is the bootstrap
* Everything else lives in GitHub and is fetched on demand
* No more manual delete/re-upload cycle
* Version history, diffs, and rollback come free with git

**GitHub repo:** `https://github.com/AAlejo-Q/Upstream-EAM-AIWorkflow`

### Goal 2 — Cross-Functional Skill Alignment (Products ↔ Dev)

**Problem:** Products (Aaron/Asher) and Dev operate with separate Claude instances that have different skills, different analysis methods, and different mental models. Stories get authored with one set of assumptions, then Dev's Claude reads them through a different lens — finding missed validations, misinterpreted scope, and functionalities that fell through the cracks.

**Status:** Full analysis of the dev team's workflow has been completed. Specific alignment gaps identified (see "Gap Analysis" section below). Recommendations drafted but not yet implemented.

### Goal 3 — Deeper Analysis to Reduce Gaps and Assumptions

**Problem:** The current Products Phase 1 analysis catches a lot but things still slip through — especially screen-specific behaviors, edge-case validations, and code paths that aren't obvious from the main form class. The dev team's analysis skill (`Analyze-classic-gui-screen`) is significantly more thorough, querying 7+ metadata tables and dispatching 3 parallel sub-agents.

**Status:** Specific enhancement areas identified from dev team's analysis skill. Recommendations drafted but not yet implemented.

\---

## What's Been Done

### 1\. GitHub Repo Created and Populated

**Repo:** `https://github.com/AAlejo-Q/Upstream-EAM-AIWorkflow` (public)

**Structure:**

```
Upstream-EAM-AIWorkflow/
├── README.md                          ← Repo overview + setup guide
├── project-files/                     ← All Claude project knowledge files (24 files)
│   ├── eam-project-instructions.md
│   ├── eam-workflow.md
│   ├── eam-story-core.md
│   ├── eam-ac-library.md
│   ├── eam-decomposition.md
│   ├── eam-user-story-template-learnings.md
│   ├── project-context.md
│   ├── project\_personas.md
│   ├── screen-inventory.md -- outdated, need to use screen-status
│   ├── DRI\_Footprint\_Screens.xlsx
│   ├── dri-module.md
│   ├── qra-modules.md
│   ├── qdo-qca-qcfs.md
│   ├── batch-and-qpec.md
│   ├── shared-infrastructure.md
│   ├── upstream-codebase-guide.md
│   ├── upstream-codebase-reference.md
│   ├── batch-process-qa-review.md
│   ├── ado-connection.md
│   ├── skill-learnings.md
│   ├── safe-popm-best-practices.md
│   ├── safe-templates.md
│   └── SETUP.md
└── skills/                            ← All Claude skill files (9 files)
    ├── ado-connection/SKILL.md
    ├── batch-process-qa-review/SKILL.md
    ├── eam-user-story-template/SKILL.md
    ├── release-notes-export/SKILL.md
    ├── safe-popm-best-practices/SKILL.md
    ├── skill-learnings/
    │   ├── SKILL.md
    │   └── learnings/eam-user-story-template.md
    ├── triage-low-medium-bugs/SKILL.md
    └── upstream-codebase-guide/SKILL.md
```

### 2\. GitHub MCP Configured in Claude Desktop

Aaron and Asher's `claude\_desktop\_config.json` have been updated to include a GitHub MCP server entry using `@modelcontextprotocol/server-github` using a PAT for the Quorum-Enterprise org repo.

Aaron's Config file location: `%APPDATA%\\Claude\\claude\_desktop\_config.json`

Asher's Config file location: `C:\\Users\\asher.maddox\\AppData\\Roaming\\Claude\\claude\_desktop\_config.json`

### 3\. Dev Team's Workflow Fully Analyzed

The dev team's repo (`Quorum-Enterprise/Quorum.Upstream.AITools`) and all ADO wiki documentation have been read and analyzed. See the full analysis below.

\---

## Dev Team's Workflow — Full Analysis

### Architecture

The dev team uses **Claude Code** running out of `Quorum-Enterprise/Quorum.Upstream.AITools`. They have:

* Direct file system access to local source repo clones
* SQL Server MCP (`sqlserver-metadata`) for metadata queries
* Azure DevOps MCP for code search
* Serena MCP server for semantic code analysis
* GitHub CLI (`gh`) for package management and PR creation

### Their Migration Lifecycle (8 Steps)

|Step|Skill|Purpose|Output|
|-|-|-|-|
|1|`Analyze-classic-gui-screen`|Deep screen analysis with 3 parallel sub-agents|5 markdown files (Overview, Technical, SQL, CodeGen, Test Scenarios)|
|2|`Spec-screen`|Generate human-reviewable migration blueprint|`Screen-Spec.md`|
|3|`Analyze-ado-stories`|Gap-analyze Products' ADO stories against spec|`ADO-Story-Analysis.md`|
|4a|`Migrate-foundation-CRUD`|Foundation + CRUD web artifacts (9 phases)|Code in QRA.Web + RemainingWork.md|
|4b|`Migrate-validations`|Validation rules (4-tier classification)|Validation classes + unit tests|
|4c|`Migrate-special`|Batch process actions|UIController methods + JS handlers|
|5|`Audit-screen`|6-category gap analysis against SpecKit standard|Audit report with readiness gate|
|6|`Remediate-screen`|Apply audit fixes (never auto-remediates contradictions)|Code fixes + remediation summary|
|7a-c|`Unit-test-\*` (split)|MSTest suite with ADO story AC traceability|5-6 test files per screen|
|8|`Document-screen`|End-user + developer documentation|HTML spec + Markdown reference|

### Key Skill Details

**Analyze-classic-gui-screen (Step 1)** — the most comprehensive analysis in the pipeline:

* 3 parallel sub-agents: Agent A (source code, 14+ ADO code searches), Agent B (metadata SQL — grid config, columns, code tables, pick lists, SQL keys across 6 query steps), Agent C (CodeGen XML analysis)
* Wave 3 resolves `RES\_MSG\_\*` resource strings to canonical English text (critical for accurate error messages)
* Wave 4 writes all 5 files simultaneously
* Output: `documentation/Analysis/ClassicGUI/{ScreenId}-{Name}/`

**Spec-screen (Step 2)** — the canonical migration blueprint:

* Reads the 5 analysis files from Step 1
* Fills a comprehensive template covering: Database (primary/secondary tables, key columns, audit columns), Data Objects, Service Layer, Screen Layout (filters, grids, toolbar, detail panel), every Business Rule individually, Lookups/Dropdowns, Context Data Dependencies, Special Behaviors, Cross-Screen Navigation, Print/Export with grid config flags, Notes (full DB lookup workflow for GCDE\_CMNT\_TYPE and QARCH\_CODE\_NOTE\_CATEGORY), Unit Test guidance, Open Questions
* Identifies recommended web pattern from base class mapping
* Flags uncertain items as `TODO: verify`
* Auto-invokes `Analyze-ado-stories` if ADO Feature ID is provided
* Output: `documentation/SpecKit/modules/{Module}/{ScreenId}-{Name}-Screen-Spec.md`

**Analyze-ado-stories (Step 3)** — the Products-Dev bridge:

* Fetches Products' ADO stories under a Feature
* Loads the Screen-Spec
* Performs 4-way gap analysis: (1) In stories, not in spec; (2) In spec, no story backing; (3) Contradictions; (4) Open questions
* Report is consumed by Audit-screen, Remediate-screen, and Unit-test-screen
* Output: `documentation/SpecKit/modules/{Module}/{ScreenId}-ADO-Story-Analysis.md`

**Migrate-validations (Step 4b)** — uses a 4-tier classification:

* Tier 1: Pure in-memory (DO property checks) — fully implemented
* Tier 2: DB-lookup validations — fully implemented with batch-lookup pattern
* Tier 3: Commit-time / collection-level — fully implemented
* Tier 4: Side-effects, auto-populate, user dialogs, PPN writes — stubbed with `NotImplementedException`
* No-stubs policy: empty Validate() for Tier 1-3 is explicitly prohibited

**Audit-screen (Step 5)** — has a migration readiness gate:

* 6 categories: File Completeness, Architecture/Pattern, Functionality Coverage, Screen Registration, Special Behaviors, ADO Story Gaps
* Blocks if ADO Story Analysis report contains unresolved contradictions
* Each gap classified as Critical or Minor

### Supporting Skills

* `Update-github-repos` / `Update-ado-repos` — keep source references current
* `Document-code-knowledge` — capture patterns and insights
* `Analyze-github-issue` / `Review-github-pr` — GitHub work item analysis
* `Upstream-expert-review` — multi-agent parallel expert PR review
* `Screen-status-report` — reports for Asher on screen status
* `Sprint-report` — sprint review reports
* `consume-qfc-packages` / `report-qfc-package-status` — QFC package management
* `task-sync` — deprecated (marked for deletion)
* `Review-github-pr` — old PR review (marked for deletion)

### Dev Team Skill Enhancement Efforts (In Progress)

From the dev team's own status notes:

* `Migrate-foundation-CRUD` — needs work; loses context mid-task, needs more gates (User Story 1813786)
* `Migrate-validations` — Connor building TDD approach (User Story 1810286)
* `Migrate-special` — needs assessment of what constitutes "special"
* `Audit-screen` / `Remediate-screen` — Prasanna reviewing combination with `Review-migration-implementation`
* Unit test skills — Connor recently adjusted; revisiting test case alignment with coverage

\---

## Gap Analysis: Products ↔ Dev

### Gap 1: Duplicate Analysis, Different Findings

Our Phase 1 analysis and their `Analyze-classic-gui-screen` both analyze the same screens independently. Their skill is significantly more thorough:

* Queries 7+ metadata tables (grid config, columns, code tables, pick lists, SQL keys, pick list return columns)
* 14+ specific code searches via ADO
* Resolves resource strings to canonical English text
* Checks CodeGen status across 5 layers
* Produces 5 structured documents

Our Phase 1 is comparatively ad-hoc. When both produce different findings, our stories and their spec diverge before implementation starts.

### Gap 2: Screen-Spec vs Stories — Parallel Artifacts

The `Screen-Spec.md` is the dev team's source of truth for implementation. Our ADO stories are a separate artifact with a different structure and different level of detail. When `Analyze-ado-stories` compares them, it's comparing two independently-authored documents.

### Gap 3: One-Directional Feedback Loop

`Analyze-ado-stories` finds gaps and contradictions. That report feeds into the dev pipeline (Audit, Unit Tests). But it never comes back to Products. We never see the gap report, so we never update stories to close the gaps.

### Gap 4: Validation Tier Misalignment

Dev's `Migrate-validations` classifies rules into 4 tiers. Our -02 stories describe validations in user-voice Given/When/Then without this tiering. Dev's Claude has to re-derive the classification from our ACs — sometimes getting it wrong or finding rules we missed.

\---

## Recommended Changes (Not Yet Implemented)

### For Goal 2 (Cross-Functional Alignment):

1. **Consume Dev's analysis output** — When a Screen-Spec or analysis files exist for a screen, our Phase 1 should consume them rather than analyzing from scratch.
2. **Add Phase 1.5 gap check** — After drafting stories, run the equivalent of `Analyze-ado-stories` against the spec. Catches gaps before sprint planning.
3. **Close the feedback loop** — When gap reports exist, trigger story amendments on our side.
4. **Feed -02 ACs into dev pipeline** — Structure our validation ACs so they map cleanly to the dev team's 4-tier classification.

### For Goal 3 (Deeper Analysis):

1. **Metadata table queries** — Query `QARCH\_CNFG\_GRIDCTRL\_NET` and `QARCH\_CNFG\_GRIDCTRL\_COL\_NET` for every screen. We already have `quorum-metadata-dev` MCP for this.
2. **SQL key inventory** — Complete `QARCH\_SQL` query by screen prefix for every SELECT/INSERT/UPDATE/DELETE operation.
3. **Resource string resolution** — Include canonical `RES\_MSG\_\*` text in -02 ACs so dev's Claude has exact expected error messages.
4. **Pick list and code table validation** — Systematic check of every `CDTBL\_ID` and `PKLIST\_ID`.
5. **Notes behavior detection** — Systematic workflow for detecting and documenting Notes (Comments → Notes migration).

\---

## Key References

### Repos

* **Products and Dev team repo:** `https://github.com/Quorum-Enterprise/Quorum.Upstream.AITools` (private, requires Quorum-Enterprise SSO-authorized PAT with `repo` scope)

### ADO Wiki Pages (Dev Team Documentation)

* Upstream AI Skills overview: `https://dev.azure.com/QuorumSoftware/Quorum/\_wiki/wikis/Quorum.wiki/11534/Upstream-AI-Skills`
* Migration Skills: `https://dev.azure.com/QuorumSoftware/Quorum/\_wiki/wikis/Quorum.wiki/12308/Migration-Skills`
* Supporting Skills: `https://dev.azure.com/QuorumSoftware/Quorum/\_wiki/wikis/Quorum.wiki/12310/Supporting-Skills`
* Package Management: `https://dev.azure.com/QuorumSoftware/Quorum/\_wiki/wikis/Quorum.wiki/12311/Package-Management-Tools`

### ADO Organization

* **Code search:** `QuorumSoftware` org
* **Stories/work items:** `Quorum` project

### Active Feature IDs

* 1847639	BA010	BA Mass Update
* 1847640	BA015	BA Address Mass Update
* 1847637	CW005	Check Register Query/View
* 1812407	CW010	State Withholding Tax Maintenance
* 1813155	CW011	State Backup Withholding Tax Maintenance
* 1813156	CW015	Non-Resident Alien Withholding Tax Maintenance
* 1847638	CW020	Void Check Maintenance
* 1813158	EC010	Escheat Processing Rules
* 1813173	GB015	Gas Balancing Manual Input
* 1810117	JE005	JE Group Type Maintenance
* 1810118	JE010	Account Setup Maintenance
* 1810119	JE011	Account/Subledger Xref
* 1810120	JE015	Account Group
* 1810121	JE020	Account/Account Group Xref
* 1810122	JE025	Account Group Template
* 1810123	JE030	Subledger Maintenance
* 1813149	JE100	JE Post Selection/Submittal
* 1847401	JE101	Subledger Detail Query
* 1813150	JE102	Accounting Month Roll Date
* 1847403	JE110	GL MTD Detail Query
* 1847402	JE111	GL History Detail Query
* 1833359	JE305	HIST Reversal Batch Selection
* 1833360	JE310	HIST Reversal Batch Execution
* 1833361	JE315	MTD Reversal Batch Selection
* 1833362	JE320	MTD Reversal Batch Execution
* 1810116	ML006	SOD DRI Links
* 1813146	PN020	PPN Creation
* 1813147	PN025	PPN Query
* 1813168	RC005	Contract Maintenance
* 1813169	RC036	Price Terms Maintenance
* 1813151	RD010	Reversal DOI Deletion
* 1813152	RD030	RD Pre Tracking Results
* 1813153	RD031	RD Post Tracking Results
* 1813170	STG1	Stage Table Data Maintenance
* 1813160	TR010	Lease Master Data - MMS Royalty Reporting
* 1813162	TS006	Severance Tax Common Maintenance
* 1813143	TX020	State Tax Rate Query
* 1813144	TX021	State Tax Rate Maintenance
* 1813171	VA005	Gas Volumes
* 1813172	VA015	Liquid Volumes
* 1813174	VA035	VA Results
* 1813142	VL006	DOI Accounting Rules
* 1810115	VL025	DRI Link Maintenance
* 1810114	VL031	VL Direct Revenue Input
* 1813148	VL032	VL Direct Revenue Input Upload
* 1813154	VL050	VL Full Revenue Results
* 1813141	VL100	Revenue Selection/Submittal
* 1839726	EC005	Escheat Maintenance/History
* 1813167	JE205	Manual JE Batch Maintenance
* 1813167	JE206	Manual JE Detail
* 1813167	JE207	Manual JE Tax and Adj
* 1813167	JE210	Manual JE Batch Approval
* 1813162	TS005	Severance Tax Common Query
* 1813142	VL005	DOI Accounting Data Query
* 1786975	Screens - Division of Interest
* 1789563	Screens - 12 - System and Admin - Batch and Report
* 1786985	Screens - Esuite/Esuite Linking (Business Associates)
* 1786987	Screens - Master Data/Production/Gas Balancing
* 1786977 — Security satellite (all screens)
* 1806481 — API satellite (all screens)

### Team

* Aaron Alejo — Business Analyst
* Asher Maddox — Product Owner
* Sarah Paul — Product Manager
* Jimmy Bidwell — Engineering Lead (dev team)
* Kelsey Pryor — Developer 
* Prasanna Joshi — Developer 
* Connor Morris — Developer 
* Owen House — junior Developer
* Volodymyr Borodaievskii — QA
* Dmytro Domashovets — QA
* Bohdana Redko — QA
* Bohdan Berkut — Developer
* Vitaliy Tomchuk — Developer

\---

## What the New Session Should Do Next

The new project should be set up to work from GitHub as its source of truth. The first working session should:

1. **Verify GitHub access** — Confirm MCP tools can read and write to `Quorum-Enterprise/Quorum.Upstream.AITools`
2. **Design the new workflow** — Taking the recommended changes above and the dev team's analysis as input, restructure the Products story authoring workflow to:

   * Consume dev team's Screen-Spec and analysis files as Phase 1 input
   * Add metadata table queries to Phase 1
   * Add a post-drafting gap check (Phase 1.5 or Phase 4.5 enhancement)
   * Include validation tier classification in -02 stories
   * Establish the feedback loop from gap reports to story amendments
3. **Update the project files in GitHub** — As workflow files are updated (eam-workflow.md, eam-story-core.md, etc.), commit them to the GitHub repo
4. **Test the new workflow** — Pick a screen and run through the restructured workflow end-to-end
