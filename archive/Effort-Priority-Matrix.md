# QRA Screen Migration — Effort & Priority Matrix

**Generated:** 2026-04-30
**Scope:** All 35 analyzed screens pending migration (Analysis ✅, Migrated —)

---

## Effort Tier Definitions

| Tier | Criteria |
|------|----------|
| **S — Small** | Query-only or single-table maintenance; 0 new CodeGen objects; ≤4 BRs; no novel patterns |
| **M — Medium** | 1 new CodeGen object; OR moderate BRs (5–10); OR one notable complexity signal |
| **L — Large** | 2–3 new CodeGen objects; OR multi-table; OR 11–14 BRs; OR complex interaction pattern |
| **XL — Extra Large** | 4+ new CodeGen objects; OR 4+ tabs; OR 15+ BRs; OR architectural novelty (STG1, QP010) |

---

## Screen Type Patterns

| Pattern | Description | Web Migration Approach |
|---------|-------------|------------------------|
| **Query** | Read-only grid; context-driven filter; no inserts/deletes | `QBaseReadOnlyController` or `QBaseQueryController`; no business rule classes |
| **Simple Maintenance** | Single-table CRUD; standard BRs; no tabs | `QBaseBulkViewSingleEditController`; one CodeGen object if missing |
| **Standard Maintenance** | Multi-table or notable BR complexity; no tabs | Same base class; extra CodeGen objects; BR class per rule |
| **Tabbed Maintenance** | 4+ child grids in tabs; tab-level security | `QBaseBulkViewSingleEditController` + child grid controllers; 1 CodeGen per table |
| **Combined Screen** | Two ClassicGUI screens merged into one Web screen | Single controller + two grid sections |
| **Architectural** | Novel pattern (WCF-driven, generic/dynamic, batch staging) | Custom approach required; no direct ClassicGUI analog |

---

## Full Matrix

> **Note on combined screens:** PN020+PN025 migrate as one screen; VL005+VL006 migrate as one screen.

