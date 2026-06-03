# DRI Module Technical Reference

## Overview

The Direct Revenue Input (DRI) module handles third-party revenue statement entry, validation,
and processing in QRA. It is the highest-priority module for EAM migration, covering Feature
1786972 (03-Direct Revenue Input) and parts of Feature 1786980 (04-Revenue Distribution).

---

## DRI Screen-to-Code Mapping

| Screen | Code | Class | Table | Pattern |
|--------|------|-------|-------|---------|
| VL031 Query | Direct Revenue Input Query | QFrmVLManualInput | RONL_MANL_INPUT | Filter-Detail (read-only, 500-row pagination) |
| VL031 Maint | DRI Maintenance | QFrmVLManualInputHTD | RONL_MANL_INPUT | Editable detail (launched from query) |
| VL032 | DRI Upload | QFrmDRIStatementDetailUpload | — | Upload companion to VL031 |
| VL025 | DRI Link Maintenance | QFrmMILinkMaintenance | RONL_MI_LINK_EFF_DT | Filter-Detail with 22+ business rules |
| ML006 | SOD DRI Links / Statement Group | QFrmDRIStatementGroup | RONL_MI_STMNT_GRP → HDR | Master-detail with status workflow |
| DR022 | Statement Details | QFrmDRIStatementDetail | RONL_MI_STMNT_DETAIL + TAX + MKT_CST | Parent-child-grandchild |
| VL026 | MI Link Mass Changes | QFrmMILinkMassEndDate | — | Dialog: end-date, override, clone |

---

## DRI Database Tables

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| RONL_MI_LINK_EFF_DT | MI Link configuration (remitter→property mapping) | MANL_INPUT_ID, RMITR_NO, RMITR_SUB, MAJ_PROD_CD, PROP_NO, DO_TYPE_CD, DO_MAJ_PROD_CD, TIER, EFF_DT_FROM, EFF_DT_TO |
| RONL_MANL_INPUT | Manual input records (revenue data) | RMITR_NO, RMITR_SUB, MANL_INPUT_ID, PRDN_DT, PROD_CD |
| RONL_MI_STMNT_GRP | Statement groups | STMNT_GRP_NO, STMNT_GRP_AMT, STMNT_GRP_AMT_DRVD |
| RONL_MI_STMNT_HDR | Statement headers | STMNT_SEQ_NO, STMNT_NO, STMNT_DT, STMNT_AMT, STMNT_AMT_DRVD, STMNT_STATUS_CD |
| RONL_MI_STMNT_DETAIL | Statement line items | STMNT_SEQ_NO, LINE_NO, GROSS_VALUE, NET_AMOUNT |
| RONL_MI_STMNT_DETAIL_TAX | Severance tax per line | STMNT_SEQ_NO, LINE_NO, SEV_TAX_TYPE_CD, SEV_TAX_VAL, DEDUCT_REIMB_CD |
| RONL_MI_STMNT_DETAIL_MKT_CST | Marketing cost adjustments | STMNT_SEQ_NO, LINE_NO, MKT_CST_VAL, DEDUCT_REIMB_CD |
| RXRF_MI_RMITR_DFLT | Remitter default values | RMITR_NO, RMITR_SUB, STMNT_FLD_CD, DFLT_VALUE |
| RXRF_MI_SOD | MI-to-SOD cross-reference | Updated when MI Link effective dates change |
| RTRN_MI_STMNT_INPUT_SNAPSHOT | Staging for batch processing | STMNT_RUN_ID, STMNT_SEQ_NO |

---

## DRI Workflow (Month-End Close)

### Step 1: MI Link Setup (VL025)
Land/accounting teams configure remitter→property/DOI/tier/well mappings.
Records are effective-dated. This is the most complex DRI screen (22 business rules).

### Step 2: Direct Revenue Input (VL031/VL032)
Revenue accountants enter or upload statement data. VL031 = manual, VL032 = bulk upload.
Design decision: VL032 should be merged into VL031 in the web version.

### Step 3: Statement Group Management (ML006)
Statements organized into groups. Status lifecycle:
```
NEW → RFR (Ready for Review) → APV (Approved) → PRC (Processed)
```
- NEW: Initial state, fully editable
- RFR: Set via Links menu, ready for review
- APV: Requires Statement Amount = Derived Amount AND Group Amount = Derived Group Amount
- PRC: Read-only, processed by DRIPROC batch

### Step 4: Statement Detail Entry (DR022)
Line items within each statement. Data hierarchy:
```
RONL_MI_STMNT_HDR (header — 1 per statement)
  └── RONL_MI_STMNT_DETAIL (details — N per statement)
        ├── RONL_MI_STMNT_DETAIL_TAX (taxes — M per detail)
        └── RONL_MI_STMNT_DETAIL_MKT_CST (marketing — P per detail)
```

