# ServiceNow ESP — Interview Questions & Answers

---

## PART 1: Core Concepts

**Q1: What is the difference between a Service Portfolio and a Service Catalog?**
**A:**
| Aspect | Service Portfolio | Service Catalog |
|--------|-----------------|----------------|
| **Scope** | ALL services: pipeline + active + retired | Only ACTIVE services |
| **Audience** | IT leadership, Service Owners | End users, business requestors |
| **Purpose** | Strategic lifecycle management | Fulfillment and request management |

The Catalog is the public-facing storefront. The Portfolio is the complete inventory including what's coming, what's live, and what's been retired.

---

**Q2: What are the three stages in the Service Portfolio lifecycle?**
**A:**
1. **Pipeline** — Service under development, not yet available to users
2. **Active (Catalog)** — Live service available for request or consumption
3. **Retired** — No longer offered; historical record maintained

---

**Q3: What is a Service Offering and how does it differ from a Service?**
**A:** A **Service** is the broad capability (e.g., "Cloud Storage"). A **Service Offering** is a specific, scoped, and priced package of that service for a defined consumer (e.g., "Cloud Storage — Enterprise Tier: 10TB, 99.99% SLA, $200/user/year").

One service can have multiple offerings. The `service_offering` table stores these in ServiceNow.

---

**Q4: What is a Service Owner and why are they critical to ESP?**
**A:** A Service Owner is the individual accountable for a service's quality, cost, SLA compliance, and lifecycle decisions — from inception through retirement. Unlike a project manager (time-bounded), the Service Owner has ONGOING accountability.

Without designated Service Owners, services grow stale, SLAs drift, catalog items become outdated, and retirement decisions never happen.

---

**Q5: What is the CSDM and why does ESP depend on it?**
**A:** CSDM (Common Service Data Model) is ServiceNow's blueprint for organizing service data. ESP requires CSDM alignment because:
- Service Portfolio records link to Application Services in CSDM
- Without CSDM, impact analysis doesn't work (no path from service to CI)
- Cost attribution requires the CSDM chain: Business App → Application Service → Infrastructure CIs

CSDM layers relevant to ESP:
```
Business Capability → Business Application → Application Service → Technical Service → Infrastructure CIs
```

---

**Q6: How does the Service Catalog request flow work in ServiceNow?**
**A:**
1. User browses Service Catalog → finds Catalog Item
2. Fills out request form (variables)
3. Submits → Request (`sc_request`) and Request Item (`sc_req_item`) created
4. Fulfillment workflow triggers (approval, tasks, provisioning steps)
5. Workflow completes → item delivered
6. SLA clock tracked throughout; breach triggers escalation

---

**Q7: What are the types of Service Catalog items in ServiceNow?**
**A:**
| Type | Description |
|------|------------|
| **Catalog Item** | Standard request — creates an RITM (Request Item) |
| **Record Producer** | Creates a record in a specific table (incident, case) — looks like a catalog item but produces a record |
| **Order Guide** | Bundle of catalog items ordered together as a set (e.g., new employee kit) |
| **Content Item** | Information only — no fulfillment; links to KB article or policy |

---

**Q8: What is Enterprise Service Management (ESM)?**
**A:** ESM extends the ServiceNow service management model beyond IT to all enterprise departments. Instead of IT being the only "service provider," HR, Finance, Legal, Facilities also deliver services through a unified Service Catalog and Portfolio.

**Business value:** Single portal for employees to request anything (laptop, leave, expense report, contract review) — regardless of which department fulfills it.

---

**Q9: What tables are most important for ESP in ServiceNow?**
**A:**
| Table | Purpose |
|-------|---------|
| `service_offering` | Service Offering records |
| `cmdb_ci_service` | Service CI in CMDB hierarchy |
| `cmdb_ci_business_app` | Business Application CI |
| `sc_cat_item` | Catalog Items |
| `sc_category` | Catalog categories |
| `sc_request` | Service requests |
| `sc_req_item` | Request Items (individual items in a request) |