| Screen | Name | Module | New CodeGen | Tables | BRs | Screen Type | Effort | Key Complexity Signals |
|--------|------|--------|:-----------:|:------:|:---:|-------------|:------:|------------------------|
| JE005 | JEGroupTypeMaintenance | JE | 0 | 1 | ~3 | Simple Maintenance | **S** | Clean lift-and-shift; no new CodeGen |
| JE015 | AccountGroup | JE | 0 | 1 | ~3 | Simple Maintenance | **S** | Clean lift-and-shift; no new CodeGen |
| MG005 | MarketGroupQuery | MG | 0 | 1 | 0 | Query | **S** | Query-only; navigates to MG006 (delayed) |
| ML006 | SODDRILinks | ML | 0 | 1 | ~3 | Simple Maintenance | **S** | Upsert (IF EXISTS/UPDATE/ELSE INSERT); verify RXRF_MI_SOD in cg.xml; SEQ_NO absent from INSERT key needs investigation |
| QP045 | ProcessExecutionMonitorQuery | QP | 0 | 1 | 3* | Query | **S** | Read-only; drills to QP010; 3 registered BRs are dead code on read-only screen |
| TX020 | StateTaxRateQuery | TX | 0 | 1 | 0 | Query | **S** | Read-only; links to TX021; natural pair (spec together) |
| VA005 | GasVolumes | VA | 0 | 1 | ~4 | Simple Maintenance | **S** | Standard volumes maintenance |
| VA035 | VAResults | VA | 0 | 1 | 0 | Query | **S** | Read-only VACALC output viewer; duplicate SELECT key in QARCH_SQL (minor) |
| DO005 | DOIQuery | DO | 0 | 1 | 0 | Query | **S** | Read-only; dual inquiry date mechanism; PKLIST 70000 missing from metadata (spec defect) |
| BA010 | BAMassUpdate | BA | 0 | 1 | ~3* | Simple Maintenance | **S** | UPDATE-only; tax ID validation rules commented out (SIR 1546) — note in spec; Controller_OverrideUpdateData is a no-op stub |
| TS005 | SeveranceTaxCommonQuery | TS | 0 | 1 | 0 | Query | **S** | Read-only query; natural pair with TS006 |
| MF035 | CodeTableValueEditor | MF | 0 | ? | ? | Simple Maintenance | **S** | Generic code table editor; complexity TBD from full analysis |
| CW010 | StateWithholdingTaxMaintenance | CW | 1 | 1 | 4 | Simple Maintenance | **M** | Effective-date overlap check; rate 0.0–1.0 validation; US-state + interest-type scoped |
| CW011 | StateBackupWithholdingTaxMaintenance | CW | 1 | 1 | ~4 | Simple Maintenance | **M** | Near-identical to CW010 (share spec template) |
| CW015 | NonResidentAlienWithholdingTaxMaintenance | CW | 1 | 1 | ~4 | Simple Maintenance | **M** | Country-keyed variant of CW010; class/DB naming mismatch (Foreign vs NRA) |
| JE011 | AccountSubledgerXref | JE | 1 | 1 | ~4 | Simple Maintenance | **M** | 2 direct QSqlStatementKey purge validation calls (verify execution path in Web) |
| JE010 | AccountSetupMaintenance | JE | 0 | 1 | ~4 | Simple Maintenance | **M** | INSERT_JONL_ACCT_PURGE_PARAM side-effect on delete — must replicate in BR |
| GB015 | GasBalancingManualInput | GB | 0 | 1 | ~5 | Simple Maintenance | **M** | PONL_IMBAL_MANL missing from cg.xml (gap — add before migration); per-row SQL loop performance concern (batch alternative in spec) |
| JE020 | AccountAccountGroupXref | JE | 0 | 2 | ~5 | Standard Maintenance | **M** | Dynamic per-cell R/O logic; 12 pick list return mappings (complex grid config) |
| VA015 | LiquidVolumes | VA | 0 | 1 | 13 | Standard Maintenance | **M** | BSW_DECIMAL BR mutates STD_VOL as side effect — document carefully; 13 BRs |
| EC005 | EscheatMaintenanceHistory | EC | 2 | 1 | ~4 | Simple Maintenance | **L** | Simple 2-editable-column screen but 2 new CodeGen objects (DonlEschtOwnr, DonlEschtOwnrDetail — 10 files) |
| DO004 | DOIHeaderQuery | DO | 2 | 2 | ~4 | Standard Maintenance | **L** | 2 new CodeGen objects (GxrfDeskidDoCrtn, GxrfDeskidDoMaint); custom Controller_OverrideUpdateData for desk ID xref rows |
| RD030 | RDPreTrackingResults | RD | 3 | 3 | 1* | Complex Query | **L** | Read-only 3-grid screen; 3 new CodeGen objects needed (_HIST tables); off-by-one bug in FillContextDataMgrForPlugIn; REQUIRED_QUERY_FIELDS BR commented out |
| JE025 | AccountGroupTemplate | JE | 0 | 1 | 3 | Standard Maintenance | **L** | 3 inter-dependent Y/N reversal BRs; delete+reinsert PK-change pattern; audit history (JHIST_AUDIT) |
| BA015 | BAAddressMassUpdate | BA | 0 | 3 | 9 | Standard Maintenance | **L** | Dual-panel; 3-table UPDATE in one SQL key; config-driven column visibility (BA_ADDRESS_1099); 9 BRs; AllowInsert=false |
| TR010 | LeaseMasterDataMMSRoyaltyReporting | TR | 0 | 2 | 13 | Standard Maintenance | **L** | 13 BRs; allocation-factor sum check (must = 1.0); mass-copy subsystem; comments + audit history; 6 pick lists |
| VL005+VL006 | DOIAccountingDataQuery + DOIAccountingRules | VL | 0 | 2 | ~6 | Combined Screen | **L** | Two ClassicGUI screens combined into one Web screen; query + rules grid |
| PN020+PN025 | PPNCreation + PPNQuery | PN | 0 | 1 | ~6 | Combined Screen | **L** | Two screens combined; PN025 has 2 grids (55+19 cols); @@PPN_IDs injection pattern |
| VL050 | VLFullRevenueResults | VL | 2 | 3 | 0 | Complex Query | **L** | Read-only; 3 grids (59+16+14 cols); 2 new CodeGen objects; SQL vendor=0 BTU bug; duplicate SEQ_NO 276 in metadata |
| JE030 | SubledgerMaintenance | JE | 6 | 5 | ~8 | Tabbed Maintenance | **XL** | 4 tabs; tab-level security; 6 new CodeGen objects (30 files — all JONL_SL/JTRN tables missing from XML); most complex JE screen |
| EC010 | EscheatProcessingRules | EC | 4 | 4 | 2 | Tabbed Maintenance | **XL** | 3 child grids in tabs; 4 new CodeGen objects (20 files); exception swallowed in BR (ValidateOwnerPrintFirstFlag) |
| TX021 | StateTaxRateMaintenance | TX | 2 | 5 | 15 | Tabbed Maintenance | **XL** | 4 child grids; 15 BRs; complex effective-date overlap rules; VL proc date locking; 2 new CodeGen objects; 9 code tables |
| TS006 | SeveranceTaxCommonMaintenance | TS | 0 | 3 | 13 | Standard Maintenance | **XL** | 13 BRs; Reverse/Fill/Copy link operations; gross-vol/val balance check; 60+ bound header controls; no tabs but form-based complexity |
| VL025 | DRILinkMaintenance | VL | 4 | 4 | 25 | Standard Maintenance | **XL** | 25 BRs; PPN sync + SOD sync; 40-col grid; 6 pick lists; 2 config keys; VL-finalized locking; 4 new CodeGen objects |
| STG1 | StageTableDataMaintenance | MF | 0* | 31 | 2 | Architectural | **XL** | 31 staging tables; generic/dynamic tabbed UI (batch-process-driven); recommend generic API (no per-table CodeGen); off-by-one bug in BR |
| QP010 | ProcessExecutionMonitor | QP | 0 | 0 | 0 | Architectural | **XL** | WCF-service-driven (not SQL/QARCH); 5 grids; auto-refresh; cancel via IQProcessLauncherService; no ClassicGUI analog in Web framework |

