# EAM Workflow Restructuring — Session Handoff

**Source session:** June 3, 2026 (Aaron Alejo + Claude, EAM Claude Desktop project)
**Purpose:** Full context transfer for the new EAM project — captures all decisions, analysis, and next steps so a new Claude session can continue without losing ground.

---

## Who You're Working With

- **Aaron Alejo** — Business Analyst, EAM project at Quorum Software
- **Asher Maddox** — Product Owner, co-manages the EAM project and Claude workspace
- Both are on the Products side of the myQ Accounting Modernization (EAM) project — migrating QRA/QDO classic desktop screens to a modern Azure-based web application

---

## The Three Goals

### Goal 1 — Version-Controlled Project Migration (DECIDED)

**Problem:** Today, Aaron and Asher manage ~24 project knowledge files and ~9 skill files through a manual cycle: edit locally → delete old version from Claude project → re-upload new version. This is error-prone, creates sync gaps, and scales badly.

**Decision:** GitHub is the **single source of truth**. All project files and skills live in a GitHub repo. The Claude project pulls files from GitHub via MCP rather than relying on static uploads.

**Why this works:** The EAM project is above the RAG retrieval threshold (~450KB of project files + skill content). Above this threshold, Claude's project knowledge already does selective chunk retrieval internally — the same as a tool call. The token overhead difference between project knowledge retrieval and GitHub MCP fetch is ~200-500 tokens per call, which is marginal at this scale.

**Architecture:**
- Project instructions (system prompt) live in the Claude project itself — this is the bootstrap
- Everything else lives in GitHub and is fetched on demand
- No more manual delete/re-upload cycle
- Version history, diffs, and rollback come free with git

**GitHub repo:** `https://github.com/AAlejo-Q/Upstream-EAM-AIWorkflow`

### Goal 2 — Cross-Functional Skill Alignment (Products ↔ Dev)

**Problem:** Products (Aaron/Asher) and Dev operate with separate Claude instances that have different skills, different analysis methods, and different mental models. Stories get authored with one set of assumptions, then Dev's Claude reads them through a different lens — finding missed validations, misinterpreted scope, and functionalities that fell through the cracks.

**Status:** Full analysis of the dev team's workflow has been completed. Specific alignment gaps identified (see "Gap Analysis" section below). Recommendations drafted but not yet implemented.

### Goal 3 — Deeper Analysis to Reduce Gaps and Assumptions

**Problem:** The current Products Phase 1 analysis catches a lot but things still slip through — especially screen-specific behaviors, edge-case validations, and code paths that aren't obvious from the main form class. The dev team's analysis skill (`Analyze-classic-gui-screen`) is significantly more thorough, querying 7+ metadata tables and dispatching 3 parallel sub-agents.

**Status:** Specific enhancement areas identified from dev team's analysis skill. Recommendations drafted but not yet implemented.

---

## What's Been Done

### 1. GitHub Repo Created and Populated

**Repo:** `https://github.com/AAlejo-Q/Upstream-EAM-AIWorkflow` (public)

Contains all 24 project knowledge files under `project-files/` and all 9 skill files under `skills/`.

### 2. GitHub MCP Configured in Claude Desktop

Aaron's `claude_desktop_config.json` has been updated with a `github` MCP server entry using `@modelcontextprotocol/server-github`.

### 3. GitHub Access Skill Created

A `github-access` skill bootstraps GitHub MCP access, preferring MCP tools with curl fallback.

### 4. Dev Team's Workflow Fully Analyzed

The dev team's repo and all ADO wiki documentation have been read and analyzed.

---

## Dev Team's Workflow — Full Analysis

### Architecture

The dev team uses **Claude Code** running out of `Quorum-Enterprise/Quorum.Upstream.AITools`. They have direct file system access to local source repo clones, SQL Server MCP for metadata queries, Azure DevOps MCP for code search, Serena MCP for semantic analysis, and GitHub CLI for package management.

### Their Migration Lifecycle (8 Steps)

| Step | Skill | Purpose | Output |
|------|-------|---------|--------|
| 1 | `Analyze-classic-gui-screen` | Deep analysis with 3 parallel sub-agents | 5 markdown files |
| 2 | `Spec-screen` | Migration blueprint | `Screen-Spec.md` |
| 3 | `Analyze-ado-stories` | Gap-analyze Products' stories vs spec | `ADO-Story-Analysis.md` |
| 4a | `Migrate-foundation-CRUD` | Foundation + CRUD (9 phases) | Code + RemainingWork.md |
| 4b | `Migrate-validations` | Validation rules (4-tier) | Validation classes + tests |
| 4c | `Migrate-special` | Batch process actions | UIController + JS |
| 5 | `Audit-screen` | 6-category gap analysis | Audit report + readiness gate |
| 6 | `Remediate-screen` | Apply fixes (never auto-remediates contradictions) | Code fixes |
| 7a-c | `Unit-test-*` (split) | MSTest with AC traceability | 5-6 test files |
| 8 | `Document-screen` | Documentation | HTML + Markdown |

