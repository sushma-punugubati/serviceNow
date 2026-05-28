# ServiceNow BCM — Common Issues, Pitfalls & Troubleshooting

Issues encountered during BCM implementation and ongoing support, based on ServiceNow community posts, partner experience, and real-world implementations.

---

## Issue 1: BCM Plans Exist But Were Never Tested — They Don't Actually Work

**Category:** Process / Testing  
**Severity:** Critical — the most dangerous BCM failure mode

**The Problem:**
The organization has BCPs and DRPs — they're documented, stored in ServiceNow, and referenced in regulatory reports. But they've never been tested. When an actual crisis occurs, plans reference systems that were decommissioned 2 years ago, staff members who left 18 months ago, and procedures that don't match current technology.

**Why It Happens:**
BCM testing requires significant time commitment — full simulations take a full day or more. Operational pressure causes testing to be deferred: "We'll do it after this project." It gets deferred indefinitely.

**Root Cause:**
An untested plan is a hypothesis. Only testing reveals whether recovery is actually achievable within RTO targets. Without testing, the BCM program is compliance theater.

**Fix / Prevention:**
- Mandatory testing schedule: configure BCM Exercise records with annual full simulation and quarterly tabletop — escalation if not completed on schedule
- "Paper" compliance is not enough: regulators (DORA, FFIEC) require evidence of actual testing, not just plan documentation
- Start with tabletops: 2-hour discussion exercises for each critical function — low cost, high discovery value
- Post-test: every finding becomes a remediation task in ServiceNow — tracked to closure before next test

---

## Issue 2: BIA Not Updated After Infrastructure Changes — RTO Targets Are Wrong

**Category:** Data Quality / Process  
**Severity:** High — plan is built on outdated assumptions

**The Problem:**
A Business Impact Analysis was completed 18 months ago. Since then: a major ERP migration happened, 3 new cloud services were deployed, and a legacy data center was decommissioned. The BIA still references the old data center and doesn't account for the new cloud dependencies. The stated RTO of 4 hours for the ERP system is now based on an architecture that no longer exists.

**Why It Happens:**
BIA is treated as a one-time project deliverable, not a living document. Nobody triggers a BIA review when infrastructure changes.

**Root Cause:**
BIA depends on accurate dependency data from CMDB. When CMDB isn't maintained OR when BIA isn't linked to CMDB, infrastructure changes don't flow into the BIA.

**Fix / Prevention:**
- Link BIA critical functions to CMDB Application Services — when CMDB changes, BCM is prompted to review
- Change Management integration: Change Requests touching Tier 1 critical system CIs automatically trigger a BCM impact review task
- Annual BIA refresh: mandatory annual review process for all BIA records in ServiceNow
- Cloud migration projects: include "BIA update" as a workstream in every major infrastructure project

---

## Issue 3: Recovery Time Objectives Set Unrealistically — Gaps Never Identified

**Category:** Configuration / Process  
**Severity:** High — false confidence in recovery capability

**The Problem:**
During BIA workshops, business stakeholders set RTO targets based on what they WANT (e.g., "We want the payment system back in 30 minutes") rather than what's technically achievable. IT knows it takes 4 hours to restore the payment system, but doesn't push back. The BIA records 30 minutes as the RTO. Plans are never tested. The gap between aspirational and achievable RTO is never visible.

**Why It Happens:**
Business stakeholders set RTOs from a business impact perspective without input from IT on feasibility. IT is often not in the BIA workshop.

**Root Cause:**
RTO setting requires both business input (what's the impact of X hours downtime?) AND technical input (what's the minimum achievable recovery time with current infrastructure?). Without both, RTOs are wishes, not plans.

**Fix / Prevention:**
- Joint RTO workshop: Business stakeholder AND IT architect / DR team together
- Technical feasibility check: for every RTO set, IT must confirm: "Can we actually achieve this? What investment is required if not?"
- CMDB validation: ServiceNow can compare documented recovery procedures against RTO — flag impossible RTOs
- Investment case: if RTO = 30 minutes but current capability = 4 hours → the gap is a funded project, not an ignored discrepancy

---

## Issue 4: BCM Plans Not Maintained — Outdated Contact Lists and Procedures

**Category:** Maintenance / Governance  
**Severity:** High — plans fail at invocation due to wrong information

**The Problem:**
BCM plans list the Crisis Management Team as 5 people. Three of them have left the company. The DRP for the trading system lists an IT contact who now manages a different system. The alternate work location contract was with a company that closed. Nobody updated the plans.

