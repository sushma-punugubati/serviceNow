# ServiceNow Service Mapping — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is Service Mapping and what problem does it solve?**
**A:** Service Mapping automatically traces and documents the technical dependencies of a business service — showing which applications, databases, middleware, and infrastructure components support each service.

**The problem it solves:** In most organizations, service topology is either undocumented or maintained in Visio diagrams and spreadsheets that are immediately stale after the first infrastructure change. When a P1 incident occurs, engineers spend 30-60 minutes just figuring out what depends on what before they can start resolving. Service Mapping makes the dependency graph live, automatic, and CMDB-backed — so impact is understood in seconds, not hours.

---

**Q2: What is an Entry Point in Service Mapping?**
**A:** An Entry Point is the starting CI for a Service Mapping run — the "front door" of the application. Service Mapping connects to the entry point and traces network connections outward to discover all downstream dependencies.

**Common entry point types:**
- **URL:** `https://app.company.com` — discovers from the public-facing URL, automatically finds the load balancer or web server behind it
- **IP:Port:** Direct IP address and port — useful for internal services not accessible by URL
- **CI-based:** Starts from a specific CMDB CI record
- **Host + Process:** Hostname and process name — starts from a known running process

**Choosing the right entry point:** For web applications, URL entry points are most natural. For backend services (batch processors, APIs), IP:Port or CI-based entry points work better.

---

**Q3: What is a Service Mapping Pattern?**
**A:** A Pattern defines how to discover and map a specific technology. It contains:
1. **Trigger:** The condition that activates the pattern (e.g., "process name contains oracle")
2. **Connection steps:** How to discover what this technology connects to
3. **Attribute steps:** How to collect configuration details (version, config file, listener port)
4. **Relationship steps:** What relationship type to create between this CI and its dependencies

ServiceNow ships with out-of-box patterns for Apache, IIS, Tomcat, JBoss, Oracle, MySQL, MSSQL, WebSphere, and many others. Custom patterns can be built for proprietary or specialized applications.

---

**Q4: What is Tag-Based Service Mapping and when would you choose it over connection-based?**
**A:** Tag-Based Service Mapping reads cloud resource tags (AWS tags, Azure tags, Kubernetes labels) to build service topology instead of tracing live network connections.

**Choose Tag-Based when:**
- Microservices / containers: applications are ephemeral, connections change constantly
- Auto-scaling environments: instances come and go faster than connection tracing can track
- Cloud-native apps where services communicate via API calls that are not persistent connections
- Teams already have a strong tagging governance practice

**Choose Connection-Based when:**
- Traditional 3-tier applications with stable, persistent connections
- On-premises applications where network connections are reliable indicators of dependency
- Applications where tag governance cannot be enforced

**In practice:** Many organizations use both — connection-based for on-premises and hybrid, tag-based for cloud-native.

---

**Q5: How does Service Mapping relate to the CMDB?**
**A:** Service Mapping is a CMDB *populator* — its output is CI records and CI relationship records in the CMDB.

**What Service Mapping creates:**
- CI records for each discovered application component (Tomcat instance, Oracle DB, Apache server)
- `cmdb_rel_ci` relationship records linking each CI to what it depends on
- Application Service record (`cmdb_ci_service`) as the top-level service entity
- Relationship linking the Application Service to the Business Service

**Why this matters:** The relationships Service Mapping creates are what power BSM Maps, Event Management CI binding, and Change impact analysis. Without Service Mapping, these relationships have to be entered manually and are immediately stale.

---

**Q6: What is the difference between Service Mapping and Horizontal Discovery?**
**A:**
| Horizontal Discovery | Service Mapping |
|---------------------|-----------------|
| Finds infrastructure components (servers, network devices) | Traces application dependencies on top of infrastructure |
| Result: individual CI records | Result: CI records + relationships forming service topology |
| Scope: IP ranges | Scope: entry point → connection tracing |
| Asks: "What CIs exist?" | Asks: "How are CIs connected into a service?" |

**Together:** Discovery populates the CMDB with CI records. Service Mapping populates the *relationships* between those CIs. Both are needed. Service Mapping cannot run effectively if CMDB does not have good CI records from Discovery.

---

**Q7: What credentials does Service Mapping need and how are they secured?**
**A:** Service Mapping needs credentials to access target systems and inspect their configuration:
- **SSH:** Linux/Unix servers — reads process lists, configuration files, network connections
- **WMI:** Windows servers — same purpose
- **Database connectors:** Oracle, MySQL, MSSQL — discovers database version, instances, schemas
- **Cloud APIs:** AWS IAM role, Azure Service Principal — for cloud tier mapping

**Security:** Credentials are stored in the ServiceNow Credential Store (encrypted), referenced by alias — never hardcoded in patterns. Service Mapping credentials should be **read-only service accounts** with access only to the specific commands needed — not admin accounts. Audit credential usage: review which patterns use which credentials and apply least privilege.

---

**Q8: How do you validate that a Service Map is accurate?**
**A:** Validation approaches:

1. **Cross-reference with architecture documents:** Compare the service map output against the application's architecture diagram or runbook — do all expected tiers appear?

2. **Network connection verification:** SSH to a discovered CI and run `netstat -an` — confirm the active connections shown in Service Mapping match what is actually running on the server

3. **Test with a planned outage:** In a test environment, stop one tier and confirm the service map accurately predicts which upstream components report errors

4. **Interview the application team:** Show the map to the developers/architects who own the application — they can immediately spot missing or incorrect dependencies

5. **Validate relationship types:** Confirm that the relationships created (Depends On, Runs On, Connects To) accurately describe the actual dependency nature

---

## PART 2: Implementation and Troubleshooting

**Q9: Service Mapping is configured but the map is blank after running. What do you investigate?**
**A:** Systematically work through the likely failure points:

1. **Check the Service Mapping Task log:** The task record has a log section showing what was attempted and why it stopped. "Connection refused," "Authentication failed," and "No pattern matched" each point to different issues.

2. **Validate the entry point:** Manually connect to the entry point IP:port — confirm it is actively listening. `telnet <IP> <port>` or `curl <URL>` from the MID Server's perspective.

3. **Check MID Server connectivity:** Confirm the MID Server can reach the entry point CI on the network. Firewall rules may block the MID Server from the application tier even if end-user traffic is allowed.

4. **Check credentials:** Verify SSH/WMI credentials are valid by using them manually from the MID Server host.

5. **Check pattern matching:** The entry point technology must match an available pattern. If the application is built on a technology not covered by a pattern, Service Mapping cannot proceed.

---

**Q10: The service map only shows 2 of the expected 5 tiers. What are the likely causes?**
**A:**
1. **No active connections at mapping time:** Service Mapping traces live connections. If the app server's connection pool to the database was established at startup and no new requests are in-flight at mapping time, the connection may not appear in `netstat` for Service Mapping to follow.

   **Fix:** Use patterns that read configuration files (e.g., read JDBC URL from web.xml) rather than relying on live connections.

2. **Missing credentials for deeper tiers:** Service Mapping may have SSH credentials for the app servers but not for the database servers — it stops where credentials fail.

3. **Pattern not matching:** The database or middleware version may not match any available pattern. Custom patterns may be needed.

4. **Depth limit reached:** Service Mapping has a configurable maximum depth. Increase the depth limit if the map is being truncated.

5. **Firewall blocking MID Server from deeper tiers:** MID Server can reach the web tier but is blocked from the database tier.

---

**Q11: What is a Mapping Group and why would you use it?**
**A:** A Mapping Group is a container that associates a set of patterns, a MID Server, and optionally a set of credentials for a specific mapping context.

**Use cases:**
- Different data center segments need different MID Servers (in different network zones)
- Different application types need different pattern sets (Java apps vs. .NET apps vs. databases)
- Different environments (Production vs. Non-Production) need different credential sets

**Example:** A Mapping Group for "Production Oracle Applications" might specify: use MID Server PROD-MID-01, use the Oracle Pattern set, use the PROD-ORACLE-READ credential. A separate Mapping Group for "Cloud Services" uses a different MID Server and the AWS/Azure pattern set.

---

**Q12: How does Service Mapping integrate with Change Management?**
**A:** Service Mapping enriches Change Management through CMDB relationships:

1. **Impact analysis on change requests:** When a Change Request is submitted for a CI, the affected business services are automatically shown by traversing the service map upstream from the CI — "this database change affects the Online Banking service and the Customer Portal service"

2. **CAB review context:** CAB members can view the BSM Map for the affected CI directly from the Change Request — no separate investigation needed

3. **Post-change validation:** After a change is implemented, re-run Service Mapping for the affected service to confirm the topology is still intact (and detect any unintended relationship changes)

4. **Risk scoring:** Change risk calculators can use service map data — a change to a CI that supports 5 business services is inherently higher risk than one that supports 0

---

## PART 3: Advanced Topics

**Q13: How would you approach Service Mapping for a cloud-native microservices application?**
**A:** Connection-based tracing does not work well for microservices — services communicate via APIs on ephemeral ports, use service meshes (Istio), and scale horizontally with containers that start and stop constantly.

**Approach for microservices:**
1. **Tag-Based Service Mapping:** Enforce a tagging standard across all microservices (service name, version, environment, team). Use these tags to build the service topology automatically.

2. **Service Mesh integration:** If the application uses Istio or AWS App Mesh, integrate those telemetry sources — they have a complete view of service-to-service communication.

3. **Kubernetes namespace mapping:** Use namespace labels to group microservices by application — all services in the `payment-service` namespace belong to the Payment Application.

4. **Hybrid approach:** Map the cloud infrastructure (load balancers, managed databases, message queues) with connection-based or tag-based discovery; map the microservice layer using the service mesh or registry.

---

**Q14: What is the relationship between Service Mapping and Event Management?**
**A:** Service Mapping provides the topology data that makes Event Management meaningful:

1. **Business service impact:** When an alert arrives for a CI, Event Management can trace the service map to identify which business services are impacted — the alert is not just "server X is down" but "server X is down and it supports the Online Banking service (affecting 40,000 users)"

2. **Alert grouping:** Correlation rules can group alerts that affect CIs in the same service topology — all alerts affecting the Payment Service are grouped together

3. **Probable cause:** AIOps can use the service topology to identify the root cause CI — when alerts fire across multiple tiers simultaneously, the CI at the bottom of the dependency chain (the database, for example) is more likely to be the root cause than the app tier

Without Service Mapping, Event Management alerts have CI context but no service context. With Service Mapping, alerts have full business impact context.

---

**Q15: What are the limitations of Service Mapping?**
**A:**
1. **Requires active connections:** Connection-tracing only finds dependencies that have live connections at mapping time — batch processes that run at night may not be discovered during daytime mapping runs

2. **Credential dependency:** Every tier that needs to be mapped requires valid credentials on the MID Server — missing or expired credentials silently stop the map at that tier

3. **Pattern coverage:** Technologies not covered by a pattern cannot be mapped — custom patterns must be developed for proprietary or unusual applications

4. **Ephemeral environments:** Containers and serverless resources come and go too quickly for traditional connection-based mapping — tag-based or service mesh approaches are needed

5. **Network complexity:** Environments with strict micro-segmentation firewalls may block Service Mapping's connection probes at certain tiers even with credentials

6. **Resource intensive:** Service Mapping runs consume MID Server resources — running too many concurrent mapping tasks can impact Discovery performance on the same MID Server
