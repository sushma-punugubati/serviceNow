# ServiceNow SecOps — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is ServiceNow SecOps and what problem does it solve?**
**A:** SecOps connects existing security tools to IT operations workflows, enabling faster vulnerability remediation and security incident response. 

**Problem it solves:** Security teams find vulnerabilities and incidents but must manually hand off to IT via email/spreadsheets. There's no visibility, no SLAs, no automation. SecOps provides a unified platform where findings flow automatically to the right IT team with full context, priority, and tracking.

---

**Q2: What are the three core modules of SecOps?**
**A:**
1. **SIR (Security Incident Response):** Manages the lifecycle of security incidents from detection to post-incident review
2. **VR (Vulnerability Response):** Identifies, prioritizes, and remediates vulnerabilities using scanner data + CMDB context
3. **TI (Threat Intelligence):** Enriches incidents with external threat data (IOCs, threat actor profiles)

Optional: **SOAR** (Security Orchestration, Automation & Response) — automates response actions outside ServiceNow (separate paid plugin).

---

**Q3: What is the difference between a Security Incident (SIR) and an ITSM Incident?**
**A:**
| Aspect | SIR Security Incident | ITSM Incident |
|--------|----------------------|---------------|
| **What is it?** | Potential or confirmed security breach/attack | Service disruption or failure |
| **Trigger** | SIEM alert, phishing report, scanner finding | User reports system is down |
| **Table** | `sn_si_incident` | `incident` |
| **Response** | Contain, eradicate, recover | Restore service |
| **Focus** | Who did it? How? What was exposed? | What broke? How to fix? |
| **Urgency driver** | Business impact + threat severity | Service impact + user count |

---

**Q4: What is a Vulnerable Item?**
**A:** A Vulnerable Item is a record representing one specific vulnerability on one specific CI. 

Example: CVE-2024-12345 found on server `prod-db-01` = one Vulnerable Item record.

The same CVE on `prod-web-01` = a DIFFERENT Vulnerable Item record.

Each Vulnerable Item can have a Remediation Task assigned to the team responsible for that CI.

---

**Q5: Why does the CMDB matter so much in SecOps?**
**A:** The CMDB provides the context that turns a list of vulnerabilities into a prioritized action list:
- **Business service mapping:** CVE on a server supporting "Online Banking" = CRITICAL; same CVE on a test server = LOW
- **Ownership:** CMDB knows which team owns `prod-db-01` → Remediation task auto-assigned
- **Maintenance windows:** CMDB patch schedules inform when to remediate
- **Business impact:** CMDB CI criticality score directly affects vulnerability priority calculation

Without CMDB: "10,000 vulnerabilities — fix all of them."
With CMDB: "Fix these 50 first — they're on systems supporting revenue-critical services."

---

**Q6: What is the SIR (Security Incident) lifecycle?**
**A:**
```
Detection → Triage → Analysis → Containment → Eradication → Recovery → Post-Incident Review
```
| Stage | Key Activities |
|-------|---------------|
| Detection | Alert from SIEM, scanner, user report |
| Triage | Validate alert, assign priority, assign analyst |
| Analysis | Scope the incident — what was compromised? |
| Containment | Isolate affected systems, block malicious IPs |
| Eradication | Remove malware, close attack vector |
| Recovery | Restore systems, verify clean state |
| Post-Incident Review | Document lessons, update runbooks, improve defenses |

---

**Q7: What is the VR workflow end-to-end?**
**A:**
```
1. Scanner runs (Tenable, Qualys, Rapid7)
2. Scan results imported into ServiceNow (via integration)
3. Vulnerable Items created per CVE per CI
4. CMDB match: which CI is affected?
5. Business criticality score applied
6. Priority = CVSS score × Business Impact
7. Remediation Task created and assigned
8. IT team patches/mitigates
9. Scanner re-scans to verify fix
10. Vulnerable Item closed
```

---

**Q8: What are IOCs and how does Threat Intelligence use them?**
**A:** IOCs (Indicators of Compromise) are artifacts that indicate a system has been compromised:
- Malicious IP addresses
- Known malware domain names
- File hashes of malicious executables
- Suspicious URLs

**TI in SecOps:**
- Pulls IOCs from external threat feeds (STIX/TAXII, commercial feeds, government feeds)
- When a Security Incident is created, TI automatically checks if any involved IPs/domains/hashes match known IOCs
- If match: incident immediately escalated and enriched with attacker profile data

---

**Q9: What is SOAR and when is it needed?**
**A:** SOAR (Security Orchestration, Automation & Response) automates response workflows that interact with systems OUTSIDE ServiceNow:
- Automatically block a malicious IP in the firewall (Palo Alto API)
- Disable a compromised user account (Active Directory API)
- Isolate an infected endpoint (CrowdStrike API)
- Pull memory dump from compromised server (SSH command)

**When needed:** When your security playbooks have steps that could be automated but currently require manual execution across multiple tools.

**Important:** SOAR is a **separate paid plugin** — not included in base SecOps.

---

**Q10: What are the 4 SecOps data enrichment tables?**
**A:**
1. **Firewall** — Network traffic data from firewalls
2. **Malware** — Malware detection details (file hash, AV signature)
3. **Network Statistics** — Network performance and connectivity data
4. **Running Processes** — Active process list from the affected endpoint

