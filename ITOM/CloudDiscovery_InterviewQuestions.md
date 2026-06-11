# ServiceNow Cloud Discovery — Interview Questions & Answers
## AWS and Azure Cloud-Based Discovery

---

## PART 1: Core Concepts

**Q1: What is Cloud Discovery and how does it differ from on-premises Discovery?**
**A:** Cloud Discovery uses cloud provider APIs to automatically discover and populate CMDB with cloud resources (VMs, databases, storage, networks). The key differences from on-premises discovery:

| On-Premises | Cloud Discovery |
|------------|-----------------|
| Scans IP ranges with network probes | Calls cloud management APIs |
| Requires SSH/WMI credentials on target hosts | Uses IAM roles / Service Principals — no stored passwords |
| MID Server connects directly to device IPs | MID Server calls cloud APIs (usually internet-accessible) |
| Finds: servers, switches, printers | Finds: VMs, RDS, S3, VPCs, Lambdas, AKS, etc. |

**Key advantage:** Cloud Discovery is API-driven and credential-less (for AWS) — no agents on cloud instances, no stored passwords on the MID Server. It discovers resources you did not even know existed (shadow IT).

---

**Q2: What is IAM Role-based discovery for AWS and why is it preferred?**
**A:** IAM Role-based discovery allows ServiceNow to discover AWS resources without storing any AWS credentials (access keys or passwords) on the MID Server.

**How it works:**
1. An IAM Role is created in the target AWS account with read-only permissions on the services to be discovered
2. The MID Server (or the IAM principal it runs as) is given `sts:AssumeRole` permission for this role
3. When Discovery runs, the MID Server calls AWS STS to assume the role, receiving temporary credentials (valid 1 hour)
4. Discovery uses these temporary credentials to call EC2, RDS, S3 APIs etc.

**Why preferred:**
- No permanent credentials stored anywhere — reduces attack surface
- Temporary credentials rotate automatically — no manual rotation needed
- Cross-account discovery is easy: create the same role in each account
- Full audit trail via AWS CloudTrail: every API call is logged

---

**Q3: What AWS resources does ServiceNow Cloud Discovery support?**
**A:** ServiceNow discovers a wide range of AWS resource types including:

| Service | CI Class |
|---------|----------|
| EC2 Instances | `cmdb_ci_vm_instance` |
| S3 Buckets | `cmdb_ci_cloud_storage_account` |
| RDS Instances | `cmdb_ci_db_instance` |
| VPCs | `cmdb_ci_network` |
| Elastic Load Balancers | `cmdb_ci_lb_service` |
| Lambda Functions | `cmdb_ci_cloud_function` |
| EKS Clusters | Container cluster CI |
| Security Groups | Firewall rule set CI |
| Elastic IPs | IP Address CI |
| Auto Scaling Groups | (metadata linked to EC2 CIs) |

The list expands with each ServiceNow release as AWS adds new services.

---

**Q4: How does Azure Cloud Discovery authenticate to Azure APIs?**
**A:** Azure Discovery uses an **Azure Service Principal** (App Registration in Azure AD) with Reader permissions on the target subscription(s).

**Setup steps:**
1. Create an App Registration in Azure AD → this creates a Service Principal
2. Create a Client Secret for the App Registration (or use a certificate — more secure)
3. Grant the Service Principal `Reader` role on the target subscription(s) or at Management Group level
4. In ServiceNow: create a Discovery credential with Tenant ID, Client ID (App ID), and Client Secret

**Alternative — Managed Identity:** If the MID Server runs as an Azure VM, assign it a Managed Identity and grant it Reader permissions on the subscription. This is fully credential-less — no Client Secret stored anywhere.

---

**Q5: What Azure resources does ServiceNow Cloud Discovery support?**
**A:** Key Azure resource types discovered by ServiceNow:

| Azure Resource | CI Class |
|----------------|----------|
| Virtual Machines | `cmdb_ci_vm_instance` |
| Storage Accounts | `cmdb_ci_cloud_storage_account` |
| Virtual Networks | `cmdb_ci_network` |
| SQL Databases | `cmdb_ci_db_instance` |
| App Services / Function Apps | `cmdb_ci_cloud_function` |
| Load Balancers | `cmdb_ci_lb_service` |
| AKS Clusters | Container cluster CI |
| Subscriptions | `cmdb_ci_cloud_service_account` |

---

**Q6: How do you handle multi-account AWS discovery?**
**A:** The recommended pattern for multiple AWS accounts:

1. **Create the same IAM Role** (same name, same permissions) in every target account
2. **Trust policy:** Each role's trust policy allows assumption from the MID Server's IAM principal (the "hub" account)
3. **In ServiceNow:** Create a Cloud Account record for each AWS account, each pointing to the same role name in different accounts
4. **Discovery runs:** ServiceNow iterates through all Cloud Account records, assumes the role in each account, discovers resources independently