**Net Amount Validation** (critical business rule):
```
Net Amount = Gross Value
  + Σ(Tax amounts where Deduct/Reimb = 'R')
  - Σ(Tax amounts where Deduct/Reimb = 'D')
  + Σ(Marketing amounts where Deduct/Reimb = 'R')
  - Σ(Marketing amounts where Deduct/Reimb = 'D')
```
Validation uses Math.Round to 2 decimal places.

**Derived Amount**: Sum of all Net Amounts for the statement → updates header → updates group.

### Step 5: Batch Processing (DRIPROC)
From ML006, submit records via two modes:
- **Submit Checked**: All flagged records, uses StatementRunID = -1
- **Submit Highlighted**: Grid-selected records that must be APV status
  - Gets new StatementRunID via GetNextSeqNoImmediate
  - Stages to RTRN_MI_STMNT_INPUT_SNAPSHOT
  - Launches DRIPROC batch via QUpsUtility.LaunchBatchProcess

---

## VL025 Business Rules (22 registered)

| Rule ID | Type | Description |
|---------|------|-------------|
| CONVERT_MANL_INPUT_ID_TO_UPPER | Data | Force Manual Input ID to uppercase |
| VALIDATE_VLA_REQUIRED_FIELDS | Validation | Required fields differ based on VLA flag |
| VALIDATE_MI_EXISTS | Validation | Prevent delete if associated MIs exist |
| VALIDATE_DO_APPROVED | Validation | Division Order must be approved |
| VALIDATE_BA_APPROVED | Validation | Business Associate (remitter) must be approved |
| VALIDATE_INACTIVE_DATE | Validation | Check inactive date constraints |
| VALIDATE_ROYALTY_SELECTION_CODES | Validation | Validate royalty selection code combinations |
| VALIDATE_ACTIVE_APPV_SALES_CONTRACT | Validation | Contract must be active and approved |
| VALIDATE_MAJ_PROD_WELL_COMPL_XREF | Validation | Major product / well completion cross-ref valid |
| VALIDATE_Y_LEGACY_PPA_FLAG | Validation | Legacy PPA flag validation |
| VALIDATE_MARKET_REP_AND_GROUP | Validation | Contract market rep and group validation |
| VALIDATE_ACTIVE_REV_DO | Validation | Active revenue Division Order exists |
| VALIDATE_RMTR_NO_SUB | Validation | Remitter number/sub validation on insert |
| SET_GAS_STANDARD_PRESSURE | Auto-populate | Set gas standard pressure from property |
| POPULATE_DESKID_NO | Auto-populate | Auto-populate Desk ID number |
| EFF_DT_OVERLAP_CHECK | Validation | Effective date ranges must not overlap |
| PROMPT_TO_UPDATE_PRIOR_LEGACY_PPA_FLAG | Prompt | Prompt user about prior PPA flag |
| PPN_RV16_CHECK | Integration | Create PPN records on insert/modify/delete |
| MI_PROCESSED_CHECK | Validation | Warn if MI has been processed in date range |
| VALIDATE_ALLOCATION_FLAG | Validation | CA Basis requires Value Allocation flag |
| VALIDATE_VL_PROC_TYPE | Validation | Property VL Process Type cannot be 'CI' |
| UPDATE_RXRF_MI_SOD | Integration | Update SOD xref when effective dates change |
| ALLOCATION_CHECK | Validation | Allocation basis check |
| PREVENT_INF_CHANGES | Validation | Prevent changes to INF-type DOIs |

### VL025 BusinessRules/ Folder (12 standalone classes)

| Class | Purpose |
|-------|---------|
| QBusinessRuleMILinkCheckAllocationBasis | Allocation basis consistency |
| QBusinessRuleMILinkCheckInactiveDate | Inactive date validation |
| QBusinessRuleMILinkMajProdWellXRef | Major product / well cross-reference |
| QBusinessRuleMILinkPopulateDeskID | Auto-populate desk ID from mapping |
| QBusinessRuleMILinkRoySelection | Royalty selection code validation |
| QBusinessRuleMILinkSetPriorLegacyPPAFlag | Legacy PPA flag propagation |
| QBusinessRuleMILinkSetStandardPressureForGas | Gas standard pressure from property |
| QBusinessRuleMILinkValidateContract | Sales contract validation |
| QBusinessRuleMILinkValidateContractMarketRep | Contract market rep validation |
| QBusinessRuleMILinkValidateLegacyPPAFlag | Legacy PPA flag validation |
| QBusinessRuleMILinkValidatePrdnMeasurement | Production measurement validation |
| QBusinessRuleMILinkValidateRemitter | Remitter existence validation |

