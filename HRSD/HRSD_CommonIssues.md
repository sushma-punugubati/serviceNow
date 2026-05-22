# ServiceNow HRSD — Common Issues, Pitfalls & Troubleshooting

Issues encountered during HRSD implementations and ongoing support, based on ServiceNow community posts, implementation experience, and real-world projects.

---

## Issue 1: Cases Routing to the Wrong COE or Assignment Group

**Category:** Configuration / Routing  
**Severity:** High — HR requests land with the wrong team; SLAs breach; employee trust erodes

**The Problem:**
An employee submits a payroll inquiry and it lands in the Benefits queue. Or general HR cases pile up in a single default assignment group because routing rules are missing. HR agents spend time re-routing rather than resolving.

**Why It Happens:**
HR Service records are not mapped to the correct COE and Topic Detail. When the mapping is incomplete, ServiceNow uses the default assignment group on the HR Service record rather than routing by COE-specific logic. Also common: routing conditions using HR Criteria reference fields that are blank on many HR Profiles because data migration was incomplete.

**Root Cause:**
- HR Services created without COE assignment
- HR Criteria conditions referencing HR Profile fields that are null
- No Assignment Rules configured for the COE — just relying on HR Service default

**Fix / Prevention:**
- Audit all HR Services: verify each has a COE, Topic Category, Topic Detail, and assignment group
- Test HR Criteria by checking actual HR Profile records for your test users — null fields silently fail
- Build Assignment Rules (not just HR Service defaults) for flexibility — Assignment Rules fire before the HR Service default
- Add a catch-all assignment group per COE: cases that match the COE but no specific rule land in the team inbox

---

## Issue 2: Lifecycle Event Activities Not Firing

**Category:** Configuration / Lifecycle Events  
**Severity:** High — onboarding/offboarding tasks missing; employees don't get laptops, access, etc.

**The Problem:**
A new hire onboarding lifecycle event is triggered but only 3 of 12 activities fire. IT never receives the laptop provisioning task. The new hire starts day 1 with no equipment and no system access.

