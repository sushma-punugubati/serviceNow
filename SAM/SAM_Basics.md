# ServiceNow SAM Pro — Fundamentals & Study Guide

> Sources: ServiceNow Docs | ServiceNow Community | SAM Pro Implementation Guide | Now Learning

---

## 1. What is SAM Pro?

**SAM Pro (Software Asset Management Professional)** is ServiceNow's module for managing the full lifecycle of software licenses. It continuously compares what software is *installed* (from Discovery) against what is *licensed* (from purchase orders and contracts) to give an organization an up-to-date, automated compliance position.

> **Think of it as:** A continuous audit that runs every week instead of once a year. Instead of finding out you have a compliance gap during a vendor audit, SAM Pro tells you every Monday morning exactly where you stand — and flags risks before they become expensive.

**The core problem SAM Pro solves:**
- Organizations typically have thousands of software titles installed across tens of thousands of devices
- License agreements are complex: per-user, per-device, per-processor, concurrent, subscription
- Compliance gaps are only discovered during vendor audits — by which point back-payment obligations can be enormous
- SAM Pro automates the comparison of installed vs. licensed and surfaces risks continuously

**SAM Pro vs. basic software tracking:**
| Basic software inventory | SAM Pro |
|--------------------------|---------|
| Lists what is installed | Lists what is installed AND compares to what is licensed |
| No normalization | Normalizes 400+ software name variants to canonical models |
| No compliance view | Continuous compliance position with over/under-licensed flags |
| No reclamation | Identifies unused licenses and automates recovery |
| Reactive (audit-driven) | Proactive (weekly reconciliation) |

---

## 2. Core Plugin

| Plugin | ID |
|--------|----|
| Software Asset Management Professional | `com.snc.sam.professional` |
| Software Asset Management Foundation | `com.snc.samp` |
| Asset Management | `com.snc.asset` |
| Discovery | `com.snc.discovery` |

---

## 3. The SAM Pro Process Flow

SAM Pro operates as a continuous cycle:

```
Discovery (what is installed)
        ↓
Normalization (map raw names to Software Models)
        ↓
Entitlement (what is purchased/licensed)
        ↓
Reconciliation (installed vs. licensed)
        ↓
Compliance Position (over/under-licensed)
        ↓
Action (reclaim unused, purchase more, or accept risk)
        ↓
(repeat weekly)
```

**Step 1 — Discovery:** The CMDB is populated with Software Installation records via SCCM, Intune, or ServiceNow Discovery. Each installation record captures: software product name, version, publisher, install date, and the CI it is installed on.

**Step 2 — Normalization:** Raw software names from discovery are mapped to canonical Software Models. The same product may appear as "Adobe Acrobat Pro 2020," "Acrobat Pro DC," "Adobe Systems Acrobat" — normalization maps them all to one model.

**Step 3 — Entitlement:** License Entitlement records capture what the organization has purchased: the Software Model, quantity, license metric, and agreement terms.

**Step 4 — Reconciliation:** SAM Pro runs a comparison (Reconciliation Schedule) that counts installations against entitlements per software model and calculates the compliance position.

**Step 5 — Compliance Position:** The result is a per-software-model compliance view: over-licensed (too many licenses purchased), under-licensed (more installed than purchased), or compliant.

---

## 4. Software Models

A **Software Model** is the canonical definition of a software product in ServiceNow. It is the key entity that ties installations to entitlements.

### Key attributes of a Software Model:
| Attribute | Description |
|-----------|-------------|
| Publisher | The software vendor (Microsoft, Adobe, Oracle) |
| Product Name | Canonical product name (Adobe Acrobat Pro) |
| Version | Specific version (2020, 2023, DC) or "All versions" |
| License Metric | How the license is counted (per user, per device, per processor) |
| Manufacturer Part Number | Used for matching to publisher pack data |

### Software Model Catalog
The **Software Model Catalog** is the list of all software models in the organization. Models can come from:
1. **Content Library / Publisher Packs** — pre-built models from ServiceNow for major publishers
2. **Manual creation** — created by the SAM team for internally developed or niche software
3. **Normalization rules** — automatically assigned when a software installation name matches a rule

---

## 5. Publisher Packs

**Publisher Packs** are pre-built sets of Software Models provided by ServiceNow in the Content Library for major software publishers.

Available publisher packs include: Microsoft, Adobe, Oracle, IBM, SAP, Symantec, Citrix, VMware, and others.

### What a Publisher Pack provides:
- Pre-normalized Software Models with correct publisher/product/version data
- Defined License Metrics aligned with the publisher's actual licensing rules
- Manufacturer Part Numbers for matching to PO line items
- Entitlement Metric definitions (Microsoft per-user CAL, per-device CAL, processor licenses, etc.)

