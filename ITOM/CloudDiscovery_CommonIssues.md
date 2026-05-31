# ServiceNow Cloud Discovery — Common Issues, Pitfalls & Troubleshooting
## AWS and Azure Cloud-Based Discovery

---

## Issue 1: AWS Discovery Failing — IAM Permissions Insufficient

**Category:** Configuration / Authentication
**Severity:** Critical — no cloud resources being discovered

**The Problem:**
Cloud Discovery runs but no EC2 or RDS instances appear in CMDB. The ECC Queue shows errors referencing "Access Denied" or "not authorized to perform: ec2:DescribeInstances."

**Why It Happens:**
The IAM Role the MID Server is assuming does not have the required permissions for the services being discovered. Common mistake: the role has EC2 permissions but not RDS, S3, or Lambda permissions — those resource types silently fail.

**Root Cause:**
IAM Role policy missing required service permissions.

**Fix / Prevention:**
- Review the IAM Role policy and compare against ServiceNow's documented minimum permissions list for each resource type
- Test permissions manually: from the MID Server, use the AWS CLI to run `aws ec2 describe-instances --region us-east-1` — if it succeeds, EC2 permissions are correct
- Add permissions incrementally: start with EC2, verify it works, add RDS, verify, etc.
- Use AWS IAM Policy Simulator to test the discovery role's permissions before enabling Discovery

---

## Issue 2: Azure Discovery Not Discovering All Subscriptions

**Category:** Configuration / Authentication Scope
**Severity:** High — some subscriptions missing from CMDB

**The Problem:**
ServiceNow discovers resources in 3 of 8 Azure subscriptions. The other 5 subscriptions' resources are completely absent from CMDB even though they are active.

**Why It Happens:**
The Service Principal (App Registration) was granted the Reader role only on specific subscriptions, not at the Management Group level. When new subscriptions were added to the organization, the Service Principal was not granted access to them.

**Root Cause:**
Service Principal role assignment scope too narrow — subscription-level instead of Management Group level.

**Fix / Prevention:**
- Assign the Service Principal Reader role at the **Management Group** level (covers all current and future subscriptions)
- Alternatively, automate role assignment: when a new subscription is created in Azure, a governance policy automatically grants the ServiceNow Service Principal Reader access
- Periodically audit: compare subscriptions in Azure portal to subscriptions represented in CMDB

---

## Issue 3: Terminated EC2 Instances Remaining as "Operational" in CMDB

**Category:** Data Quality / CI Lifecycle
**Severity:** High — stale CIs causing incorrect impact analysis

**The Problem:**
CMDB shows 3,200 EC2 instances in "Operational" state. Running a live query against AWS shows only 2,100 active instances. 1,100 terminated instances are still "Operational" in CMDB — they were terminated without CMDB being updated.

**Why It Happens:**
ServiceNow Discovery creates and updates CI records but does not automatically retire them when cloud resources are terminated. Discovery only discovers what is running — it does not diff against what was previously running.

**Root Cause:**
No CI lifecycle management configured for cloud CIs.

**Fix / Prevention:**
- Configure **CI Staleness**: in the Discovery Schedule, set a staleness threshold — CIs not discovered for 2+ consecutive runs automatically move to "Absent" state
- Implement **EventBridge integration**: subscribe to EC2 `terminated` state change events, trigger a ServiceNow REST API call to retire the CI immediately
- Run a **monthly reconciliation**: scheduled report comparing CMDB "Operational" EC2 CIs against live AWS inventory via the AWS API — flag CIs that no longer exist in AWS

---

## Issue 4: Cloud Resource Tags Not Mapping to CMDB Fields

**Category:** Configuration / Data Mapping
**Severity:** Medium-High — CMDB CIs missing ownership and cost allocation data

**The Problem:**
EC2 instances are being discovered correctly but the `assigned_to`, `cost_center`, and `environment` fields on the CI records are all blank. Teams cannot identify who owns each cloud resource from the CMDB.

**Why It Happens:**
Cloud resource tags exist on the EC2 instances (Owner, CostCenter, Environment) but the Discovery transformation rules have not been configured to map these tag values to the corresponding CMDB CI fields.

**Root Cause:**
Tag-to-field mapping not configured in Discovery transformation rules.

**Fix / Prevention:**
- In the Cloud Account's Discovery configuration, add tag mapping rules: `Tag "Owner" → CI field "assigned_to"`, `Tag "CostCenter" → CI field "cost_center"`
- Account for case-sensitivity: AWS tags are case-sensitive and teams may use "owner" vs. "Owner" inconsistently — add mapping rules for both variants
- Add a post-discovery validation report: CIs discovered in the last 24 hours with blank `assigned_to` field — this surfaces untagged or incorrectly tagged resources

---

## Issue 5: Duplicate CI Records for Cloud Resources After Discovery

**Category:** Data Quality / IRE
**Severity:** High — CMDB integrity issues

**The Problem:**
After enabling Cloud Discovery alongside the existing SCCM-based discovery, EC2 instances are getting duplicate CI records. Each EC2 instance now has two CI records — one from Cloud Discovery (using AWS Instance ID as the key) and one from SCCM (using hostname as the key).

**Why It Happens:**
IRE identification rules for the `cmdb_ci_vm_instance` class use hostname as the primary identifier. SCCM discovered the instance with hostname `prod-app-01`. Cloud Discovery discovered the same instance with AWS Instance ID `i-0abc123456789`. These do not match, so IRE creates a new record instead of updating the existing one.

**Root Cause:**
Multiple data sources using different primary identifiers for the same CI class, and IRE not configured to handle cross-source correlation.

