# ServiceNow SPM — Common Issues, Pitfalls & Troubleshooting

Issues encountered during SPM implementation and ongoing support, based on ServiceNow community posts, partner experience, and implementation patterns.

---

## Issue 1: Planning Attributes Not Configured Before Resource Management

**Category:** Configuration / Implementation  
**Severity:** Critical — breaks resource management entirely

**The Problem:**
Teams start building resource plans before defining Planning Attributes (employee type, expense type, skill, role). When attributes are later changed or added, existing resource data becomes inconsistent or needs full rebuild.

**Why It Happens:**
Project managers want to start entering projects immediately. The foundational data setup step feels "IT configuration" rather than "business setup" — it gets skipped.

**Root Cause:**
Planning Attributes are the foundational currency for all resource allocation in SPM. Adding new attribute types after resource plans exist is extremely disruptive — existing records don't have the new attribute populated, breaking capacity calculations.

**Fix / Prevention:**
- Define ALL Planning Attributes before any resource planning begins
- Run a workshop: "What dimensions do you need to slice resource data by?" (by skill, by department, by CapEx/OpEx?)
- Document the attribute schema and get sign-off before configuring

**Community Reference:** ServiceNow Community — "Resource Plan attributes after go-live is painful" (multiple threads)

---

## Issue 2: Over-Customization Breaking Upgrades

**Category:** Implementation Best Practice  
**Severity:** High — causes significant upgrade failure rates

**The Problem:**
Customers modify base SPM tables, base UI pages, and base workflows directly instead of using scoped customization. On the next ServiceNow version upgrade, customizations conflict with out-of-box changes and break.

**Why It Happens:**
"We need it to work exactly like our old system" — clients customize SPM to mirror legacy tools instead of adopting the out-of-box process.

**Root Cause:**
Direct modifications to base SPM tables (`dmn_demand`, `pm_project`) prevent clean upgrades. ServiceNow ships new functionality in those base tables — customizations collide.

**Fix / Prevention:**
- Use scoped applications for all customizations
- Extend base tables instead of modifying them (create child tables)
- Store business logic in Business Rules / Script Includes within scoped apps
- Before ANY customization: ask "Can I achieve this with configuration (field visibility, choice lists, UI policies) instead of code?"

---

## Issue 3: Demand Board Process Not Defined — Demands Go Nowhere

**Category:** Governance / Process  
**Severity:** High — undermines the entire demand management investment

**The Problem:**
SPM is configured, Demand Management is active, demands are being created — but the Demand Board never meets, demands accumulate without decisions, and users stop submitting because "nothing ever happens."

**Why It Happens:**
The technology was implemented but the governance process wasn't. No one defined: Who sits on the Demand Board? How often do they meet? What are the scoring criteria? Who has final say?

**Root Cause:**
SPM is a process tool — the process must be designed by humans, not the tool. Technology without governance fails.

**Fix / Prevention:**
- Before go-live: define Demand Board membership, meeting cadence, and decision criteria
- Configure scoring thresholds that auto-recommend "Approve" vs "Review" vs "Reject"
- Assign a Portfolio Manager role with ownership of the demand pipeline
- Set SLA: demands must be reviewed within 30 days of submission — escalation if overdue

---

## Issue 4: Resource Assignments Show 200%+ Utilization — Nobody Adjusts

**Category:** User Adoption  
**Severity:** Medium — defeats the purpose of capacity planning

**The Problem:**
SPM shows resources at 200-300% capacity. Instead of using this as a signal to descope projects, managers override it, ignore the warnings, and commit to everything anyway. The capacity data becomes background noise.

**Why It Happens:**
Without a mandate from leadership, project managers feel they can't push back on delivery commitments. The SPM data shows overcommitment but nobody has authority to act on it.

**Root Cause:**
Organizational culture of overcommitment isn't fixed by SPM alone. The tool surfaces the problem but leadership must act on it.

**Fix / Prevention:**
- Establish a policy: "No project can be approved if the lead resource is >100% committed"
- Require Scenario Planning to be shown to the Demand Board — include a "resource-constrained scenario"
- Make resource utilization a regular agenda item in portfolio reviews
- CISO/CIO must visibly enforce capacity limits — once they decline a project based on utilization, the behavior changes

---

## Issue 5: Time Cards Not Completed — Project Health Calculations Are Wrong

**Category:** User Adoption  
**Severity:** High — makes automated health metrics unreliable

**The Problem:**
SPM's automated project health calculation relies on actual hours worked (from time cards) vs. planned hours. If team members don't fill in time cards, the health calculation shows all projects as healthy even when they're behind.

**Why It Happens:**
Time card completion is universally resisted by technical staff. "I'm coding, not tracking hours." Managers don't enforce it because they also find it burdensome.

**Root Cause:**
Time card completion requires a culture change + management enforcement. Without both, the feature provides no value.

**Fix / Prevention:**
- Make time card completion a mandatory Friday task — block access to certain systems until weekly time card is approved (where technically feasible)
- Keep time cards simple: by project, not by task
- Show developers how their time data is used — "your hours affect whether your team gets headcount approved"
- Management must complete time cards too — lead by example

---

## Issue 6: SPM Financials Not Aligned to Actual Budget Process

**Category:** Configuration / Integration  
**Severity:** Medium — SPM financials live in a silo

**The Problem:**
Investment Objects are set up in SPM with Cost Plans and Benefit Plans, but actual spend comes from a financial system (SAP, Oracle Financials). The two systems don't sync — SPM shows planned spend, finance shows actual spend, and they never agree. Nobody trusts either number.

**Why It Happens:**
Financial integration is complex and often deprioritized to "Phase 2" which never arrives.

