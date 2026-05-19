# ServiceNow SPM — Interview Questions & Answers

---

## PART 1: Fundamentals

**Q1: What is ServiceNow SPM and what was it previously called?**
**A:** SPM (Strategic Portfolio Management) is ServiceNow's platform for aligning IT investments with business strategy — managing demand, resources, projects, programs, and financials in one place. It was previously called **ITBM (IT Business Management)**. The rename reflects its expanded use beyond IT to enterprise-wide strategic planning.

---

**Q2: What are the four architecture layers of SPM?**
**A:**
1. **Strategic** — Goal frameworks, OKRs, scenario planning
2. **Intake** — Demand management, ideas portal, innovation capture
3. **Execution** — Project management, agile development, resource management
4. **Governance** — Portfolio financials, time cards, reporting and analytics

---

**Q3: What is the difference between a Portfolio, Program, and Project in SPM?**
**A:**
- **Portfolio:** A collection of programs and projects aligned to a strategic business goal. Owned by a Portfolio Owner.
- **Program:** A group of related projects that share resources and deliver a combined outcome larger than any individual project.
- **Project:** An individual initiative with defined scope, timeline, budget, tasks, and resources.

*Hierarchy: Portfolio → Programs → Projects → Tasks*

---

**Q4: What is the SPM maturity model?**
**A:** Five stages:
1. **Foundation** — Clean data, user records, org structure
2. **Crawl** — Single source of truth using out-of-box processes
3. **Walk** — Executive governance, portfolio review cadence
4. **Run** — ROI measurement, agile integrated with governance
5. **Fly** — AI-driven optimization and predictive analytics

---

**Q5: What is a Demand in SPM and what is its lifecycle?**
**A:** A Demand is a formal request for new work or investment. Lifecycle:
```
Idea → Demand Created (with business case) → Scored & Assessed → 
Demand Board Review → Approved → Project Created / Rejected
```

---

**Q6: What scoring frameworks are used in Demand Management?**
**A:**
- **RICE:** Reach × Impact × Confidence ÷ Effort
- **WSJF:** Weighted Shortest Job First — favors high-value short items
- **MoSCoW:** Must Have / Should Have / Could Have / Won't Have
- **Fibonacci:** Story point estimation for agile work
- **Value vs. Effort matrix:** Simple 2×2 prioritization

---

**Q7: What is the Plan Item table and why is it important?**
**A:** `sn_align_core_plan_item` is the foundational SPM entity. All strategic and work items (demands, projects, programs, goals) build on or relate to Plan Items. It provides a unified, flexible data structure for portfolio hierarchy and relationships.

---

**Q8: What is an Investment Object in SPM?**
**A:** The Investment Object is the financial core table for SPM. It contains three components:
- **Cost Plan & Breakdown:** Planned spending over time
- **Benefit Plan & Breakdown:** Expected business value delivered
- **Funding/Budget Breakdown:** How money is allocated

Every major SPM plan item has an associated Investment Object tracking its finances.

---

**Q9: What is the Alignment Lens in SPM?**
**A:** The Alignment Lens (`sn_align_core_lens`) allows organizations to create flexible, configurable hierarchy views among plan items. Different business stakeholders can view the same portfolio data through different hierarchical lenses — e.g., by capability, by business unit, or by strategic theme.

---

## PART 2: Configuration & Roles

**Q10: What are Planning Attributes in Resource Management and why must they be configured early?**
**A:** Planning Attributes are the common currency for resource allocation:
- Employee type, expense type (CapEx/OpEx), skill, role, group

They **must be defined before** configuring capacity planning because:
- Resource plans and assignments use these attributes
- Adding new planning attributes after the fact is extremely difficult and can break existing resource data
- It's the most common configuration mistake in SPM implementations

---

**Q11: What is the difference between Resource Plans and Resource Assignments?**
**A:**
- **Resource Plans (legacy):** Older approach to allocating resources — created separate plan records
- **Resource Assignments (modern):** Direct assignment of team members to work items — simpler, more visible, the preferred modern approach

New SPM instances have Resource Assignments enabled by default. Existing instances can enable via system property `sn_pw.enable_resource_planning = true`.

---

**Q12: What are the two SPM licensing packages?**
**A:**
- **SPM Standard:** For organizations with traditional waterfall methodology
- **SPM Professional:** For organizations using Agile or hybrid methodologies end-to-end

---

**Q13: What is the Demand Board?**
**A:** The Demand Board is the governance committee (executives, portfolio managers, business leaders) that reviews scored demands and decides which get approved for investment and which get rejected. It brings discipline to portfolio decisions.

---