---

**Q10: How does ESP connect to ITSM Incident Management?**
**A:** When an Incident is raised in ITSM:
- The "Business Service" field links the incident to the affected Service Portfolio entry
- Service health dashboards update automatically — incident shows on the service's health record
- SLA for incident resolution linked back to the service's service level commitments
- Major Incidents (P1) can formally impact the service and drive a Problem record

This means the Service Owner sees all incidents affecting their service in one view.

---

## PART 2: Configuration & Technical

**Q11: How would you structure a Service Catalog for a large enterprise?**
**A:** Best practice structure:
1. **Top-level categories by department or service domain** (IT, HR, Finance, Facilities)
2. **Sub-categories by type** (within IT: Hardware, Software, Access, Infrastructure)
3. **Catalog items** within sub-categories — specific, clearly named requests
4. **Consistent naming convention:** "[Action] [Object]" — e.g., "Request New Laptop," "Reset Password," "Provision VPN Access"
5. **Separate catalogs** if different audiences need completely different views (e.g., IT Service Catalog vs. Employee Service Center)

---

**Q12: What is a Variable Set and why is it used?**
**A:** A Variable Set is a reusable group of form fields (variables) that can be shared across multiple catalog items. Instead of recreating the same fields on every item, a Variable Set is created once and attached to many items.

**Example:** "Requester Information" Variable Set — Contains: Manager name, Cost Center, Business Justification — applied to every request requiring approval.

---

