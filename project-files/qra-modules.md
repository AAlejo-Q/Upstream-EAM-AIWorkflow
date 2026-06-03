# QRA ClassicGUI — Module Reference

## Repo: `Quorum.Upstream.QRA.ClassicGUI` (1,566 files, 15MB)

The QRA ClassicGUI is organized into 18 module assemblies plus a Shared assembly. Each assembly
is a .NET project (`.csproj`) under the solution root, following the naming convention
`Quorum.Upstream.QRA.{Prefix}`.

---

## Module Catalog

| # | Assembly | Prefix | Files | Domain | Migration Feature |
|---|----------|--------|-------|--------|-------------------|
| 1 | QRA.VL | VL | 103 | DRI, Revenue Distribution, DOI Accounting | 1786972 (DRI), 1786980 (Rev Dist) |
| 2 | QRA.JE | JE | 95 | Journal Entry, GL Posting, Reversals, Subledgers | 1786978 |
| 3 | QRA.MF | MF | 184 | Master Files, Code Tables, Reference Data | 1786987 (Master Data) |
| 4 | QRA.DO | DO | 157 | Division of Interest (also serves QDO product) | 1786975 |
| 5 | QRA.TV | TV | 142 | Tax Views — Tax Query/Analysis | 1786981 (Tax/Reg) |
| 6 | QRA.TS | TS | 131 | Tax/Severance — Tax Input, Common Maint | 1786981 (Tax/Reg) |
| 7 | QRA.VA | VA | 88 | Volumes/Production — Gas, Liquid, Results | 1786987 (Master Data) |
| 8 | QRA.AD | AD | 87 | Acquisitions | — |
| 9 | QRA.CW | CW | 82 | Check Write — Checks, Withholding, Escheat | 1786986 |
| 10 | QRA.PO | PO | 59 | Pay Orders | — |
| 11 | QRA.CI | CI | 55 | Contract Interface | — |
| 12 | QRA.CA | CA | 49 | Contractual Allocation | — |
| 13 | QRA.TR | TR | 49 | MMS Royalty Reporting | 1786981 (Tax/Reg) |
| 14 | QRA.PN | PN | 34 | PPN (Pending Payment Notification) | 1786980 (Rev Dist) |
| 15 | QRA.GB | GB | 22 | Gas Balancing | 1789563 (Batch/Report) |
| 16 | QRA.FL | FL | 22 | Flow/Allocation | — |
| 17 | QRA.SP | SP | 19 | Well Completion/Property Cross-references | 1786987 |
| 18 | QRA.RD | RD | 19 | Revenue Distribution Results | 1786980 |
| 19 | QRA.TO | TO | 19 | Takeout | — |
| 20 | QRA.Shared | — | 46 | Shared Business Rules, Eventing, ID Classes | Foundation |

---

## Module Deep Dives

### VL — DRI / Revenue Distribution / DOI Accounting (103 files) ██████████ DEEP

**Fully explored.** See `dri-module.md` in the `eam-work-items` skill for complete reference.

Key forms: QFrmMILinkMaintenance (VL025, 22 business rules), QFrmVLManualInput/QFrmVLManualInputHTD (VL031),
QFrmDRIStatementDetailUpload (VL032), QFrmDRIStatementGroup (ML006), QFrmDRIStatementDetail (DR022),
QFrmMILinkMassEndDate (VL026), QFrmDOIAccountingRules (VL006), QFrmDOIAccountingDataQuery (VL005),
QFrmRevenueSelectionSubmittal (VL100), QFrmFullRevenueResults (VL050).

Primary tables: RONL_MI_LINK_EFF_DT, RONL_MANL_INPUT, RONL_MI_STMNT_GRP/HDR/DETAIL/TAX/MKT_CST,
RXRF_MI_RMITR_DFLT, RXRF_MI_SOD, RTRN_MI_STMNT_INPUT_SNAPSHOT.

---

### JE — Journal Entry (95 files) ████░░░░░░ MODULE MAP

**Forms identified, business rules not yet cataloged.**

