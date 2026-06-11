# ServiceNow ITSM — Interview Notes
## "What did you do in ITSM?" + Challenges Faced

---

## 1. OVERVIEW — What is ITSM?

ITSM is ServiceNow's foundational module and the area where I have the deepest, most consistent experience — it has been the core of every role I have held on the platform over 8+ years. At Dell, ITSM is the primary operational platform for the entire IT organization: Incident, Problem, Change, Service Catalog, and Request management run through ServiceNow.

ITSM at Dell is not a simple implementation — it is a large, complex, high-volume platform with weekly production releases, multi-level catalog approval workflows, Domain Separation configuration, LDAP integration, and custom scripting for automation across dozens of business processes.

Key areas I work on daily:
- Incident, Problem, Change, and Service Catalog configuration and support
- Business Rules, Client Scripts, UI Policies, UI Actions, ACLs
- SLA definitions and lifecycle management
- Flow Designer automation for approval and fulfillment workflows
- Agent Workspace and Playbooks for major incident response
- Performance Analytics dashboards for ITSM leadership
- LDAP integration and Domain Separation configuration
- Weekly release management: update sets, GitLab CI/CD, DEV/TEST/UAT/PROD promotion

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Incident Management Configuration

- Managed the full **Incident lifecycle** at Dell — from intake configuration (category/subcategory taxonomy, assignment rules, auto-assignment via LDAP group membership) through resolution and SLA tracking
- Configured the **Priority matrix** Business Rule: Impact × Urgency → Priority calculated automatically on every incident save, with overrides requiring ITIL Manager role (prevents analysts from manually lowering priority to game metrics)
- Set up **Major Incident process** with dedicated Workspace: when an incident is flagged P1, the Major Incident Workspace activates a guided playbook that walks coordinators through bridge setup, stakeholder notification cadence, and escalation steps
- Configured **inbound email processing** for the support mailbox — email parsing rules that route to the correct assignment group based on the TO address, subject keywords, and sender domain. Built deduplication logic to prevent reply emails from creating new incidents
- Maintained the **SLA definitions** for all priority levels — business-hours-aware SLAs with pause conditions for On Hold states and custom wait reasons. Fixed several legacy SLA configurations that were running on 24/7 clock when they should have been business-hours-only

### 2.2 Service Catalog Build-Out

- Built and maintained the **Service Catalog** at Dell — the catalog grew from 45 to 120+ items over my time here. I own the catalog architecture: item categorization, variable design, approval routing, and execution plan structure
- Designed complex **multi-level approval workflows** using Flow Designer: hardware requests above $3,000 went to Requester's manager → IT Director → Finance → Procurement in sequence, with each stage having a defined SLA and auto-escalation if not actioned
- Built **Execution Plans** for complex service items (New Employee Setup, Contractor Onboarding) that spawn parallel tasks to IT, Facilities, HR, and Security simultaneously — the RITM auto-closes when all tasks in all execution plan stages are complete
- Configured **Record Producers** for self-service incident submission, change request creation, and HR request routing — giving employees a structured intake form that creates the right type of record in the right module without the service desk as intermediary

### 2.3 Scripting and Automation

