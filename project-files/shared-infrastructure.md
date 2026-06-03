# Shared Infrastructure — Technical Reference

---

## Metadata Repository (4.2GB)

### Repo: `Quorum.Upstream.Metadata` (4,216,187,955 bytes)

The Metadata repo is the **largest repo in the entire Upstream Suite** — over 4GB. It contains
the QARCH (Quorum Architecture) framework that drives screen registration, navigation, security,
pick lists, code tables, and configuration for ALL four products.

### QARCH Framework Tables (referenced across all products)

| Table Pattern | Purpose | Used By |
|--------------|---------|---------|
| `QARCH_SEC_*` | Security — object permissions, field permissions, role assignments | All products |
| `QARCH_CNFG_*` | Configuration — application settings, screen defaults, feature flags | All products |
| `QARCH_ENTITY_CTRL_PROPERTIES` | Screen metadata — SHORT_NAME, LONG_NAME, navigation | All products |
| `QARCH_PROC_*` | Process definitions — batch process registration, parameters, steps | QPEC |
| `QARCH_PROC_EXEC_*` | Process execution — run history, status tracking, message logs | QPEC |
| `QARCH_CODE_DISPLAY_CTRL` | Pick list / parameter type display controls | All products |
| `QARCH_CTRL_PARAM` | Process parameter definitions | QPEC |

### MCP Tools for Metadata (quorum-metadata-dev / quorum-metadata-sup)

The project has MCP servers that expose metadata for AI-enabled workflows:

| Tool | Purpose |
|------|---------|
| `get_app_configuration_details` | Get config keys and values by name |
| `get_batch_process_all_details` | Get batch process definition, parameters, steps |
| `get_codetable_all_details` | Get code table definition (QFCodeTable, QFCodeTableFor) |
| `get_codetable_values` | Execute SELECT to get table data directly |
| `get_grid_all_details` | Get grid definition and column details |
| `get_picklist_all_details` | Get pick list definition, columns, input/output details |
| `get_process_parameter_details` | Get process parameter details from QARCH_CTRL_PARAM |
| `get_process_step_all_details` | Get process step definition, parameters, configurations |
| `get_parameter_type_details` | Get parameter type details from QARCH_CODE_DISPLAY_CTRL |
| `get_reg_sql_details` | Get registered SQL details by name |
| `get_cache_stats` | Get cache statistics for data sources |
| `list_data_sources` | List all available data sources |
| `refresh_cache` | Refresh cache for specific or all sources |

**Usage tip**: When investigating a screen's business rules, pick lists, or configuration,
use `quorum-metadata-dev:get_picklist_all_details` or `quorum-metadata-dev:get_codetable_all_details`
to get the metadata definitions. Use `quorum-metadata-dev:get_batch_process_all_details` to
understand batch process parameters and steps.

### Key Config Keys (QARCH_CNFG_*)

| Key | Purpose | Referenced By |
|-----|---------|--------------|
| `SHOW_TS_RRIDS` | Show/hide Revenue Run ID columns on TS screens | TS005, TS006 |
| `DRI_DEFAULT_GRAV` | Default gravity value for DRI (40.0) | TS006 |
| `PROPERTY_NUMBER_LENGTH` | Property number length for MI Link suffix parsing | VL025 Mass Changes |
| Various `FEATURE_FLAG_*` | Feature toggles per customer | All screens |

### Security Model (QARCH_SEC_*)

```
QARCH_SEC_OBJECT         — Screen/entity-level permissions (e.g., "QFRMMILINKMAINTENANCE")
QARCH_SEC_FIELD          — Field-level permissions (show/hide/read-only per role)
QARCH_SEC_ROLE           — Role definitions
QARCH_SEC_USER_ROLE      — User → Role assignments
QARCH_SEC_PRIVILEGE       — Privilege definitions (Execute, Read, Write, Delete)
```

Each ClassicGUI screen checks security on load:
```csharp
// QRA pattern
QSecurityUtility.IsAllowed(QSecurityPrivilege.ObjectOther, "SECURITY_ENTITY_ID", QSecurityPrivilege.PermissionExecute)

// QCA/QCFS pattern
// Security checked via metadata registration (QARCH_ENTITY_CTRL_PROPERTIES)
```

---

## Shared Database (32MB)

### Repo: `Quorum.Upstream.Database` (32,744,829 bytes)

Contains SQL schema definitions shared across all products. This is the **cross-product schema**
that includes:
- QARCH framework tables
- Shared reference data tables
- Cross-product lookup tables
- Common code tables (GCDE_* pattern)

### Product-Specific Databases

| Product | Database Repo | Size | Key Table Prefixes |
|---------|--------------|------|-------------------|
| QRA | `QRA.Database` | 11MB | RONL_*, RTRN_*, RXRF_*, TONL_*, THIST_*, JONL_*, CONL_* |
| QCA | `QCA.Database` | 4.7MB | Product-specific JIB, GL, revenue tables |
| QCFS | `QCFS.Database` | 5.3MB | AP, AR, GL, BR, MF module tables |
| Shared | `Database` | 32MB | QARCH_*, GCDE_*, common reference tables |

### QRA Table Prefix Convention

