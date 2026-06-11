# ServiceNow SPM — Interview Notes
## "What did you do in SPM?" + Challenges Faced

---

## 1. OVERVIEW — What is SPM?

SPM (Strategic Portfolio Management), previously known as ITBM/PPM, is ServiceNow's module for managing the demand for new initiatives, tracking them through approval and funding, executing them as projects, and reporting on portfolio health to leadership. It answers the question: "What are we working on, why are we doing it, and do we have the capacity to do it all?"

SPM connects business demand with IT delivery. A business stakeholder submits an idea or demand through the portal; it gets scored and prioritized; funded demands become projects; projects consume resource capacity; and leadership gets a real-time portfolio view without waiting for weekly status emails.

Key components worked on:
- Demand Management and Idea Portal
- Project Request and Approval workflows
- Portfolio and Program configuration
- Resource Management and Capacity Planning
- Timesheets and Actual vs. Planned tracking
- Executive dashboards and PA reporting

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Demand Management and Idea Portal

- Configured the **Idea Portal** as the entry point for new initiative requests — business stakeholders could submit ideas through the Employee Center with a structured intake form covering business justification, estimated benefit, rough cost, and strategic alignment
- Set up the **demand workflow**: submitted ideas went through a preliminary scoring phase (automated scoring based on strategic alignment criteria) before being reviewed by the portfolio team for formal demand creation
- Configured **Demand Templates** for different types of requests: new system implementations, infrastructure upgrades, compliance-driven projects, and innovation investments — each template captured different fields relevant to its category
- Built **prioritization scoring models** with weighted criteria: strategic alignment (40%), ROI estimate (30%), compliance obligation (20%), risk reduction (10%) — each field had a defined scoring guide so stakeholders rated consistently rather than subjectively

### 2.2 Project Request and Approval Workflows

- Configured **multi-stage approval workflows** for demand funding: demands above a threshold went to the Business Sponsor → IT Director → Portfolio Committee → Finance for budget approval, with each stage having a defined SLA and escalation if not actioned
- Set up **gate reviews** at key milestones (Concept, Planning, Execution, Close) — each gate required documented deliverables before the project could progress, enforced by a Flow Designer flow that prevented state transition without all gate criteria met
- Configured **project status reporting**: project managers updated a weekly status report within the project record and the system automatically generated a RAG (Red/Amber/Green) status based on schedule variance, budget variance, and risk count
- Built an **automated project closure process** — when a project reached the Close gate, the system automatically created tasks for: lessons learned documentation, resource release confirmation, budget final reconciliation, and benefit realization planning

### 2.3 Resource Management and Capacity Planning

- Built out **Resource Plans** for ongoing and planned projects — project managers defined resource needs by role and time period; resource managers allocated named individuals or groups against those requests
- Configured **Capacity Planning views** showing available vs. committed capacity by team and time period — this gave resource managers visibility into over-allocation before it caused delivery problems rather than after
- Tied project demands back to actual resource availability: when a new demand was added to the portfolio, the resource impact was immediately visible in the capacity view so leadership could make informed prioritization decisions rather than committing to everything and under-delivering
- Set up **timesheet integration** for project teams — actual hours logged flowed into project cost tracking, enabling comparison of planned vs. actual effort at both the task and project level

### 2.4 Portfolio and Executive Reporting

- Configured **Portfolio views** showing all active and planned projects with RAG status, budget, schedule, and strategic theme alignment in a single dashboard
- Built **Performance Analytics indicators** for the portfolio: on-time delivery rate, budget variance by project, demand-to-delivery cycle time (from idea submission to project kickoff), and resource utilization by team
- Set up **automated executive reports** delivered weekly to portfolio stakeholders — the report pulled live SPM data and showed: what is in flight, what is at risk, what is in the pipeline, and what decisions are needed this week
- Configured **benefit realization tracking** for completed projects — post-go-live, project owners were prompted to record actual benefits achieved vs. the projected ROI that justified the investment

---

## 3. CHALLENGES FACED

### Challenge 1: Demand Scoring Model Not Trusted by Stakeholders

**Problem:** After deploying the automated demand scoring model, business stakeholders immediately questioned the rankings. Projects they believed were high priority were scored lower than they expected, and they assumed the model was wrong rather than questioning their own assumptions. The portfolio committee was overriding model scores manually in almost every review meeting.

