# ServiceNow SecOps — Customer Success Stories

Real-world examples of organizations using ServiceNow Security Operations to reduce vulnerability exposure, accelerate incident response, and automate security workflows.

---

## Story 1: Global Retailer — Reducing Vulnerability Remediation Time by 78%

**Industry:** Retail  
**Challenge:** A national retailer with 2,000+ stores ran quarterly Tenable vulnerability scans and produced a 40,000-item CSV report. The security team emailed sections of the report to regional IT managers, who manually created tickets. Priority was inconsistent — a LOW CVE on a POS system was often fixed before a CRITICAL CVE on a payment processing server. Remediation SLAs were tracked in Excel, and nothing enforced them.

**What SecOps Solved:**
- Tenable integration: scan results flow directly into ServiceNow VR (Vulnerability Response) — no CSV, no email
- Vulnerable Items created automatically — one record per CVE per CI
- CMDB integration: POS systems linked to "Payment Processing" business service → CRITICAL business criticality score applied
- Priority formula: CVSS × Business Criticality → CRITICAL CVEs on payment systems rise to top automatically
- Remediation Tasks auto-created and assigned to the team that owns each CI in CMDB
- SLA: CRITICAL = 7 days, HIGH = 30 days — automated escalation if missed

**Results:**
- Remediation cycle time: 47 days average → 10 days for CRITICAL vulnerabilities
- PCI-DSS compliance: first year passed with zero findings related to vulnerability management
- IT team efficiency: 80% reduction in manual ticket creation
- Business focus: security team now spending time on analysis, not administration

**Key SecOps Features Used:** VR, Tenable Integration, CMDB Business Criticality, Vulnerable Items, SLA Management

---

## Story 2: Healthcare System — Incident Response at HIPAA Speed

**Industry:** Healthcare  
**Challenge:** A healthcare network with 20 hospitals was experiencing 300+ security alerts per day from their SIEM (Splunk). The security operations team of 8 analysts was manually triaging each alert, creating incidents in a separate ticketing system, and performing investigations with no standard process. Average time from SIEM alert to incident creation: 4 hours. HIPAA required breach determination within 60 days — they were routinely exceeding 45 days just to classify an incident.

**What SecOps Solved:**
- Splunk integration: SIEM alerts create ServiceNow Security Incidents automatically — 0-minute lag
- Playbooks: common incident types (phishing, ransomware, unauthorized access) have step-by-step analyst guidance
- Threat Intelligence: IOC matching runs automatically when incident created — known malicious IPs flagged instantly
- CMDB integration: affected systems automatically linked to business services and HIPAA-scoped systems
- SOAR: common containment actions automated (block IP in firewall, disable user account) — approved by analysts with one click

**Results:**
- Alert to Security Incident: 4 hours → under 5 minutes (automation)
- HIPAA breach determination: average 45 days → 12 days
- Analyst capacity: same 8 analysts handling 3× the alert volume without additional headcount
- Ransomware containment: endpoint isolation triggered in under 10 minutes via SOAR (was: 2+ hours of manual action)

**Key SecOps Features Used:** SIR, Splunk Integration, Playbooks, Threat Intelligence (IOC matching), CMDB Integration, SOAR

---

## Story 3: Financial Services Firm — Closing the Security-IT Gap

**Industry:** Financial Services  
**Challenge:** A mid-sized investment management firm had a classic problem: the security team found vulnerabilities, emailed them to IT, IT lost the emails, security re-sent, IT said "we're too busy," and nothing got fixed. The CISO's quarterly board report showed the same 200 critical vulnerabilities for 6 consecutive quarters. When they were finally hit by a ransomware attack, the initial infection vector was a 14-month-old unpatched Exchange vulnerability that had been in the "email queue" the whole time.

**What SecOps Solved:**
- VR deployed with Qualys integration — vulnerabilities flow directly to IT as Remediation Tasks (not emails)
- IT managers see their team's Remediation Task queue on their dashboard — no email, no excuses
- SLA tracking: CRITICAL vulnerability tasks escalate to IT director if not started within 48 hours
- Metrics dashboard: CISO can see open vulnerabilities by age, owner, business service in real-time
- Risk-based prioritization: post-breach analysis showed the Exchange CVE would have been CRITICAL priority in VR (high CVSS + CMDB shows it supports business email = critical service)

**Results:**
- Post-implementation: no vulnerability over 30 days old in CRITICAL priority bucket (ever again)
- CISO board report: quarterly vulnerability metrics now show trending improvement, not stagnation
- IT + Security SLA agreement formalized: first time in company history
- The Exchange ransomware CVE class (high CVSS, email infrastructure) now in auto-escalation policy

**Key SecOps Features Used:** VR, Qualys Integration, Remediation Tasks, SLA Escalation, Executive Dashboards

---

## Story 4: Technology Company — Threat Intelligence Enrichment Accelerating Response

**Industry:** Technology / Software  
**Challenge:** A cloud software company with 500 employees was receiving Threat Intelligence feeds from multiple vendors (Recorded Future, AlienVault OTX) but had no way to operationalize them. Analysts had to manually cross-reference IOC feeds against SIEM alerts — a process that took 2-3 hours per incident. Meanwhile, nation-state actors were known to be targeting their IP.