| Prefix | Meaning | Example |
|--------|---------|---------|
| `RONL_` | Revenue ONLine — live transaction data | RONL_MI_LINK_EFF_DT, RONL_MANL_INPUT |
| `RTRN_` | Revenue TuRN — staging/snapshot tables | RTRN_MI_STMNT_INPUT_SNAPSHOT |
| `RXRF_` | Revenue X-ReFerence — cross-reference tables | RXRF_MI_SOD, RXRF_MI_RMITR_DFLT |
| `TONL_` | Tax ONLine — tax transaction data | TONL_TAX_INPUT, TONL_MMS_RYL_LSE_MST |
| `THIST_` | Tax HISTory — historical/audit tables | THIST_AUDIT3 |
| `JONL_` | Journal ONLine — JE transaction data | JONL_JE_* |
| `CONL_` | Check ONLine — check write data | CONL_CW_* |
| `DONL_` | DOI ONLine — division order data | DONL_DO_HDR, DONL_DO_DTL |
| `GCDE_` | General CoDE — shared code/reference tables | GCDE_MAJ_PROD, GCDE_STATE |

---

## Shared.Web (32MB)

### Repo: `Quorum.Upstream.Shared.Web` (32,021,545 bytes)

Contains the shared web framework used by all product Web repos (QRA.Web, QDO.Web, QCA.Web, QCFS.Web).

Provides:
- Base controller classes
- Shared authentication/authorization middleware
- Common data contract definitions
- Shared service infrastructure
- Web framework utilities (grid helpers, export, pagination)
- Shared JavaScript/CSS libraries for the web UI

---

## Shared.ClassicGUI (2.5MB)

### Repo: `Quorum.Upstream.Shared.ClassicGUI` (2,518,769 bytes)

Shared WinForms components used by both QCA.ClassicGUI and QCFS.ClassicGUI:
- Common base form classes
- Shared user controls
- Utility functions
- Common dialog implementations

Note: QRA.ClassicGUI uses `Application.ClassicGUI` (4.2MB) as its shared framework,
which is a separate repo with the QARCH-integrated base classes (QFrmBase*, etc.).

---

## Application.ClassicGUI (4.2MB)

### Repo: `Quorum.Upstream.Application.ClassicGUI` (4,224,052 bytes)

The **primary shared framework** for QRA and QDO ClassicGUI screens. Contains:
- `QFrmBase` — Root base form class
- `QFrmBaseFilterDetailQra` — Filter-detail pattern base
- `QFrmBaseMasterDetail` — Master-detail pattern base
- `QFrmBaseGrid` — Single-grid pattern base
- `QContextDataMgr` — Screen-to-screen navigation with context passing
- `QUpsUtility` — Utility class including `LaunchBatchProcess()`, `LaunchScreen()`
- `QBusinessRule<T>` — Business rule framework (rule registration, trigger types)
- `QNotesSetupInfo` — Comments/notes integration
- Pick list infrastructure (pick button controls, pick dialog management)
- Grid infrastructure (inline editing, row state tracking, pagination)
- SQL statement key infrastructure (registered SQL management)

### Key Utility Classes

| Class | Purpose | Used By |
|-------|---------|---------|
| `QUpsUtility` | Batch launch, screen launch, general utilities | All QRA/QDO screens |
| `QContextDataMgr` | Pass data between screens during navigation | All screen-to-screen links |
| `QBusinessRule<T>` | Register and execute business rules | All screens with validation |
| `QraUtility` | QRA-specific utilities (PPN creation, etc.) | QRA screens |
| `QUtilityDO` | DOI-specific utilities (approval checks, etc.) | VL, DO screens |
| `QUtilityVL` | VL-specific utilities (VL finalize checks, etc.) | VL screens |
| `QSecurityUtility` | Security permission checks | All screens |

---

## Reports (58MB)

### Repo: `Quorum.Upstream.Reports` (58,447,341 bytes)

Contains report templates for all products:
- SSRS (SQL Server Reporting Services) report definitions (.rdl)
- Crystal Reports definitions (.rpt) — legacy
- Report parameter definitions
- Shared report datasets

---

## Other Infrastructure

### Application.MiddleTier (514KB)
Shared deployment host for middle-tier services. Multi-target configuration
(Cloud, OnPremDirect, OnPremNI, Debug) mirrors QPEC deployment modes.

### Application.MaintenanceApp (618KB)
System maintenance utility for database operations, schema updates, and administrative tasks.

### Help (53MB)
Online help system with compiled help files for all products.

### Tools (1MB)
Development and deployment tools, build scripts, utility applications.

### CheckSignature (70KB)
Utility for check signature image management (used by CW module).

---

## TO EXPLORE NEXT

1. [ ] Explore Metadata repo directory structure — understand QARCH table schema definitions
2. [ ] Use MCP tools (quorum-metadata-dev) to query actual pick list definitions for DRI screens
3. [ ] Use MCP tools to query batch process definitions (DRIPROC, JEPOSTONLY parameters)
4. [ ] Explore Shared.Web framework — understand base controller patterns for Blazor migration
5. [ ] Read Application.ClassicGUI source — understand QFrmBase class hierarchy in detail
6. [ ] Map Database repo schema files to product table prefixes
7. [ ] Explore QRA.Database schema for table relationships and FK constraints
