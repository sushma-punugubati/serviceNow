# ServiceNow CMDB — Interview Notes
## "What did you do in CMDB?" + Challenges Faced

---

## 1. OVERVIEW — What is CMDB?

The CMDB (Configuration Management Database) is the central repository in ServiceNow that stores all Configuration Items (CIs) and their relationships. It is the backbone of the entire platform — ITSM, ITOM, SecOps, IRM, and CSM all depend on accurate CMDB data for impact analysis, change risk assessment, root cause analysis, and service mapping.

A well-maintained CMDB is the difference between knowing "a server went down" and knowing "a server went down and it hosts the payment processing application which supports 3 business services and 400 active customer contracts." The relationships are where the real value lives, not just the inventory.

Key areas worked on:
- Discovery scheduling and MID Server setup
- CSDM (Common Service Data Model) framework alignment
- Service Graph Connectors (SCCM, Intune)
- AWS and Azure cloud discovery
- Identification rules and reconciliation engine (IRE)
- CMDB health score and data quality remediation
- Event Management and CI binding

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 CSDM Alignment at Dell

- Led the effort to align our CMDB to the **Common Service Data Model (CSDM)** framework — the goal was to structure CI data consistently across the platform so Service Mapping, Event Management, and reporting all worked from the same data model
- Mapped existing CI classes to the correct CSDM layers: hardware, software, application services, technical services, and business services — fixed relationship types so the dependency graph reflected actual infrastructure relationships
- Used **Service Graph Connectors** to pull in data from SCCM with proper relationship mapping instead of raw import sets that ignored the model — this cleaned up a significant amount of stale and mis-classified CI data
- Created CI class governance documentation so new CIs created by any team followed the CSDM structure consistently

### 2.2 AWS and Azure Cloud Discovery

- Configured **AWS credential-less discovery** using IAM role-based authentication — granted the MID Server an IAM role with minimum required permissions (no stored credentials, no agents on instances). ServiceNow assumes the IAM role and discovers EC2 instances, S3 buckets, RDS databases, VPCs, and related resources automatically
- Wrote custom scripts to map each discovered AWS resource to the correct customer account in CMDB using role-based discovery, keeping multi-tenant environments cleanly separated without manual CI maintenance
- Configured **Azure cloud discovery** to scan and pull cloud resources (VMs, storage accounts, virtual networks, subscriptions) into the CMDB automatically — wrote custom CI classification scripts to ensure Azure resource types mapped to the correct CMDB CI classes
- Set up scheduled Discovery runs for both cloud environments with appropriate intervals: near-real-time for EC2 state changes, daily full scans for inventory reconciliation

### 2.3 MID Server Setup and Management

- Set up and configured multiple **MID Servers** for different network segments — each segment had its own MID Server with appropriate credentials and network access to the devices in that zone
- Managed MID Server performance: monitored the Work Queue depth and worker thread utilization, tuned concurrent thread counts based on network segment complexity, and set up alerts when the Work Queue exceeded threshold
- Configured **pattern-based and credential-less discovery** where possible to reduce the attack surface from stored credentials on the MID Server
- Handled MID Server upgrades as part of the platform upgrade process — validated MID Server compatibility with the new ServiceNow version before platform promotion

### 2.4 Identification Rules and Reconciliation

- Tuned **IRE (Identification and Reconciliation Engine)** rules to use stable identifiers: serial numbers for physical hardware, VM UUIDs for virtual machines, hostname + domain for servers where UUIDs were unavailable
- Fixed duplicate CI records caused by IP address being used as the primary identifier for virtual machines that changed IPs — merged duplicates using the CMDB Deduplication feature and updated identification rules to prevent recurrence
- Configured **reconciliation rules** to define which data source was authoritative for each attribute — Discovery was authoritative for IP/OS/RAM, SCCM was authoritative for installed software, manual entry was blocked for auto-discovered fields

### 2.5 Event Management and CI Binding

- Integrated **Splunk alerts** into ServiceNow Event Management — infrastructure events flowed into the Event Management console automatically with proper alert rules and thresholds to reduce noise
- Set up **CI binding rules** so each alert mapped to the correct CI in the CMDB — when Splunk fired an alert for a specific server hostname, the Event Management rule matched it to the CMDB CI record using the host.name field
- Configured alert deduplication and correlation rules to prevent alert storms from flooding the console — grouped related alerts into a single Event record and only created one incident per group

---

## 3. CHALLENGES FACED

### Challenge 1: Duplicate CIs from Multiple Discovery Sources

**Problem:** After enabling both Discovery and Service Graph Connectors (SCCM + Intune), the CMDB started accumulating duplicate CI records. A laptop that existed once physically now had 3 records — one from Discovery, one from SCCM, one from Intune — each with slightly different hostnames or IP addresses.