### Key Skill Details

**Analyze-classic-gui-screen** — 3 parallel sub-agents: Agent A (14+ code searches), Agent B (7+ metadata SQL queries), Agent C (CodeGen XML). Resolves RES_MSG_* resource strings. Produces Overview, Technical Analysis, SQL Analysis, CodeGen Analysis, Test Scenarios.

**Spec-screen** — Fills comprehensive template from analysis files. Covers database, data objects, service layer, screen layout, every business rule individually, lookups, context dependencies, special behaviors, navigation, print/export, Notes (full DB lookup), unit test guidance, open questions.

**Analyze-ado-stories** — Fetches Products' ADO stories, compares ACs against spec. 4-way analysis: stories-not-in-spec, spec-only, contradictions, open questions. Report consumed by all downstream skills.

**Migrate-validations** — 4-tier: Tier 1 (in-memory) fully implemented, Tier 2 (DB-lookup) fully implemented, Tier 3 (commit-time) fully implemented, Tier 4 (side-effects) stubbed.

**Audit-screen** — 6 categories with migration readiness gate that blocks on unresolved contradictions.

### Dev Enhancement Efforts In Progress
- `Migrate-foundation-CRUD` — needs more gates (US 1813786)
- `Migrate-validations` — TDD approach (US 1810286)
- `Audit/Remediate` — Prasanna reviewing combination
- Unit tests — Connor adjusting coverage alignment

---

## Gap Analysis: Products ↔ Dev

### Gap 1: Duplicate Analysis, Different Findings
Both sides analyze screens independently. Dev's analysis is far more thorough (7+ metadata tables, 14+ code searches, resource string resolution, CodeGen status). Different findings → divergent stories and specs.

### Gap 2: Screen-Spec vs Stories — Parallel Artifacts
Screen-Spec is dev's source of truth. Our stories are a separate artifact with different structure. `Analyze-ado-stories` compares two independently-authored documents.

### Gap 3: One-Directional Feedback Loop
Gap reports feed into dev pipeline but never come back to Products. Stories never get updated to close the gaps.

### Gap 4: Validation Tier Misalignment
Dev classifies rules into 4 tiers. Our -02 stories use user-voice G/W/T without tiering. Dev's Claude re-derives classification — sometimes incorrectly.

---

## Recommended Changes (Not Yet Implemented)

### Goal 2 (Alignment):
1. Consume dev's Screen-Spec and analysis files as Phase 1 input
2. Add Phase 1.5 gap check after drafting stories
3. Close the feedback loop from gap reports to story amendments
4. Structure -02 ACs to map to dev's 4-tier validation classification

### Goal 3 (Deeper Analysis):
1. Metadata table queries (QARCH_CNFG_GRIDCTRL_NET, COL_NET) via quorum-metadata-dev MCP
2. SQL key inventory (QARCH_SQL by screen prefix)
3. Resource string resolution (canonical RES_MSG_* text in -02 ACs)
4. Pick list and code table validation (CDTBL_ID, PKLIST_ID)
5. Notes behavior detection workflow

---

## Key References

### Repos
- Products: `https://github.com/AAlejo-Q/Upstream-EAM-AIWorkflow`
- Dev team: `https://github.com/Quorum-Enterprise/Quorum.Upstream.AITools` (private)

### ADO Wikis
- Skills overview: `.../11534/Upstream-AI-Skills`
- Migration: `.../12308/Migration-Skills`
- Supporting: `.../12310/Supporting-Skills`
- Packages: `.../12311/Package-Management-Tools`

### ADO
- Code search org: `QuorumSoftware`
- Stories project: `Quorum`
- Features: 1786972 (DRI), 1786978 (JE), 1786977 (Security), 1806481 (API), 1792329 (JE100)

### Team
Aaron Alejo (BA), Asher Maddox (PO), Sarah Paul (PM), Jimmy Bidwell (Eng Lead), Kelsey Pryor (Dev), Prasanna Joshi (Dev), Connor Morris (Dev)