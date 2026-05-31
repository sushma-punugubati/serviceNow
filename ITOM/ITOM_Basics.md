# ServiceNow ITOM — Fundamentals & Study Guide

> Sources: ServiceNow Docs | ServiceNow Community | ITOM Implementation Guide | Now Learning

---

## 1. What is ITOM?

**IT Operations Management (ITOM)** is ServiceNow's suite of capabilities for automatically discovering, mapping, monitoring, and managing IT infrastructure. ITOM answers the operational questions that ITSM depends on: What systems exist? How are they connected? Which services depend on what? What is happening right now?

> **Think of it as:** Giving IT a live, self-updating map of the entire infrastructure. Without ITOM, the CMDB is a photograph taken months ago. With ITOM, it is a live video feed — continuously updated as infrastructure changes.

**ITOM Core Capabilities:**

| Capability | What it does |
|-----------|-------------|
| **Discovery** | Scans the network and populates CMDB with CI records automatically |
| **Service Mapping** | Traces application dependencies to build a live service topology map |
| **Event Management** | Ingests monitoring alerts, correlates them, and creates actionable incidents |
| **Health Log Analytics** | Uses ML to detect anomalies in log and metric data (AIOps) |
| **Cloud Provisioning & Governance** | Manages cloud resource lifecycle through ServiceNow |

---

## 2. ServiceNow Discovery

### What is Discovery?

Discovery is the automated process of scanning IT infrastructure to find devices, servers, applications, and cloud resources — and populating the CMDB with accurate, current CI records.

**Without Discovery:** CIs are created manually and immediately start becoming stale.
**With Discovery:** CIs are created and updated automatically every time Discovery runs (typically daily).

### Discovery Architecture

```
ServiceNow Cloud
      |
   ECC Queue (communication channel)
      |
   MID Server (on-premises)
      |
   Internal Network (firewalls, switches, servers, databases)
```

### How Discovery Works — Step by Step

1. **Trigger:** Discovery Schedule fires (or manual trigger)
2. **Scan IP ranges:** Discovery sends SNMP/TCP probe to each IP in the configured range
3. **Device found:** If a device responds, Discovery identifies its type (server, network device, printer)
4. **Probe sent:** MID Server sends probes appropriate for the device type (SSH for Linux, WMI for Windows, SNMP for network devices)
5. **Raw data collected:** Probe returns raw data (OS version, running processes, installed software, network interfaces)
6. **Sensor processes data:** Sensors parse raw probe data into structured CI attribute values
7. **IRE runs:** Identification and Reconciliation Engine determines: is this CI new or existing? What data source wins for each attribute?
8. **CMDB updated:** CI record created or updated in CMDB

### Discovery Types

| Type | What it finds | Example |
|------|---------------|---------|
| **Horizontal Discovery** | Infrastructure components | Servers, network devices, storage arrays, printers |
| **Vertical Discovery** | Application stacks on top of infrastructure | Tomcat + Oracle DB + Apache HTTP running on a specific server |
| **Cloud Discovery** | Cloud-native resources | AWS EC2, S3, RDS; Azure VMs, Storage Accounts |
| **Pattern-based Discovery** | Complex application topology using reusable patterns | SAP, Oracle E-Business Suite, custom applications |

---

## 3. The MID Server

### What is the MID Server?

The **MID (Management, Instrumentation, and Discovery) Server** is a Java application installed in the customer's network that acts as a secure bridge between the ServiceNow cloud and internal infrastructure.

**Why it is needed:** ServiceNow runs in the cloud and cannot directly access servers behind corporate firewalls. The MID Server sits inside the network perimeter and executes probes, collects results, and sends them back to ServiceNow.

### MID Server Architecture

```
ServiceNow Cloud (SaaS)
        ↕ (HTTPS outbound from MID Server — no inbound firewall rules needed)
    MID Server (Java app, Windows or Linux)
        ↕ (internal network protocols — SSH, WMI, SNMP, JDBC)
    Target devices and servers
```

**Key point:** MID Server initiates the connection to ServiceNow — ServiceNow does not connect inbound to the MID Server. This means only outbound HTTPS (port 443) needs to be opened from the MID Server to ServiceNow, which most firewalls already allow.

### MID Server Roles

| Role | Purpose |
|------|---------|
| **Discovery** | Scans IP ranges and collects CI data |
| **Orchestration** | Executes remote scripts and commands on target systems |
| **IntegrationHub** | Runs integration actions that need access to internal systems |
| **Event Management** | Receives events from on-premises monitoring tools |
| **Import Sets** | Pulls data from internal databases via JDBC |

### MID Server Performance Tuning

Key configuration parameters:
- **Worker Threads:** Number of concurrent probes. Default 25; can be increased for large networks (75-150 for enterprise)
- **Maximum ECC Queue size:** Limits work queue depth to prevent MID Server overload
- **Probe timeout:** How long to wait for a response from a target device
- **Batch size:** How many IPs to probe in each batch

**Monitoring MID Server health:**
- Work Queue depth (high = MID Server is falling behind)
- Worker thread utilization
- Probe success/failure rate
- MID Server upgrade status vs. platform version

