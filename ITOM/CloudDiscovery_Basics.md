# ServiceNow Cloud Discovery — Fundamentals & Study Guide
## AWS and Azure Cloud-Based Discovery

> Sources: ServiceNow Docs | AWS Discovery Implementation Guide | Azure Cloud Discovery Reference | ServiceNow Community

---

## 1. What is Cloud Discovery?

**Cloud Discovery** is ServiceNow's capability to automatically discover cloud infrastructure resources (virtual machines, databases, storage, networking components) and populate the CMDB with accurate, current CI records — without storing credentials on the MID Server.

> **Think of it as:** Extending ServiceNow Discovery into the cloud. Just as horizontal discovery scans on-premises IP ranges and creates CI records for physical servers, Cloud Discovery scans cloud accounts via APIs and creates CI records for virtual machines, managed databases, object storage, virtual networks, and other cloud resources.

**Why Cloud Discovery is different from on-premises Discovery:**

| On-Premises Discovery | Cloud Discovery |
|----------------------|-----------------|
| Scans IP ranges with probes | Calls cloud provider APIs |
| Uses stored credentials (SSH/WMI) | Uses IAM roles / Service Principals (no stored passwords) |
| MID Server connects to target IPs | MID Server calls cloud management APIs |
| Finds: servers, network devices, printers | Finds: VMs, databases, storage, VPCs, subscriptions |
| Manual IP range configuration | Discovers all resources in an account/subscription automatically |

---

## 2. AWS Cloud Discovery

### How AWS Discovery Works

ServiceNow uses the **AWS APIs** (via the MID Server) to query each AWS account for its resources. The MID Server assumes an **IAM Role** in the target account and calls AWS API endpoints to retrieve resource lists and configuration data.

```
ServiceNow → MID Server → AWS API (assume IAM Role) → EC2 API, RDS API, S3 API...
                                ↓
                         Resource data returned
                                ↓
                         CMDB CI records created/updated
```

### Credential-less Authentication — IAM Role

**AWS credential-less discovery** is the recommended approach — no username, password, or access key is stored on the MID Server.

**How it works:**
1. An IAM Role is created in the target AWS account with read-only permissions for the services to be discovered
2. The MID Server's EC2 instance (or the IAM User the MID Server runs as) is granted permission to **assume** this role via `sts:AssumeRole`
3. When Discovery runs, the MID Server calls `AWS STS AssumeRole` to get temporary credentials (valid for 1 hour)
4. Discovery uses these temporary credentials to call AWS APIs — no permanent credentials stored anywhere

**IAM Policy for Discovery (minimum permissions):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "rds:Describe*",
        "s3:GetBucketLocation",
        "s3:ListAllMyBuckets",
        "elasticloadbalancing:Describe*",
        "lambda:List*",
        "eks:Describe*",
        "eks:List*",
        "vpc:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Supported AWS Resource Types

| Resource Type | CI Class in CMDB | AWS API Used |
|---------------|-----------------|-------------|
| EC2 Instances | `cmdb_ci_vm_instance` | ec2:DescribeInstances |
| S3 Buckets | `cmdb_ci_cloud_storage_account` | s3:ListAllMyBuckets |
| RDS Instances | `cmdb_ci_db_instance` | rds:DescribeDBInstances |
| VPCs | `cmdb_ci_network` | ec2:DescribeVpcs |
| Security Groups | `cmdb_ci_ip_firewall_rule_set` | ec2:DescribeSecurityGroups |
| Elastic Load Balancers | `cmdb_ci_lb_service` | elasticloadbalancing:DescribeLoadBalancers |
| Lambda Functions | `cmdb_ci_cloud_function` | lambda:ListFunctions |
| EKS Clusters | `cmdb_ci_cloud_function` | eks:ListClusters |
| IAM Roles | (metadata, not CI) | iam:ListRoles |
| Elastic IPs | `cmdb_ci_ip_address` | ec2:DescribeAddresses |

### Multi-Account AWS Discovery

For organizations with multiple AWS accounts (typical in enterprise):
1. Create the same IAM Role (same name, same permissions) in each account
2. Configure the ServiceNow AWS discovery to use a single "hub" account that can assume roles into each target account via cross-account role assumption
3. Each account's resources are discovered independently and tagged with the correct account ID
4. CMDB CI records include the AWS Account ID as an attribute — resources from different accounts never mix

---

## 3. Azure Cloud Discovery

### How Azure Discovery Works

ServiceNow uses the **Azure Resource Manager (ARM) APIs** to discover Azure resources. Authentication uses an **Azure Service Principal** (or Managed Identity) with read permissions on the target subscription.

```
ServiceNow → MID Server → Azure ARM API (Service Principal) → Resource Groups, VMs, Storage...
                                ↓
                         Resource data returned
                                ↓
                         CMDB CI records created/updated
```

### Authentication — Azure Service Principal

**Creating the Service Principal:**
1. In Azure Active Directory, create an App Registration (this creates the Service Principal)
2. Create a Client Secret for the App Registration (or use a certificate)
3. Assign the Service Principal the **Reader** role on the target subscription(s)
4. In ServiceNow, create a Discovery credential with the Tenant ID, Client ID, and Client Secret

**Credential storage:** Unlike on-premises WMI/SSH credentials, the Azure Service Principal Client Secret is stored in the ServiceNow Credential Store. While not fully credential-less, the Client Secret can be rotated without changing the ServiceNow configuration (just update the Credential Store record).

