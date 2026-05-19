# ServiceNow ITAM — Common Issues, Pitfalls & Troubleshooting

Issues encountered during ITAM implementation and ongoing support, based on ServiceNow community posts, partner experience, and real-world implementations.

---

## Issue 1: Software Normalization Not Set Up — Compliance Calculations Are Wrong

**Category:** Configuration / SAM  
**Severity:** Critical — makes compliance results meaningless

**The Problem:**
Discovery finds software installations and creates Software Installation records — but the same product appears as 47 different records: "Microsoft Office 365," "MS Office 365," "Office 365 E3," "Microsoft 365 Apps for Enterprise." SAM compliance calculations treat each as a different product, requiring separate entitlements for each variation. Compliance position is wildly inaccurate.

**Why It Happens:**
Software Normalization is not automatic — it requires configuration of normalization rules or enabling Publisher Packs. Many implementations skip this step and go live with raw Discovery data.

**Root Cause:**
Without normalization, the same product installed on 10,000 machines might create 50 different "product" records in SAM. There's no single compliance view.

**Fix / Prevention:**
- Enable Publisher Packs for your top 10 software vendors before go-live
- Configure the Software Library to recognize and normalize product name variants
- Run a normalization pass before calculating any compliance positions
- Establish a normalization review process: monthly review of "unrecognized software" list

**Community Reference:** ServiceNow Community — "Software Normalization best practices" — most viewed ITAM thread

---

## Issue 2: Discovery Creating Duplicate Hardware Asset Records

**Category:** Configuration / HAM  
**Severity:** High — corrupts asset inventory

**The Problem:**
Every time Discovery runs, it creates new Hardware Asset records instead of updating existing ones. After 3 months, a 5,000-device environment has 15,000 asset records. Asset counts are triple actual, financial reconciliation is impossible, and IT staff lose trust in the system.

**Why It Happens:**
Hardware Identification Rules are misconfigured. Discovery uses hostname or IP address as the identifier — but if a device gets a new IP (DHCP) or hostname changes, Discovery can't find the existing record and creates a duplicate.

**Root Cause:**
Volatile identifiers (IP, hostname) should not be used as primary identifiers for hardware. More stable identifiers (serial number, MAC address) should be the matching criteria.

**Fix / Prevention:**
- Review Identification Rules for Hardware CI class: use Serial Number + MAC Address as primary identifiers
- Run ITAM Deduplication tool to merge existing duplicates
- Set Discovery schedule to update existing records, not create new ones
- Monitor: after each Discovery run, check for newly created vs. updated asset records

---

## Issue 3: Asset Lifecycle Stages Not Followed — Inaccurate Inventory States

**Category:** Process / Adoption  
**Severity:** Medium — inventory reflects wrong status

**The Problem:**
Assets are received and deployed but the asset state isn't updated (remains "In Stock" in ServiceNow when actually deployed to a user). Or assets are retired from service but remain as "In Use" because nobody processed the retirement in ServiceNow. Reports show 200 laptops "In Use" when 40 have actually been retired.

**Why It Happens:**
IT staff process the physical work (deploy the laptop, dispose of the device) but skip the ServiceNow workflow step to update asset state. It feels like "extra admin work."

**Root Cause:**
Asset lifecycle management requires consistent process discipline. If IT staff can bypass the ServiceNow workflow, they will.

**Fix / Prevention:**
- Make ServiceNow ITAM the trigger for physical work, not the record-keeping after
- Asset Task workflows: the IT tech STARTS work in ServiceNow (receive task), performs it, and marks complete
- Build accountability into the process: monthly asset audit compares ServiceNow state vs. physical reality
- Manager reviews: asset tasks not completed within SLA escalate to IT manager

---

## Issue 4: Entitlements Not Loaded — License Workbench Shows No Compliance Data

**Category:** Configuration / SAM  
**Severity:** Critical — SAM has no value without entitlements

**The Problem:**
ServiceNow SAM is deployed, Discovery is finding software installations, but the License Workbench shows every product as "Unknown compliance" or shows compliance data only for products where entitlements were manually entered. 90% of the software portfolio shows no compliance position.

