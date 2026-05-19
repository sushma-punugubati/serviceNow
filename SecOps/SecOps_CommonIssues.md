# ServiceNow SecOps — Common Issues, Pitfalls & Troubleshooting

Issues encountered during SecOps implementation and ongoing support, based on ServiceNow community posts, partner experience, and real-world security operations implementations.

---

## Issue 1: Vulnerability Scanner Integration Not Importing Data Correctly

**Category:** Integration / VR  
**Severity:** Critical — VR has no data without scanner integration

**The Problem:**
Tenable/Qualys/Rapid7 integration is configured but scan results aren't appearing in ServiceNow. Or only partial results import (10,000 of 50,000 vulnerabilities). Or imports fail silently — no error message, but no data.

**Why It Happens — Multiple Causes:**
1. **API credentials expired or insufficient permissions** — scanner API key doesn't have permission to export all data
2. **Large scan results exceed ServiceNow import limits** — very large scan files cause timeouts
3. **Transformation map misconfigured** — data imports but maps to wrong fields
4. **MID Server not reaching scanner API endpoint** — firewall blocks outbound call to scanner cloud

**Root Cause:**
Scanner integrations pull large datasets from external APIs. Any break in the chain (authentication, network, transform) results in missing data.

**Fix / Prevention:**
- Test import manually: trigger a manual import for a small scan and verify results appear
- Check Integration logs in ServiceNow: `System Import Sets → Import Sets` — look for errors
- Verify API credentials: test scanner API directly from the MID Server using curl
- For large scans: configure chunked imports (batch by asset group or subnet)
- Set up monitoring: daily check that yesterday's scan data arrived

**Community Reference:** ServiceNow Community — "Tenable/Qualys integration issues" — top SecOps integration thread

---

## Issue 2: Vulnerable Item Priority Calculation Ignoring CMDB Business Criticality

**Category:** Configuration / VR  
**Severity:** High — prioritization is wrong, wrong CVEs get fixed first

**The Problem:**
VR creates Vulnerable Items with priority based only on CVSS score. The CMDB link exists but business criticality isn't being applied to the priority calculation. A CVSS 6.0 vulnerability on a payment system is ranked lower than a CVSS 7.0 on a test server — the opposite of what should happen.

**Why It Happens:**
The integration between VR and CMDB requires that:
1. CIs are correctly mapped to Business Services in CMDB
2. Business Service criticality is populated in CMDB
3. VR priority calculation is configured to use the business criticality score

Any one of these missing breaks the full prioritization.

**Root Cause:**
Priority calculation in VR pulls business criticality from the CI's linked Business Service in CMDB. If CIs aren't mapped to Business Services, or Business Services don't have criticality scores, the calculation defaults to CVSS score alone.

**Fix / Prevention:**
- CMDB prerequisite: every CI must be mapped to a Business Service with a criticality rating (Critical/High/Medium/Low)
- VR configuration: verify `sn_vul.calculator.use_business_criticality = true` system property is enabled
- Test: find a known CRITICAL business service CI and confirm its Vulnerable Items show elevated priority
- Ongoing: CI to Business Service mapping must be maintained — if CMDB team removes the relationship, priority breaks

---

## Issue 3: SIEM Integration Creating Duplicate Security Incidents

**Category:** Integration / SIR  
**Severity:** High — analyst workload multiplied

**The Problem:**
Splunk/QRadar integration creates Security Incidents in ServiceNow, but every time the SIEM updates the same alert, a new Security Incident is created. The same phishing campaign generates 47 separate Security Incidents instead of 1 incident with 47 alerts grouped together.

**Why It Happens:**
The SIEM integration correlation logic isn't configured. Without deduplication rules, each SIEM event creates a new ServiceNow record.

**Root Cause:**
SIEM integrations need deduplication logic: "if a Security Incident already exists for this alert type / source IP / timeframe, add to the existing incident rather than creating a new one."

**Fix / Prevention:**
- Configure deduplication in the SIEM integration: match by alert type + source IP + time window
- ServiceNow SIR has built-in correlation — configure "Correlation Rules" to group related events
- Set a "lookback window": "if a Security Incident for this source IP was created in the last 4 hours, add to it"
- Review daily: count of new Security Incidents created — sudden spikes indicate deduplication failure

---

## Issue 4: Remediation Tasks Not Being Completed — IT Teams Ignore Them

**Category:** Process / Adoption  
**Severity:** High — VR provides no remediation value if tasks aren't worked

**The Problem:**
VR creates Remediation Tasks and assigns them to IT teams based on CI ownership in CMDB. IT teams don't work the tasks. The Remediation Task queue grows to thousands of items. Critical vulnerabilities from 6 months ago are still open. The CISO asks "why aren't vulnerabilities being fixed?" — and the answer is "IT doesn't know they have to."

