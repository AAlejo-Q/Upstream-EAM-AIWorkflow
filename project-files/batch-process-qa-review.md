---
name: batch-process-qa-review
description: >
  Independently analyze a QRA/QDO batch process, search ADO for related bugs across all projects,
  validate the QA team's EAM Epic 5 wiki against codebase findings, generate DRI-scoped test scenarios
  with regression coverage from real bugs, and produce PO/SME feedback for the QA team. Trigger on:
  'review the wiki for [process]', 'validate QA wiki', 'analyze [process]', 'bugs for [process]',
  'test scenarios for [process]', 'assess for automation', or any Epic 5 batch process name (TAXJEPOST,
  TAXCOMBO, JEPOSTONLY, BKRVNU, DRIPROC, VLPROC, VACALC, CWPROC, RDPROCNEW, UNDOPROC, VLMIUPLOAD,
  VLMIUSER, VLDTLUPLD, VLDTLUSER, BKRVN-PPN, etc.) alongside 'wiki', 'review', 'QA', or 'automation'.
  Key principle: build independent understanding FIRST from code and metadata, then compare against wiki.
---

# Batch Process QA Review — SME Validation & Test Scenario Generation

> **Purpose:** Enable Aaron (PO/SME) to independently validate QA team wiki documentation for
> Quorum Upstream batch processes and generate comprehensive, DRI-workflow-scoped test scenarios
> that account for real production bugs.
>
> **Key principle:** Build independent understanding FIRST, then compare against the wiki. Never
> read the QA wiki before completing the independent analysis — the wiki is the thing being
> validated, not a source of truth.

---

## When to Use

- Aaron asks to review, validate, or assess a QA team wiki for a batch process
- Aaron asks to analyze a batch process for QA automation candidacy
- Aaron asks to find bugs or work items related to a batch process
- Aaron asks to create test scenarios for a batch process
- Any EAM Epic 5 batch process name is mentioned alongside "wiki", "review", "QA", "test scenarios", or "automation"

---

## Workflow Overview

```
Phase 1: Independent Analysis (DO NOT read QA wiki yet)
  → Code search → Metadata lookup → Screen/controller analysis → Build understanding

Phase 2: Bug & Work Item Discovery
  → Search ADO across ALL projects → Catalog bugs, requirements, requests → Extract regression scenarios

Phase 3: Wiki Validation (NOW read the QA wiki)
  → Compare wiki against independent findings → Score by category → Identify gaps, errors, missing coverage

Phase 4: Test Scenario Generation
  → Scope to relevant workflow (DRI for VL processes) → Generate categorized scenarios → Include regression from bugs

Phase 5: SME Feedback
  → Summarize gaps for Aaron to relay to QA → Flag risks → Suggest phased automation approach
```

---

## Phase 1: Independent Analysis

**Goal:** Build a complete technical understanding of the batch process WITHOUT consulting the QA wiki.

### Step 1.1 — Load Required Skills

Before any ADO or code operations, always:
1. Read `ado-connection/SKILL.md` and follow the auth bootstrap
2. Read `upstream-codebase-guide/SKILL.md` for repo catalog and module maps
3. Load ADO tools via `tool_search`: code search, work item search, wiki tools, and work item CRUD

### Step 1.2 — Process Metadata Lookup

Use the Quorum Metadata MCP to get the process definition:

```
quorum-metadata-dev:get_batch_process_all_details → processId: ["<PROCESS_ID>"]
```

This returns:
- **Process definition** (QARCH_CTRL_PROCESS): name, description, priority, estimated runtimes, cancellable flag, cleanup process
- **Parameters** (QARCH_CTRL_PROCESS_PARAM): input params with IDs, sequence, required/optional, defaults
- **Step associations** (QARCH_CTRL_PROC_PROCSTEP): ordered steps with sequence numbers
- **Process type** (QARCH_CTRL_PROCESS_TYPE): QRA_SCRN, QRA_ADMIN, etc. — tells you if it's screen-launched or admin-runnable
- **Lock configuration** (QARCH_LOCK_MASTER_PROCESS): lock behavior, retry settings

Also look up the individual process steps:
```
quorum-metadata-dev:get_process_step_all_details → processStepId: ["<STEP_ID_1>", "<STEP_ID_2>"]
```

This gives you the DLL, class name, connection ID, and step-level parameters.

**Always look up related processes too.** If the process is a composite (like TAXJEPOST = TRTAXCOMBO + JEPOST), also look up any standalone versions of those steps (e.g., TAXCOMBO, JEPOSTONLY) to understand the full process family.