**Why It Happens:**
Entitlements (what you're licensed to use) must be entered into ServiceNow from procurement records. This is a manual data load task that requires significant effort — it's commonly skipped or partially completed.

**Root Cause:**
SAM compliance = Entitlements vs. Usage. If entitlements don't exist in ServiceNow, the math can't be done.

**Fix / Prevention:**
- Make entitlement loading a go-live prerequisite — SAM is not live until top 20 products have entitlements loaded
- Integrate with procurement system or software asset management spreadsheets for bulk entitlement load
- Prioritize: focus first on highest-spend vendors (Microsoft, Oracle, SAP, Adobe)
- Ongoing: procurement workflow should create/update entitlements in SAM when licenses are purchased

---

## Issue 5: HRSD Not Integrated — License Reclaim at Offboarding Is Manual

**Category:** Integration  
**Severity:** High — direct cost impact (wasted license spend)

**The Problem:**
Employees leave the company but their software licenses (Microsoft 365, Salesforce, Adobe Creative Cloud) remain assigned. Over 12 months, a company with 20% annual turnover has 20% of licenses assigned to departed employees. License reclaim never happens, and the company pays for unused seats every renewal cycle.

**Why It Happens:**
Without HRSD integration, ITAM doesn't know when employees leave. IT might find out days or weeks later when a manager mentions it, or never.

**Root Cause:**
License reclaim requires a trigger from HR. Without the integration, licenses are orphaned invisibly.

**Fix / Prevention:**
- Integrate HRSD offboarding workflow with ITAM: employee departure event → license reclaim task created automatically
- Set SLA: all software licenses reclaimed within 24 hours of departure
- Monthly audit: licenses assigned to inactive users flagged automatically
- High-cost SaaS (Salesforce at $150+/month/user) should have immediate reclaim automation

---

## Issue 6: Oracle / SAP Licensing Miscalculated in Virtual Environments

**Category:** Configuration / SAM  
**Severity:** Critical — multi-million dollar audit exposure

**The Problem:**
Oracle licensing requires counting ALL processor cores in a VMware cluster where Oracle software could run — not just the cores where it does run. Companies configure Oracle Publisher Pack but don't account for the VMware partitioning factor. SAM calculates they need 20 licenses; Oracle says they owe 80 licenses based on the full cluster.

**Why It Happens:**
Oracle's "hard partitioning vs soft partitioning" rules are extremely complex. Publisher Packs automate the calculation but require correct input data about the virtualization environment.

**Root Cause:**
VMware is "soft partitioning" in Oracle's view — Oracle can see all CPUs in a cluster. Only hard partitioning (LPAR on IBM, OVM) counts specific processors. Without this distinction configured in SAM, the calculation is wrong.

**Fix / Prevention:**
- Engage a specialist for Oracle SAM — this is genuinely complex and errors cost millions
- Ensure Discovery captures full VMware cluster topology, not just individual VM CPU counts
- Oracle Publisher Pack requires proper VMware infrastructure data to calculate correctly
- For Oracle specifically: consider Oracle LMS (License Management Services) engagement before audit risk

---

## Issue 7: End-of-Life Assets Not Tracked or Acted On

**Category:** Reporting / Integration  
**Severity:** Medium-High — security and compliance risk

**The Problem:**
ITAM contains asset records with warranty expiry and end-of-life dates, but no process exists to act on EOL approaching assets. IT discovers EOL hardware and software in a security audit, not from ITAM dashboards.

**Why It Happens:**
EOL tracking data exists in ITAM but nobody has subscribed to the dashboard or set up automated notifications. The data is there — the process to act on it isn't.

**Root Cause:**
ITAM data without a consumption process provides no value. EOL dates are passive data unless someone actively monitors them.

**Fix / Prevention:**
- Configure scheduled reports: "Assets reaching EOL in next 90 days" — auto-emailed to IT managers
- SecOps integration: EOL assets flagged with higher vulnerability priority automatically
- Hardware refresh workflow: when asset hits EOL − 180 days, a Refresh Project is created in SPM
- Software EOL: vendor end-of-support dates tracked; SecOps and IT alerted when support ends

---

## Issue 8: Cloud Assets Not in ITAM — Shadow Cloud Spend

**Category:** Scope / Cloud ITAM  
**Severity:** High — significant untracked spend

**The Problem:**
ITAM tracks on-premises hardware and software licenses but cloud resources (AWS EC2 instances, Azure VMs, SaaS subscriptions purchased on credit cards) are completely invisible. Finance is getting surprise cloud bills; IT doesn't know what cloud services are in use.

**Why It Happens:**
Traditional ITAM implementations focus on physical assets and enterprise software. Cloud assets require different discovery mechanisms (API integration with AWS/Azure) and different mental models.

**Root Cause:**
Cloud ITAM is a separate capability that requires explicit setup. It doesn't happen automatically from Discovery (which scans internal networks).

**Fix / Prevention:**
- Cloud ITAM: configure ServiceNow integrations with AWS (Cost Explorer API), Azure (Cost Management API), and GCP
- SaaS discovery: integrate with Identity Provider (Okta, Azure AD) to find all SaaS apps in use
- Cloud governance: establish a policy that all cloud services must be requested through ServiceNow catalog
- Tagging enforcement: require all cloud resources to have a "cost center" tag — ITAM reports by tag

---

## Issue 9: ITAM and CMDB Out of Sync — Asset Records Don't Match CI Records

**Category:** Data Quality / Integration  
**Severity:** Medium — reduces trust in both systems

**The Problem:**
The same physical server has an Asset record in ITAM showing "Retired" but the CI record in CMDB still shows "Operational." Alternatively, CMDB has a CI for a server that doesn't exist in ITAM as an Asset. SecOps vulnerability findings reference a CI that has no asset owner — nobody knows who is responsible.

**Why It Happens:**
ITAM and CMDB are managed by different teams (Asset Management vs. Discovery/CMDB team) with different processes. Without explicit integration, they drift apart.

**Root Cause:**
The `cmdb_ci` (CI record) and `alm_asset` (Asset record) are linked in ServiceNow but the link must be maintained. When one record is updated without updating the other, sync breaks.

**Fix / Prevention:**
- Business Rule: when Asset state changes to "Retired," auto-update linked CI state to "Retired"
- Monthly reconciliation: report showing Assets and CIs that are out of sync
- Single ownership: assign a team responsible for BOTH asset and CI record for each device class
- Discovery should update both Asset and CI records in the same run — verify this is configured

---

## Issue 10: Stockroom Management Not Adopted — Assets Lost Between Receipt and Deployment

**Category:** Process / HAM  
**Severity:** Medium — asset tracking gap

**The Problem:**
Assets are received by IT and placed in a physical storage room, but ServiceNow's Stockroom records aren't updated. Assets sit in the "In Stock" state but no one knows they're physically available. New equipment is purchased when existing stock is available — waste and delay.

**Why It Happens:**
IT receiving staff don't have a streamlined process to update ServiceNow when assets arrive. The physical stockroom and the digital stockroom are disconnected.

**Root Cause:**
Receiving workflow requires staff to complete Asset Tasks in ServiceNow when equipment arrives. Without this step being enforced, physical and digital stock diverge.

**Fix / Prevention:**
- Barcode scanning at receiving: scan asset → ServiceNow mobile app → automatic Stockroom update
- Receiving Task workflow: IT receiving cannot mark "complete" until ServiceNow asset record is updated
- Real-time stockroom dashboard: IT managers see available stock before approving new purchase requests
- Periodic physical audit: count physical stock vs. ServiceNow — investigate any discrepancy

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| License Workbench shows "Unknown compliance" | Entitlements not loaded | Load entitlements for top software products |
| Duplicate hardware asset records | Identification Rules using volatile IDs | Switch to serial number / MAC as identifier |
| Software compliance wildly inaccurate | Software normalization not configured | Enable Publisher Packs; run normalization |
| Departed employees still have licenses | No HRSD integration | Build offboarding → license reclaim integration |
| ITAM and CMDB show different asset states | No sync between tables | Add Business Rule to sync state changes |
| Cloud costs invisible in ITAM | Cloud ITAM not configured | Add AWS/Azure API integration |
| EOL assets not being replaced | No EOL notification process | Configure scheduled EOL reports and refresh workflows |

---

## Sources and References

- ServiceNow Community: community.servicenow.com — ITAM, HAM, SAM forums
- ServiceNow Docs: ITAM Implementation Guide, SAM Best Practices
- ServiceNow Knowledge Conference sessions on "SAM Pro Implementation"
- LinkedIn: "ITAM implementation lessons" by ServiceNow practitioners
- Oracle LMS documentation on hard vs. soft partitioning
- ServiceNow Now Create ITAM methodology documentation
