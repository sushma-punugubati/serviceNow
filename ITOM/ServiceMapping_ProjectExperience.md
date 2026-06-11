# ServiceNow Service Mapping — Interview Notes
## "What did you do in Service Mapping?" + Challenges Faced

---

## 1. OVERVIEW — What is Service Mapping?

Service Mapping automatically discovers and visualizes the full technical dependency chain of a business service — tracing from the front-end entry point through every application layer, middleware, and database all the way to the infrastructure. It answers the question "what depends on what" in a live, continuously updated form backed by the CMDB.

In my work at Dell, Service Mapping is one of the most operationally critical ITOM capabilities we manage. When a P1 incident fires, the BSM Map for the affected service immediately shows the dependency chain — no manual investigation needed to understand scope. And before any change is made to infrastructure, we can see exactly which services are at risk.

Key work I have done:
- Service Mapping configuration for multi-tier enterprise applications (Java, .NET, Oracle, MSSQL)
- Entry point design for load-balanced and direct-access services
- Pattern configuration and custom pattern development for internal applications
- CSDM-aligned relationship mapping (ensuring Service Mapping creates CSDM-compliant relationships)
- BSM Map configuration for operational use
- Tag-based Service Mapping for AWS workloads
- Service Map freshness monitoring

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Service Mapping for Multi-Tier Applications

- Configured Service Mapping for **20+ business-critical services** at Dell — covering the full range from simple 2-tier internal tools to complex 5-tier enterprise applications
- Designed **Entry Points** for each service: URL-based entry points for web-facing applications, IP:Port entry points for internal APIs and backend services, CI-based entry points for batch processing applications that have no listening network port
- Configured **Mapping Groups** aligned to network segments — production services on PROD-MID-01, non-production on DEV-MID-01, cloud services on CLOUD-MID-01. Each Mapping Group has the appropriate credential set and pattern configuration for its scope
- Maintained the **Credential Store** for Service Mapping: SSH credentials for Linux app servers, WMI for Windows servers, database credentials for Oracle and MSSQL discovery, AWS IAM role references for cloud tier mapping

### 2.2 CSDM-Aligned Relationship Creation

- Worked to ensure Service Mapping creates **CSDM-compliant relationships** — the default relationship types created by some patterns did not align to the CSDM standard, which broke Service Mapping's integration with the higher-level Business Service views
- Updated pattern configuration to use CSDM relationship types: "Runs on" (application to server), "Depends on" (service to service), "Hosted on" (database to host server) — aligned to the CSDM framework we implemented for the broader CMDB
- Validated that discovered Application Service records sat at the correct CSDM layer and linked correctly to the Business Service records created by the architecture team

### 2.3 BSM Map Configuration for Operations

- Configured **BSM Maps** for the 10 highest-priority business services at Dell — these maps are displayed on the NOC dashboard and used by the major incident team during P1 incidents
- Set up **impact propagation rules** on the BSM Map: when a CI in the map is flagged as impacted (either by an alert binding or manual incident update), the impact propagates up the dependency chain and colors the affected business service red on the map
- Built a **service health dashboard** using BSM Map data: leadership can see at a glance which business services are green/amber/red without needing to look at raw alerts

### 2.4 Tag-Based Service Mapping for AWS

- Configured **Tag-Based Service Mapping** for the AWS workloads at Dell — the cloud teams had adopted a consistent tagging standard so this was feasible
- Worked with the AWS architecture team to define the required tags: `Application`, `Tier`, `Environment`, `Owner` — enforced via AWS Service Control Policy
- Configured the ServiceNow Tag-Based Mapping engine to read these tags and build service topology from them — cloud resources automatically placed into the correct application and tier in the service map without connection tracing

---

## 3. CHALLENGES FACED

### Challenge 1: Service Map Missing Database Tier — Connection Pool Issue

**Problem:** For one of our critical order management applications, Service Mapping consistently discovered the web tier and app tier but never found the Oracle database — even though the app clearly uses Oracle and Oracle is the most critical dependency. Impact analysis for this service was misleading: it suggested the service had no database dependency.

