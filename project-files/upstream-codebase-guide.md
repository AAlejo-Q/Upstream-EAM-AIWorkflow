---
name: upstream-codebase-guide
description: >
  Comprehensive technical reference for the Quorum Upstream Suite codebase covering all four products
  (QRA, QDO, QCA, QCFS) plus shared infrastructure (QPEC, Metadata, Shared.Batch, Shared.Web).
  Use this skill whenever you need to understand screen-to-code mappings, batch process lifecycles,
  cross-module data flows, metadata architecture, or any aspect of the Quorum Upstream codebase
  for analysis, work item creation, migration planning, or technical gap assessment. Trigger on any
  mention of: ClassicGUI forms, QRA modules (VL, JE, CW, VA, DO, MF, TS, TV, TR, etc.), QDO
  division orders, QCA cost accounting, QCFS consolidated financials, QPEC process execution,
  batch processes (DRIPROC, JEPOSTONLY, BKRVNU, VACALC, etc.), Upstream database tables,
  business rules, screen patterns (filter-detail, master-detail), code table lookups, pick lists,
  security architecture, or any Quorum Upstream technical question. Also trigger when exploring
  repos in the QuorumSoftware ADO organization.
---

# Quorum Upstream Suite — Codebase Learning Guide

## Purpose

This is a living reference for understanding the Quorum Upstream Suite codebase. It maps every
product, module, batch process, and shared infrastructure component to help with migration
analysis, work item creation, and technical gap assessment for the EAM project.

**ADO Organization**: QuorumSoftware (`dev.azure.com/QuorumSoftware`)
**ADO Project for repos**: QuorumSoftware
**ADO Project for work items**: Quorum

## How to Use This Skill

1. **Start here** for the product/module overview and repo catalog
2. **Read `references/qra-modules.md`** for deep QRA ClassicGUI module maps (screen→class→table→rules)
3. **Read `references/qdo-qca-qcfs.md`** for the other three products
4. **Read `references/batch-and-qpec.md`** for batch process lifecycles and QPEC orchestration
5. **Read `references/shared-infrastructure.md`** for Metadata, Shared.Batch, Shared.Web, Database

When exploring code, always use `azure-devops:repo_list_directory` and `azure-devops:search_code`
to read actual source files. This guide tells you WHERE to look; the repos have the truth.

---

## Product Suite Overview

The Quorum Upstream Suite consists of four products sharing a common infrastructure:

| Product | Full Name | Purpose | ClassicGUI Repo | File Count |
|---------|-----------|---------|-----------------|------------|
| **QRA** | Revenue Accounting | Revenue processing, DRI, JE posting, check write, tax | QRA.ClassicGUI | 1,566 |
| **QDO** | Division Orders | DOI management, owner interest tracking | *Shared with QRA ClassicGUI (DO module)* | — |
| **QCA** | Cost Accounting | JIB, GL, subledger, revenue, timekeeping | QCA.ClassicGUI | ~115 screens |
| **QCFS** | Consolidated Financial System | AP, AR, GL, bank recon, fixed assets | QCFS.ClassicGUI | ~438 files |

### Key Architectural Insight
QDO has NO separate ClassicGUI or Database repo (both are size=0). QDO screens live inside
`QRA.ClassicGUI` under the `Quorum.Upstream.QRA.DO` module (157 files). QDO has its own
Web repo (27MB) with the modern service layer, API, and web app.

---

## Complete Repository Catalog (75 repos)

### QRA — Revenue Accounting (16 repos)

| Repo | Size | Purpose |
|------|------|---------|
| `QRA.ClassicGUI` | 15MB | WinForms source — **THE migration source** for EAM |
| `QRA.ClassicGui.Tax` | 137KB | Tax-specific ClassicGUI extensions |
| `QRA.Web` | 14MB | Existing ASP.NET MVC web app (layered architecture) |
| `QRA.Batch` | 80MB | .NET batch process DLLs (11 process libraries) |
| `QRA.ClassicBatch` | 27MB | Legacy C# batch code |
| `QRA.Database` | 11MB | SQL schema definitions |
| `QRA.API` | 3.2MB | REST API endpoints |
| `QRA.API.Specs` | 2.7MB | API specification/tests |
| `QRA.Application.APIHost` | 398KB | API hosting layer |
| `QRA.Application.MiddleTier` | 1.7MB | Deployment host (multi-config) |
| `QRA.Application.Web` | 58MB | Web application host |
| `QRA.Tax` | 195KB | Tax calculation library |
| `QRA.Events` | 40KB | Domain event definitions |
| `QRA.Messaging.Publisher` | 60KB | Event publishing |
| `QRA.DocumentManagement.EventHandlers` | 166KB | Document event handlers |
| `QRA.DesignStudio.PackageSource` | 81KB | Design studio config |