**Solution:** Implemented IRE identification rules that used the combination of serial number + MAC address as the primary key for hardware assets. For laptops without serial numbers (some BIOS configurations don't expose them), used the combination of hostname + domain. Ran the CMDB Deduplication tool to merge existing duplicates, then monitored the "newly created vs. updated" ratio in subsequent Discovery runs to confirm the rules were working.

---

### Challenge 2: CSDM Alignment — Resistance to Reclassifying Existing CIs

**Problem:** When we started the CSDM alignment project, several application teams pushed back on reclassifying their CIs. They had built reports and dashboards using the old CI class structure and were worried that reclassification would break their existing views.

**Solution:** Created a phased migration plan: new CIs followed CSDM immediately; existing CIs were migrated by class in a prioritized order (starting with the classes that were furthest from CSDM standards). Ran a pre-migration audit to identify which reports would be affected and updated them before migrating the underlying CI class. Communication and stakeholder management was as important as the technical migration.

---

### Challenge 3: AWS Discovery Not Populating All Resource Types

**Problem:** After setting up AWS cloud discovery, EC2 instances were discovered correctly but S3 buckets and RDS databases were missing from the CMDB. The IAM role had been granted EC2 read permissions but the team had missed the permissions for S3:GetBucketLocation and rds:DescribeDBInstances.

**Solution:** Reviewed the ServiceNow documentation for required AWS permissions per resource type, updated the IAM policy to include all required permissions, and re-ran discovery. Added a post-discovery validation check: a scheduled report comparing expected resource counts from AWS Cost Explorer to CMDB CI counts per resource type — a delta of more than 5% triggers an alert for investigation.

---

### Challenge 4: CI Binding Failures in Event Management

**Problem:** Splunk was sending events with IP addresses in the host.name field. Our CMDB stored CIs by hostname, not IP address. CI binding was failing for about 40% of events, resulting in unbound alerts sitting in the console with no CI context.

**Solution:** Added a secondary CI binding rule: if hostname binding fails, attempt IP-to-CI lookup using the IP address attribute on the cmdb_ci_server table. Also worked with the Splunk team to update alert templates to include both hostname and IP in the payload, so ServiceNow had two fields to try for binding.

---

### Challenge 5: CMDB Health Score — Data Without Action

**Problem:** The CMDB Health Dashboard showed 38% completeness score. It had been at 38% for 14 months. No one owned it, no one acted on it. The score was reported as a metric but never improved.

**Solution:** Broke the health score into actionable CI class-level targets assigned to specific owners: the networking team owned network device completeness, the server team owned server CI completeness. Set quarterly targets and tied them to change advisory board approval — changes touching CIs below a completeness threshold required a CMDB remediation plan as part of the change record.

---

### Challenge 6: MID Server Performance Under Heavy Discovery Load

**Problem:** During a full infrastructure scan, the MID Server Work Queue depth climbed above 5,000 items and Discovery effectively stalled. Probes were timing out and CI records were not being updated.

**Solution:** Analyzed the MID Server thread configuration — it was running with default 25 worker threads for a network segment with 3,000+ devices. Increased worker threads to 75, split the Discovery schedule into two off-peak windows to reduce concurrent load, and separated credential-intensive probes (SSH/WMI) from lighter SNMP probes onto different Discovery schedules.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **CI (Configuration Item)** | Any component that needs to be managed to deliver IT services — servers, apps, databases, network devices |
| **CSDM** | Common Service Data Model — ServiceNow's framework for structuring CMDB data consistently across all modules |
| **IRE** | Identification and Reconciliation Engine — determines if a discovered CI is new or existing, and which data source wins for each attribute |
| **Service Graph Connector** | Pre-built integration that pulls external data (SCCM, Intune, AWS, Azure) into CMDB with proper CSDM relationship mapping |
| **MID Server** | Java agent on the internal network acting as a bridge between ServiceNow cloud and internal infrastructure |
| **CI Binding** | Matching an event, alert, or incident to its corresponding CI in the CMDB |
| **Credential-less Discovery** | Discovery using API calls (AWS IAM role, Azure AD app registration) rather than stored username/password — more secure |
| **Reconciliation Rules** | Define which data source is authoritative for each CI attribute — prevents Discovery from overwriting manually-set fields |

---

## 5. METRICS / OUTCOMES TO QUOTE

- CMDB completeness improved from **38% to 74%** after ownership-based remediation program
- AWS/Azure cloud discovery reduced manual CI maintenance effort by **~90%** — cloud resources updated automatically vs. weekly manual spreadsheet updates
- CI binding success rate in Event Management improved from **60% to 96%** after dual hostname/IP binding rules
- Duplicate CI rate reduced from **~12% to under 1%** after IRE tuning and deduplication
- CSDM alignment enabled Service Mapping accuracy improvement: service topology maps went from **45% accuracy to 82%**
- MID Server Discovery throughput increased **3x** after thread tuning and schedule splitting
