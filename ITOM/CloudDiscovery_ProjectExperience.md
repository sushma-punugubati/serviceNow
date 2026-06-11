# ServiceNow Cloud Discovery — Interview Notes
## "What did you do in Cloud Discovery?" + Challenges Faced

---

## 1. OVERVIEW — What is Cloud Discovery?

Cloud Discovery extends ServiceNow's CMDB population to cloud infrastructure by using cloud provider APIs instead of network probes. Rather than scanning IP ranges with credentials, Cloud Discovery assumes IAM roles (AWS) or uses Service Principals (Azure) to call cloud APIs and discover every resource in an account — including resources IT had no idea existed.

At Dell, Cloud Discovery was transformative for our AWS and Azure footprints. Before Cloud Discovery, cloud infrastructure was either undiscovered or manually maintained in CMDB — both approaches resulted in stale, incomplete data. After Cloud Discovery, CMDB cloud CI coverage went from roughly 60% to over 98%, and we discovered hundreds of resources that had been created outside the normal provisioning process.

Key work I did:
- AWS credential-less discovery using IAM role assumption across multiple accounts
- Azure cloud discovery using Service Principal with Reader permissions
- Custom CI-to-tenant mapping scripts for multi-tenant environments
- Tag-to-CMDB field mapping configuration
- Cloud CI lifecycle management (handling terminated instances)
- Integration with Service Mapping for cloud service topology

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 AWS Credential-Less Discovery

- Configured **AWS credential-less discovery** using IAM role assumption — the MID Server assumes a `ServiceNowDiscoveryRole` in each target AWS account via `sts:AssumeRole`. No access keys or passwords stored anywhere
- Worked with the AWS infrastructure team to define the **minimum IAM permissions** needed for each resource type: `ec2:Describe*` for EC2, `rds:Describe*` for RDS, `s3:GetBucketLocation` + `s3:ListAllMyBuckets` for S3, `elasticloadbalancing:Describe*` for ELBs, `lambda:List*` for Lambda
- Deployed the IAM Role via **AWS CloudFormation StackSet** across all AWS accounts in our organization simultaneously — no manual role creation per account. The trust policy allowed assumption only from our specific MID Server IAM principal
- Configured **Cloud Account records** in ServiceNow for each AWS account, with the corresponding IAM Role ARN. A single Discovery Schedule iterates through all accounts in one run

### 2.2 Multi-Tenant AWS Discovery with CMDB Isolation

- Wrote **custom CI classification scripts** in the Discovery transformation rules to map each discovered AWS resource to the correct customer account in CMDB using the AWS Account ID and resource tags
- All resources in AWS Account 123456789 mapped to CMDB Company "Production Engineering"; all resources in Account 987654321 mapped to "Development Engineering" — cross-contamination prevented at the data mapping layer
- Set up **CMDB ACLs** so each team could only see CI records for their own company — Production team members could not see Development cloud CIs and vice versa
- Added the `Environment` AWS tag (`production`, `staging`, `development`) as a CMDB CI attribute — used by Change Management for automatic risk scoring (changes to production CIs carry higher risk)

### 2.3 Azure Cloud Discovery

- Configured **Azure cloud discovery** using a Service Principal with Reader permissions granted at the Management Group level — covers all current and future subscriptions without needing to update permissions when new subscriptions are added
- Discovered Azure VMs, Storage Accounts, Virtual Networks, Subnets, SQL Databases, and App Services — configured each resource type in the Discovery Schedule
- Set up **tag-to-CMDB field mapping**: Azure `Owner` tag → CI `assigned_to`, Azure `CostCenter` tag → CI `cost_center`, Azure `Application` tag → link to Application CI for Service Mapping
- Configured **Subnet-to-VNet relationships** in CMDB — each discovered VM CI is linked to its subnet, which links to the VNet, which links to the Subscription CI. The full Azure networking hierarchy is visible in CMDB

### 2.4 Cloud CI Lifecycle Management

- Implemented **CI staleness management** for cloud CIs: EC2 instances and Azure VMs not discovered in 2 consecutive runs automatically move to "Absent" state. After 7 days in Absent state, they transition to "Retired"
- Built a **reconciliation job** that runs nightly: compares CMDB "Operational" cloud CIs against live AWS/Azure inventory via API. CIs in CMDB but not in live inventory are flagged in a report for review the next morning
- For critical accounts, implemented **EventBridge integration** for near-real-time CI state updates: EC2 instance termination events trigger immediate CMDB CI retirement via a ServiceNow REST API call — no waiting for the next Discovery run

### 2.5 Integration with Service Mapping

- After Cloud Discovery populated the CMDB with AWS CI records, configured **Tag-Based Service Mapping** for the cloud-native applications — the `Application` and `Tier` tags on cloud resources are read by Service Mapping to build service topology
- The result: CMDB has both the infrastructure CIs (from Cloud Discovery) and the service topology relationships (from Tag-Based Service Mapping) — complete service maps for cloud applications

---

## 3. CHALLENGES FACED

### Challenge 1: AWS IAM Permissions Causing Silent Partial Discovery

**Problem:** After enabling Cloud Discovery for a large AWS account, EC2 instances were being discovered correctly but RDS databases and Lambda functions were completely absent from CMDB. No error was visible in the Discovery Status record — the run completed with "Success" status.

**Solution:** Investigated the ECC Queue input records in detail (not just the summary) and found silent "Access Denied" errors specifically for RDS and Lambda API calls. The IAM Role policy had EC2 permissions but the team that created the role had not included `rds:Describe*` or `lambda:List*`. Added the missing permissions to the IAM policy and re-ran Discovery. Also added a post-discovery validation report: compares expected resource types in the Discovery Schedule configuration against resource types actually present in CMDB — a missing resource type triggers an investigation task.