**Key result:** Resources from each account are discovered separately and tagged with their AWS Account ID in CMDB. Resources from Account A and Account B never mix in the CMDB even though discovered by the same ServiceNow instance.

---

**Q7: How do you map cloud resource tags to CMDB fields?**
**A:** Cloud resource tags are mapped to CMDB CI fields in the Discovery transformation rules.

**Common mappings:**

| Cloud Tag | CMDB Field | Purpose |
|-----------|-----------|---------|
| `Owner` | `assigned_to` | Who is responsible for this resource |
| `Environment` | `environment` | prod / dev / test |
| `Application` | linked to Application CI | Which application this resource belongs to |
| `CostCenter` | `cost_center` | Financial allocation |
| `Team` | `assignment_group` | Which team manages this resource |
| `Service` | Link to Service Map | Tag-Based Service Mapping |

**Why this matters:** Tags are the primary governance mechanism in cloud. If teams apply tags consistently, ServiceNow can automatically populate CMDB with accurate ownership, environment, and cost allocation data — without manual data entry.

---

**Q8: What is credential-less discovery and how does it work for AWS vs. Azure?**
**A:**

**AWS — Fully Credential-less (IAM Role):**
- MID Server assumes an IAM Role in the target account
- AWS STS issues temporary credentials (expire in 1 hour)
- No permanent access key stored on MID Server or in ServiceNow
- Cross-account: MID Server can assume roles in any account it is trusted by

**Azure — Near Credential-less (Managed Identity):**
- MID Server runs as an Azure VM with a Managed Identity assigned
- Azure automatically manages the identity credentials — no secrets stored anywhere
- Managed Identity is granted Reader on target subscriptions
- Technically no stored secret, but the MID Server VM must be an Azure VM (not on-premises)

**Azure — Service Principal (standard approach, not fully credential-less):**
- Client Secret is stored in ServiceNow Credential Store (encrypted)
- Client Secret must be rotated periodically (typically 1-2 years)
- More common than Managed Identity when MID Server is on-premises

---

## PART 2: Implementation and Troubleshooting

**Q9: Walk me through setting up AWS Discovery from scratch.**
**A:**
1. **Create IAM Role in AWS:**
   - Create a new IAM Role in the target account
   - Attach a policy with read permissions for all services to discover (ec2:Describe*, rds:Describe*, s3:List*, etc.)
   - Configure trust policy to allow assumption by the MID Server's IAM User or EC2 role

2. **Configure MID Server:**
   - Ensure MID Server has network access to AWS API endpoints (aws.amazon.com — port 443)
   - If MID Server is an EC2 instance: attach an instance profile with sts:AssumeRole permission
   - If MID Server is on-premises: create an IAM User with sts:AssumeRole permission, configure AWS CLI on MID Server host

3. **Create Cloud Account in ServiceNow:**
   - Navigate to Discovery → Cloud Accounts → New
   - Account type: Amazon Web Services
   - Enter AWS Account ID and IAM Role ARN
   - Assign MID Server

4. **Create Discovery Schedule:**
   - Type: Cloud (AWS)
   - Select the Cloud Account created
   - Set schedule frequency (nightly recommended)
   - Enable the discovery types needed (EC2, RDS, S3, etc.)

5. **Test:**
   - Trigger a manual discovery run
   - Monitor ECC Queue for errors
   - Verify CI records appear in CMDB with correct attributes
   - Check for any IAM permission errors in the Discovery log

---

**Q10: AWS Discovery is running but S3 buckets are not being discovered. What do you investigate?**
**A:**
1. **Check IAM permissions:** Confirm the IAM Role policy includes `s3:GetBucketLocation` and `s3:ListAllMyBuckets`. S3 permissions are often forgotten because S3 is not "compute."

2. **Check S3 Discovery configuration:** In the Discovery Schedule, confirm S3 is enabled as a discovery type — it may be unchecked.

3. **Check region configuration:** S3 ListAllMyBuckets is a global API call, not regional — ensure the Discovery Schedule is not restricted to a specific region that excludes the S3 global endpoint.

4. **Check for bucket policy restrictions:** Some S3 buckets have bucket policies that explicitly deny access to specific IAM principals. If the discovery IAM Role is blocked by a bucket policy, that bucket will not be discovered.

5. **Review ECC Queue errors:** Look for S3-specific errors in the ECC Queue input records from the last Discovery run.

---

**Q11: How do you handle cloud resource CI staleness — instances that were terminated but still show in CMDB?**
**A:** Terminated cloud instances do not automatically disappear from CMDB because ServiceNow only creates/updates CIs — it does not delete them by default.