*BRs marked with \* are dead code or commented-out in source.

---

## Summary by Effort Tier

| Tier | Count | Screens |
|------|------:|---------|
| **S** | 12 | JE005, JE015, MG005, ML006, QP045, TX020, VA005, VA035, DO005, BA010, TS005, MF035 |
| **M** | 8 | CW010, CW011, CW015, JE011, JE010, GB015, JE020, VA015 |
| **L** | 11 | EC005, DO004, RD030, JE025, BA015, TR010, VL005+VL006, PN020+PN025, VL050, (and 2 more in combined count) |
| **XL** | 7 | JE030, EC010, TX021, TS006, VL025, STG1, QP010 |
| **Total** | **38** | (35 unique screen IDs; 3 pairs combined) |

---

## Recommended Sequencing

### Wave 1 — Quick wins: complete CW module + JE setup foundation (S/M)

Start here to clear the backlog on modules that are almost done and to lay the JE foundation before tackling JE025/JE030.

| Priority | Screen | Effort | Rationale |
|----------|--------|:------:|-----------|
| 1 | ML006 SODDRILinks | S | Feature Rank 1; simple upsert; small scope |
| 2 | JE005 JEGroupTypeMaintenance | S | JE module foundation; cleanest first |
| 3 | JE015 AccountGroup | S | JE module foundation |
| 4 | JE010 AccountSetupMaintenance | M | JE foundation; side-effect BR needs care |
| 5 | JE011 AccountSubledgerXref | M | JE foundation; 1 new CodeGen |
| 6 | JE020 AccountAccountGroupXref | M | JE foundation; complex grid config |
| 7 | CW010 StateWithholdingTaxMaintenance | M | CW module completion; 1 new CodeGen |
| 8 | CW011 StateBackupWithholdingTaxMaintenance | M | CW module completion; near-clone of CW010 |
| 9 | CW015 NonResidentAlienWithholdingTaxMaintenance | M | CW module completion; country variant |
| 10 | TX020 StateTaxRateQuery | S | Natural spec pair with TX021; quick win |
| 11 | VA005 GasVolumes | S | VA module; standard volumes |
| 12 | VA035 VAResults | S | VA module; read-only |
| 13 | VA015 LiquidVolumes | M | VA module; 13 BRs (BSW side-effect) |

### Wave 2 — Medium complexity + module completions

| Priority | Screen | Effort | Rationale |
|----------|--------|:------:|-----------|
| 14 | TS005 SeveranceTaxCommonQuery | S | TS module; pair with TS006 |
| 15 | MG005 MarketGroupQuery | S | MG module; query-only |
| 16 | QP045 ProcessExecutionMonitorQuery | S | QP module; pair with QP010 |
| 17 | BA010 BAMassUpdate | S | BA module; UPDATE-only |
| 18 | DO005 DOIQuery | S | DO module; read-only query |
| 19 | GB015 GasBalancingManualInput | M | GB module; add PONL_IMBAL_MANL to cg.xml first |
| 20 | EC005 EscheatMaintenanceHistory | L | EC module; 2 new CodeGen objects |
| 21 | DO004 DOIHeaderQuery | L | DO module; 2 new CodeGen; custom override |
| 22 | JE025 AccountGroupTemplate | L | JE module; PK-change pattern |
| 23 | BA015 BAAddressMassUpdate | L | BA module; 9 BRs; dual-panel |
| 24 | TR010 LeaseMasterDataMMSRoyaltyReporting | L | TR module; 13 BRs; mass-copy |
| 25 | VL005+VL006 DOIAccounting* | L | VL module; combined; query + rules |
| 26 | PN020+PN025 PPNCreation+Query | L | PN module; combined; large grid |
| 27 | RD030 RDPreTrackingResults | L | RD module; 3 new CodeGen; off-by-one bug fix |
| 28 | VL050 VLFullRevenueResults | L | VL module; 2 new CodeGen; 3 grids |

