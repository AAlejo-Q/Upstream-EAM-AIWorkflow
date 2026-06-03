# Batch Processes and QPEC — Technical Reference

---

## QRA Batch Process DLLs

### Repo: `Quorum.Upstream.QRA.Batch` (80MB, 11 process libraries)

Each DLL is a .NET project following `Quorum.QRA.Process.QPDll{Name}` naming. Each contains:
- `QPS{ProcessName}.cs` — Process step implementation(s)
- `QSegProc{Name}.cs` — Segmented process wrapper(s)
- `DBGen/` — Database code generation artifacts
- `MsgLog/` — Message log XSL templates

| # | DLL Name | Domain | Key Processes |
|---|----------|--------|---------------|
| 1 | **QPDllRevenueAcctgVA** | Volume/Allocation | VACALC — Volume allocation calculation, well completion date loading |
| 2 | **QPDllCheckWrite** | Check Write | BKRVNU — Check file generation, PDF splitting, Enverus pre-dynamic export, PO owner detail reports |
| 3 | **QPDllBookRev** | Revenue Booking | BKRVNU — Book revenue processing |
| 4 | **QPDllContractualAllocation** | Contractual Allocation | CA processing |
| 5 | **QPDllDivisionOrder** | Division Orders | DOI processing |
| 6 | **QPDllEstimates** | Revenue Estimates | Estimate calculations |
| 7 | **QPDllGasBalancing** | Gas Balancing | Gas balance calculations |
| 8 | **QPDllMarketDeduction** | Market Deductions | Market cost deduction processing |
| 9 | **QPDllRoyaltyReporting** | MMS Royalty | Royalty report generation |
| 10 | **QPDllSeveranceReporting** | Severance Tax | Tax reporting and filing |
| 11 | **IntSAP** | SAP Integration | SAP data exchange interface |

### Additionally in QRA.Batch:
- `Quorum.QRA.Process.Interface/` — Shared process interfaces
- `Quorum.QRA.SegProc.Interface/` — Segmented process interfaces
- `Quorum.QRA.Process.UnitTests/` — Unit tests for batch processes

---

## QPDllCheckWrite Detail (Most Files: 10)

| File | Purpose |
|------|---------|
| `QPSGenerateCheckFile.cs` | Main check file generation process step |
| `QSegProcGenerateCheckFile.cs` | Segmented process wrapper for check generation |
| `QPSCWCheckFileSplitter.cs` | Split large check files by segment |
| `QSeqProcCWCheckFileSplitter.cs` | Sequence wrapper for splitter |
| `QPSSplitPDFByBookmarks.cs` | Split check PDFs by bookmarks |
| `QSegProcSplitPDFByBookmarks.cs` | Segment wrapper for PDF splitter |
| `QPSPOOwnerDetailReport.cs` | Pay Order owner detail report |
| `QSegProcPOOwnerDetailReport.cs` | Segment wrapper for PO report |
| `QPSEnverusPreDynExp.cs` | Enverus pre-dynamic export integration |
| `QCheckExport.cs` | Check export utility |
| `QXMLGenerator.cs` | XML generation for check data |

---

## QPDllRevenueAcctgVA Detail (Volume/Allocation)

| File | Purpose |
|------|---------|
| `QPSLoadWellCompletionDates.cs` | Load well completion dates for VA processing |
| `QPSPrgPstgWellComplDts.cs` | Progress posting for well completion dates |
| `QSegProcLoadWellDates.cs` | Segmented process for well date loading |

---

## QRA ClassicBatch (27MB)

### Repo: `Quorum.Upstream.QRA.ClassicBatch`

Legacy C# batch processes that predate the modern QPDll architecture. These are typically
larger, monolithic batch programs. Not yet source-read — exploration needed to map which
processes are here vs. in QRA.Batch.

---

## Known Batch Process Names (referenced in ClassicGUI screens)

| Process ID | Trigger Screen | Description | DLL |
|-----------|---------------|-------------|-----|
| **DRIPROC** | ML006 (Statement Group) | Process DRI statements | IntSAP or ClassicBatch |
| **JEPOSTONLY** | JE100 (JE Posting Control) | Post journal entries to GL | ClassicBatch |
| **BKRVNU** | — (scheduled) | Book revenue / generate checks | QPDllBookRev + QPDllCheckWrite |
| **VACALC** | — (scheduled) | Volume allocation calculation | QPDllRevenueAcctgVA |
| **TAXJEPOST** | — (scheduled) | Post tax journal entries | ClassicBatch |
| **ESCHEAT** | EC005/EC010 | Process unclaimed property | ClassicBatch |

