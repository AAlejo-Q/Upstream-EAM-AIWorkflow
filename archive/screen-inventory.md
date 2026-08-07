# EAM Screen Inventory

## Source

Data compiled from SharePoint "DRI Footprint Screens.xlsx" (last modified 2026-03-19)
and ADO Feature descriptions under Epic 1786991.

---

## Complete Screen List with POC Status

### Check Write (Feature 1786986)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| CW005 | Check Register Query/View | P1 | Classic | ~35% | 28 (16C/12M) |
| CW010 | State Withholding Tax Maint | P1 | Classic | ~75% | 12 (6C/4M/2L) |
| CW011 | State Backup Withholding Tax | P1 | Classic | ~85% | 8 (3C/3M/2L) |
| CW015 | Non-Resident Alien Withholding | P1 | Classic | ~85% | — |
| CW020 | Void Check Maintenance | P1 | Classic | — | — |
| EC005 | Escheat Maintenance/History | P2 | Classic | — | — |
| EC010 | Escheat Processing Rules | P2 | Classic | — | — |

### Contracts (Feature 1786987)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| RC005 | Contract Maintenance | P1 | Classic | — | — |
| RC010 | Measurement Point Contract Xref | P1 | Classic | **0% — NO IMPLEMENTATION** | 45+ (35C/10M) |
| RC036 | Price Terms Maintenance | P1 | Classic | — | — |

### Direct Revenue Input (Feature 1786972) — HIGH PRIORITY

| Code | Name | Priority | Source | POC Status | Gaps | Design Approach |
|------|------|----------|--------|------------|------|-----------------|
| VL025 | DRI Link Maintenance | P1 | Classic | ~75-80% | 22 (8C/10M/4L) | Lift and Shift |
| VL031 | VL Direct Revenue Input | P1 | Classic | ~85% | 12 (7C/5M) | Lift and Shift |
| VL032 | VL Direct Revenue Input Upload | P3 | Classic | ~75% | 18 (11C/7M) | Lift and Shift |
| VL100 | Revenue Selection/Submittal | P1 | Classic | **~15% prototype** | 47+ (majority CRIT) | Redesign |

### Revenue Distribution (Feature 1786980)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| ML006 | SOD DRI Links | P1 | Classic | ~65% | 18 (10C/5M/3L) |
| PN020 | PPN Creation | P1 | Classic | — | — |
| PN025 | PPN Query | P3 | Classic | — | — |
| RD010 | Reversal DOI Deletion | P3 | Classic | ~55% | 18 (8C/7M/3L) |
| RD030 | RD Pre Tracking Results | P3 | Classic | — | — |
| RD031 | RD Post Tracking Results | P3 | Classic | — | — |
| TX020 | State Tax Rate Query | P1 | Classic | — | — |
| TX021 | State Tax Rate Maintenance | P1 | Classic | — | — |
| VL005 | DOI Accounting Data Query | P3 | Classic | ~30% (query only) | 16 (8C/5M/3L) |
| VL006 | DOI Accounting Rules | P1 | Classic | ~30% (query only) | 42 (28C/14M) |
| VL050 | VL Full Revenue Results | P3 | Classic | — | — |

### Journal Entry (Feature 1786978) — LARGEST GROUP

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| JE005 | JE Group Type Maintenance | P1 | Classic | ~85% | 9 (3C/4M/2L) |
| JE010 | Account Setup Maintenance | P1 | Classic | ~85% | 8 (3C/5M) |
| JE011 | Account/Subledger Xref | P1 | Classic | ~85% | 8 (3C/5M) |
| JE015 | Account Group | P1 | Classic | ~92% | 4 (2C/2M) |
| JE020 | Account/Account Group Xref | P1 | Classic | — | — |
| JE025 | Account Group Template | P1 | Classic | ~78% | 18 (8C/6M/4L) |
| JE030 | Subledger Maintenance | P1 | Classic | — | — |
| JE100 | JE Post Selection/Submittal | P1 | Classic | ~65% | 15 (8C/5M/2L) |
| JE102 | Accounting Month Roll Date | P2 | Classic | — | — |
| JE110 | GL MTD Detail Query | P3 | Classic | ~75% | 14 (7C/5M/2L) |
| JE111 | GL History Detail Query | P3 | Classic | ~75% | 10 (4C/4M/2L) |
| JE205 | Manual JE Batch Maintenance | P1 | Classic | ~55% | 28 (16C/12M) |
| JE206 | Manual JE Detail | P2 | Classic | ~65% | 31 (18C/13M) |
| JE207 | Manual JE Tax and Adj | P2 | Classic | — | — |
| JE210 | Manual JE Batch Approval | P2 | Classic | ~92% | 8 (0C/3M/5L) |
| JE305 | HIST Reversal Batch Selection | P2 | Classic | ~85% | 8 (4C/3M/1L) |
| JE310 | HIST Reversal Batch Execution | P2 | Classic | ~92% | — |
| JE315 | MTD Reversal Batch Selection | P2 | Classic | ~85% | — |
| JE320 | MTD Reversal Batch Execution | P2 | Classic | — | — |