**Why It Happens:**
Activity conditions are the most common culprit. Each activity in a lifecycle event can have a condition — if the condition evaluates to false (or errors), the activity silently skips. Common causes:
- Activity condition references HR Profile field that is blank at hire time
- Offset date calculation results in a past date (e.g., "fire 7 days before start date" but the hire was entered with only 2 days' notice)
- Activity is set to a state that prevents firing (Inactive, Inactive for testing)

**Root Cause:**
Lifecycle Event Activity conditions are not validated against realistic HR data. Activities fail silently — no error visible to HR or IT.

**Fix / Prevention:**
- Test lifecycle events with complete HR Profile data — don't test with placeholder profiles
- Review all activity conditions: ensure referenced fields are populated at trigger time
- Add a default/catch-all activity with no condition for critical tasks (laptop provisioning, badge creation)
- Build a monitoring process: after each lifecycle event fires, run a report of "activities expected vs. activities created" — flag any gap
- Notify HR admins when lifecycle event activity count is below expected range

---

## Issue 3: Employee Document Management Access Too Broad or Too Restrictive

**Category:** Security / Configuration  
**Severity:** High — data privacy risk or HR agents can't access documents they need

**The Problem:**
Scenario A: A payroll specialist can view employee disciplinary documents stored by Employee Relations — a privacy violation. Scenario B: An HR Business Partner can't access offer letters they need to process a role change because the document category is locked to Recruiting only.

**Why It Happens:**
Document categories in EDM are created without explicit role-based restrictions, so they inherit the default HR data visibility (all HR agents). Or, restrictions are applied so tightly that even required roles are excluded.

**Root Cause:**
- Document category security not configured — defaults to broad HR visibility
- COE security policies not extended to EDM document categories
- Role assignments for document access done incorrectly (role name vs. COE-specific group)

**Fix / Prevention:**
- Map every document category to the COE that owns it — only that COE's agents can access
- Create a data classification matrix: document type → roles allowed → COE restriction
- Test with personas: log in as a Benefits analyst and confirm they cannot see Payroll or ER documents
- Encrypt sensitive fields (salary, SSN, medical) using column-level encryption even for authorized HR agents — only elevate to field-level on a need-to-see basis

---

## Issue 4: HR Criteria Silently Failing — Wrong Employees See HR Services

**Category:** Configuration  
**Severity:** Medium — wrong HR services visible to wrong employee populations

**The Problem:**
A "Parental Leave" HR Service is configured to show only to full-time permanent employees. But it's also appearing for contractors. Or a US-only HR Service appears for employees in India.

**Why It Happens:**
HR Criteria are configured correctly in isolation but the HR Profile records for some employees are missing the field values the criteria depend on. When the field is blank, the HR Criteria evaluation returns null — which is treated as "not excluded" rather than "excluded." The service appears for everyone whose profile has an empty field.

**Root Cause:**
HR Criteria uses a positive-match logic: the service shows if the criteria match OR if criteria can't be evaluated (null). Missing HR Profile data = criteria can't evaluate = service appears by default.

**Fix / Prevention:**
- Ensure HR Profile records are complete before launching HRSD — run a data quality report
- For HR Criteria based on Employment Type or Location: set defaults on the HR Profile record during data migration
- Use negative/exclusion criteria where possible: "Hide from contractors" is safer than "Show only to full-time"
- Test HR Criteria with a matrix: employee type A, B, C vs. service visibility expected — run regression before every release
- After go-live, run monthly HR Profile completeness audit

---

## Issue 5: Lifecycle Event Timing Issues — Tasks Fire Too Early or Too Late

**Category:** Configuration  
**Severity:** Medium-High — tasks arrive before HR can act; or arrive after the employee starts

**The Problem:**
The "Create IT accounts" task in the onboarding lifecycle event fires 14 days before the hire date — IT gets the request when a background check hasn't been completed yet, creating accounts for candidates who might not be hired. Or the "Recover assets" offboarding task fires a day after the last day, so IT shows up after the ex-employee has already left.

**Why It Happens:**
Lifecycle Event activity offsets are set relative to the event trigger date without accounting for upstream process dependencies. Offset logic is configured in days — but doesn't account for business days, weekends, or timezone differences.

**Root Cause:**
- Offsets configured in calendar days not business days
- No gating condition: "only fire if background check passed"
- Trigger date set to "hire date" but process requires "background check complete date"

**Fix / Prevention:**
- Map each activity's required offset to the real business process — interview the HR process owners, not just the system admin
- Use conditions on activities to gate timing-sensitive tasks: "fire only if HR Profile status = 'Background Cleared'"
- For critical offboarding tasks (asset recovery, access revocation): fire 1 day BEFORE last day, not on last day
- Consider multiple lifecycle event triggers for onboarding phases: one for "background cleared," one for day 1

---

## Issue 6: Bulk Case Creation Causing Performance Issues

**Category:** Performance / Scale  
**Severity:** Medium — system slowdown during mass HR events

**The Problem:**
An HR admin uses bulk case creation to send an open enrollment reminder to 15,000 employees. The system runs for 2 hours, the instance becomes slow for all users, and half the cases fail silently without any error message to the admin.

**Why It Happens:**
Bulk case creation in HRSD creates one case record per employee synchronously when not chunked properly. Large populations overwhelm the system if triggered without throttling or async processing.

**Root Cause:**
- Bulk case creation triggered synchronously for large populations
- No chunking or async execution configured
- No error handling for partial failures

**Fix / Prevention:**
- For bulk operations >500 employees: always use scheduled jobs or background processing
- Test bulk case creation in sub-UAT environments with realistic record counts before production
- Use targeted HR Criteria to limit the affected population to exactly who needs the case
- Build a post-run reconciliation: count cases created vs. population size — alert on mismatch

---

## Issue 7: Employee Center Not Showing the Right HR Services

**Category:** Configuration / Portal  
**Severity:** Medium — employees can't find or submit the right request

**The Problem:**
Employees log into the Employee Center but can't find the "Change Direct Deposit" service. The HR team says it's configured, but employees see a completely different set of services than what HR expects.

**Why It Happens:**
Multiple layers of visibility control affect what appears in the Employee Center:
1. HR Service must be Active
2. HR Service must be associated with the correct Topic Detail/Category
3. HR Criteria on the service must match the employee's HR Profile
4. The Employee Center page/topic must be configured to show that category
5. The service must be marked as "Available via portal"

Any one of these being wrong = service invisible to the employee.

**Root Cause:**
HR Services are created and marked active, but the portal visibility settings and HR Criteria are not validated end-to-end from the employee's perspective.

**Fix / Prevention:**
- Test every HR Service by logging in as an actual employee persona (use impersonation)
- Create a test matrix: employee type → expected visible services → actual visible services
- After any HR Service change, run a portal smoke test before notifying employees
- Document the "visibility checklist" for every HR Service — active, portal-available, criteria set, category linked

---

## Issue 8: Integration with Workday Failing — Lifecycle Events Not Triggering

**Category:** Integration  
**Severity:** High — onboarding/offboarding automation breaks; manual work floods HR

**The Problem:**
New hires are entered in Workday but the onboarding lifecycle event in ServiceNow never fires. HR has to manually create onboarding cases for every new hire — the automation that was the core value of the HRSD implementation isn't working.

**Why It Happens:**
The Workday-to-ServiceNow integration (typically via IntegrationHub) sends hire data to ServiceNow, but the trigger condition for the lifecycle event isn't met because:
- The HR Profile record wasn't created/updated by the integration payload
- The lifecycle event trigger condition checks a field the integration doesn't populate
- The integration payload arrives but lacks the hire date, causing the offset calculation to fail

**Root Cause:**
Integration field mapping between Workday's data model and ServiceNow's HR Profile fields is incomplete. The lifecycle event trigger fires on HR Profile insert/update but the critical trigger fields aren't populated.

**Fix / Prevention:**
- Map every Workday field the lifecycle event trigger depends on — document it before integration build
- Build integration validation: after each Workday payload, verify HR Profile was created with all required fields
- Test the full end-to-end flow: Workday hire → IntegrationHub → HR Profile creation → lifecycle event trigger → activities fire
- Add dead-letter queue monitoring: Workday events that fail to process alert the integration admin immediately

---

## Issue 9: HR Agents Cannot View Cases Assigned to Their Group

**Category:** Security / Roles  
**Severity:** High — HR agents can't do their work

**The Problem:**
A newly onboarded HR Benefits agent has the `sn_hr_core.case_writer` role but cannot see any cases in the Benefits COE queue. The cases exist — the agent just can't see them.

**Why It Happens:**
The `sn_hr_core.case_writer` role grants the ability to write, but COE-level security policies restrict visibility to only the agent's COE assignment group. If the agent's user record is not a member of the Benefits COE assignment group, they see nothing — even with the writer role.

**Root Cause:**
Role assignment and assignment group membership are two separate configurations. The agent was given the role but not added to the correct assignment group.

**Fix / Prevention:**
- Onboarding checklist for HR agents: role + assignment group membership (both required)
- Create an HR group management process: new HR agent → role + group assigned in one workflow
- Test with new agent personas during UAT: verify case visibility before go-live
- Quarterly audit: HR agents with case_writer role but no COE group assignment = flag for review

---

## Issue 10: Case Templates Not Applied — HR Agents Re-entering Same Data Repeatedly

**Category:** Configuration / Usability  
**Severity:** Low-Medium — inefficiency; data entry errors; inconsistent case records

**The Problem:**
HR agents creating "New Hire Benefits Enrollment" cases enter the same 8 fields manually every time. No template auto-populates standard fields. Cases end up with inconsistent data because each agent does it differently.

**Why It Happens:**
Case Templates were not created during implementation. The HR Service is linked to a COE but not to a template. Templates require explicit configuration — they don't auto-generate from HR Service setup.

**Root Cause:**
Templates are a separate configuration step often overlooked in implementations focused on "getting routing working." The team assumed templates were optional and skipped them.

**Fix / Prevention:**
- For every high-volume HR Service, create a Case Template with standard fields pre-populated
- Link the template to the HR Service — when an agent creates a case from that service, the template auto-applies
- Create templates for: standard routing fields, assignment groups, priority, description boilerplate, fulfillment instructions
- Treat template creation as a go-live requirement for the top 20 HR Services

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Cases routing to wrong team | HR Service not mapped to COE or routing rules missing | Audit HR Service COE/Topic Detail assignments |
| Lifecycle event activities not firing | Activity conditions fail due to blank HR Profile fields | Check HR Profile completeness for trigger population |
| HR Services not visible on Employee Center | Visibility condition, HR Criteria, or portal config | Impersonate employee and check each visibility layer |
| Documents accessible by wrong team | EDM document category not restricted to owning COE | Map document categories to COE security policies |
| Onboarding not triggered after Workday hire | Integration not populating lifecycle trigger fields | Validate Workday payload → HR Profile field mapping |
| HR agent sees no cases | Has role but not added to COE assignment group | Add agent to the correct assignment group |
| Bulk case creation slow/failing | Population too large for synchronous execution | Switch to async/scheduled processing |
| Tasks firing on wrong day | Offset in calendar days, not business days | Review offset logic and add business day calculation |