**What SecOps Solved:**
- Threat Intelligence (TI) module deployed with STIX/TAXII feed integration
- IOC database populated: malicious IPs, domains, file hashes continuously updated from feeds
- When Security Incident created, TI automatically checks all IPs/domains/hashes in the incident against the IOC database
- Match found: incident priority escalated, attacker profile data attached (nation-state? criminal group? targeted campaign?)
- Analyst sees: "This IP is attributed to APT-29 (Cozy Bear). 3 other companies in your sector were hit this week."

**Results:**
- IOC matching: manual 2-3 hours → automated in under 60 seconds
- Nation-state attributed incidents: identified and escalated in minutes (was: sometimes never identified)
- Analyst investigation quality improved — attacker context available immediately, not after hours of research
- Threat feed ROI realized: feeds were being paid for but unusable before TI module; now operationalized

**Key SecOps Features Used:** Threat Intelligence (TI), IOC Database, STIX/TAXII Integration, Incident Auto-Enrichment, Priority Escalation

---

## Story 5: Government Agency — SOAR Automation Handling Phishing at Scale

**Industry:** Government / Public Sector  
**Challenge:** A large federal agency's 50,000 employees were targeted by 500-800 phishing emails per day. The security team had 6 analysts manually investigating reported phishing emails — clicking links in sandboxes, analyzing headers, checking URLs against threat feeds, deleting emails from user inboxes. Each investigation took 25-40 minutes. The team was drowning and real incidents were delayed.

**What SecOps Solved:**
- SOAR playbook deployed for phishing response:
  1. Employee reports phishing → Security Incident created automatically
  2. SOAR extracts URL/domain/sender from email headers (API to email gateway)
  3. URL checked against Threat Intelligence IOC database automatically
  4. If known malicious: email automatically deleted from ALL inboxes (API to Exchange/O365) — no human needed
  5. Domain blocked in web proxy (API to Bluecoat/Zscaler) automatically
  6. If unknown URL: sandboxed automatically, verdict returned to analyst for review
- Full workflow: known phishing = 0 analyst time; unknown phishing = 10-minute analyst review (not 40)

**Results:**
- 70% of phishing incidents handled with zero analyst intervention (known IOC match → auto-remediate)
- Remaining 30%: analyst investigation time 40 minutes → 10 minutes
- Phishing response capacity: 6 analysts now handle 5× the volume
- Dwell time eliminated: malicious emails removed from all inboxes within 90 seconds of report (was: hours or days)

**Key SecOps Features Used:** SIR, SOAR, Phishing Playbook, TI Integration, Email Gateway API, Web Proxy API

---

## Story 6: Manufacturing Company — Securing OT/ICS with Unified Vulnerability Management

**Industry:** Manufacturing / Industrial  
**Challenge:** A manufacturing company operating industrial control systems (ICS) across 12 factories had an IT security team but no OT (Operational Technology) security coverage. When Claroty (an OT security scanner) identified 3,400 vulnerabilities in PLCs, HMIs, and SCADA systems, the results sat in Claroty's platform with no way to get them to the right teams for remediation. OT vulnerabilities are uniquely complex: patches can't simply be applied without risking production line downtime.

**What SecOps Solved:**
- Claroty integration: OT vulnerability data flows into ServiceNow VR alongside IT vulnerability data
- CMDB: OT CIs added (PLCs, HMIs, SCADA servers) with factory and production line as business service
- Priority formula modified: "Production Impact" field added — CVE on a bottleneck production PLC = CRITICAL
- Remediation Tasks linked to OT maintenance windows (CMDB tracks maintenance schedules per factory)
- Compensating controls tracked: when patching isn't possible, network segmentation or monitoring as alternative remediation is documented in VR

**Results:**
- First unified IT + OT vulnerability view in company history
- 127 CRITICAL OT vulnerabilities remediated in 90 days (previously: zero visibility, zero remediation)
- Production-safe remediation: zero production outages caused by security patching (maintenance windows respected)
- Cyber insurance underwriter satisfied — documented OT vulnerability program reduced premium by 18%

**Key SecOps Features Used:** VR, Claroty Integration, CMDB (OT CIs), Custom Priority Formula, Compensating Controls, Maintenance Window Integration

---

## Common Themes Across SecOps Success Stories

| Theme | Why It Matters |
|-------|---------------|
| **SIEM → Incident automation** | Manual alert triage is the #1 analyst bottleneck — automation resolves it |
| **CMDB context drives prioritization** | Same CVE = CRITICAL on payment server, LOW on test server — without CMDB, you can't know |
| **Closing the Security-IT gap** | Vulnerabilities stay open forever when communication is email — Remediation Tasks fix this |
| **SOAR for high-volume repetitive tasks** | Phishing is the #1 SOAR use case — massive volume, known response steps |
| **Threat Intelligence operationalization** | Threat feeds are waste of money without TI module to act on them |
| **OT/ICS security** | Manufacturing and utilities increasingly need IT + OT unified in one platform |

---

## Key Metrics SecOps Customers Report

| Metric | Typical Result |
|--------|---------------|
| MTTD (Mean Time to Detect) | Not changed by SecOps — improved by SIEM. SecOps reduces handoff time. |
| MTTR (Mean Time to Respond) | 40-78% reduction |
| Alert to Incident creation | Hours → minutes (with SIEM integration) |
| CRITICAL vulnerability remediation cycle | 30-90 days → 7-14 days |
| Phishing response time (SOAR) | 40 min analyst → 90 seconds automated |
| Analyst capacity improvement | 2-5× more alerts handled per analyst |
