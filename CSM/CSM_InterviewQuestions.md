# ServiceNow CSM — Interview Questions & Answers

> Covers: conceptual questions, scenario-based questions, technical configuration questions, and CIS-CSM exam-style questions.

---

## PART 1: Conceptual / Fundamentals

---

**Q1: What is ServiceNow CSM and how does it differ from ITSM?**

**A:** ServiceNow CSM (Customer Service Management) is designed to manage **external customer** service — handling cases, contacts, accounts, and customer journeys across multiple channels. ITSM (IT Service Management) is designed for **internal employees** — managing incidents, problems, changes, and service requests.

Key differences:
- ITSM uses **Incidents**; CSM uses **Cases**
- ITSM serves **employees**; CSM serves **customers/consumers**
- ITSM uses **Assets**; CSM uses **Sold Products / Install Base Items**
- ITSM uses **OLAs/SLAs**; CSM uses **Entitlements + Service Contracts**
- CSM supports **omnichannel** (portal, email, chat, voice) for external customers

---

**Q2: What are the three main business models CSM supports, and how do they differ?**

**A:**
- **B2B (Business-to-Business)**: Company serves corporate clients. Uses **Accounts and Contacts** as the customer data model. Example: A software company supporting enterprise customers.
- **B2C (Business-to-Consumer)**: Company serves individuals. Uses **Consumers** instead of Accounts/Contacts. Example: A telecom company supporting residential subscribers.
- **B2B2C**: Hybrid — company serves businesses AND those businesses' end customers. Uses Account + Consumer together. Example: A bank serving both the corporate entity and its retail customers.

---

**Q3: What is the difference between a Contact and a Consumer in CSM?**

**A:**
- **Contact**: A person who works at an Account (company). Used in B2B scenarios. Stored in `customer_contact` table, extends `sys_user`. Belongs to exactly one account.
- **Consumer**: An individual customer not tied to a company. Used in B2C scenarios. Stored in `csm_consumer` table.

**Memory trick:** Contact = Corporate (B2B). Consumer = Consumer/individual (B2C).

> This is one of the **most tested concepts** in CIS-CSM exams.

---

**Q4: What is an Entitlement? How does it relate to Service Contracts and Cases?**

**A:** An **Entitlement** defines what level of support a customer is entitled to — support type (email only, phone + email), SLA tier, and tracking method (cases or hours).

Relationship:
```
Service Contract → contains → Entitlements → applied to → Cases
```
- A **Service Contract** is the master agreement (e.g., 3-year support agreement)
- It contains one or more **Entitlements** (e.g., Gold Support, Silver Support)
- When a case is created, CSM checks the Entitlement to determine the SLA and allowed channels

**Two tracking methods:**
- **Case-based**: Customer gets X cases per contract period
- **Hour-based**: Customer gets X support hours per contract period

---

**Q5: What is the difference between a Sold Product and an Install Base Item?**

**A:**
- **Sold Product**: A product/service sold to a specific account or consumer. It's the "purchase record." Example: "Autovity bought 50 Enterprise Licenses."
- **Install Base Item**: The deployed, configured instance of a sold product. Has a serial number, location, configuration details. Example: "License #47 assigned to user John, installed on server SRV-001 at Autovity's Chicago office."

**Analogy**: Sold Product = car purchased. Install Base Item = that specific car with VIN, current mileage, service history.

---

**Q6: What is a Partner in CSM and how does a Partner differ from a Customer?**

**A:**
- **Customer**: Buys your products/services and contacts you for support
- **Partner**: A third party that sells YOUR products to their own customers and can manage cases on behalf of those customers in your ServiceNow instance

Partners have special roles (`sn_customerservice.partner`) that allow them to:
- Create cases for their own accounts OR their customers' accounts
- View/edit cases they created
- Manage their customer accounts

**Example:** A reseller company that sells your cloud software to small businesses and handles Tier-1 support, escalating to you only for complex issues.

