# EAM Project — Role Personas & Interaction Guide

> **Purpose:** This document defines the four critical roles on the EAM (Enterprise Asset Management) project. Use it as the primary reference for tailoring responses, expectations, and collaboration style depending on which persona you are interacting with.

---

## 1. Product Manager (PM)

### Identity

| Attribute | Detail |
|---|---|
| **SAFe Accountability** | ART-level content authority |
| **Reports to** | Portfolio / Business Owner |
| **Key Partners** | Product Owner, System Architect, Release Train Engineer, Business Owners |

### Core Responsibilities

- **Owns the ART backlog.** Has content authority over features and enablers at the ART level; decides what goes in, what comes out, and in what order.
- **Defines product strategy, vision, and roadmaps.** Creates and communicates the solution vision that aligns with portfolio strategic themes. Maintains both the solution roadmap (multi-year) and the PI roadmap (near-term, higher fidelity).
- **Explores markets and users.** Conducts primary and secondary research, applies market segmentation, identifies market rhythms and events, and deeply understands customer needs.
- **Connects with the customer.** Knows the customer, knows the stakeholders, identifies problems to be solved, and develops whole-product solutions.
- **Facilitates Continuous Exploration.** Drives the hypothesize → collaborate & research → architect → synthesize loop that feeds the ART backlog.
- **Manages and prioritizes the ART backlog.** Guides feature creation, calculates and applies WSJF for economic prioritization, establishes feature acceptance criteria, and collaborates with the System Architect on capacity allocation between business features and enablers.
- **Delivers value.** Defines and quantifies value for the ART; negotiates scope during PI Planning; contributes to business value scoring with Business Owners.
- **Gets and applies feedback.** Tests benefit hypotheses, obtains feedback from customers and stakeholders, shares feedback with the ART, and evolves solution design.

### PI Planning Role

- Presents the **product/solution vision** and **top 10 features** on Day 1.
- Socializes features before PI Planning so there are no surprises.
- Negotiates scope with teams during breakouts.
- Participates in the management review and problem-solving session.
- Accepts team PI objectives and assigns business value with Business Owners on Day 2.
- Provides feedback on ART PI risks and participates in the confidence vote.

### What the PM Cares About (When Interacting)

- Strategic alignment — does the work trace back to portfolio themes and the vision?
- Market timing — are we releasing at the right moment given market rhythms and events?
- Economic prioritization — is WSJF being applied consistently?
- Feature-level outcomes — are benefit hypotheses being validated?
- Roadmap integrity — are we on track against committed and forecasted milestones?

### How to Tailor Responses for the PM

- Frame everything in terms of **customer value and business outcomes**, not technical implementation.
- Reference strategic themes, vision, and roadmap alignment.
- Speak in features, epics, and benefit hypotheses — not stories or tasks.
- Provide WSJF-ready information (user-business value, time criticality, risk reduction / opportunity enablement, job size).
- Highlight cross-team dependencies and risks that may affect the PI roadmap.

---

## 2. Product Owner (PO)

### Identity

| Attribute | Detail |
|---|---|
| **SAFe Accountability** | Team-level content authority |
| **Reports to** | Product Manager (functionally for backlog alignment) |
| **Key Partners** | Business Analyst, Agile Team, Scrum Master / Team Coach, Product Manager |

### Core Responsibilities

- **Owns the team backlog.** Has content authority over stories and team-level enablers; prioritizes all work the team pulls into iterations.
- **Contributes to the vision and roadmap.** Develops the team-level understanding of the vision informed by customer feedback; assists with ART backlog prioritization and educates the ART during PI Planning.
- **Manages and prioritizes the team backlog.** Guides story creation, prioritizes backlog items, accepts stories, and supports architectural runway at the team level.
- **Connects with the customer.** Knows the customer and stakeholders, represents the end user, identifies problems, and develops whole-product thinking at the team level.
- **Supports the team in delivering value.** Balances stakeholder perspectives, elaborates stories, fosters built-in quality, and participates in all team and ART events.
- **Gets and applies feedback.** Tests benefit hypotheses at the story level, obtains feedback from customers and stakeholders, shares feedback with the team, and evolves solution design.

### Iteration-Level Role

- **Iteration Planning:** Leads the event; communicates story details and priorities; ensures capacity allocation balances user stories, enablers, maintenance, and refactors; establishes iteration goals with the team.
- **Team Sync:** Attends as a team member; clarifies story intent and acceptance criteria; resolves impediments in the meet-after; watches for opportunities to release value early.
- **Backlog Refinement:** Prepares and refines upcoming stories with the team; adds acceptance criteria; identifies dependencies; maintains a "ready" backlog two iterations deep.
- **Iteration Review:** Accepts completed stories against acceptance criteria and definition of done; decides whether incomplete stories are split, continued, delayed, or abandoned.
- **Iteration Retrospective:** Participates as a team member; supports relentless improvement.

### PI Planning Role

- Leads team breakouts, decomposing features into stories.
- Drafts and finalizes team PI objectives (committed and uncommitted).
- Identifies and manages team-level dependencies and risks.
- Participates in the draft plan review and final plan review.
- Participates in the confidence vote.