---

### Challenge 2: Multi-Tenant CI Isolation — Resources from Different Accounts Cross-Contaminating

**Problem:** When I set up Cloud Discovery for the second AWS account, CI records from both accounts started mixing in the CMDB. A developer in Account B (Development) could see Production server CIs from Account A. This was a compliance violation — production infrastructure details should not be visible to developers.

**Solution:** The issue was that the initial Discovery setup had no Company mapping — all CIs went into the default company. Added two layers of isolation:

First, configured the Discovery transformation rule to set the `company` field based on the AWS Account ID attribute returned with each resource (`Account ID 123456789 → company = "Production"`). This tagged all CIs correctly at discovery time.

Second, wrote a one-time remediation script to back-fill the `company` field on all existing CIs that had been discovered without it.

Third, added CMDB ACLs restricting CI visibility by `company` — Production company CIs only visible to Production team group.

After both changes, cross-tenant visibility was fully eliminated.

---

### Challenge 3: Terminated EC2 Instances Inflating CMDB — Ghost CIs

**Problem:** Six months after enabling Cloud Discovery, I noticed the EC2 CI count in CMDB was 4,800 — but our AWS Console showed only 2,600 running or stopped instances. About 2,200 CI records were for terminated instances that had been deleted from AWS but never updated in CMDB. These ghost CIs were showing up in impact analysis and causing false alarms.

**Solution:** Three-part fix:

First, enabled CI staleness rules: EC2 CIs not seen in 3 consecutive daily Discovery runs (3 days) automatically move to Absent state.

Second, built the reconciliation job: nightly comparison of CMDB Operational EC2 CIs against live AWS inventory. Found and retired 2,200 ghost CIs in the first run.

Third, for accounts with rapid instance turnover (auto-scaling groups), implemented EventBridge: EC2 termination events trigger a REST call to ServiceNow immediately retiring the CI — CMDB accurate within minutes of termination, not days.

---

### Challenge 4: Azure Client Secret Expiry Causing Silent Discovery Failure

**Problem:** Three months after I had set up Azure Discovery, the CMDB Azure VM count stopped updating. New VMs were being provisioned in Azure but not appearing in CMDB. No alert was generated. The issue was discovered when a project team asked why their new Azure VMs were not in the CMDB for a change request.

**Solution:** Checked the Azure Cloud Account record in ServiceNow — the last successful discovery run was 12 weeks prior, exactly when the Client Secret had expired. Added the Client Secret expiry date to the CMDB Configuration Item for the ServiceNow platform itself, and set up a PA indicator that alerts 30 days before any cloud credential expires. Renewed the Client Secret immediately. Going forward, also explored migrating to certificate-based authentication (certificates can be set to 2-year validity) and for the cloud-hosted MID Server, started the migration to Managed Identity to eliminate the secret entirely.

---

### Challenge 5: Cloud Discovery and On-Premises Discovery Creating Duplicate CIs

**Problem:** EC2 instances that also had an SCCM agent installed were getting two CI records: one from Cloud Discovery (keyed on Instance ID) and one from SCCM-based Discovery (keyed on hostname). CMDB had 800+ duplicate pairs, breaking impact analysis and Service Mapping for the affected instances.

**Solution:** Added the AWS Instance ID as a secondary identifier in the IRE identification rule for `cmdb_ci_vm_instance`. The rule now tries hostname first (for SCCM-discovered records), then falls back to Instance ID (for Cloud-discovered records). When the same instance is discovered by both sources, IRE recognizes them as the same CI and merges the attributes per the reconciliation rules — SCCM wins for OS/software data, Cloud Discovery wins for IP, region, and account attributes. Ran CMDB Deduplication to merge the 800 existing duplicate pairs. Duplicate rate returned to under 1%.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **IAM Role Assumption (AWS)** | MID Server assumes an IAM Role in target accounts via STS — no stored credentials, temporary tokens rotate automatically |
| **Credential-less Discovery** | Discovery without permanent stored credentials — IAM roles (AWS) or Managed Identity (Azure) |
| **Service Principal (Azure)** | App Registration in Azure AD with Reader permissions on target subscriptions — used to authenticate Azure API calls |
| **Cross-Account Discovery** | Same IAM Role deployed in all AWS accounts, MID Server assumes each role in sequence |
| **Tag Mapping** | Cloud resource tags (Owner, Environment, Application) mapped to CMDB CI fields for automatic data population |
| **CI Lifecycle Management** | Handling terminated/deleted cloud resources — CI staleness rules or event-driven retirement |
| **Multi-Tenant Isolation** | Account ID + Company tag → CMDB company field → ACL restriction ensures tenant data separation |
| **EventBridge Integration** | Near-real-time CI updates triggered by AWS state change events (instance termination, stop, start) |

---

## 5. METRICS / OUTCOMES TO QUOTE

- AWS Cloud Discovery: cloud CI CMDB coverage improved from **~60% to 98%+** — discovered 800+ unregistered resources in first scan
- Zero stored credentials for AWS Discovery — fully credential-less IAM role assumption across **multiple AWS accounts**
- Ghost CI problem resolved: **2,200 stale terminated EC2 instances** retired from CMDB in first reconciliation run
- Multi-tenant isolation: **100% of cloud CIs** correctly assigned to company with CMDB ACL enforcement after mapping fix
- Duplicate CI rate reduced to **under 1%** after IRE cross-source identification rule update
- Azure Client Secret expiry monitoring: zero undetected credential expiry incidents since adding 30-day advance alert