**Q13: How do you manage Catalog Item approvals?**
**A:** Approvals in catalog fulfillment workflows:
- **Approval rules:** embedded in the fulfillment workflow — triggers when workflow reaches the approval step
- **Approval via:** manager approval (based on requested-for user's manager), group approval, or specific approver
- **Auto-approval:** for low-risk items under a cost threshold (e.g., software licenses < $50)
- **Multi-level approval:** sequential or parallel approvals for high-value requests

---

**Q14: How does Service Portfolio financial management work in ServiceNow?**
**A:**
- **Cost Allocation:** CMDB infrastructure costs → attributed to Application Services → attributed to Business Applications (CSDM-driven)
- **Labor:** ITSM time tracking + SPM timecards allocated to services
- **License costs:** ITAM SAM data attributed to services
- **Total Service Cost** = Infrastructure + Labor + Licenses
- Reported via Service Portfolio dashboards — enables "should we keep vs. retire" decisions

---

**Q15: What is the difference between a Catalog Item and a Record Producer?**
**A:**
| Aspect | Catalog Item | Record Producer |
|--------|-------------|----------------|
| **Creates** | Request (`sc_request`) + Request Item (`sc_req_item`) | Record in ANY specified table |
| **Used for** | Service requests | Creating incidents, cases, change requests from catalog UI |
| **Workflow** | Catalog fulfillment workflow | Workflow on the target table (e.g., incident workflow) |
| **Example** | "Request New Laptop" | "Report a Security Incident" → creates `sn_si_incident` |

---

## PART 3: Scenario-Based Questions

**Q16: A business unit says "the IT catalog is useless — we can never find what we need." How would you fix this?**
**A:**
1. **Catalog Audit:** review all existing items — are they named clearly? Categorized logically?
2. **User research:** talk to the business unit — what are they actually looking for? What are they requesting today via email instead of catalog?
3. **Rename items:** use business language, not IT language ("Get a New Laptop" not "Hardware Provisioning Request")
4. **Improve search:** ensure catalog items have good keywords and descriptions
5. **Restructure categories:** align to how users think, not how IT is organized
6. **Remove dead items:** retire catalog items with zero requests in 12 months
7. **Add missing items:** items requested via email = items that should be in the catalog

---

**Q17: The organization wants to retire a legacy VPN service and replace it with Zero Trust Network Access. How does ESP manage this transition?**
**A:**
1. **New service:** Create "Zero Trust Network Access" as a Pipeline service in the Service Portfolio
2. **Development:** As SPM delivers the project, ESP tracks it as Pipeline
3. **Launch:** When ready, move to Active status; catalog item published for request
4. **Transition:** Set a retirement date for the legacy VPN
5. **Communication:** Notify all current VPN users of the cutover date
6. **Retire:** Move legacy VPN to Retired status in the portfolio — not deleted (historical record maintained)
7. **Asset cleanup:** ITAM removes associated VPN licenses; CMDB updates CI relationships

---

**Q18: How do you measure whether your Service Portfolio is healthy?**
**A:** Key metrics for Service Portfolio health:
- **Service coverage:** % of IT services with a named Service Owner
- **Catalog freshness:** % of catalog items reviewed/updated in past 12 months
- **SLA compliance:** % of requests fulfilled within SLA (by service)
- **CSAT scores:** Customer satisfaction per service
- **Abandoned requests:** Requests started but not submitted (catalog items with high abandonment = poor UX)
- **Email bypass rate:** Volume of emails to IT teams requesting things that should be in catalog
- **Incident rate by service:** Services with high incident counts are poor quality and should be improved or retired

---

**Q19: A new regulation requires documenting all services that process personal data. How would ESP support this?**
**A:**
1. **Custom attribute:** Add a "Processes Personal Data" (Yes/No) attribute to Service records
2. **Service Owner accountability:** Service Owners review their service and confirm PII processing status
3. **IRM integration:** Services flagged "Processes Personal Data" → linked to GDPR/HIPAA compliance requirements in IRM
4. **CMDB linkage:** Services linked to underlying systems — CMDB shows which databases hold the data
5. **Report:** Compliance team can pull a single report of all services processing personal data with their owners

---

**Q20: What is the biggest pitfall when implementing a Service Catalog?**
**A:** **Building the catalog around IT's organizational structure instead of user intent.**

IT teams naturally organize catalog items by team: "Infrastructure Requests," "Security Requests," "Application Team Requests." Users don't know which team does what — they know what they want to do ("Get a new laptop," "Set up MFA").

The result: a catalog that makes perfect sense to IT and is completely unusable to everyone else.

**Prevention:**
- Design catalog navigation from the user's perspective (card sort exercises, user testing)
- Name items using action verbs and plain language
- Put the same item in multiple places if users look in multiple places
- Test with 10 users before go-live: "Can you find how to request X?"

---

## PART 4: Exam-Style Quick Facts

| Question | Answer |
|----------|--------|
| Service Portfolio stages | Pipeline, Active (Catalog), Retired |
| Service Offering table | `service_offering` |
| Catalog Item table | `sc_cat_item` |
| Request table | `sc_request` |
| Request Item table | `sc_req_item` |
| Record Producer creates | A record in any specified table |
| Order Guide | Bundle of catalog items |
| Variable Set | Reusable group of form fields shared across catalog items |
| ESM = | Enterprise Service Management — beyond IT to all departments |
| CSDM requirement | ESP needs CSDM for impact analysis and cost attribution |
| Service Owner = | Accountable for service quality, cost, lifecycle (ongoing, not project-based) |

---

## Study Checklist
- [ ] Know: Portfolio vs. Catalog distinction clearly
- [ ] Know: Three lifecycle stages (Pipeline/Active/Retired)
- [ ] Know: Service Offering vs. Service definition
- [ ] Know: Service Owner role and responsibilities
- [ ] Know: CSDM hierarchy and why ESP depends on it
- [ ] Know: Catalog item types (Standard, Record Producer, Order Guide, Content)
- [ ] Know: Variable Set concept
- [ ] Know: ESM definition and business value
- [ ] Know: Key tables (service_offering, sc_cat_item, sc_request, sc_req_item)
- [ ] Know: How to approach catalog redesign (user-centric, not IT-centric)
