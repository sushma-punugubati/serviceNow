# ServiceNow SecOps — Study Materials Index

> Security Operations: SIR (Security Incident Response) + VR (Vulnerability Response) + TI (Threat Intelligence)

## Files in This Folder

### 1. [SecOps_Basics.md](SecOps_Basics.md)
Core concepts, SIR/VR/TI modules, key tables, SOAR, enrichment, CMDB integration, roles, glossary.

### 2. [SecOps_CustomerSuccessStories.md](SecOps_CustomerSuccessStories.md)
Real-world security scenarios: phishing response, vulnerability management at scale, SIEM integration.

### 3. [SecOps_InterviewQuestions.md](SecOps_InterviewQuestions.md)
30+ Q&As covering SecOps architecture, SIR, VR, TI, SOAR, integrations, and scenario questions.

### 4. [SecOps_CommonIssues.md](SecOps_CommonIssues.md)
Common SecOps implementation pitfalls: alert fatigue, CMDB gaps, scanner integration failures, false positives.

## Quick Reference

```
SIR  = Security Incident Response (lifecycle: New → Analysis → Contain → Eradicate → Recover → Review)
VR   = Vulnerability Response (scanner → CMDB → prioritize → remediate)
TI   = Threat Intelligence (IOCs → enrich incidents)
SOAR = Security Orchestration Automation & Response (separate plugin)
SIEM = Feeds events into SIR automatically

Key metrics:
  MTTD = Mean Time to Detect
  MTTR = Mean Time to Respond/Resolve

Enrichment tables = Firewall | Malware | Network Statistics | Running Processes
CMDB role        = Maps vulnerabilities/incidents to business services for prioritization
```

## Sources
- [ServiceNow SecOps — Reco.ai Guide](https://www.reco.ai/hub/servicenow-security-operations-secops)
- [SecOps Knowledge & Troubleshooting — ServiceNow Community](https://www.servicenow.com/community/secops-articles/security-operations-secops-knowledge-amp-troubleshooting/ta-p/2757839)
- [ServiceNow SecOps Transformation PDF](https://www.servicenow.com/content/dam/servicenow-assets/public/en-us/doc-type/success/reference-architecture/secops-transformation.pdf)
