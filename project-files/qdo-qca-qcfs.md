# QDO, QCA, and QCFS — Product Reference

---

## QDO — Division Orders

### Architecture

QDO is architecturally unique: it has **no separate ClassicGUI or Database repo**. QDO screens
live inside `QRA.ClassicGUI` under the `Quorum.Upstream.QRA.DO` module (157 files). QDO has
its own modern Web repo (27MB) with a full service layer.

### QDO.Web Layer (27MB)

```
Quorum.QDO.CoreInterface/       — Interface definitions
Quorum.QDO.DQS.DataSource/      — Data Query Service data sources
Quorum.QDO.DQS.QueryView/       — DQS query view definitions
Quorum.QDO.Events/              — Domain events
Quorum.QDO.ServiceClient/       — Service client proxies
Quorum.QDO.ServiceCore/         — Business logic implementation
Quorum.QDO.ServiceInterface/    — Service interfaces
Quorum.QDO.Validations/         — Entity-level validations
Quorum.QDO.Web.Controllers/     — MVC controllers
Quorum.QDO.Web/                 — Views, ViewModels, Scripts
Quorum.QDO.Tests/               — Unit/integration tests
```

### QDO Screens (via QRA.ClassicGUI DO module)

| Screen | Name | Source Form | Web Status |
|--------|------|-------------|------------|
| DO004 | DOI Header Query | Classic | POC ~55% |
| DO005 | DOI Query | Web | POC ~75% |
| DO006 | DOI Setup | Web | POC ~65% |
| DO016 | MEG Setup | Web | POC ~90% |
| DO105 | DOI History | Web | — |
| MG004 | Market Group Header Query | Classic | POC ~60% |
| MG005 | Market Group Query | Classic | POC ~60% |
| MG006 | Market Group | Web | POC ~75% |
| BA005 | Business Associate | Web | — |

### QDO Key Tables
- `DONL_DO_HDR` — Division Order headers
- `DONL_DO_DTL` — Division Order details (interest ownership)
- `DONL_DO_APPROVAL` — DOI approval tracking
- Market group and MEG tables

### QDO.API (983KB)
Separate REST API with endpoints for DOI CRUD, market groups, and approval workflows.

### QDO Batch (85KB)
Minimal — most DOI processing is triggered from QRA batch processes (QPDllDivisionOrder).

---

## QCA — Cost Accounting

### Source: CLAUDE.md in QCA.ClassicGUI repo (comprehensive documentation)

### Architecture
- **Language**: C# (.NET Framework), Windows Forms
- **Pattern**: Smart client (thick client) with direct database access, MDI
- **Screens**: 115 screens across 7 functional modules
- **Database**: `CORE_DEVA1UPS_QCA` (primary), `CORE_DEVA1UPS_QFC` (shared)

### Module Breakdown (7 modules)

#### 1. General Ledger (GL) — 10 screens
Chart of accounts, account maintenance, journal entries.
- `frmChartAcct` — Chart of Accounts
- `frmGLAcct` — GL Account Maintenance
- `frmGLJEHdr` — GL Journal Entry Header

#### 2. Joint Interest Billing (JIB) — 22 screens (largest)
Billing header/detail, AFE, property allocation, expense processing.
- `frmJIBHdr` — JIB Billing Header
- `frmJIBAFE` — Authorization for Expenditure
- `frmJIBPropAllocCur` — Property Allocation Current
- `frmJIBExpRateCur` — Expense Rate Current

#### 3. Open Invoice — 8 screens
Invoice processing, accounts receivable, cash receipts.
- `frmOpenInvHdr` — Open Invoice Header
- `frmARCustInv` — AR Customer Invoice
- `frmCashReceiptHdr` — Cash Receipt Header

#### 4. Revenue — 12 screens
Revenue accounting, owner relations, production allocation.
- `frmOwnerRev` — Owner Revenue
- `frmOwnerRevAdj` — Owner Revenue Adjustment
- `frmRevProdAlloc` — Revenue Production Allocation

#### 5. Setup — 40 screens (largest module)
System configuration, business units, entities, codes, rates.
- `frmBusUnit` — Business Unit
- `frmEntity` — Entity Maintenance
- `frmAcctPeriod` — Accounting Period
- `frmProperty` — Property

#### 6. Subledger — 14 screens
Transaction processing, vouchers, purchase orders.
- `frmSLTrans` — Subledger Transaction
- `frmSLVoucherHdr` — Subledger Voucher Header
- `frmSLPOHdr` — Subledger Purchase Order Header

#### 7. Timekeeping — 9 screens
Time entry, employee management, pay rates.
- `frmTimeHdr` — Time Entry Header
- `frmEmployee` — Employee Maintenance
- `frmPayRate` — Pay Rate Maintenance

### QCA Screen Patterns
- **Base Classes**: `frmMaintBase` (CRUD), `frmAdvSearchBase` (search), `frmBase` (common)
- **Metadata**: 99 of 115 screens (86%) registered in `QARCH_ENTITY_CTRL_PROPERTIES`
- **Naming**: `SHORT_NAME` → `LONG_NAME` mapping (e.g., `QCA_JIB_HDR` → "JIB Billing Header")

### QCA Batch Processes
- `QCA.Batch` (570KB) — .NET batch processes
- `QCA.ClassicBatch` (3.7MB) — Legacy batch code
- Integration with QPEC for scheduling

### QCA → QCFS Integration
- JIB billing feeds GL journal entries
- JIB amounts feed AR (accounts receivable)
- Revenue allocations feed GL postings

---

## QCFS — Consolidated Financial System

### Source: CLAUDE.md in QCFS.ClassicGUI repo (comprehensive documentation)

