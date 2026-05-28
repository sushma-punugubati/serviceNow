# ServiceNow ESP — Common Issues, Pitfalls & Troubleshooting

Issues encountered during ESP and Service Catalog implementation, based on ServiceNow community posts, partner experience, and real-world implementations.

---

## Issue 1: Service Catalog Organized by IT Team, Not User Intent

**Category:** Design / User Experience  
**Severity:** Critical — low adoption if not fixed; the most common ESP failure

**The Problem:**
The Service Catalog goes live with categories like "Infrastructure Team Requests," "Application Support Requests," "Security Team Requests," "Network Team Requests." Users — especially non-IT staff — have no idea which team handles what. Adoption stays below 30% because the catalog is navigable to IT, not to the people who need to use it.

**Why It Happens:**
Catalog is built by IT teams who are naturally organizing around their own structure. Nobody ran user testing before go-live.

**Root Cause:**
Catalog UX must be designed from the user's perspective, not IT's organizational chart. Users don't think "I need the Infrastructure team" — they think "I need a new laptop."

**Fix / Prevention:**
- Redesign categories around user actions: "Get Equipment," "Set Up Access," "Get Software," "Fix a Problem"
- Rename all items in plain action-oriented language: "Get VPN Access" not "Remote Access Provisioning"
- Run a card sort exercise: give 20 employees the catalog items and ask them to group them logically
- Test with 10 real users before go-live: "Find me how to request X" — if they can't in 30 seconds, redesign

---

## Issue 2: Service Owners Not Assigned — Catalog Goes Stale

**Category:** Governance  
**Severity:** High — catalog degrades over time without ownership

**The Problem:**
Catalog items are built during implementation but nobody owns them afterward. After 12-18 months: item names reference old systems that were replaced, workflows route to teams that no longer exist, approval rules point to managers who left, and fulfillment times are wildly inaccurate. Users submit requests and wait weeks with no update.

**Why It Happens:**
"IT built the catalog" — but IT is a department, not an individual. No specific person is responsible for ensuring each catalog item stays current.

**Root Cause:**
Service Ownership must be established for every catalog item/service. Without owners, no one notices when items become broken.

**Fix / Prevention:**
- Every catalog item must have a designated Service Owner
- Quarterly catalog review: Service Owners confirm their items are accurate
- Automated reminders: IRM sends notification to Service Owner if their catalog item hasn't been reviewed in 90 days
- Metrics: catalog items with low CSAT scores escalated to Service Owner for review
- "Unpublished" state: items not reviewed in 180 days automatically unpublished

---

## Issue 3: Too Many Catalog Items — Users Can't Find Anything

**Category:** Design  
**Severity:** High — poor user experience drives email bypass

**The Problem:**
Over time the catalog accumulates: every IT team added items, old items were never removed, and slight variations were created instead of reusing existing items. A catalog with 600+ items where users can't find what they need in 30 seconds is as bad as having no catalog.

**Why It Happens:**
New items are easy to add; retiring items requires someone to decide they're no longer needed — a decision nobody makes proactively.

