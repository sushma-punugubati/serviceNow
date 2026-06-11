# ServiceNow ITSM — Common Issues, Pitfalls & Troubleshooting

Issues encountered during ITSM implementations and ongoing support, based on real-world implementations and ServiceNow community experience.

---

## Issue 1: SLA Not Pausing When Incident Goes On Hold

**Category:** Configuration / SLA
**Severity:** High — SLA breaches reported that are not the team's fault

**The Problem:**
Analysts put incidents On Hold (waiting for vendor response) but the SLA timer keeps running. When the vendor responds 2 days later, the incident breaches SLA even though IT was not responsible for the delay. The team's SLA compliance metrics are being unfairly penalized.

**Why It Happens:**
The SLA Definition's Pause Condition does not include the On Hold state. Alternatively, "On Hold" is a custom state that was added after the SLA definition was created — the pause condition uses a hardcoded state number that did not include the new state.

**Root Cause:**
SLA Pause Condition not updated when the On Hold state was added or the pause logic was not configured during initial SLA setup.

**Fix / Prevention:**
- Open the SLA Definition record and update the Pause Condition to include: `state = 3` (On Hold) or use the state name if using a display value condition
- If using dynamic pause conditions (wait reason field), ensure all wait values trigger the pause
- After fixing, run a retroactive SLA recalculation for recent incidents in On Hold state
- During SLA setup, always create and test pause conditions for all "waiting" states before go-live

---

## Issue 2: Incidents Being Created for Every Email Reply — Duplicate Tickets

**Category:** Configuration / Inbound Email
**Severity:** High — ticket volume inflated, agents waste time managing duplicates

**The Problem:**
When customers reply to incident notifications (which contain the incident number in the subject), a new incident is created instead of the reply being added to the existing ticket. The ITSM queue fills with duplicate tickets and customers get confused by multiple ticket numbers.

**Why It Happens:**
The inbound email action is not parsing the existing ticket number from the email subject or thread ID. The email action is configured to always create a new incident regardless of whether a matching open ticket exists.

**Root Cause:**
Missing "check for existing ticket" logic in the inbound email action script.

**Fix / Prevention:**
- Update the inbound email action to parse the ticket number from the email subject (regex: `INC[0-9]+`)
- Add a pre-action script that checks for an open incident with the matching number
- If found: add email body as work note and update state from "Awaiting Response" to "In Progress"
- If not found: create new incident as normal
- Test with reply emails before go-live — this is one of the most common ITSM gotchas

---

## Issue 3: Assignment Groups Not Routing Correctly — Incidents Landing in Wrong Queue

**Category:** Configuration / Routing
**Severity:** High — incidents not actioned by the right team

**The Problem:**
Incidents categorized as "Network / VPN" are routing to the "Help Desk" group instead of the "Network Team" group. VPN incidents sit unactioned in the Help Desk queue until someone manually reassigns them.

**Why It Happens:**
Assignment rules are either: (1) not configured for this category/subcategory combination, (2) configured with incorrect conditions, or (3) overridden by a Business Rule that sets a default assignment group on all incidents regardless of category.

**Root Cause:**
Conflicting assignment logic — a Business Rule sets `assignment_group = Help Desk` on every new incident, then the Assignment Rule tries to set the correct group, but Business Rule runs after and overwrites it.

**Fix / Prevention:**
- Review assignment rule priority and Business Rule execution order
- Use Assignment Rules (in sys_assignment_rule) rather than Business Rules for routing — they are designed for this purpose and run in a predictable order
- If a Business Rule sets a default group, add a condition: only set default group if no assignment rule has already assigned the ticket
- Test routing by creating incidents in each category/subcategory combination and confirming correct assignment group

---

## Issue 4: Catalog Item Approval Workflow Not Triggering

**Category:** Configuration / Service Catalog
**Severity:** Medium-High — requests fulfilled without required approval

**The Problem:**
Expensive hardware requests (laptops over $2,000) should require manager approval before fulfillment. After deploying the catalog item, several requests were fulfilled without any approval being requested.

**Why It Happens:**
The approval condition on the catalog workflow was configured to check `requested_for.manager != null`. But several users in the system have no manager set on their user record — for these users, the condition evaluates to false and no approval is generated.

**Root Cause:**
Workflow approval condition is not handling null manager records gracefully — assumes all users have a manager.

**Fix / Prevention:**
- Update approval condition to handle null manager: if manager is null, escalate to the IT Director as fallback approver
- Add a data quality check: run a report of users with no manager set and resolve the gaps
- Add a Business Rule that prevents catalog submission for items above a cost threshold if the requester has no manager defined
- Test approval flows with user accounts that have no manager, as well as those with managers — edge cases are where workflows fail

---

## Issue 5: Business Rule Performance Issues — ITSM Forms Loading Slowly

**Category:** Performance / Scripting
**Severity:** Medium — agent productivity impact

**The Problem:**
The Incident form takes 8-12 seconds to load. Agents complain it is too slow, and some are reverting to email-based communication. The issue started after a new Business Rule was deployed.

**Why It Happens:**
A new Business Rule runs on every incident load (When = "onLoad") and performs a GlideRecord query without a limit — it loops through all incidents to calculate some aggregate, which is extremely slow.

**Root Cause:**
Unoptimized GlideRecord query in a Business Rule — no query limit, running on form load, iterating through thousands of records.

**Fix / Prevention:**
- Review any Business Rules with `When = Before` or `When = After` on the incident table that were recently added
- Use `GlideAggregate` instead of `GlideRecord` for count/sum operations
- Add `.setLimit()` to any GlideRecord query that does not need all records
- Move aggregate calculations to a Scheduled Job that runs periodically and stores results, rather than calculating on each form load
- Use the Transaction Log (sys_log) and ServiceNow Performance Analytics to identify slow transactions

