# ServiceNow ITSM — Interview Questions & Answers

---

## PART 1: Incident and Problem Management

**Q1: What is the difference between an Incident and a Problem?**
**A:**
- **Incident:** An unplanned interruption to a service or degradation in quality. Goal: restore service as fast as possible. Not concerned with root cause.
- **Problem:** The underlying root cause of one or more incidents. Goal: prevent incidents from recurring by eliminating the root cause.

**Example:** Email is down for 500 users (Incident). The investigation finds that a misconfigured DNS update caused the outage (Problem). The DNS fix is the permanent resolution.

---

**Q2: What is a Known Error?**
**A:** A Problem becomes a **Known Error** when the root cause has been identified but a permanent fix has not yet been applied. A Known Error typically has a documented workaround — a temporary solution that restores service without fixing the root cause.

**Why it matters:** Known Errors are visible to the service desk — when a new Incident comes in matching a Known Error, agents can apply the workaround immediately without re-investigating, reducing resolution time.

---

**Q3: How is Incident Priority calculated in ServiceNow?**
**A:** Priority is calculated from a combination of **Impact** (how many users or services are affected) and **Urgency** (how time-sensitive the fix is). A priority matrix maps the combination to a P1-P4 priority level.

In ServiceNow, the `priority` field on the Incident table is typically read-only and calculated automatically via a Business Rule: `(Impact × Urgency) → Priority`. This ensures consistent prioritization across all analysts.

---

**Q4: What happens during a Major Incident?**
**A:** A Major Incident (P1) triggers an enhanced response process:
1. A Major Incident bridge is convened with relevant resolver groups
2. A **Major Incident Manager** (separate from the technical resolver) coordinates communication
3. Stakeholder communications sent on a defined cadence (every 30-60 minutes)
4. All resolution steps documented in the incident work notes for audit trail
5. After resolution: a **Post-Incident Review (PIR)** is conducted
6. A **Problem record** is created to investigate root cause and prevent recurrence

In ServiceNow, Major Incidents can have dedicated **Major Incident Workspace** with guided playbooks for coordinators.

---

**Q5: What is the difference between Incident Resolution and Incident Closure?**
**A:**
- **Resolution:** The fix has been applied and service restored. The incident moves to "Resolved" state. The clock on resolution SLA stops.
- **Closure:** The requester confirms the issue is resolved, or the incident auto-closes after a configured number of days in Resolved state (typically 3-5 business days).

**Why the distinction matters:** Some organizations use the Resolved state for SLA measurement but require explicit closure for audit purposes. ITIL also recognizes that some incidents that appear resolved may recur — the closure confirmation step catches these.

---

**Q6: What is the Problem Management process flow?**
**A:**
1. **Problem identified** — from a Major Incident, from a trend of recurring incidents, or proactively
2. **Investigation** — Problem Tasks assigned to investigate root cause
3. **Root cause identified** — Problem becomes a Known Error, workaround documented
4. **Permanent fix planned** — Change Request raised to implement the fix
5. **Fix implemented** — via Change Management process
6. **Problem closed** — Root cause eliminated, associated Known Errors closed
7. **PIR completed** — Lessons learned documented

---

**Q7: What are the three types of Changes in ServiceNow ITSM?**
**A:**
| Type | Description | Risk | Approval |
|------|-------------|------|----------|
| **Standard** | Pre-approved, routine, low-risk (password reset, standard software install) | Low | Pre-approved — no per-change CAB review |
| **Normal** | Planned change requiring assessment and approval | Medium-High | CAB approval required |
| **Emergency** | Urgent fix to restore service, cannot wait for normal CAB | Variable | Emergency CAB (ECAB) — expedited |

---

**Q8: What is the Change Advisory Board (CAB)?**
**A:** The CAB is the group that reviews and approves Normal Changes before implementation. Typically meets weekly. Membership includes IT management, business representatives, and major service owners.

**CAB responsibilities:**
- Review change details, risk, and implementation plan
- Check for scheduling conflicts with other changes or business events
- Approve, reject, or request more information
- Review post-implementation results of previous changes

In ServiceNow, the CAB Workbench provides a structured interface for CAB members to review, vote, and approve changes with full CMDB context (what is the affected CI, what services depend on it).

---

**Q9: What is a Catalog Item vs. a Record Producer?**
**A:**
- **Catalog Item:** Creates an `sc_req_item` (RITM) record in the Service Catalog. Used when the requester needs IT to fulfill a request (e.g., "Order a new laptop").
- **Record Producer:** Creates a record in a different table (e.g., `incident`, `change_request`, a custom table) from a catalog-style form. Used when the request should create something other than a Request Item — for example, a self-service Incident submission form that creates an `incident` record directly.