### Batch Trigger Pattern (from ClassicGUI)
```csharp
// Screen launches batch via QPEC
QUpsUtility.LaunchBatchProcess("DRIPROC", parameters);

// Parameters typically include:
// - OperatingBusinessSegmentCode
// - RunID (e.g., StatementRunID for DRIPROC)
// - Date ranges
// - User ID
```

---

## QPEC — Process Execution Controller

### Repo: `Quorum.Upstream.Application.QPEC` (12MB)

QPEC is the **central orchestrator** for all batch processes across the entire Upstream Suite.
It runs as a service and manages scheduling, execution, monitoring, and logging for all products.

### Architecture

```
┌─────────────────────────────────────────┐
│           QPEC Application              │
│                                         │
│  ┌──────────┐  ┌────────────────────┐  │
│  │ Schedule  │  │ Process Execution  │  │
│  │ Engine    │──│ Engine             │  │
│  └──────────┘  └────────────────────┘  │
│       │              │                  │
│       ▼              ▼                  │
│  QScheduleEngine.ini  QPDll*.dll        │
│                      IntSAP.dll         │
│                      ClassicBatch.exe   │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  QARCH_PROC_* tables (Metadata DB)      │
│  - Process definitions                  │
│  - Process steps                        │
│  - Process parameters                   │
│  - Execution history                    │
│  - Schedule definitions                 │
└─────────────────────────────────────────┘
```

### Configuration Files

| File | Purpose |
|------|---------|
| `QPEC.ini` | Default QPEC configuration |
| `QPEC.Cloud.ini` | Cloud deployment config |
| `QPEC.OnPremDirect.ini` | On-premises direct connection config |
| `QPEC.OnPremNI.ini` | On-premises non-interactive config |
| `QScheduleEngine.ini` | Schedule engine configuration |
| `QScheduleEngine.Cloud.ini` | Cloud schedule config |
| `QScheduleEngine.OnPremDirect.ini` | On-prem schedule config |
| `QSubscribers.xml` | Event subscriber definitions |
| `QDataTranslations.xml` | Data translation mappings |
| `metadata.config.xml` | Metadata connection configuration |

### Deployment Modes
1. **Cloud** — Azure-hosted QPEC (target for EAM Epic 2 Feature 2.13)
2. **OnPremDirect** — Direct database connection
3. **OnPremNI** — Non-interactive (service mode)
4. **Debug** — Local development

### Native Bridge
`Quorum.Upstream.Application.QPEC.Native/` — Contains C++ interop for legacy batch processes
that are still compiled as C++ (referenced via `Quorum.QFC.CPP` git submodule).

### Screen Integration
Screens launch processes via `QUpsUtility.LaunchBatchProcess()` which:
1. Creates a process execution record in `QARCH_PROC_EXEC_*` tables
2. Passes parameters as key-value pairs
3. QPEC picks up the execution record and dispatches to the correct DLL
4. Progress is tracked via `QARCH_PROC_EXEC_STATUS` table
5. Screens can monitor via QP010 (Process Execution Monitor) and QP043 (Process Launcher)

---

## Shared.Batch Framework

### Repo: `Quorum.Upstream.Shared.Batch` (8.5MB)

Contains the shared batch framework used by all products, plus cross-product integration processes.

### Projects