### Step 1.3 — Code Search

Search across all repos for the process ID. Start broad with `includeFacets: true` and `top: 2` to see which repos contain references:

```
azure-devops:search_code → searchText: "<PROCESS_ID>", project: "QuorumSoftware", includeFacets: true, top: 2
```

The facets tell you which repos to drill into. Key repos to check:

| Repo Pattern | What You'll Find |
|---|---|
| `Quorum.Upstream.QRA.Web` | Modern web controller — invocation logic, decision branching, pre-submission validations |
| `Quorum.Upstream.QRA.ClassicGUI` | WinForms screen — same logic as Web but in the legacy UI, often more complete |
| `Quorum.Upstream.QRA.ClassicBatch` | C++ batch source — internal processing logic (TRTAXCOMBO, JEPOST internals, etc.) |
| `Quorum.Upstream.QRA.Batch` | .NET batch DLLs |
| `Quorum.Upstream.Metadata` / `*.Upstream.Metadata` | Process registration in QARCH tables (JSON metadata files) |
| `Quorum.Upstream.Database` | SQL scripts, stored procedures, triggers |

**What to extract from each code hit:**

From **Web/ClassicGUI controllers:**
- How is the process invoked? (which screen, which button, what user action)
- What decision logic determines which process to launch? (e.g., TAXJEPOST vs JEPOSTONLY)
- What pre-submission validations does the screen perform before submitting to QPEC?
- What parameters are passed to the process?
- What confirmation prompts does the user see?

From **ClassicBatch/Batch source:**
- Internal step-by-step processing logic
- Error codes and their trigger conditions
- Database tables read from and written to
- Transaction boundaries (BEGIN TRAN / COMMIT / ROLLBACK)
- Config keys that alter behavior

### Step 1.4 — Compile Independent Analysis

Organize findings into these sections:

1. **What it is** — Process name, ID, composite steps, DLLs
2. **Business purpose** — What it does in plain terms, where it sits in the month-end close cycle
3. **Technical specification** — Metadata, parameters, steps, estimated runtimes, lock config
4. **Invocation points** — Which screens, which buttons, what decision logic routes to this process
5. **Pre-execution validations** — Screen-level guards before process submission
6. **Key database tables** — Input tables, output tables, temp tables, conflict tables
7. **Related processes** — The process family (standalone variants, undo/clean process)
8. **Configuration dependencies** — Config keys that alter behavior

> **CRITICAL:** Do NOT read the QA wiki until this phase is complete. The independent analysis
> is the baseline for validation. If you read the wiki first, you'll anchor on it and miss gaps.

---

## Phase 2: Bug & Work Item Discovery

**Goal:** Find every ADO work item that mentions the process — bugs, requirements, requests, tasks — across ALL projects.

### Step 2.1 — Broad Work Item Search

Search without project filter to catch bugs in QuorumSoftware, Quorum, QuorumServices, myQuorum Cloud, etc.:

```
azure-devops:search_workitem → searchText: "<PROCESS_ID>", top: 25
```

### Step 2.2 — Categorize Findings

Sort results into:

| Category | What to Look For |
|---|---|
| **Bugs (direct)** | Bugs where the process ID appears in the title, repro steps, or root cause analysis |
| **Bugs (indirect)** | Bugs where the process is mentioned in comments/history as part of investigation |
| **QA automation stories** | Existing stories under Epic 5 for this process (functional analysis, technical analysis, AT development, validation) |
| **Requirements/patches** | Software update packages, hotfixes that included fixes for this process |
| **Cloud ops items** | Script reviews, database refreshes, outage RCAs mentioning this process |

### Step 2.3 — Extract Regression Scenarios from Bugs

For each bug, capture:

- **ADO ID and project** — The work item number and which project it's in
- **Root cause summary** — One sentence explaining what went wrong
- **Tables involved** — Which DB tables were affected
- **Reproducibility notes** — What data setup or conditions triggered the bug
- **Fix status** — Closed/fixed, or still open

These become Category G (Regression) test scenarios in Phase 4.

### Step 2.4 — Note Existing QA Work