**Alternative — Managed Identity:** If the MID Server runs as an Azure VM, it can use an Azure Managed Identity to authenticate to Azure APIs — fully credential-less, no stored secrets.

### Supported Azure Resource Types

| Resource Type | CI Class in CMDB | Azure API Resource Type |
|---------------|-----------------|------------------------|
| Virtual Machines | `cmdb_ci_vm_instance` | Microsoft.Compute/virtualMachines |
| Storage Accounts | `cmdb_ci_cloud_storage_account` | Microsoft.Storage/storageAccounts |
| Virtual Networks (VNet) | `cmdb_ci_network` | Microsoft.Network/virtualNetworks |
| Subnets | `cmdb_ci_ip_subnet` | Microsoft.Network/virtualNetworks/subnets |
| SQL Databases | `cmdb_ci_db_instance` | Microsoft.Sql/servers/databases |
| Load Balancers | `cmdb_ci_lb_service` | Microsoft.Network/loadBalancers |
| Azure Kubernetes Service | `cmdb_ci_cloud_function` | Microsoft.ContainerService/managedClusters |
| Azure Functions | `cmdb_ci_cloud_function` | Microsoft.Web/sites (kind: functionapp) |
| Resource Groups | (grouping, not CI) | Microsoft.Resources/resourceGroups |
| Subscriptions | `cmdb_ci_cloud_service_account` | Subscription-level metadata |

### Multi-Subscription Azure Discovery

For organizations with multiple Azure subscriptions:
1. Assign the Service Principal **Reader** role at the **Management Group** level (covers all subscriptions)
2. ServiceNow discovers all subscriptions the Service Principal has access to in a single run
3. CMDB CI records include the Subscription ID as an attribute for isolation

---

## 4. CMDB CI Mapping for Cloud Resources

### CSDM Alignment for Cloud CIs

Cloud resources must be mapped to the correct CSDM layer:

| Cloud Resource | CSDM Layer | CSDM CI Class |
|---------------|-----------|---------------|
| AWS EC2 / Azure VM | Infrastructure / Hardware | `cmdb_ci_vm_instance` |
| AWS RDS / Azure SQL | Application | `cmdb_ci_db_instance` |
| AWS Lambda / Azure Functions | Application | `cmdb_ci_cloud_function` |
| AWS VPC / Azure VNet | Infrastructure / Network | `cmdb_ci_network` |
| AWS S3 / Azure Storage | Infrastructure / Storage | `cmdb_ci_cloud_storage_account` |
| AWS EKS / AKS | Application | (container cluster CI) |

### Cloud Tags → CMDB Attributes

Cloud resource tags are a critical data source for CMDB attributes:
- `Owner` tag → CI `owned_by` field
- `Environment` tag → CI `environment` field
- `Application` tag → CI linked to Application CI
- `CostCenter` tag → CI `cost_center` field
- `Team` tag → CI `assignment_group` field

Mapping cloud tags to CMDB fields is configured in the Discovery transformation rules.

---

## 5. Role-Based Discovery for Multi-Tenant Environments

In enterprise and MSP environments, cloud resources from different tenants (business units, customers, subsidiaries) must be isolated in the CMDB.

### AWS Multi-Tenant Approach

Each tenant has a dedicated AWS account. The Discovery configuration uses a separate IAM Role assumption per account:
- Tenant A → Assume `DiscoveryRole` in Account 123456789
- Tenant B → Assume `DiscoveryRole` in Account 987654321
- Each account's resources tagged with `Company: TenantA` or `Company: TenantB`
- Discovery transformation maps `Company` tag → CMDB `company` field
- CMDB ACLs restrict Tenant A's team to only see CIs where `company = TenantA`

### Azure Multi-Tenant Approach

Each tenant has a dedicated Azure subscription. The Service Principal is granted Reader at the Management Group level, and all subscriptions are discovered. Each CI record is associated with its subscription, which maps to the company.

---

## 6. Cloud Discovery Schedule Configuration

### Key Configuration Elements

| Element | Purpose |
|---------|---------|
| **Discovery Schedule** | Defines when and how often cloud resources are scanned |
| **Cloud Account** | The AWS account or Azure subscription to discover |
| **MID Server** | Which MID Server executes the discovery (needs API access) |
| **Credential** | IAM Role ARN (AWS) or Service Principal (Azure) |
| **Inclusion/Exclusion Filters** | Limit discovery to specific regions, resource groups, or tags |

### Recommended Discovery Frequency

| Resource Type | Recommended Schedule |
|---------------|---------------------|
| Running VMs / EC2 instances | Every 6-12 hours (state changes frequently) |
| Storage, VNets, Subnets | Daily (less volatile) |
| Full account inventory | Weekly (catch long-tail resources) |
| New account onboarding | Manual trigger first, then scheduled |

---

## 7. Key Tables for Cloud Discovery

| Table | Label | Purpose |
|-------|-------|---------|
| `cmdb_ci_vm_instance` | VM Instance | AWS EC2 / Azure VM CI records |
| `cmdb_ci_cloud_service_account` | Cloud Service Account | AWS Account / Azure Subscription records |
| `cmdb_ci_cloud_storage_account` | Cloud Storage Account | S3 buckets / Azure Storage Accounts |
| `cmdb_ci_db_instance` | Database Instance | RDS / Azure SQL instances |
| `cmdb_ci_network` | Network | VPCs / VNets |
| `discovery_schedule` | Discovery Schedule | Cloud discovery schedule configuration |
| `cloud_account` | Cloud Account | AWS/Azure account/subscription records |