### Master Data / Production (Feature 1786987)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| CC010 | Cost Center Maintenance | P3 | Web | ~40% | 35 (22C/13M) |
| MF035 | Code Table Value Editor | P1 | Classic | ~65% | 28 (15C/8M/5L) |
| PD035 | Well Completion | P2 | Web | ~35% | 29+ (12+C) |
| PR005 | Property | P2 | Web | ~70% | 22 (16C/6M) |
| SP025 | Well Completion/Property/DOI Xref | P2 | Web | ~35% | 31 (18C/9M/4L) |
| STG1 | Stage Table Data Maintenance | P1 | Classic | **~15% prototype** | 32 (18C/8M/6L) |
| VA005 | Gas Volumes | P1 | Classic | ~85% | 8 (4C/4M) |
| VA015 | Liquid Volumes | P1 | Classic | ~92% | 6 (3C/3M) |
| VA035 | VA Results | P2 | Classic | ~75% | 6 (2C/4M) |

### DOI (Feature 1786975)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| DO004 | DOI Header Query | P3 | Classic | ~55% | 18 (12C/6M) |
| DO005 | DOI Query | P3 | Web | ~75% | 15 (8C/7M) |
| DO006 | DOI Setup | P3 | Web | ~65% | 28 (15C/13M) |
| DO016 | MEG Setup | P3 | Web | ~90% | 8 (3C/5M) |
| DO105 | DOI History | P3 | Web | — | — |
| MG004 | Market Group Header Query | P3 | Classic | ~60% (query) | 28 (15C/8M/5L) |
| MG005 | Market Group Query | P3 | Classic | ~60% (query) | 28 (15C/8M/5L) |
| MG006 | Market Group | P2 | Web | ~75% | 18 (11C/7M) |
| BA005 | Business Associate | P3 | Web | — | — |

### eSuite/BA (Feature 1786985)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| BA005 | Business Associate | P3 | Web/Classic | — | — |
| BA010 | BA Mass Update | P3 | Classic | — | — |
| BA015 | BA Address Mass Update | P3 | Classic | — | — |
| BA030 | Contact Maintenance | P3 | Classic | — | — |
| CC010 | Cost Center Maintenance | P3 | Web | ~40% | 35 (22C/13M) |

### Tax and Reg (Feature 1786981)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| TR010 | Lease Master Data - MMS Royalty | P1 | Classic | ~15% (shell only) | 48 (28C/15M/5L) |
| TS005 | Severance Tax Common Query | P1 | Classic | ~75% | 7 (0C/3M/4L) |
| TS006 | Severance Tax Common Maint | P1 | Classic | — | — |
| TS010 | Texas Tax Master | — | Classic | **0% — NOT MIGRATED** | 48 (32C/11M/5L) |
| TX005 | State Tax Information Query | — | Classic | **0% — NO IMPLEMENTATION** | 28 (18C/10M) |

### System/Batch (Feature 1789563)

| Code | Name | Priority | Source | POC Status | Gaps |
|------|------|----------|--------|------------|------|
| QP010 | Process Execution Monitor | P2 | Classic | ~85% | 4 (2C/2M) |
| QP043 | Process Launcher - QRA | P2 | Classic | ~45% | 18 (8C/6M/4L) |
| QP045 | Process Execution Monitor Query | P2 | Classic | ~70% | 20 (7C/8M/5L) |
| QP085 | Report Launcher - QRA | P2 | Classic | ~65% | 18 (8C/6M/4L) |
| QP110 | Active Lock Monitor | P2 | Classic | ~85% | 8 (3C/3M/2L) |
| GB015 | Gas Balancing Manual Input | P2 | Classic | ~75% | 7 (3C/4M) |

---

## Design Approach Summary

| Approach | Count | Description |
|----------|-------|-------------|
| Lift and Shift | ~55 | Direct migration preserving existing UX |
| Redesign | ~8 | Significant UX improvement (MJE combo) |
| Combine | ~5 | Merge related screens into unified workflow (VL005 and VL006, TS005 and TS006) |
| Use Existing Web | ~12 | Leverage already-built MyQuorum Web screens |

---

## Screens with Zero Implementation (Highest Risk)

1. **RC010** — Measurement Point Contract Xref (0%, 45+ gaps)
2. **TX005** — State Tax Information Query (0%, 28 gaps)
3. **TS010** — Texas Tax Master (0%, 48 gaps)
4. **STG1** — Stage Table Data Maintenance (~15%, handles 316 staging tables)
5. **VL100** — Revenue Selection/Submittal (~15%, 47+ gaps, 6-tab complex screen)
6. **TR010** — Lease Master Data (~15%, 48 gaps)
