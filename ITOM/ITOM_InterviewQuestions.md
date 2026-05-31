# ServiceNow ITOM — Interview Questions & Answers

---

## PART 1: Discovery and MID Server

**Q1: What is ITOM and what are its core capabilities?**
**A:** ITOM (IT Operations Management) provides automated infrastructure visibility through four main capabilities:
- **Discovery:** Scans the network and populates CMDB with CI records automatically
- **Service Mapping:** Traces application dependencies to build live service topology maps
- **Event Management:** Ingests monitoring alerts, correlates them, and creates actionable incidents
- **Health Log Analytics:** ML-based anomaly detection in log and metric data

Together, ITOM ensures the CMDB reflects reality and that IT teams have context — not just alerts — when things go wrong.

---

**Q2: What is the MID Server and why is it required?**
**A:** The MID (Management, Instrumentation, and Discovery) Server is a Java application installed inside the customer's network that acts as a secure bridge between ServiceNow (cloud) and on-premises infrastructure.

It is required because ServiceNow cannot directly access internal systems behind corporate firewalls. The MID Server initiates outbound HTTPS connections to ServiceNow — no inbound firewall rules needed — and executes probes against internal targets using SSH, WMI, SNMP, and JDBC.

A single MID Server can handle a few hundred to a few thousand devices depending on hardware and network complexity. Large enterprises use multiple MID Servers segmented by network zone.

---

**Q3: What is the difference between Horizontal and Vertical Discovery?**
**A:**
| Horizontal Discovery | Vertical Discovery |
|---------------------|--------------------|
| Finds infrastructure components | Finds application stacks on top of infrastructure |
| Discovers: servers, network devices, printers, storage | Discovers: databases, web servers, app servers, middleware |
| Result: Hardware and OS CIs in CMDB | Result: Application CI records with relationships to their host infrastructure |
| Example: "Find all Linux servers in 10.0.0.0/24" | Example: "Find Oracle DB running on server X and map its listener, instances, and schemas" |

Both types are needed for a complete CMDB: Horizontal gives you the foundation, Vertical gives you the application layer on top.

---

**Q4: What is the IRE and what problem does it solve?**
**A:** The IRE (Identification and Reconciliation Engine) solves two problems:

1. **Identification:** When Discovery finds a CI, is it new or does a record already exist? Without stable identification rules, every discovery run creates duplicate records. IRE uses configurable rules (serial number, VM UUID, hostname+domain) to match discovered CIs to existing records.

2. **Reconciliation:** When multiple data sources (Discovery, SCCM, manual entry) provide different values for the same CI attribute, which wins? IRE applies reconciliation rules to determine the authoritative source per attribute.

**Real impact:** Before IRE was tuned, a laptop changing its IP address would create a new CI record every time. After setting the identifier to Serial Number + MAC Address, the same CI record is updated regardless of IP changes.

---

**Q5: What is the difference between a Probe and a Sensor?**
**A:**
- **Probe:** Collects raw data from a target system — connects via SSH/WMI/SNMP and runs commands or queries. Returns raw output (command line text, WMI query results).
- **Sensor:** Processes the raw probe output and writes structured data to CMDB tables. Contains the parsing logic that transforms raw text into CI attribute values.

**Together:** Probe = data collector (like a scout). Sensor = data interpreter (like an analyst). Probe collects → Sensor writes.

**Modern alternative:** Discovery Patterns handle both functions in a single reusable, configurable unit — more powerful than probes/sensors for complex applications.

---

**Q6: What are Discovery Patterns and when do you use them?**
**A:** Discovery Patterns are reusable, multi-step logic units that define how to identify, collect data from, and build relationships for complex technologies. They combine what probes and sensors did separately into a single, maintainable unit.

**When to use patterns:**
- Complex applications with multi-tier relationships (SAP, Oracle EBS, custom applications)
- Technologies where simple probe/sensor logic is insufficient
- Credential-less discovery scenarios where patterns can use API calls instead of OS-level credentials

**Pattern components:**
1. **Entry point:** How to detect the technology is running (e.g., check for a specific process name)
2. **Steps:** Commands or API calls to collect configuration data
3. **Relationships:** How discovered components relate to each other and to the host CI

---

**Q7: What is CI Binding in Event Management?**
**A:** CI Binding is the process of matching an incoming alert (from Splunk, Nagios, etc.) to the correct CI record in the CMDB. Without CI binding, an alert saying "Server PROD-DB-01 is down" is just a string — with CI binding, it is linked to the correct cmdb_ci_server record, which shows what business services depend on that server and who owns it.

**CI Binding rules typically use:**
- Hostname → CI name match
- IP address → CI IP address match
- FQDN → CI FQDN match (fallback)

**Why it matters:** Unbound alerts have no business context. Bound alerts trigger CI-aware impact analysis, show the affected business service, and route to the correct team — all automatically.

---

**Q8: What is Alert Correlation in Event Management?**
**A:** Alert Correlation groups related alerts into a single Alert Group, preventing alert storms from creating hundreds of separate incidents for a single underlying issue.

