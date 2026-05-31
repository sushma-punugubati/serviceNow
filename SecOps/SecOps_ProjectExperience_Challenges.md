# ServiceNow SecOps — Interview Notes
## "What did you do in SecOps?" + Challenges Faced

---

## 1. OVERVIEW — What is SecOps?

ServiceNow SecOps (Security Operations) bridges the gap between the security team and IT operations by bringing security incident response, vulnerability management, and threat intelligence into the same platform as ITSM and CMDB. Instead of security and IT operating in separate tools and communicating by email, SecOps creates a unified workflow where security incidents are tracked like ITSM incidents — with owners, SLAs, CMDB context, and automated response playbooks.

The key insight: most security incidents require IT action (patching, configuration changes, access revocation). When security and IT operate in the same platform, response time drops dramatically because there is no gap between "security identifies the problem" and "IT takes action."

Key areas worked on:
- Security Incident Response (SIR)
- Vulnerability Response
- Event Management integration with Splunk
- CI binding and alert correlation
- IRM integration for risk-based prioritization
- Threat Intelligence integration

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Security Incident Response (SIR) Configuration

- Configured the **Security Incident Response** module — set up Security Incident types (phishing, malware, data breach, unauthorized access, DDoS) with appropriate workflows, escalation paths, and containment playbooks for each type
- Built **automated triage workflows** in Flow Designer: when a Security Incident was created, the flow automatically enriched the record with CMDB data (what systems are affected, what business services depend on them, who owns the CI), Threat Intelligence lookups, and historical incident data for the same affected systems
- Configured **Security Incident SLAs** aligned with incident severity: P1 security incidents (active breach, data exfiltration) required acknowledgment within 15 minutes and containment actions within 2 hours; P2-P4 had progressively longer response windows
- Set up **cross-module integration**: P1 Security Incidents automatically created linked ITSM Major Incidents so the IT ops team was simultaneously engaged on the technical resolution while SecOps managed the security response

### 2.2 Event Management Integration with Splunk

- Integrated **Splunk SIEM alerts** into ServiceNow Event Management — configured the Splunk Universal Forwarder to send alerts matching defined security event rules to ServiceNow via REST
- Set up **event rules** in ServiceNow Event Management to process incoming Splunk alerts: alert severity mapping, deduplication rules to prevent the same alert from creating multiple events, and correlation rules to group related alerts into a single Event record
- Configured **CI binding** — mapped incoming event data (hostname, IP address, MAC address) to CMDB CI records so each Event was associated with the correct CI and its business service context
- Built the **AIOps alert-to-incident workflow**: events that exceeded defined severity thresholds and remained unacknowledged for 5 minutes automatically created Security Incidents in SecOps with the Event details pre-populated — eliminated the manual step of a SOC analyst creating incidents from the Splunk console

### 2.3 Vulnerability Response

- Configured **Vulnerability Response** to receive scan results from Qualys and Tenable via scheduled import — vulnerability findings were normalized, deduplicated against existing open vulnerabilities, and matched to CMDB CIs using the affected host data
- Set up **vulnerability prioritization** using the combination of CVSS score AND CI business criticality from the CMDB — a critical vulnerability on a business-critical CI was prioritized differently from the same vulnerability on a dev/test server. This was critical for focusing the patching team on what actually mattered to the business
- Configured **remediation SLA targets** by vulnerability severity + CI criticality: Critical vuln on High-criticality CI: 15-day remediation SLA; Critical vuln on Low-criticality CI: 45-day SLA — aligned with the organization's risk appetite
- Built **remediation workflows** that automatically created ITSM Change Requests for approved patches — the Security team approved the remediation plan in SecOps, which triggered the Change process in ITSM without a separate manual handoff

### 2.4 IRM Integration

- Integrated SecOps with the **IRM module** — Security Incidents above P2 severity automatically created or updated Risk records in IRM, ensuring the risk register reflected current security posture without manual intervention by the risk team
- Configured **vulnerability-to-risk mapping**: high-CVSS vulnerabilities affecting business-critical CIs automatically updated the associated operational risk record in IRM (increasing likelihood score) so the risk register stayed current between formal quarterly risk assessments
- Built automated **risk score recalculation** triggered by SecOps events: when a P1 Security Incident was created, the IRM risk records for the affected business service had their likelihood automatically increased pending investigation and resolution

### 2.5 Threat Intelligence Integration

- Configured **Threat Intelligence** data feeds — integrated STIX/TAXII threat feeds from ISAC (Information Sharing and Analysis Center) sources, automatically enriching Security Incidents with known indicators of compromise (IoCs), malware families, and threat actor TTPs
- Set up **observable lookup** on Security Incident creation: IP addresses, file hashes, and domain names extracted from the incident were automatically looked up against the threat intelligence database and enriched with context (known malicious, associated with specific APT groups, etc.)

---

## 3. CHALLENGES FACED

### Challenge 1: Alert Storm from Splunk Flooding Event Management

**Problem:** After enabling the Splunk-to-Event Management integration, the Event Management console received over 15,000 alerts in the first 24 hours. The SOC team was completely overwhelmed — they could not find real threats in the noise, and the system was creating hundreds of Security Incidents automatically from low-confidence alerts.

