# ServiceNow CMDB — Common Issues, Pitfalls & Troubleshooting

Issues encountered during CMDB implementation and ongoing support, based on ServiceNow community posts, partner experience, and real-world implementations.

---

## Issue 1: Duplicate CI Records from Discovery — The #1 CMDB Problem

**Category:** Configuration / Discovery  
**Severity:** Critical — the most common CMDB failure mode

**The Problem:**
After running Discovery for several weeks, the CMDB is filled with duplicate CI records. A server that exists once in reality has 3-5 records in the CMDB. Relationship maps are broken, impact analysis is unreliable, and CI counts are inflated.

**Why It Happens:**
Identification Rules use volatile attributes — usually IP address or hostname — as the matching key. Virtual machines change IPs, servers get renamed, and Discovery creates a new record instead of finding the existing one.

**Root Cause:**
Identification Rules (IRE — Identification and Reconciliation Engine) must use stable, unique identifiers. IP addresses are not stable. Hostnames change.

**Fix / Prevention:**
- Use Serial Number + MAC Address as primary identifiers for physical hardware
- For VMs: use the VM UUID (vSphere/Hyper-V VM Identifier)
- For servers: combine hostname + domain name if hostname alone is reused across environments
- After fixing rules: run CMDB Deduplication to merge existing duplicate records
- Monitor: after each Discovery run, check "newly created vs. updated" ratio — spikes indicate new duplicates

**Community Reference:** ServiceNow Community — "IRE duplicate records" — hundreds of threads, top CMDB issue by volume

---

## Issue 2: Stale CI Records — The CMDB Doesn't Reflect Reality

**Category:** Data Quality  
**Severity:** High — CMDB becomes a historical archive, not a current map

**The Problem:**
Discovery runs daily but decommissioned servers remain as "Operational" in the CMDB for months or years. CI records exist for servers that were physically removed. Impact analysis recommends contacting team owners for servers that no longer exist.

**Why It Happens:**
Discovery is good at finding what IS in the environment. It's not automatically good at marking CI records for systems that have been removed — especially if a server is turned off gracefully before disposal.

**Root Cause:**
Absent systems stop being discovered — but the CMDB record remains unless a cleanup process actively manages CI lifecycle.

**Fix / Prevention:**
- Configure CI Staleness: if a CI is not discovered for X days, automatically change state to "Absent"
- Set CI staleness thresholds by CI class (servers: 14 days; network devices: 7 days)
- After "Absent" state, workflow notifies CI owner: "This system wasn't found — confirm decommission?"
- Integrate with change management: Change for "Decommission" should retire the CI record

---

## Issue 3: CMDB Health Score Is Low — Nobody Acts on It

**Category:** Governance  
**Severity:** High — CMDB quality degrades without active management

**The Problem:**
The CMDB Health Dashboard shows 40% completeness. The dashboard has been showing 40% for 18 months. Nobody is assigned to improve it. It's treated as an interesting metric, not an actionable task.

**Why It Happens:**
CMDB health dashboards are implemented but ownership isn't assigned. "IT owns CMDB" is too vague — specific teams must own specific CI classes.

**Root Cause:**
CMDB health improvement requires: (1) defined targets, (2) specific owners, (3) a remediation workflow, and (4) management accountability. Without all four, it stays at 40% forever.

**Fix / Prevention:**
- Assign CI Class Owners: "Server team owns all `cmdb_ci_server` records and is accountable for completeness"
- Set quarterly health targets: "By Q3, server completeness must reach 85%"
- Create Remediation Tasks automatically from CMDB Health — low-completeness records generate tasks
- Include CMDB health score in team KPIs / quarterly business reviews

---

## Issue 4: Reconciliation Rules Not Configured — Wrong Data Wins

**Category:** Configuration  
**Severity:** High — incorrect data stored in CMDB

**The Problem:**
Multiple data sources populate the same CI attributes. Discovery says a server has 32GB RAM. The ITAM Asset record says 16GB. A manual update from a field engineer says 64GB. The CMDB stores whichever value was most recently written — often the wrong one.