### What the PO Cares About (When Interacting)

- Story quality — are stories well-written (INVEST), properly sized, and testable?
- Acceptance criteria — are they specific, unambiguous, and in Given-When-Then format?
- Team capacity and velocity — can the team realistically deliver what's planned?
- Dependencies — what is blocking or could block the team?
- Iteration goals — does the planned work align with PI objectives?
- Definition of done — is every story meeting the agreed standard?

### How to Tailor Responses for the PO

- Frame work in terms of **stories, acceptance criteria, and iteration goals**.
- Use personas and user-voice format ("As a [role], I want … so that …").
- Provide ready-to-refine story details: acceptance criteria in Given-When-Then, story maps, and split patterns.
- Flag dependencies, risks, and capacity concerns early.
- Be specific about what "done" looks like for each piece of work.

---

## 3. Business Analyst (BA)

### Identity

| Attribute | Detail |
|---|---|
| **SAFe Accountability** | Cross-functional team member; requirements and analysis specialist |
| **Reports to** | Product Manager (directly) |
| **Key Partners** | Product Owner, Product Manager, Software Engineers, System Architect, UX |

### Core Responsibilities

> *Note: The BA role is not explicitly defined as a standalone SAFe role. In this project, the BA fills the functional, business, and — where needed — technical gaps between the PM, PO, and the rest of the Agile Team. The responsibilities below are derived from SAFe practices and mapped to BA activities.*

- **Bridges the PM ↔ PO gap.** Translates the PM's feature-level vision into PO-ready story-level detail. Decomposes features into stories, story maps, and acceptance criteria so the PO can prioritize and the team can build.
- **Bridges the PO ↔ Engineering gap.** Elaborates requirements to a level that removes ambiguity for engineers without over-constraining design. Ensures stories are "just right" — not insufficient, not overly prescriptive.
- **Supports Continuous Exploration.** Conducts customer research, develops personas, creates journey maps, and performs competitive / market analysis to feed the PM's vision and roadmap.
- **Drives requirements elicitation and analysis.** Facilitates workshops, empathy interviews, and design thinking activities. Produces artifacts such as personas, journey maps, story maps, process flows, and data models.
- **Supports backlog management at both levels.**
  - *ART backlog:* Helps the PM write feature descriptions, benefit hypotheses, and acceptance criteria; supports WSJF estimation by providing analysis on value and risk.
  - *Team backlog:* Helps the PO write and refine stories, craft Given-When-Then acceptance criteria, and maintain a healthy "ready" backlog.
- **Manages dependencies and risks.** Identifies cross-team and cross-system dependencies early; surfaces risks during PI Planning breakouts and ongoing refinement.
- **Validates and verifies.** Participates in Iteration Reviews; helps evaluate whether acceptance criteria are met; supports benefit hypothesis testing post-release.
- **Facilitates communication.** Acts as the connective tissue between business stakeholders, the PO, and the engineering team — ensuring shared understanding across all parties.

### PI Planning Role

- Supports the PM in preparing the vision, top 10 features, and PI roadmap materials.
- Participates in team breakouts alongside the PO; helps decompose features into stories in real time.
- Documents dependencies and risks identified during planning.
- Helps draft team PI objectives that are SMART and jargon-free.

### Iteration-Level Role

- Actively participates in Backlog Refinement — often the primary person preparing story details for the PO to review and prioritize.
- Attends Iteration Planning to clarify requirements and answer team questions.
- Joins Team Sync when story-level clarification is needed.
- Participates in Iteration Review to validate delivered functionality against acceptance criteria.
- Contributes to Iteration Retrospective for process improvement.

### What the BA Cares About (When Interacting)

- Clarity and completeness of requirements — no gaps between intent and implementation.
- Traceability — can every story trace back to a feature, and every feature to the vision?
- Stakeholder alignment — are the PM, PO, and team on the same page?
- Feasibility — has the team confirmed they understand and can build what's described?
- Quality of acceptance criteria — are they testable, specific, and unambiguous?

### How to Tailor Responses for the BA

- Provide **detailed, structured analysis** — the BA thrives on specificity.
- Include artifacts: story maps, process flows, Given-When-Then criteria, data models, decision matrices.
- Connect the dots between strategy (PM-level) and execution (PO/team-level).
- Highlight gaps, assumptions, and open questions that need resolution.
- Offer multiple decomposition options when breaking down features into stories.

---

## 4. Software Engineer (Dev)

### Identity

| Attribute | Detail |
|---|---|
| **SAFe Accountability** | Agile Team member; defines, builds, tests, and delivers solution functionality |
| **Reports to** | Scrum Master / Team Coach (for process); Product Owner (for priorities) |
| **Key Partners** | Product Owner, Business Analyst, other Agile Team members, System Architect |

### Core Responsibilities

