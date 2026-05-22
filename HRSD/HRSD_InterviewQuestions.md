# ServiceNow HRSD — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is ServiceNow HRSD and what problem does it solve?**
**A:** HR Service Delivery (HRSD) is ServiceNow's module that applies the same structured case management, routing, SLA tracking, and self-service capabilities used in ITSM to HR operations. The core problem it solves:
- Employees emailing HR inboxes get inconsistent, untracked responses
- HR teams have no visibility into workload, SLAs, or bottlenecks
- Cross-functional events (onboarding, offboarding) require manual coordination across IT, Facilities, HR, and Finance

HRSD turns every HR request into a trackable case with an owner, SLA, and audit trail — and automates multi-department events like onboarding via Lifecycle Events.

---

**Q2: What is a Center of Excellence (COE) in HRSD?**
**A:** A COE is a functional grouping of HR services that maps to a specific HR department (Benefits, Payroll, Employee Relations, Workforce Administration, etc.). COEs serve as the routing backbone of HRSD:
- Cases are routed to a COE, then to an assignment group within it
- Each COE has its own security policy — Benefits agents cannot see Payroll cases
- The hierarchy is: COE → Topic Category → Topic Detail → HR Service

COEs let you model your HR org structure in ServiceNow so routing mirrors real-world HR team boundaries.

---

**Q3: What is the difference between HR Criteria and User Criteria?**
**A:**
- **HR Criteria**: Evaluated against HR Profile fields — employment type, location, job code, HR-specific attributes. Used to show/hide HR Services based on who the employee is from an HR perspective.
- **User Criteria**: Evaluated against standard user fields — role, group, company, department from the `sys_user` table. Used broadly across all ServiceNow modules.

**When to use HR Criteria:** When visibility depends on employment status, job function, or any field that lives on the HR Profile record rather than the user record. Example: show "Parental Leave" only to full-time permanent employees — that distinction lives on the HR Profile, not the user record.

---

**Q4: What is a Lifecycle Event and how does it work?**
**A:** A Lifecycle Event is a bundled set of tasks that fires automatically when a significant HR event occurs — onboarding, offboarding, transfer, leave of absence. It coordinates work across multiple departments simultaneously.

Architecture:
- **Lifecycle Event Definition** → **Activity Sets** (phases: Pre-Start, Day 1, First 30 Days) → **Activities** (individual tasks with conditions and offsets)
- Activities can be HR Tasks, Service Catalog tasks, Approvals, or Notifications
- Each activity has an **offset** (when relative to the event date) and a **condition** (whether to fire)

The power: one hire in Workday triggers a lifecycle event in ServiceNow that automatically creates IT provisioning tasks, Facilities badge tasks, HR paperwork tasks, and manager orientation tasks — all routed to the correct teams.

---

**Q5: What are the HR Case states and what do they mean?**
**A:**
| State | Meaning |
|-------|---------|
| Draft | Being created; not yet submitted to HR |
| Ready | Submitted; awaiting assignment/pickup |
| Work in Progress | Actively being resolved by HR agent |
| Awaiting Approval | Pending HR manager or stakeholder sign-off |
| Awaiting Acceptance | Waiting for employee to accept/acknowledge |
| Suspended | On hold — waiting for external input or time-based |
| Closed Complete | Resolved successfully |
| Closed Incomplete | Closed without full resolution |
| Cancelled | Withdrawn before completion |

---

**Q6: What is Restricted Caller Access (RCA)?**
**A:** RCA is a security setting at the COE level that prevents employees from directly creating cases in that COE via the platform or API. When RCA is enabled, employees must use the Employee Center self-service portal — the system acts as the caller on their behalf. 

**Why use it:** For sensitive COEs (Employee Relations, Payroll, Investigations), you don't want employees bypassing the intake workflow or manipulating case fields directly. RCA enforces routing and ensures all cases go through the proper intake funnel.

---

**Q7: What is Employee Document Management (EDM) and how is it secured?**
**A:** EDM is the capability for storing and managing sensitive HR documents (offer letters, I-9s, performance reviews, accommodation requests). Security mechanisms:
- **Column-level encryption**: Documents encrypted by default in the database
- **Role-based access**: Only roles explicitly granted access to a document category can view/download
- **COE-linked document categories**: Documents owned by a COE are visible only to that COE's agents
- **Edge Encryption**: For highest-sensitivity documents — encrypted before leaving ServiceNow servers
- **Retention Policies**: Documents auto-purged after retention period per legal requirements