**Why It Happens:**
Reconciliation Rules define which data source is authoritative for each attribute. Without explicit rules, last-write wins — a destructive default.

**Root Cause:**
When no Reconciliation Rule exists for an attribute, any data source can overwrite it. A manual entry by a junior engineer can override Discovery data from 10,000 CI records.

**Fix / Prevention:**
- Define data source hierarchy: Discovery > ITSM > Manual Entry for hardware attributes
- Set Reconciliation Rules explicitly for all key attributes (hostname, RAM, CPU, OS, IP address)
- Lock critical attributes: "only Discovery can update hostname" — block manual overwrites
- Review Reconciliation Logs monthly to see what's being overwritten and by what source

---

## Issue 5: Service Mapping Fails to Build Complete Maps

**Category:** Configuration / Service Mapping  
**Severity:** High — one of Service Mapping's most common failure points

**The Problem:**
Service Mapping is configured and initiated but produces incomplete maps — showing only 2-3 layers instead of the full application stack. Or the map doesn't connect correctly to the business service. Or mapping runs for hours and times out.

**Why It Happens — Multiple Causes:**
1. **Credentials not configured:** Service Mapping needs credentials (SSH for Linux, WMI for Windows) to probe each CI — missing credentials = mapping stops at that layer
2. **MID Server not reaching the subnet:** Application servers in a DMZ or isolated network segment aren't reachable from the MID Server
3. **Pattern not matching:** Vertical Discovery pattern doesn't recognize the specific version of middleware or database
4. **Firewall blocking probes:** Service Mapping probes are blocked between network segments

**Fix / Prevention:**
- Credentials: create a dedicated Service Mapping service account with read access to all target systems
- MID Server placement: verify MID Server can reach ALL subnets that contain application CIs
- Test patterns: run pattern test on a sample CI before full mapping
- Firewall rules: document required ports (SSH=22, WMI=135, SNMP=161) and verify they're open

**Community Reference:** ServiceNow Community — "Service Mapping incomplete topology" — high-traffic thread

---

## Issue 6: MID Server Single Point of Failure

**Category:** Architecture  
**Severity:** High — Discovery and Service Mapping stop completely when MID Server goes down

**The Problem:**
A single MID Server is installed for all Discovery. When the MID Server host machine requires maintenance or fails, Discovery stops, price refreshes stop, and CMDB stops receiving updates. This is noticed days later when CIs start going stale.

**Why It Happens:**
Single MID Server deployments are common in initial implementations to keep it simple. High-availability considerations are deferred.

**Root Cause:**
MID Server has no built-in clustering — redundancy must be explicitly architected.

**Fix / Prevention:**
- Deploy at least 2 MID Servers in production (can cover same IP ranges)
- ServiceNow automatically load-balances across multiple MID Servers for the same application
- Place MID Servers in different availability zones if using cloud
- Monitor MID Server health in ServiceNow — alert when a MID Server goes offline

---

## Issue 7: Too Many CI Classes — CMDB Becomes Hard to Manage

**Category:** Configuration / Governance  
**Severity:** Medium — management overhead increases

**The Problem:**
During initial implementation, the CMDB team creates custom CI classes for every type of IT asset imaginable — 50+ custom classes. Over time, relationships between classes become unclear, queries become complex, and new team members can't navigate the CMDB structure.

**Why It Happens:**
"We need a custom class for this specific type of asset" is a common implementation pattern. Without governance, class proliferation occurs.

**Root Cause:**
ServiceNow has a rich out-of-box CI class hierarchy. Custom classes should only be created when out-of-box classes truly can't support the use case.

**Fix / Prevention:**
- Default to using OOTB CI classes — they're designed for 90% of use cases
- Before creating a custom class: can this be handled with a custom attribute on an existing class?
- Annual CI class review: identify unused classes and consolidate
- Follow CSDM guidance on CI class hierarchy

---

## Issue 8: CMDB Not Used for Change Management Impact Assessment

**Category:** Adoption / ITSM Integration  
**Severity:** High — reduces change management effectiveness