---

**Q7: Describe the Case lifecycle in CSM.**

**A:** Cases flow through these states:

```
New → Open → In Progress → Pending → Resolved → Closed
```

| State | When Used |
|-------|-----------|
| New | Just created, not yet assigned |
| Open | Assigned to an agent |
| In Progress | Agent actively working it |
| Pending | Waiting on customer, third party, or other team |
| Resolved | Solution provided, waiting for customer confirmation |
| Closed | Final/terminal state — completely done |

> **Exam fact:** "Closed" is the final state, not "Resolved."

---

**Q8: What is Major Issue Management in CSM?**

**A:** Major Issue Management handles widespread problems that affect many customers simultaneously.

**How it works:**
1. A **Major Case** is created (either manually or automatically via ITOM monitoring)
2. Individual customer cases are linked to the Major Case
3. Bulk notifications sent to all affected customers
4. Agents work the root cause once, not 500 times
5. When resolved, all linked cases can be mass-closed
6. Automated resolution notifications go to all customers

**Example:** Cloud service outage affecting 500 customers. One Major Case, 500 linked cases, automated outage communications.

---

**Q9: What is Proactive Customer Service Operations (PCSO)?**

**A:** PCSO integrates CSM with **ITOM (IT Operations Management)** so the system detects and responds to issues before customers even notice.

**Flow:**
1. ITOM monitoring detects service degradation
2. Alert triggers automatic Case creation in CSM
3. CSM sends outbound notifications to affected customers
4. When monitoring confirms issue resolved, cases auto-close and resolution notifications sent

**Business value:** Transform from reactive ("fix it when customers call") to proactive ("fix it before customers know").

---

**Q10: What is the role of the Customer Service Portal?**

**A:** The Customer Service Portal is a self-service web interface for external customers to:
- Create and track their own cases
- Search knowledge base articles
- Participate in community forums
- Manage their profile, products, and assets
- Chat with Virtual Agent

**Business benefit:** **Case deflection** — customers resolve issues without creating a case or calling, reducing agent workload and cost.

---

## PART 2: Roles & Configuration

---

**Q11: What are the key CSM roles and what can each do?**

**A:**

**Internal roles:**
| Role | Key Capability |
|------|---------------|
| `sn_customerservice_agent` | Create, view, edit cases; work customer issues |
| `sn_customerservice_manager` | Manage agents/groups, override actions, admin skills |
| `sn_customerservice_admin` | Full CSM configuration |

**External roles:**
| Role | Key Capability |
|------|---------------|
| `sn_customerservice.customer` | Create and view own cases, see own assets |
| `sn_customerservice.customer_case_manager` | Manage all cases for their account |
| `sn_customerservice.customer_admin` | Full account data access |
| `sn_customerservice.partner` | Create cases for own or customer accounts |
| `sn_customerservice.partner_admin` | Full partner + customer account management |
| `sn_customerservice.consumer` | B2C customer — create/view own cases |

---

**Q12: What methodology does ServiceNow use for CSM implementations?**

**A:** ServiceNow uses the **Now Create** methodology. (Not Agile, not SIM, not OCM — this is specifically called out in exam questions.)

---

**Q13: Which AWA service channels can be used with CSM?**

**A:** CSM supports three AWA (Advanced Work Assignment) service channels:
1. **Case** — work items from case records
2. **Chat** — live chat interactions
3. **Walk-up** — in-person service desk interactions

> **Exam trap:** Email and telephony are NOT AWA channels for CSM.

---

**Q14: What is a Work Item in the context of Advanced Work Assignment?**

**A:** A **Work Item** is a single piece of work to be handled by one agent in the AWA system. When a case, chat, or walk-up interaction is routed, AWA creates a Work Item and assigns it to the best available agent based on skills and capacity.

---

**Q15: How is a forum configured in CSM Communities?**

