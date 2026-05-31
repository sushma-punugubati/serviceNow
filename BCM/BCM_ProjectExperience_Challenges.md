# ServiceNow BCM — Interview Notes
## "What did you do in BCM?" + Challenges Faced

---

## 1. OVERVIEW — What is BCM?

BCM (Business Continuity Management) in ServiceNow is a module that helps organizations plan for, respond to, and recover from disruptive incidents. It provides a structured framework for defining Business Continuity Plans (BCPs), Recovery Plans, Crisis Management processes, and conducting readiness exercises — all within the ServiceNow platform.

BCM sits at the intersection of Risk Management and Operations. It captures what needs to be protected (critical business services and processes), how they can be recovered (recovery plans and tasks), and proves that the plans work (exercises and testing). Unlike a spreadsheet-based BCP program, ServiceNow BCM ties plans directly to CMDB CIs, ITSM incidents, and IRM risk records — giving a live, connected view of organizational resilience.

Key components worked on:
- Business Continuity Plans (BCPs) — structured plans per business unit or service
- Recovery Plans — step-by-step recovery procedures with task assignments
- Crisis Management — incident command structure and communication templates
- BCM Exercises — simulated tests to validate plan readiness
- Sites and Resources — alternate work locations, resource inventories
- Business Impact Analysis (BIA) — tied to IRM risk assessments

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Business Impact Analysis and Plan Configuration

- Worked alongside the GRC/IRM team to align BCM with the existing Risk Management framework — BCPs were linked to business services defined in the CMDB so we had a direct line from CI to business impact to recovery procedure
- Configured **Business Continuity Plans** for critical business units — each plan defined the Recovery Time Objective (RTO) and Recovery Point Objective (RPO) for the services it covered
- Set up **Business Process records** with business impact ratings and dependency mappings, linking to CMDB CIs so we could trace which infrastructure supported each critical process
- Built **Plan Templates** so each business unit could instantiate a new BCP from a consistent structure rather than starting from scratch — maintained consistency and reduced setup effort for new plans

### 2.2 Recovery Plan Configuration

- Configured **Recovery Plans** with step-by-step task sequences for IT systems recovery — each task had an owner, a time estimate, and dependency logic (similar to Lifecycle Event activity sequencing in HRSD)
- Set up automatic task generation when a Crisis Event was declared — the system created and assigned recovery tasks across IT, Facilities, and Business Operations without manual intervention
- Built notification templates for different crisis severity levels — P1 declarations triggered immediate executive notifications via email and SMS; P2/P3 used standard notification channels

### 2.3 BCM Exercises and Testing

- Configured and ran **BCM Exercises** to simulate crisis scenarios and validate recovery plan completeness — tracked exercise results against expected RTOs and logged gaps as follow-up tasks
- Used exercise results to drive plan updates: if a recovery task consistently took longer than estimated, updated the task duration and adjusted RTO calculations
- Built a **BCM dashboard** showing exercise completion rates, plan update frequency, and RTO compliance — used by the business continuity officer for quarterly board reporting

### 2.4 Integration with IRM and ITSM

- Integrated BCM with the **IRM module** — high-impact risks in IRM that crossed a defined threshold automatically created BCM planning tasks to ensure a response plan existed for each identified risk
- Linked BCM **Crisis Events** to **ITSM Major Incidents** — when a P1 incident was declared in ITSM, a corresponding Crisis Event was created in BCM to trigger the formal business continuity response alongside technical resolution
- Configured automated stand-down: when the ITSM P1 incident moved to Resolved, the BCM Crisis Event received a notification to begin the stand-down and lessons-learned phase

### 2.5 Access Control and Documentation

- Configured **role-based access** — plan details were visible to BCM administrators and plan owners, but sensitive recovery procedures (executive contact lists, alternate site locations) were restricted to crisis management roles only
- Maintained all BCM design documentation in Confluence: plan hierarchy, IRM/ITSM integration architecture, and post-exercise review notes

---

## 3. CHALLENGES FACED

