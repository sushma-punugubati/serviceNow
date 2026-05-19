# ServiceNow SecOps — Fundamentals & Study Guide

> Sources: [Reco.ai SecOps Guide](https://www.reco.ai/hub/servicenow-security-operations-secops) | [SecOps Community](https://www.servicenow.com/community/secops-articles/security-operations-secops-knowledge-amp-troubleshooting/ta-p/2757839)

---

## 1. What is ServiceNow SecOps?

**ServiceNow Security Operations (SecOps)** connects your existing security tools to prioritize and respond to vulnerabilities and security incidents faster by integrating security responses with IT operations workflows.

> **Think of it as:** The bridge between your security tools (scanners, SIEM, firewalls) and the IT teams who actually fix things. Instead of security teams exporting CSV files and emailing IT, everything flows automatically — the right person gets the right task with the right context.

**Core value:**
- **Speed:** 40% faster mean-time-to-resolve (MTTR) documented in customer cases
- **Context:** Every security alert enriched with CMDB data (who owns the affected system? what business service does it support?)
- **Automation:** Routine response steps automated via SOAR workflows

---

## 2. SecOps Core Modules

| Module | Abbreviation | What It Does |
|--------|-------------|-------------|
| **Security Incident Response** | SIR | Manages full lifecycle of security incidents |
| **Vulnerability Response** | VR | Identifies, prioritizes, and remediates vulnerabilities |
| **Threat Intelligence** | TI | Enriches incidents with external threat data |
| **Security Orchestration, Automation & Response** | SOAR | Automates response workflows (separate plugin) |
| **Configuration Compliance** | CC | Verifies system configurations against security policies |

---

## 3. Module 1: Security Incident Response (SIR)

### What Is a Security Incident?
A security incident is any unauthorized access, breach, malware infection, phishing attack, or suspicious activity that threatens the organization's information assets.

> **Key difference from ITSM Incident:** An ITSM Incident = service disruption (something broke). A Security Incident = potential breach or attack (something malicious).

### SIR Lifecycle

```
Detection → Triage → Analysis → Containment → Eradication → Recovery → Post-Incident Review
```

| Stage | What Happens |
|-------|-------------|
| **Detection** | Alert from SIEM, scanner, user report, or threat intel feed |
| **Triage** | Is this a real incident or false positive? Assign priority |
| **Analysis** | Investigate scope — what was affected, how did attacker get in? |
| **Containment** | Stop the spread — isolate affected systems |
| **Eradication** | Remove malware, close attack vector, patch vulnerability |
| **Recovery** | Restore systems to normal operation |
| **Post-Incident Review** | Document lessons learned, update runbooks |

### SIR Integrations
- **SIEM** (Splunk, QRadar, Microsoft Sentinel): Auto-creates security incidents from alerts
- **Firewalls** (Palo Alto, Cisco): Provides network context for incidents
- **Endpoint Detection** (CrowdStrike, Carbon Black): Endpoint threat data
- **Email Security** (Proofpoint, Mimecast): Phishing incident data

---

## 4. Module 2: Vulnerability Response (VR)

### What Is Vulnerability Response?
VR connects vulnerability scanner results to the CMDB to prioritize and assign remediation tasks based on **business impact** — not just CVE severity score.

### Why This Matters
A scanner might find 10,000 vulnerabilities. Without context, where do you start? With VR:
- CVE-2024-XXXX on a test server (owned by Dev, no business impact) = LOW priority
- Same CVE on the server running "Online Banking" (HIGH business impact) = CRITICAL priority

### VR Workflow

```
Scanner runs → Vulnerability items imported → 
CMDB match (which CI is affected?) → Business impact scored →
Remediation task created → Assigned to IT team → Tracked to closure
```

### VR Integrations
- **Tenable** (Nessus): Most common scanner integration
- **Qualys**: Enterprise vulnerability scanner
- **Rapid7 InsightVM**: Vulnerability management platform
- **National Vulnerability Database (NVD)**: CVSS scores pulled automatically
- **Container Vulnerability Response**: Specific handling for container/Kubernetes vulns

### Key Concepts in VR

| Concept | Definition |
|---------|-----------|
| **Vulnerability Item** | Single instance of a vulnerability on a specific CI |
| **Remediation Task** | Work item created and assigned to IT team to fix the vulnerability |
| **Affected Item** | The CI that has the vulnerability |
| **CVSS Score** | Common Vulnerability Scoring System — base severity (0-10) |
| **Business Criticality** | How important the affected CI/service is to the business |
| **Priority Score** | Combines CVSS + business criticality to decide what to fix first |
| **Exception** | Approved decision to accept a vulnerability (with documentation) |
| **False Positive** | Vulnerability flagged by scanner that doesn't actually exist |

---

## 5. Module 3: Threat Intelligence (TI)

**Threat Intelligence** enriches security incidents with external data about known threats:

- **IOCs (Indicators of Compromise):** IP addresses, domains, file hashes known to be malicious
- **Threat Feeds:** External sources publishing known attack indicators
- **Automatic correlation:** When an incident is created, TI automatically links known IOCs

**Example:** A firewall blocks traffic from IP 185.234.x.x. Threat Intelligence confirms this IP is associated with a known ransomware command-and-control server. The security incident is immediately escalated.

---

## 6. SOAR — Security Orchestration, Automation & Response

**SOAR** extends SecOps to automate response workflows that interact with systems outside ServiceNow.

> **Note:** SOAR is a **separate paid plugin** — it is NOT included in the base SecOps suite.

**What SOAR can do:**
- Automatically isolate a compromised endpoint (call CrowdStrike API)
- Block a malicious IP in the firewall (call Palo Alto API)
- Disable a compromised user account (call Active Directory API)
- Pull system logs from affected servers (SSH commands)
- Send notifications to Slack/Teams channels

**Activity Packs (standard):**
- Security Incident Response Orchestration
- Threat Intelligence Orchestration
- Vulnerability Response Orchestration

---

## 7. Data Enrichment Tables

When a security incident is created, these 4 standard tables capture contextual data:

| Table | Data It Holds |
|-------|-------------|
| **Firewall** | Network traffic data from firewall logs |
| **Malware** | Malware detection details (file hash, AV signature) |
| **Network Statistics** | Network performance and connectivity data |
| **Running Processes** | Active processes on the affected endpoint |

**How enrichment works:**
```
External System (e.g., endpoint scanner)
  → SOAR workflow retrieves data (JSON/XML)
  → Data Enrichment Map transforms data
  → Enrichment table record created
  → Linked to original security incident
  → Analyst has full context without leaving ServiceNow
```

---

## 8. CMDB's Critical Role in SecOps

The CMDB is the secret weapon in SecOps. Without it, security teams know **WHAT** is vulnerable but not **WHY IT MATTERS**.

With CMDB integration:
- Vulnerability found on `server-prod-001` → CMDB says it's the primary DB for "Online Banking"
- Online Banking = $2B revenue per day → immediate CRITICAL priority
- CMDB says owner is "Infrastructure Team, John Smith" → task auto-assigned
- CMDB shows patch window schedule → remediation planned during maintenance window

---

## 9. Key Tables in SecOps

| Table Name | Label | Purpose |
|-----------|-------|---------|
| `sn_si_incident` | Security Incident | Security incident records (SIR) |
| `sn_vul_vulnerable_item` | Vulnerable Item | Single vulnerability on a specific CI (VR) |
| `sn_vul_entry` | Vulnerability Entry | Vulnerability definition record |
| `sn_vul_remediation_task` | Remediation Task | Work item to fix vulnerability |
| `sn_ti_ioc` | Indicator of Compromise | IOC records from threat feeds |
| `sn_ti_threat_group` | Threat Group | Known threat actor records |
| `sn_si_task` | Security Incident Task | Sub-tasks within a security incident |
| `sn_si_obs_firewall` | Firewall Observation | Enrichment: firewall data |
| `sn_si_obs_malware` | Malware Observation | Enrichment: malware detection data |
| `sn_si_obs_netstat` | Network Statistics | Enrichment: network data |
| `sn_si_obs_process` | Running Process | Enrichment: process data |

---

## 10. Workflow Triggers in SecOps

**Workflow Triggers** monitor specific tables and automatically fire workflows when conditions are met:

- Monitor security incident table for new HIGH-priority incidents → auto-assign to senior analyst
- Monitor vulnerable item table for CRITICAL CVEs → immediately create remediation task
- Monitor for incidents with IOC matches → escalate to threat hunting team

**Key feature:** Triggers can fire multiple times on the same record (unlike standard workflow conditions).

---

## 11. Key Roles

| Role | Responsibilities |
|------|----------------|
| **Security Analyst (L1)** | Triage incoming alerts, basic investigation, escalation |
| **Security Analyst (L2)** | Deep investigation, containment decisions |
| **Incident Commander** | Coordinates response for major incidents |
| **Vulnerability Manager** | Oversees VR program, SLA compliance |
| **Threat Intelligence Analyst** | Manages threat feeds, IOC analysis |
| **SecOps Admin** | Configures integrations, workflow triggers, escalation rules |
| **IT Remediation Team** | Executes patch deployment and vulnerability fixes |

---

## 12. Key Metrics

| Metric | Definition | Target |
|--------|-----------|--------|
| **MTTD** | Mean Time to Detect — how long before incident is identified | < 1 hour |
| **MTTR** | Mean Time to Respond/Resolve | Varies by severity |
| **Posture Score** | Overall security standing (% of risks mitigated) | Improve over time |
| **Vulnerability SLA** | % of vulnerabilities remediated within defined timeframes | > 90% |
| **False Positive Rate** | % of alerts that are not real threats | Minimize |
| **Backlog Reduction** | Reducing open vulnerability count over time | Downward trend |

---

## 13. SecOps Maturity Phases

| Phase | Focus |
|-------|-------|
| **Foundational** | Basic SIR setup, manual processes, SIEM integration |
| **Crawl** | Standardize incident response, VR from one scanner |
| **Walk** | Automate enrichment, expand scanner coverage, TI integration |
| **Run** | SOAR automation for routine responses, metrics dashboards |
| **Fly** | AI-driven threat detection, predictive risk, full automation |

---

## 14. Glossary

| Term | Simple Explanation |
|------|-------------------|
| **SIR** | Security Incident Response — manages security breach lifecycle |
| **VR** | Vulnerability Response — finds and fixes security weaknesses |
| **TI** | Threat Intelligence — external data about known threats |
| **SOAR** | Security Orchestration Automation & Response — automates response |
| **IOC** | Indicator of Compromise — known malicious IP, domain, or file hash |
| **CVE** | Common Vulnerabilities and Exposures — standardized vulnerability ID |
| **CVSS** | Common Vulnerability Scoring System — severity score 0-10 |
| **SIEM** | Security Information & Event Management — alert aggregation system |
| **Vulnerable Item** | One vulnerability on one specific CI |
| **Remediation Task** | Work item assigned to IT to fix a vulnerability |
| **MTTD** | Mean Time to Detect |
| **MTTR** | Mean Time to Respond/Resolve |
| **False Positive** | Alert that turned out not to be a real threat |
| **Threat Feed** | External source publishing known malicious indicators |
| **MID Server** | Communication bridge (also used by SecOps for data collection) |
| **Enrichment** | Adding context (firewall logs, process data) to an incident |