**A:** A forum can be configured in three ways:
- **Public** — anyone can view and participate
- **Private** — only invited members can participate
- **Membership** — users request or are invited to join

> **Exam note:** "Secret" is NOT a forum configuration type in CSM.

---

**Q16: What is the CSM Configurable Workspace form ribbon used for?**

**A:** The form ribbon in CSM Configurable Workspace provides agents with:
- Timeline and SLA details at a glance
- Quick links for calls or emails
- Quick case overview/summary
- Actions for the current case

> **What it CANNOT do:** Add or remove customer/consumer records. That's done through the contextual side panel.

---

**Q17: What feature in the CSM Configurable Workspace contextual side panel allows creating a new contact or consumer?**

**A:** The **"Lookup and Verify"** feature. It allows agents to search for an existing contact/consumer or create a new one without leaving the case.

---

**Q18: What is the CSM plugin ID and what does it require to activate?**

**A:** The core CSM plugin ID is `com.sn_customerservice`. It requires an **admin role** to activate and is the foundation for all CSM functionality.

---

## PART 3: Scenario-Based Questions

---

**Q19: A customer calls about a technical issue, but when the agent checks the account, the service contract expired last month. What should happen?**

**A:** CSM's **entitlement verification** should flag this during case creation. The system checks whether the customer has an active entitlement for the product they're calling about. With an expired contract:
- The agent should be notified the account is out of entitlement
- Depending on configuration, the case may still be created (pay-per-incident) or blocked
- The agent should offer renewal options
- Some implementations configure a "grace period" entitlement for recently expired contracts

---

**Q20: A global bank has subsidiaries in 20 countries. How would you structure the CSM account hierarchy?**

**A:** Use CSM's **Account Hierarchy** feature:
- Create a **Parent Account** for the global bank (e.g., "Global Bank Corp")
- Create **Child Accounts** for each country subsidiary (e.g., "Global Bank - Germany", "Global Bank - Singapore")
- Assign **Parent Account** field on each child account
- A **Customer Admin** at the parent level can see all subsidiary data
- Contacts/cases can be tracked at the subsidiary level but visible up the hierarchy
- Service contracts can be applied at parent level (covering all subsidiaries) or at subsidiary level (country-specific terms)

---

**Q21: Your company has 500 customers affected by a software bug that causes data export to fail. How do you handle this in CSM?**

**A:** This is a **Major Issue Management** scenario:

1. Create a **Major Case**: "Data Export Failure — v4.2 Bug"
2. Link all 500+ existing customer cases to this Major Case
3. Send bulk **outbound communication** to all affected accounts: "We've identified the issue and are working on a fix"
4. Create a **Problem** record linked to the Major Case for the engineering team
5. Engineering creates a **Change** for the bug fix
6. When fix deployed, **mass-close** all linked cases
7. Send automated resolution notification to all customers
8. Publish **Knowledge Article** documenting the issue and fix

---

**Q22: An agent receives a chat from a customer who wants to add a new device to their account. The agent cannot find the customer in the system. What should they do?**

**A:** Use the **Lookup and Verify** feature in the CSM Configurable Workspace contextual side panel to:
1. Search by email, phone, or name
2. If not found, create a new **Consumer** (B2C) or **Contact** (B2B) record directly from the panel
3. Link the new record to the active chat interaction
4. Create a case with the new record attached
5. Add the sold product/device to their account record

---

**Q23: How would you set up CSM to support a partner model where resellers manage Tier-1 support for your customers?**

**A:**
1. Create **Partner Account** records for each reseller company
2. Create **Partner Contact** records for each reseller's support agents
3. Assign `sn_customerservice.partner` role to partner users
4. Assign `sn_customerservice.partner_admin` role to the partner firm's admin
5. Create **Account Relationships** linking each partner to their managed customer accounts
6. Configure entitlements: partner-supported accounts get Tier-1 SLA; escalations to your team get different SLA
7. Partners log in via Customer Portal with their own view, filtered to their assigned customer accounts
8. Partner portal shows: their customers' cases, ability to create cases, escalation workflow to your team

