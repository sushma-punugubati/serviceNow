# ServiceNow SAM Pro — Interview Notes
## "What did you do in SAM Pro?" + Challenges Faced

---

## 1. OVERVIEW — What is SAM Pro?

SAM Pro (Software Asset Management Professional) manages the full lifecycle of software licenses — from purchase through deployment, usage monitoring, and reclamation. The core goal: always know how many licenses you own, how many are installed, and whether you are compliant — without waiting for a vendor audit to find out.

In my work at Dell, SAM Pro is one of the most business-critical modules we manage. A single unplanned Microsoft True-Up can result in millions in back-payment. SAM Pro turns that risk into a managed weekly process.

Key areas I worked on:
- Software Model catalog setup and normalization
- Publisher Pack configuration (Microsoft, Adobe, Oracle, IBM)
- License Entitlement records from contract data
- Automated Reconciliation Schedules
- SaaS License Management (Microsoft 365, Zoom, Salesforce, Tableau)
- Reclamation workflows for unused license recovery
- Compliance dashboards and executive reporting
- Integration with SCCM and Intune for installation data

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Software Model Catalog and Normalization

- Set up the **Software Model catalog** starting with Publisher Pack installation for the top 15 publishers by installation volume — Microsoft, Adobe, Oracle, VMware, IBM, Symantec, Citrix covered the majority of the environment
- Built **normalization rules** for the remaining software not covered by publisher packs — sorted unrecognized installations by volume, prioritized the top 100 (which covered 80%+ of remaining unrecognized records), and created models and rules for each
- The normalization challenge at Dell was significant: SCCM returned over 600 distinct software name strings for what were actually about 120 unique products. Microsoft Office alone had 47 variants depending on version, channel, and install source
- Configured **automatic normalization re-runs** on a nightly schedule so newly discovered software was normalized within 24 hours of Discovery finding it

### 2.2 License Entitlement Setup

- Built **Entitlement records** for all enterprise software agreements — worked from the contract repository to enter the correct quantity, license metric, agreement terms, and end date for each agreement
- Handled complex **Microsoft EA entitlements**: mapped the EA coverage (Office, Windows, SQL Server, Exchange, SharePoint, System Center) to the correct Software Models and configured entitlement quantities by metric type (per-user CAL, per-device CAL, processor licenses)
- Configured **multi-metric entitlements** for Oracle Database where the same product can be licensed per-processor or per-named-user — set up separate entitlement records by metric type and configured the reconciliation to apply the correct metric per deployment environment
- Set up an **entitlement review schedule**: entitlement records with agreements expiring within 90 days triggered a review task to the software asset team so renewals were never missed

### 2.3 Reconciliation Schedules and Compliance Reporting

- Configured **weekly Reconciliation Schedules** for all software in scope — results were available every Monday morning for the software asset team review
- Built **compliance dashboards** in Performance Analytics: compliance position by vendor (over/under-licensed), trend over time, and top 10 compliance risks by financial exposure
- Created **executive summary reports** showing total software spend, compliance position, and reclamation pipeline — delivered automatically to the CIO office on the first Monday of each month
- Set up **threshold alerts**: when an under-licensed position exceeded 5% of the total entitlement for a major vendor (Microsoft, Oracle, Adobe), an automated task was created for the SAM team to investigate before it became an audit risk

### 2.4 Reclamation Workflow

- Configured the **Reclamation workflow** with 90-day no-usage threshold for standard software and 180-day threshold for project tools (Visio, AutoCAD, MS Project) — these tools are used periodically and a 90-day gap does not mean the user no longer needs them
- Set up **user notification emails** with clear subject lines and explicit consequence language — the notification explained that the software would be uninstalled if no response was received by a specific date
- Built a self-service **"I still need this" button** in the Employee Center that extended the reclamation hold by 6 months without requiring the user to contact the service desk
- Integrated the reclamation uninstall tasks with SCCM for automatic software removal on Windows endpoints — eliminated the manual uninstall step for the top 30 most commonly reclaimed applications

### 2.5 SaaS License Management

- Configured **SaaS License Management** for Microsoft 365, Zoom, Salesforce, Tableau, and Slack via IntegrationHub spokes — pulled actual usage data (last login date, active sessions, feature usage) from each vendor API nightly
- Built a **SaaS utilization dashboard** showing seat count vs. active users by application — this dashboard was reviewed by the software asset team monthly and by Finance quarterly for renewal negotiations
- Identified significant unused seat pools: Zoom had 18% of seats with no login in 60+ days, Tableau had 24% low-utilization seats — these findings directly informed renewal negotiations
- Set up **SaaS reclamation** process: monthly report of users with no SaaS login in 60+ days sent to their managers for confirmation, then seat de-provisioned if confirmed inactive

### 2.6 Publisher Pack Configuration (Deep Work)

- Did deep work on the **Microsoft Publisher Pack** — configured the EA coverage mapping, handled M365 license stacking rules (E3 vs. E5 includes/excludes), and set up the annual True-Up tracking report
- Configured the **Adobe Publisher Pack** for Creative Cloud licensing — handled the named user vs. shared device licensing distinction for creative labs vs. individual workstations
- Got into **Oracle Publisher Pack** configuration for Oracle Database on VMware — had to correctly configure the VMware processor factor tables and understand Oracle's hard partition vs. soft partition rules because getting this wrong would have resulted in a massive compliance exposure

---