**Why It Happens:**
IT teams aren't part of the security culture. They see Remediation Tasks as security team work, not IT work. Without SLA enforcement and management attention, tasks are ignored.

**Root Cause:**
Remediation requires a partnership between Security (who finds and prioritizes) and IT (who patches and mitigates). Without explicit SLA agreements, management visibility, and escalation — it doesn't happen.

**Fix / Prevention:**
- SLA agreement: Security and IT agree in writing: CRITICAL vulnerabilities must be remediated in 7 days
- IT manager dashboard: shows their team's open Remediation Task count and SLA status
- Escalation workflow: if Remediation Task is overdue, escalate to IT Director and CISO
- Monthly metric: "Mean Time to Remediate" by team — visible to CIO, creates accountability
- Include in IT team performance reviews: vulnerability remediation rate is a measurable metric

---

## Issue 5: SOAR Playbooks Failing Due to API Credential Issues

**Category:** Configuration / SOAR  
**Severity:** High — automated response stops working silently

**The Problem:**
SOAR playbooks were working fine for 3 months. Then they silently stopped. The IP blocking playbook runs but the firewall rule isn't being applied. User account disable playbook runs but the Active Directory account remains active. No error is visible to the analyst.

**Why It Happens:**
SOAR playbooks call external APIs (firewall, Active Directory, endpoint). API credentials (service accounts, API keys, OAuth tokens) expire. When credentials expire, playbook steps fail — but if error handling isn't configured, the failure is silent.

**Root Cause:**
API credentials have expiry dates. SOAR integrations need credential rotation processes and error handling that surfaces failures to analysts.

**Fix / Prevention:**
- Track credential expiry: log all SOAR integration credentials with expiry dates in a secrets management tool
- Test playbooks weekly with an automated synthetic test — not just when an incident occurs
- Configure error handling: if a playbook step fails, create a ServiceNow task for the analyst to take manual action
- Alert on credential failures: failed SOAR API calls should generate notifications, not silent failures

---

## Issue 6: Threat Intelligence Feeds Not Maintained — IOC Database Goes Stale

**Category:** Configuration / Threat Intelligence  
**Severity:** Medium-High — TI enrichment provides wrong or outdated intel

**The Problem:**
Threat Intelligence feeds were configured at go-live (6 months ago). The IOC database now has 50,000 entries but nobody has validated if the feeds are still updating. Old IOC entries from 2 years ago remain as "active" — some have been taken down (sinkholed domains, decommissioned IPs). Incidents are being flagged against IOCs that are no longer real threats.

**Why It Happens:**
Threat feeds require maintenance. IOCs have a shelf life — a malicious IP from 2 years ago may now be assigned to a legitimate company. Without regular feed maintenance, the IOC database becomes noise.

**Root Cause:**
Threat Intelligence is not a set-and-forget capability. IOC databases must be regularly refreshed and stale indicators must be expired.

**Fix / Prevention:**
- Set IOC expiry dates: most TI feeds include expiry — configure IRM to respect expiry dates
- Feed health monitoring: check feed last-updated timestamp weekly — alert if a feed hasn't updated in 7 days
- IOC confidence scoring: use confidence levels from feeds; don't treat a low-confidence IOC the same as a high-confidence one
- Regular review: quarterly review of IOC database — prune entries older than 12 months with no recent confirmation

---

## Issue 7: Security Incident Playbooks Too Vague — Analysts Don't Follow Them

**Category:** Process / SIR  
**Severity:** Medium — inconsistent incident response

**The Problem:**
Playbooks are configured in SIR for common incident types (phishing, ransomware, unauthorized access). But the steps are so vague that analysts don't know what to actually do. "Step 3: Investigate the incident" — investigate how? Using what tools? Looking for what?

**Why It Happens:**
Playbooks were built by a project manager or GRC team, not by experienced security analysts who actually handle incidents. The result is a process diagram that doesn't guide actual work.

**Root Cause:**
Effective playbooks must be written by the people who do the work — experienced SOC analysts. They must include specific tool names, specific queries, specific decision criteria.

**Fix / Prevention:**
- Involve analysts: have actual SOC analysts write the playbook steps — they know what they do in practice
- Specific steps: "Query Splunk for all events from this IP in the last 24 hours using query: index=firewall src_ip=[INCIDENT_IP]"
- Decision points: "If memory dump shows known malware signature → escalate to Tier 3. If clean → close."
- Test each playbook on a real (or simulated) incident before publishing
- Review after every major incident: "Did the playbook help or hinder? What step was confusing?"

---

## Issue 8: VR Scope Too Large — 100,000 Vulnerable Items Overwhelms Teams

**Category:** Process / VR  
**Severity:** High — VR becomes unusable