---

**Q24: A new junior agent is struggling to resolve technical cases consistently. What CSM feature would you recommend implementing?**

**A:** Implement **Playbooks** and **Guided Decisions**:

- **Playbooks**: Configure step-by-step resolution guides for each common case type. Agent opens a case, playbook auto-activates based on case type/category, guides through every required step.
- **Guided Decisions**: Build decision trees for technical diagnosis. Agent answers yes/no questions and the system identifies the most likely root cause and resolution path.
- **Knowledge Article linking**: Configure suggestions so relevant KB articles auto-surface based on case description keywords.

This reduces resolution time, ensures consistent customer experience, and accelerates new agent onboarding.

---

**Q25: What's the difference between a case being "Resolved" and being "Closed"?**

**A:**
- **Resolved**: The agent has provided a solution, but the case is not yet officially done. Typically, the customer is notified and given time to confirm the solution worked. The case can be reopened if the solution didn't work.
- **Closed**: The final, terminal state. Case cannot be reopened from Closed. Either customer confirmed resolution, or an auto-close timer expired after resolution.

---

## PART 4: Technical / Configuration Questions

---

**Q26: What is the table name for CSM Cases?**

**A:** `sn_customerservice_case`

---

**Q27: What table does the Contact record extend?**

**A:** `customer_contact` extends `sys_user`. This means every Contact automatically has a ServiceNow user record and can be given portal login credentials.

---

**Q28: What table does the Account record extend?**

**A:** `customer_account` extends `core_company`.

---

**Q29: How does CSM handle communication channels (omnichannel)?**

**A:** CSM uses **Interactions** as the channel-agnostic record for each communication event. Interactions have channel-specific subtypes:
- **Email Interaction** — from inbound email
- **Chat Interaction** — from live chat
- **Voice/Phone Interaction** — from CTI-connected phone calls

Interactions can either:
- Create new Cases automatically
- Link to existing Cases

This allows a single Case to have multiple Interactions across different channels (customer emailed, then called, then chatted — all linked to one Case).

---

**Q30: What is the difference between account-level and product-level entitlements?**

**A:**
- **Account-level entitlement**: All products/cases for that account get this support level. Good for blanket enterprise agreements.
- **Product-level entitlement**: Specific support rules tied to a specific sold product or install base item. Good when different products have different support tiers (e.g., "Premium" software gets 24/7 support; "Standard" hardware gets 9-5 support).

---

**Q31: What is Special Handling Notes in CSM?**

**A:** Special Handling Notes are configurable flags/notes that appear automatically when a specific customer's case is opened. They alert agents to specific instructions, sensitivities, or requirements for dealing with that customer.

**Examples:**
- "This is a VIP account — escalate immediately if SLA is breached"
- "Customer prefers contact via email only — do not call"
- "Legally sensitive account — all communications must be reviewed by legal team"

---

**Q32: Can CSM integrate with ITSM? What happens when a case requires IT involvement?**

**A:** Yes. CSM-ITSM integration (one of the core CSM exam domains) allows:
- Cases to automatically create **Incidents** in ITSM when IT involvement is needed
- Agents can see the linked Incident status from the Case
- When the Incident resolves, the Case is notified/updated
- **Change requests** can be linked to cases for product/service changes
- **Problem records** can be created from multiple related cases

This "connected service" model means customers don't need to re-explain their issue to IT teams.

---

**Q33: What is the CSM Scoping Guide?**

**A:** The CSM Scoping Guide is a ServiceNow best practices document that helps implementation teams define project scope, prioritize features, and set customer expectations during CSM deployments. It covers:
- Required vs. optional features per implementation phase
- Common scope expansion pitfalls
- Business process mapping templates
- Stakeholder alignment checklists

---