**Solution:** Investigated the mapping task log and confirmed the Oracle pattern was not matching any connections from the app server. SSH'd to the app server during a mapping run and ran `netstat -an | grep <oracle_ip>` — confirmed the connection pool was established (connections present) but in CLOSE_WAIT state rather than ESTABLISHED. The pattern was looking for ESTABLISHED connections only. Switched to using the Oracle pattern's configuration-file-reading mode: it reads the JDBC URL from the application's `datasource.xml` configuration file, which always contains the database connection string regardless of connection state. Database tier appeared in the map on the next run.

---

### Challenge 2: Service Maps Drifting After Infrastructure Changes

**Problem:** Several service maps became inaccurate over time as the infrastructure team made changes — servers upgraded, databases migrated, load balancers replaced — without triggering Service Mapping re-runs. During a P1 incident on the customer portal, the BSM Map showed a decommissioned server as a dependency, wasting 20 minutes of investigation time on a server that had not existed for 6 weeks.

**Solution:** Integrated Service Mapping with the Change Management process: any Normal Change that modifies a CI tagged as "Service Mapped" triggers an automatic Service Mapping re-run for all services that CI belongs to, scheduled for the day after the change implementation window. Also added a "map freshness" check: a scheduled PA report flags any service map that has not been successfully updated in more than 14 days. This catches cases where scheduled mapping runs silently fail due to credential issues or network changes.

---

### Challenge 3: Tag-Based Mapping Incomplete — Tag Governance Gaps

**Problem:** After deploying Tag-Based Service Mapping for AWS, only about 60% of EC2 instances appeared in service maps. The other 40% had inconsistent or missing tags — some had `Service` tag, some had `service` (lowercase), some had `ApplicationName`, some had nothing. The teams that owned ungoverned resources pushed back on adding tags ("we're too busy").

**Solution:** Two-pronged approach: First, got leadership support to add tag compliance to the team's quarterly OKRs — non-compliance was visible at the manager level. Second, wrote an AWS Lambda function that ran nightly and reported missing/malformed tags per team with the estimated cost of the untagged resources — nobody wanted to explain untagged $50K/month resources to their VP. Within 6 weeks, tag compliance went from 60% to 91%. For the remaining 9%, built a temporary workaround: manually created CMDB CI relationships for the most critical untagged resources so the service maps were accurate even without tags.

---

### Challenge 4: BSM Map Showing Circular Dependencies

**Problem:** One of our internal DevOps toolchain service maps showed a circular dependency loop: the CI/CD orchestrator called the artifact repository which called a notification service which called back to the CI/CD orchestrator. Service Mapping was recursing through the loop until it hit the depth limit, producing a map with hundreds of duplicate CI nodes.

**Solution:** Configured a pattern exclusion rule for the specific notification service → CI/CD orchestrator call path — this relationship existed in the architecture but was not a dependency (it was a callback notification, not a service dependency). Also reduced the maximum recursion depth for this mapping group from the default to 6 levels, which was sufficient to capture all real dependencies without following notification loops. Documented the circular call pattern in the service's architecture documentation to prevent future confusion.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **Entry Point** | Starting CI for a Service Mapping run — the front door of the application |
| **Connection Tracing** | Follows live network connections from the entry point outward to discover dependencies |
| **Pattern** | Defines how to discover and map a specific technology (Apache, Oracle, Tomcat, etc.) |
| **Tag-Based Mapping** | Builds topology from cloud resource tags instead of live connections — used for cloud-native/containers |
| **Mapping Group** | Associates patterns, MID Server, and credentials for a specific mapping context |
| **BSM Map** | Visual topology map of a Business Service showing all supporting CIs and their relationships |
| **CSDM Alignment** | Service Mapping must create relationships using CSDM-standard types for integration with higher-level service views |
| **Map Freshness** | How recently the service map was last successfully updated — stale maps are dangerous during incidents |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Service Mapping deployed for **20+ critical business services** — topology maps available for all Tier 1 services
- BSM Maps available to NOC within **60 seconds of P1 declaration** — no manual dependency investigation needed
- Change-caused incidents reduced: pre-change impact analysis using service maps identified hidden dependencies in **~35%** of reviewed changes
- Oracle database tier discovery issue resolved — **100%** of expected database dependencies now visible in maps for affected applications
- Tag-Based Mapping coverage: AWS workload service topology coverage improved from **60% to 91%** after tag governance program
- Map freshness monitoring: zero stale map incidents after automated freshness checks and change-triggered re-mapping