**Solution:** Ran a retrospective with the portfolio committee using 12 months of completed projects as test data: applied the scoring model to projects that were already completed and compared the model's ranking to the actual business value delivered. The model's rankings correlated well with actual outcomes. Used this data to walk the committee through specific examples — "this project that scored 72 delivered 3x its projected benefit; this one that scored 45 was cancelled after 6 months." After seeing the model's track record, manual overrides dropped by 70%.

---

### Challenge 2: Resource Allocation Conflicts Across Projects

**Problem:** Three high-priority projects were all requesting the same senior architect for the same 6-week period. All three project managers had confirmed the architect's availability without checking the capacity view — each had received verbal confirmation but none had formal resource allocations in SPM.

**Solution:** Made formal resource allocation in SPM a mandatory step before a project could move from Planning to Execution gate. The system blocked gate progression if required roles had no allocation record. Added a conflict resolution workflow: when a resource allocation conflict was detected, the system notified the resource manager and the affected project managers simultaneously and created a resolution task tracked in SPM.

---

### Challenge 3: Demand Pipeline Not Reflecting Reality — Shadow Portfolios

**Problem:** Despite having the SPM demand portal, several business units continued managing their project pipeline in Excel spreadsheets and SharePoint lists. The SPM pipeline only captured about 60% of actual demand — leadership was making portfolio prioritization decisions with incomplete information.

**Solution:** Two-part approach: first, made SPM the required channel for any IT resource request above a certain size (>200 hours), enforced through the budget approval process — Finance would not approve project budgets without a demand record in SPM. Second, worked with business unit leaders to run an SPM intake workshop and migrate their existing Excel pipelines into SPM demands. Within two quarters, pipeline coverage increased to over 90%.

---

### Challenge 4: Capacity Planning Data Not Accurate Enough to Trust

**Problem:** The resource capacity view showed most teams at 80-90% utilization, which seemed high. When we dug in, we found that project managers were allocating resources at 100% for the full duration of a project even when the resource only needed 20% of their time for planning phases. The capacity view was showing everyone as over-allocated before actual delivery work even started.

**Solution:** Configured **phase-based resource allocation**: resource allocations were broken into phases aligned with the project schedule, with realistic effort percentages per phase. Also added a baseline allocation cap guidance: project managers were coached that resources should not be allocated above 75% in the SPM plan (leaving headroom for unplanned work). Ran a calibration exercise with the top 10 project managers to normalize their allocation approach.

---

### Challenge 5: Executive Dashboard Showing Different Numbers Than Status Emails

**Problem:** Portfolio leadership started receiving different project status information from the SPM dashboard vs. the manual status emails project managers were still sending. Some PMs had not updated their SPM records weekly and leadership lost trust in the SPM data.

**Solution:** Set up an automated reminder system: project managers received a reminder task every Friday to update their project status record before close of business. If the status record was not updated by Monday morning, the project automatically turned Red in the dashboard with a note "Status not updated" — which was visible to the project sponsor. The public visibility of missing updates incentivized PMs to update promptly. After one cycle, update compliance went from 55% to 91%.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **Demand** | A request for IT resources or investment — the starting point of the SPM pipeline |
| **Idea Portal** | The self-service entry point where business stakeholders submit initiative ideas |
| **Project** | A formal effort to deliver a defined outcome — created from an approved and funded Demand |
| **Portfolio** | A collection of programs and projects aligned to a strategic theme or business unit |
| **Resource Plan** | Defines what roles/skills are needed for a project, in what quantity, and over what time period |
| **Capacity View** | Shows available vs. committed capacity for resources by team and time period |
| **Gate Review** | A milestone checkpoint where a project must meet defined criteria before proceeding |
| **RAG Status** | Red/Amber/Green — quick visual indicator of project health (schedule, budget, risk) |
| **Benefit Realization** | Post-project tracking of actual business value delivered vs. projected ROI |
| **Timesheet** | Records actual time spent on project tasks — feeds into planned vs. actual cost tracking |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Demand pipeline coverage increased from **60% to 90%+** after linking SPM to budget approval process
- Portfolio status update compliance improved from **55% to 91%** after automated reminder and Red-status consequence
- Resource allocation conflicts reduced by **~65%** after mandatory formal allocation before gate progression
- Executive reporting eliminated manual assembly — portfolio status report generated in **minutes vs. 3 hours** of manual compilation
- Scoring model override rate dropped by **70%** after demonstrating historical correlation with actual project outcomes
- Capacity planning accuracy improved: over-allocation incidents discovered after the fact reduced by **~50%** in the following quarter