**Why It Happens:**
Plans are created during an implementation project with clear ownership. Post-go-live, nobody has a maintenance mandate. Staff turnover, organizational changes, and vendor changes happen constantly — plans silently become wrong.

**Root Cause:**
BCM plans require continuous maintenance. Without a defined maintenance process with accountability, plans degrade at the rate of organizational change.

**Fix / Prevention:**
- Annual plan review: every BCM plan reviewed annually — Service Owner confirms accuracy or updates
- HR integration: when an employee named in a BCM plan departs, automatically trigger a plan update task
- Vendor contract tracking: contracts referenced in BCMs (alternate sites, recovery vendors) tracked with expiry alerts
- Quarterly mini-review: contact lists verified every quarter (5-minute task per plan)

---

## Issue 5: CMDB Not Linked to BCM — Technical Dependencies Unknown

**Category:** Integration  
**Severity:** High — plans describe what to recover but not the full dependency chain

**The Problem:**
BCM plans document that "the Online Banking Application must be recovered within 2 hours." But the plan doesn't mention that Online Banking depends on a middleware ESB, which depends on an Oracle database cluster, which runs on a VMware cluster on 3 physical hosts in the data center. During a real DR event, the team restores the application servers but forgets the ESB — and Online Banking fails anyway.

**Why It Happens:**
BCM plans are written by business analysts who don't know the technical dependency chain. IT knows the infrastructure but doesn't maintain the BCM documentation.

**Root Cause:**
CMDB dependency maps contain the full technical picture. Without linking BCM critical functions to CMDB service maps, plans miss dependencies.

**Fix / Prevention:**
- Link every BCM Critical Business Function to the CMDB Application Service CI
- CMDB Service Map review: during BIA, walk through the full CMDB dependency map for each critical function
- DRP should mirror CMDB dependency sequence: restore in the order CMDB shows as the dependency chain
- Automate gap detection: if CMDB shows infrastructure recovery time > documented RTO → generate a risk record

---

## Issue 6: Exercise Findings Not Remediated Before Next Test

**Category:** Process  
**Severity:** High — exercises provide no improvement if findings aren't acted on

**The Problem:**
Annual tabletop exercise is completed. 15 findings are documented. Next year's tabletop happens — and 12 of the same 15 findings appear again. The exercise has become an annual ritual of documenting the same problems without fixing them. Regulators ask for evidence of "continuous improvement" — there is none.

**Why It Happens:**
Exercise findings create remediation tasks, but no one enforces completion before the next exercise. Finding owners deprioritize BCM remediation in favor of operational work.

**Root Cause:**
Exercise findings must have deadlines, owners, and management escalation — like any other compliance finding. Without enforcement, they're suggestions.

**Fix / Prevention:**
- Mandatory closure: critical exercise findings must be closed before the next exercise of the same plan
- Management escalation: findings open 30+ days past due date escalate to the BCM Sponsor
- IRM integration: exercise findings linked to IRM Risk Register — findings are treated as risks, not optional improvements
- Exercise dashboard: "% of prior exercise findings closed" is a BCM health metric reviewed quarterly

---

## Issue 7: BCM Scope Limited to IT — Business Process Continuity Ignored

**Category:** Scope  
**Severity:** High — BCM plans don't cover full business impact

**The Problem:**
BCM plans focus entirely on IT system recovery. But when a data center fails:
- IT restores all systems in 3 hours (within RTO)
- But the call center agents don't know the workaround procedures to handle customers DURING the 3 hours
- Customer-facing staff can't access the manual order entry process
- Nobody knows which customers need to be proactively called

The IT DRP was executed perfectly, but the business still failed customers.

**Why It Happens:**
IT owns BCM, IT writes the plans, IT tests the plans. Business process continuity (what employees DO during downtime) is never considered.

**Root Cause:**
BCM scope must cover BUSINESS processes, not just IT recovery. Manual workarounds, staff training, and customer communication are equally important.

**Fix / Prevention:**
- BCP scope: every BCP must include: manual workaround procedures, staff action checklists, customer communication templates
- Business participation: BCP writing requires the business function owner, not just IT
- Training: staff must practice manual procedures during tabletop exercises — not just IT staff
- Combined exercise: full simulation tests BOTH IT recovery AND business continuity procedures simultaneously

---

## Issue 8: Single BCM Manager — Program Collapses When They Leave

**Category:** Organizational Risk / Governance  
**Severity:** High — BCM program becomes single-person dependent