These tables are populated by SOAR workflows pulling data from external systems and linked back to the security incident for analyst context.

---

## PART 2: Configuration & Integration

**Q11: How does SIEM integration with SIR work?**
**A:**
1. SIEM (Splunk, QRadar, Sentinel) detects an anomaly and generates an alert
2. SIEM integration creates a ServiceNow Security Incident automatically
3. Incident enriched with alert details, affected assets (CMDB lookup), and IOC checks
4. Priority assigned based on SIEM severity + business impact
5. Analyst notified and investigation begins in ServiceNow
6. All actions tracked in ServiceNow; resolved in ServiceNow; synced back to SIEM if needed

---

**Q12: What vulnerability scanners does VR integrate with?**
**A:**
- **Tenable (Nessus/Tenable.io):** Most common
- **Qualys:** Enterprise-grade
- **Rapid7 InsightVM:** Popular alternative
- **CrowdStrike Spotlight:** Endpoint-based
- **Microsoft Defender:** For Microsoft environments

---

**Q13: What are Workflow Triggers in SecOps?**
**A:** Workflow Triggers monitor specific tables for changes and fire workflows when conditions are met. Unlike standard workflow conditions, they can fire multiple times on the same record.

**SecOps example:** Monitor Vulnerable Item table → when a CRITICAL CVE is imported → immediately create Remediation Task and notify the IT team.

---

## PART 3: Scenario-Based Questions

**Q14: A phishing email is reported by 10 employees. How does SecOps handle this?**
**A:**
1. Email security tool (Proofpoint/Mimecast) or employee report triggers Security Incident creation
2. Incident type: "Phishing" — SIRT notified
3. Analysis: How many employees received it? Did any click the link? Any credentials compromised?
4. CMDB: Which systems are potentially affected?
5. Containment: Block malicious URL/domain in email gateway and web proxy (SOAR automates this)
6. Eradication: Delete emails from all inboxes (API to mail server)
7. Recovery: Force password reset for any users who clicked
8. Threat Intelligence: Add phishing domain to IOC database
9. Post-incident: User awareness training triggered

---

**Q15: A vulnerability scanner finds 50,000 vulnerabilities. How does VR prioritize which to fix first?**
**A:**
1. CVSS base score pulled from NVD for each CVE
2. CMDB lookup: match each Vulnerable Item to its CI
3. CI business criticality applied (from CMDB service mapping)
4. Priority formula: CVSS × Business Criticality × Exploitability
5. Top result: CRITICAL-priority CVEs on CRITICAL business services
6. Automated Remediation Tasks created for top-priority items
7. SLA set: CRITICAL = 7 days, HIGH = 30 days, MEDIUM = 90 days
8. Teams assigned based on CI ownership in CMDB

---

**Q16: How would you explain MTTR improvement to a CISO who doesn't know ServiceNow?**
**A:** "Before SecOps, when we found a vulnerability:
1. Security analyst exports scanner report as CSV (30 min)
2. Emails it to IT manager (immediate — but often ignored)
3. IT manager manually creates tickets for each item (days)
4. No priority, no business context, no SLA tracking

With SecOps:
1. Scanner data flows automatically into ServiceNow (minutes)
2. System matches each vulnerability to the right CI in CMDB (automatic)
3. Business criticality applied — only top 50 flagged as urgent (automatic)
4. Tickets auto-created and assigned to right teams (immediate)
5. SLA clock starts, escalation fires if missed

Result: What took 2 weeks now happens in hours — that's your MTTR improvement."

---

**Q17: What is the difference between a Vulnerability and a Security Incident?**
**A:**
- **Vulnerability:** A weakness in a system that COULD be exploited (not yet attacked)
- **Security Incident:** A confirmed or suspected exploitation or breach (attack is happening/happened)

VR handles vulnerabilities proactively. SIR handles active attacks reactively.

---

## PART 4: Exam-Style Quick Facts

| Question | Answer |
|----------|--------|
| SIR table | `sn_si_incident` |
| Vulnerable Item table | `sn_vul_vulnerable_item` |
| Remediation Task table | `sn_vul_remediation_task` |
| IOC table | `sn_ti_ioc` |
| SOAR is | Separate paid plugin |
| MTTD = | Mean Time to Detect |
| MTTR = | Mean Time to Respond/Resolve |
| Enrichment tables | Firewall, Malware, Network Stats, Running Processes |
| CVE = | Common Vulnerabilities and Exposures |
| CVSS = | Common Vulnerability Scoring System (0-10) |
| IOC = | Indicator of Compromise |
| SIR final stage = | Post-Incident Review |
| Common scanner integrations | Tenable, Qualys, Rapid7 |

---

## Study Checklist
- [ ] Know: SIR vs ITSM Incident distinction
- [ ] Know: SIR lifecycle stages in order
- [ ] Know: VR workflow end-to-end
- [ ] Know: What a Vulnerable Item is (one CVE, one CI)
- [ ] Know: How CMDB enables VR prioritization
- [ ] Know: IOC definition and TI role
- [ ] Know: SOAR is a SEPARATE paid plugin
- [ ] Know: 4 enrichment tables
- [ ] Know: MTTD and MTTR definitions
- [ ] Know: Common scanner integrations
- [ ] Know: Workflow Triggers purpose
