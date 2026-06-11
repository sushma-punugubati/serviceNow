# ServiceNow SAM Pro — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is SAM Pro and what problem does it solve?**
**A:** SAM Pro (Software Asset Management Professional) is ServiceNow's module for continuously managing software license compliance. It solves the problem of organizations only discovering license compliance gaps during vendor audits — which is expensive and reactive.

SAM Pro continuously compares *installed* software (from Discovery/SCCM/Intune) against *licensed* software (from purchase orders and contracts) and surfaces compliance risks weekly. It also identifies unused licenses for reclamation, reducing costs.

**The business impact:** A single unplanned Microsoft True-Up or Oracle audit can cost millions in back-payment. SAM Pro turns that risk into a managed, weekly process.

---

**Q2: What is a Software Model?**
**A:** A Software Model is the canonical definition of a software product in SAM Pro. It serves as the key linking entity between what is *installed* (software installation records) and what is *licensed* (entitlement records).

Key attributes: Publisher, Product Name, Version, License Metric.

Without Software Models, you cannot reconcile — Discovery returns raw software names like "Microsoft Office 365 ProPlus 2019 (x64)" but the license is purchased as "Microsoft 365 Apps for Enterprise." The Software Model normalizes these to a single entity so comparison is possible.

---

**Q3: What is Software Normalization and why does it matter?**
**A:** Software Normalization is the process of mapping the many raw software names returned by Discovery to canonical Software Models.

**Why it matters:** A single software product can appear with hundreds of different names in Discovery data depending on version, installer source, operating system, and how the publisher named it at install time. Without normalization:
- The same product has 50 entries in your software catalog
- Reconciliation counts are wrong because installations don't match to models
- Compliance position is inaccurate

**How it works in ServiceNow:**
1. Publisher Packs provide pre-built normalization rules for major publishers
2. Custom normalization rules can be created for software not covered by publisher packs
3. The normalization engine matches raw names against rules and assigns the correct Software Model

---

**Q4: What are Publisher Packs?**
**A:** Publisher Packs are pre-built sets of Software Models provided by ServiceNow in the Content Library for major software publishers (Microsoft, Adobe, Oracle, IBM, SAP, Citrix, VMware, etc.).

**What they include:**
- Pre-normalized Software Models with correct publisher/product/version data
- License Metric definitions aligned with the publisher's actual licensing rules
- Manufacturer Part Numbers for PO matching
- Complex license metric handling (e.g., Microsoft CAL types, Oracle processor factors)

**Why they matter:** Building accurate Software Models and normalization rules for complex publishers like Microsoft or Oracle from scratch requires deep licensing expertise. Publisher Packs provide this out of the box, significantly reducing implementation effort and risk of error.

---

**Q5: What is a License Entitlement?**
**A:** A License Entitlement record captures what an organization has purchased for a specific Software Model — the quantity, metric, terms, and agreement reference.

Key attributes: Software Model, Quantity, License Metric (per user / per device / per processor), Entitlement Type (perpetual / subscription / OEM), Start Date, End Date, Agreement link.

Entitlements are the "what we own" side of reconciliation. Without accurate entitlements, SAM Pro cannot calculate whether the organization is compliant.

---

**Q6: What are the common License Metrics in SAM Pro?**
**A:**
| Metric | Description | Example |
|--------|-------------|---------|
| Per Named User | One license per authorized user | Office 365 per-user |
| Per Device | One license per installed device | Windows per device CAL |
| Per Concurrent User | Max simultaneous users | Citrix concurrent licensing |
| Per Processor | Based on physical processor count | SQL Server per processor |
| Per Core | Based on CPU core count | Oracle Database per core |
| Subscription (SaaS) | Monthly/annual per seat | Salesforce, Zoom, Slack |

Understanding license metrics is critical — the same software can have completely different compliance calculations depending on which metric is in the agreement.

---

**Q7: What is Software Reconciliation?**
**A:** Software Reconciliation is the process that compares installed license counts against entitlement quantities and calculates the compliance position for each Software Model.

**Process:**
1. Reconciliation Schedule runs (typically weekly)
2. For each Software Model: count installations in CMDB (from Discovery)
3. Compare against Entitlement quantity for the same model
4. Calculate variance: Installed - Entitled = Compliance Position

**Results:**
- **Compliant:** Installations ≤ Entitlements
- **Under-licensed:** More installed than purchased — compliance risk, potential audit exposure
- **Over-licensed:** More purchased than installed — cost optimization opportunity (reduce on next renewal)
- **Unreconciled:** No entitlement found for installed software

---

**Q8: What is SaaS License Management in SAM Pro?**
**A:** SaaS License Management extends SAM Pro to cloud applications where Discovery cannot see installations because the software runs in a browser, not as a local install.

