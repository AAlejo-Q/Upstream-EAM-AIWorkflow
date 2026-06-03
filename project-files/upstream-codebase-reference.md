# Upstream Suite — Claude Agent Instructions

**Trust these instructions.** Only search for additional information if the instructions below are incomplete or found to be in error.

> **Origin:** Enhanced from `.github/copilot-instructions.md`, enriched with domain knowledge from *Petroleum Accounting: Principles, Procedures & Issues (5th Edition)* by Dennis Jennings, Joseph Feiten, and Horace Brock (PricewaterhouseCoopers / Professional Development Institute, 2000).

---

## 1. Repository Overview

Upstream Suite is Quorum Software's upstream oil & gas accounting platform. It manages **Joint Interest Billing (JIB)**, **Authorizations for Expenditure (AFE)**, **Revenue Accounting**, **Division Orders**, **General Ledger**, **Subledger**, **Accounts Payable/Receivable**, and related financial workflows for the oil & gas industry. The codebase is a multi-repository workspace (~16,000 C# files, **~1,723 C++ files**, ~992 WinForms files, 184 .csproj projects, 19 .sln solutions) targeting **.NET Framework 4.8** on Windows with **SQL Server** databases. **Important:** Many batch processes exist in parallel C++ (ClassicBatch) and C# (Batch) implementations — always search both.

### 1.1 Product Modules

| Abbreviation | Full Name | Purpose | Key Entities |
|---|---|---|---|
| **QCA** | Quorum Cost Accounting | AFE, JIB, GL, Subledger, Revenue, Fixed Assets, QPOT, LOS | AfeHeader, AfeDetail, JtrnSlDetail, JonlAcct |
| **QCFS** | Quorum Core Financials | AP, AR, GL, Bank Reconciliation, Fixed Assets, Check Run | Voucher, Vendor, Account, BatchVoucherMaster, InvoiceJournal |
| **QDO** | Quorum Division Orders | Division order interest tracking, transfers, bearer groups | DonlDoHdr, DonlDoDetail, DonlBearerGrpHdr |
| **QRA** | Quorum Revenue Accounting | Revenue distribution, royalty reporting, severance tax, check writing | GonlProp, PonlWellCompl, RtrnChkRgstr, RtrnRdInput |
| **ESuite** | Enterprise Suite | Shared web platform: Business Associates, Contracts, Meters, Orgs, Cost Centers | BusinessAssociate, ContractHeader, MeterHeader, Organization |

### 1.2 Module Interconnections

```
ESuite (master data: BA, Contracts, Orgs, Meters, Cost Centers)
  | provides master data to all modules
QDO (Division Orders -> bearer groups -> owner interests on properties)
  | DOI interests feed into revenue calculation
QRA (Revenue Accounting -> book revenue -> check writing -> severance/royalty)
  | revenue JEs post to GL, AP vouchers created
QCA (Cost Accounting -> JIB billing -> AFE tracking -> subledger)
  <-> cross-posts with QCFS GL
QCFS (Core Financials -> AP vouchers -> GL posting -> bank reconciliation -> check runs)
```

---

## 2. Upstream Oil & Gas Accounting Domain Knowledge

> **Reference:** *Petroleum Accounting — Principles, Procedures & Issues (5th Edition)* by Dennis Jennings, Joseph Feiten, Horace Brock. Text extracts available in `Best Practice docs/pdf_*.txt`.

### 2.1 Industry Structure & Organizational Context

**Company Types:**
- **Independents**: E&P companies without refining/marketing operations (most Quorum customers)
- **Integrated companies**: Combine upstream (E&P), downstream (refining, marketing), and midstream (gathering, processing, transportation)

**Core Functional Departments in E&P Companies:**

| Department | Function | Software Touchpoints |
|---|---|---|
| **Exploration** (Geology & Geophysics) | Accumulate and analyze subsurface data to identify drilling prospects | G&G cost tracking (Acct 801), prospect management |
| **Land & Lease Acquisition** | Negotiate leases, manage mineral rights, maintain title records, ensure timely rental payments | Property master (GONL_PROP), unproved property tracking (Acct 211) |
| **Drilling & Production (Petroleum Engineering)** | Exploitation, reservoir engineering, drilling operations, field management | AFE system, well WIP (Accts 241/243), LOE (Acct 710) |
| **Marketing** | Sell oil, gas, and NGL to purchasers; negotiate contracts and pricing | Revenue distribution, check input, purchaser contracts |
| **Revenue Accounting** | Maintain DOI master file, distribute revenue, compute production taxes, issue owner checks | QRA module — the heart of revenue processing |
| **Joint Interest Accounting** | Bill working interest partners, track JIB receivables, audit statements | QCA JIB module, COPAS compliance |
| **Property Accounting** | Track unproved/proved property costs, WIP, equipment, DD&A | Cost center management, fixed assets |
| **Accounts Payable** | Process vendor invoices, manage payment runs | QCFS voucher/check run system |
| **Tax & Regulatory** | Federal/state tax returns, royalty reporting, ONRR/MMS filings | QRA tax reporting (TS/TV/TR/TO areas) |

**Critical IT Concept — Division of Interest (DOI) File:**
The DOI file is the *central nervous system* of upstream accounting. It records every owner's decimal interest in every property. Revenue distribution, JIB billing, tax reporting, and check writing all depend on DOI accuracy. The Land department maintains it; Revenue Accounting consumes it. Errors in the DOI cascade to every downstream financial process.

### 2.2 Accounting Methods — Successful Efforts vs Full Cost

Two mutually exclusive accounting methods exist for upstream oil & gas under US GAAP. The choice affects how costs flow through the system and which accounts are used:

**Successful Efforts (SE) — FASB ASC 932 / SFAS 19:**
- Only costs of **successful** exploration are capitalized; dry exploratory wells are expensed to Account 804 (Dry Holes)
- Amortization computed per individual property, lease, field, or reservoir (never above field level in practice)
- Geological & Geophysical (G&G) costs expensed when incurred (Account 801)
- Unproved property carrying costs expensed periodically (Account 802). Impairment assessed property-by-property (Account 806 / contra 219)
- Exploratory well WIP (Accounts 241/243) held pending determination of proved reserves — transferred to proved (231/233) if successful, expensed to 804 if dry
- Development wells (both productive and dry) **always capitalized** — a dry development well still adds value by confirming reservoir boundaries
- Favored by larger producers (~20 of top 20 companies use SE)
- **One-year deferral limit**: Exploratory well costs in WIP must be expensed after one year unless (a) sufficient proved reserves exist AND (b) active drilling is planned

**Full Cost (FC) — SEC Regulation S-X Rule 4-10:**
- **All** exploration and development costs capitalized into a single country-wide cost center
- Subject to a quarterly **ceiling test**: net capitalized cost <= PV10 of proved reserves + lower of cost/market of unproved properties + cost of properties not subject to amortization - deferred income tax. Excess is written down (Account 760/761)
- Amortization is country-wide using unit-of-production over proved reserves
- No concept of "dry hole expense" — all wells are capitalized regardless of outcome
- FC-only accounts: 227 (Abandoned Properties), 228 (Ceiling Impairment), 229 (Unsuccessful Exploration), 236-238 (FC accumulation)
- No gain/loss on property sales (except if proceeds so large they distort amortization rate)

**Cost Categories (both methods):**

| Category | SE Treatment | FC Treatment | Key Accounts |
|---|---|---|---|
| **Acquisition** (lease bonus, broker fees, filing) | Capitalize; track proved vs unproved | Capitalize | 211 (Unproved Leaseholds), 221 (Proved Leaseholds) |
| **Exploration** (G&G, exploratory drilling, test-wells) | Capitalize only successful wells; expense G&G, dry holes | Capitalize all | 240 (G&G WIP), 241/243 (Well WIP), 801, 804 |
| **Development** (development drilling, platforms, facilities) | Capitalize all (even dry development wells) | Capitalize all | 231 (Intangible), 233 (Tangible), 241/243 (WIP) |
| **Production** (LOE, workovers restoring production) | Expense as incurred | Expense as incurred | 710 (LOE subaccounts .001-.022) |

### 2.3 Chart of Accounts Structure

The industry-standard chart of accounts (per *Petroleum Accounting* Appendix 5) maps directly to database entities:

| Range | Category | Key Accounts | System Mapping |
|---|---|---|---|
| **100-109** | Cash | 100 General Bank, 102 Payroll, 103 Tax Deposit | Bank reconciliation |
| **110-119** | Short-term Investments | Marketable securities, interest-bearing accounts | Treasury |
| **120-129** | Accounts Receivable | 120 Oil/Gas Sales AR, 121 Gas Imbalance AR, 123 JIB AR | AR subledger, JIB billing |
| **130-139** | Inventories | 130 Crude Inventory, 131 Gas in Storage, 135 Materials & Supplies | Field inventory (38% of companies track) |
| **140-149** | Other Current Assets | Prepaid expenses, futures margin accounts | Hedging |
| **210-219** | Unproved Property | 211 Unproved Leaseholds (.001-.999 per lease), 214 Fee Interests, 219 Impairment Allowance | `GONL_PROP`, Prospect tracking |
| **220-226** | Proved Property | 221 Proved Leaseholds, 222-225 Other proved interests, 226 Accum Amort | Property master -> DD&A |
| **227-229** | Full Cost Only | 227 Abandoned Properties, 228 Ceiling Impairment, 229 Unsuccessful Exploration | FC cost pool |
| **230-239** | Wells & Development | 231 Intangible (.001-.07 per AFE line), 232 Accum Amort, 233 Tangible (.001-.009), 234 Accum Amort | AFE detail lines, Fixed Assets |
| **240-248** | WIP | 241 Intangible WIP (.001-.029 per AFE line), 243 Tangible WIP (.030-.049), 244 Workovers, 245 Support, 246 Gas Processing, 247 Enhanced Recovery | AFE tracking; `QCTRL_AFE_HDR/DETAIL` |
| **258-269** | Equipment & Facilities | General support, autos, office, buildings, land | Fixed assets |
| **300-349** | Current Liabilities | 301 Vouchers Payable, 302 Revenue Distributions Payable, 304 Revenue in Suspense, 305 Advances from JI Owners, 306 Gas Imbalance Payable, 320 SevTax Payable, 321 Ad Valorem Tax | AP, Revenue, Tax modules |
| **350-369** | Clearing/Allocation | 350 District, 351 Region, 352 Support Facility, 360 Revenue Control, 361 Billing Control | Cost allocation, JIB billing control |
| **400-439** | Long-Term Liabilities | 410 Site Restoration (ARO), 420 Deferred Income Taxes, 430 Deferred Revenue | ARO, tax, production payments |
| **500-530** | Stockholders' Equity | Preferred/common stock, retained earnings, dividends | Financial statements |
| **600-630** | Revenue | 601 Oil, 602 Gas, 603 NGL, 604-606 Royalty Oil/Gas/NGL, 607 Net Profit Interests, 610 Hedging | Revenue distribution |
| **710** | LOE | .001 Supervision, .003 Labor, .005 Chemicals, .006 Fuel, .007 Power, .008 Water Disposal, .009 Compression, .010 Pumping, .011 Transportation, .012 Saltwater Disposal, .014 Equipment Rentals, .018 Insurance, .019 Ad Valorem Tax, .022 Other | LOE subaccounts -> cost center allocation |
| **725-749** | DD&A | 725 FC Amort, 726 SE Acquisition Depletion, 732 SE Intangible Depreciation, 734 SE Tangible Depreciation, 735 DR&A Amort | DD&A batch process |
| **800-806** | Exploration Expense | 801 G&G, 802 Undeveloped Carrying, 803 Test-Well Contributions, 804 Dry Holes, 805 Strat Test Wells, 806 Unproved Impairment | SE method only |
| **900-960** | G&A & Other | 900-919 Overhead (salaries, rent, office, professional fees), 940 Interest Expense, 950 Income Tax | Overhead allocation |

**Intangible vs Tangible Distinction (critical for AFE system):**
- **Intangible Drilling Costs (IDC)** (Account 231/241): Costs with no salvage value — labor, cement, chemicals, mud, fuel, rig mobilization, site prep. For tax purposes, IDCs are generally **immediately deductible** (IRC S263(c))
- **Tangible Well Equipment** (Account 233/243): Physical items with salvage value — casing, tubing, wellhead, pumping unit, separators, tanks. Tax: depreciated under MACRS over 7 years
- AFE forms split along this line: Items 001-022 (Intangible) and Items 030-045 (Tangible). Costs further split at **"Cost to Casing Point"** and **"Complete"** boundaries

### 2.4 Unproved Property Acquisition, Retention & Surrender

**Lease Fundamentals:**
The mineral lease is the foundational legal instrument. A lessor (mineral rights owner) grants a lessee (oil company) the right to explore and produce in exchange for a bonus payment and ongoing royalties.

**Core Lease Provisions:**

| Provision | Description | Typical Terms |
|---|---|---|
| **Lease Bonus** | Cash paid upfront for exploration/production rights | $1-$1,000s/acre onshore; $millions for offshore federal blocks |
| **Primary Term** | Maximum period to commence drilling | 2-7 years (2-3 increasingly common) |
| **Delay Rentals** | Payment to defer drilling obligation by 1 year | $1-$5/acre; much less than bonus |
| **Royalty** | Lessor's share of production revenue, free of costs | Typically 1/8 (12.5%); negotiable up to 1/5+ |
| **Shut-in Royalties** | Payment when well is capable but not producing (no market, no pipeline) | Usually equals delay rental amount |
| **Offset Clause** | Requires operator to drill offset wells when adjacent wells prove productive | Failure may trigger compensatory royalties |
| **Pooling/Unitization** | Combine leased acreage with others; ownership prorated by acreage | Conservation-driven; reduces development costs |

**Accounting for Unproved Properties:**
- SE method: Capitalize acquisition costs (Acct 211); expense carrying costs (delay rentals -> Acct 802, ad valorem taxes -> 802.002)
- FC method: Capitalize everything including delay rentals; exclude from amortization base until proved
- **70-90% of unproved acreage is ultimately abandoned without production** — impairment assessment required at least annually
- Two assessment methods: (1) significant properties assessed individually; (2) non-significant properties assessed as a group using allowance method (Acct 219)
- Surface rights are distinct from mineral rights — operator must separately negotiate surface access/damages

### 2.5 Property, Ownership & Interest Types

**Property & Well Hierarchy:**
- **Property (GONL_PROP)**: A mineral interest or lease — the fundamental unit for revenue and cost tracking. Identified by `PropNo`. Property grouping varies: 58% of companies group by field, 13% by property, 11% by well
- **Well Completion (PONL_WELL_COMPL)**: A producing wellbore on a property. Multiple wells per property. Has API number, status, production data. A well may have multiple completions (zones)
- **Flow Grid (PONL_FLOW_GRID)**: Production measurement point mapping volumes to wells/properties
- **Cost Center**: Can be Unit/Property (1), Well Completion (2), Property (3), Prospect (4), Billable Allocation Group (5), Inventory Warehouse (6), or Other (7) — defined in `QUpsGlobalConstants.CostCenterTypes`

**Mineral Interest Types (per SFAS 19/ASC 932):**

| Interest Type | Cost Bearing? | Revenue Receiving? | Typical Holder | System Representation |
|---|---|---|---|---|
| **Working Interest (WI)** | Yes — bears proportionate share of all drilling/operating costs | Yes — receives NRI share of revenue | Operator and partners | DOI with WI interest type; JIB billing |
| **Royalty Interest (RI)** | No — cost-free | Yes — typically 12.5%-25% of gross production | Landowner (mineral rights owner) | DOI detail; revenue distribution |
| **Overriding Royalty (ORRI)** | No — cost-free | Yes — carved from WI, typically 1%-5% | Geologist, broker, prior assignor | DOI detail; created in farmout transactions |
| **Net Profits Interest (NPI)** | Not directly, but share computed net of costs | Yes — share of net profit | Varies | Account 607; computed after cost deduction |
| **Production Payment (PP)** | No | Yes — terminates after stipulated amount received | Financing counterparty | Not a permanent interest; tracked separately |
| **Carried Interest** | Deferred — carried party bears no cost until payout | Deferred — carried party receives no revenue until payout | Farmout assignor (carried party) | Payout tracking; nonconsent penalty (typically 300-500%) |
| **Fee Interest** | Varies | Yes — includes both surface and mineral rights | Original property owner | Combines royalty + potential WI |

**Key Ownership Formulas:**
- **Net Revenue Interest (NRI)** = Working Interest x (1 - Royalty Burden). Example: 50% WI with 20% total royalty = 50% x 0.80 = 40% NRI
- **Royalty Burden** = Aggregate of all royalty, ORRI, and other non-cost-bearing interests on the lease
- Revenue to WI owner = Gross Revenue x NRI decimal. Revenue to RI owner = Gross Revenue x RI decimal