**Example:** A network switch failure generates: 50 "server unreachable" alerts, 30 "database connection failed" alerts, and 20 "application timeout" alerts — all caused by the same root cause. Without correlation, 100 separate incidents are created. With correlation rules, all 100 alerts are grouped into one Alert Group, one incident is created, and the alert count in the incident provides context for the scope.

**Correlation methods:**
- **Rule-based:** Group alerts matching specific conditions (same CI, same time window, same alert type)
- **ML-based (AIOps):** Learns alert patterns over time and automatically groups alerts that historically occur together

---

**Q9: What is the difference between Event Management and Incident Management?**
**A:**
| Event Management | Incident Management |
|-----------------|---------------------|
| Receives raw alerts from monitoring tools | Manages the human response to service disruptions |
| Automated processing — no human required for filtering | Human-owned — analyst works the ticket |
| Goal: surface meaningful signals from alert noise | Goal: restore service and communicate with stakeholders |
| Lives in ITOM | Lives in ITSM |
| Feeds into Incident Management | Receives incidents from Event Management |

**The relationship:** Event Management is the automated intake and triage layer. When an Event Management alert group crosses a threshold (e.g., a critical CI is down for 5 minutes), it automatically creates an ITSM Incident for the human team to respond to.

---

**Q10: What is the ECC Queue?**
**A:** The **ECC (External Communications Channel) Queue** is the communication mechanism between ServiceNow and the MID Server. It is a database table (`ecc_queue`) that acts as a message queue:

- **Output records:** ServiceNow writes instructions for the MID Server (probe to run, target IP, credentials to use)
- **Input records:** MID Server writes results back (probe output, sensor data, status updates)

The MID Server polls the ECC Queue every few seconds for new output records, executes the probe, and writes results back as input records. ServiceNow processes input records and updates CMDB accordingly.

**Why ECC Queue monitoring matters:** If the ECC Queue has a large backlog of unprocessed records, Discovery is falling behind. Monitoring queue depth is a key MID Server health indicator.

---

## PART 2: Event Management and Operations

**Q11: How does ServiceNow integrate with Splunk for Event Management?**
**A:** Integration approaches:
1. **Splunk Add-on for ServiceNow:** Splunk app that sends alerts matching defined search queries to the ServiceNow Event Management REST API endpoint (`/api/global/em/jsonv2`)
2. **MID Server connector:** For on-premises Splunk instances that cannot reach ServiceNow directly — MID Server polls Splunk's REST API and forwards events
3. **Webhook:** Configure Splunk alert actions to POST to ServiceNow's Event Management REST endpoint on alert trigger

**Configuration in ServiceNow:**
- Define an Alert Source for Splunk
- Configure field mapping: Splunk `severity` → ServiceNow `severity`, Splunk `host` → ServiceNow `node` (for CI binding)
- Set up Alert Rules to deduplicate and process incoming events
- Configure CI Binding rules to match the `node` field to CMDB CI records

---

**Q12: How do you prevent alert storms in Event Management?**
**A:** Three layers of protection:

1. **Source filtering:** Work with the monitoring tool (Splunk, Nagios) to only send alerts above a severity threshold — do not send every informational message to ServiceNow

2. **Deduplication rules:** If the same alert fires for the same source multiple times within a time window, update the existing Alert record rather than creating new ones

3. **Correlation rules:** Group related alerts into Alert Groups — one CI failure generates many downstream alerts, but all should be one Alert Group, one Incident

**Additional:** Alert suppression during maintenance windows — when a CI has an active Change Request, incoming alerts for that CI can be suppressed or tagged as maintenance-related rather than creating noise incidents.

---

**Q13: What is AIOps in the context of ServiceNow ITOM?**
**A:** AIOps (Artificial Intelligence for IT Operations) in ServiceNow refers to ML-based capabilities within Event Management and Health Log Analytics that:

1. **Alert correlation:** Learns patterns of alerts that occur together and automatically groups them (without needing manual correlation rules)
2. **Anomaly detection:** Detects unusual patterns in metric and log data before they become outages
3. **Probable cause identification:** When multiple alerts fire, identifies the most likely root cause CI rather than treating all alerts equally
4. **Noise reduction:** Filters out alerts that are informational or historically non-actionable

**Practical impact:** An environment with 10,000 daily alerts might generate 50 truly actionable incidents with well-tuned AIOps, vs. 500 incidents with only rule-based filtering.

---

**Q14: How does ITOM integrate with ITSM?**
**A:** ITOM and ITSM are tightly integrated:

1. **CMDB context in ITSM:** When an incident is created, the Configuration Item field links to the CMDB CI — ITSM agents see what the CI is, who owns it, what services depend on it, and recent changes made to it

2. **Event Management → Incident creation:** Event Management automatically creates ITSM Incidents when alert thresholds are breached — the incident is pre-populated with CI context, alert details, and business service impact

3. **Change Management → Discovery suppression:** When a Change Request is in "Implement" state, alerts for the affected CI can be suppressed or tagged as change-related to prevent noise incidents