**How it works:**
- Integrates with SaaS vendor APIs via IntegrationHub spokes (Salesforce, Zoom, Slack, Microsoft 365, Tableau, etc.)
- Pulls actual usage data: last login date, active sessions, features accessed
- Compares usage against purchased subscription seats
- Identifies unused seats (no login in 60+ days) for reclamation or seat reduction on renewal

**Key insight:** Unlike on-premise software where "installed = in use," SaaS subscriptions can be assigned but never used. Paying for seats that employees never log into is pure waste — SaaS License Management finds these automatically.

---

**Q9: What is the Reclamation process?**
**A:** Reclamation is the automated process of identifying unused software licenses and recovering them for reassignment or contract reduction.

**Workflow:**
1. SAM identifies software installations with no usage signal for X days (configurable, typically 60-90 days)
2. Notification to the assigned user and manager: "This software appears unused — do you still need it?"
3. User response options: "I still need it" (suppresses for 6 months) or no response
4. After the response window closes: automated uninstall task created in ITSM
5. On confirmed uninstall: license returned to available pool

**Business value:** Enterprise organizations typically have 15-25% of software licenses in "installed but unused" state. Reclaiming these reduces renewal costs and compliance exposure.

---

**Q10: What is the difference between an Entitlement and an Allocation?**
**A:**
- **Entitlement:** The total licenses purchased — what the organization owns. "We have 1,000 Office licenses."
- **Allocation:** The assignment of an entitlement to a specific user, device, or cost center. "500 of those 1,000 licenses are allocated to the Finance team."

Allocations support chargeback/showback scenarios where different business units are billed for their software usage. They also support multi-site or multi-subsidiary deployments where the enterprise entitlement needs to be distributed.

---

## PART 2: Configuration and Implementation

**Q11: How does Discovery feed into SAM Pro?**
**A:** Discovery (via ServiceNow Discovery, SCCM, or Intune via MID Server) populates Software Installation records in the CMDB table `cmdb_sam_sw_install`. Each record links:
- The software product name (raw, pre-normalization)
- The CI it is installed on
- Installation date, version

SAM Pro then runs normalization rules against the raw software names to assign Software Models to each installation. The normalized, model-linked installations are what the Reconciliation Schedule uses for the compliance calculation.

**Important:** Discovery data quality directly impacts SAM Pro accuracy. If SCCM is not scanning all devices, or if some device types are not covered by SCCM agents, there will be blind spots in the installation data.

---

**Q12: How would you handle a complex Microsoft enterprise agreement in SAM Pro?**
**A:** Microsoft EAs (Enterprise Agreements) are among the most complex to implement in SAM Pro due to the breadth of products and license types:

1. **Install the Microsoft Publisher Pack** — provides pre-built models for all Microsoft products with correct license metrics
2. **Map the EA terms:** Identify the covered products, license quantities, and metric type (per user vs. per device for CAL products)
3. **Configure Entitlement records** for each product family (Office, Windows, SQL Server, etc.) with correct quantities
4. **Handle CAL complexity:** Windows Server CALs, SQL CALs — each has per-user and per-device variants with different counting rules. Publisher pack handles the logic.
5. **Set up True-Up tracking:** EAs typically have an annual True-Up — configure a scheduled reconciliation and report to prepare for True-Up before the Microsoft review date
6. **Configure M365 SaaS tracking** via the M365 IntegrationHub spoke for cloud seat usage

---

**Q13: What is the Content Library in SAM Pro?**
**A:** The Content Library is ServiceNow's managed repository of pre-built Software Models, normalization rules, and Publisher Packs. It is continuously updated by ServiceNow as new software products are released.

**How to use it:**
- Navigate to Software Asset > Content Library
- Search for a publisher or product
- Subscribe to publisher packs that are relevant to your organization
- Models and normalization rules are automatically downloaded and applied

**Why it matters:** Without the Content Library, organizations must manually create and maintain hundreds or thousands of Software Models. The Content Library handles the ongoing maintenance of major publisher models, reducing the SAM team's maintenance burden significantly.

---

**Q14: How does SAM Pro handle software on virtual machines and Citrix/Terminal Server environments?**
**A:** Virtual and shared environments require specific handling because a single physical server may host many users running the same software via virtualization.

**RDS/Terminal Server (Windows Server):** Software installed on an RDS server is accessible to all connected users. SAM Pro needs to count **concurrent users** (not installations) if the license metric is per-concurrent-user, or **named users** with RDS access rights if per-user metric.

**Citrix virtual apps:** Similar to RDS — one installation serves many users. If the license is per-device and users access via thin clients, the thin client count matters, not the server count.