**Key difference:** Catalog Item → creates RITM. Record Producer → creates a record in any table you specify.

---

**Q10: Explain the Request lifecycle — REQ, RITM, and Catalog Task.**
**A:**
- **REQ (sc_request):** The parent request record created when any catalog submission occurs. Contains billing/requester info. One REQ per submission session.
- **RITM (sc_req_item):** One RITM per catalog item in the request. If you order 3 things in one cart, you get 1 REQ with 3 RITMs.
- **Catalog Task (sc_task):** Sub-tasks of an RITM, each assigned to a different fulfillment team (e.g., provision laptop = Task 1: hardware team; Task 2: OS imaging team; Task 3: software install team).

**Flow:** Employee submits → REQ created → RITM(s) created → Approvals (if any) → Catalog Tasks created → All tasks done → RITM auto-closes → All RITMs done → REQ auto-closes.

---

## PART 2: Service Catalog, Knowledge, and SLA

**Q11: What is the difference between Impact and Urgency in Incident Management?**
**A:**
- **Impact:** How many users, processes, or services are affected — the breadth of the problem. ("Is it affecting 1 user or 500?")
- **Urgency:** How time-critical is the resolution — the business tolerance for the outage. ("Is this a month-end financial close or a standard workday?")

A P1 incident requires both high Impact AND high Urgency. A single executive's laptop being down might be High Urgency (they need it now) but Low Impact (only 1 user affected), which would typically make it P2, not P1.

---

**Q12: How do SLA pause conditions work?**
**A:** An SLA pause condition is a rule that stops the SLA timer when a specified condition is true — typically when the incident/request is waiting for something outside the IT team's control.

**Common pause conditions:**
- State = "On Hold" (waiting for requester, vendor, or third party)
- A specific field is set to a value (e.g., "Waiting for Customer" field = true)

**How to configure in ServiceNow:**
In the SLA Definition record, the **Pause Condition** field accepts a GlideRecord condition. When the condition evaluates to true, the SLA task moves to "Paused" state and the timer stops.

**Retroactive pause:** If an On Hold state was set before the pause condition was configured (common when adding pause conditions to existing SLA definitions), retroactive pause credit goes back to when the state was actually set, not when the condition was added.

---

**Q13: What are Business Rules and when do you use them in ITSM?**
**A:** Business Rules are server-side scripts that run automatically when a record is created, updated, or deleted — before or after the database operation.

**Common ITSM Business Rules:**
- **Priority calculation:** Automatically set priority based on Impact × Urgency matrix
- **Auto-assignment:** Route incident to correct group based on category/subcategory
- **SLA trigger:** Create SLA task when incident is created with certain priority
- **Field population:** Auto-fill caller's department and location from user record
- **State validation:** Prevent closing an incident without a Resolution Note

**When to use vs. Flow Designer:**
Business Rules are best for synchronous database-level logic (validation, field calculation, auto-population). Flow Designer is better for complex multi-step workflows (approvals, multi-system actions, conditional branching across multiple steps).

---

**Q14: What are Client Scripts and when do you use them in ITSM?**
**A:** Client Scripts run in the browser on the agent/user's form. They execute JavaScript client-side without requiring a round-trip to the server.

**Types:**
| Type | When it runs |
|------|-------------|
| **onLoad** | When the form loads |
| **onChange** | When a specific field value changes |
| **onSubmit** | When the form is submitted |

**Common ITSM Client Scripts:**
- Show/hide fields based on category selection (onChange)
- Validate that a field is populated before submission (onSubmit)
- Auto-populate fields client-side from another field's value (onChange)
- Display a warning message when a certain combination of values is selected (onChange)

---

**Q15: What are ACLs and how do they protect ITSM data?**
**A:** ACLs (Access Control Lists) are rules that control who can read, write, create, or delete records and fields in ServiceNow.

**ACL structure:**
- **Table ACL:** Controls access to the entire table (e.g., only members of `itil` role can read incident records)
- **Field ACL:** Controls access to a specific field within a table (e.g., only members of `itil_admin` role can edit the Priority field)

**ITSM use cases:**
- Prevent analysts from deleting incident records
- Restrict the Priority field to managers and above
- Allow requesters to view their own incidents but not others
- Prevent anyone from modifying a Closed incident record

**Important:** ACLs are evaluated server-side. They cannot be bypassed through client-side scripts. Always test ACLs logged in as the actual user persona, not as admin (admin bypasses all ACLs).