**The Problem:**
Scanner integration is working perfectly — it's found 100,000 Vulnerable Items. Remediation Tasks have been auto-created for all of them. IT teams receive queues of 1,000+ tasks. Nobody knows where to start. Teams feel overwhelmed and stop looking at the queue entirely.

**Why It Happens:**
VR was deployed without clear scope or remediation SLA tiering. Every Vulnerable Item creates a Remediation Task regardless of priority. The result is a flood that demotivates teams.

**Root Cause:**
Not all vulnerabilities need immediate action. Without priority-based filtering and SLA tiers, everything is equally urgent — which means nothing is urgent.

**Fix / Prevention:**
- Focus first: only create Remediation Tasks automatically for CRITICAL and HIGH priority items
- MEDIUM: report monthly, no automatic task
- LOW: track but don't task — address only when patching for other reasons
- Risk acceptance: allow IT managers to formally accept LOW/MEDIUM risks (documented in VR)
- Phased rollout: start VR with 1-2 CI classes (e.g., servers only), prove value, then expand

---

## Issue 9: SecOps and ITSM Incidents Confused — Wrong Process Applied

**Category:** Process / Governance  
**Severity:** Medium — security incidents handled as service desk tickets

**The Problem:**
Service desk staff receive reports of suspicious emails or locked accounts and create standard ITSM Incidents instead of Security Incidents (SIR). Security team never sees them. The potential phishing campaign or compromised account sits in the ITSM queue waiting for an L1 tech to "fix" the locked account — not a security investigation.

**Why It Happens:**
Service desk staff don't know the difference between a service interruption and a security event. Triage training wasn't provided.

**Root Cause:**
SecOps and ITSM serve different purposes but share similar-sounding trigger events. Without clear triage guidelines and training, security events are misclassified.

**Fix / Prevention:**
- Training: service desk must know the list of security triggers: phishing, malware popup, suspicious login, ransomware note, unexplained account lockout
- Triage form: add "Is this a potential security event?" question to service desk intake
- Automated promotion: ITSM Incident can be promoted to SIR Security Incident with one click
- Business Rule: if ITSM Incident category = "Malware" or "Phishing" → automatically create a Security Incident

---

## Issue 10: CMDB CI Ownership Not Maintained — Remediation Tasks Assigned to Wrong Teams

**Category:** Data Quality / CMDB  
**Severity:** High — vulnerabilities can't be remediated because they're assigned to wrong people

**The Problem:**
VR correctly identifies the CI affected by a vulnerability and creates a Remediation Task assigned to the CI's Support Group in CMDB. But the Support Group in CMDB was last updated 2 years ago — the team was reorganized, the old group was renamed, and the new team doesn't see the tasks.

**Why It Happens:**
CMDB CI ownership data (Support Group, Managed By) becomes stale when organizational changes occur. IT doesn't update CMDB when teams are restructured.

**Root Cause:**
VR assignment logic is only as accurate as CMDB ownership data. Stale CMDB data → wrong Remediation Task assignments → vulnerabilities unaddressed.

**Fix / Prevention:**
- CMDB ownership audit: quarterly review of CI Support Group assignments — verify they're still accurate
- HR/ITSM integration: when a team is reorganized, trigger a CMDB ownership review workflow
- VR monitoring: Remediation Tasks with no activity in 14 days → investigate if assigned team is correct
- CI owner confirmation: annual process where Support Groups confirm they still own their CI set

---

## Quick Troubleshooting Reference

| Symptom | Likely Cause | First Step |
|---------|-------------|-----------|
| Scanner data not importing | API credentials; MID Server network; transformation map | Check Integration logs; test API call from MID Server |
| Vulnerable Items all same priority | CMDB business criticality not linked | Verify CI → Business Service mapping; check VR priority property |
| Duplicate Security Incidents from SIEM | No deduplication/correlation logic | Configure Correlation Rules; set lookback window |
| Remediation Tasks piling up unworked | No IT accountability; no SLA enforcement | Agree SLA; add IT manager dashboard; configure escalation |
| SOAR playbooks silently failing | API credentials expired | Test each playbook integration; add error handling steps |
| 100,000 Vulnerable Items — teams overwhelmed | No priority filtering on task creation | Only auto-create tasks for CRITICAL/HIGH; accept LOW/MEDIUM risk |
| Security events handled as ITSM incidents | Service desk triage training gap | Train on security triggers; add Business Rule for auto-promotion |

---

## Sources and References

- ServiceNow Community: community.servicenow.com — SecOps, VR, SIR, SOAR forums
- ServiceNow Docs: Security Operations Implementation Guide, VR Best Practices, SOAR Configuration
- ServiceNow Knowledge Conference: "SecOps Deployment Best Practices" sessions
- ServiceNow Now Create SecOps methodology documentation
- LinkedIn: ServiceNow SecOps practitioners sharing implementation lessons
- SANS Institute: SOC implementation best practices (complementary to SecOps platform guidance)