---

**Q8: What is Employee Journey Management (EJM) and how does it differ from Lifecycle Events?**
**A:**
- **Lifecycle Events**: The operational layer — creates and assigns tasks to teams (IT, HR, Facilities). Focused on ensuring work gets done.
- **Employee Journey Management**: The experience layer — creates a personalized journey visible to the employee. Focused on the employee's perspective and engagement.

EJM adds:
- **Journey Maps**: Visual onboarding/offboarding progress the employee sees in the Employee Center
- **Campaigns**: Proactive HR check-ins and nudges (30-day survey, anniversary messages)
- **Experience Packs**: Pre-built journey templates for specific industries or employee types
- **Moments**: Point-in-time notifications delivered at meaningful milestones

Think of it this way: Lifecycle Events ensure IT gets the laptop provisioned. EJM ensures the new hire knows it's coming and feels welcomed throughout the process.

---

**Q9: How does HRSD integrate with Workday?**
**A:** The Workday integration is typically bidirectional via IntegrationHub:
- **Workday → ServiceNow**: Hire events, job data changes, termination events trigger lifecycle events in ServiceNow. Employee data (name, location, department, job code) populates the HR Profile.
- **ServiceNow → Workday**: LOA approvals, job data updates made in ServiceNow can push back to Workday as the system of record.

The integration uses Workday's REST APIs (or SOAP for older versions). A common pattern: Workday fires a webhook or HRSD polls Workday on a schedule; the payload creates/updates the HR Profile, which triggers the lifecycle event condition.

---

**Q10: What roles are involved in an HRSD implementation?**
**A:**
| Role | Responsibilities |
|------|-----------------|
| `sn_hr_core.admin` | Full configuration access — COEs, services, lifecycle events, security |
| `sn_hr_core.manager` | View all cases in their COE, manage queues, run reports |
| `sn_hr_core.case_writer` | Create and update cases in their assigned COE |
| `sn_hr_core.case_reader` | Read-only access to cases in their COE |
| `sn_hr_core.basic` | Employee self-service — submit requests through Employee Center |

---

## PART 2: Technical Deep Dive

**Q11: A lifecycle event fires but only 3 of 12 activities create tasks. What do you check first?**
**A:** Activity conditions. Each activity has an optional condition — if it evaluates to false, the activity silently skips. Systematic check:
1. Open the Lifecycle Event record → check the Activities list → look for activities in "Skipped" or missing state
2. For each missing activity: review the condition expression — check if it references an HR Profile field that was null at trigger time
3. Check the offset: if the offset produces a past date (e.g., −14 days from hire date but hire was entered late), the activity won't fire
4. Check the Activity state — is it Active? An Inactive activity never fires
5. Enable the lifecycle event debug log to see evaluation results for each activity

---

**Q12: How would you configure an onboarding lifecycle event that has different tasks for US vs. non-US employees?**
**A:** Two approaches:

**Option A (Conditions within one event):** Build one lifecycle event with all activities. For each US-specific activity, add a condition: `current.u_work_country == 'US'`. For EU activities: `current.u_work_country != 'US'`. Clean single event, more conditions to maintain.

**Option B (Separate events per region):** Create a "US Onboarding" lifecycle event and a "Global Onboarding" lifecycle event. Use HR Criteria on each: the US event fires only when work_country = US; the global event fires otherwise. Cleaner activity lists, more lifecycle event records to manage.

**Best practice:** For minor variations (3-4 different activities), use conditions within one event. For fundamentally different processes (US has 20 steps, EU has 8 completely different steps), use separate events.

---

**Q13: An HR Benefits agent says she can't see any cases in her queue. What's your troubleshooting approach?**
**A:** Check in this order:
1. **Role**: Does she have `sn_hr_core.case_writer` or `sn_hr_core.case_reader`? (Check `sys_user_has_role`)
2. **Assignment group**: Is she a member of the Benefits COE assignment group? Role alone isn't enough — group membership controls visibility
3. **COE Security Policy**: Verify the Benefits COE security policy includes her role
4. **Active cases**: Are there actually cases in the Benefits queue right now? (Confirm with admin view)
5. **Filter issue**: Is she accidentally filtering to "My Cases" only? (Portal filter vs. all-group-cases filter)