**Root Cause:**
SPM financial data is only as good as its integration with the system of record for financials. Without integration, it becomes a manual double-entry system.

**Fix / Prevention:**
- During project initiation: build a data flow between SPM and the financial system
- If full integration isn't feasible: define a monthly reconciliation process (not ideal but better than nothing)
- Align SPM cost plan periods to financial budget periods (fiscal year, quarters)
- Assign a Finance Analyst as part of the SPM team — financial discipline must be embedded

---

## Issue 7: Agile Stories Not Visible in Portfolio Hierarchy

**Category:** Configuration  
**Severity:** Medium — hybrid portfolio visibility fails

**The Problem:**
Agile teams work in the Agile Development module (epics, stories, sprints) but their work doesn't roll up to the Portfolio view. Portfolio managers can see waterfall projects but not what the agile teams are delivering. Leadership asks: "What did the mobile app team ship this quarter?" — nobody knows in SPM.

**Why It Happens:**
Agile Development module stories must be explicitly linked to Plan Items (Programs or Projects) for rollup to work. Without this linking, they exist in a separate silo.

**Root Cause:**
Agile stories are not automatically in the portfolio hierarchy — the link must be configured and maintained.

**Fix / Prevention:**
- Configure Story → Epic → Program → Portfolio linking when setting up Agile Development
- Train agile teams: "Every epic must be linked to a Program in SPM"
- Use Hybrid Roadmap view — it only works when both waterfall and agile items are in the hierarchy
- Product owners are responsible for maintaining the linkage — make it part of sprint ceremonies

---

## Issue 8: SPM Data Migration Brings in Dirty Data from Legacy Tools

**Category:** Implementation  
**Severity:** High — destroys user trust immediately

**The Problem:**
When migrating from legacy PPM tools (Planview, Clarity, MS Project), organizations migrate all historical projects — including cancelled, abandoned, duplicate, and poorly structured projects. SPM is immediately cluttered with unusable data. Users say "SPM is worse than what we had."

**Why It Happens:**
"We don't want to lose our history" — stakeholders insist on migrating everything, including garbage.

**Root Cause:**
Data quality issues in legacy systems compound in the new system. Migrating bad data is worse than migrating no data.

**Fix / Prevention:**
- Perform data cleansing BEFORE migration, not after
- Define migration scope: "We will only migrate active projects and projects closed in the last 2 years"
- Migrate a sample first (50 projects) — validate data quality before full migration
- Accept that some historical data will be lost — this is a feature, not a bug

---

## Issue 9: No Executive Sponsorship — Dashboard Data Goes Unused

**Category:** Adoption / Organizational  
**Severity:** High — undermines entire ROI of SPM

**The Problem:**
SPM is implemented with excellent dashboards showing portfolio health, resource utilization, financial status. Executives continue to request manual PowerPoint status reports from PMO staff. The SPM dashboards exist but no one with decision-making authority uses them.

**Why It Happens:**
Executives are comfortable with existing reporting habits. SPM adoption requires behavior change at the top — the hardest change to drive.

**Root Cause:**
Without executive mandate to use SPM dashboards, the PMO office must maintain both the dashboard AND the PowerPoint, doubling their work.

**Fix / Prevention:**
- CIO/CISO must visibly use SPM dashboards in portfolio review meetings — from day one
- Eliminate the competing PowerPoint — make SPM the ONLY source of portfolio status
- Train executive sponsors on SPM navigation — make them users, not just sponsors
- Celebrate the first board meeting where SPM data was used — make it visible

---

## Issue 10: Scenario Planning Not Used — Big Decisions Still Made by Gut Feel

**Category:** Adoption  
**Severity:** Medium — key SPM feature underutilized

**The Problem:**
Scenario Planning is one of SPM's most powerful features but is rarely used. Budget decisions are still made by HiPPO (Highest Paid Person's Opinion) rather than modeled scenarios.

**Why It Happens:**
Scenario Planning requires input data (resource availability, project costs, benefit projections) to be accurate. When underlying data is poor, scenarios aren't credible.

**Root Cause:**
Scenario Planning is only as good as the data going in. If cost plans and resource data are rough estimates, scenarios become informed guesses rather than analytical recommendations.

**Fix / Prevention:**
- Improve data quality before promoting Scenario Planning as a capability
- Start with simple scenarios: "Fund top 10 by WSJF vs. top 10 by strategic alignment"
- Present scenarios to the Demand Board with explicit assumptions — "these are the inputs, here are the trade-offs"
- Once Scenario Planning influences one big budget decision, adoption follows

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Resources showing 200%+ utilization | Overcommitment is real — not a bug | Review and descope projects; engage leadership |
| Demands never get approved | No Demand Board governance | Define board, cadence, scoring criteria |
| Project health always shows green | Time cards not completed; manual status override | Enforce time card completion; disable manual override |
| SPM financials don't match actuals | No financial system integration | Build integration or define manual reconciliation |
| Agile work not in portfolio view | Stories not linked to Plan Items | Link epics to Programs in portfolio hierarchy |
| Upgrades break custom workflows | Base table customization | Move to scoped app; extend base tables |
| Nobody uses the dashboards | No executive sponsorship | CIO must mandate SPM as single source of truth |

---

## Sources and References

- ServiceNow Community: community.servicenow.com — search "SPM implementation issues"
- ServiceNow Docs: SPM Implementation Guide
- ServiceNow Now Create methodology for SPM
- LinkedIn: "ServiceNow SPM lessons learned" posts from certified practitioners
- ServiceNow Knowledge Conference (Knowledge23/24) SPM track sessions
