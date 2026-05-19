# ServiceNow ITAM — Customer Success Stories

Real-world examples of organizations using ServiceNow IT Asset Management to control costs, maintain compliance, and eliminate audit risk.

---

## Story 1: Global Bank — Surviving a Microsoft Licensing Audit

**Industry:** Financial Services  
**Challenge:** A major international bank received an unexpected Microsoft audit letter. With 85,000 employees across 40 countries, their software inventory was tracked in 14 different spreadsheets maintained by regional IT teams. They had no unified view of Microsoft installations vs. entitlements — and 90 days to respond to the audit.

**What ITAM Solved:**
- ServiceNow Discovery scanned 85,000 endpoints in 3 days — first time the bank had seen a complete software inventory
- Software Normalization consolidated 47 variations of "Microsoft Office" into standardized product records
- License Workbench calculated actual compliance position — turns out they were over-licensed by 12,000 seats (wasted $4.2M/year)
- Publisher Packs for Microsoft automated the complex License SA (Software Assurance) calculations

**Results:**
- Audit passed — bank provided clean evidence in 60 days, 30 days ahead of deadline
- Discovered they owned more licenses than used — turned an audit threat into a negotiation opportunity (renegotiated Microsoft EA for 18% cost reduction)
- Annual Microsoft spend reduced by $4.2M by reclaiming and not renewing unused licenses
- "Audit readiness" posture established — next audit will take days, not emergency weeks

**Key ITAM Features Used:** SAM, Software Discovery, Software Normalization, Publisher Packs (Microsoft), License Workbench

---

## Story 2: Healthcare System — Hardware Asset Tracking Across 40 Hospitals

**Industry:** Healthcare  
**Challenge:** A healthcare network operating 40 hospitals and 200+ clinics had no reliable count of its physical IT assets. Laptops disappeared from clinical areas, medical workstations ran end-of-life Windows versions, and the IT team discovered assets in the field that had been written off financially years earlier. Every compliance audit flagged missing asset records.

**What ITAM Solved:**
- HAM deployed with Hardware Asset Workspace — all 95,000 physical assets tracked from receipt to retirement
- Asset Task workflows automated: receiving → deployment → transfer → retirement
- Stockroom management created per hospital — spare devices available instantly without emergency procurement
- EOL tracking integrated with SecOps — assets running EOL Windows versions automatically flagged for priority patching

**Results:**
- 100% of clinical devices now tracked — for the first time since the network was formed
- "Ghost assets" (written off but still in use): 3,200 found — $8.1M in assets recovered
- EOL asset refresh completed before HIPAA audit — zero findings related to unsupported OS versions
- Device refresh cycle automated — IT now knows 90 days in advance which devices are due for replacement

**Key ITAM Features Used:** HAM, Hardware Asset Workspace, Asset Tasks, Stockroom Management, EOL Tracking

---

## Story 3: Retail Chain — Eliminating Oracle License Compliance Risk

**Industry:** Retail  
**Challenge:** A national retail chain with 1,200 stores ran Oracle databases across on-premises data centers and some instances had migrated to virtualized environments. Oracle's "Processor" licensing model requires counting all processors in a virtualized cluster — not just where the database runs. The company had no idea what their actual Oracle footprint was and feared a surprise audit.

**What ITAM Solved:**
- SAM with Oracle Publisher Pack automatically calculated processor-based license requirements including virtualization factors
- Discovery identified 14 Oracle instances running on shared VMware clusters — meaning ALL processors in those clusters counted
- License Workbench showed they were under-licensed by 47 Oracle Database Enterprise Edition licenses (exposure: $6.8M)
- Remediation plan: migrate 3 Oracle workloads off shared clusters to dedicated hosts → reduced license count by 29

**Results:**
- Under-licensing gap closed before Oracle audit — purchased only what was truly needed ($2.1M true-up vs. $6.8M exposure if audited)
- Oracle infrastructure architecture changed proactively — future licensing costs reduced by 31%
- Full Oracle compliance documentation now maintained continuously in ServiceNow
- Internal audit committee satisfied for first time that Oracle risk was under control

**Key ITAM Features Used:** SAM, Oracle Publisher Pack, Discovery (Virtualization-aware), License Workbench

---

## Story 4: Technology Company — SaaS License Rationalization and Cost Recovery

**Industry:** Technology / Software  
**Challenge:** A 5,000-person software company had organically adopted 340+ SaaS applications through department-level credit card purchases. Finance had no idea what was being paid for, HR couldn't enforce application standards, and IT had no way to reclaim licenses when employees left.