**Q14: How does SPM integrate with ITSM and CMDB?**
**A:**
- **CMDB integration:** Maps projects and programs to specific infrastructure CIs and business services. Enables impact analysis — "if I stop Project X, which services are affected?"
- **ITSM integration:** Demand data can be enriched with incident and problem trends. High-incident areas surface as demands for investment.

---

## PART 3: Scenario-Based Questions

**Q15: A business unit submits 50 demands in Q1 but you only have budget for 10 projects. How would you use SPM to prioritize?**
**A:**
1. Configure scoring criteria in Demand Management (RICE or WSJF)
2. All 50 demands scored automatically on defined criteria
3. Use Scenario Planning to model different funding scenarios (what if we fund the top 10 by WSJF score vs. top 10 by strategic alignment?)
4. Present scenarios to the Demand Board with full financial projections
5. Board selects scenario; approved demands convert to projects with full audit trail
6. Rejected demands documented with reason — not lost

---

**Q16: A project manager says her project is "green" but it's actually behind schedule and over budget. How does SPM address this?**
**A:** This is a classic "watermelon project" (green outside, red inside). SPM addresses it through:
- **Automated health calculation** based on actual milestone completion, budget actuals, and resource utilization — not self-reported status
- **AI Now Assist** surfaces at-risk signals even when status is reported green
- **Executive dashboards** show objective metrics rather than subjective status reports
- **Time card integration** shows actual hours vs. planned — reveals true project health

---

**Q17: An organization wants to run both Agile (Scrum) and Waterfall projects in the same portfolio. Is this possible in SPM?**
**A:** Yes. SPM supports **hybrid portfolios**:
- Waterfall projects: managed via Project Workspace with milestones and Gantt
- Agile work: managed via Agile Development module (stories, sprints, backlogs)
- **Hybrid roadmaps:** Show both waterfall and agile work in a unified timeline view
- Resource management works across both — a resource can work on agile stories AND waterfall milestones

---

**Q18: What are the most common SPM implementation failures?**
**A:**
1. **Over-customization** — resisting out-of-box processes
2. **Big bang rollout** — trying to implement everything at once
3. **No planning attributes defined upfront** — resource management becomes broken
4. **Recreating Excel in SPM** — digitizing old broken processes instead of redesigning
5. **No executive sponsorship** — leadership doesn't actually use the dashboards
6. **Data migration without cleansing** — dirty data undermines trust
7. **No governance model** — unclear who approves demands and by what criteria

---

## PART 4: Financial & Strategy Questions

**Q19: What is the difference between CapEx and OpEx in SPM context?**
**A:**
- **CapEx (Capital Expenditure):** Investment in long-term assets — building new capabilities, developing software from scratch. Capitalized on the balance sheet; depreciated over time.
- **OpEx (Operating Expenditure):** Ongoing operational costs — maintenance, support, running existing systems. Expensed immediately.

SPM tracks both via Planning Attributes so finance teams can report correctly. Many organizations use SPM to help decide whether work should be capitalized or expensed.

---

**Q20: What is Scenario Planning in SPM?**
**A:** Scenario Planning lets portfolio managers model "what if" questions:
- "What if we reduce budget by 20%? Which projects would we cut, and what's the business impact?"
- "What if we add 5 more developers? Which additional demands can we deliver?"

Multiple scenarios can be compared side-by-side before committing to a plan.

---

## PART 5: Exam-Style Quick Facts

| Question | Answer |
|----------|--------|
| Previous name of SPM | ITBM (IT Business Management) |
| Core/base SPM table | `sn_align_core_plan_item` (Plan Item) |
| Demand table | `dmn_demand` |
| Financial table | Investment Object |
| SPM packages | Standard, Professional |
| Scoring framework that favors short high-value work | WSJF |
| Resource allocation attributes must be defined | Before capacity planning configuration |
| Maturity stages | Foundation, Crawl, Walk, Run, Fly |
| SPM governance body | Demand Board |
| AI feature in SPM | Now Assist for SPM |

---

## Study Checklist
- [ ] Know: SPM was formerly ITBM
- [ ] Know: Portfolio → Program → Project hierarchy
- [ ] Know: RICE, WSJF, MoSCoW scoring frameworks
- [ ] Know: Planning Attributes must be defined early
- [ ] Know: 5 maturity stages in order
- [ ] Know: Plan Item is the foundational table
- [ ] Know: Investment Object = Cost + Benefit + Funding
- [ ] Know: SPM Standard vs Professional licensing
- [ ] Know: Demand lifecycle end-to-end
- [ ] Know: Hybrid portfolios support both Agile and Waterfall