### Wave 3 — Complex / XL screens

| Priority | Screen | Effort | Rationale |
|----------|--------|:------:|-----------|
| 29 | TX021 StateTaxRateMaintenance | XL | TX module; 15 BRs; 4 tabs; 2 new CodeGen |
| 30 | EC010 EscheatProcessingRules | XL | EC module; 4 tabs; 4 new CodeGen |
| 31 | TS006 SeveranceTaxCommonMaintenance | XL | TS module; 13 BRs; Reverse/Fill/Copy |
| 32 | VL025 DRILinkMaintenance | XL | VL module; 25 BRs; 4 new CodeGen; PPN+SOD sync |
| 33 | JE030 SubledgerMaintenance | XL | JE module; 6 new CodeGen; 4 tabs; security |
| 34 | STG1 StageTableDataMaintenance | XL | MF module; architectural redesign; generic API |
| 35 | QP010 ProcessExecutionMonitor | XL | QP module; architectural — WCF-driven; no direct Web analog |
| 36 | MF035 CodeTableValueEditor | S–? | MF module; complexity TBD |

---

## CodeGen Object Backlog

Before migrating screens that need new CodeGen objects, these XML entries must be added to `QRA.CSharp.cg.xml`:

| Screen | Table | CodeGen Class | Files Needed |
|--------|-------|---------------|:------------:|
| CW010 | RONL_CW_ST_TAX_WITHLD | RonlCwStTaxWithld | 5 |
| CW011 | RONL_CW_ST_BACKUP_TAX_WITHLD | RonlCwStBackupTaxWithld | 5 |
| CW015 | RONL_CW_CTRY_TAX_WITHLD | RonlCwCtryTaxWithld | 5 |
| JE011 | JXRF_ACCT_SL | JxrfAcctSl | 5 |
| GB015 | PONL_IMBAL_MANL | *(verify class name)* | 5 |
| DO004 | GXRF_DESKID_DO_CRTN | GxrfDeskidDoCrtn | 5 |
| DO004 | GXRF_DESKID_DO_MAINT | GxrfDeskidDoMaint | 5 |
| EC005 | DONL_ESCHT_OWNR | DonlEschtOwnr | 5 |
| EC005 | DONL_ESCHT_OWNR_DETAIL | DonlEschtOwnrDetail | 5 |
| EC010 | 4 ST Rule tables | 4 classes | 20 |
| RD030 | 3 _HIST tables | 3 classes | 15 |
| VL025 | RXRF_MI_SOD | RxrfMiSod | 5 |
| VL025 | GCDE_DSTRB | GcdeDstrb | 5 |
| VL025 | GCDE_DO_MAJ_PROD | GcdeDoMajProd | 5 |
| VL025 | VVL_DSTRB_ROY_SEL | VvlDstrbRoySel | 5 |
| VL050 | RTRN_FINAL_HIST_TAX | RtrnFinalHistTax | 5 |
| VL050 | RTRN_FINAL_HIST_ADJ | RtrnFinalHistAdj | 5 |
| TX021 | RONL_ST_TAX_RATE_EXMPT | RonlStTaxRateExmpt | 5 |
| TX021 | RONL_ST_TAX_RATE_MKT_ADJ | RonlStTaxRateMktAdj | 5 |
| JE030 | 6 JONL_SL/JTRN tables | 6 classes | 30 |

**Total new files required before all screens can migrate: ~145 generated files across 27+ new CodeGen objects.**

---

## Shared Component Reuse Opportunities

| Opportunity | Screens That Benefit |
|-------------|----------------------|
| Effective-date overlap BR (shared base class or utility) | CW010, CW011, CW015, TX021 |
| Allocation-factor sum validation utility | TR010 |
| PK-change-delete+reinsert pattern | JE025 |
| @@PPN_IDs injection pattern | PN025 (and VL031 already migrated) |
| Mass-copy subsystem | TR010, TX021 |
| Read-only enforcement via VL proc date | TX021, VL025 |
| Tab-level security pattern | JE030, EC010, TX021 |
| Generic staging table API | STG1 (architectural spike needed) |
