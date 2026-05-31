# ServiceNow CSDM — Common Issues, Pitfalls & Troubleshooting

---

## Issue 1: Resistance to CI Reclassification — Teams Do Not Want Their CIs Changed

**Category:** Change Management / Governance
**Severity:** High — CSDM implementation stalls

**The Problem:**
The CSDM alignment project requires reclassifying 2,000 application CIs from generic `cmdb_ci_appl` to the correct CSDM classes. Application owners push back — their dashboards, reports, and monitoring alerts all reference the old CI class. Changing them "will break everything."

**Why It Happens:**
CSDM reclassification requires coordination across multiple teams. Every consumer of CMDB data (dashboards, reports, integrations, scripts) that references the old CI class must be updated before the migration — otherwise reclassification does cause disruption.

**Root Cause:**
No dependency analysis performed before reclassification — impact of the change on existing consumers not understood.

**Fix / Prevention:**
- Before reclassifying any CI class, run a report of all records that reference the class (reports, PA indicators, Business Rules, scripts, integration queries)
- Update all consumers to use the new CSDM class first
- Only reclassify CIs after all consumers are updated
- Use a phased migration: create new CIs in the correct CSDM class, run in parallel with old CIs for one sprint, then retire old CIs

---

## Issue 2: Application Service Layer Missing — All Features Anchored Incorrectly

**Category:** Data Model / Architecture
**Severity:** Critical — ITSM, ITOM, and Event Management all produce wrong results

**The Problem:**
Service Mapping creates beautiful topology maps, but Event Management cannot correlate alerts to business services. Change impact analysis returns empty results. ITSM incident CI field links to servers, not services. All CMDB features seem to be working in isolation rather than together.

**Why It Happens:**
The Application Service layer was never created. The CMDB has infrastructure CIs (servers, databases) but no Application Service records linking them to business services. Without the Application Service layer, there is no bridge between technical events and business impact.

**Root Cause:**
CSDM implementation focused only on infrastructure CIs (Foundation level) without creating the service layers (Crawl/Walk).

**Fix / Prevention:**
- Create Application Service records for the top critical services immediately
- Link Application Services to the infrastructure CIs that support them (Service Mapping can do this automatically)
- Link Application Services to Business Applications for the full CSDM chain
- Validate by testing an Event Management alert: a bound alert on an infrastructure CI should propagate impact to the Application Service and Business Service via CSDM relationships

---

## Issue 3: Service Graph Connector Creating Duplicate CIs Alongside Existing Discovery Records

**Category:** Data Quality / IRE
**Severity:** High — CMDB inflation, wrong CI counts

**The Problem:**
After deploying the SCCM Service Graph Connector, every Windows server now has two CI records: one from ServiceNow Discovery (created for months before the connector was deployed) and one from the SCCM connector. The connector's IRE identification rules are not matching the existing Discovery CI records.

**Why It Happens:**
ServiceNow Discovery created CI records using hostname as the primary identifier. The SCCM Service Graph Connector uses a different identifier (SCCM ResourceID or hardware GUID). IRE cannot match across the two different identifier types, so it creates new records.

**Root Cause:**
IRE identification rule mismatch between Discovery-created CIs and Service Graph Connector-created CIs.

**Fix / Prevention:**
- Before deploying a Service Graph Connector, review its IRE identification rules and compare to the existing Discovery identification rules
- Add a cross-source correlation rule: if SCCM ResourceID is found on a CI, use that to match; fallback to hostname + domain
- Run the CMDB Deduplication tool after deployment to merge existing duplicates
- Going forward: configure Discovery to populate SCCM ResourceID on server CIs so the connector can find the existing record on first import

---

## Issue 4: CSDM Relationship Types Not Matching — Service Mapping Creates Non-Standard Relationships

**Category:** Configuration / CSDM Compliance
**Severity:** High — BSM maps exist but Event Management cannot use them

**The Problem:**
Service Mapping has been running and maps look visually correct. But the CMDB Health Score shows low compliance. Event Management binds alerts to CIs but cannot traverse to the business service. Impact analysis in Change Management returns empty results.

**Why It Happens:**
Service Mapping patterns were configured using custom relationship types ("Deployed On," "Part Of") rather than CSDM-standard types ("Runs on," "Depends on"). The relationship records exist but use non-CSDM type names that Event Management and Change Impact Analysis cannot traverse.

**Root Cause:**
Service Mapping pattern relationship steps not updated to use CSDM-standard relationship types after CSDM adoption.

**Fix / Prevention:**
- Review all Service Mapping patterns' relationship step configurations — update them to use CSDM-standard relationship types
- After updating patterns, re-run Service Mapping for all services — this will recreate relationship records with the correct types
- Add a CSDM compliance check to the Service Mapping post-run validation: any newly created relationship using a non-standard type triggers a review task

---

## Issue 5: Business Application Records Not Linked to Application Services

**Category:** Data Model / Walk Level
**Severity:** Medium-High — Application Portfolio incomplete, Change Management lacking context

**The Problem:**
Business Application records exist (created by the Application Portfolio team) and Application Service records exist (created by ITOM/Service Mapping). But they are not linked to each other. From a CSDM perspective, the business layer and technical layer are two separate islands.

**Why It Happens:**
Business Application records and Application Service records were created by different teams (business vs. IT) using different processes without a formal linking step. No one owned the connection between the two layers.

**Root Cause:**
No defined ownership for the Business Application → Application Service relationship in CSDM.

**Fix / Prevention:**
- Create a formal "CSDM link" process: whenever a new Application Service is created, the IT owner must identify and link the parent Business Application within 5 business days
- Build a report: Application Service records with no Business Application parent — assign as weekly tasks to Application Service owners
- During the initial remediation: run a matching script using application name similarity to auto-suggest links, then have owners confirm