Check if the QA team already has stories in flight for this process. Record:
- Which stories exist and their state (New, Active, Test, Closed)
- Which wiki page they reference (this is the wiki you'll validate in Phase 3)
- Who's assigned

---

## Phase 3: Wiki Validation

**Goal:** NOW read the QA wiki and compare it against your independent findings.

### Step 3.1 — Fetch the Wiki

The QA team's wiki pages live under the EAM Epic 5 wiki path. Fetch using:

```
azure-devops:wiki_get_page_content → url: "<wiki_url_from_story_comments>"
```

Or if you don't have the URL, search:
```
azure-devops:search_wiki → searchText: "<PROCESS_ID>", project: ["Quorum"]
```

> **IMPORTANT:** Do NOT use the EAM wiki base path from Aaron's memory
> (`/Quorum/North America/Upstream/QA Strategy/Revenue/QRA/Batch Processes`) as a source
> if Aaron explicitly says to do an independent analysis first. That wiki IS the thing being validated.

### Step 3.2 — Systematic Comparison

Compare the wiki against your independent analysis using this scorecard:

| Category | What to Check | Grade Criteria |
|---|---|---|
| Process identity & metadata | Process ID, name, app layer, parameters, steps, DLLs, runtimes | A = complete and accurate; C = mostly right; F = missing or wrong |
| Internal processing logic | Step-by-step execution, business rules, error handling, transaction safety | A = source-level detail; C = high-level only; F = missing |
| Screen-level invocation/routing | How the process gets triggered, decision logic between process variants | A = documented; F = not covered |
| Screen-level pre-submission validations | Guard checks before QPEC submission | A = documented; F = not covered |
| Output tables & verification | All tables written to, with verification queries | A = complete; C = partial (e.g., only covers one step's outputs); F = missing key outputs |
| Historical bug regression scenarios | References to production bugs with test scenarios | A = bugs cataloged with scenarios; F = no bug references |
| Error code catalog | Error codes with trigger conditions | A = comprehensive; C = partial; F = missing |
| Test scenarios — positive | Happy path coverage | A = covers full workflow; C = covers internals only; F = missing |
| Test scenarios — negative | Guard checks and failure handling | A = covers both screen and batch level; C = batch only; F = missing |
| Data preparation & verification | Setup queries, input data examples, output verification queries | A = complete; C = partial; F = missing |

### Step 3.3 — Identify Gaps

For each gap, document:
- **What's missing** — Specific content that should be there
- **Why it matters** — Impact on test coverage or automation accuracy
- **Where in the code** — Point to the source file/method that proves the gap
- **Recommendation** — What the QA team should add

### Step 3.4 — Flag Inaccuracies

If the wiki states something that contradicts the code, call it out with evidence.

---

## Phase 4: Test Scenario Generation

**Goal:** Generate a comprehensive, categorized test scenario catalog scoped to the relevant workflow.

### Step 4.1 — Scope to the Relevant Workflow

**DRI workflow scope (for VL-sourced processes):** Only include scenarios relevant to the DRI
month-end close cycle. Exclude non-VL source testing (MJE, CW, PCW, CI) — those belong to
their respective Feature automation efforts.

Ask Aaron to confirm the scope if it's ambiguous.

### Step 4.2 — Scenario Categories

Use these standard categories:

| Category | Layer | Description |
|---|---|---|
| **A: Process Routing** | SCREEN | How the screen decides which process to launch |
| **B: Pre-Submission Validations** | SCREEN | Guard checks before QPEC submission |
| **C: Happy Path (Step-by-Step)** | BATCH + DATA | Core processing logic with data verification |
| **D: Batch-Level Negative** | BATCH | Failure handling inside the batch steps |
| **E: Output Verification** | BATCH + DATA | Post-execution output table and status verification |
| **F: End-to-End Integration** | ALL | Full workflow from upstream through downstream consumer |
| **G: Regression (from bugs)** | ALL | Scenarios derived from historical production bugs |

### Step 4.3 — Scenario Table Format

Use this format for each scenario:

```markdown
| ID | Scenario | Pri | Source | Pre-Conditions | Action | Expected Result | Verification |
```

- **ID:** Category letter + sequence (e.g., A-01, G-03)
- **Pri:** P1 (must-have), P2 (important), P3 (edge case)
- **Source:** WIKI (in QA wiki), NEW (gap from independent analysis), REGRESSION (from ADO bug)

### Step 4.4 — Build Regression Scenarios (Category G)

For each bug discovered in Phase 2, create a test scenario that:
1. Describes the original failure condition
2. Specifies the data setup that triggers it
3. Defines what "correct behavior" means (the fix)
4. Links back to the ADO bug ID

### Step 4.5 — Summary Table

Always end with a summary showing count of scenarios by category, priority, and whether they're new vs already in the wiki. This makes the coverage gap immediately visible.

### Step 4.6 — Automation Phasing

Suggest a phased automation order:
- **Phase 1:** Routing + full happy path → establishes the framework
- **Phase 2:** Pre-submission validations + batch negatives → defensive coverage
- **Phase 3:** Detailed data verification + regression → depth and confidence

---

## Phase 5: SME Feedback

**Goal:** Provide Aaron with actionable feedback to relay to the QA team.

### Step 5.1 — Gap Summary for QA

Create a concise bullet list of what's missing from the wiki, written for Aaron to copy/paste to the QA team. No jargon about "layers" or "categories" — just plain statements of what needs to be added.

### Step 5.2 — Risk Flags

Call out any technical risks discovered during analysis:
- Configuration flags that could cause unexpected behavior (e.g., STOP_PROCESS_ON_ERR_IND = false)
- Tables that are conflict-prone (e.g., DONL_DO_HDR with GXRF_DESKID_RVNU_VAL)
- Performance-sensitive operations (e.g., bulk INSERTs that caused production outages)

### Step 5.3 — Process Triggering Clarification

Always document HOW the process is triggered from the user's perspective:
- Which screen, which button, what the user sees
- Whether the process can be run from the batch scheduler directly (check PROCESS_TYPE_CD)
- Any confirmation prompts the user sees before submission

This is critical for QA automation — they need to know whether to drive through the screen or submit directly to QPEC.

---

## Output Artifacts

Each review session should produce:

| Artifact | Format | Content |
|---|---|---|
| Independent Analysis | Markdown file | Full technical analysis (Phase 1 + 2 output) |
| Wiki Validation & Test Scenarios | Markdown file | Comparison scorecard + categorized test scenarios (Phase 3 + 4 output) |
| DRI-Scoped Test Scenarios | Markdown file (if requested) | Filtered scenarios for the relevant workflow only |
| Gap Summary | Inline (conversation) | Bullet list for Aaron to relay to QA |
| Regression Table | Inline (conversation, copyable) | Category G scenarios in table format |

---

## Anti-Patterns — What NOT to Do

- **Do NOT read the QA wiki before completing the independent analysis.** The wiki is the artifact under review, not a reference source. Reading it first anchors your analysis.
- **Do NOT include test scenarios for workflows outside the DRI scope** (unless Aaron explicitly asks). MJE, CW, PCW, CI routing tests belong to their respective Feature automation.
- **Do NOT create ADO work items without Aaron's review.** Always present findings for review first.
- **Do NOT assume the wiki is wrong when it says something you didn't find.** The wiki may have details from C++ source that's harder to discover via code search. Flag it as "could not independently verify" rather than "incorrect."
- **Do NOT skip the bug search.** Historical bugs are the highest-value source of regression test scenarios. Always search across all ADO projects (QuorumSoftware, Quorum, QuorumServices, myQuorum Cloud).

---

## Example Session Flow

**Aaron says:** "Review and analyze the TAXJEPOST batch process. Look for bugs. Don't use the QA wiki."

**Claude does:**
1. Loads ADO tools, verifies connection
2. Searches code for "TAXJEPOST" across repos → finds Constants.cs, QUIControllerJournalEntryPosting.cs, QFrmJEPostingControl.cs
3. Looks up metadata: `get_batch_process_all_details(["TAXJEPOST", "TAXCOMBO", "JEPOSTONLY"])` + `get_process_step_all_details(["TRTAXCOMBO", "JEPOST"])`
4. Reads the full source from Web controller and ClassicGUI to understand decision logic and validations
5. Searches ADO work items: `search_workitem("TAXJEPOST", top: 25)` → finds 6 bugs, 4 QA stories, 3 cloud ops items
6. Compiles independent analysis into a markdown file
7. Presents to Aaron for review

**Aaron says:** "Now validate the QA wiki and create test scenarios."

**Claude does:**
1. Fetches the QA wiki page via `wiki_get_page_content`
2. Systematically compares wiki against independent analysis using the scorecard
3. Identifies 7 gaps (screen routing, pre-submission validations, JEPOST outputs, DOI cleanup, bugs, STOP_ON_ERR risk, parameter ambiguity)
4. Generates 49 DRI-scoped test scenarios across 7 categories
5. Produces summary table showing 32 of 49 (65%) are new/not in wiki
6. Suggests 3-phase automation approach

**Aaron says:** "List out what's missing from the wiki for me to tell QA."

**Claude does:**
1. Produces concise bullet list of gaps, organized by theme
2. Written in plain language Aaron can copy to QA team

---

*This skill is the PO's lens on QA documentation quality — bridging domain knowledge, codebase
reality, and production history into actionable feedback that makes test automation more comprehensive.*