The most common cause: agent was given the role but wasn't added to the assignment group — two separate steps that both must be completed.

---

**Q14: How does the Employee Center determine which HR Services an employee sees?**
**A:** Multiple layers evaluated in sequence:
1. **HR Service Active flag**: Only active services are eligible
2. **"Available via portal" flag**: Service must be marked as available in the Employee Center
3. **HR Criteria**: If HR Criteria are configured on the service, the employee's HR Profile fields must match
4. **Employee Center page configuration**: The Topic Category/Detail must be linked to the Employee Center topic page
5. **User Criteria** (if configured): Broader user-level conditions

All layers must pass. A service invisible to an employee is almost always failing at HR Criteria (blank HR Profile field) or the portal availability flag.

---

**Q15: What is the difference between an HR Task and a Catalog Task within a Lifecycle Event?**
**A:**
- **HR Task** (`sn_hr_core_task`): Owned by the HR module. HR agents work these in the HR Workspace. Used for HR-specific work: process paperwork, verify data, contact employee.
- **Catalog Task** (from Service Catalog): Owned by the ITSM/catalog framework. IT, Facilities, or other non-HR teams work these in their own queue. Used for: provision laptop, create AD account, assign desk, order badge.

In a lifecycle event, you use both: HR Tasks for what HR does, Catalog Tasks for what IT, Facilities, and other teams do. This keeps HR work in HR Workspace and IT work in IT queues — right tool for the right team.

---

## PART 3: Scenario-Based Questions

**Q16: You receive a requirement: "When an employee is terminated, their manager should receive a task to collect the laptop within 24 hours." How would you build this?**
**A:**
1. Open (or create) the Offboarding Lifecycle Event
2. Add an Activity in the "Day of Termination" Activity Set
3. Activity type: Catalog Task (routes to IT or Facilities)
4. Assignment: route to the terminated employee's manager (use dynamic assignment: `current.manager`)
5. Offset: 0 (fires same day as lifecycle event trigger)
6. Add SLA: task due within 24 hours (configure on the Catalog Task template)
7. Add escalation: if not complete in 24 hours, escalate to IT manager
8. Condition: no condition — always fire for all terminations (or add condition if only for employees with issued laptops: check ITAM asset assigned)

---

**Q17: HR says that employees in India are seeing a US-only "FMLA Leave" HR Service. How do you fix this?**
**A:**
The service lacks proper HR Criteria or the HR Criteria condition isn't working.

