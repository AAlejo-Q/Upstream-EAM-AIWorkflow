---
name: safe-popm-best-practices
description: "SAFe Product Owner/Product Manager best practices for creating Features and User Stories in Azure DevOps. Use this skill whenever creating, writing, or refining Features, Requirements, User Stories, Epics, or backlog items. Also use when discussing WSJF prioritization, acceptance criteria, benefit hypotheses, story splitting, estimation, PI planning, or any SAFe/Agile product management concepts. Trigger this skill for any work item creation in ADO projects following SAFe methodology, especially the EAM (Enterprise Accounting Modernization) project."
---

# SAFe POPM Best Practices Skill

> **Source:** SAFe® Product Owner/Product Manager Workbook (2025.09.16)
> **Primary Use:** Structuring and generating Features and Requirements in Azure DevOps

## When to Use This Skill

- Creating or refining **Features** in ADO
- Creating or refining **Requirements** (User Stories) in ADO
- Writing **benefit hypotheses** or **acceptance criteria**
- Applying **WSJF prioritization** to backlog items
- Splitting large stories or features
- Discussing SAFe roles, events, or artifacts
- Any backlog management in SAFe context

---

## Quick Reference

### SAFe Work Item Hierarchy

```
Epic → Feature → Requirement (Story)
```

| Level | Owner | Timeframe | ADO Type |
|-------|-------|-----------|----------|
| Epic | Portfolio | Multi-PI | Epic |
| Feature | Product Management | 1 PI (8-12 weeks) | Feature |
| Story | Product Owner | 1 Iteration (2 weeks) | **Requirement** |

> **Note:** QuorumSoftware ADO uses "Requirement" instead of "User Story"

---

## Feature Essentials

### Seven Required Elements

1. **Benefit Hypothesis** — "If [capability], then [measurable outcome]"
2. **Acceptance Criteria** — Testable conditions (4-6 items)
3. **NFRs** — Performance, security, compliance requirements
4. **WSJF Enablement** — Enough info for prioritization
5. **Right-Sized** — Completable in one PI
6. **Dependencies & Risks** — Documented blockers
7. **Traceability** — Links to parent Epic, child stories

### Feature Naming Convention

```
[ScreenID] - [Brief Descriptive Name]
```

Example: `VL032 - Post Prior Period Notification Screen`

### Golden Rule

> Features describe **WHAT** and **WHY** — never HOW.

---

## Requirement (Story) Essentials

### User Voice Format

```
As a [user role],
I want to [capability],
So that [business value].
```

### INVEST Criteria

| I | N | V | E | S | T |
|---|---|---|---|---|---|
| Independent | Negotiable | Valuable | Estimable | Small (≤8 pts) | Testable |

### Story Naming Convention

```
[FeatureID]-[Seq]: [Brief action description]
```

Example: `VL032-01: Display prior period adjustment listing`

---

## Acceptance Criteria — Gherkin Format

Always use Given-When-Then syntax:

```gherkin
Given [precondition/context],
When [action/trigger],
Then [expected outcome].
```

**Example:**
```gherkin
Given I am on the VL032 screen,
When the screen loads,
Then I see a grid displaying all pending prior period adjustments.
```

---

## WSJF Prioritization

### Formula

```
WSJF = Cost of Delay ÷ Job Size
```

**Cost of Delay** = User-Business Value + Time Criticality + Risk Reduction/Opportunity Enablement

### Scale

```
1, 2, 3, 5, 8, 13, 20
```

**Rules:**
- Do one column at a time
- Each column must have at least one "1" (the smallest)
- Higher WSJF = Do first

---

## Story Point Estimation

### What Story Points Measure

- **Volume** — How much work?
- **Complexity** — How difficult?
- **Knowledge** — What do we know?
- **Uncertainty** — What's unknown?

### Sizing Guide

| Points | Guidance |
|--------|----------|
| 1-3 | Well-understood, small |
| 5 | Medium, some unknowns |
| 8 | Large but fits in iteration |
| 13+ | **Split the story** |

---

## Enablers and Spikes

### Enablers

Technical work that builds the architectural runway:
- **Exploration** — Research, prototyping
- **Architecture** — Technical design
- **Infrastructure** — Platform capabilities
- **Compliance** — Regulatory/security

### Spikes

Time-boxed research to reduce risk or increase estimate reliability.

**When to use:**
- New technology evaluation
- Performance concerns
- Uncertain estimates
- Multiple architectural options

**Spike title format:** `Spike - [Research objective]`

---

## Templates

For detailed HTML templates for ADO Description fields, see:
- `references/feature-template.md` — Full Feature template
- `references/requirement-template.md` — Full Requirement (Story) template

---

## Anti-Patterns to Avoid

### Feature Anti-Patterns

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| Solution as feature ("Add caching") | Outcome focus ("Load time < 3s") |
| Missing benefit hypothesis | Add "If... then... by [metric]" |
| Vague criteria ("fast", "user-friendly") | Specific, testable criteria |
| Too large for one PI | Decompose into smaller features |

### Story Anti-Patterns

| ❌ Don't | ✅ Do Instead |
|----------|---------------|
| Component story ("Build database layer") | User-focused story |
| > 8 story points | Split the story |
| No acceptance criteria | Add Given/When/Then |
| Technical task framing | Frame as enabler with outcome |

---

## EAM Project Context

**Epic #1786991:** Enterprise Accounting Modernization - Desktop to Web - QRA MVP

### Domain Terms

| Term | Meaning |
|------|---------|
| PPA | Prior Period Adjustment |
| DRI | Decimal Right Interest |
| QDO | Quorum Division Order module |
| QRA | Quorum Revenue Accounting module |
| MCP | Model Context Protocol (AI agent tools) |

### Key Personas

| Persona | Focus |
|---------|-------|
| Revenue Accountant | Accuracy, efficiency, audit trail |
| Auditor | Compliance, traceability |
| System Administrator | Security, monitoring |

---

## References

For detailed templates and examples, read:
- `/references/feature-template.md` — HTML template for Feature descriptions
- `/references/requirement-template.md` — HTML template for Requirement descriptions
- `/references/wsjf-examples.md` — WSJF calculation examples