# ServiceNow ITAM — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is ITAM and what does it cover?**
**A:** IT Asset Management (ITAM) is a centralized solution for tracking, controlling, and optimizing IT assets across their entire lifecycle. It covers:
- **HAM:** Hardware Asset Management — physical devices
- **SAM:** Software Asset Management — license compliance
- **Cloud ITAM:** Cloud resource tracking and cost optimization
- **Mobile:** Mobile device management

---

**Q2: What is the difference between HAM, SAM, and ITAM?**
**A:**
- **ITAM** = umbrella term for all asset management disciplines
- **HAM** = managing physical hardware (servers, laptops, printers) from purchase to disposal
- **SAM** = managing software licenses, subscriptions, and compliance (entitlements vs. usage)

Key analogy: ITAM is the department. HAM and SAM are the teams within it.

---

**Q3: What are the 6 stages of the asset lifecycle?**
**A:**
1. **Planning** — Identify need, budget, vendor selection
2. **Procurement** — Purchase order, vendor engagement
3. **Receiving** — Asset received, record created, serial logged
4. **Deployment** — Assigned to user/location, CI linked in CMDB
5. **Maintenance** — Support, repairs, upgrades
6. **Retirement** — Data wipe, disposal, financial write-off

---

**Q4: What is the difference between an Asset and a CI?**
**A:**
- **Asset** (`alm_asset`): Financial/inventory record — tracks cost, ownership, depreciation
- **CI** (`cmdb_ci`): Logical/infrastructure record — tracks relationships, dependencies, status

A server has BOTH — they are linked. Asset record = "what we paid for it." CI record = "what it runs and who depends on it."

---

**Q5: What are Publisher Packs and why are they important?**
**A:** Publisher Packs are pre-built license models and normalization rules for major software vendors (Microsoft, Oracle, IBM, Adobe, SAP). They include:
- Predefined license metric templates
- Normalization rules for product name standardization
- Compliance calculation logic

**Why important:** Without them, teams manually build compliance rules for every vendor. Publisher Packs automate ~80% of SAM setup for major vendors and reduce time-to-value significantly.

---

**Q6: What is Software Normalization and why is it needed?**
**A:** Software normalization standardizes product names discovered across the environment. Example:
- "MS Office 365" + "Microsoft Office365" + "Office 365 E3" → all normalize to one product record

**Why needed:** Without normalization, compliance calculations are meaningless — you'd show 3 separate products requiring 3 separate entitlements when they're all the same thing.

---

**Q7: What is the License Workbench?**
**A:** The License Workbench is the central SAM dashboard providing:
- Compliance status per software product
- Usage vs. entitlements (is the company over or under-licensed?)
- Optimization recommendations (reclaim unused licenses)
- Audit readiness indicators

---

**Q8: What is the compliance position formula?**
**A:**
```
Compliance Position = Entitlements Owned − Actual Usage
  Positive number = Over-licensed (wasted money — optimize)
  Zero            = Perfectly compliant
  Negative number = Under-licensed (AUDIT RISK — buy more)
```

---

**Q9: What is an Asset Task?**
**A:** Asset Tasks are workflow-driven actions triggered at each lifecycle stage:
- **Receiving task:** Verify asset, update serial, store in stockroom
- **Deployment task:** Assign to user, update location, link to CMDB CI
- **Repair task:** Route to IT support, track repair
- **Retirement task:** Data wipe, remove from service, financial write-off

---

**Q10: What role does Discovery play in ITAM?**
**A:** Discovery automatically scans the network to:
1. Find hardware devices → creates/updates Hardware Asset records
2. Find installed software → feeds SAM normalization
3. Populate CMDB with CI records linked to assets
4. Keep records current without manual entry

Without Discovery, ITAM relies on manual entry and periodic audits — error-prone and always lagging behind reality.

---

## PART 2: SAM Deep Dive

**Q11: What are common software license types in SAM?**
**A:**
| Type | Description |
|------|-------------|
| Named User | One specific user per license |
| Concurrent | X users simultaneously |
| Device-based | Per device, not user |
| Enterprise Agreement | Site-wide or org-wide |
| Subscription/SaaS | Monthly/annual per user |
| OEM | Bundled with hardware |

---

**Q12: What is the SAP or Oracle licensing challenge in SAM?**
**A:** Oracle and SAP licenses are notoriously complex:
- **Oracle:** "Processor" licenses — must count all processors the software *could* run on, not just where it *does* run. In virtualized environments this is extremely difficult.
- **SAP:** Named user + engine metrics; indirect access (connecting non-SAP systems to SAP can trigger additional licensing)
- Publisher Packs for these vendors help automate the calculation, but expert review is still needed

---