## PART 5: CIS-CSM Exam Prep — Quick Facts

| Question | Answer |
|----------|--------|
| Exam questions | 60 multiple choice |
| Duration | 90 minutes |
| Passing score | ~70% (42/60) |
| Cost | $315 |
| Prerequisite | CSA certification |
| Implementation methodology | **Now Create** |
| Final case state | **Closed** |
| Contact table | `customer_contact` (extends `sys_user`) |
| Case table | `sn_customerservice_case` |
| Core plugin ID | `com.sn_customerservice` |
| Contact vs Consumer | Contact = B2B, Consumer = B2C |
| AWA channels for CSM | Case, Chat, Walk-up |
| Work item | Single assignment unit in AWA |
| Forum types | Public, Private, Membership (not "Secret") |
| Form ribbon CANNOT do | Add/remove customer records |
| "Lookup and Verify" feature | Create contact/consumer from contextual panel |
| Largest exam domain (30%) | CSM Configuration |

---

## PART 6: Tricky "Gotcha" Questions

---

**Q34: A colleague says "CSM uses the Asset table like ITSM does." Is this correct?**

**A:** No. CSM uses the **Product table** and **Product Model table**, not the ITSM Asset table and Asset Model table. These are distinct tables optimized for the customer-facing product management model. Confusing these is a common ITSM-to-CSM transition mistake.

---

**Q35: An interviewer asks: "What's better — Agile or Now Create for CSM projects?" How do you answer?**

**A:** ServiceNow specifically recommends **Now Create** for CSM implementation projects. Now Create is ServiceNow's proprietary implementation methodology tailored to the platform's architecture and customer journey framework. While many teams incorporate Agile sprint practices within a Now Create framework, the formal methodology is Now Create. (This is a CIS-CSM exam fact.)

---

**Q36: True or false — Mobile phone browsers support the desktop version of CSM Workspaces.**

**A:** **False.** Mobile phone browsers cannot support the desktop version of the CSM Workspace UI. Browser support varies per UI version, and desktop-specific CSM Workspaces require a desktop/laptop browser.

---

**Q37: A customer's case has been in "Resolved" state for 5 days and they haven't responded. What typically happens?**

**A:** Most CSM implementations configure an **auto-close rule**: if a case remains in "Resolved" state for a defined period (commonly 5-7 days) without customer response, it automatically transitions to **"Closed."** An automated notification is sent to the customer informing them the case has been closed. If they experience the issue again, they're typically directed to create a new case.

---

**Q38: When would you use a Case Task vs. creating a new Case?**

**A:**
- **Case Task**: Use when the work can be broken into sub-steps that are part of the same customer issue. All tasks roll up under the parent case. The customer sees one case, agents internally coordinate via tasks. Good for: multi-team resolution where the customer issue is one thing.
- **New Case**: Use when the customer has a completely separate, unrelated issue. Also use for partner-managed sub-cases. Good for: different issues, different products, different entitlements.

---

## Study Checklist

- [ ] Memorize: Contact = B2B (`customer_contact`), Consumer = B2C (`csm_consumer`)
- [ ] Know: Final case state = **Closed**
- [ ] Know: Implementation methodology = **Now Create**
- [ ] Know: CSM plugin = `com.sn_customerservice`
- [ ] Know: AWA channels = Case, Chat, Walk-up
- [ ] Know: Entitlement = defines support type + channels + SLA
- [ ] Know: Service Contract → contains → Entitlements
- [ ] Know: Sold Product → Install Base Item (specific instance)
- [ ] Know: Partner can create cases for customer accounts
- [ ] Know: Form ribbon ≠ add/remove customers (that's contextual panel "Lookup and Verify")
- [ ] Know: Forum types = Public, Private, Membership (not Secret)
- [ ] Know: CSM uses Product table (not ITSM Asset table)
- [ ] Practice: Draw the Account → Contact → Contract → Entitlement → Case data model from memory