**Division Orders & Revenue Distribution:**
- **Division Order (DO)**: Legal instrument signed by all interest owners confirming their decimal interest in a property's production. Based on title opinion. Industry form: NADOA (National Association of Division Order Analysts) Model Form
- **DOI File (DONL_DO_HDR + DONL_DO_DETAIL)**: Master record of each owner's decimal interest. Source of truth for revenue distribution. When a new DO is received, it is entered as a **pending** DOI (`DVD_`/`DONL_DVD_` tables) and activated after review
- **Revenue in Suspense (Account 304)**: Revenue held when a valid division order is not on file, title is disputed, owner address is unknown, or minimum check threshold not met. Must be paid/escheated per state unclaimed property laws (dormancy periods vary: TX=3yr, OK=5yr, etc.)

**Bearer Groups & Revenue Distribution:**
- **Bearer Group (DONL_BEARER_GRP_HDR/DETAIL)**: Defines how revenue costs/deductions are allocated among interest owners. Members have percentage shares. Changes trigger PPN recalculation
- **GWI Bearer Group**: Gross Working Interest bearer group — allocates deductions among WI owners
- **PPI Bearer Group**: Pipeline/Processing Interest bearer group
- **SOD (Special Owner Distribution)**: Custom deduction treatment for specific owners. Owners with `SOD_GRP_NO IS NOT NULL` receive special handling (different severance tax rates, transportation deduction exemptions, tax-exempt entities)
- **TIK (Third-party Interest Keeper)**: Working interest owner whose revenue/deductions are managed by a third party. Presence of TIK on a property changes PPN routing (LD20 instead of LD18)

### 2.6 Exploration, Drilling & Well Classification

**Well Classification (drives accounting treatment):**

| Well Type | Definition | SE Treatment | Key WIP Accounts |
|---|---|---|---|
| **Exploratory Well** | Drilled in unproven area to find new reserves | Capitalize if successful -> 231/233; dry hole -> expense 804 | 241 (Intangible WIP), 243 (Tangible WIP) |
| **Development Well** | Drilled in proven area to exploit known reserves | **Always capitalize** (even if dry — confirms reservoir boundaries) | 241/243 -> 231/233 |
| **Stratigraphic Test Well** | Drilled to obtain geological information (no production intent) | Exploratory-type: may capitalize if results usable | 805 if unsuccessful |
| **Service Well** | Injection, disposal, observation wells | Capitalize as development cost | 231/233 |

**Drilling Cost Flow:**
```
AFE Approved -> Costs incurred -> WIP (241 Intangible / 243 Tangible)
  -> Well successful? --Yes--> Transfer to Proved (231/233)
  -> Well dry?
       |-- Exploratory --> Expense to Dry Holes (804)
       +-- Development --> Still capitalize (231/233) -- confirms reservoir
```

**Casing Point Decision:** After drilling to objective depth, the operator decides whether to complete or abandon the well. This is the critical junction where costs transition from "drilling" to "completion" and where the intangible/tangible cost split becomes final.

**Interest Capitalization (FASB ASC 835-20 / FAS 34):** Interest costs on borrowed funds are capitalized as part of well costs during the construction period (from spud date to rig release). Ceases when asset is substantially complete.

**Supplemental AFE:** When actual costs exceed the original AFE estimate by a material threshold (typically 10-15%), a supplemental AFE must be prepared and approved, maintaining the audit trail of budget vs actual.

### 2.7 Authorization for Expenditure (AFE)

AFE is the capital expenditure authorization and tracking workflow:

1. **AFE Creation** — Estimate prepared with cost detail lines organized as: Intangible items (001-022: location/site prep, rig mobilization, fuel, water, chemicals, cement, logging, testing, etc.) and Tangible items (030-045: casing, tubing, wellhead, pumping unit, flowlines, separators, tanks). Further split at "Cost to Casing Point" and "Complete" boundaries. Includes cost center splits and supplements
2. **Workflow** — Routed for approval through configurable approval chains. States: Estimate -> Pending -> Approved -> Active -> Closed
3. **Project tracking** — Actual costs compared to approved amounts in real-time. Over-budget triggers supplemental AFE
4. **Partner billing** — Authorized costs allocated to JIB partners per their WI percentages

**AFE Status Codes:** Estimate, Pending, Approved, Active, Supplemented, Closed, Rejected, Cancelled
**AFE Workflow Actions:** Save, Delete, WFSubmit, WFApprove, WFReject, WFRouteAndReturn, WFSendForComment, WFFinalReject, WFReSubmit, WFDelegate, WFKill
**Key AFE Tables:** `QCTRL_AFE_HDR` (header), `QCTRL_AFE_DETAIL` (line items), linked to cost centers and billing entries

### 2.8 Joint Operations, JOA & JIB

**Joint Operating Agreement (JOA):**
The legal framework governing jointly-owned oil & gas properties. The AAPL (American Association of Professional Landmen) Form 610 is the industry standard. Key provisions:
- **Operator designation** — one WI owner operates on behalf of all; operator is not intended to profit or lose from operator role
- **Accounting procedure** — governed by COPAS (Council of Petroleum Accountants Societies) model form, attached as exhibit to the JOA
- **Non-consent (Independent Operations)** — if a partner declines to participate in an operation, the consenting parties proceed, and the non-consenting party's share is "carried" at a penalty (typically 300-500% cost recovery before the non-consenter's interest reverts)
- **Preferential right to purchase** — if a party wants to sell their interest, other parties may have first right of refusal
- **Area of Mutual Interest (AMI)** — geographic zone where parties have first right to participate in new opportunities; prevents unfair use of joint venture exploration data for personal gain

**COPAS Accounting Procedure (5 sections):**

| Section | Content | Key Rules |
|---|---|---|
| **I. General Provisions** | Definitions, adjustments, audit rights | **24-month audit window** — partners must contest charges within 24 months of calendar year end or charges deemed accepted |
| **II. Direct Charges** | 12 categories of costs charged directly to the joint account | Labor, material, transportation, services, equipment rentals, damages/losses, legal, taxes, insurance, communications, ecological/safety, abandonment |
| **III. Overhead** | Operator compensation for G&A | Two methods: (1) **Fixed-rate** — separate drilling rate and producing rate per well/month; (2) **Percentage** — % of total costs. Overhead is NOT a profit margin — it reimburses the operator for back-office costs |
| **IV. Material Pricing** | Valuation of material transfers | **Condition A** = new (100%), **Condition B** = used/good (75% of new), **Condition C** = used/repairable (50% of new). Material moving between JA properties: transfers at condition value |
| **V. Inventories** | Physical inventory procedures | Annual or periodic physical count; reconciliation to book; adjustment to joint account |

**COPAS Audit & Billing Rules (critical for software compliance):**
- Operator bills Non-Operators by last day of month for prior month's share
- Non-Operators pay within 15 days of receipt
- Late payments accrue compound monthly interest: U.S. Treasury 3-month rate + 3%
- Audit exceptions must be raised within 24-month window
- Lead audit company issues report within 180 days of field work completion
- Operator allows/denies exceptions within 180 days of report receipt
- Affiliate charges cannot exceed average commercial rates; affiliates must open records for audit

**COPAS Overhead Rates:**
- **Drilling wells**: Rate per month from spud date (onshore) or equipment arrival (offshore) through rig release; no charge during 15+ day suspensions
- **Producing wells**: Rate per month for active completions (producing, injecting, or water supply)
- **Major construction**: Percentage of total project costs above a dollar threshold
- **Catastrophe**: Same percentage approach for calamitous single-occurrence events
- Rates adjusted annually per COPAS-recommended percentage; subject to review at 2-year and challenge at 4-year intervals

**JIB (Joint Interest Billing):**
JIB is the process of billing working interest partners for their share of property costs:

1. **Operator incurs costs** — drilling, operating, overhead (direct charges per COPAS Section II)
2. **Cost allocation** — costs distributed to properties/wells via AFE or operating cost center
3. **Operator "cut-back method"** — operator records 100% of gross cost, then at month-end "cuts back" to its WI share. Non-operator share becomes JIB receivable (Account 123). Alternative: split at time of recording
4. **JE Billing Entry** — creates billing records allocating each partner's share of costs
5. **Billing statement** — sent monthly to partners showing their proportionate share of each charge
6. **Partner payment tracking** — payments tracked in subledger (`JTRN_SL_DETAIL`). Partners typically have 30 days to pay
7. **Overhead charges** — per COPAS rates, added to each billing period