---

## DR022 (Statement Detail) Business Rules

| Rule ID | Trigger | Description |
|---------|---------|-------------|
| MAKE_APPROVED_READONLY | AfterQuery | Grids read-only when status = APV or PRC |
| DEFAULT_VALUES_FOR_RMITR | AfterQuery | Set field defaults from RXRF_MI_RMITR_DFLT |
| PRIMARY_KEY_CHANGE_LOGIC | StartingUpdate | Detect PK changes, convert update to insert |
| DETAIL_SET_DEFAULT_VALUES | BeforeInsert | Auto-assign line numbers, statement seq no |
| VALIDATE_SEV_TAX_TYPE | BeforeUpdate | Tax types must exist on DOI Accounting table |
| VALIDATE_BTU_FACTOR | BeforeUpdate | BTU factor within min/max from GCDE_MAJ_PROD |
| VALIDATE_NET_AMT | BeforeCommit | Net = Gross ± taxes ± marketing adjustments |
| CALCULATE_DRVD_AMT | BeforeUpdate | Sum of Net Amounts for statement |
| TIMESTAMP | BeforeUpdate | User ID + timestamp on every record |
| UPDATE_DRVD_AMT_FOR_GROUP | AfterUpdate | Rollup derived amount to group level |

---

## ML006 (Statement Group) Business Rules

| Rule ID | Trigger | Description |
|---------|---------|-------------|
| TIMESTAMP | BeforeUpdate | Standard timestamp |
| CALCULATE_DRVD_GRP_AMT | BeforeUpdate | Sum STMNT_AMT_DRVD for all headers in group |
| VALIDATE_BA_APPROVED | BeforeUpdate | Remitter BA must be approved |

### ML006 Status Transitions

- **NEW → RFR**: Via Links > "Ready For Review" menu (not PRC records)
- **RFR → APV**: Via Approve button (requires security: MI_STATEMENT_APPROVAL)
  - Validation: STMNT_AMT must equal STMNT_AMT_DRVD
  - Validation: STMNT_GRP_AMT must equal STMNT_GRP_AMT_DRVD
- **APV → PRC**: Via DRIPROC batch processing

### ML006 Submit/Process Logic

Two submit modes from toolbar:
1. **SubmitSelectedRecords**: Processes all checkbox-flagged records (STMNT_RUN_ID = -1)
2. **SubmitHighlightedRecords**: Processes grid-highlighted records
   - Validates all selected are APV status
   - Gets new STMNT_RUN_ID via sequence
   - Stages each to RTRN_MI_STMNT_INPUT_SNAPSHOT
   - Launches DRIPROC batch process

---

## VL025 Mass Changes (QFrmMILinkMassEndDate)

Three modes:
1. **Standard (End Date only)**: Sets new effective-to date on selected records
2. **End Date + Override**: Changes field values on existing records + sets end date
3. **End Date + Clone**: Creates new records with modified values + end-dates originals

Override fields available: Remitter, Major Product, Manual Input ID, Property, DO Type,
DO Major Product, Tier, Distribution Code, Well, Completion, Contract, Production MP,
Payment Level Override, Standard Pressure, Inactive Date, Calculate Sev Tax,
Legacy PPA, and 10 Royalty Selection Codes.

---

## POC Gap Summary (DRI Screens)

| Screen | Parity | Critical Gaps |
|--------|--------|---------------|
| VL025 | ~75-80% | Mass changes, PPA validation, contract market rep, allocation basis, auto-population, comments, granular security |
| VL031 | ~85% | Service layer gaps, code table dropdowns hardcoded, query filter validation |
| VL032 | ~75% | File upload/parsing engine, validation pipeline, combine with VL031 |
| VL100 | ~15% | Entire service layer, 6 tabs, batch integration, PPN, run ID tracking, 8 DB views, 4 staging tables |
| ML006 | ~65% | Mass insert, effective date logic, overlap validation, multiple contracts validation, PPN creation, Excel import/export |
| VL005 | ~30% | Detail dialog child grids (taxes/adjustments), navigation to VL006 |
| VL006 | ~30% | Entire maintenance screen, 15+ business rules, tax types detail grid, desk ID xref |

---

## Key Dependencies

- VL025 → VL031/VL032 (Links must exist before revenue input)
- ML006 → DR022 (Statement groups before statement details)
- VL100 → All VL screens (Revenue submission requires all upstream data)
- VL100 → QPEC/DRIPROC (Epic 2 Feature 2.13 - QPEC Containerization)
- All screens → Feature 1786989 (Web App Framework)
- All screens → Feature 1786990 (Data Access Layer)
- All screens → Feature 1786977 (Security/Access Control)