### Challenge 1: BIA Data Quality — CMDB Gaps Broke Impact Calculations

**Problem:** The Business Impact Analysis relied on the CMDB to identify critical business processes and their supporting CIs. Our CMDB had significant data quality gaps — several critical application CIs were missing relationships to their business services, meaning BIA results were incomplete and RTOs were calculated without full dependency context.

**Solution:** Ran a targeted CMDB remediation before the BCM rollout: identified all business services in scope, traced CI dependencies using Service Mapping, and filled gaps manually where automation could not resolve them. Added a data quality gate: no BCP could be published until all its linked CIs had "Operational" status and at least one confirmed dependency relationship.

---

### Challenge 2: Recovery Task Ownership Gaps

**Problem:** Several recovery tasks had no assigned owner — plan authors had listed team names but not specific individuals. When we ran a dry-run exercise, 8 out of 42 tasks had no one available to action them.

**Solution:** Added a mandatory validation rule: tasks without a named individual (not just a group) could not move from Draft to Active status. Also configured automated reminders to task owners 48 hours before exercises to confirm availability and readiness.

---

### Challenge 3: BCM and ITSM Running in Parallel Without Visibility

**Problem:** During a real P1 incident, the ITSM team and BCM team were working simultaneously but not in sync — ITSM resolved the technical issue while BCM ran the business recovery process, but neither team had real-time visibility into the other's status.

**Solution:** Linked the ITSM Major Incident to the BCM Crisis Event via a relationship record and created a combined workspace view showing both technical resolution tasks and business recovery tasks side by side. When ITSM moved to Resolved, BCM was automatically notified to begin the stand-down phase.

---

### Challenge 4: Exercise Participation and Follow-Through

**Problem:** Business stakeholders treated BCM exercises as low priority — attendance dropped to 60% after the first two exercises, and action items from exercise findings were rarely completed.

**Solution:** Worked with the BCM program owner to tie exercise completion to a compliance metric tracked in IRM. Non-participation was logged as a policy exception, which had audit implications. Introduced lighter tabletop exercises (2-hour discussion-based) as an alternative to full simulation exercises for less-critical plans — this significantly improved engagement.

---

### Challenge 5: Multiple Out-of-Sync Plan Versions in Circulation

**Problem:** Business units were exporting BCPs as Word documents and editing them locally, leading to multiple conflicting versions. During a crisis drill, one team used a plan that was 6 months out of date with incorrect contact information.

**Solution:** Locked BCPs in ServiceNow as the system of record — restricted exports and configured the Employee Center to display a read-only "current plan" view. All updates had to go through a formal plan review workflow in ServiceNow with version history tracked automatically.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **BCP (Business Continuity Plan)** | Structured plan defining how an org responds to and recovers from a disruption — includes scope, RTOs, recovery procedures |
| **RTO (Recovery Time Objective)** | Maximum acceptable time to restore a process or system after disruption |
| **RPO (Recovery Point Objective)** | Maximum acceptable amount of data loss, measured in time |
| **BIA (Business Impact Analysis)** | Assessment of how critical each business process is and the impact of its loss |
| **BCM Exercise** | Simulated test of a BCP to validate completeness and executability |
| **Crisis Management** | The incident command process that activates during a real disruption |
| **Recovery Plan** | Step-by-step task list for restoring a specific system or process |
| **BCM-IRM Integration** | High-impact risks in IRM trigger BCM planning tasks automatically |
| **BCM-ITSM Integration** | P1 Major Incidents trigger Crisis Events for coordinated response |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Reduced crisis response coordination time — structured response initiated **within 30 minutes** of P1 declaration vs. ad-hoc coordination that previously took hours
- BCM exercise participation increased from **60% to 92%** after tying completion to compliance metrics in IRM
- Eliminated out-of-sync plan versions — **100% of active BCPs managed in ServiceNow** as system of record
- BIA coverage improved from **40% to 85%** of critical services with validated CI relationships
- RTO achievement in exercises improved from **72% to 89%** after closing task ownership gaps
