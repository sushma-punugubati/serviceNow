# ServiceNow Service Mapping — Common Issues, Pitfalls & Troubleshooting

---

## Issue 1: Service Map Blank After Run — Entry Point Not Connecting

**Category:** Configuration / Entry Point
**Severity:** Critical — no map produced at all

**The Problem:**
Service Mapping is configured with an entry point URL. The mapping task runs and completes in under 1 minute, but the service map shows only the entry point CI with no dependencies discovered.

**Why It Happens:**
The entry point is connecting successfully to the CI but no patterns are matching the responding service. Or the entry point IP/URL resolves to a load balancer IP that is not in the CMDB — Service Mapping cannot start from a CI it cannot identify.

**Root Cause:**
Entry point CI not found in CMDB, or no pattern matches the application at the entry point.

**Fix / Prevention:**
- Verify the entry point URL resolves to a CI in CMDB — run a Discovery on the entry point IP first
- Check the Mapping Task log for "No pattern matched" errors
- Test credentials manually: SSH from the MID Server host to the entry point server and confirm the credentials work
- If using a URL entry point for a load-balanced application, ensure the load balancer CI is in CMDB and matches the resolved IP

---

## Issue 2: Service Map Missing Database Tier — Connection Pooling Issue

**Category:** Configuration / Pattern
**Severity:** High — incomplete service topology, wrong impact analysis

**The Problem:**
Service Mapping discovers the web tier and app tier but the Oracle database is never added to the map. The application definitely uses Oracle — it is critical to the service — but Service Mapping has no knowledge of it.

**Why It Happens:**
The application uses a JDBC connection pool initialized at startup. At Service Mapping run time, the connection pool is established but no individual connections show as active in `netstat` — all connections are sitting idle in the pool. Service Mapping traces active connections and sees nothing pointing to the database.

**Root Cause:**
Connection tracing requires active in-flight connections. Connection pools hold connections idle until needed — idle pool connections do not appear in `netstat` output the same way active queries do.

**Fix / Prevention:**
- Use Oracle-specific patterns that read the application's configuration files (web.xml, application.properties) to extract the JDBC URL — this finds the database regardless of connection activity
- Schedule Service Mapping during peak load times when the connection pool has active queries in flight
- Manually add the database CI relationship as a baseline, then use Service Mapping to validate and update it rather than discover from scratch
- Consider using Tag-Based Service Mapping for the database tier if tagging governance is in place

---

## Issue 3: Service Mapping Discovering Wrong CI — Same Hostname Different Environments

**Category:** Configuration / CMDB Data Quality
**Severity:** High — production service map incorrectly includes non-production CIs

**The Problem:**
The Production Online Banking service map includes CI records for dev/test servers. The topology map looks correct in structure, but the CIs in it are from non-production environments.

**Why It Happens:**
CMDB has duplicate CI records with the same hostname for different environments (prod-app-01 exists in both production and development domains). Service Mapping connects to the entry point IP address (which is production) but the IRE identification rule matches on hostname and finds the dev CI record first.

**Root Cause:**
Ambiguous CI identification when the same hostname exists across multiple environments or domains in CMDB.

**Fix / Prevention:**
- Implement Domain Separation: production and development CIs in separate domains — Service Mapping runs in the production domain and only matches production CIs
- Add environment as part of the IRE identification rule (hostname + environment = unique identifier)
- Enforce naming conventions that include environment prefix: `prod-app-01` vs. `dev-app-01` cannot be confused

---

## Issue 4: Service Mapping Credentials Failing After Password Rotation

**Category:** Operations / Credential Management
**Severity:** Medium-High — service maps stop updating silently

**The Problem:**
Service Mapping has been running successfully for months. After a quarterly security-mandated service account password rotation, service maps stop updating. No error notification is generated. The maps are stale but nobody notices until a P1 incident occurs and the map is wrong.

**Why It Happens:**
Service account passwords were rotated as part of a security policy but the ServiceNow Credential Store was not updated. Service Mapping silently fails to authenticate — the mapping task "completes" but produces no updates.

**Root Cause:**
Credential Store not updated after password rotation, no monitoring on Service Mapping task success/failure rate.

**Fix / Prevention:**
- Include Credential Store updates in the service account rotation runbook — this step must be part of the rotation SOP, not optional
- Add monitoring: if a Service Mapping task completes with 0 new relationships discovered (when historical runs always find updates), trigger an alert
- Set up a weekly "map freshness" check: if a service map's last successful update is more than 7 days ago, create a task for the ITOM team to investigate
- Test credential validity before rotation is complete: update Credential Store first, then rotate the service account password