## 3. CHALLENGES FACED

### Challenge 1: Normalization Backlog — 600+ Unrecognized Software Name Variants

**Problem:** When we first ran SAM Pro against our SCCM data, the normalization dashboard showed over 600 distinct unrecognized software name strings. The reconciliation schedule was producing near-zero results because most installations had no Software Model assigned.

**Solution:** Installed all available Publisher Packs immediately — this handled about 65% of installations automatically. For the remaining 35%, sorted unrecognized software by installation count and ran a normalization sprint over 3 weeks where the SAM team and I created models and rules for the top 150 products by volume. After the sprint, normalization coverage was above 92% and reconciliation results were trustworthy enough to share with leadership.

---

### Challenge 2: Entitlement Data Accuracy — Finance and IT Had Different Numbers

**Problem:** When we first built Entitlement records using data from the IT procurement team, Finance reported that our numbers were wrong. Their ERP system showed different quantities for 8 major software agreements. This created a credibility problem for SAM Pro before we had even run the first reconciliation.

**Solution:** Set up a joint reconciliation session with IT and Finance: pulled every contract from the legal repository, validated quantities against the master agreement documents (not just POs or renewals), and reconciled against the ERP data. Found three sources of discrepancy: expired maintenance agreements still being counted in ERP but correctly excluded from SAM Pro, licenses allocated to a divested business unit that IT had removed but Finance had not, and a purchase order that had been split across two fiscal years with only one half entered in SAM Pro. After the session, Finance and IT signed off on the SAM Pro entitlement data as the authoritative source.

---

### Challenge 3: Oracle Virtualization Licensing — Getting the Rules Right

**Problem:** We had Oracle Database running on VMware clusters. Oracle's licensing rules for virtualized environments are extremely complex — Oracle does not recognize VMware soft partitioning, which means licenses are counted based on all physical processors in the VMware cluster, not just the VMs running Oracle. Getting this wrong in SAM Pro would have either over-reported compliance (exposing us to audit liability) or under-reported (making us look non-compliant when we weren't).

**Solution:** Worked with the Oracle Publisher Pack documentation and with our Oracle licensing specialist to configure the processor factor tables correctly for each VMware cluster running Oracle. Set up a separate Discovery schedule specifically for the Oracle Database VMs that captured the physical host processor count (not the VM CPU count). Built a custom Oracle compliance report that showed the correct processor-based license count for each cluster separately, which the software asset team validated against the Oracle-provided LMS scripts before we trusted the SAM Pro numbers.

---

### Challenge 4: SaaS API Token Expiry Breaking Usage Data Sync

**Problem:** Three months after go-live, the SaaS utilization dashboard for Salesforce started showing stale data. The last successful sync was 3 weeks prior. No one had noticed because there were no alerts on SaaS integration failures.

**Solution:** Added monitoring on all SaaS integration scheduled imports: if a scheduled job had not completed successfully in 48 hours, an alert was sent to the SAM team. Fixed the immediate Salesforce issue by refreshing the OAuth token (it had expired on a 90-day rotation that we had not calendared). Then documented the token rotation schedule for every SaaS integration and set calendar reminders 2 weeks before each expiry. Also created a dedicated integration service account for each SaaS connection so token management was not dependent on a specific employee's credentials.

---

### Challenge 5: Reclamation Program — User and Manager Resistance

**Problem:** When we launched the reclamation program, several managers pushed back strongly — they saw the reclamation notifications as IT "taking away" software from their teams without understanding the business need. A VP sent an escalation to the CIO after one of their team members had Visio reclaimed mid-project.

**Solution:** Two changes. First, added a 180-day threshold for Visio, AutoCAD, MS Project, and other project tools (up from 90 days) — these tools are legitimately used periodically and 90 days was too aggressive. Second, added a manager notification (not just user) for any reclamation affecting someone in their team, with a 10-day window for the manager to respond and retain the software. The VP's escalation actually helped: it got leadership attention on the reclamation program and resulted in a published policy that gave the program legitimacy — managers understood it was a cost control measure, not arbitrary IT action.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **Software Model** | Canonical definition of a software product — the linking entity between installations and entitlements |
| **Publisher Pack** | Pre-built normalization and license metric content for major publishers from ServiceNow Content Library |
| **Normalization** | Mapping raw software name strings from Discovery to canonical Software Models |
| **Entitlement** | What is purchased — quantity, metric, terms, agreement reference |
| **Reconciliation** | Comparing installed count to entitlement quantity to produce compliance position |
| **SaaS License Management** | Tracking actual usage of cloud application seats via vendor API — identifies unused subscriptions |
| **Reclamation** | Identifying unused licenses and recovering them through automated uninstall or de-provisioning |
| **Compliance Position** | Over-licensed (cost savings opportunity), Under-licensed (audit risk), Compliant |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Normalization coverage improved from near-zero to **92%+ within 3 weeks** of publisher pack deployment and normalization sprint
- First reconciliation identified under-licensed position across 4 products — addressed proactively before next vendor True-Up cycle
- Reclamation program: **8-12% of software licenses recovered** in first year — significant cost savings on renewals
- SaaS unused seat identification: **18-24% of SaaS seats** flagged as reclamation candidates across monitored applications
- License compliance position: shifted from reactive audit-driven to **proactive weekly reconciliation** with automated alerts
- Oracle Database compliance validated before LMS audit — **zero penalty exposure** from the Oracle compliance review