**Approaches:**

1. **CI Lifecycle Management via Discovery:** Configure the Discovery Schedule's CI staleness settings — if an EC2 instance is not discovered for X consecutive runs, automatically change its CI state to "Absent" or "Retired"

2. **AWS EventBridge integration:** Subscribe to EC2 state change events (instance termination) via AWS EventBridge → ServiceNow REST API. When a termination event fires, a Flow Designer flow updates the CMDB CI to "Retired" immediately — no waiting for the next Discovery run

3. **Daily reconciliation job:** A scheduled job compares CMDB EC2 CIs in "Operational" state against the current live EC2 inventory via the AWS API. Any CI not in the live inventory is flagged for review

**Best practice:** Combine near-real-time event-driven updates (EventBridge) for state changes with scheduled discovery for catching any missed events.

---

**Q12: What is the difference between discovering cloud infrastructure and cloud services?**
**A:**
- **Cloud Infrastructure (IaaS):** Resources you manage — EC2 instances, Azure VMs, databases you deploy. You are responsible for the OS, configuration, and application. Discovery treats these like on-premises servers.

- **Cloud Services (PaaS/SaaS):** Managed services where the provider handles the infrastructure — AWS Lambda, Azure Functions, AWS RDS (managed), Azure Cosmos DB. You only interact via API or console.

**Discovery differences:**
- IaaS: Discovery can introspect the OS (if agent/SSH access is available) AND discover via API
- PaaS: Discovery only uses cloud APIs — no OS-level access. CI records have fewer attributes (no OS version, no RAM, because these are managed by AWS/Azure)
- Serverless: No persistent infrastructure to discover — Lambda functions are discovered as function definitions, not running instances

---

**Q13: How does Cloud Discovery integrate with Service Mapping?**
**A:** Cloud Discovery and Service Mapping are complementary for cloud services:

1. **Cloud Discovery populates CMDB with cloud CIs:** EC2 instances, RDS databases, ELBs, VPCs
2. **Service Mapping builds relationships between them:** Traces from the load balancer entry point through EC2 instances to RDS — OR uses Tag-Based Mapping to build topology from resource tags
3. **Result:** A complete service map that includes both the cloud infrastructure layer (from Cloud Discovery) and the dependency relationships (from Service Mapping)

**Tag-Based Mapping relies on Cloud Discovery:** For Tag-Based Service Mapping to work, the cloud resources must be in the CMDB (from Cloud Discovery). The tags on those CI records are what Service Mapping reads to build the topology.

---

## PART 3: Advanced

**Q14: How would you set up discovery for an organization with 50 AWS accounts?**
**A:**
1. **Standardize the IAM Role:** Create a CloudFormation StackSet that deploys the same IAM Role (name: `ServiceNowDiscoveryRole`) with the same permissions across all 50 accounts simultaneously. Use AWS Organizations to deploy via StackSet.

2. **Hub-and-spoke trust:** The trust policy on each role allows assumption only from the MID Server's IAM principal in the hub/management account. This centralizes control.

3. **Automated Cloud Account creation:** Write a script or use the ServiceNow REST API to create Cloud Account records in ServiceNow for each new AWS account as it is onboarded — triggered by AWS Organizations account creation events.

4. **Single Discovery Schedule covering all accounts:** ServiceNow iterates through all Cloud Account records in a single scheduled run — no need for 50 separate schedules.

5. **Tag-based isolation:** Require `Account` and `Environment` tags on all resources. Map these to CMDB `company` and `environment` fields for proper tenant isolation.

---

**Q15: What are the security best practices for Cloud Discovery credentials?**
**A:**
**AWS:**
- Use IAM Role assumption (no stored credentials) — always preferred
- Apply least-privilege permissions: only the specific `Describe*`, `List*`, `Get*` actions needed
- Enforce MFA for any IAM User used for discovery (if IAM User, not role, must be used)
- Enable AWS CloudTrail: all API calls made by the discovery role are logged
- Restrict the trust policy: only allow assumption from the specific MID Server principal, not `*`

**Azure:**
- Prefer Managed Identity over Service Principal if MID Server is an Azure VM
- If using Service Principal: use a certificate instead of a Client Secret for authentication
- Assign Reader role at the lowest necessary scope (specific subscription, not whole tenant)
- Rotate Client Secrets on a schedule (at minimum annually) — maintain rotation runbook in Confluence
- Enable Azure AD audit logs to track Service Principal authentication events

**General:**
- Never grant write or delete permissions to Discovery credentials — read-only is sufficient and reduces blast radius if credentials are compromised
- Separate credentials per environment: PROD discovery credential ≠ DEV discovery credential