**Nonoperator Recording:**
- Nonoperator receives JIB statement and records: `DR Property/WIP accounts, CR 301 Vouchers Payable` (at their WI share only)
- Must reconcile JIB charges against their own AFE tracking and raise disputes within the 24-month audit window

**Key JIB Tables:** `JBCDE_*` (billing code definitions, business segment rules, owner rules), `JONL_ACCT` (accounts), `JTRN_SL_DETAIL` (subledger transactions), clearing accounts 360 (Revenue Control) / 361 (Billing Control)
**EDI Systems for JIB:** CODE, CDEX, GRADE, JADE, JIBE — electronic data interchange formats for transmitting billing data between operators and partners

### 2.9 Production & Volume Measurement

**Production Infrastructure:**
- Wells produce commingled gas, oil, and water requiring separation and treatment
- **Separators** (two-phase or three-phase) use gravity/centrifugal force to separate products
- **Treaters** (heater-treater, electrostatic) remove emulsions
- Multi-stage separation maximizes product recovery before sale

**Quality Standards & Measurement:**

| Product | Measurement Unit | Standard Conditions | Quality Threshold |
|---|---|---|---|
| Crude Oil | Barrels (bbl) = 42 US gallons | Temperature corrected to 60 degrees F | <1% Basic Sediment & Water (BS&W) |
| Natural Gas | mcf (thousand cubic feet) | 14.73 psia, 60 degrees F | <7 lbs water vapor per MMcf |
| NGL/Condensate | Gallons (GPM x mcf processed) | Product-specific | Per product specs |

**Measurement Points:**
- **Oil**: LACT unit (Lease Automatic Custody Transfer) provides automated measurement; alternative is manual tank gauging with run tickets
- **Gas**: Sales meter (purchaser-owned) with check meter (operator-owned) for verification; tolerance variance typically 2-5%
- **API Gravity**: Measures crude oil density — higher number = lighter, more valuable crude; affects pricing via gravity adjustment scales

**Energy Conversions:**
- Gas energy measured in mmBtu (million British thermal units)
- Barrel of oil equivalent (BOE): ~6 mcf gas = 1 bbl oil (energy content basis); value basis can be ~10:1
- Standard pressure base varies by state: 14.65-15.025 psia

### 2.10 Marketing & Revenue Accounting

**Revenue Types & Accounts:**

| Product | Revenue Account | Measurement | Pricing |
|---|---|---|---|
| **Oil** | 601 | Barrels (bbl) at 60 degrees F | Posted price or contract price per bbl |
| **Gas** | 602 | Volume (mcf) x heating value (mmBtu/mcf) x price/mmBtu | Spot or contract per mmBtu |
| **NGL/Condensate** | 603 | Gallons (GPM x mcf processed) | Product-specific prices (ethane, propane, butane, natural gasoline) |
| **Royalty Oil/Gas/NGL** | 604/605/606 | Operator pays royalty owner's share, often "in-kind" or in-value | Same pricing as above |
| **Net Profit Interests** | 607 | Computed as percentage of net profit (revenue - costs) | Per NPI agreement |

**Oil Marketing:**
- Customer base: ~30 major refineries control 90% of U.S. capacity; refineries optimize for consistent crude types
- Contract types: Evergreen (month-to-month), Spot (short-term), Exchange (crude swap with quality/location differentials)
- Posted Price Bulletins with gravity adjustment scales (premiums for light, discounts for heavy crude)
- Financial instruments: NYMEX futures; Exchange of Futures for Physicals (EFP); most positions closed out rather than physically settled

**Gas Marketing:**
- Nomination process: Producer reserves pipeline capacity by forecasting monthly volumes
- Firm vs. Interruptible service: Different priority access to pipeline capacity
- Structural changes ongoing: Direct sales to non-residential customers, bypassing pipeline intermediaries

**Gas Value Calculation:**
- Raw gas value: `Volume (mcf) x Heating Value (mmBtu/mcf) x Price ($/mmBtu)`
- Dry gas vs saturated (wet) gas: Saturated gas contains NGLs; after processing, dry residue gas and NGL products result
- **Shrinkage**: Volume of gas lost during processing (NGLs extracted); remaining "dry" residue gas is less than inlet volume

**Gas Plant (NGL) Settlement Methods:**

| Method | How It Works | Revenue Split |
|---|---|---|
| **Net-back** | Processor pays wellhead value = plant-outlet product values minus processing fees and shrinkage | Producer receives calculated wellhead value |
| **Keep-whole** | Processor keeps the liquids, returns equivalent BTU value as dry gas to producer | Producer whole on BTU basis |
| **Percentage-of-proceeds (POP)** | Producer receives a fixed percentage of NGL product sales | Fixed split (e.g., 70/30) |
| **Product retention** | Processor retains a percentage of each product as processing fee | Variable by product |

**Revenue Distribution Flow:**
1. **Revenue deck** received from purchaser (oil run ticket or gas settlement statement)
2. **Revenue distribution (RD)** (`RTRN_RD_INPUT`) allocates gross revenue to each interest owner per DOI decimals
3. **Journal entry:** `DR 120 (Oil/Gas Sales AR)` + `DR 710.011 (Severance Tax LOE)` -> `CR 302 (Revenue Distributions Payable)` + `CR 601/602/603 (Revenue)`
4. **Deductions** applied per owner — severance tax, transportation, processing, marketing (allocated via bearer groups)
5. **Check writing (CW)** (`RTRN_CHK_RGSTR`) generates payment checks to each owner for their net revenue
6. **Severance/royalty reporting** filed with state agencies (TX RRC, TX CPA Gas, NM OCD, OK Tax Commission, etc.)
7. **Revenue in suspense** (Account 304): Amounts held when DOI is missing, disputed, below minimum check threshold, or owner address unknown

### 2.11 Production Taxes

**Severance Tax (Account 320 payable / 710.011 or separate deduction):**
State tax on production at the wellhead. Rates vary by state and product:
- Texas: Oil 4.6% of market value; Gas 7.5% of market value (with exemptions for high-cost, low-producing wells)
- Oklahoma: Gross production tax 7% (oil) / 7% (gas) with various incentive reductions
- New Mexico: Oil 3.75% + conservation tax; Gas rate varies by category
- Tax-exempt owners (certain governmental entities, Indian tribal interests) tracked via `TAX_EXMPT_FL` on DOI; their shares excluded from tax calculation via SOD groups

**Ad Valorem Tax (Account 321):**
Annual property tax on the assessed value of mineral interests, assessed at county level. Typically paid by WI owners. Recorded as LOE (Account 710.019).

**API State Codes:** Used throughout the system for state-level reporting — e.g., TX=42, OK=35, NM=30, LA=22, CO=08, WY=56, ND=38, PA=37.

### 2.12 Gas Imbalances

When gas is produced from a jointly-owned property, each WI owner is entitled to take-in-kind their proportionate share. When owners take more or less than their entitled share, an **imbalance** results.

**Two Imbalance Types:**
1. **Pipeline Gas Imbalances (Producer/Transporter)**: Physical flow vs. nominations; settled via imbalance trading, volumetric makeup, or cash
2. **Producer Gas Imbalances (among WI owners)**: One owner takes/sells different volume than entitled share ("overlift" vs. "underlift")

**Two accounting methods (per EITF 90-22):**

| Method | Revenue Recognition | Balance Sheet Impact | Usage |
|---|---|---|---|
| **Sales (Production) Method** (51%) | Record revenue only when gas is actually sold by the interest owner, regardless of entitlement | No receivable/payable for imbalances; adjust reserves for DD&A to reflect volumes sold vs produced | Simpler; majority practice |
| **Entitlement Method** (49%) | Record revenue based on entitled share of production, not actual sales | **Underlift** (took less than entitled): DR 121 (Gas Imbalance AR). **Overlift** (took more than entitled): CR 306 (Gas Imbalance Payable) | More theoretically correct; matches production |

**SEC Valuation Rules (EITF 90-22) — intentionally conservative:**
- Gas imbalance **receivable**: Valued at the LOWEST of (a) price at time of production, (b) current market price, (c) contract price
- Gas imbalance **payable**: Valued at the GREATEST of the same three prices