| Project | Purpose |
|---------|---------|
| **Quorum.Upstream.Shared.Process.Interface** | Core process step interfaces (IProcessStep, ISegmentedProcess) |
| **Quorum.Upstream.Shared.SegProc.Interface** | Segmented process interfaces |
| **Quorum.Upstream.Shared.QPECProcessSteps** | Shared QPEC process step implementations |
| **Quorum.Upstream.Shared.Process.QDynamicExport** | Dynamic data export engine |
| **Quorum.Upstream.Shared.SQLReporting** | SQL Reporting Services integration |
| **Quorum.Upstream.Shared.DocManager** | Document management batch operations |
| **Quorum.Upstream.Shared.UpstreamWebServices** | Web service batch clients |
| **Quorum.Upstream.Shared.IntQRA** | QRA-specific integration processes |
| **Quorum.Upstream.Shared.IntQCFS** | QCFS-specific integration processes |
| **Quorum.Upstream.Shared.IntQLS** | QLS-specific integration processes |
| **Quorum.Upstream.Shared.IntSharedLib** | Shared integration library |
| **Quorum.Upstream.Shared.IntUpstream** | Cross-product upstream integration |
| **Quorum.Upstream.Shared.IntUpstreamServiceItems** | Service item definitions |
| **Quorum.Upstream.Shared.IntXmlDataProvider** | XML data provider for integration |

### Process Framework Pattern
```
IProcessStep                    — Single atomic process step
  └─ QPSxxx.cs                 — Implementation (e.g., QPSGenerateCheckFile)

ISegmentedProcess               — Multi-segment process (runs per business segment)
  └─ QSegProcxxx.cs            — Wrapper that iterates segments
       └─ calls IProcessStep    — Delegates to step implementation

Process.Interface               — Shared interfaces
SegProc.Interface               — Segmented interfaces
QPECProcessSteps                — Shared utility steps
```

### Deployment Scripts
```
Quorum.Upstream.QRA.Copy-Batch.ps1        — Copy QRA batch DLLs to deployment
Quorum.Upstream.QCA.Copy-Batch.ps1        — Copy QCA batch DLLs
Quorum.Upstream.QCFS.Copy-Batch.ps1       — Copy QCFS batch DLLs
Quorum.Upstream.Shared.Copy-App-UpstreamQPEC.ps1 — Copy QPEC app
```

---

## Batch Process Lifecycle (General Pattern)

```
1. TRIGGER
   └─ User clicks "Submit" on a screen (e.g., ML006 → DRIPROC)
   └─ OR scheduled by QScheduleEngine
   └─ OR triggered by another process completing

2. STAGING
   └─ Screen stages data to a staging/snapshot table
      (e.g., RTRN_MI_STMNT_INPUT_SNAPSHOT for DRIPROC)
   └─ Gets a RunID from sequence generator

3. LAUNCH
   └─ QUpsUtility.LaunchBatchProcess("PROCESS_NAME", params)
   └─ Creates record in QARCH_PROC_EXEC_* tables
   └─ QPEC picks up the execution request

4. EXECUTION
   └─ QPEC loads the QPDll or ClassicBatch executable
   └─ For segmented processes: iterates business segments
   └─ Each step logs progress to QARCH_PROC_EXEC_STATUS

5. OUTPUT
   └─ Results written to output tables
   └─ Status updated (Success/Failure/Warning)
   └─ Message log written (XML format)
   └─ Downstream processes may be triggered automatically

6. MONITORING
   └─ QP010 (Process Execution Monitor) shows real-time status
   └─ QP043 (Process Launcher) allows manual re-launch
   └─ QP045 (Process Execution Monitor Query) for history
```

---

## QCA and QCFS Batch Processes

### QCA.Batch (570KB)
Smaller set of batch processes for cost accounting. Integration processes handle:
- JIB billing calculations
- Revenue allocation processing
- Expense rate calculations

### QCA.ClassicBatch (3.7MB)
Legacy batch processes. Larger than the modern QCA.Batch, suggesting significant
legacy code still active.

### QCFS.Batch (1MB)
Batch processes for consolidated financials:
- Period close processing
- GL posting from subledgers
- Bank reconciliation matching
- Depreciation calculations
- 1099/reporting generation

---

## TO EXPLORE NEXT

1. [ ] Read DRIPROC batch source — trace full lifecycle from ML006 submit to revenue output
2. [ ] Read JEPOSTONLY batch source — understand JE posting parameters and GL table writes
3. [ ] Read BKRVNU (QPDllBookRev + QPDllCheckWrite) — check generation lifecycle
4. [ ] Read QScheduleEngine configuration — understand batch scheduling
5. [ ] Explore QPEC.Native bridge — which processes still use C++ backend
6. [ ] Map QRA.ClassicBatch contents vs QRA.Batch to understand modern vs legacy split
7. [ ] Read Shared.Batch Process.Interface to understand IProcessStep/ISegmentedProcess pattern