**The Problem:**
One highly capable person built the entire BCM program, maintains all the plans, runs all the exercises, and is the only one who understands the ServiceNow BCM configuration. When they resign or go on extended leave, the BCM program stops functioning. Plans aren't updated, exercises aren't scheduled, and regulators notice.

**Why It Happens:**
BCM is often perceived as a niche expertise. Organizations rely on one expert rather than building program resilience.

**Root Cause:**
A BCM program that depends on one person isn't resilient — which is ironic for a program designed to build resilience.

**Fix / Prevention:**
- Cross-train: at minimum 2 people who understand ServiceNow BCM configuration and can maintain plans
- Document program processes: how to schedule exercises, how to update plans, how to produce regulatory evidence — documented in ServiceNow Knowledge
- Succession planning: BCM Manager role explicitly included in organizational continuity planning
- ServiceNow access: ensure BCM responsibilities are assigned by ROLE, not by individual username

---

## Issue 9: SecOps and BCM Not Integrated — Cyber Incidents Don't Trigger BCP

**Category:** Integration  
**Severity:** Medium-High — BCM is bypassed during the most likely modern crisis

**The Problem:**
The most common crisis trigger in 2024-2026 is a cyberattack. But when a ransomware event occurs, the security team works the incident in SecOps (SIR) and the BCM plans are never activated. Business units don't invoke manual workarounds. Leadership isn't informed via the Crisis Management process. The BCM program exists for disasters but isn't connected to the threats that are actually occurring.

**Why It Happens:**
BCM was built for physical disasters (data center fire, natural disaster). Cyber threats were added as a scenario later but the integration between SecOps and BCM wasn't built.

**Root Cause:**
BCM invocation workflows should be triggered by security incident severity, not just physical events.

**Fix / Prevention:**
- ServiceNow integration: when SIR Security Incident reaches P1/Critical severity, trigger a BCM activation workflow
- Cyber-specific scenarios: add ransomware, DDoS, and data breach as standard BCM scenarios in exercise calendar
- SecOps playbooks should include: "If containment will take > X hours, invoke BCP for affected systems"
- Test the integration: include a cyber scenario in the annual full simulation

---

## Issue 10: Regulatory Evidence Not Producible on Short Notice

**Category:** Compliance / Reporting  
**Severity:** Medium-High — regulatory exam risk

**The Problem:**
A regulator announces an examination and requests BCM evidence: "Show us your critical function list, last 2 years of testing records, finding remediation status, and evidence of board-level oversight." The BCM team spends 2 weeks hunting through email archives, SharePoint, and an old ticketing system to piece together the evidence.

**Why It Happens:**
BCM activities were performed but not consistently documented in ServiceNow. Testing was done via meetings and notes; findings tracked in Excel; board reporting done via PowerPoint.

**Root Cause:**
Regulatory evidence must be captured in the system, not reconstructed after the fact. If it's not in ServiceNow, it's not producible quickly.

**Fix / Prevention:**
- Everything in ServiceNow: all exercises, findings, remediation, and approvals logged in BCM — not email or spreadsheets
- Board sign-off: annual BCM program review includes a ServiceNow record with executive approvals
- Regulatory evidence package: build a saved report in ServiceNow that produces the regulatory evidence package on demand
- Test the evidence package: annually, produce the full evidence package and review it — before the regulator asks

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Plans outdated — wrong contacts and systems | No maintenance process | Annual review + HR integration for departing employees |
| RTO targets never achieved in testing | Unrealistic RTOs set without IT input | Joint business + IT RTO workshop; technical feasibility check |
| Exercise findings repeat year after year | No enforcement of finding remediation | Mandatory closure policy; IRM integration for tracking |
| BCM doesn't cover business process workarounds | IT-only scope | Add business process procedures to every BCP |
| Cyber incidents don't trigger BCM | SecOps/BCM not integrated | Build SIR → BCM activation workflow for P1 incidents |
| Regulatory evidence takes weeks to produce | Activities not documented in ServiceNow | Capture all BCM activities in ServiceNow from day one |

---

## Sources and References

- ServiceNow Community: community.servicenow.com — IRM, Operational Resilience, BCM forums
- ServiceNow Docs: BCM Implementation Guide, Operational Resilience Configuration
- ISO 22301: International Standard for BCM Management Systems
- DORA Technical Standards (EU financial services)
- FFIEC Business Continuity Planning Handbook
- ServiceNow Knowledge Conference: "Operational Resilience in Practice" sessions
- LinkedIn: BCM practitioners and ServiceNow IRM experts sharing implementation lessons