**Q13: How does SaaS subscription tracking work in ITAM?**
**A:** ITAM tracks SaaS through:
- Integration with Identity Providers (Okta, Azure AD) for user activity data
- Usage APIs from SaaS vendors
- Monitoring login frequency to identify unused seats (reclaim licenses)
- Contract management for renewal tracking and cost optimization

---

**Q14: What is the Software Asset Workspace?**
**A:** The Software Asset Workspace provides a unified view for SAM professionals:
- Compliance status across all software products
- Usage analysis and optimization actions
- Reconciliation workflows
- Vendor audit readiness

---

## PART 3: HAM Deep Dive

**Q15: What is the Hardware Asset Workspace?**
**A:** The Hardware Asset Workspace is a unified interface for managing the complete physical asset lifecycle:
- Asset tracking dashboards
- Processing asset tasks (receive, deploy, repair, retire)
- Stockroom management
- Inventory audit support

---

**Q16: What is a Stockroom in ITAM?**
**A:** A Stockroom is a physical storage location tracked in ServiceNow for assets that aren't currently assigned to a user:
- New assets waiting deployment
- Returned assets being refurbished
- Spare parts inventory
- Assets pending retirement/disposal

Multiple stockrooms can be defined (e.g., by location/site).

---

**Q17: How does ITAM handle End-of-Life (EOL) assets?**
**A:**
1. ITAM tracks EOL/EOM dates for hardware and software
2. Reports surface assets approaching EOL 90/60/30 days in advance
3. Refresh projects are created in SPM for replacement planning
4. Budget forecasting uses EOL data to predict hardware refresh costs
5. Vulnerable EOL assets are flagged for SecOps prioritization

---

## PART 4: Integrations & Scenarios

**Q18: How does ITAM integrate with HR Service Delivery for onboarding/offboarding?**
**A:**
- **Onboarding:** Employee hire triggers a ServiceNow request → ITAM auto-assigns laptop, software licenses, and peripherals from stockroom → Day-1 readiness
- **Offboarding:** Employee departure triggers asset recovery workflow → IT picks up hardware → licenses reclaimed → data wiped → asset returned to stockroom or retired

---

**Q19: How does ITAM support SecOps?**
**A:** ITAM provides SecOps with:
- Asset ownership (who to contact for remediation)
- Lifecycle status (is this a production asset or a test machine?)
- EOL status (EOL assets = higher vulnerability priority)
- Location and criticality data for business impact scoring

Without ITAM data, SecOps can find vulnerabilities but can't efficiently prioritize remediation.

---

**Q20: A software vendor audit letter arrives. How does ITAM help?**
**A:**
1. Pull License Workbench report showing entitlements vs. actual usage
2. Review compliance position for the specific vendor
3. Identify any shortfalls and calculate exposure
4. Pull historical procurement data to verify license purchases
5. Run software normalization to ensure all installations are captured
6. If shortfall found: negotiate true-up before audit findings or purchase additional licenses
7. If positive compliance position: provide evidence to auditor with documentation

---

**Q21: What is the difference between ITAM and ITSM in ServiceNow?**
**A:**
| Aspect | ITAM | ITSM |
|--------|------|------|
| Focus | Asset tracking and optimization | Service delivery and incident/change/problem |
| Primary question | "What do we own? Is it compliant?" | "What's broken? How do we fix it?" |
| Key tables | alm_asset, alm_hardware, alm_license | incident, change_request, problem |
| Main output | Compliance reports, cost optimization | Resolved tickets, service uptime |
| Integration | Assets linked to ITSM CIs and tickets | Incidents reference CIs (from ITAM) |

---

## PART 5: Exam-Style Quick Facts

| Question | Answer |
|----------|--------|
| Hardware asset table | `alm_hardware` |
| Software license table | `alm_license` |
| Base asset table | `alm_asset` |
| Asset task table | `alm_asset_task` |
| HAM = | Hardware Asset Management |
| SAM = | Software Asset Management |
| Compliance Position formula | Entitlements − Usage |
| Over-licensed = | Wasted money (positive position) |
| Under-licensed = | Audit risk (negative position) |
| Publisher Pack = | Pre-built vendor license models |
| Software normalization = | Standardizing product names |
| License Workbench = | Central SAM compliance dashboard |

---

## Study Checklist
- [ ] Know: HAM vs SAM vs ITAM definitions clearly
- [ ] Know: 6 lifecycle stages in order
- [ ] Know: Asset vs CI distinction
- [ ] Know: Publisher Packs purpose
- [ ] Know: Compliance Position formula
- [ ] Know: Software normalization and why it's needed
- [ ] Know: Stockroom concept
- [ ] Know: How Discovery feeds ITAM
- [ ] Know: HRSD integration for onboarding/offboarding
- [ ] Know: How ITAM supports vendor audit defense