**Gas Balancing Agreement (GBA):** Contract among WI owners governing how imbalances are tracked, settled, and limited. May include cash balancing or volumetric balancing. QRA batch process `GasBalancing` handles this.

**Royalty & Tax Treatment Nuance:** Most companies pay royalties on sales basis even when using entitlement method for revenue recognition. Only 12% of companies adjust operating expenses for imbalances under sales method.

### 2.13 Production Costs & Lease Operating Expense (LOE)

**LOE (Account 710)** captures all costs to operate and maintain producing wells after completion:

| Subaccount | Description | Examples |
|---|---|---|
| 710.001 | Supervision & Overhead | Field superintendent salary allocated to property |
| 710.003 | Labor (Operating) | Pumper, roustabout, field operator wages |
| 710.005 | Chemicals/Treating | Corrosion inhibitor, paraffin treatment, scale inhibitor |
| 710.006 | Fuel & Lubricants | Fuel for engines at well site |
| 710.007 | Power & Utilities | Electricity for pumping units and compressors |
| 710.008 | Water Disposal | Produced water hauling and disposal injection |
| 710.009 | Compression | Gas compression at wellhead or gathering system |
| 710.010 | Pumping & Well Service | Pulling unit, swab, rod jobs |
| 710.011 | Severance/Production Tax | Allocated per well/property |
| 710.012 | Saltwater Disposal | Saltwater injection well operation |
| 710.014 | Equipment Rentals | Compressor rental, temporary equipment |
| 710.018 | Insurance | Well/property-specific insurance |
| 710.019 | Ad Valorem Tax | County property tax on mineral interests |
| 710.022 | Other LOE | Catch-all for miscellaneous lifting costs |

**Direct vs Indirect Costs:**
- **Direct costs**: Charged to a specific property (most LOE subaccounts above)
- **Indirect costs** (district/regional overhead): Accumulated in clearing accounts (350=District, 351=Region, 352=Support Facility) and allocated to properties on a periodic basis (monthly). Allocation methods: per-well, production-based, or cost-based
- Clearing account balance should be zero after allocation
- Classification of direct vs. indirect is company-specific and arbitrary — consistency is what matters

**Workover Capitalization Rule (key judgment area):**
- Workover that **adds proved reserves or extends a well's life significantly** -> Capitalize (Account 244 WIP -> 231/233)
- Workover that **restores existing production** (cleaning, pump repair, recompletion to same zone) -> Expense as LOE (710.010 or appropriate subaccount)
- This distinction is a key judgment area and validation rules may enforce policy

**Field Inventory:** Only 38% of E&P companies record field inventory — most treat materials as expense when acquired. When tracked, materials use Account 135 (Materials & Supplies).

### 2.14 Depreciation, Depletion & Amortization (DD&A)

DD&A is calculated using the **unit-of-production (UOP) method** — the only method allowed for oil & gas producing assets under both SE and FC.

**UOP Formula:**
```
DD&A = (Capitalized Cost - Accumulated Amortization) / (End-of-Period Reserves + Current Period Production) x Current Period Production
```

**SE Method — Three Separate Computations:**

| Asset Pool | Amortized Over | Account DR / CR |
|---|---|---|
| **Proved property acquisition costs** (Account 221 leaseholds) | **Total proved reserves** (developed + undeveloped) | DR 726 / CR 226 |
| **Intangible well/development costs** (Account 231) | **Proved developed reserves** only | DR 732 / CR 232 |
| **Tangible well equipment** (Account 233) | **Proved developed reserves** only | DR 734 / CR 234 |

**Additional DD&A Rules:**
- **Dismantlement, Removal & Abandonment (DR&A):** Asset retirement obligation accrual (Account 410) -> amortized over proved developed reserves alongside well costs (DR 735 / CR 235)
- **Joint oil/gas production**: Convert to common unit (BOE) using **6 mcf gas = 1 barrel oil** (energy content basis). Both production and reserves must use same conversion
- **Significant development project exclusion (ASC 932-360-35-19):** If a major development (e.g., offshore platform) has wells still to be drilled, exclude a proportionate share of costs from the amortization base until all planned wells are completed
- **Property groupings for amortization**: Industry practice: 58% by field, 13% by property, 11% by well, 5% by reservoir. Grouping must not be broader than a field (SFAS 19). Grouping choice significantly impacts DD&A amounts
- **Interim period revisions**: Two approaches: (1) revise rate prospectively in the quarter of change, or (2) recalculate year-to-date and true-up
- **Nonoperating interests (royalties, ORRI):** When reserve data is unavailable, commonly amortized **straight-line over 8-10 years**
- **Reserve estimate changes**: Prospective only (APB 20) — no prior period adjustments. Declining reserves + rising costs = increased per-unit amortization (potential income distortion)

**FC Method:** Single country-wide cost center. Amortizable base includes all capitalized costs (successful and unsuccessful) plus estimated future development costs. Amortized over total proved reserves. Subject to ceiling test.

### 2.15 Asset Retirement Obligations (ARO)

Unavoidable future costs to dismantle, remove equipment, and restore/reclaim surface lands. Required by federal/state regulations or contracts.

**Two Accounting Approaches:**
1. **Adjusted Amortization (Negative Salvage Value)**: Accrue ARO over field life via unit-of-production; credit accumulated amortization (not a liability). Can result in negative book value near end-of-life. Majority practice
2. **Liability Accrual Method**: Debit amortization expense, credit liability account. More theoretically sound

**FASB Proposed Standard (now ASC 410):** Recognize ARO liability at acquisition/development using present value (credit-adjusted risk-free rate + profit margin); capitalize corresponding asset cost; amortize systematically; passage of time = interest expense

**Key Nuance:** Avoid double-counting: don't reduce amortization base by salvage value AND reduce future DR&A costs for salvage. Reversionary interests and production payments may create partial ARO obligations.

### 2.16 Farmouts, Carried Interests & Unitizations

**Pooling of Capital Principle (ASC 932-360-25):**
Oil & gas conveyances are accounted for as "poolings of capital" — generally **no gain or loss** is recognized at the time of the transaction.

**Farmout (ASC 932-360-25-13(b)):**
- **Farmee (assignee)** agrees to drill a well on **farmor (assignor)**'s lease in exchange for earning a WI
- Farmor accounting: Leasehold cost of the farmed-out acreage becomes the cost basis of any retained interest (ORRI, back-in WI). If entire lease is farmed out with no retained interest, basis -> $0
- Farmee accounting: Records all drilling costs. Allocates **zero cost** to the mineral interest acquired — cost basis of earned WI = drilling costs only

**Carried Interest:**
- **Carrying party** pays 100% of costs and records 100% of revenues until **payout** (point where carrying party has recovered all costs + penalty from revenue)
- **Carried party** records no revenue or expense until payout
- **Nonconsent-to-carried-interest**: When a JOA partner declines to participate, consenting parties carry the non-consenting party's share at a **penalty (typically 300-500%)**. After penalty recovered from nonconsenter's revenue share, their WI reverts

**Unitization (ASC 932-360-25-13(f)):**
- Multiple lease/tract owners combine interests into a single operating **unit** (field-wide or area-wide)
- Each party receives a **participation factor** (decimal interest in the unit) based on acreage, reserves, or combination
- **Equalization**: Pre-unitization costs equalized via cash payments or disproportionate spending (additional investment, not gain/loss)
- **Redetermination**: Participation factors periodically recomputed; adjustments via volume overtakes/undertakes

**Property Conveyances (Sales):**
- Conveyance classification depends on risk assumption, not contract terminology (substance over form)
- **Production Payments**: May be classified as (1) loan/debt, (2) prepaid commodity sale, (3) volume production payment (VPP treated as mineral interest sale), or (4) outright sale — depending on who bears production and pricing risk
- EITF 88-18 criteria determine financing vs. sale classification

### 2.17 Prior Period Notification (PPN) System

PPNs trigger revenue reversal and recalculation when ownership or allocation changes retroactively. Each PPN has a **Letter Designation (LD) code**:

| LD Code | Name | Trigger | Creates Live PPN? | Opens CA? |
|---|---|---|---|---|
| **LD01** | New DO | New division order interest created | Yes | Yes |
| **LD02** | Interest Change | DOI interest percentage modified | Yes | Yes |
| **LD04** | Pending to Active | DOI status changed from pending to active | Yes | Yes |
| **LD05** | Active to Inactive | DOI deactivated | Yes | Yes |
| **LD18** | Bearer Group Change (SOD) | Bearer group change where SOD owners exist (no TIK) | Yes (RV only) | No |
| **LD19** | No Recoup | `RECOUP_FL = 'N'` on DOI — creates RRV/RRC reversals but NO live PPN, NO BKRVNU submission | **No** | No |
| **LD20** | Bearer Group Change (TIK) | Bearer group change where TIK/RIK owners present — overrides LD18 | Yes | Yes |
| **LD45** | Owner Exception Maintenance | Change to settlement code, SOD group, tax exempt flag, GWI/PPI bearer group, TEG, or MEG | Yes | Depends |

**Key PPN rules:**
- LD19 is the **only PPN exclusion code** — it creates audit-trail reversals without recalculation
- LD18 vs LD20 precedence: If TIK owners are present, LD20 is generated instead of LD18
- LD45 triggers on ANY change to 7 owner-level exception fields on active DOIs

### 2.18 Accounts Payable / Voucher Processing (QCFS)

The AP/voucher workflow in QCFS:

1. **Voucher entry** — invoice received, coded to GL accounts with code blocks (cost center, property, AFE, etc.)
2. **Batch management** — vouchers grouped into batches for processing
3. **Approval workflow** — routed through approval chains (similar to AFE but for expenditures)
4. **Posting** — approved batches posted to GL, updating actuals. `DR Property/WIP/LOE accounts, CR 301 Vouchers Payable`
5. **Check run** — AP payments generated, printed, bank reconciled
6. **Export** — ACH files, positive pay files, bank-specific formats (Wells Fargo, Chase, Citibank, etc.)

**QCFS Action Types:** Save, Validate, Query, Delete, WFSubmit, WFApprove, WFReject, WFRouteAndReturn

### 2.19 Key Industry Terminology Quick Reference

| Term | Definition |
|---|---|
| **Allowable** | Maximum production rate permitted by state regulatory agency (e.g., TX RRC) |
| **API Number** | Unique well identifier assigned by state (2-digit state + 3-digit county + 5-digit well + optional 2-digit sidetrack/completion) |
| **BOE** | Barrel of Oil Equivalent — 6 mcf gas = 1 bbl oil (energy content basis) |
| **BS&W** | Basic Sediment & Water — crude oil impurity measure (<1% acceptable) |
| **Casing Point** | Decision point after drilling to objective depth: complete or plug/abandon |
| **COPAS** | Council of Petroleum Accountants Societies — sets standard accounting procedures for joint operations |
| **Cycling** | Gas reinjection to maintain reservoir pressure and maximize condensate/NGL recovery |
| **Dry Hole** | Well drilled to objective depth with no producible hydrocarbons |
| **EFP** | Exchange of Futures for Physicals — converts financial position to physical delivery |
| **Farmout** | Assignment of WI to a farmee who agrees to drill; farmor retains ORRI or back-in WI |
| **Fee Interest** | Ownership of both surface and mineral rights |
| **Fracturing** | Rock treatment to increase production flow (hydraulic or acid fracturing) |
| **LACT** | Lease Automatic Custody Transfer — automated oil measurement system |
| **Landman** | Professional responsible for identifying mineral-rights owners and negotiating leases |
| **NADOA** | National Association of Division Order Analysts — standardizes division order forms |
| **Nomination** | Producer's monthly forecast of gas volumes to pipeline (reserves capacity) |
| **Payout** | The point at which the carrying party has recovered costs (+ penalty) from revenue; carried interest reverts |
| **Proved Developed Reserves** | Reserves recoverable from existing wells with current equipment — denominator for well/development cost DD&A |
| **Proved Undeveloped (PUD)** | Reserves on undrilled locations — included in total proved but not developed reserves |
| **Shrinkage** | Volume of gas lost during processing when NGLs are extracted |
| **Spud Date** | Date drilling physically begins on a well — starts interest capitalization clock |
| **Take-in-Kind** | Each WI owner physically takes and sells their proportionate share of production directly |
| **Venture** | A grouping of properties under common management/partnership for accounting and billing purposes |
| **VPP** | Volume Production Payment — obligation to deliver a specified quantity of production |
| **Wellbore** | The physical hole drilled; may have multiple completions (zones) |

### 2.20 Key Industry Organizations

| Organization | Role |
|---|---|
| **COPAS** | Develops standardized accounting forms for joint venture accounting |
| **AAPL** | Develops model forms (e.g., Form 610 operating agreements) |
| **NADOA** | Standardizes division order forms and practices |
| **API** | Industry research, training, state code standards |
| **IPAA** | Trade association for independents |
| **SPE/SPEE** | Petroleum engineer societies; maintain reserve definition standards |
| **ONRR** (formerly MMS) | Oversees federal royalty accounting and audits |
| **State Conservation Commissions** | Regulate drilling permits and production reporting (e.g., TX RRC) |

### 2.21 Database Table Naming Conventions

Tables follow a prefix convention indicating domain area:

| Prefix | Domain | Examples |
|---|---|---|
| `GONL_` | General/shared | `GONL_PROP` (Property), `GONL_ACCTG_MTH` (Accounting Month), `GONL_PROP_VNTR` (Property Venture) |
| `PONL_` | Production | `PONL_WELL_COMPL` (Well Completion), `PONL_FLOW_GRID`, `PONL_MP` (Measurement Point) |
| `DONL_` | Division Orders | `DONL_DO_HDR` (DOI Header), `DONL_DO_DETAIL`, `DONL_BEARER_GRP_HDR`, `DONL_DVD_BEARER_GRP_DETAIL` |
| `JONL_` | Journal Accounts | `JONL_ACCT` (Account), `JONL_MANL_JE_DETAIL` (Manual JE) |
| `JTRN_` | Journal Transactions | `JTRN_GL_HIST` (GL History), `JTRN_SL_DETAIL` (Subledger) |
| `RTRN_` | Revenue Transactions | `RTRN_CHK_RGSTR` (Check Register), `RTRN_RD_INPUT` (Revenue Distribution) |
| `SCTRL_` | System Control | `SCTRL_BA_ENTITY` (Business Associate), `SCTRL_BUS_UNIT`, `SCTRL_COST_CNTR` |
| `TONL_` / `TTRN_` / `TSTG_` | Tax | Severance/tax reporting entities |
| `QCTRL_` | QCA Control | `QCTRL_AFE_HDR` (AFE Header), `QCTRL_AFE_DETAIL` |
| `JBCDE_` | JIB Code Definition | `JBCDE_OWNER_RULE`, `JBCDE_BUS_SEG` |
| `RONL_` | Revenue Online | `RONL_RVNU_STAT` (Revenue Status) |
| `DVD_` / `DONL_DVD_` | Division Order (pending) | Pending/draft DOI changes before approval |
| `PXRF_` / `JXRF_` / `SXREF_` | Cross-references | Well-to-Property, Account-to-Group mapping |
| `QSTG_` / `SSTG_` / `JSTG_` | Staging | Import/interface staging tables |
| `AONL_` | Acquisition & Disposition | `AONL_ACQ_DISP_PKG` — property transfer packages |

---

## 3. Architecture & Code Patterns

### 3.1 Layered Architecture

Each product module follows this dependency flow:

```
+-------------------------------------------------------------+
|  Web.Controllers (MVC UI Controllers + REST API Controllers)  |
|  |- QUIController{Feature}.cs  -- UI state, screen logic      |
|  +- {Feature}Controller.cs     -- REST API endpoints          |
+---------------------------------------------------------------+
|  ServiceInterface  (WCF [ServiceContract] partial interfaces)  |
|  +- IQ{Module}Service_{Feature}.cs                             |
+---------------------------------------------------------------+
|  ServiceCore  (Business logic -- partial class implementation) |
|  +- Q{Module}ServiceCore_{Feature}.cs                          |
+---------------------------------------------------------------+
|  Validations  (Numbered rule pipeline per feature)             |
|  +- {Feature}/{FeaturePrefix}{Number}.cs                       |
+---------------------------------------------------------------+
|  DataAccess  (3-tier: Db -> Cache -> Configurable facade)      |
|  +- Q{Module}DataAccess_{Entity}.cs                            |
+---------------------------------------------------------------+
|  DAL  (SQL/stored procedure layer)                             |
|  +- {Entity}DAL.cs -> maps to SQL table                        |
+---------------------------------------------------------------+
|  DataObject  (POCO entities with change tracking)              |
|  +- {Entity}DO.cs + {Entity}DOExt.cs                           |
+---------------------------------------------------------------+
```

