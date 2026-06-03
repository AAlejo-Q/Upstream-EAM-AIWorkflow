# EAM Project Context

## Overview

**Enterprise Accounting Modernization (EAM)** — Migrating Quorum's MyQuorum Upstream Revenue
Accounting (QRA) desktop WinForms/Classic GUI application (hosted via QCloud/Citrix) to a modern
Blazor Server web application on Azure.

**Target Tech Stack**: .NET 8+, Blazor Server, API-first architecture, EF Core + Dapper,
Azure SQL, MCP tool exposure for AI-enabled agentic month-end close.

**ADO Organization**: QuorumSoftware (dev.azure.com/QuorumSoftware)
**ADO Projects**: "Quorum" (new work items), "QuorumSoftware" (repos + legacy items)
**Key People**: Sarah Paul (Product Management), Ryan Westenhover (Engineering)

---

## ADO Work Item Hierarchy

### Parent Epic: 1786997 — "EAM - 6 Epics" (Quorum project, New state, Priority 2)

Related to closed epic 1779138 in QuorumSoftware project (original, restructured Jan-Mar 2026).

### Six Sub-Epics

| # | Epic | ID | Features | Target | Priority |
|---|------|---------|----------|--------|----------|
| 1 | Desktop to Web QRA MVP | 1786991 | 21 | Q4 2026 | 5 |
| 2 | Cloud Infrastructure | 1786992 | 16 | Q4 2026 | 3 |
| 3 | Private Cloud | 1786993 | 0 | Q4 2027 | 8 |
| 4 | Full Suite (850+ screens) | 1786994 | 0 | Q2 2027 | 6 |
| 5 | Test Coverage | 1786995 | 0 | Q4 2027 | 4 |
| 6 | Batch Modernization | 1786996 | 0 | Q4 2027 | 7 |

### Epic 1 Features (21 total, 0 user stories exist anywhere)

**Foundation (4)**:
- 1786989: Web App Framework (Blazor Server scaffold, auth, routing)
- 1786990: Data Access Layer (EF Core + Dapper, connection management)
- 1786977: Security/Access Control (QARCH_SEC_* tables)
- 1786983: Metadata/Config (QARCH_CNFG_* tables)

**Screen Migration Groups (13)**:
- 1789636: 00-Pilot Robot Training (P2)
- 1786976: 01-Screen Inventory
- 1786987: 02-Master Data/Production (11 screens)
- 1786972: 03-Direct Revenue Input (4 screens) — DRI focus
- 1786980: 04-Revenue Distribution (11 screens)
- 1786978: 05-Journal Entry (19 screens, largest group)
- 1786986: 06-Check Write (7 screens)
- 1786981: 07-Tax/Reg (3 screens)
- 1786985: 08-eSuite/BA (5 screens, all P3)
- 1786975: 09-DOI (14 screens, mostly P3)
- 1786984: 10-Admin Maintenance
- 1786974: 11-Admin Security
- 1789563: 12-Batch/Report (5 screens)

**Cross-cutting (4)**:
- 1786973: MCP Tools
- 1786982: E2E Tests
- 1786979: Performance Testing
- 1786988: Training/Docs

### Epic 2 Features (16 total)

Infrastructure features: Azure Subscription, SQL Migration, IaC Templates, CI/CD (Web + DB),
Monitoring/Logging, Customer Monitoring, Secrets/Key Vault, Network Security, Security Scanning,
Disaster Recovery, Customer Migration Tooling, QPEC Containerization, Ops Training, Pilot Validation,
Local Dev Environment.

---

## Codebase Architecture

### Repository Structure (QuorumSoftware ADO project)

Each product (QRA, QDO, QCA, QCFS) has repos following: `Quorum.Upstream.{Product}.{Layer}`

**Key QRA Repos**:
- `Quorum.Upstream.QRA.ClassicGUI` (1,566 files) — WinForms source, THE migration source
- `Quorum.Upstream.QRA.Web` (685 files) — Existing ASP.NET MVC web app
- `Quorum.Upstream.QRA.Application.MiddleTier` (34 files) — Deployment host
- `Quorum.Upstream.QRA.Application.QPEC` — Process execution controller
- `Quorum.Upstream.QRA.Database` (11MB) — Schema definitions
- `Quorum.Upstream.QRA.Batch` (80MB) — Legacy batch code
- `Quorum.Upstream.Application.ClassicGUI` — Shared framework

### ClassicGUI Module Prefixes → Screen Codes

| Prefix | Files | Maps To |
|--------|-------|---------|
| VL | 108 | DRI, Revenue Distribution, DOI Accounting |
| JE | 99 | Journal Entry |
| CW | 86 | Check Write |
| VA | 93 | Volumes / Master Data |
| AD | 92 | Acquisitions |
| MF | 190 | Code Tables / Master Files |
| DO | 162 | Division of Interest |
| TS | 135 | Tax/Severance |
| TV | 147 | Tax Views |

### Web App Architecture (QRA.Web)

Layered: `CoreInterface` → `DataContracts` → `ServiceInterface` → `ServiceCore` → `ServiceClient` → `Web.Controllers` → `Web` (Views/ViewModels/Scripts)

Also: QPAMapper/QPAModel (allocation), SAPProxies (SAP integration), Validations (per entity)

### Key Patterns in Legacy Code

- **Forms**: `QFrm{Name}.cs` + `QFrm{Name}.Designer.cs` + `.resx`
- **Business Rules**: Registered in `InitializeBusinessRules()` with named IDs
- **Data Access**: SQL statement keys like `(QSqlStatementKey)"SELECT_TABLE_NAME"`
- **Filter-Detail**: `QFrmBaseFilterDetailQra` pattern for query+edit screens
- **Master-Detail**: Parent grid + child grid with `QueryChildGrid()` calls
- **Context Data**: Screen linking via `QContextDataMgr` for navigation between screens

---

## SAFe Readiness Gaps (as of March 2026)

1. **No user stories exist** anywhere in the ADO hierarchy
2. Epics 3-6 have zero child features
3. ~8 of 37 features have acceptance criteria
4. No iteration/sprint assignments
5. Dependencies in text only, not formal ADO links
6. A Blazor POC exists but isn't reflected in ADO work items