---

## Issue 5: Tag-Based Mapping Not Working — Inconsistent Tags Across Cloud Resources

**Category:** Configuration / Tag Governance
**Severity:** High — cloud service topology incomplete or incorrect

**The Problem:**
Tag-Based Service Mapping was deployed for AWS infrastructure. Only 40% of EC2 instances are appearing in service maps. The rest are untagged or have inconsistently formatted tags (some have "Service" tag, others have "service", others have "ServiceName").

**Why It Happens:**
No tag governance policy was enforced when cloud resources were created. Tags were applied manually by individual engineers with no standard — different teams used different tag keys and different casing.

**Root Cause:**
Missing tag governance — no enforced standard for required tags or tag key naming.

**Fix / Prevention:**
- Enforce tags at the AWS account level using Service Control Policies (SCPs): required tags must be present before a resource can be created
- Create a tag normalization script that standardizes existing tags to the agreed format
- Add tag compliance reporting: weekly report of resources with missing or malformed required tags, sent to team owners
- Update the Service Mapping configuration to handle case-insensitive tag key matching as a short-term workaround

---

## Issue 6: Service Map Loop — Circular Dependency Discovered

**Category:** Configuration / Discovery Logic
**Severity:** Medium — Service Mapping run hangs or creates incorrect relationship loops

**The Problem:**
Service Mapping task starts but never completes — it runs for hours and eventually times out. Reviewing the in-progress map shows CI A → CI B → CI C → CI A (a circular dependency loop). Service Mapping keeps recursing indefinitely.

**Why It Happens:**
Some application architectures have actual circular dependencies (load balancer forwards to app server which calls an API on the same load balancer). Service Mapping follows all connections and does not inherently detect cycles.

**Root Cause:**
Circular network connections in the application architecture cause Service Mapping to loop indefinitely.

**Fix / Prevention:**
- Configure the Mapping Group's maximum depth limit — Service Mapping will stop after reaching the limit even if not all connections are traced
- Manually exclude specific CI-to-CI relationship types that are causing the loop
- Configure exclusion patterns: CI addresses within the same subnet as the entry point can be excluded from recursive tracing

---

## Issue 7: Service Maps Stale — Not Reflecting Recent Infrastructure Changes

**Category:** Operations / Schedule
**Severity:** Medium — topology maps used for incident response are outdated

**The Problem:**
An infrastructure team migrated the application database from Oracle 11g to Oracle 19c and moved it to a new server. Two months later, during a P1 incident, the Service Map still shows the old Oracle 11g server as the database tier — which was decommissioned 6 weeks ago.

**Why It Happens:**
Service Mapping runs on a weekly schedule. Changes to the infrastructure between runs are not reflected until the next scheduled run. If the next run has issues (credential failure, network change), it might not update for weeks.

**Root Cause:**
Service Mapping schedule too infrequent for the rate of infrastructure change; no process to trigger re-mapping after significant infrastructure changes.

**Fix / Prevention:**
- Integrate Service Mapping with Change Management: when a Normal Change modifies a CI that is in a service map, automatically trigger a Service Mapping re-run for that service after the change window closes
- Increase Service Mapping schedule frequency for critical services (nightly for Tier 1 services, weekly for Tier 2)
- Add a post-change verification step: confirming the service map is accurate is part of the change closure process

---

## Issue 8: Pattern Not Matching Custom Application

**Category:** Configuration / Patterns
**Severity:** Medium — one or more tiers not discovered

**The Problem:**
The organization has a custom-built Node.js application running on Linux servers. Service Mapping discovers the load balancer and nginx reverse proxy, but stops at the Node.js tier — no pattern exists for Node.js in the out-of-box ServiceNow content.

**Why It Happens:**
ServiceNow's out-of-box patterns cover standard enterprise technologies. Custom or less-common technologies require custom pattern development.

**Root Cause:**
No pattern available for the technology stack in use.

**Fix / Prevention:**
- Build a custom pattern for the Node.js application: define the trigger (process name = "node"), connection steps (read active TCP connections from `/proc/net/tcp`), attribute steps (read package.json for application version), relationship steps
- Use the Pattern Designer in ServiceNow Studio — no coding required for simple patterns; JavaScript available for complex logic
- Check the ServiceNow Share platform — community-contributed patterns are available for many technologies not covered by OOTB content
