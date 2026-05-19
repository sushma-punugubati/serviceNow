# ServiceNow SPM — Fundamentals & Study Guide

> Sources: [JitendraZaa SPM Guide](https://www.jitendrazaa.com/blog/servicenow/servicenow-spm-complete-guide/) | [ServiceNow SPM Community](https://www.servicenow.com/community/spm-articles/strategic-portfolio-management-spm-welcome-guide/ta-p/2368193)

---

## 1. What is SPM?

**Strategic Portfolio Management (SPM)** is ServiceNow's AI-driven platform for connecting business strategy to execution. It integrates demand intake, resource allocation, financial planning, project delivery, and performance analytics into one place.

> **Think of it as:** The "command center" where business leaders and IT teams decide which projects get funded, who works on what, and how to measure success.

**Previous name:** IT Business Management (ITBM) — renamed to SPM to reflect broader enterprise applicability beyond just IT.

**Core value:** Ensure every project, program, and initiative is tied to a strategic goal. No more "rogue" projects that consume resources without delivering business value.

---

## 2. Architecture Layers

| Layer | What It Does | Examples |
|-------|-------------|---------|
| **Strategic** | Set goals and align everything to them | OKRs, Goal Frameworks, Scenario Planning |
| **Intake** | Capture ideas and requests | Innovation Hub, Demand Management, Ideas Portal |
| **Execution** | Deliver the work | Project Management, Agile, Resource Management |
| **Governance** | Track money, time, and risk | Portfolio Financials, Time Cards, Reporting |

---

## 3. Core Modules Explained Simply

### 3.1 Demand Management
Centralizes all incoming requests for new work — from business ideas to IT project requests.

- **Who uses it:** Portfolio managers, business stakeholders
- **What it does:** Scores, prioritizes, and routes demands for approval
- **Output:** Approved demands convert into projects/programs

**Scoring frameworks used:**
| Framework | Meaning |
|-----------|---------|
| **RICE** | Reach × Impact × Confidence ÷ Effort |
| **WSJF** | Weighted Shortest Job First — favors high-value, short efforts |
| **MoSCoW** | Must Have, Should Have, Could Have, Won't Have |
| **Fibonacci** | Story point estimation for agile work |
| **Value vs Effort** | Simple matrix — high value + low effort = top priority |

### 3.2 Project Portfolio Management (PPM)
Organizes projects into programs and portfolios with real-time health tracking.

- **Portfolio:** A collection of programs/projects aligned to a strategic goal
- **Program:** A group of related projects sharing resources and outcomes
- **Project:** Individual initiative with tasks, milestones, resources, and budget

### 3.3 Resource Management
Tracks who is working on what, how much capacity is available, and where conflicts exist.

- **Planning Attributes:** Employee type, skill, role, expense type (CapEx/OpEx), group
- **Capacity heatmaps:** Show who is over/under-allocated across the portfolio
- **Resource Assignments:** Modern approach replacing older "Resource Plans"

> **Key implementation fact:** Define planning attributes (skill, role, expense type) BEFORE configuring resource management. Adding them later is very difficult.

### 3.4 Financial Planning (Portfolio Financials)
Tracks budget vs. actuals for projects and portfolios.

- **Cost Plan:** Planned costs by time period
- **Benefit Plan:** Expected business value delivered
- **Funding/Budget Breakdown:** How money is allocated
- **CapEx vs OpEx:** Capital vs. operating expense tracking

### 3.5 Innovation Management
Captures raw ideas before they become formal demands.

- An "Ideas Portal" where anyone in the org can submit ideas
- Ideas are evaluated, refined, and promoted to Demands if valuable
- Prevents ideas from being lost in email chains

### 3.6 Agile Development
Manages scrum/agile work directly in SPM.

- Backlog management, sprint planning, story tracking
- Links agile work to parent initiatives for strategic visibility
- Supports hybrid teams (some waterfall, some agile)

---

## 4. Key Tables

| Table Name | Label | Purpose |
|-----------|-------|---------|
| `sn_align_core_plan_item` | Plan Item | Core SPM entity — base for all strategic/work items |
| `dmn_demand` | Demand | Core demand records |
| `sn_align_core_demand` | Demand (Roadmapping) | Demand linked to roadmap/alignment views |
| `pm_project` | Project | Project records |
| `pm_program` | Program | Program records |
| `sn_gf_goal` | Goal | Strategic goals/OKRs |
| `sn_align_core_lens` | Alignment Lens | Creates hierarchy views among plan items |
| `time_card` | Time Card | Time tracking by resource |
| `sn_align_core_investment` | Investment Object | Financial core — cost/benefit/funding data |
| `dmn_demand_assessment` | Demand Assessment | Scoring records for demands |

---

## 5. SPM vs Traditional PM Tools

| Aspect | Traditional PM Tools | ServiceNow SPM |
|--------|---------------------|----------------|
| Scope | Individual projects | Enterprise-wide portfolio |
| Strategy alignment | Manual, spreadsheet-based | Built-in Goal Frameworks |
| Resource view | Project-by-project | Cross-portfolio capacity planning |
| Financial tracking | Separate systems | Integrated with project records |
| Methodologies | Usually one (Waterfall or Agile) | Waterfall + Agile + Hybrid unified |
| Reporting | Manual compilation | Real-time dashboards |

---

## 6. Five-Stage Maturity Model

Organizations implement SPM progressively:

| Stage | Name | What Happens |
|-------|------|-------------|
| 1 | **Foundation** | Clean CMDB, user records, org structure established |
| 2 | **Crawl** | Single source of truth using out-of-box processes |
| 3 | **Walk** | Active executive governance; portfolio review cadence |
| 4 | **Run** | ROI measurement; Agile integrated with governance |
| 5 | **Fly** | Predictive analytics and AI-driven portfolio optimization |

---

## 7. Phased Implementation Sequence

| Phase | Modules Deployed | Focus |
|-------|-----------------|-------|
| 1 | Project Management, Time Cards | Basic tracking and time capture |
| 2 | Demand Management, Resource Plans | Standardized intake and prioritization |
| 3 | Portfolio Financials, Reporting | Budget management and dashboards |
| 4 | Strategic Planning, Goal Frameworks | OKR alignment, strategy execution |
| 5 | Agile Development, Hybrid Roadmaps | Scrum integration, unified view |

---

## 8. Demand Management Workflow

```
Raw Idea / Business Request
  ↓
Idea Capture (Ideas Portal or Service Catalog)
  ↓
Demand Creation (business case added)
  ↓
Assessment & Scoring (RICE / WSJF / MoSCoW)
  ↓
Demand Board Review (governance approval)
  ↓
Approved → Project Created (with audit trail)
Rejected → Documented with reason
```

---

## 9. Roles & Governance

| Role | Responsibilities |
|------|----------------|
| **Portfolio Owner** | Owns a strategic portfolio; accountable for outcomes |
| **Demand Manager** | Manages intake, scoring, and routing of demands |
| **Resource Manager** | Oversees allocation, capacity, and conflict resolution |
| **Project Manager** | Manages individual projects within a portfolio |
| **Demand Board** | Governance body that reviews and approves/rejects demands |
| **Executive Leadership** | Reviews portfolio health and strategic alignment |

---

## 10. SPM Licensing Packages

| Package | Best For |
|---------|---------|
| **SPM Standard** | Organizations with traditional waterfall methods aligned to strategy |
| **SPM Professional** | Organizations managing end-to-end with Agile or hybrid methodologies |

---

## 11. Integration Points

- **CMDB:** Maps projects to infrastructure and services for impact analysis
- **ITSM:** Links demands to incidents and problems for data-driven prioritization
- **HR Service Delivery:** Resource management uses employee records
- **External tools:** APIs for Jira, Microsoft Project, Power BI
- **Financial systems:** ERP integration for actual cost sync

---

## 12. AI Capabilities (Now Assist for SPM)

| Capability | What It Does |
|-----------|-------------|
| Health Analysis | Identifies at-risk projects before derailment |
| Resource Monitoring | Surfaces allocation conflicts proactively |
| Prescriptive Recommendations | Suggests actions like reallocation |
| Natural Language Queries | "Show all at-risk Digital Transformation projects" |
| Status Summaries | Auto-generates project status reports |

---

## 13. Business Problems SPM Solves

| Problem | SPM Solution |
|---------|-------------|
| Budget opacity | Real-time cost tracking + ROI measurement |
| Prioritization chaos | Automated demand scoring with RICE/WSJF |
| Resource over-allocation | Cross-portfolio capacity heatmaps |
| Poor strategy alignment | Goal Frameworks link projects to objectives |
| Methodology fragmentation | Hybrid roadmaps unify waterfall + agile |
| Reactive reporting | Live dashboards for executives |

---

## 14. Glossary

| Term | Simple Explanation |
|------|-------------------|
| **Demand** | A formal request for new work or investment |
| **Portfolio** | Collection of programs/projects serving a strategic goal |
| **Program** | Group of related projects sharing resources |
| **Plan Item** | The fundamental SPM entity all work records extend from |
| **Investment Object** | Financial record for a plan item (cost + benefit + funding) |
| **Alignment Lens** | A view that shows hierarchy relationships among plan items |
| **CapEx** | Capital Expenditure — investment in long-term assets |
| **OpEx** | Operating Expenditure — ongoing operational costs |
| **RICE** | Reach-Impact-Confidence-Effort scoring framework |
| **WSJF** | Weighted Shortest Job First — Agile prioritization |
| **OKR** | Objectives and Key Results — strategy measurement framework |
| **Demand Board** | Governance committee that approves/rejects demands |
| **Resource Assignment** | Modern way to allocate team members to work items |
| **Time Card** | Record of hours worked by a resource on a project |
| **CSDM** | Common Service Data Model — foundational data layer |