| Screen Code | Form Class | Purpose |
|------------|-----------|---------|
| JE005 | QFrmGroupTypeMaintenance | JE Group Type maintenance |
| JE010 | QFrmAcctSetupMaintenance | Account Setup maintenance |
| JE011 | QFrmAcctSubledgerXref | Account/Subledger cross-reference |
| JE015 | QFrmAcctGroupMaintenance | Account Group maintenance |
| JE020 | QFrmAcctAcctGroupXref | Account/Account Group cross-reference |
| JE025 | QFrmAcctGroupTemplate | Account Group Template |
| JE030 | QFrmSubledgerMaintenanceDetail | Subledger Maintenance |
| JE100 | QFrmJEPostingControl | JE Post Selection/Submittal — **key batch trigger** |
| JE102 | QFrmAcctMnthRollDate | Accounting Month Roll Date |
| JE110 | QFrmGeneralLedgerDetailQuery | GL MTD Detail Query |
| JE111 | QFrmGeneralLedgerDetailHistQuery | GL History Detail Query |
| JE205 | QFrmManualJEBatchMaintenance | Manual JE Batch Maintenance — **complex, 28 gaps** |
| JE206 | QFrmManualJEDetail | Manual JE Detail |
| JE207 | QFrmManualJETaxAdjCtgy | Manual JE Tax and Adjustment |
| JE210 | QFrmManualJEBatchApproval | Manual JE Batch Approval |
| JE305 | QFrmHistoryReversalBatchSelection | HIST Reversal Batch Selection |
| JE310 | QFrmHistoryReversalBatchExecution | HIST Reversal Batch Execution |
| JE315 | QFrmMTDReversalBatchSelection | MTD Reversal Batch Selection |
| JE320 | QFrmMTDReversalBatchExecution | MTD Reversal Batch Execution |
| — | QFrmAccountsReceivableTransferSelect | AR Transfer Selection |
| — | QFrmAcctRecvXferQuery | AR Transfer Query |
| — | QFrmCtrPtyRemitterXref | Counterparty Remitter cross-reference |
| — | QFrmFiscalYearMaintenance | Fiscal Year Maintenance |
| — | QFrmGeneralLedgerMTDSummaryQuery | GL MTD Summary Query |
| — | QFrmStatAcctXref | Statistical Account cross-reference |
| — | QFrmSubledgerDetailQuery | Subledger Detail Query |
| — | QFrmSubledgerReconciliation | Subledger Reconciliation |
| — | QFrmSummarySubledgerDetail | Summary Subledger Detail |
| — | QFrmSummarySubledgerQuery | Summary Subledger Query |
| — | QFrmManualJEBatchQuery | Manual JE Batch Query |
| — | QueuedProcessDO | Queued process data object |

**Key workflow**: Manual JE entry (JE205 → JE206 → JE207 → JE210 approval) then JE posting (JE100 → JEPOSTONLY batch → GL tables JE110/JE111).

**Batch trigger**: JE100 (QFrmJEPostingControl) launches JEPOSTONLY batch process.

---

### CW — Check Write (82 files) ████░░░░░░ MODULE MAP

| Screen Code | Form Class | Purpose |
|------------|-----------|---------|
| CW005 | QFrmCheckRegisterQryView | Check Register Query/View |
| CW010 | QFrmStateWithholdingTax | State Withholding Tax Maintenance |
| CW011 | QFrmStateBackupWithholdingTaxMaint | State Backup Withholding Tax |
| CW015 | QFrmForeignWithholdingTax | Non-Resident Alien Withholding (was: Foreign) |
| CW020 | QFrmVoidCheckMaint | Void Check Maintenance |
| EC005 | QFrmEscheatMaintenanceHistory | Escheat Maintenance/History |
| EC010 | QFrmEscheatProcessingRules | Escheat Processing Rules |
| — | QFrmCheckRegisterMaint | Check Register Maintenance |
| — | QFrmCheckStatusMaint | Check Status Maintenance |
| — | QFrm1099QueryMaint | 1099 Query/Maintenance |
| — | QFrm1099SumQueryMaint | 1099 Summary Query/Maintenance |
| — | QFrmAddUpdateTaxInput | Add/Update Tax Input |
| — | QFrmBulkCheckPrint | Bulk Check Print |
| — | QFrmCDRXRF_CW_CDEX_BAQuery | CW Check Distribution BA Query |
| — | QFrmEscheatOwnerDetail | Escheat Owner Detail |
| — | QFrmEscrowMaintenanceHistory | Escrow Maintenance/History |
| — | QFrmEscrowOwnerDetail | Escrow Owner Detail |
| — | QFrmEscrowOwnerProcessingRules | Escrow Owner Processing Rules |
| — | QFrmEscrowStateProcessingRules | Escrow State Processing Rules |
| — | QFrmInterestPaymentsQuery | Interest Payments Query |
| — | QFrmManualCheckApproval | Manual Check Approval |
| — | QFrmManualCheckDetail | Manual Check Detail |
| — | QFrmManualCheckHeader | Manual Check Header |
| — | QFrmManualCheckTaxes | Manual Check Taxes |
| — | QFrmPropDepletionInput | Property Depletion Input |
| — | QFrmReleasePpaPermissionCW | Release PPA Permission |

**Key workflow**: Revenue results → Check generation (BKRVNU batch) → Check register (CW005) → Withholding taxes (CW010/011/015) → Void checks (CW020) → Escheat processing (EC005/010) → 1099 reporting.

---

### VA — Volumes/Production (88 files) ███░░░░░░░ DIRECTORY

| Screen Code | Form Class | Purpose |
|------------|-----------|---------|
| VA005 | QFrmGasVolumes* | Gas Volumes entry |
| VA015 | QFrmLiquidVolumes* | Liquid Volumes entry |
| VA035 | QFrmVAResults* | Volume/Allocation Results |