4. **Problem Management ← CMDB:** Problem investigations use CMDB relationship maps to trace root cause across CI dependencies

---

## PART 3: Advanced and Scenario-Based

**Q15: You are setting up Discovery for the first time. What are the first 5 steps you take?**
**A:**
1. **Install and configure MID Server:** Deploy MID Server in each network segment that needs to be discovered. Validate connectivity to ServiceNow (MID Server status = "Up") and to target networks.

2. **Define IP ranges:** Create Discovery IP ranges covering the subnets to be scanned. Start with a small test range (one /24) before running against the full environment.

3. **Configure credentials:** Add SSH credentials (Linux), WMI credentials (Windows), SNMP community strings (network devices) to the MID Server credential store.

4. **Create Discovery Schedule:** Define a Discovery Schedule with the IP ranges, MID Server, and run frequency (start with manual trigger for testing, then set a nightly schedule).

5. **Run and validate:** Trigger a manual Discovery run. Review: ECC Queue for errors, Discovery Status for success/failure counts, CMDB for newly created/updated CI records. Check IRE logs for duplicate CI issues.

---

**Q16: A Discovery run completes but many CI records are missing expected attributes. What do you investigate?**
**A:**
1. **Check probe success:** In the Discovery Status record, review "Probe results" — look for probes that completed vs. failed. Missing attributes often trace to a probe that returned no data.
2. **Check credentials:** If WMI or SSH probes are failing, the credential may be wrong, expired, or the target service (WinRM, SSH) may not be running.
3. **Check sensor errors:** In the ECC Queue, look for input records with errors — sensor parsing failures result in raw data that is never written to CMDB.
4. **Check firewall rules:** Discovery may be reaching devices (PING succeeds) but specific protocols are blocked (WMI on port 135/445 blocked by Windows Firewall, SNMP on UDP 161 blocked).
5. **Check CI class:** If attributes are missing for a specific CI class, the sensor for that class may not be configured to populate those fields.

---

**Q17: What is the difference between Pattern-based Discovery and Credential-less Discovery?**
**A:**
- **Pattern-based Discovery:** Uses structured, multi-step patterns instead of individual probe/sensor pairs. The pattern defines entry points, connection steps, and relationship mapping. Can use credentials OR be credential-less depending on the target.

- **Credential-less Discovery:** Discovers resources without storing passwords or keys on the MID Server. Instead uses API-based authentication:
  - AWS: IAM Role assumed by the MID Server — no stored credentials
  - Azure: Service Principal / Managed Identity
  - vSphere: API token with read permissions

**Why credential-less matters:** Eliminating stored credentials from the MID Server reduces the attack surface significantly. If the MID Server is compromised, there are no credentials to steal.

---

**Q18: How does ITOM Discovery handle multi-tenant environments?**
**A:** In multi-tenant scenarios (e.g., an MSP managing multiple customers, or an enterprise with separate subsidiaries), CI records must be isolated by tenant. Approaches:

1. **Domain Separation:** Each tenant's CIs are created in their own Domain. Discovery Schedules are domain-aware — CIs discovered in a domain-specific schedule are associated with that domain.

2. **Company field on CI records:** Each CI record has a Company field. Discovery can be configured to set the Company field based on IP range or other attributes — "IPs in 10.1.x.x belong to Company A."

3. **Role-based cloud discovery:** For AWS/Azure, each tenant has their own cloud account. ServiceNow uses separate IAM roles per account to discover each tenant's resources independently, ensuring CIs from Account A never mix with Account B.

---

**Q19: What is a BSM Map?**
**A:** A **BSM (Business Service Management) Map** is a visual representation of a Business Service and all the CIs that support it, showing the dependency relationships as a connected graph.

In ServiceNow, BSM Maps are generated from the CMDB relationship data populated by Service Mapping or manual CI relationships. They show:
- The Business Service at the top
- Technical services and applications below it
- Infrastructure (servers, databases, network) at the bottom
- Relationship types between each layer

**Practical use:** During an incident, the BSM Map for the affected service immediately shows every CI in the dependency chain — which server is down, what applications run on it, what business service is affected, and who the business owner is.

---

**Q20: How do you handle Discovery credential security on the MID Server?**
**A:** MID Server credentials (SSH, WMI, SNMP) are stored in the ServiceNow Credential Store (not on the MID Server filesystem) and transmitted securely. Best practices:

1. **Use the Credential Store:** Never hardcode credentials in probes or patterns — always reference the ServiceNow Credential Store
2. **Principle of least privilege:** Discovery credentials should have read-only access to the minimum required information — no admin rights needed for most discovery scenarios
3. **Credential rotation:** Rotate discovery credentials on the same schedule as service account passwords; update the Credential Store record and restart the Discovery Schedule
4. **Credential-less where possible:** Use IAM roles (AWS), Service Principals (Azure), or API tokens rather than username/password for cloud and modern infrastructure
5. **Separate credentials per environment:** DEV, TEST, and PROD should have separate credential sets so a credential leak in DEV does not affect PROD