---

## Issue 6: CMDB Health Score Not Improving Despite Remediation Efforts

**Category:** Governance / Measurement
**Severity:** Medium — team effort not reflected in score

**The Problem:**
The CMDB team has been working for 3 months to improve data quality. They have fixed hundreds of CI records manually. But the CMDB Health Score has not moved — still showing 42% after all the work.

**Why It Happens:**
Manual fixes to individual CI records do not address the root cause — the data sources (Discovery, import sets) continue creating non-compliant records every day. Fixing individual records is overwhelmed by the ongoing stream of non-compliant data.

**Root Cause:**
Remediation focused on symptoms (fixing existing records) rather than root cause (fixing the data sources creating non-compliant records).

**Fix / Prevention:**
- Shift the remediation focus from fixing existing records to fixing the data pipelines creating non-compliant records
- For each compliance issue, trace it to its source: which Discovery probe, which import set, which Service Graph Connector created this non-compliant record?
- Fix the source — fix it once and every future record from that source will be compliant
- Only manual-fix records that have no systematic source (true one-off data quality issues)

---

## Issue 7: CSDM Breaking Existing Performance Analytics Indicators

**Category:** Impact Management / Reporting
**Severity:** Medium — management dashboards broken during migration

**The Problem:**
After migrating server CIs from `cmdb_ci_computer` to the correct CSDM class `cmdb_ci_server`, multiple Performance Analytics indicators that were trending the server count and health metrics broke. The indicators were filtering on `cmdb_ci_computer` table records — now empty.

**Why It Happens:**
PA indicators reference specific table names. When CIs are reclassified to a different table (even one that extends the original), the indicator's data collection query no longer finds the records in the expected table.

**Root Cause:**
PA indicators not updated before CI reclassification completed.

**Fix / Prevention:**
- Before any CI reclassification, run a full audit of PA indicators that reference the source table
- Update indicators to query the new CSDM-aligned table before migration
- Where indicators should cover multiple CI classes (e.g., "all compute CIs" regardless of specific class), use the parent table `cmdb_ci_hardware` as the query scope
- Add PA indicator validation to the CSDM migration testing checklist

---

## Issue 8: CSDM Layer Confusion — Teams Using Wrong Entity Types

**Category:** Governance / Training
**Severity:** Medium — CSDM model inconsistent across teams

**The Problem:**
Different teams are using Application Service and Technical Service for the same things. Some teams create Technical Service records for applications (should be Application Service). Others create Application Service records for database clusters (should be Technical Service). The CSDM model is inconsistent across the organization.

**Why It Happens:**
The distinction between Application Service and Technical Service is subtle and the platform does not enforce the distinction technically. Teams without CSDM training use whatever feels right.

**Root Cause:**
Insufficient CSDM training and no governance enforcement.

**Fix / Prevention:**
- Create clear one-page CSDM guidance: "When to create an Application Service vs. a Technical Service" with 5 concrete examples from your organization
- Add a mandatory CSDM training module for anyone with CMDB write access
- Add a governance review step: new Application Service and Technical Service records above a threshold require CSDM admin approval before publication
- Build a report: Application Service and Technical Service records created in the last 30 days — weekly review by the CSDM governance team to catch misclassifications early

---

## Issue 9: Service Offering Not Linked to Service Catalog Items

**Category:** Data Model / Service Portfolio
**Severity:** Low-Medium — Service Catalog and CSDM disconnected

**The Problem:**
CSDM Service Offerings are defined and linked to Application Services. But the Service Catalog items that employees use to request those services have no connection to the Service Offering. SLAs defined on Service Offerings are not applied to requests coming through the catalog.

**Why It Happens:**
Service Catalog items and Service Offerings were built by different teams (ITSM team vs. CSDM/IT Architecture team) without a formal integration step. The platform supports linking catalog items to Service Offerings, but this was never configured.

**Root Cause:**
Missing integration step between Service Catalog build and CSDM Service Offering governance.

**Fix / Prevention:**
- Add a Service Offering lookup field to the catalog item creation process — every new catalog item must be linked to the appropriate Service Offering
- Use Service Offering to drive SLA assignment on catalog requests: request items inherit the SLA from their parent Service Offering rather than having manually configured SLAs per item
- Build a report: catalog items with no Service Offering link — remediate quarterly

---

## Issue 10: CSDM Not Reflecting Cloud Resources Correctly

**Category:** Configuration / Cloud CSDM
**Severity:** Medium — cloud infrastructure invisible in CSDM service view

**The Problem:**
On-premises services are well-mapped in CSDM. But cloud-native applications (running on AWS) have no Application Service records and their infrastructure CIs (EC2 instances) have no relationships to any service. Cloud workloads are invisible in BSM maps and impact analysis.

**Why It Happens:**
Cloud Discovery was deployed to populate CMDB with cloud CIs, but the Service Graph Connector for AWS was not configured to create CSDM relationships — only CI records. And no one created Application Service records for cloud-native applications.

**Root Cause:**
Cloud Discovery and CSDM implementation were treated as separate initiatives — discovery team did not create service-layer records, CSDM team did not include cloud applications in scope.

**Fix / Prevention:**
- Include cloud application owners in the Application Portfolio process — cloud-native apps need Business Application and Application Service records just like on-premises apps
- Configure the AWS Service Graph Connector relationship mapping to create CSDM-aligned relationships between EC2 instances and their VPCs, security groups, and load balancers
- Use Tag-Based Service Mapping for cloud applications to automatically create Application Service topology from AWS tags
