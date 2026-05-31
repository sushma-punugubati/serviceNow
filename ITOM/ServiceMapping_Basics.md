# ServiceNow Service Mapping — Fundamentals & Study Guide

> Sources: ServiceNow Docs | ServiceNow Community | ITOM Service Mapping Implementation Guide

---

## 1. What is Service Mapping?

**Service Mapping** is a ServiceNow ITOM capability that automatically traces and visualizes the technical dependencies of a business service — showing exactly which applications, middleware, databases, and infrastructure components support that service, and how they are connected.

> **Think of it as:** An X-ray of your IT services. When "Online Banking" is slow, Service Mapping shows you that Online Banking depends on → Apache Load Balancer → 3 Tomcat App Servers → Oracle RAC Database → 2 Physical Servers in Data Center East. You know immediately what to investigate and what to protect during a change.

**Why Service Mapping matters:**
- **Incident response:** Know what CIs are involved in a service outage without manual investigation
- **Change management:** Before changing any CI, see every service that depends on it
- **CMDB accuracy:** Relationships discovered by Service Mapping are more reliable than manually entered ones
- **Event Management:** Bind alerts to their business service impact automatically

**Service Mapping vs. manual relationship creation:**
| Service Mapping | Manual Relationships |
|----------------|---------------------|
| Automated — discovers actual running connections | Manual — someone has to know and enter it |
| Self-healing — re-runs discover changes | Static — immediately becomes stale |
| Scales to thousands of services | Practical for only a few dozen |
| Requires MID Server and credentials | No discovery infrastructure needed |

---

## 2. Service Mapping Architecture

### Key Concepts

```
Business Service (top-level — what the business cares about)
        ↓
Application Service (e.g., "Online Banking Application")
        ↓
Technical Services (e.g., "Load Balancer Cluster", "App Server Farm", "Database Tier")
        ↓
Infrastructure CIs (servers, databases, network devices)
```

Service Mapping populates the relationships between these layers automatically by tracing network connections from an entry point.

### Entry Points

An **Entry Point** is the starting CI for a Service Mapping run — the front door of the application. Service Mapping starts at the entry point and follows connections outward.

**Common entry point types:**
| Entry Point Type | Example |
|-----------------|---------|
| **URL** | `https://banking.company.com` — discovers from the public-facing URL |
| **IP:Port** | `10.1.2.3:443` — discovers from a specific IP and port |
| **CI** | A specific CMDB CI record — starts mapping from that CI |
| **Host and Process** | Hostname + Process name — starts from a known process on a server |

### Mapping Process

1. **Entry point identified** → Service Mapping connects to the entry point (load balancer, web server, etc.)
2. **Connection tracing** → Examines active network connections from the entry point CI
3. **Pattern matching** → For each discovered connection, Service Mapping applies patterns to identify what technology is running (Apache, Tomcat, Oracle, etc.)
4. **Relationship creation** → Creates CI records and relationship records in CMDB for each discovered component
5. **Recurse outward** → Follows connections from each newly discovered CI until no new connections are found (or depth limit is reached)
6. **Map complete** → Full topology map stored in CMDB as CI relationships

---

## 3. Service Mapping Patterns

A **Pattern** in Service Mapping defines how to discover a specific technology:
- **Entry Point Pattern:** How to identify and classify the entry point CI
- **Connection Pattern:** How to trace connections from a known CI to find downstream dependencies
- **Application Pattern:** How to discover the configuration of a specific application type (Apache, Tomcat, Oracle, MSSQL, etc.)

### Pattern Components

| Component | Purpose |
|-----------|---------|
| **Trigger** | Condition that activates the pattern (e.g., process name = "httpd") |
| **Connection Steps** | Commands or queries to discover connections |
| **Attribute Steps** | Commands to collect CI attributes (version, config file location) |
| **Relationship Steps** | Defines what relationship type to create between connected CIs |

### Out-of-Box Patterns

ServiceNow ships with patterns for common technologies:
- Web servers: Apache, IIS, Nginx
- Application servers: Tomcat, JBoss, WebSphere, WebLogic
- Databases: Oracle, MySQL, MSSQL, PostgreSQL, MongoDB
- Middleware: MQ Series, ActiveMQ, RabbitMQ
- Cloud: AWS EC2 patterns, Azure VM patterns
- Containers: Kubernetes, Docker

---

## 4. Tag-Based Service Mapping