### Architecture
- **Language**: C# (.NET Framework), Windows Forms
- **Pattern**: Three-tier — Forms → Middle Tier (WCF/Web API) → SQL Server
- **Files**: 438+ files across 15 projects
- **Database**: `CORE_DEVA1UPS_QCFS` (primary)

### Module Breakdown (6 financial modules + 9 supporting projects)

#### Financial Modules

##### 1. Accounts Payable (AP) — `Quorum.QCFS.AP`
Vendor invoice entry, voucher creation/approval, payment processing, AP aging.
- Voucher management with approval workflow
- Check printing and EFT/ACH payment generation
- Vendor master data and 1099 processing

**Key workflow**: Receive invoice → Create voucher → Enter distributions → Approve → Schedule payment → Generate check/EFT → Post to GL

##### 2. Accounts Receivable (AR) — `Quorum.QCFS.AR`
Customer invoicing, billing distribution, payment receipts, AR aging/collections.
- Invoice entry and recurring invoice processing
- Cash receipt entry and payment application
- Lockbox processing, statement generation

**Key workflow**: Create invoice → Enter line items → Print/distribute → Receive payment → Apply to invoices → Post to GL

##### 3. General Ledger (GL) — `Quorum.QCFS.GL`
Journal entry creation, chart of accounts, account inquiry, period close.
- Manual and recurring journal entries
- Account hierarchy and budget entry
- Period open/close processing

**Key workflow**: Review transactions → Create adjustments → Run allocations → Generate trial balance → Review financials → Close period

##### 4. Bank Reconciliation (BR) — `Quorum.QCFS.BR`
Statement import, transaction matching, reconciliation processing.
- Multi-format bank statement import
- Automated + manual matching
- Adjustment entry creation

##### 5. Fixed Asset Management (MF) — `Quorum.QCFS.MF`
Asset tracking, depreciation calculations, transfers/disposals.
- Multiple depreciation methods
- Book vs. tax depreciation
- Asset location tracking

##### 6. System Management (SM) — `Quorum.QCFS.SM`
Configuration, user management, security, audit trails.
- Company setup and parameters
- Role-based permissions
- Workflow configuration

#### Supporting Projects (9)

| Project | Purpose |
|---------|---------|
| `Quorum.QCFS.UpstreamGUI` | Main application shell, menu, navigation, startup |
| `Quorum.QCFS.Shared` | Shared business logic, data access, utilities |
| `Quorum.QCFS.UserControls` | Reusable UI controls (grids, lookups, date pickers) |
| `Quorum.QCFS.UtilGUI` | UI utilities, common dialogs, form helpers |
| `Quorum.QCFS.BSGUI` | Business Service GUI — service invocation, progress tracking |
| `Quorum.QCFS.BSMGUI` | Business Service Manager — monitoring, job management |
| `Quorum.QCFS.ExternalInterface` | External system integration, import/export, file handlers |
| `Quorum.QCFS.QReports` | Report viewer, parameter dialogs, Crystal/SSRS integration |
| `Quorum.QCFS.QUpstreamRuleValidator` | Business rule and data validation framework |

### QCFS Key Design Patterns
- **Master-Detail**: Master grid → Detail form → Drill-down → Save/Cancel
- **Lookup**: Popup forms, quick search, favorites, recently used
- **Batch Processing**: Select records → Preview → Execute → Review results
- **Document Management**: Attach docs to transactions, version control, workflow

### QCFS Key Concepts
- **Voucher**: AP document representing vendor invoice
- **Distribution**: Allocation of amount to GL accounts
- **Posting**: Moving transactions from subledger to GL
- **Period Close**: End of accounting period process
- **Business Unit**: Organizational unit for financial reporting

### QCFS Integration Points
- Receives JIB/revenue data from QCA
- Receives JE postings from QRA
- Receives tax payment data from QRA
- Sends GL balances to reporting/consolidation
- Bank reconciliation integrates with check write data from QRA

---

## Cross-Product Comparison

| Aspect | QRA | QDO | QCA | QCFS |
|--------|-----|-----|-----|------|
| **ClassicGUI** | Own repo (1,566 files) | Shared with QRA (DO module, 157 files) | Own repo (115 screens) | Own repo (438 files) |
| **Web Repo** | 14MB (MVC) | 27MB (MVC) | 9.5MB | 26MB |
| **Database** | Own (11MB) | Shared with QRA (0 bytes) | Own (4.7MB) | Own (5.3MB) |
| **Batch** | 80MB (11 DLLs) | 85KB (minimal) | 570KB | 1MB |
| **ClassicBatch** | 27MB | 0 (shared) | 3.7MB | — |
| **Form prefix** | QFrm{Name} | QFrm{Name} (in QRA.DO) | frm{Name} | frm{Name} |
| **Base classes** | QFrmBase* (QARCH) | QFrmBase* (QARCH) | frmMaintBase, frmBase | frmBase, frmMaintBase |
| **Security** | QARCH_SEC_* | QARCH_SEC_* | QARCH metadata | QARCH metadata |

---

## TO EXPLORE NEXT

1. [ ] Read QCA.ClassicGUI `.github/Quorum.Upstream.QCA.ClassicGUI_Summary.md` for full 115-screen inventory
2. [ ] Read QCFS.ClassicGUI `.github/Quorum.Upstream.QCFS.ClassicGUI_Summary.md` for full screen inventory
3. [ ] Map QDO.Web service layer to DO module forms (which service endpoints serve which screens)
4. [ ] Trace QCA→QCFS integration: JIB billing → GL postings data flow
5. [ ] Trace QRA→QCFS integration: JE posting → GL, Tax → AP data flows