**Key configuration:** Set the CI's "Virtual" flag correctly in CMDB and configure Software Models with the correct metric for virtual environments. The Microsoft Publisher Pack specifically handles Windows Server RDS licensing rules.

---

**Q15: Walk me through the steps you would take if SAM Pro showed a 20% under-licensed position for Adobe Acrobat.**
**A:**
1. **Verify the entitlement data:** Check the Entitlement record for Adobe Acrobat — confirm the quantity is correct and the agreement is current. An expired entitlement would incorrectly show as under-licensed.
2. **Verify the installation data:** Review the installation records contributing to the count. Are there clearly miscategorized entries (e.g., Adobe Reader being counted as Acrobat Pro due to a normalization error)?
3. **Check normalization rules:** Confirm that the normalization rules are not pulling in installation records that should not map to this model.
4. **Validate license metric:** Confirm the agreement is per named user. If it is per-device, recalculate — the device count may differ from the user count.
5. **Calculate actual exposure:** If the under-licensed position is confirmed, calculate the cost to become compliant vs. the risk of an audit.
6. **Remediation options:** Purchase additional licenses, reclaim unused installations (run usage analysis first), or pursue a contract renegotiation.
7. **Document:** Record the finding, root cause, and remediation plan in an IRM risk record if the exposure exceeds the risk threshold.

---

## PART 3: Scenarios and Advanced Topics

**Q16: How do you handle software that has no Publisher Pack and no normalization rule?**
**A:**
1. Query the `cmdb_sam_sw_install` table for all unrecognized software names (Software Model = null)
2. Sort by installation count — address high-volume unrecognized software first
3. For each unrecognized product: manually create a Software Model with the correct publisher, product name, and license metric
4. Create a normalization rule mapping the raw name variants to this model
5. Re-run normalization for the affected installations
6. For software with no license agreement: create an Entitlement with quantity 0 — this will show as under-licensed, making the risk visible

---

**Q17: What happens to SAM Pro data during a ServiceNow upgrade?**
**A:** SAM Pro data (entitlements, reconciliation results, software models) is stored in the database and survives upgrades. The main risks are:
1. **Publisher Pack compatibility:** Major upgrades may require Publisher Pack updates to align with new platform features. Subscribe to Publisher Pack update notifications.
2. **Custom normalization rules:** Validate that custom rules still apply correctly post-upgrade — sometimes rule priority or matching logic changes between versions.
3. **Scheduled reconciliation:** Verify that Reconciliation Schedules are still active and configured correctly post-upgrade.
4. **IntegrationHub spokes for SaaS:** Spoke versions may need updating to maintain SaaS API compatibility after upgrade.

---

**Q18: How is SAM Pro licensed and what is the difference between SAM Foundation and SAM Pro?**
**A:**
| SAM Foundation | SAM Pro |
|----------------|---------|
| Basic software inventory and tracking | Full compliance management |
| Manual model creation only | Publisher Packs and Content Library |
| No automated reconciliation | Automated Reconciliation Schedules |
| No reclamation workflows | Full reclamation automation |
| No SaaS License Management | SaaS License Management included |
| Included with most ServiceNow licenses | Separate licensed product |

SAM Foundation gives you the data structure. SAM Pro gives you the automation, compliance intelligence, and publisher content that makes SAM actionable at enterprise scale.

---

**Q19: What is a Software Usage Right?**
**A:** A Software Usage Right defines the specific conditions under which an entitlement can be used — it extends the entitlement with rules about:
- **Geographic restrictions:** License is valid only in specific countries
- **Metric downgrade rights:** License for the full edition can be used for a lesser edition
- **Second use rights:** Licensed user can install on a second device (e.g., laptop + home PC)
- **Virtual environment rights:** Whether the license covers virtual instances and under what conditions

Usage Rights are particularly important for Microsoft licensing, where the EA terms define complex usage rights that affect how many licenses are actually needed.

---

**Q20: How do you build the business case for SAM Pro?**
**A:** The business case typically rests on three pillars:

1. **Audit risk avoidance:** Calculate the potential exposure from the top 5 software vendors. Microsoft, Oracle, IBM, and SAP audits routinely result in settlements of $1M-$50M+ for enterprise organizations. Even reducing audit exposure by 50% justifies the investment.

2. **License reclamation savings:** Enterprise organizations typically have 15-25% of licenses in unused or over-provisioned state. On a $10M annual software spend, 15% reclamation = $1.5M in renewal cost avoidance.

3. **Operational efficiency:** Without SAM Pro, a SAM team of 5 people spends most of their time manually tracking licenses in spreadsheets. SAM Pro automates the data collection and comparison, freeing the team for higher-value activities like contract negotiation and vendor management.