---

## 4. Discovery Probes, Sensors, and Patterns

### Probes
A **Probe** gathers raw data from a target system. Examples:
- `Linux - Hardware` probe: SSH to Linux server, runs `dmidecode`, `lscpu`, `free -m` to collect hardware data
- `Windows - WMI` probe: WMI query to Windows server for OS version, installed software, running services
- `SNMP - Interfaces` probe: SNMP query to network device for interface list and status

### Sensors
A **Sensor** processes raw probe output and writes structured data to CMDB tables. The sensor contains the parsing logic that transforms raw command output into CI attribute values.

**Together:** Probe collects raw data → Sensor writes structured CI records.

### Patterns
**Patterns** are reusable, multi-step discovery logic for complex applications and technologies. A pattern defines:
1. How to identify a running technology (e.g., "look for oracle.exe process")
2. How to collect detailed configuration data
3. How to map relationships between discovered components

Patterns are the modern replacement for probes/sensors — they are more powerful, easier to maintain, and support credential-less discovery.

### Classifiers and Identifiers (Legacy)
For non-pattern-based discovery:
- **Classifier:** Determines the CI class (e.g., "if port 22 is open and it responds to SSH, classify as Unix Server")
- **Identifier:** Determines the CI's unique identity (e.g., "use Serial Number + MAC Address as the key to find existing CI records")

---

## 5. IRE — Identification and Reconciliation Engine

### What is IRE?

The **IRE** is the engine that decides two things when Discovery finds a CI:
1. **Identification:** Is this a new CI, or does a record already exist for it in the CMDB?
2. **Reconciliation:** For each attribute, which data source is authoritative? (Discovery, SCCM, manual entry?)

### Identification Rules

Identification rules define the unique key for each CI class:

| CI Class | Recommended Key |
|----------|----------------|
| Physical servers | Serial Number + MAC Address |
| Virtual machines | VM UUID (from hypervisor) |
| Network devices | MAC Address + Serial Number |
| Applications | Application name + host CI |
| Cloud instances | Cloud provider instance ID |

**Why this matters:** Using IP address as the identifier causes duplicate CI records whenever a server changes its IP. Using a stable hardware identifier prevents duplicates.

### Reconciliation Rules

Reconciliation rules define which data source wins for each attribute:

| Attribute | Authoritative Source |
|-----------|---------------------|
| IP Address | Discovery (most current) |
| Installed Software | SCCM (more complete than Discovery) |
| Serial Number | Asset Management (from procurement) |
| Owner/Department | HR System (via import set) |
| Custom manual fields | Manual entry (do not overwrite) |

---

## 6. Event Management

### What is Event Management?

**Event Management** receives alerts from external monitoring tools (Splunk, Nagios, Dynatrace, SolarWinds, etc.) and processes them into actionable ServiceNow Events, which can then be correlated into Incidents.

### Event Flow

```
Monitoring Tool (Splunk, Nagios, etc.)
        ↓  (REST/webhook/connector)
ServiceNow Event Management (receives raw alert)
        ↓  (Alert Rules — filter, deduplicate, correlate)
Event Record created
        ↓  (CI Binding — match event to CMDB CI)
Alert Record with CI context
        ↓  (Alert Correlation Rules — group related alerts)
Correlated Alert Group
        ↓  (Incident Creation Rule)
ITSM Incident created (with CI, service, and alert context)
```

### Key Event Management Concepts

| Concept | Definition |
|---------|-----------|
| **Event** | Raw alert received from a monitoring tool |
| **Alert** | Processed event after deduplication — represents an ongoing condition |
| **CI Binding** | Matching an alert to its corresponding CI in the CMDB |
| **Alert Correlation** | Grouping related alerts into a single Alert Group |
| **Alert Rule** | Defines how incoming events are processed (severity mapping, deduplication) |
| **AIOps** | ML-based alert correlation that learns normal patterns and surfaces true anomalies |

### Event Management Integration Methods

| Method | Use Case |
|--------|---------|
| **REST API** | Most monitoring tools — push events to ServiceNow REST endpoint |
| **MID Server connector** | On-premises monitoring tools that cannot reach ServiceNow directly |
| **Email integration** | Legacy tools that send alert emails |
| **JDBC polling** | Tools that write alerts to a database table |

---

## 7. Key ITOM Tables Reference

| Table | Label | Purpose |
|-------|-------|---------|
| `cmdb_ci` | Configuration Item | All CI records |
| `cmdb_rel_ci` | CI Relationship | Relationships between CIs |
| `discovery_schedule` | Discovery Schedule | Scheduled Discovery runs |
| `ecc_queue` | ECC Queue | Communication between ServiceNow and MID Server |
| `mid_server` | MID Server | MID Server records |
| `em_event` | Event | Raw events from monitoring tools |
| `em_alert` | Alert | Processed, deduplicated alerts |
| `sa_service` | Business Service | Service Mapping service records |
| `cmdb_ci_service` | IT Service | IT Service CI |
| `svc_mapping_task` | Service Mapping Task | Service Mapping run results |