*Exact class names pending source-read. Module also includes BusinessRules/ and Utility/ subdirectories.

**Key role**: VA is the entry point for production data. VACALC batch processes volume/allocation calculations that feed into revenue distribution.

---

### MF — Master Files / Code Tables (184 files) ███░░░░░░░ DIRECTORY

Largest QRA module. Contains code table editors, reference data maintenance, and shared
master file screens. Key screen: MF035 (QFrmCodeTableValueEditor) — the generic code
table editor used across the entire suite.

Has BusinessRules/, Utility/, and a nested Quorum/ subdirectory.

---

### DO — Division of Interest (157 files) ███░░░░░░░ DIRECTORY

Serves both QRA and QDO products. Key screens: DO004 (DOI Header Query), DO005 (DOI Query),
DO006 (DOI Setup), DO016 (MEG Setup), DO105 (DOI History), MG004-MG006 (Market Groups).

Has BusinessRules/ and Utility/ subdirectories.

---

### TS — Tax/Severance (131 files) ████████░░ GOOD

Forms include: QFrmTSTaxCommonQuery (TS005), QFrmTSTaxCommon (TS006).
See `FEATURE_1786981_TAX_REG_STORIES.md` in the project files for detailed story breakdown.
Primary table: TONL_TAX_INPUT + child tables (TAX_FREE, MKT_ADJ).

Has BusinessRules/ subdirectory with standalone validation classes.

---

### TV — Tax Views (142 files) ███░░░░░░░ DIRECTORY

Tax query and analysis screens. Has BusinessRules/ and Utility/ subdirectories.
Includes TX005 (State Tax Information Query), TX020/TX021 (State Tax Rate Query/Maintenance).

---

### TR — MMS Royalty Reporting (49 files) ████████░░ GOOD

Key screen: TR010 (QFrmMMSRoyaltyLeaseMaster) — Lease Master Data for MMS royalty.
Primary table: TONL_MMS_RYL_LSE_MST. See Feature 1786981 stories for detail.
Audit history writes to THIST_AUDIT3 (Audit Object ID: 55).

---

### Smaller Modules

| Module | Files | Key Forms / Purpose |
|--------|-------|---------------------|
| AD (Acquisitions) | 87 | Acquisition management screens |
| PO (Pay Orders) | 59 | Payment order processing |
| CI (Contract Interface) | 55 | Contract management, has BusinessRules/ |
| CA (Contractual Allocation) | 49 | Allocation processing, has BusinessRules/, Validations/ |
| PN (PPN) | 34 | Pending Payment Notification creation/query (PN020, PN025) |
| GB (Gas Balancing) | 22 | Gas balancing manual input (GB015) |
| FL (Flow/Allocation) | 22 | Flow/allocation screens |
| SP (Well Completion/Property) | 19 | Cross-reference screens (SP025), has Utility/ |
| RD (Revenue Distribution) | 19 | Revenue distribution results (RD010, RD030, RD031) |
| TO (Takeout) | 19 | Takeout processing |

---

### Shared Assembly (46 files)

`Quorum.Upstream.QRA.Shared` contains:
- **BusinessRules/** — Shared business rule classes used across modules
- **Eventing/** — Domain event definitions for QRA
- **IDClasses/** — Constants for pick IDs, code table IDs, security IDs, SQL statement keys

Key ID classes define constants like:
- `QPickIdQraMF`, `QPickIdQraDO`, `QPickIdQraVA` — Pick list identifiers
- `QCodeTableIdQraShared` — Code table identifiers
- `QSqlStatementKeyQra` — SQL statement key constants
- `QSecurityPrivilege` — Security permission constants

---

## Web Layer Architecture (QRA.Web, 685 files)

```
Quorum.QRA.CoreInterface/          — Interface definitions
Quorum.QRA.DataContracts/          — DTOs and data contracts
Quorum.QRA.ServiceInterface/       — Service interfaces
Quorum.QRA.ServiceCore/            — Business logic implementation
Quorum.QRA.ServiceClient/          — Service client proxies
Quorum.QRA.Web.Controllers/        — MVC controllers
Quorum.QRA.Web/                    — Views, ViewModels, Scripts
    QPAMapper/ QPAModel/           — Allocation mapping
    SAPProxies/                    — SAP integration
    Validations/                   — Per-entity validation
    Tests/ UnitTestsDeprecated/    — Test projects
```

---

## TO EXPLORE NEXT

Priority items for deepening this reference:
1. [ ] Read JE205 (QFrmManualJEBatchMaintenance) source — largest JE screen, 28 critical gaps
2. [ ] Read JE100 (QFrmJEPostingControl) — batch trigger, understand JEPOSTONLY parameters
3. [ ] Read CW005 (QFrmCheckRegisterQryView) — 28 critical gaps, complex check register
4. [ ] Map VA005/VA015 business rules — production data is upstream of everything
5. [ ] Explore MF035 (Code Table Value Editor) — generic editor used everywhere
6. [ ] Trace DO module screens to QDO.Web service layer