**QCFS additionally has** two legacy layers not present in QCA/QDO/ESuite:
- **BL (Business Logic layer)**: DataSet-based domain objects (`BaseBL -> BaseBLExt -> MasterfileBL -> VoucherJournal`). Each entity wraps typed DataSets
- **BS (Business Service layer)**: Transaction orchestration (`BsTransaction -> VoucherTransaction`, `BsWorkflow -> VoucherWorkflow`). Handles posting pipelines, batch building, and bank-specific exports
- **BLScript**: Lua scripting engine (NLua) for runtime-customizable business logic without recompilation

### 3.2 Key Class Hierarchies

**Service Layer:**
```
QServiceCoreBase (Quorum.QFC.Metadata.ServiceServer)
  |- QCAServiceCore   : IQCAService   (partial, 46 feature files)
  |- QCFSServiceCore  : IQCFSService  (partial, 18 feature files)
  |- QDOServiceCore   : IQDOService   (partial, 50+ feature files)
  +- QESUITEServiceCore : IQESUITEService (partial, 50+ feature files)
```

**DI Container:**
All feature interfaces resolve to the same singleton ServiceCore instance. Data access interfaces create new instances per resolve.

**Validation Framework:**
```
QFCValidationBase<T> (QFC framework)
  |- QCAValidationBase<T>    -> AFEValidationBase<T>    -> AFEMaint000010..000690
  |- QCFSValidationBase<T>   -> APWFValidationBase<T>   -> APWFVoucher000010..000640
  |- QDOValidationBase<T>    -> QDODonlDoHdr_000001..000037
  +- QESUITEValidationBase<T>-> QESUITEValidationBAEntity0001..0087
```

**DataObject Hierarchy:**
```
BaseDO (QFC framework)
  |- ControlTableBaseDO -> AfeHeaderDO, DonlDoHdrDO, BusinessAssociateDO
  |- CodeTableBaseDO    -> GcdeDoTypeDO, GcdeBusUnitBaDO
  |- TransactionBaseDO  -> JtrnSlDetailDO, RtrnChkRgstrDO
  |- ReadOnlyViewBaseDO -> view-based query DOs
  |- CrossRefTableBaseDO
  +- StagingTableBaseDO
```

**Controller Hierarchy:**
```
QInterfaceControllerBase (QFC.Web)
  |- QCAInterfaceControllerBase -> AFEInterfaceControllerBase -> QUIControllerAFEMaintenance
  |- QCFSInterfaceControllerBase -> QUIControllerVoucher
  |- QDOInterfaceControllerBase -> QUIControllerDonlDoHdr
  +- QESUITEInterfaceControllerBase -> QUIControllerBusinessAssociate
```

### 3.3 Naming Conventions

| Layer | File Pattern | Example |
|---|---|---|
| **REST API Controller** | `{Feature}Controller.cs` in `APIControllers/` | `AuthorizationForExpenditureController.cs` |
| **MVC UI Controller** | `QUIController{Feature}.cs` | `QUIControllerVoucher.cs` |
| **Service Interface** | `IQ{Module}Service_{Feature}.cs` (partial) | `IQCAService_AFEMaintenance.cs` |
| **Service Core** | `Q{Module}ServiceCore_{Feature}.cs` (partial) | `QCAServiceCore_AFEMaintenance.cs` |
| **Validation Rule** | `{Prefix}{Number}.cs` | `AFEMaint000010.cs`, `QDODonlDoHdr_000013.cs` |
| **Data Object (generated)** | `{Entity}DO.cs` in `CodeGen/` | `AfeHeaderDO.cs` |
| **Data Object (ext)** | `{Entity}DOExt.cs` | `AfeHeaderDOExt.cs` |
| **DAL (generated)** | `{Entity}DAL.cs` in `CodeGen/` | `AfeHeaderDAL.cs` |
| **Constants** | `Q{Entity}Constants.cs` | `QAfeHeaderConstants.PropertyNames.BusUnitCode` |
| **DI Container** | `Q{Module}ServiceServerContainer.cs` | In ServiceCore project root |
| **Batch Process Step** | `QPS{ProcessName}.cs` or `QSegregated{ProcessName}.cs` | `QPSJIBStatementPrint.cs` |
| **Screen Registration** | `ScreenRegistrationSpecs.cs` | In `Web.Core/ScreenRegistration/` |

### 3.4 Code Generation

Significant portions of DO, DAL, DataAccess, and DataAccessInterface code are **auto-generated**:
- Generated code lives in `CodeGen/` subdirectories within each project
- Hand-written extensions live in `*Ext.cs` partial class files alongside or above `CodeGen/`
- **Never modify files in `CodeGen/` directories** — they will be overwritten
- Code generation is driven by XML definitions in `Quorum.QFC.CodeGenTemplates`

---

## 4. Project Map

### 4.1 QCA Web Solution (13 projects)
**Path:** `Quorum.Upstream.QCA.Web/Quorum.Upstream.QCA.Web.sln`

**QCA Features:** AFE (AfeAcct, AFEInbox, AFEMaintenance, AfePrdDetail, AFESeqDef, AFESubLedger, AFESubscr, AFETransactionCommitCost, AfeTypeSetup, BulkAfeHeader), JE (JEBilling, JECostSplit, JETransfer, JEImport, JERateMaintenance, JERawData, etc.), MaterialTransfer, Security

### 4.2 QCFS Web Solution (28 projects)
**Path:** `Quorum.Upstream.QCFS.Web/Quorum.Upstream.QCFS.Web.sln`

**QCFS Features:** APBatchSubmittal, APInbox, Voucher (largest at 5,230 lines), Invoice, Payments, PostWeb, VendorContract, BatchBankReconDetail, BatchDepositCreation, BatchJournal

**Note:** QCFS uses dual-controller pattern and `Quorum.QCFS.New.*` namespace for CodeGen types to disambiguate from legacy `Quorum.QCFS.*` BL/DATA types.

### 4.3 QDO Web Solution (12 projects)
**Path:** `Quorum.Upstream.QDO.Web/Quorum.Upstream.QDO.Web.sln`

**QDO Features:** DonlDoHdr, DonlDoDetail, DOICopy, DOICreation, DOIMaintenance, DOIPending, BearerGrpSetup, MarketGroup, UnitToTract, WorkspaceMaintenance

### 4.4 ESuite Web Solution (46 projects)
**Path:** `Quorum.ESuite.Web/Quorum.Esuite.Web.sln`

**ESuite Features:** BusinessAssociate (87 validation rules), Contacts, CostCenters, GLEntries, OrgHierarchies, Meters, Contracts

### 4.5 Shared Web Solution (28+ projects)
**Path:** `Quorum.Upstream.Shared.Web/Quorum.Upstream.Shared.Web.sln`

Contains DataObject, DataAccess, DataAccessInterface, DAL, and ViewModels for all modules.

**Key file:** `Quorum.Upstream.Shared.CoreInterface/QUpsGlobalConstants.cs` (844 lines) — defines CostCenterTypes, DBTableNames, DBColumnNames.

### 4.6 Database Solution
**Path:** `Quorum.Upstream.Database/Quorum.Upstream.Database.sln`

**FluentMigrator 3.2.16** with timestamped ordered migrations.
**Database owners:** QCA (~400 migrations), QCFS, QRA, QFCENGS (ESuite primary), QFCUPS (ESuite secondary), WF (Workflow), UPSARCV (Archive), UPSSCR (Screen)

### 4.7 ClassicGUI Repositories

| Module | Repo | Screen Count | Key Areas |
|---|---|---|---|
| **QCA** | `Quorum.Upstream.QCA.ClassicGUI/` | ~114 | AFE, JIB, Code Tables, Fixed Assets, Lease Operating, Payout |
| **QCFS** | `Quorum.Upstream.QCFS.ClassicGUI/` | ~249 | AP, AR, Bank Recon, GL, Masterfiles, System/Security |
| **QRA** | `Quorum.Upstream.QRA.ClassicGUI/` | **~456** | 20 areas — largest ClassicGUI repo |

**QRA ClassicGUI Area Reference:**

