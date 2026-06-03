# SAFe Templates for EAM Work Items

## Epic Template

```markdown
### Epic Hypothesis Statement

**Funnel entry date**: [Date]
**Epic name**: [Name]
**Epic Owner**: [Name]

**Epic description**:
For [target customers/users]
Who [statement of need or opportunity]
The [solution name]
Is a [solution category]
That [key benefit/capability]
Unlike [current alternative/competitor]
Our solution [primary differentiator]

**Business outcomes** (measurable, time-bound):
- [Outcome 1 with metric and timeframe]
- [Outcome 2 with metric and timeframe]

**Leading indicators** (early signals of success):
- [Indicator 1]
- [Indicator 2]

**Nonfunctional requirements**:
- [NFR 1: performance, security, scalability, etc.]
- [NFR 2]
```

### Epic Example (EAM)

```markdown
### Epic Hypothesis Statement

**Funnel entry date**: January 28, 2026
**Epic name**: Desktop to Web QRA MVP
**Epic Owner**: Sarah Paul

**Epic description**:
For revenue accountants and land analysts at oil & gas companies
Who currently depend on Citrix-hosted WinForms for daily revenue processing
The MyQuorum Accounting Web Application
Is a modern Blazor Server replacement
That delivers browser-based access to all critical DRI, JE, and CW workflows
Unlike the current QCloud/Citrix-dependent Classic GUI
Our solution eliminates remote desktop overhead, enables API-first automation,
and supports AI-assisted month-end close via MCP tools.

**Business outcomes**:
- 100% of pilot customer month-end close completed via web app within 6 months of launch
- 50% reduction in Citrix licensing costs within 12 months
- Zero data regression vs. ClassicGUI for migrated screens

**Leading indicators**:
- 5 DRI screens at full parity within first PI
- Pilot customer completes one full month-end cycle on web by Q4 2026

**Nonfunctional requirements**:
- Screen load <2s, query <3s for 10K rows
- QARCH_SEC_* security model fully replicated
- Azure SQL with <100ms latency to app tier
```

---

## Feature Template

```markdown
### Feature: [Verb + Noun — clear, concise name]

**Parent Epic**: [Epic ID and name]
**Feature ID**: [ADO ID]
**Priority**: [P1/P2/P3]
**Size**: [S/M/L/XL relative to PI capacity]

**Benefit Hypothesis**:
[Who] benefits from [what capability] by [how it helps them],
resulting in [measurable business outcome].

**Acceptance Criteria**:
1. [Testable criterion 1]
2. [Testable criterion 2]
3. [Testable criterion 3]
...

**Design Approach**: [Lift and Shift / Redesign / Combine]
**Source**: [ClassicGUI form name or Web view path]
**Primary Table(s)**: [Database table names]
**Business Rules**: [Count and key rule names]
**Dependencies**: [Other features or enablers required]
```

### Feature Example (EAM Screen Migration)

```markdown
### Feature: Migrate VL025 — DRI Link Maintenance to Blazor

**Parent Epic**: 1786991 — Desktop to Web QRA MVP
**Feature ID**: 1786972
**Priority**: P1
**Size**: XL (22 business rules, effective-date logic, mass changes, PPN integration)

**Benefit Hypothesis**:
Revenue accountants benefit from browser-based DRI Link Maintenance by eliminating
Citrix dependency for their most complex daily configuration screen, reducing setup
time for new remitter mappings by 30% through improved UX and smart defaults.

**Acceptance Criteria**:
1. Filter-detail query against RONL_MI_LINK_EFF_DT replicates all legacy filter criteria
2. Inline grid editing supports add/edit/delete with row state tracking
3. All 22 registered business rules pass validation (see DRI module reference)
4. Effective date overlap check prevents invalid date ranges
5. Mass Changes dialog supports End Date, End Date + Override, and End Date + Clone modes
6. PPN records created on insert/modify/delete matching legacy INSERT_PPN_VL025 behavior
7. Read-only state enforced for VL-finalized and MI-processed records
8. RXRF_MI_SOD updated when effective dates change
9. Security: QFRMMILINKMAINTENANCE object-level and field-level permissions enforced
10. Performance: Query returns within 3s for 500-row result set

**Design Approach**: Lift and Shift
**Source**: QFrmMILinkMaintenance.cs (ClassicGUI VL module)
**Primary Table**: RONL_MI_LINK_EFF_DT
**Business Rules**: 22 registered + 12 standalone rule classes in BusinessRules/ folder
**Dependencies**: Feature 1786989 (Web App Framework), Feature 1786990 (Data Access Layer),
Feature 1786977 (Security/Access Control)
```

---

## User Story Template

```markdown
### User Story

**Title**: As a [role], I want [activity] so that [business value]
**Parent Feature**: [Feature ID and name]
**Story Points**: [1/2/3/5/8/13]
**Iteration**: [Target iteration or "Backlog"]

**Description**:
[2-3 sentences providing additional context. Reference the persona,
the screen, and the specific workflow step this story addresses.]

**Acceptance Criteria** (Given-When-Then):

**AC1**: [Short description]
- Given [precondition]
- When [action]
- Then [expected result]

**AC2**: [Short description]
- Given [precondition]
- When [action]
- Then [expected result]

**AC3**: [Short description]
- Given [precondition]
- When [action]
- Then [expected result]

**Technical Notes**:
- Source: [ClassicGUI method or form reference]
- Table(s): [Database tables involved]
- Business Rule(s): [Specific rule IDs if applicable]

**Definition of Done**:
- [ ] Acceptance criteria verified
- [ ] Unit tests written and passing
- [ ] Code reviewed
- [ ] Deployed to staging
- [ ] Accepted by Product Owner
```