---

## Issue 6: SLA Breach Notifications Not Sending

**Category:** Configuration / Notifications
**Severity:** Medium — teams unaware of SLA breaches until too late

**The Problem:**
The team configured SLA breach notifications to alert the assignment group when an incident reaches 80% of SLA elapsed time. Nobody is receiving notifications. SLA breaches are happening without warning.

**Why It Happens:**
Three common causes: (1) Notification is disabled, (2) The notification's condition (Send When) does not match the actual SLA task state changes, or (3) The notification references a field that is null for the affected records (e.g., `assignment_group.manager.email` but no group manager is set).

**Root Cause:**
Notification condition not aligned with SLA task state changes, or email recipient resolution failing due to null values.

**Fix / Prevention:**
- Check the Notification record: confirm it is Active, the Condition matches, and the Recipients are correctly defined
- Test the notification: create a test incident with a short SLA and manually verify the notification fires at 80% elapsed
- Add fallback recipients: if `assignment_group.manager` is null, fall back to a static distribution list email
- Review the notification log (sys_notification_log) for failed send attempts — errors are logged there

---

## Issue 7: Change Management — Unauthorized Changes Bypassing CAB

**Category:** Process / Configuration
**Severity:** High — changes deployed without approval, causing incidents

**The Problem:**
Several production incidents were traced to changes that were made directly in production without going through the Change Management process. The teams involved said they "forgot" to raise a change request because the change was "minor."

**Why It Happens:**
Change Management is a process, not just a technical control. If ServiceNow ITSM does not enforce change controls at the infrastructure/access level, determined teams can bypass the process entirely.

**Root Cause:**
No technical enforcement of the change process — teams can deploy changes without a corresponding approved Change Request in ServiceNow.

**Fix / Prevention:**
- Integrate Change Management with deployment tools: CI/CD pipelines should require an approved Change Request number before production deployment jobs will run
- Add ACLs to production configuration management systems: production access requires an active approved Change Request in ServiceNow
- Configure ServiceNow to track "unauthorized changes" — CMDB CI changes made without an associated Change Request are flagged automatically
- Training and culture: "minor" changes that cause incidents are still changes — P1 incidents traced to undocumented changes should have the responsible team's change compliance rate reported to leadership

---

## Issue 8: Knowledge Articles Not Appearing in Incident Search Results

**Category:** Configuration / Knowledge Management
**Severity:** Medium — self-service deflection not working, unnecessary ticket creation

**The Problem:**
Agents and employees report that when they search the incident form for related KB articles, the results do not include articles that are clearly relevant to the incident topic. The KB search seems to return random results.

**Why It Happens:**
Three common causes: (1) Knowledge Base is restricted to a user criteria that does not include the searcher, (2) The relevant articles are in a category that is not connected to the catalog item or incident form's KB search, (3) Articles have not been indexed since they were published.

**Root Cause:**
Knowledge Base user criteria or category association misconfigured, or search index is stale.

**Fix / Prevention:**
- Verify the Knowledge Base User Criteria: confirm the roles/groups of the searcher match the KB access rules
- Check KB Category to CI/Catalog Item mapping: ITSM incident forms should show articles from the relevant categories based on the incident's category field
- Trigger a Knowledge Base index rebuild if articles are not appearing after publication
- Test KB search as the actual user persona (not admin) — admin sees everything regardless of user criteria

---

## Issue 9: Incident State Not Updating When Resolved — Stuck in "In Progress"

**Category:** Scripting / Configuration
**Severity:** Medium — reporting inaccurate, SLA not closing

**The Problem:**
Agents report that incidents are not automatically moving to "Resolved" when they fill in the Resolution Code and Resolution Notes fields. The state stays at "In Progress" and agents have to manually change the state.

**Why It Happens:**
The Business Rule that should auto-set state to Resolved when Resolution Code is populated is either: not active, has a condition that is not being met, or was overridden by another Business Rule that sets state back to In Progress.

**Root Cause:**
Business Rule logic gap or rule ordering conflict.

**Fix / Prevention:**
- Review active Business Rules on the incident table filtering for rules that change the State field
- Check execution order (Order field) — ensure the "set Resolved when resolution code set" rule runs after any rules that might reset State
- Use the Business Rule debugging: enable `gs.log()` statements temporarily to confirm which rules are firing in what order
- Validate the Business Rule condition: `current.close_code.changesTo()` may not fire if the field is being set via a script rather than a user action

---

## Issue 10: Catalog Item Variables Not Saving — RITM Missing Required Data

**Category:** Configuration / Service Catalog
**Severity:** High — fulfillment team missing information needed to complete request

**The Problem:**
After a catalog item is submitted, the RITM record is missing several variable values. The fulfillment team cannot complete the request without calling the requester back for the missing information, creating delays and frustration.

**Why It Happens:**
Catalog item variables are not being persisted to the RITM record or are being cleared by a Script Include or Catalog Client Script that modifies variables unexpectedly. Another cause: variables set to "Insert/Update" visibility are not included in the form submission correctly.

**Root Cause:**
Variable persistence issue — either the variable's "Map to field" mapping is incorrect, or a script is clearing the value before submission.

**Fix / Prevention:**
- Audit each variable: confirm "Map to field" is configured correctly if the variable should populate a native RITM or Incident field
- Review all Catalog Client Scripts (onSubmit type) that run on the affected catalog item — confirm none are clearing required field values
- Test the catalog item submission end-to-end: submit as a test user, inspect the resulting RITM record fields and variable values, confirm all expected data is present
- For complex catalog items with many variables: add a confirmation summary page before final submission so the requester can verify their inputs