| Abbrev | Full Name | Screens | Purpose |
|---|---|---|---|
| **AD** | Acquisition & Disposition | 28 | Property transfer packages |
| **CA** | Contractual Allocation | 16 | Gas plant volume allocation |
| **CI** | Check Input | 17 | Revenue check receipt entry |
| **CW** | Check Writing | 26 | Owner payment disbursement, 1099, escheat |
| **DO** | Division Orders | **51** | DOI management, bearer groups, PPN triggers |
| **FL** | Flow Lines | 6 | Meter/pipeline infrastructure |
| **GB** | Gas Balancing | 6 | Gas imbalance tracking |
| **JE** | Journal Entries | 29 | GL/subledger, manual JE |
| **MF** | Masterfile | **~58** | Property, price/tax terms, formulas |
| **PN** | PPN | 10 | Prior period adjustments |
| **PO** | Payout | 19 | Carried interest payout tracking |
| **RD** | Revenue Distribution | 5 | Owner-level distribution history |
| **SP** | Supply Processing | 5 | Gas supply volumes, shrinkage |
| **TO** | Tax Output (MMS/ONRR) | 5 | Federal OGOR volumetric reporting |
| **TR** | Tax Reporting (Royalty) | 15 | Federal/state royalty reporting |
| **TS** | Tax/Severance Setup | ~45 | State tax master data |
| **TV** | Tax Volumes | ~42 | State volumetric tax reporting |
| **VA** | Volumetric Allocation | 28 | Production hierarchy: fields, wells, flow grids |
| **VL** | Volumes/Valuation | 35 | Revenue valuation, manual volume input, DRI |

### 4.8 Batch Process Architecture (QPEC)

All batch processes inherit from `QPSUpstreamBase -> QProcessStepBase`. Segregated variants run in parallel per business unit.

**QRA Batch** (13 process projects — largest): BookRevenue, CheckWrite, ContractualAllocation, DivisionOrder, Estimates, GasBalancing, MarketDeduction, RevenueAcctgVA, RoyaltyReporting, SeveranceReporting, SAP integration

---

## 5. Build & Test

**Runtime:** .NET Framework 4.8 | **IDE:** Visual Studio 2022 Enterprise | **MSBuild:** 17.10.4

All projects use **packages.config** NuGet format. Always restore with `-p:RestorePackagesConfig=true` before building.

```bash
# 1. cd into the solution directory first
cd "<SolutionDir>"

# 2. NuGet Restore (REQUIRED)
"C:/Program Files/Microsoft Visual Studio/2022/Enterprise/MSBuild/Current/Bin/MSBuild.exe" "<Solution>.sln" -t:Restore -p:RestorePackagesConfig=true -p:Configuration=Debug -verbosity:minimal

# 3. Build
"C:/Program Files/Microsoft Visual Studio/2022/Enterprise/MSBuild/Current/Bin/MSBuild.exe" "<Solution>.sln" -t:Build -p:Configuration=Debug -verbosity:minimal

# 4. Run Tests
dotnet test "<TestProject>/bin/Debug/<TestProject>.dll" --logger "console;verbosity=minimal"
```

**Test Framework:** MSTest v2 with Moq 4. Test data from Excel `.xlsx` files. Test naming: `{RuleName}_{Scenario}_{P|F}`.

---

## 6. Azure DevOps Integration

**Organization**: `https://dev.azure.com/QuorumSoftware`

**Available MCP Tools**:
- `azure-devops_wiql-workitems`: Query work items using WIQL
- `azure-devops_get-workitem-history`: Get full history AND discussion comments
- `azure-devops_add-workitem-comment`: Add a comment to a work item
- `azure-devops_update-workitem`: Update work item fields/state
- `azure-devops_projects`: List available ADO projects

> **Do NOT use `azure-devops_get-workitem-comments`** — returns 404 errors. Always use `azure-devops_get-workitem-history` for comments.

### 6.1 Database Query Policy

**Default**: Query CORE environments only (DEV and SUP). Do NOT query client databases unless explicitly instructed.

| Environment | Server | Purpose |
|---|---|---|
| **CORE REL 17** | `QHOUDB32.QDEV.NET\SQL02` | Primary — CORE_REL assessments |
| **CORE DEVA1** | `qdddevsql12\SQL2022` | Latest dev state, schema changes |
| **CORE SUP17** | `QHOUDB33\SQL02` | Support/hotfix branch state |
| **CORE SUPA1** | `QDSCORSQL01\SQL2022` | A1 support state |
| **CORE TST17** | `QDDDEVSQL03.QDEV.NET\SQL2019` | Test environment |

**Database naming convention**: `{ENV}UPS_{MODULE}` — e.g., `CORE_REL17UPS_QRA`, `CORE_DEVA1UPS_QCA`

---

## 7. CI/CD & Validation

- **Azure Pipelines** with centralized templates from `Quorum.Pipeline.Templates` repo
- **Branching**: GitFlow — `develop` (alpha), `release/*` (beta), `master`/`support/*` (GA), `hotfix/*`, `feature/*`
- **Versioning**: GitVersion (current: 17.8.0)
- **NuGet feeds**: Private Azure DevOps feeds + nuget.org
- **Code coverage target**: 80%

---

## 8. Search Discipline & Anti-Blindspot Rules

### 8.1 Dual Implementation Architecture

This codebase has **parallel implementations** for the same functionality. Many batch processes exist in BOTH modern C# and legacy C++. **Never assume a process exists in only one implementation layer.**

| Layer | Technology | Repository Pattern | Extensions |
|---|---|---|---|
| Modern Web/API | C# / .NET 4.8 | `*.Web/`, `*.Batch/`, `Shared.Web/` | `.cs` |
| Legacy Batch | C++ / Win32 | `*.ClassicBatch/` | `.cpp`, `.h` |
| Legacy GUI | C# / WinForms | `*.ClassicGUI/` | `.cs` |
| Database | SQL Server | `Upstream.Database/` | `.sql` |

### 8.2 Mandatory Search Coverage

When searching for ANY database table, column, process, or feature, ALL of these must be checked:
- `**/*.cs` — Modern C# code
- `**/*.cpp` — Legacy C++ code (QSQL_*.cpp contain embedded SQL as string literals)
- `**/*.sql` — Database migration scripts
- `**/ClassicGUI/**/*.cs` — WinForms screens
- `**/ClassicBatch/**/*.cpp` — Legacy batch processes
- QCFS-specific: `BL/`, `BS/`, `Upstream/`, `DATA/` (`.xsd`)
- Cross-module: `ESuite.Integration.{Module}.*`

### 8.3 C++ SQL Repository Pattern

There are **541 QSQL_*.cpp files**. SQL is built as C++ string concatenation — table names appear as **string fragments**, not typed references. SQL IDs defined in companion `QSQLID_*.h` files. Some tables are ONLY referenced in C++ code.

### 8.4 Anti-Assumption Rules

1. **Never assume implementation language** from database registration alone
2. **Never assume modern repo contains all functionality** — check ClassicBatch alongside Batch
3. **Never limit file extension** to `.cs` when searching for SQL or table references
4. **Never conclude "not found"** until ALL file types and ALL repos have been searched
5. **Verify file identity** — confirm the file matches the target functionality before deep analysis

---

## 9. Quick Reference: Finding Code

| To find... | Look in... |
|---|---|
| Business logic for a feature | `{Module}.ServiceCore/Q{Module}ServiceCore_{Feature}.cs` |
| Validation rules | `{Module}.Validations/{Feature}/{Prefix}{Number}.cs` |
| Data object definition | `Shared.Web/Quorum.{Module}.DataObject/CodeGen/{Entity}DO.cs` |
| Data access layer | `Shared.Web/Quorum.{Module}.DAL/` |
| UI controller | `{Module}.Web.Controllers/QUIController{Feature}.cs` |
| REST API controller | `{Module}.Web.Controllers/APIControllers/{Feature}Controller.cs` |
| Database migration SQL | `Upstream.Database/FluentMigrator/EmbeddedScripts/MSSQL/{DbOwner}/` |
| WinForms screen (QRA) | `QRA.ClassicGUI/Quorum.Upstream.QRA.{Area}/QFrm{Screen}.cs` |
| Batch process (C#) | `{Module}.Batch/` or `Shared.QPECProcessSteps/` |
| Batch process (C++ legacy) | `{Module}.ClassicBatch/QPDll{Area}/QPS*.cpp` |
| SQL in C++ batch | `{Module}.ClassicBatch/QPDll{Area}/QSQL_*.cpp` |
| Global constants | `Shared.Web/Shared.CoreInterface/QUpsGlobalConstants.cs` |
| Cross-module DB access | `ESuite.Web/ESuite.Integration.{Module}.*` |
| PPN documentation | `Documentation/LD*_*.md` |
