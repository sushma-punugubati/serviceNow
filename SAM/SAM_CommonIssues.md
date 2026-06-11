# ServiceNow SAM Pro — Common Issues, Pitfalls & Troubleshooting

Issues encountered during SAM Pro implementations and ongoing support.

---

## Issue 1: Software Normalization Backlog — Thousands of Unrecognized Installations

**Category:** Configuration / Data Quality
**Severity:** Critical — without normalization, reconciliation is impossible

**The Problem:**
After enabling SAM Pro against existing CMDB data, the normalization dashboard shows 40,000+ unrecognized software installation records. The reconciliation schedule runs but produces meaningless results because most installations have no Software Model assigned.

**Why It Happens:**
Organizations accumulate years of software installation data from Discovery/SCCM without any normalization in place. Every version, language pack, hotfix update, and OEM variant creates a distinct software name string — the same product can appear hundreds of ways.

**Root Cause:**
No normalization rules exist for the existing software catalog. Publisher Packs have not been installed.

**Fix / Prevention:**
- Install Publisher Packs for the top 10 publishers by installation volume immediately — this typically handles 60-70% of installations
- For remaining unrecognized software: sort by installation count and address the top 50 by volume (which usually covers 80%+ of the remaining gap)
- Create a "normalization backlog" report filtered to unrecognized software with >10 installations — work through it systematically
- Schedule monthly reviews of newly unrecognized software to prevent backlog accumulation
- Do not try to normalize everything at once — prioritize by volume and risk (licensed software vs. freeware)

---

## Issue 2: Reconciliation Shows Wrong Counts — Over-Licensed When Actually Compliant

**Category:** Configuration / Reconciliation
**Severity:** High — inaccurate compliance position erodes trust in SAM data

**The Problem:**
Reconciliation reports 500 over-licensed Office seats. Finance paid for 2,000 licenses. But IT leadership knows they have 2,000 employees all using Office. The SAM dashboard is wrong, and no one trusts it.

**Why It Happens:**
Multiple root causes are common:
- Entitlement record quantity is wrong (entered manually and not validated)
- Normalization is incomplete — some Office installations are not mapping to the correct model
- Multiple Entitlement records exist for the same product (one per purchase order, not consolidated)
- License metric mismatch — entitlement says per-device, but reconciliation is counting per-user

**Root Cause:**
Entitlement data quality or normalization gaps causing installation count to not match entitlement scope.

**Fix / Prevention:**
- Validate entitlement quantities against the actual vendor agreement document
- Consolidate multiple entitlement records for the same product under a single record with the correct total quantity (or use the Entitlement Allocation feature to represent multiple POs under one total)
- Verify normalization coverage: check what percentage of total installations have a Software Model assigned — it should be above 95% for major products before trusting reconciliation
- Confirm the license metric on the entitlement matches the agreement terms exactly

---

## Issue 3: SaaS Integration Failing — Vendor API Authentication Errors

**Category:** Integration / SaaS License Management
**Severity:** High — SaaS usage data not flowing, compliance position incomplete

**The Problem:**
The Salesforce SaaS integration shows "Authentication Failed" errors in the IntegrationHub logs. Last successful data pull was 3 weeks ago. No one noticed until the monthly SAM review showed Salesforce seat data was stale.

**Why It Happens:**
SaaS vendor APIs use OAuth tokens or API keys that expire or get revoked. When a token expires, the integration fails silently unless monitoring is in place. Connected App credentials in Salesforce can also be revoked when a Salesforce admin refreshes security settings.

**Root Cause:**
OAuth token expiration or API key rotation without updating the ServiceNow credential.