- **Implements the vision and roadmap.** Works from the team backlog to build, test, and deliver increments of value every iteration.
- **Participates in planning.** Collaborates with the PO and BA during PI Planning breakouts and Iteration Planning to understand stories, estimate work (story points / estimating poker), and commit to iteration goals.
- **Applies built-in quality practices.** Follows engineering standards including unit testing, test automation, continuous integration, code reviews, and adherence to NFRs.
- **Supports the Continuous Delivery Pipeline.** Contributes to Continuous Integration (develop, test, build, stage), Continuous Deployment (deploy, verify, monitor, respond), and Release on Demand readiness.
- **Refines and estimates.** Participates in Backlog Refinement to ask clarifying questions, identify technical risks, split stories, and estimate using relative story points.
- **Identifies and raises risks and impediments.** Flags technical debt, dependency issues, and feasibility concerns during Team Sync, breakouts, and retrospectives.
- **Demonstrates working software.** Demos completed stories, spikes, refactors, and NFR work during Iteration Review and contributes to the System Demo.
- **Improves relentlessly.** Actively participates in Iteration Retrospectives; drives improvement in technical practices, tooling, and team processes.

### PI Planning Role

- Participates in team breakouts; decomposes features into stories alongside the PO and BA.
- Estimates story size and overall capacity.
- Identifies technical dependencies, enabler needs, and architectural runway gaps.
- Helps draft team PI objectives from a technical feasibility perspective.
- Participates in the confidence vote.

### Iteration-Level Role

- **Iteration Planning:** Estimates stories, identifies tasks, commits to iteration goals.
- **Team Sync:** Reports progress, raises impediments, coordinates with teammates.
- **Backlog Refinement:** Asks technical questions, flags complexity, suggests story splits.
- **Iteration Review:** Demos working functionality; receives feedback.
- **Iteration Retrospective:** Suggests and commits to process and technical improvements.

### What the Engineer Cares About (When Interacting)

- Technical clarity — what exactly should the system do, and what are the edge cases?
- Acceptance criteria — how will we know it's done and correct?
- Dependencies — what other teams, systems, or services does this touch?
- NFRs — what are the performance, security, scalability, and reliability expectations?
- Architectural runway — is the infrastructure in place, or do we need enablers first?
- Estimation accuracy — is the story small and well-understood enough to estimate confidently?

### How to Tailor Responses for the Engineer

- Be **concrete and technical** — provide specific acceptance criteria, data formats, API contracts, system behaviors, and edge cases.
- Use Given-When-Then for testable scenarios.
- Call out NFRs explicitly (performance targets, security requirements, platform constraints).
- Identify enabler stories or spikes needed before implementation can begin.
- Keep scope focused — if a story is too large (> 8 points), suggest split patterns.
- Respect the team's autonomy on *how* to build; focus on *what* and *why*.

---

## Role Interaction Map

```
┌─────────────────────────────────────────────────────────┐
│                    PORTFOLIO / BUSINESS OWNERS           │
│                  (Strategic Themes, Lean Budgets)        │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │    PRODUCT MANAGER     │
              │  (Vision, Roadmap,     │
              │   ART Backlog,         │
              │   Feature Priorities)  │
              └─────┬──────────┬───────┘
                    │          │
          ┌─────────▼──┐  ┌───▼──────────┐
          │  BUSINESS   │  │   PRODUCT    │
          │  ANALYST    │◄►│   OWNER      │
          │ (Analysis,  │  │ (Team        │
          │  Bridging,  │  │  Backlog,    │
          │  Artifacts) │  │  Stories,    │
          └─────┬───────┘  │  Priorities) │
                │          └───┬──────────┘
                │              │
                └──────┬───────┘
                       │
                       ▼
              ┌────────────────────────┐
              │   SOFTWARE ENGINEER    │
              │  (Build, Test,         │
              │   Deliver, Demo)       │
              └────────────────────────┘
```

### Key Interaction Patterns

| From → To | Nature of Interaction |
|---|---|
| **PM → PO** | Vision, top features, ART backlog priorities, scope negotiation |
| **PM → BA** | Feature definitions, benefit hypotheses, market research needs, roadmap context |
| **PO → BA** | Story refinement requests, acceptance criteria reviews, dependency analysis |
| **PO → Engineer** | Story priorities, acceptance criteria, iteration goals, story acceptance |
| **BA → PO** | Refined stories, story maps, Given-When-Then criteria, gap analysis |
| **BA → Engineer** | Requirements elaboration, process flows, data models, clarification |
| **BA → PM** | Analysis artifacts, research findings, feasibility feedback, dependency risks |
| **Engineer → PO** | Estimates, technical risks, demo of working software, impediments |
| **Engineer → BA** | Technical questions, edge cases, feasibility concerns, spike findings |

---

## Quick Reference: Content Authority

| Backlog Level | Content Authority | Key Collaborators |
|---|---|---|
| **Portfolio Backlog** | Epic Owners / LPM | PM, Business Owners |
| **ART Backlog** (Features & Enablers) | **Product Manager** | PO, System Architect, BA |
| **Team Backlog** (Stories & Enablers) | **Product Owner** | BA, Engineers, SM/TC |

---

*This document is a living reference. Update it as the EAM project evolves and roles are further refined.*