**Fix / Prevention:**
- Add the AWS Instance ID as a secondary identifier in the IRE identification rules for `cmdb_ci_vm_instance` — try hostname first, then AWS Instance ID as fallback
- Alternatively, configure SCCM discovery to populate the AWS Instance ID field (available from the AWS Systems Manager agent installed on the instance)
- Run CMDB Deduplication to merge existing duplicate records
- In future: use AWS Systems Manager Session Manager tags to ensure SCCM includes the Instance ID in its discovery data

---

## Issue 6: AWS Discovery Not Discovering Cross-Region Resources

**Category:** Configuration / Region Scope
**Severity:** Medium-High — cloud resources in some regions missing from CMDB

**The Problem:**
The organization uses EC2 instances in us-east-1, us-west-2, and eu-west-1. Cloud Discovery only shows resources from us-east-1 — the other two regions are completely missing.

**Why It Happens:**
The Discovery Schedule is configured with a specific AWS region (us-east-1). EC2, RDS, and other regional services in other regions are not scanned.

**Root Cause:**
Discovery Schedule not configured for multi-region discovery.

**Fix / Prevention:**
- Update the Discovery Schedule to enable "Discover all regions" or add the additional regions explicitly to the region list
- For global services (S3, IAM, CloudFront): these are not region-specific and should be discovered via the global endpoint
- Review all Cloud Account configurations and confirm the region list includes all regions where the organization operates

---

## Issue 7: Azure Service Principal Client Secret Expiry — Discovery Stops Working

**Category:** Operations / Credential Management
**Severity:** High — cloud discovery silently stops

**The Problem:**
Azure Discovery worked fine for 11 months, then suddenly stopped discovering new VMs and updating existing CI records. No alert was generated. The issue was only discovered during a CMDB audit.

**Why It Happens:**
The Azure Service Principal Client Secret was configured with a 12-month expiry (Azure default). After 12 months, the Client Secret expired and all API calls return 401 Unauthorized. ServiceNow does not automatically generate an alert when a Cloud Account credential expires.

**Root Cause:**
Client Secret expiry without monitoring or renewal process.

**Fix / Prevention:**
- Set a ServiceNow SLA on the credential expiry date: when the Client Secret expiry date is within 30 days, create a task for the ITOM team to renew it
- Switch to certificate-based authentication (certificates can be set to longer validity periods and generate more visible warnings on expiry)
- If MID Server is an Azure VM, migrate to Managed Identity — no credential expiry
- Calendar reminder: add a reminder 45 days before any Service Principal secret expiry in the team's shared calendar

---

## Issue 8: Cloud Discovery Performance — Large AWS Account Taking Too Long

**Category:** Performance / Scheduling
**Severity:** Medium — discovery window overrun, stale data

**The Problem:**
An AWS account with 15,000 EC2 instances takes 14 hours to complete a discovery run. By the time discovery finishes, the data is half a day old. The organization needs more frequent and faster discovery.

**Why It Happens:**
Large accounts with many resources require many sequential API calls. Default Discovery configuration does not parallelize API calls across regions.

**Root Cause:**
Single-threaded Discovery run processing large API response volumes sequentially.

**Fix / Prevention:**
- Enable **parallel region discovery**: configure Discovery to run API calls for each region in parallel rather than sequentially
- Use **incremental discovery**: after the initial full scan, only re-discover resources that have changed since the last run (use AWS Config or CloudTrail to identify recently changed resources)
- Split large accounts: use separate Discovery Schedules for different resource types (EC2 schedule, RDS schedule, S3 schedule) with different frequencies based on how volatile each type is
- Add a second MID Server in the same region as the AWS account to distribute API call load

---

## Issue 9: Resource Group / VNet Hierarchy Not Reflecting in CMDB for Azure

**Category:** Configuration / Relationship Mapping
**Severity:** Medium — Azure organizational structure not visible in CMDB

**The Problem:**
Azure VMs are discovered and CI records are created correctly. However, the CMDB does not show which Resource Group each VM belongs to, or which VNet/Subnet it is connected to. The Azure hierarchy is flat in CMDB — just a list of VMs with no grouping.

**Why It Happens:**
Resource Group and VNet discovery is enabled separately from VM discovery. If only VM discovery was enabled when configuring the Discovery Schedule, the containing Resource Groups and networking components are not discovered.

**Root Cause:**
Incomplete Cloud Discovery type configuration — VM enabled but Resource Group, VNet, and Subnet discovery not enabled.

**Fix / Prevention:**
- Enable all relevant discovery types in the Discovery Schedule: Virtual Machines AND Resource Groups AND Virtual Networks AND Subnets
- After enabling, verify the relationship records in `cmdb_rel_ci` show VMs linked to their VNet and Resource Group
- Add a post-discovery relationship check: VMs without a VNet relationship are flagged for review

---

## Issue 10: Multi-Tenant CI Isolation Breaking — Cross-Account CI Contamination

**Category:** Configuration / Data Isolation
**Severity:** Critical — security and compliance risk

**The Problem:**
In a multi-account AWS environment, CMDB CI records for resources from Account A (Production) and Account B (Development) are being mixed in the same Company in CMDB. Development team members can see Production CI records.

**Why It Happens:**
The Cloud Discovery configuration does not map the AWS Account ID to the CMDB `company` field. All discovered CIs fall into the default company or are left blank, making them visible to all users.

**Root Cause:**
Missing tag-to-company mapping and no CMDB ACL enforcing tenant isolation.

**Fix / Prevention:**
- Map `Account ID` (from Cloud Discovery metadata) → CMDB `company` field in the transformation rules
- Require `Company` tag on all AWS resources: `Production`, `Development` — map this tag to CMDB `company` field
- Configure CMDB ACLs to restrict CI visibility by `company` field — users in the Production team only see CIs where `company = Production`
- Run an audit: identify all cloud CIs with blank `company` field and remediate