**What ITAM Solved:**
- Cloud ITAM integrated with Okta (Identity Provider) to discover ALL SaaS applications in use
- Usage data from Okta combined with license cost data — calculated cost-per-active-user for each app
- Offboarding workflow (integrated with HRSD) automatically triggered license reclaim when employees departed
- 340 apps rationalized into categories: Keep, Consolidate, Eliminate

**Results:**
- SaaS spend reduced by $3.4M annually:
  - 47 apps eliminated (duplicated by other tools already owned)
  - 18 apps consolidated (e.g., 4 different project management tools → 1 approved standard)
  - 12,000+ unused seats reclaimed ($1.8M saved on renewal)
- 100% of departed employees have SaaS access revoked within 1 hour (previously: average 3 weeks)
- IT now approves SaaS purchases before credit cards are used — policy enforced through ServiceNow catalog

**Key ITAM Features Used:** Cloud ITAM, SaaS Discovery (Okta integration), HRSD Integration, Software Catalog

---

## Story 5: Government Agency — IT Asset Auditability for Federal Compliance

**Industry:** Government / Public Sector  
**Challenge:** A federal agency faced recurring GAO audit findings for "inability to account for IT assets." The agency had $400M in IT assets on its books but field auditors found assets that weren't in any system, assets assigned to employees who had left years ago, and assets in data centers with no ownership record.

**What ITAM Solved:**
- HAM deployed with complete asset lifecycle workflows — every asset from procurement to disposal tracked
- Integration with HR system: when employee departs → automatic asset recovery task created for IT
- Barcoding solution integrated with ServiceNow Mobile — field auditors scan assets, ITAM records update in real-time
- Financial reconciliation: ServiceNow ITAM asset list matched to agency financial system — discrepancies flagged automatically

**Results:**
- GAO audit finding closed — zero asset accountability findings in next review cycle
- $42M in assets previously unaccounted for reconciled (most recovered, some officially written off with proper documentation)
- Asset inventory accuracy went from 62% to 99.7%
- Annual physical inventory audit (previously took 6 weeks with 30 staff) now completed in 4 days with 8 staff using mobile scanning

**Key ITAM Features Used:** HAM, Asset Lifecycle Workflows, HR Integration, Mobile Asset Management, Financial Reconciliation

---

## Story 6: Insurance Company — Automating End-to-End Employee Hardware Lifecycle

**Industry:** Financial Services / Insurance  
**Challenge:** A 12,000-employee insurance company's employee hardware experience was broken in both directions. New employees waited an average of 8 days for their laptop (manual provisioning by IT). Departing employees returned hardware to a shared mailroom with no tracking — 15% of assets were never returned.

**What ITAM Solved:**
- HRSD + ITAM integration: Employee hire event automatically triggers laptop provisioning from pre-stocked stockroom
- Stockroom management: IT maintains "pre-configured gold images" — day-1 ready laptops in regional stockrooms
- Offboarding: Employee departure triggers asset return kit shipping, then auto-creates follow-up tasks if return not confirmed
- Asset transfer tracking: every device movement logged with timestamp and responsible party

**Results:**
- New employee hardware ready: Day 1 (down from average 8 days)
- Asset return rate: 99.2% (from 85%)
- IT helpdesk tickets related to hardware procurement reduced 78%
- Annual hardware losses eliminated — saving $650K/year in unreturned device write-offs

**Key ITAM Features Used:** HAM, HRSD Integration, Stockroom Management, Asset Transfer Workflows, Asset Task Automation

---

## Common Themes Across ITAM Success Stories

| Theme | Why It Matters |
|-------|---------------|
| **Audit defense** | License audits (Microsoft, Oracle, SAP) are a universal enterprise fear — ITAM turns panic into confidence |
| **Shadow IT discovery** | Organizations consistently underestimate SaaS sprawl by 50-70% |
| **Offboarding automation** | License reclaim at departure is one of the fastest ROI wins in ITAM |
| **HAM + HRSD integration** | Employee lifecycle drives hardware lifecycle — they must be connected |
| **EOL asset visibility** | SecOps depends on ITAM knowing which assets are end-of-life |
| **Financial reconciliation** | Government and public sector require asset-to-financial-system matching |

---

## Key Metrics ITAM Customers Report

| Metric | Typical Result |
|--------|---------------|
| Software license savings (first year) | 15-30% of software spend |
| SaaS rationalization savings | 20-40% of SaaS budget |
| Asset return rate improvement | 85% → 98%+ |
| New employee hardware readiness | 8-day average → Day 1 |
| Audit audit time reduction | 70-90% faster |
| License audit findings | Zero (after ITAM implementation) |