**Fix / Prevention:**
- Set up monitoring on SaaS integration schedules: alert if a scheduled import has not run successfully in X days
- Use a dedicated service account for all SaaS API connections (not a named user's credentials) to prevent disruption when employees leave
- Document the token refresh process for each SaaS vendor and schedule a calendar reminder 2 weeks before OAuth token expiry
- Test all SaaS connections after every ServiceNow upgrade — spoke versions may change authentication handling

---

## Issue 4: Reclamation Triggering on Software That Is Still Needed

**Category:** Configuration / Reclamation
**Severity:** High — incorrect reclamation damages user trust and productivity

**The Problem:**
An employee receives a reclamation notice for Visio after not using it for 90 days. They ignore the notice (thinking it is spam). Their Visio is uninstalled. Two weeks later they start a project that requires Visio — they cannot work and are frustrated.

**Why It Happens:**
The reclamation policy is too aggressive: 90-day no-usage threshold is too short for tools used periodically for specific project types. Users do not realize that ignoring the reclamation notice means they will lose the software.

**Root Cause:**
Usage threshold too short for periodic-use software. Insufficient user communication about consequences.

**Fix / Prevention:**
- Segment the reclamation policy by software category: daily productivity tools (shorter threshold), project tools like Visio/AutoCAD/Photoshop (longer threshold, 180+ days)
- Make the consequence crystal clear in the notification: "If you do not respond by [date], this software will be uninstalled"
- Add a "I still need this" self-service button in the Employee Center that extends the reclamation hold for 6 months
- Before actual uninstall, send a final warning 3 days before the uninstall task executes

---

## Issue 5: Duplicate Software Models Creating Double-Count in Reconciliation

**Category:** Data Quality
**Severity:** Medium-High — compliance position inaccurate

**The Problem:**
SAM Pro shows Adobe Photoshop as both "compliant" and "under-licensed" at the same time. The compliance dashboard shows two separate rows for Photoshop with different counts.

**Why It Happens:**
Two Software Models exist for the same product — one created manually before the Publisher Pack was installed, and one from the Publisher Pack. Installations split between the two models, resulting in partial counts against each entitlement.

**Root Cause:**
Duplicate Software Model records with different normalization rules each capturing a subset of total installations.

**Fix / Prevention:**
- Before installing Publisher Packs, audit for manually created models that will be duplicated
- After Publisher Pack installation, run a duplicate model check and merge or deactivate manual models that are now covered by the pack
- Set the Publisher Pack model as authoritative: update normalization rules to point to the pack model, transfer any entitlements from the manual model to the pack model
- Add a governance check: before creating a new Software Model, search the Content Library first to confirm there is no existing model to use

---

## Issue 6: License Compliance Position Not Reflecting Recent Purchases

**Category:** Process / Data Currency
**Severity:** Medium — compliance position shows risk that no longer exists

**The Problem:**
SAM Pro shows the organization as under-licensed for SQL Server by 50 seats. IT purchased 100 additional SQL Server licenses 3 weeks ago. But the compliance dashboard still shows the same under-licensed position.

**Why It Happens:**
The new purchase has not been entered as an Entitlement record in SAM Pro. There is no integration between the procurement system and SAM Pro — entitlements are updated manually, and the manual update was missed.

**Root Cause:**
No formal process for creating Entitlement records when new licenses are purchased. SAM Pro is being updated reactively rather than proactively.

**Fix / Prevention:**
- Create a formal SAM intake process: every software purchase order triggers a workflow task to create or update the Entitlement record in SAM Pro before the purchase is considered complete
- Integrate the procurement system with SAM Pro via an import set or API — new PO lines automatically create draft Entitlement records for SAM team review
- Add a reconciliation check: when a new Entitlement is created, trigger an immediate reconciliation for the affected Software Model

---

## Issue 7: SCCM Data Not Covering All Devices — Blind Spots in Installation Data

**Category:** Discovery / Data Quality
**Severity:** High — unknown compliance exposure from undiscovered installations

**The Problem:**
SAM Pro shows 5,000 SQL Server installations. But the database team knows they have about 7,500 SQL Server instances across the environment. The 2,500 gap represents undiscovered installations — potential compliance exposure that SAM Pro cannot see.

**Why It Happens:**
SCCM agents are not installed on all servers. Cloud VMs, Linux servers, and servers in network segments that SCCM cannot reach are not covered. These servers' software is invisible to SAM Pro.

**Root Cause:**
Incomplete Discovery coverage — SCCM agent deployment gaps, network segment coverage gaps, or agent health issues on remote devices.

**Fix / Prevention:**
- Audit SCCM agent coverage: compare SCCM device count to CMDB server CI count — the gap represents blind spots
- For cloud infrastructure: deploy ServiceNow Discovery or AWS/Azure-native discovery tools that can scan cloud instances not covered by SCCM
- For Linux servers: ensure SCCM Linux agent is deployed or use ServiceNow Discovery with SSH-based probes
- Add a "Discovery coverage" metric to the SAM dashboard so blind spots are visible and tracked

---

## Issue 8: Publisher Pack Data Stale After ServiceNow Upgrade

**Category:** Maintenance / Platform Upgrade
**Severity:** Medium — normalization accuracy degrades over time

**The Problem:**
After a platform upgrade, several software products that were previously normalizing correctly are now showing as unrecognized. The Publisher Pack models appear to be out of date — they do not include normalization rules for the latest product versions.

**Why It Happens:**
Publisher Packs are versioned and need to be updated after ServiceNow platform upgrades. The upgrade process does not automatically update Publisher Pack content — it must be triggered manually.

**Root Cause:**
Publisher Pack version not updated post-upgrade.

**Fix / Prevention:**
- Add Publisher Pack update to the post-upgrade checklist for every ServiceNow release
- Subscribe to ServiceNow's Publisher Pack update notifications in the Content Library
- After each Publisher Pack update, run a normalization re-run to re-process any installations that were affected

---

## Issue 9: Multi-Region Compliance — Different Agreements in Different Countries

**Category:** Configuration / Complexity
**Severity:** High — entitlement scope mismatch causes false compliance readings

**The Problem:**
The organization has a global Microsoft EA covering the US and Canada, but separate agreements for Europe and Asia-Pacific. SAM Pro is treating all Microsoft installations globally against the EA entitlement, which shows as compliant for the US but masks real under-licensing in EMEA.

**Why It Happens:**
Entitlement records do not have geographic scope defined. The reconciliation engine sums all installations globally against all entitlements globally — regions with their own agreements get incorrectly pooled.

**Root Cause:**
Entitlement records missing Company or Location scope — reconciliation runs without geographic filtering.

**Fix / Prevention:**
- Define Entitlement records with explicit Company or Location scope
- Configure Reconciliation Schedules with geographic filters — run separate reconciliations per region for software with region-specific agreements
- Build regional SAM dashboards showing compliance position per region rather than one global view

---

## Issue 10: Reclamation Workflow Not Completing — Uninstall Tasks Stuck

**Category:** Integration / Workflow
**Severity:** Medium — licenses not actually recovered despite reclamation process running

**The Problem:**
The reclamation workflow creates uninstall ITSM tasks but the tasks sit in "Open" state for weeks without being actioned. Licenses show as "pending reclamation" indefinitely — no actual license recovery occurs.

**Why It Happens:**
Reclamation creates ITSM tasks and assigns them to a generic "Software Deployment" group, but the deployment team does not have a process for handling reclamation-sourced tasks differently from new software requests. They are de-prioritized and forgotten.

**Root Cause:**
No ownership or SLA defined for reclamation-sourced uninstall tasks.

**Fix / Prevention:**
- Create a dedicated ITSM assignment group for reclamation tasks with clear ownership
- Set a 5-business-day SLA on reclamation uninstall tasks — breach escalates to the SAM manager
- Add a SAM dashboard widget showing "Pending Reclamation Value" (estimated license value of tasks not yet completed) to create financial visibility into the backlog
- Automate uninstall for specific software categories using SCCM/Intune integration — for software that can be remotely uninstalled without user action, eliminate the manual task