### User Story Example (EAM)

```markdown
### User Story

**Title**: As a revenue accountant, I want to filter DRI Links by remitter, product,
and effective date so that I can quickly find the mappings I need to review

**Parent Feature**: 1786972 — Migrate VL025 DRI Link Maintenance
**Story Points**: 5
**Iteration**: PI1.1

**Description**:
The DRI Link Maintenance screen (VL025) uses a filter-detail pattern where the revenue
accountant enters search criteria in a filter panel, then views matching RONL_MI_LINK_EFF_DT
records in an editable grid. This story implements the filter panel and read-only query
results. The legacy screen supports advanced mode and as-of-date (effective date) logic.

**Acceptance Criteria**:

**AC1**: Basic filter query
- Given the VL025 Blazor page is loaded
- When I enter a Remitter Number and click Query
- Then the results grid displays matching MI Link records from RONL_MI_LINK_EFF_DT

**AC2**: Effective date filtering
- Given I have entered filter criteria including an As-Of Date
- When I execute the query
- Then only records where EFF_DT_FROM <= As-Of Date <= EFF_DT_TO are returned

**AC3**: Max rows and pagination
- Given there are more than 500 matching records
- When the initial query completes
- Then the first 500 records are displayed with a "More" button to load additional pages

**AC4**: Empty results
- Given I enter filter criteria that match no records
- When I execute the query
- Then a "No records found" message is displayed and the grid is empty

**Technical Notes**:
- Source: QFrmMILinkMaintenance.Designer.cs — uses SELECT_RONL_MI_LINK_EFF_DT with MoreAll (500 rows)
- Table: RONL_MI_LINK_EFF_DT (aliased as L in SQL)
- Filter fields: EFF_DT_FROM, EFF_DT_TO with as-of-date logic, advanced mode toggle

**Definition of Done**:
- [ ] Acceptance criteria verified
- [ ] Unit tests for filter parameter binding
- [ ] Integration test for query with sample data
- [ ] Code reviewed
- [ ] Deployed to staging
- [ ] Accepted by Product Owner
```

---

## Enabler Story Template

```markdown
### Enabler Story

**Title**: [Technical objective — clear what will be accomplished]
**Type**: [Exploration / Architecture / Infrastructure / Compliance]
**Parent Feature**: [Feature ID and name]
**Story Points**: [1/2/3/5/8/13]

**Description**:
[What technical capability is being built and why it's needed for future features.]

**Acceptance Criteria**:
1. [Testable technical criterion]
2. [Testable technical criterion]
3. [Documentation or artifact produced]

**Spike Output** (if exploration type):
- [ ] Decision documented
- [ ] Trade-offs analyzed
- [ ] Recommendation presented to team
```

---

## Quality Checklists

### Epic Quality Checklist
- [ ] Has measurable business outcomes with timeframes
- [ ] Leading indicators defined for early validation
- [ ] NFRs specified
- [ ] Fits within multi-PI timeframe
- [ ] Value stream identified
- [ ] Budget/investment horizon clear

### Feature Quality Checklist
- [ ] Benefit hypothesis identifies WHO and HOW
- [ ] 3-7 testable acceptance criteria
- [ ] Sized to fit within a single PI
- [ ] Design approach specified (lift-and-shift vs redesign)
- [ ] Source form and primary tables identified
- [ ] Business rule count known
- [ ] Dependencies explicitly listed
- [ ] WSJF scoring inputs available

### User Story Quality Checklist (INVEST)
- [ ] **I**ndependent — Can be developed without other stories in progress
- [ ] **N**egotiable — Scope can flex; doesn't over-specify implementation
- [ ] **V**aluable — Delivers value to a user or the business
- [ ] **E**stimable — Team can estimate with reasonable confidence
- [ ] **S**mall — Fits in a single iteration (2 weeks)
- [ ] **T**estable — Acceptance criteria are verifiable

### Acceptance Criteria Quality Checklist
- [ ] Written in Given-When-Then format
- [ ] Each AC tests one behavior
- [ ] No implementation details (tests WHAT not HOW)
- [ ] Covers happy path AND edge cases
- [ ] Includes error/validation scenarios
- [ ] Automatable as E2E or integration test

---

## Story Splitting Patterns (Most Useful for EAM)

### 1. Workflow Steps
Split a screen migration into: Query → CRUD → Business Rules → Batch Integration

### 2. Business Rule Variations
Each business rule becomes its own story:
- "As a revenue accountant, I want effective date overlap validation so that I cannot create conflicting MI Link records"
- "As a revenue accountant, I want Division Order approval validation so that MI Links are only created for approved DOIs"

### 3. Simple → Complex
- Story 1: Read-only query with grid display
- Story 2: Add inline editing (add/modify/delete)
- Story 3: Add child grid relationships
- Story 4: Add mass changes functionality

### 4. Data Entry Methods
- Story 1: Manual inline grid entry
- Story 2: Pick list / lookup integration
- Story 3: Bulk upload from file

### 5. Spike First
- Spike: "Investigate optimal approach for migrating effective-date overlap logic to Blazor"
- Then: Implementation stories based on spike findings

---

## WSJF Scoring Template

```markdown
| Feature | User-Biz Value | Time Crit | RR/OE | CoD | Job Size | WSJF |
|---------|---------------|-----------|-------|-----|----------|------|
| [Name]  | [1-20]        | [1-20]    | [1-20]| [sum]| [1-20]  | [CoD/Size] |
```

Rules:
- Score one column at a time across ALL features being compared
- At least one item must be scored "1" in each column
- Use Fibonacci-like scale: 1, 2, 3, 5, 8, 13, 20
- Higher WSJF = do first