### QDO — Division Orders (10 repos)

| Repo | Size | Purpose |
|------|------|---------|
| `QDO.Web` | 27MB | Modern web app (ServiceCore/Client/Controllers) |
| `QDO.API` | 983KB | REST API |
| `QDO.Application.APIHost` | 371KB | API host |
| `QDO.Application.MiddleTier` | 2.3MB | Middle tier host |
| `QDO.Application.Web` | 58MB | Web application host |
| `QDO.Batch` | 85KB | Minimal batch code |
| `QDO.DivisionOrder` | 39KB | Core DO logic library |
| `QDO.Events` | 27KB | Domain events |
| `QDO.DocumentManagement.EventHandlers` | 116KB | Document events |
| `QDO.ClassicGUI` / `QDO.ClassicBatch` / `QDO.Database` | 0 | **Empty** — shared with QRA |

### QCA — Cost Accounting (12 repos)

| Repo | Size | Purpose |
|------|------|---------|
| `QCA.ClassicGUI` | 2.1MB | WinForms — 115 screens, 7 modules |
| `QCA.Web` | 9.5MB | Modern web service layer |
| `QCA.Batch` | 570KB | .NET batch processes |
| `QCA.ClassicBatch` | 3.7MB | Legacy batch code |
| `QCA.Database` | 4.7MB | SQL schema |
| `QCA.API.Specs` | 2.7MB | API specs |
| `QCA.Application.APIHost` | 408KB | API host |
| `QCA.Application.MiddleTier` | 2.4MB | Middle tier |
| `QCA.Application.Web` | 61MB | Web app host |
| `QCA.Events` | 48KB | Domain events |
| `QCA.DocumentManagement.EventHandlers` | 174KB | Document events |
| `QCA.DesignStudio.PackageSource` | 63KB | Design studio |

### QCFS — Consolidated Financial System (13 repos)

| Repo | Size | Purpose |
|------|------|---------|
| `QCFS.ClassicGUI` | 4.3MB | WinForms — 438+ files, 15 projects, 6 modules |
| `QCFS.Web` | 26MB | Modern web service layer |
| `QCFS.Batch` | 1MB | .NET batch processes |
| `QCFS.Database` | 5.3MB | SQL schema |
| `QCFS.API` | 3.3MB | REST API |
| `QCFS.API.Specs` | 2.6MB | API specs |
| `QCFS.Application.APIHost` | 387KB | API host |
| `QCFS.Application.MiddleTier` | 2.5MB | Middle tier |
| `QCFS.Application.Web` | 59MB | Web app host |
| `QCFS.Events` | 47KB | Domain events |
| `QCFS.Messaging.Publisher` | 70KB | Event publishing |
| `QCFS.DocumentManagement.EventHandlers` | 204KB | Document events |
| `QCFS.DesignStudio.PackageSource` | 68KB | Design studio |

### Shared Infrastructure (12 repos)

| Repo | Size | Purpose |
|------|------|---------|
| `Application.ClassicGUI` | 4.2MB | **Shared WinForms framework** — base classes, utilities |
| `Application.QPEC` | 12MB | **Process Execution Controller** — batch orchestrator |
| `Application.MiddleTier` | 514KB | Shared middle tier host |
| `Application.MaintenanceApp` | 618KB | System maintenance utility |
| `Database` | 32MB | **Shared database** — cross-product schema |
| `Metadata` | 4.2GB | **QARCH framework** — screen registration, picks, security |
| `Shared.Batch` | 8.5MB | Shared batch framework + integration processes |
| `Shared.ClassicBatch` | 588KB | Legacy shared batch code |
| `Shared.ClassicGUI` | 2.5MB | Shared ClassicGUI components |
| `Shared.Web` | 32MB | Shared web framework |
| `Reports` | 58MB | Reporting templates (SSRS/Crystal) |
| `Tools` | 1MB | Development and deployment tools |
| `Help` | 53MB | Online help system |
| `CheckSignature` | 70KB | Check signature utility |

---

## Common Architectural Patterns

### ClassicGUI (WinForms) Patterns

All four products share these patterns via the `Application.ClassicGUI` shared framework:

| Pattern | Base Class | Example | Description |
|---------|-----------|---------|-------------|
| **Filter-Detail** | `QFrmBaseFilterDetailQra` | VL025, TS005 | Query panel + editable results grid |
| **Master-Detail** | `QFrmBaseMasterDetail` | ML006, JE205 | Parent grid + child grid(s) |
| **Single-Grid** | `QFrmBaseGrid` | VA005, JE015 | Standalone editable grid |
| **Form-Based** | `QFrmBase` / `frmMaintBase` | TR010, DO006 | Field-based maintenance form |