1. Open the "FMLA Leave" HR Service record
2. Check the HR Criteria field — if empty, the service is visible to everyone
3. Add HR Criteria: Work Country = United States (or Employment Location = US)
4. Verify that all India-based employees have "Work Country" populated on their HR Profile (if it's blank, criteria evaluation defaults to visible)
5. Test by impersonating an India-based employee — confirm service no longer appears
6. Test by impersonating a US employee — confirm service still appears

Root cause is almost always blank HR Profile data making the criteria unevaluable — fix the data before relying on the criteria.

---

**Q18: A company runs a global offboarding and wants different tasks for employees who were in management roles vs. individual contributors. How do you handle this?**
**A:** Use conditions on activities within the offboarding lifecycle event:

- All activities that apply to everyone: no condition
- Manager-specific activities (e.g., "Transition direct reports" task to HR HRBP): condition = `current.management_level != null` or `current.u_is_manager == true`
- IC-specific activities (e.g., "Complete knowledge transfer doc" task): condition = `current.management_level == null`

No need for separate lifecycle events — conditions within one event handle the branching cleanly. This assumes the HR Profile has a `management_level` or manager flag field populated from Workday.

---

**Q19: How would you support a company that has 4 different onboarding checklists depending on employment type?**
**A:** Build 4 separate Lifecycle Events — one per employment type:
- Full-time Onboarding
- Contractor Onboarding
- Intern Onboarding
- Student Worker Onboarding

Use a Lifecycle Event trigger condition for each:
- Full-time: `current.u_employment_type == 'Full-Time'`
- Contractor: `current.u_employment_type == 'Contractor'`

For shared activities (background check, ID badge), create Activity Templates and reference them in all 4 events. This avoids duplicating and allows shared tasks to be updated centrally.

---

**Q20: What happens if a Workday hire event arrives in ServiceNow before the HR Profile can be fully populated?**
**A:** This is the timing race condition in integrations. If the lifecycle event trigger fires before the HR Profile has all required fields (location, employment type, job code), any activity that references those fields in its condition will skip — silently.

**How to handle it:**
1. Use a wait condition or delay on the lifecycle event trigger: "wait until HR Profile is complete" (check for non-null required fields)
2. Configure the integration to send a second "profile complete" event after all Workday fields are mapped
3. Build the Workday integration to populate all lifecycle-trigger-dependent fields in the initial payload before the trigger fires
4. Add a monitoring job: 2 hours after lifecycle event fires, check expected activity count — if less than minimum, alert HR admin

---

## PART 4: Employee Journey Management

**Q21: What is an Experience Pack in EJM?**
**A:** An Experience Pack is a pre-built, reusable content bundle for a specific employee journey or persona. It includes:
- A journey template (the structure and milestones)
- Pre-built content (welcome messages, checklists, resources)
- Campaign templates (30-day check-in surveys, manager nudges)

Experience Packs are industry or role-specific: "Healthcare Onboarding Pack," "Engineering Onboarding Pack," "Remote Employee Onboarding Pack." They reduce configuration time significantly vs. building a journey from scratch.

---

**Q22: How do Journey Maps benefit employees vs. how do Lifecycle Events benefit HR?**
**A:**
- **Lifecycle Events (HR perspective)**: Ensures all tasks get created, assigned, and tracked across departments. HR and IT see their work queue; managers see what's outstanding.
- **Journey Maps (employee perspective)**: The employee sees a visual timeline of their onboarding milestones — "Background check ✓, Equipment provisioning 🔄, Training scheduled ✓." Reduces anxiety and proactive "when will I get my laptop?" emails.

They work together: Lifecycle Events do the work; Journey Maps show the employee the status of that work in a human-friendly way.

---

## PART 5: Exam-Style Quick Facts

| Question | Answer |
|----------|--------|
| Base HR Case table | `sn_hr_core_case` |
| HR Profile table | `sn_hr_core_profile` |
| HR Service table | `sn_hr_core_service` |
| HR Criteria table | `sn_hr_core_criteria` |
| Core plugin | `com.sn_hr_core` |
| Lifecycle Events plugin | `com.sn_hr_core.lifecycle_events` |
| Employee Center plugin | `com.sn_hr_core.esc` |
| COE = | Center of Excellence |
| EJM = | Employee Journey Management |
| EDM = | Employee Document Management |
| RCA = | Restricted Caller Access |
| Who can create ER cases with RCA on | HR agents only (not employees directly) |
| HR Criteria vs User Criteria | HR Criteria = HR Profile fields; User Criteria = sys_user fields |
| Lifecycle Event > Activity Set > Activity | Hierarchy within a lifecycle event |
| Activity offset | Days before/after event trigger date when activity fires |

---

## Study Checklist
- [ ] Know: COE hierarchy — COE → Topic Category → Topic Detail → HR Service
- [ ] Know: HR Case states in order and what triggers each
- [ ] Know: HR Criteria vs User Criteria — when to use each
- [ ] Know: Lifecycle Event structure — Activity Sets, Activities, Offsets, Conditions
- [ ] Know: Restricted Caller Access and why it's used
- [ ] Know: EDM security — column-level encryption, role-based access, COE-linked categories
- [ ] Know: Employee Center vs Employee Center Pro differences
- [ ] Know: EJM — Journey Maps vs Lifecycle Events (operational vs. experiential)
- [ ] Know: Common troubleshooting — activities not firing, routing failures, visibility issues
- [ ] Know: Key tables — sn_hr_core_case, sn_hr_core_profile, sn_hr_core_service
- [ ] Know: Workday integration pattern and common failure points
- [ ] Know: Roles — admin, manager, case_writer, case_reader, basic