**Tag-Based Service Mapping** is an alternative to connection-tracing for cloud and containerized environments where traditional network connection tracing is difficult or unreliable.

### How Tag-Based Mapping Works

Instead of tracing network connections, Tag-Based Service Mapping reads **tags** applied to cloud resources (AWS tags, Azure tags, Kubernetes labels) and uses them to:
1. Identify which cloud resources belong to which application/service
2. Build service topology from tag relationships

**Example AWS tags:**
```
Service:   online-banking
Tier:      database
Env:       production
Owner:     fintech-team
```

ServiceNow reads these tags and automatically places the tagged resource in the correct position in the service topology.

### When to Use Tag-Based Mapping

- **Microservices and containers:** Traditional connection tracing does not work well in ephemeral container environments
- **Cloud-native applications:** Resources come and go too quickly for connection-tracing to be reliable
- **Auto-scaling groups:** Instances spin up and down; tags are the consistent identifier
- **Serverless:** Lambda functions have no persistent connections to trace

### Tag-Based vs. Connection-Based Comparison

| Aspect | Connection-Based | Tag-Based |
|--------|-----------------|-----------|
| **Data source** | Live network connections | Cloud resource tags |
| **Best for** | Traditional on-premises apps | Cloud-native, containers |
| **Dependency** | Active connections at mapping time | Tags must be applied consistently |
| **Maintenance** | Auto-updates when connections change | Requires tag governance |
| **Accuracy** | High for stable connections | High if tags are well-managed |

---

## 5. Mapping Groups

A **Mapping Group** is a configuration that defines which patterns to apply and which MID Server to use for a specific entry point or set of entry points.

Mapping Groups allow different application types to use different discovery approaches:
- Traditional 3-tier apps → connection-tracing patterns
- Cloud-native apps → tag-based patterns
- Legacy apps with no live connections → manual relationship entry + validation

---

## 6. Service Mapping and CMDB — What Gets Created

When Service Mapping runs successfully, it creates or updates:

1. **CI records:** For each discovered component (Tomcat instance, Oracle database, Apache server)
2. **CI Relationship records:** In `cmdb_rel_ci` — linking each discovered CI to what it depends on
3. **Application Service record:** The top-level service record in `cmdb_ci_service`
4. **Business Service relationship:** Links the Application Service to the Business Service CI

These relationships power:
- **BSM Maps:** Visual topology maps for operations
- **Impact analysis:** "If this server goes down, what breaks?"
- **Event Management CI binding:** Map an alert to not just the CI but the business service it affects

---

## 7. Service Mapping Credentials

Service Mapping requires credentials to:
- SSH into Linux servers (to inspect process list and configuration)
- WMI into Windows servers
- Connect to databases (to discover DB version, instances, schemas)
- Access cloud APIs (AWS/Azure) for cloud-tier mapping

**Credentials are stored in the ServiceNow Credential Store** — Service Mapping references them by credential alias, never hardcodes them.

**Least privilege principle:** Service Mapping credentials should be read-only service accounts with access only to the specific commands needed for discovery — not admin accounts.

---

## 8. Key Tables and Fields

| Table | Label | Purpose |
|-------|-------|---------|
| `sa_service` | Service | Business Service records |
| `cmdb_ci_service` | IT Service | Application Service CIs |
| `svc_mapping_task` | Service Mapping Task | Mapping run records and results |
| `cmdb_rel_ci` | CI Relationship | Relationships discovered by Service Mapping |
| `sa_pattern` | Pattern | Service Mapping pattern definitions |
| `sa_mapping_group` | Mapping Group | Grouping of patterns and MID Server |
| `sa_entry_point` | Entry Point | Starting points for Service Mapping |

---

## 9. Service Mapping vs. Discovery — When to Use Which

| Scenario | Use Discovery | Use Service Mapping |
|----------|--------------|---------------------|
| "What servers exist in this IP range?" | YES | No |
| "What applications run on this server?" | Discovery (vertical) | Both work |
| "What is the full dependency chain for the Payment service?" | No | YES |
| "Build a live topology map for this business service?" | No | YES |
| "Keep CMDB CI count accurate?" | YES | Supplementary |
| "Know what breaks if this DB server fails?" | No | YES |

**In practice:** Discovery populates the CMDB with individual CI records. Service Mapping populates the *relationships* between those CIs that form the service topology. Both are needed for a fully accurate CMDB.