### File Naming Convention (QRA ClassicGUI)
```
QFrm{ScreenName}.cs          — Form logic (business rules, event handlers)
QFrm{ScreenName}.Designer.cs — Visual layout (controls, grids, filters)
QFrm{ScreenName}.resx        — Resources (strings, icons)
```

### File Naming Convention (QCA/QCFS ClassicGUI)
```
frm{ScreenName}.cs            — Form logic
frm{ScreenName}.Designer.cs   — Visual layout
```

### Business Rules Pattern
```csharp
protected override void InitializeBusinessRules() {
    BusinessRules.Add("RULE_NAME", new QBusinessRule<DataRow>(
        RuleType, Trigger, RuleDelegate, Columns));
}
```

### Data Access Pattern
```csharp
(QSqlStatementKey)"SELECT_TABLE_NAME"  // SQL key reference
QFrmBaseFilterDetailQra.OverrideMoreAllRowSize = 500  // Pagination
```

### Service Layer Pattern (Web repos: QRA.Web, QDO.Web, QCA.Web, QCFS.Web)
```
CoreInterface → DataContracts → ServiceInterface → ServiceCore → ServiceClient → Web.Controllers → Web
```

---

## Cross-Module Data Flows

### QRA Month-End Close Workflow
```
Production Data (VA005/VA015)
     ↓ VACALC batch
Revenue Distribution (VL/RD)
     ↓ DRI → MI Links (VL025) → Manual Input (VL031) → Statements (ML006/DR022)
     ↓ DRIPROC batch → Revenue results (VL050, VL100)
     ↓ PPN generation → DOI updates
Tax Calculation (TS/TV/TR)
     ↓ TAXJEPOST batch
Journal Entry Posting (JE100/JE205)
     ↓ JEPOSTONLY batch → GL postings
     ↓ GL MTD/History (JE110/JE111)
Check Write (CW005/CW010-CW020)
     ↓ BKRVNU batch → Check generation
     ↓ 1099 processing → Escheat/Escrow
```

### Cross-Product Integration Points
```
QRA (Revenue) ──→ QCFS (GL) : JE posting feeds general ledger
QRA (Revenue) ──→ QDO (DOI) : Revenue distribution updates DOI accounting
QRA (Tax)     ──→ QCFS (AP) : Tax payments flow to accounts payable
QCA (JIB)     ──→ QCFS (GL) : JIB billing feeds general ledger
QCA (JIB)     ──→ QCFS (AR) : JIB amounts feed accounts receivable
All products  ──→ Metadata  : QARCH_SEC_* security, QARCH_CNFG_* config
All products  ──→ QPEC      : Batch process orchestration
```

---

## Reference File Index

| File | Content | When to Read |
|------|---------|-------------|
| `references/qra-modules.md` | QRA ClassicGUI: all 18 module assemblies, form→class→table→rules mappings | When analyzing QRA screens for migration |
| `references/qdo-qca-qcfs.md` | QDO (9 modules), QCA (7 modules, 115 screens), QCFS (6 modules, 438 files) | When analyzing non-QRA products |
| `references/batch-and-qpec.md` | 11 QRA batch DLLs, QPEC architecture, ClassicBatch, Shared.Batch | When analyzing batch lifecycles |
| `references/shared-infrastructure.md` | Metadata (4.2GB), Database (32MB), Shared.Web, Shared.ClassicGUI | When analyzing framework/security/config |

---

## Status & Evolution

This skill is a **living reference** — it captures what has been explored so far and will be
updated as we deeper-dive into specific modules. Current depth levels:

| Area | Depth | Notes |
|------|-------|-------|
| QRA.VL (DRI) | ██████████ Deep | Full source code read, all business rules cataloged |
| QRA.TS/TR (Tax/Reg) | ████████░░ Good | Feature 1786981 stories written, source code referenced |
| QRA.JE (Journal Entry) | ████░░░░░░ Module map | Forms listed, business rules not yet cataloged |
| QRA.CW (Check Write) | ████░░░░░░ Module map | Forms listed, not yet source-read |
| QRA.VA/MF/DO | ███░░░░░░░ Directory | File listing done, forms not yet mapped |
| QDO.Web | ███░░░░░░░ Structure | Layered architecture identified |
| QCA | ████████░░ Good | CLAUDE.md provides 7-module breakdown with 115 screens |
| QCFS | ████████░░ Good | CLAUDE.md provides 6-module breakdown with 438 files |
| QRA.Batch | ████░░░░░░ DLL map | 11 process DLLs identified, not yet source-read |
| QPEC | ███░░░░░░░ Structure | Config files identified, execution model not yet traced |
| Metadata | ██░░░░░░░░ Identified | 4.2GB repo identified, QARCH tables referenced but not explored |
| Shared.Batch | ███░░░░░░░ Structure | Integration processes listed, not yet source-read |
