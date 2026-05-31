# ServiceNow ITOM — Customer Success Stories & Case Studies

---

## Case Study 1: Global Bank — CMDB Accuracy from 35% to 89%

**Organization:** Global investment bank, 85,000 employees, 45,000 managed CIs
**Challenge:** CMDB accuracy was 35% — stale, manually maintained records that did not reflect actual infrastructure. Change impact analysis was unreliable. Major incidents took hours longer to resolve because teams did not know what depended on affected systems.

**Solution Implemented:**
- Deployed ServiceNow Discovery across all data centers and network segments
- Configured 8 MID Servers across geographic regions
- Tuned IRE identification rules using hardware serial numbers and VMware UUIDs
- Deployed Service Mapping for 40 critical business services
- Configured CMDB health score tracking with ownership-based improvement targets

**Results:**
- CMDB accuracy improved from **35% to 89%** within 12 months
- Major incident MTTR reduced by **35%** — CI context available immediately vs. manual investigation
- Change impact assessment accuracy improved: **93%** of change-caused incidents were predicted by impact analysis (vs. 40% before)
- Manual CMDB update effort eliminated for infrastructure team: **~40 hours/week** saved

---

## Case Study 2: E-Commerce Company — Alert Noise Reduction 94%

**Organization:** Large e-commerce platform, 500 engineers, 2,000+ infrastructure CIs
**Challenge:** Monitoring tools (Datadog, PagerDuty, Splunk) were sending 15,000+ alerts per day. Engineers were experiencing alert fatigue — P1 incidents were being missed because they were buried in low-priority noise. Mean time to acknowledge a real incident was 47 minutes.

**Solution Implemented:**
- Deployed Event Management with connectors for Datadog, Splunk, and PagerDuty
- Configured severity-based filtering at each source — only High/Critical forwarded to ServiceNow
- Implemented AIOps alert correlation — learned alert patterns over 90 days
- Built CI Binding rules covering all 2,000 CIs
- Configured alert suppression during scheduled maintenance windows

**Results:**
- Daily actionable alert count reduced from **15,000 to under 900** (94% noise reduction)
- Mean time to acknowledge real incidents reduced from **47 minutes to 8 minutes**
- False positive incidents (alerts that created incidents but were not real issues): reduced by **87%**
- On-call engineer escalations reduced by **60%** — more issues resolved without escalation

---

## Case Study 3: Healthcare System — Service Mapping Enables Zero-Downtime Maintenance

**Organization:** National healthcare provider, 200+ hospitals, 500 critical applications
**Challenge:** Planned maintenance windows frequently caused unplanned outages because teams did not know all the dependencies of the systems they were changing. A routine database maintenance window took down an EMR application because a hidden dependency was not known.

**Solution Implemented:**
- Deployed Service Mapping for all 500 critical applications
- Mapped 47,000 CI relationships covering application, middleware, and infrastructure tiers
- Integrated Service Maps with Change Management: before a change is approved, the CI's service map is reviewed for hidden dependencies
- Built automated impact analysis: "if this CI is taken offline, these business services are affected"

**Results:**
- Unplanned outages from maintenance activities: reduced by **78%** in the first year
- Change-caused incidents: reduced from 22 per quarter to 5
- Average change planning time reduced: **3 hours saved per change** (no manual dependency research)
- Compliance: zero EMR-related maintenance outages in the 18 months following Service Mapping deployment

---

## Case Study 4: Technology Company — Cloud Discovery Eliminating Shadow IT Risk

**Organization:** SaaS technology company, 3,000 employees, AWS-native infrastructure
**Challenge:** Cloud infrastructure was growing faster than the CMDB could track. Engineers were spinning up EC2 instances, S3 buckets, and RDS databases without registering them. Security team had no visibility into 30-40% of the cloud estate. Vulnerability scans were missing unregistered resources.

**Solution Implemented:**
- Configured AWS credential-less discovery using IAM roles for all 12 AWS accounts
- Deployed Cloud Discovery covering EC2, S3, RDS, VPC, Lambda, and EKS
- Implemented nightly Discovery runs with real-time updates for state changes (instance start/stop)
- Tagged all cloud CIs with account, environment, and team ownership from AWS tags
- Integrated Cloud Discovery results with SecOps vulnerability scanning

**Results:**
- Cloud CI coverage: improved from **~65% to 99%** of all active cloud resources
- Shadow IT resources discovered: **847 unregistered resources** found in first scan (including 12 exposed S3 buckets)
- Security scan coverage: **100%** of cloud resources now included in weekly vulnerability scans
- Cloud spend visibility: CMDB data enabled Finance to identify **$340K in annual waste** from unused cloud resources

---

## Case Study 5: Telecommunications Provider — Event Management Reducing MTTR by 55%

**Organization:** Regional telecommunications provider, 8 million subscribers, 24/7 NOC
**Challenge:** NOC team was managing 5 separate monitoring tools (Nagios, SolarWinds, Dynatrace, Zabbix, custom in-house tool) with no unified view. P1 incidents took on average 3.2 hours to resolve because engineers spent the first hour correlating data across tools to understand the full scope.

**Solution Implemented:**
- Deployed Event Management as the unified alert aggregation layer for all 5 tools
- Configured MID Server connectors for each monitoring tool
- Implemented CI-bound alert correlation — related alerts grouped automatically by CI dependency chain
- Built service impact dashboards showing affected subscriber count per alert group
- Integrated with ITSM: P1 incidents auto-created with pre-populated impact analysis

**Results:**
- P1 MTTR reduced from **3.2 hours to 1.4 hours** (55% reduction)
- Alert-to-incident correlation time: reduced from **60 minutes of manual correlation to under 3 minutes** (automated)
- NOC screen count reduced from **5 separate tool dashboards to 1** unified ServiceNow view
- Subscriber impact quantified automatically: each incident now shows estimated affected subscriber count within 60 seconds of creation
- Annual customer SLA penalty avoidance: estimated at **$1.8M** from faster P1 resolution