**Solution:** Worked with the Splunk team to implement a tiered alert approach: only Splunk alerts with severity "Critical" or "High" flowed to Event Management for incident creation; "Medium" and "Low" alerts were logged in Event Management for visibility but did not trigger incident creation. Also configured deduplication rules: the same alert type for the same source IP within a 1-hour window was deduplicated into a single Event record. Alert volume dropped from 15,000 to ~200 actionable events per day within a week.

---

### Challenge 2: CI Binding Failures — Alerts Not Matching CMDB

**Problem:** About 35% of incoming Splunk alerts were not binding to any CI in the CMDB. These unbound alerts had no business context (what service is affected? who owns it?) making them significantly harder to triage and prioritize.

**Solution:** Analyzed the failed CI bindings and found three root causes: (1) alerts used short hostnames while CMDB stored FQDNs, (2) some servers had changed IP addresses without CMDB being updated, and (3) cloud instances had dynamic hostnames not matching any CMDB record. Fixed each: added hostname normalization in the CI binding rule (try short name, try FQDN, try with domain appended), fixed the CMDB IP address update process, and added a cloud resource tag to Splunk alert payloads that mapped to the CMDB cloud CI record. Binding success rate improved from 65% to 94%.

---

### Challenge 3: Vulnerability Prioritization — Too Many "Critical" Vulns, No Business Context

**Problem:** After the first Qualys scan import, the vulnerability list showed 2,400 "Critical" vulnerabilities. The patching team had no way to determine which to address first and the list was so long that triage paralysis set in — nothing was getting patched.

**Solution:** Applied business context filtering immediately: only vulnerabilities on CIs with "High" or "Critical" business criticality (from CMDB) AND with a known public exploit (CVSS exploitability score 3.9+) were surfaced as top priority. This reduced the actionable "patch now" list from 2,400 to 87 items — a manageable workload. The remaining 2,313 were still tracked in Vulnerability Response but with longer SLA windows appropriate to their risk level.

---

### Challenge 4: Security-IT Handoff Delays

**Problem:** When SecOps identified a vulnerability requiring patching or a Security Incident requiring access revocation, there was a formal handoff process to IT that involved email, meetings, and manually created ITSM tickets. Average time from security identification to IT action was 3.4 business days for non-critical issues.

**Solution:** Implemented automated ITSM ticket creation directly from SecOps workflows — when a vulnerability reached "Approved for Remediation" state in SecOps, a Change Request was automatically created in ITSM with all relevant context pre-populated (affected CIs, patch details, risk justification, SecOps ticket link). For security incidents requiring account disablement, an automated Task was created and assigned to the identity team with a P1 SLA. Handoff time reduced from 3.4 days to same-day for approved remediations.

---

### Challenge 5: False Positives Overwhelming SOC Analysts

**Problem:** The threat intelligence IoC lookups were generating too many false positive enrichments — common CDN IP addresses and widely-used cloud infrastructure IPs were being flagged as "potentially malicious" because they appeared in generic threat feeds. SOC analysts were spending significant time investigating IoC hits that turned out to be CDN traffic.

**Solution:** Built a **whitelist of known-good infrastructure** (CDN ranges, major cloud provider IP ranges, known corporate SaaS application IPs) that was excluded from threat intelligence lookups. Also configured IoC confidence scoring: only IoCs with a confidence score above 70 (on a 100-point scale) from the threat feed triggered enrichment alerts. Low-confidence IoCs were logged but did not trigger analyst workflow. False positive investigation time reduced by approximately 60%.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **SIR (Security Incident Response)** | Module for tracking, triaging, and resolving security incidents with automated workflows and CMDB context |
| **Vulnerability Response** | Module for ingesting vulnerability scan results, prioritizing by business context, and tracking remediation |
| **Event Management** | Receives alerts from monitoring tools (Splunk, Nagios) and correlates them into Events and Incidents |
| **CI Binding** | Matching an alert or event to the correct CI in the CMDB — provides business context for the affected system |
| **AIOps** | Automated correlation of events into incidents using defined rules — reduces manual triage |
| **Threat Intelligence** | External data about known threats (IoCs, TTPs, malware) used to enrich Security Incidents |
| **CVSS Score** | Common Vulnerability Scoring System — numerical severity rating for vulnerabilities (0-10) |
| **SecOps-IRM Integration** | Security incidents and vulnerabilities automatically update risk records in IRM |
| **Remediation SLA** | Target time to patch or remediate a vulnerability based on severity + CI business criticality |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Alert volume reduced from **15,000 to ~200 actionable events per day** after tiering and deduplication rules
- CI binding success rate improved from **65% to 94%** after hostname normalization and CMDB cleanup
- Actionable "critical patch now" vulnerability list reduced from **2,400 to 87 items** after business context filtering
- Security-to-IT handoff time reduced from **3.4 business days to same-day** after automated ITSM ticket creation
- False positive investigation time reduced by **~60%** after IoC whitelist and confidence scoring
- P1 Security Incident mean time to containment reduced by **~45%** with automated enrichment and cross-module notification