- Write and maintain **Business Rules** across the incident, change, problem, request, and catalog tables — field calculations, auto-assignment, state validations, cross-record updates, and notification triggers
- Build **Client Scripts** (onChange, onLoad, onSubmit) for the Incident, Change, and Catalog forms — field show/hide based on category selection, mandatory field enforcement, and dynamic field population from referenced records
- Use **UI Policies** for declarative (no-script) field behavior: show/hide and mandatory rules that do not require scripting
- Write **Script Includes** for reusable server-side logic — utility functions for LDAP group lookups, SLA calculations, and assignment group resolution that are called from multiple Business Rules and Flow Designer actions
- Maintain **ACLs** across the ITSM tables: protecting sensitive incident fields (caller's HR data, security incident details), controlling who can change Priority and State, and managing read access for external contractor groups

### 2.4 Flow Designer Automation

- **Flow Designer** is my primary automation tool for ITSM — I have moved all new automation to flows from the legacy Workflow Editor and have been progressively migrating older workflows as they come up for maintenance
- Key flows I have built: multi-stage catalog approval chains, automatic Problem creation from recurring incidents, Change request lifecycle (submit → CAB review → implementation → PIR), and Major Incident stakeholder notification
- Built **reusable subflows** for common patterns: "send SLA warning notification," "create CMDB relationship," "get manager approval" — these subflows are called from multiple parent flows rather than duplicating logic
- Use **IntegrationHub spokes** within flows to connect ITSM processes to external systems: notifications via Microsoft Teams, ticket creation in Jira for development defects, asset updates in the procurement system

### 2.5 Release Management

- At Dell, we do **weekly production releases** — I manage the update set process and GitLab CI/CD pipeline for ITSM deployments
- My release process: code review in DEV → update set capture → automated regression tests (ATF) in TEST → UAT sign-off → production release on Wednesday maintenance window
- Manage **ATF test suites** for the ITSM module: test cases cover the critical paths (incident creation → assignment → resolution, catalog item submission → approval → fulfillment) and run automatically on every TEST deployment
- Handle **upgrade regression testing**: when ServiceNow upgrades the platform, I coordinate regression testing across the ITSM modules — reviewing skipped records, testing customizations against the new version, and coordinating with application teams before promoting to production

### 2.6 Performance Analytics Dashboards

- Built and maintain **PA dashboards** for the ITSM management team at Dell — they review these every Monday morning:
  - SLA compliance rate by priority (first response and resolution)
  - Incident volume by assignment group and trend week-over-week
  - Change success rate (normal changes implemented without incident)
  - Catalog request fulfillment time by catalog item
  - Top 10 incident categories by volume (drives knowledge article priorities)
- Configured **PA indicators and data collection jobs** — indicators run nightly to collect ITSM metrics, breakdowns allow filtering by team, category, and time period

---

## 3. CHALLENGES FACED

### Challenge 1: SLA Breach Spike After State Model Change

**Problem:** The ITSM team added a new incident state "Pending Vendor" to track incidents waiting on third-party vendors. Within 2 weeks, SLA breach rates jumped from 8% to 31%. The team assumed the new state was causing incidents to breach because the clock was still running while waiting for the vendor.

**Solution:** Audited the SLA definitions and confirmed that the pause conditions only included "On Hold" (state 3) and not the new "Pending Vendor" state (which had been assigned state value 7). Updated all SLA pause conditions to include state 7. But the existing incidents that had been in "Pending Vendor" state for days could not benefit from retroactive pause without a manual fix. Wrote a script to retroactively recalculate SLA elapsed time for incidents in "Pending Vendor" state, crediting the pause time back. SLA breach rate returned to 9% after the fix.

---

### Challenge 2: Catalog Approval Workflow Not Reaching Finance for High-Value Requests

**Problem:** Hardware requests above $3,000 were supposed to get Finance approval as the final stage. The Flow Designer flow was configured correctly, but Finance was never receiving approval tasks. Requests were being approved without Finance sign-off.

**Solution:** Investigated the flow execution history for a test request. Found that the Finance approval stage had a condition: `requested_for.department.finance_code != null`. The department table at Dell does not have a "finance_code" field — this condition was always evaluating to false, so the Finance stage was being skipped entirely. The condition had been written by another developer referencing a field that did not exist. Fixed the condition to use the correct field (cost_center.budget_code) and tested against multiple requesters to confirm the Finance stage now activates for requests above the threshold.

---

### Challenge 3: Update Set Conflict During Production Release

**Problem:** During a Wednesday production release, the update set preview showed a conflict: a Business Rule I had modified in DEV had already been modified by another developer (for a different change) in the production instance — probably an emergency fix that bypassed the normal DEV process. Committing my update set would have overwritten the emergency fix.

**Solution:** Paused the release. Compared the two versions of the Business Rule: my version and the PROD version. Merged the logic from both into a combined version that included both the emergency fix and my new functionality. Created a new update set in DEV containing only the merged Business Rule, promoted it through TEST and UAT on an accelerated timeline (same-day testing), and committed it to production that evening. Added a policy after this incident: emergency fixes must be immediately back-ported to DEV by the developer who made them, and a Slack notification sent to the release manager so conflicts can be identified before the next release cycle.

---

### Challenge 4: LDAP Integration Breaking Incident Assignment After Directory Restructure

**Problem:** The organization restructured several IT departments and renamed 40+ Active Directory groups. After the restructure, incidents in those categories were no longer routing to the correct assignment groups — they were falling to a default "IT Service Desk" group. Hundreds of incidents per day were misrouted.

**Solution:** The LDAP integration imported the renamed AD groups as new ServiceNow groups rather than updating the existing ones — so old ServiceNow groups (with the old AD group name) still existed but were no longer syncing with any AD group. The assignment rules were pointing to the old group names. Fix: updated the LDAP integration transform map to use the group's object GUID as the matching key rather than the group name (GUIDs do not change when a group is renamed). Then ran a sync to re-link the existing ServiceNow groups to their corresponding AD groups by GUID. Rewrote the assignment rules to use the ServiceNow group sys_ids (permanent identifiers) rather than group name strings.

---

### Challenge 5: Performance Analytics Dashboard Numbers Not Matching Manual Reports

**Problem:** The ITSM VP ran a manual report (using a Report Designer list report) showing 94% SLA compliance for Q2. The PA dashboard showed 87% for the same period. Leadership was confused about which number was correct and the credibility of both the dashboard and the manual reports was in question.

**Solution:** Investigated the data source difference. The manual report was filtering on `resolved_at` (when the incident was resolved) and counting incidents. The PA indicator was counting based on the `task_sla.has_breached` field, which captured whether any SLA on the incident had breached — including response SLAs that breached even when resolution SLAs were met. The two metrics were measuring different things: "% of incidents resolved within SLA" vs. "% of incidents with no SLA breach at all." Both numbers were technically correct — they just measured different things. Worked with the VP to define a single agreed definition: "% of incidents where the Resolution SLA was met." Updated both the PA indicator and the manual report parameters to use the same definition, and the numbers aligned.

---

### Challenge 6: ATF Tests Passing in TEST But Failing in UAT

**Problem:** The ATF regression suite ran clean in TEST (all 47 tests passed). After promoting the update set to UAT, 11 tests failed. The release could not proceed. This was holding up a time-sensitive release.

**Solution:** Compared the TEST and UAT environments. Found that UAT had data differences: several test users used by the ATF suite existed in TEST but not in UAT (they had been created in TEST directly, not via the UAT data seeding process). Three tests failed because their test data setup scripts referenced user sys_ids that did not exist in UAT. Fixed by updating the ATF test data setup scripts to create required test users if they do not exist, rather than assuming they are present. Also updated the environment preparation runbook to include test user seeding as a required step before running ATF in any new environment.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **Incident** | Unplanned interruption — goal is to restore service fast, not necessarily find root cause |
| **Problem** | Root cause investigation — becomes a Known Error when cause is identified |
| **Change** | Standard (pre-approved) / Normal (CAB) / Emergency (ECAB) |
| **RITM** | Request Item — one per catalog item in a request, child of REQ |
| **SLA Pause Condition** | Stops the SLA clock while waiting for external input — must be configured for all wait states |
| **Business Rule** | Server-side script on table — runs before/after DB operations |
| **Client Script** | Browser-side JavaScript — runs on form interactions (onChange, onLoad, onSubmit) |
| **Update Set** | Container that captures config changes for cross-environment promotion |
| **ATF** | Automated Test Framework — regression test suite, should run on every deployment |
| **CAB** | Change Advisory Board — reviews and approves Normal Changes |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Managed weekly ITSM production releases for **4+ years** without a release-caused P1 incident
- Service Catalog grew from **45 to 120+ items** with complex multi-stage approval and execution plan workflows
- SLA breach investigation and fix: restored compliance from **31% breach rate to 9%** after state model change
- ATF regression coverage: **47 test cases** covering all critical ITSM paths, integrated into CI/CD pipeline
- PA dashboards used by ITSM leadership for **weekly management review** — replaced manual report assembly
- LDAP integration maintained across **40+ AD group renames** without sustained routing disruption
- Performance Analytics: SLA compliance tracking enables data-driven conversations with business stakeholders vs. anecdotal assessment