### Why Publisher Packs matter:
Without publisher packs, a SAM team must manually create and maintain Software Models for major vendors whose licensing rules are complex. Microsoft alone has hundreds of license types. Publisher packs handle this complexity automatically.

---

## 6. License Entitlements and Metrics

A **License Entitlement** record captures what the organization has purchased for a specific Software Model.

### Key attributes:
| Attribute | Description |
|-----------|-------------|
| Software Model | Which product this entitlement covers |
| Quantity | Number of licenses purchased |
| License Metric | How usage is counted (per user, per device, per processor, concurrent) |
| Entitlement Type | Perpetual, Subscription, OEM, Volume |
| Agreement | Links to the contract/purchase order |
| Start/End Date | License validity period |

### Common License Metrics:
| Metric | How it works |
|--------|-------------|
| **Per Named User** | One license per specific user who is authorized to use the software |
| **Per Device** | One license per device where software is installed |
| **Per Concurrent User** | Maximum number of users using the software simultaneously |
| **Per Processor** | Licensed based on the number of physical processors on the server |
| **Per Core** | Licensed based on the number of CPU cores (common for Oracle, SQL Server) |
| **Subscription (SaaS)** | Monthly/annual per-seat subscription — usage tracked via vendor API |

---

## 7. Software Reconciliation

**Software Reconciliation** is the process that compares installed license count against entitlement quantity and calculates the compliance position.

### Reconciliation Schedule:
- Can be run on demand or on a schedule (typically weekly)
- Processes each Software Model in scope
- Calculates: Total Installed, Total Entitled, Variance (Installed - Entitled)
- Positive variance = over-installed (under-licensed) — **compliance risk**
- Negative variance = under-installed (over-licensed) — **cost optimization opportunity**

### Reconciliation Result States:
| State | Meaning |
|-------|---------|
| **Compliant** | Installations ≤ Entitlements |
| **Over-licensed** | More licenses purchased than installed — cost optimization opportunity |
| **Under-licensed** | More installed than purchased — compliance risk |
| **Unreconciled** | No entitlement found to compare against |

---

## 8. SaaS License Management

**SaaS License Management** extends SAM Pro to cloud/SaaS applications where traditional Discovery cannot count installations.

### How it works:
- Integrates with SaaS vendor APIs (Salesforce, Zoom, Slack, Microsoft 365, Tableau) via IntegrationHub spokes
- Pulls **actual usage data**: last login date, active sessions, features used
- Compares against **subscription seats** purchased
- Identifies unused seats (no login in 60+ days) as reclamation candidates

### Key distinction from on-premise SAM:
On-premise SAM asks: "Is the software installed?" SaaS SAM asks: "Is the subscription actually being used?" A seat can be assigned to a user but never logged into — that is wasted spend.

---

## 9. Reclamation

**Reclamation** is the process of identifying unused software licenses and recovering them for reassignment or contract reduction.

### Reclamation workflow:
1. SAM identifies installations with no usage for X days (configurable threshold, typically 60-90 days)
2. Notification sent to the assigned user and their manager
3. User can respond: "I still need this" (suppresses reclamation for 6 months) or no response
4. If no response after defined period, automated uninstall request created in ITSM
5. License recovered and added back to available entitlement pool

### Why reclamation matters:
Enterprise organizations typically have 15-25% of their software licenses in "installed but unused" state. Reclaiming even a fraction of these represents direct cost savings on renewal negotiations.

---

## 10. Key Tables Reference

| Table | Label | Purpose |
|-------|-------|---------|
| `cmdb_sam_sw_product` | Software Product | Software Models |
| `samp_entitlement_allocation` | Software Entitlement | License entitlements |
| `samp_sw_install` | Software Installation | Discovered installations |
| `samp_reconciliation_result` | Reconciliation Result | Output of reconciliation run |
| `samp_sw_license` | Software License | Individual license records |
| `alm_license` | License | Base license table |
| `samp_publisher_pack` | Publisher Pack | Publisher pack definitions |
| `samp_sw_reclamation` | Reclamation | Reclamation workflow records |

---

## 11. SAM Pro vs. HAM Pro

| SAM Pro | HAM Pro |
|---------|---------|
| Software licenses | Physical hardware assets |
| Focus: compliance, cost optimization | Focus: lifecycle, financial tracking |
| Driven by: Discovery, normalization | Driven by: procurement, ASN, stockrooms |
| Key metric: over/under-licensed | Key metric: asset utilization, lifecycle cost |
| Key risk: vendor audit exposure | Key risk: untracked assets, disposal compliance |
| Integration: SCCM, Intune, SaaS APIs | Integration: ERP/purchasing, SCCM, CMDB |

Both modules are under the ITAM umbrella and work together: HAM tracks the physical device, SAM tracks the software on it.