**Root Cause:**
Without catalog governance (who can add items? what's the retirement process?), catalogs grow without bound.

**Fix / Prevention:**
- Catalog governance policy: new items require approval from Catalog Manager
- Annual rationalization: review all items with < 5 requests in the last 12 months — retire or consolidate
- Merge duplicates: if 3 items do the same thing for slightly different audiences, one item with audience-specific variables is better
- Target catalog size: most organizations work best with 100-250 high-quality items, not 600 mediocre ones

**Community Reference:** ServiceNow Community — "Service Catalog cleanup and governance" — highly active thread

---

## Issue 4: CSDM Not in Place — Service Portfolio Has No Business Context

**Category:** Architecture / Integration  
**Severity:** High — ESP is disconnected from technical reality

**The Problem:**
Service Portfolio records are created for all IT services but they can't be linked to CMDB CIs because CSDM isn't properly implemented. Incident counts can't be shown per service. Cost attribution doesn't work. Impact analysis doesn't link from service to infrastructure. The Portfolio is a list of names with no context.

**Why It Happens:**
ESP was implemented without CMDB/CSDM prerequisites being in place. Service Portfolio and Service Catalog teams don't coordinate with the CMDB team.

**Root Cause:**
ESP requires CSDM to provide the Application Service → Technical Service → Infrastructure CI chain. Without it, Service Portfolio exists in isolation.

**Fix / Prevention:**
- CMDB/CSDM readiness is a prerequisite before ESP delivers full value — assess it at project start
- Start with highest-priority services: map the top 20 services to CSDM before expanding
- ServiceNow CSDM Workbook: complete it during design, not after go-live
- Joint workshop: ESP team + CMDB team + enterprise architects to align service model before building

---

## Issue 5: Fulfillment Workflows Break After ServiceNow Upgrades

**Category:** Technical / Configuration  
**Severity:** High — catalog items stop working after upgrades

**The Problem:**
After a ServiceNow version upgrade (e.g., Tokyo → Utah → Vancouver), several catalog fulfillment workflows break. Tasks don't route correctly, approval notifications don't send, and some workflows error silently. Users submit requests and they disappear.

**Why It Happens:**
Catalog fulfillment workflows were built using deprecated workflow features or direct table modifications that weren't upgrade-safe.

**Root Cause:**
ServiceNow evolves its workflow and Flow Designer capabilities each version. Workflows built in legacy Workflow Editor may not behave the same in Flow Designer (the modern approach).

**Fix / Prevention:**
- Migrate catalog workflows to **Flow Designer** — the modern, upgrade-stable approach
- In each upgrade: include a catalog smoke test — test top 20 most-used catalog items end-to-end in a sub-production instance before upgrading production
- Avoid direct customizations on base workflow activities — use Flow Designer actions instead
- Catalog item regression test suite: automated tests that can be run after each upgrade

---

## Issue 6: Variable Sets Not Used — Duplicate Variables on Every Item

**Category:** Configuration / Maintainability  
**Severity:** Medium — maintenance overhead, inconsistency

**The Problem:**
Every catalog item was built independently — each has its own version of "Manager Approval" variables, "Cost Center" variables, and "Business Justification" fields. When the approval process changes (new field required), 200 catalog items must be individually updated.

**Why It Happens:**
Catalog items were built by different people who weren't aware of shared components, or Variable Sets weren't part of the build standard.

**Root Cause:**
Without Variable Sets, there's no reuse. Every change to a shared data element must be made manually on every catalog item.

**Fix / Prevention:**
- Create standard Variable Sets at project start: "Approval Information," "Requester Details," "Financial Coding"
- Apply Variable Sets to all relevant catalog items — changes to the Variable Set flow to all items automatically
- Catalog build standard: document required Variable Sets for all new catalog items
- Audit: identify which catalog items have duplicate variables that could be consolidated into Variable Sets

---

## Issue 7: Service Retirement Never Happens — Legacy Services Run Forever

**Category:** Governance / Lifecycle  
**Severity:** Medium — cost and technical debt accumulation

**The Problem:**
The Service Portfolio has a "Retired" lifecycle stage, but services never reach it. New services are deployed but old ones continue running alongside. IT is maintaining both the old VPN and the new Zero Trust solution. The old HR system and new HRSD both accept requests. Confusion, duplication, and cost.

**Why It Happens:**
Nobody owns the retirement decision. Retiring a service requires courage — users complain, edge cases emerge, and it's easier to just leave the old service running.

**Root Cause:**
Without a formal retirement workflow with deadlines and accountability, retirement is always deferred.

**Fix / Prevention:**
- Retirement workflow: a service moved to "Retiring" must have: retirement date, user migration plan, data archival plan, and sign-off from Service Owner
- Sunset communication: automated notifications to users of retiring services with cutover date
- Hard decommission: catalog items for retiring services are auto-unpublished on retirement date
- IT management accountability: "Services in 'Retiring' state past retirement date" is a dashboard metric reviewed monthly

---

## Issue 8: ESM Expansion Fails — Non-IT Departments Resist Adoption

**Category:** Adoption / Change Management  
**Severity:** High — ESM ROI not realized

**The Problem:**
IT deploys the Employee Service Center intending for HR, Finance, and Legal to migrate their services onto it. HR leadership declines — "our employees will be confused by an IT portal." Finance agrees to participate but doesn't assign anyone to maintain their catalog items. Legal says they'll join "in Phase 2" — which never comes.

**Why It Happens:**
Non-IT departments don't feel ownership of a platform that was positioned as an "IT system." Without a business sponsor who owns the ESM vision, IT can't force adoption.

**Root Cause:**
ESM requires executive sponsorship at a level ABOVE IT — typically a COO or CHRO who mandates the cross-department portal strategy.

**Fix / Prevention:**
- Executive sponsor: ESM must be championed by business leadership, not IT leadership
- Quick win first: demonstrate value with one non-IT department (often HR works best) — proof of concept before broad rollout
- Department-specific branding: the portal should feel like "Our Employee Center," not "The IT Portal"
- Dedicated support: assign a Catalog Manager resource to help non-IT departments build their catalog items

---

## Issue 9: Record Producers Misconfigured — Wrong Records Created

**Category:** Technical  
**Severity:** Medium — data quality issues

**The Problem:**
Record Producers in the catalog are meant to create Security Incidents (SIR) when users report suspicious activity. But the Record Producer is creating regular ITSM Incidents instead. Security team never sees the reports. Or a Record Producer meant to create a Change Request is populating mandatory fields incorrectly, leaving the created record in an error state.

**Why It Happens:**
Record Producers require correct mapping of catalog form variables to target table fields. If the target table's mandatory fields aren't mapped, or the target table name is wrong, the record is created incorrectly or fails silently.

**Root Cause:**
Record Producer configuration requires exact field mapping to the target table. Errors in mapping go unnoticed until users report the issue.

**Fix / Prevention:**
- Test every Record Producer before go-live: submit a test request and verify the correct record is created with all fields populated
- Include Record Producers in catalog regression tests after upgrades
- Add a confirmation variable: "Your report has been submitted as a Security Incident [link]" — user can verify
- Validate target table: confirm the target table name is correct; `sn_si_incident` not `incident`

---

## Issue 10: Multi-Department Catalog Has Inconsistent Fulfillment SLAs

**Category:** Process / Governance  
**Severity:** Medium — user confusion and dissatisfaction

**The Problem:**
In the unified Employee Service Center, an IT laptop request has a clearly stated 2-day SLA. An HR policy question has no SLA displayed. A Finance expense report query says "5-10 business days" while the actual process takes 24 hours. Users don't trust the SLAs because they're inconsistent and often wrong.

**Why It Happens:**
Different departments were onboarded at different times by different people with no SLA standards. Some departments don't know their actual fulfillment times.

**Root Cause:**
SLA governance requires both a standards framework (what SLA tiers exist) and accurate data per department. Neither is automatically available when ESM is implemented.

**Fix / Prevention:**
- SLA taxonomy: define standard tiers that ALL departments use (e.g., Urgent=4h, Standard=2 days, Non-urgent=5 days)
- Measure before publishing: analyze actual fulfillment times from ITSM tickets before setting SLA targets
- Regular review: quarterly review of SLA compliance per catalog item — if consistently missing, SLA needs adjustment or process needs improvement
- Display actual performance: catalog pages can show "typical fulfillment time: 1.5 days" based on historical data — more accurate than arbitrary SLA values

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Catalog adoption below 40% | Poor UX; organized by IT structure | Redesign categories by user intent; rename items |
| Catalog items never updated | No Service Owners assigned | Assign owners; quarterly review process |
| Service Portfolio disconnected from CMDB | CSDM not implemented | CSDM assessment; link top services to Application Services |
| Workflows break after upgrade | Legacy Workflow Editor usage | Migrate to Flow Designer |
| Legacy services never retired | No retirement governance | Add retirement workflow with deadlines and accountability |
| ESM departments not adopting | No executive mandate; IT-owned branding | Escalate to COO/CHRO; rebrand as Employee Center |
| Record Producers creating wrong records | Field mapping errors | Test every Record Producer; validate target table |

---

## Sources and References

- ServiceNow Community: community.servicenow.com — Service Catalog, Service Portal, CSDM forums
- ServiceNow Docs: Service Catalog Implementation Guide, ESM Playbook
- ServiceNow Knowledge Conference: "Service Catalog Best Practices", "ESM Deployment" sessions
- ServiceNow Now Create methodology for Service Catalog
- LinkedIn: ServiceNow practitioners sharing "Service Catalog lessons learned" posts