---

**Q16: What is the difference between a UI Policy and a Business Rule for field visibility?**
**A:**
| UI Policy | Business Rule |
|-----------|---------------|
| Runs client-side in the browser | Runs server-side |
| Controls field visibility, mandatory, read-only | Controls data creation, update, delete |
| Executes instantly as user changes fields | Executes on form save/submit |
| Cannot access other records | Can access and update any record |
| Best for: show/hide/mandatory based on field values | Best for: calculations, cross-record updates |

**Practical rule:** Use UI Policy for anything that affects how the form looks and behaves while the user is filling it in. Use Business Rule for anything that needs to persist data or affect other records.

---

## PART 3: Scripting, Administration, and Scenarios

**Q17: What is GlideRecord and how is it used in ITSM?**
**A:** GlideRecord is the ServiceNow JavaScript API for querying and modifying database records in server-side scripts.

**Common GlideRecord patterns in ITSM:**

```javascript
// Query incidents assigned to a group
var gr = new GlideRecord('incident');
gr.addQuery('assignment_group', 'IT Support');
gr.addQuery('state', 'IN', '1,2'); // New or In Progress
gr.query();
while (gr.next()) {
    gs.log(gr.number + ' - ' + gr.short_description);
}

// Update a specific incident
var gr = new GlideRecord('incident');
if (gr.get('sys_id', 'abc123...')) {
    gr.state = 6; // Resolved
    gr.close_code = 'Solved (Permanently)';
    gr.close_notes = 'Resolved by automated script';
    gr.update();
}
```

**Best practices:**
- Always use `addQuery()` — never build raw query strings (injection risk)
- Check `gr.next()` return value before accessing fields
- Limit fields returned with `gr.setFields()` for performance in large queries

---

**Q18: Describe a scenario where you would use a Scheduled Job in ITSM.**
**A:** **Scenario:** Close all Resolved incidents that have been in Resolved state for more than 5 business days without a response from the requester.

```javascript
// Scheduled Job: Auto-close resolved incidents
var gr = new GlideRecord('incident');
gr.addQuery('state', 4); // Resolved
gr.addQuery('resolved_at', '<', gs.daysAgoStart(5)); // Resolved more than 5 days ago
gr.query();
while (gr.next()) {
    gr.state = 7; // Closed
    gr.close_notes = 'Auto-closed: no response from requester within 5 days of resolution';
    gr.update();
}
```

Other common scheduled job uses in ITSM:
- SLA warning notifications before breach
- Escalating incidents unassigned for X hours
- Generating weekly SLA compliance reports
- Purging old closed incidents per retention policy

---

**Q19: What is the Update Set process and why is it important in ITSM development?**
**A:** An **Update Set** is a container that captures all configuration changes made in a ServiceNow instance and allows them to be moved to other instances (DEV → TEST → UAT → PROD).

**Why it matters for ITSM:**
- All ITSM customizations (Business Rules, Client Scripts, Catalog Items, SLA definitions) must travel via Update Set to move across environments
- Without Update Sets, DEV and PROD drift out of sync and changes get lost

**Best practices:**
- One Update Set per story/change — never put unrelated changes in the same set
- Name consistently: `PROJECT-ticketnum-short-description`
- Never commit directly to PROD — always promote through TEST and UAT first
- Preview an Update Set before committing — check for conflicts with existing records

---

**Q20: How would you troubleshoot an SLA that is breaching unexpectedly?**
**A:**
1. **Check the SLA Definition:** Confirm the correct SLA definition is attached to the incident. Check the Start Condition (what triggers the SLA), Pause Condition (what pauses it), and Stop Condition (what ends it).

2. **Review the SLA Task record:** Open the `task_sla` record on the incident. Check the Business Elapsed Time vs. Actual Elapsed Time — if they match, the SLA is not pausing correctly. If Business Elapsed Time is much less than Actual, the business hours schedule may be misconfigured.

3. **Check the Business Hours Schedule:** Verify the schedule is correct for the timezone and does not have gaps or incorrect holiday entries.

4. **Check the Pause Condition:** If the incident was On Hold, verify the pause condition on the SLA Definition includes the On Hold state. Test the condition expression against the incident.

5. **Check for retroactive start:** If the SLA start condition was triggered before the SLA definition was active, the elapsed time calculation may be wrong.

6. **Use SLA Debugger:** ServiceNow has a built-in SLA debugger that shows exactly why an SLA is in its current state — available from the incident form via the SLA related list.