**The Problem:**
Change Management has a "CI affected" field that links to CMDB, but change managers don't use it. Changes are submitted without CI references. Impact analysis is never run before changes. Change-related outages continue because nobody knows what depends on the CI being changed.

**Why It Happens:**
Change managers and change requestors find it burdensome to search for and attach CIs. If it's not mandatory, it doesn't happen.

**Root Cause:**
CMDB value for change management is opt-in rather than required. Without making CI attachment mandatory, the integration isn't used.

**Fix / Prevention:**
- Make CI field mandatory on Change Request form — cannot submit without at least one CI
- When CI is selected, automatically show dependent CIs and affected business services
- Change Advisory Board (CAB): review impact analysis as standard agenda item
- "Change blackout" capability: CMDB maintenance windows block changes to specific CIs automatically

---

## Issue 9: Discovery Running Without Credentials — Incomplete CI Data

**Category:** Configuration  
**Severity:** Medium — CIs discovered at surface level only

**The Problem:**
Discovery finds servers by IP scan (ping) but can't connect via SSH or WMI because credentials aren't configured. Servers appear in the CMDB as basic IP records — no hostname, no OS version, no installed software, no running processes. The CMDB has the IP but nothing useful.

**Why It Happens:**
Discovery credentials require a service account with appropriate access on every target system. Provisioning this account can be politically difficult — security teams restrict it, server teams resist it.

**Root Cause:**
Comprehensive Discovery requires a service account with read access on target systems. Without it, Discovery is limited to network-layer information only.

**Fix / Prevention:**
- Work with security team: define minimum-privilege service account for Discovery (read-only, no write access)
- Test credentials: use Discovery's "Test Credentials" feature before running full scan
- Tag CIs with missing credentials: "Credential Required" flag — track progress toward full coverage
- Automate: integrate with Active Directory or CyberArk to provision credentials without manual steps

---

## Issue 10: CSDM Not Followed — Other ServiceNow Modules Can't Consume CMDB Data

**Category:** Architecture / Standards  
**Severity:** High — limits platform value

**The Problem:**
CMDB has lots of data but SPM, IRM, SecOps, and ITOM can't use it effectively because the data isn't structured per CSDM (Common Service Data Model). Business Services exist but aren't linked to Application Services or Technical Services in the CSDM hierarchy. SecOps vulnerability findings can't be business-impact-scored because the service mapping layer doesn't exist.

**Why It Happens:**
CSDM implementation requires effort and architectural understanding. Teams often build CMDB data structures organically without following the CSDM blueprint.

**Root Cause:**
Without CSDM alignment, each module uses CMDB data differently — creating integration friction and inconsistencies across the platform.

**Fix / Prevention:**
- Adopt CSDM before building any Application Service records
- Layer: Business Capability → Business Application → Application Service → Technical Service → Infrastructure CIs
- Use CSDM Health Dashboard to measure compliance with CSDM standards
- ServiceNow provides a CSDM Workbook — complete it during design phase, before implementation

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Duplicate CI records multiplying | Identification Rules using IP/hostname | Switch to Serial Number / VM UUID |
| Stale CIs not updated after decommission | No CI staleness process | Configure CI Staleness threshold |
| CMDB health score not improving | No ownership assigned | Assign CI Class Owners with targets |
| Wrong attribute values stored | Reconciliation Rules missing | Define source priority per attribute |
| Service Mapping incomplete | Missing credentials or MID Server can't reach CI | Check credentials; verify MID Server network access |
| CI data exists but modules can't use it | CSDM not followed | Align to CSDM hierarchy |
| Discovery stops when MID Server is down | Single MID Server | Deploy secondary MID Server |

---

## Sources and References

- ServiceNow Community: community.servicenow.com — CMDB, Discovery, Service Mapping forums
- ServiceNow Docs: CMDB Implementation Guide, CSDM Workbook, Discovery Best Practices
- ServiceNow Knowledge Conference sessions: "CMDB Health Best Practices", "CSDM in Practice"
- LinkedIn: ServiceNow CMDB practitioners sharing "lessons learned" implementation posts
- ServiceNow Now Create CMDB methodology documentation
