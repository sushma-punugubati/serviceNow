# ServiceNow CSM — Interview Notes
## "What did you do in CSM?" + Challenges Faced

---

## 1. OVERVIEW — What is CSM?

CSM (Customer Service Management) in ServiceNow applies structured case management to external customer-facing operations. Where ITSM manages internal IT requests, CSM manages the relationship between an organization and its external customers — from case creation through resolution, escalation, and satisfaction measurement.

The core value: every customer interaction (call, email, chat, portal submission) becomes a tracked case with an owner, SLA, and full history. Customers can self-serve through branded portals, agents get a 360-degree view of the customer (account, past cases, entitlements, installed products), and managers get real-time visibility into team performance and SLA compliance.

Key areas worked on:
- Case types and templates per product/team
- Agent Workspace and Advanced Work Assignment (AWA)
- B2B (account-based) and B2C (consumer-based) portal configuration
- SLA enforcement and escalation workflows
- Inbound email-to-case automation
- Predictive Intelligence for auto-categorization and routing
- CRM integration via REST/IntegrationHub

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Case Management Configuration

- Defined **Case Types** for each product line and support tier — each type had its own required fields, assignment rules, SLA definitions, and resolution templates so agents were guided through the right workflow for each case category
- Configured **assignment rules** to auto-route cases to the correct team based on the case type, product, customer entitlement level, and issue category — eliminated manual triage for about 80% of incoming cases
- Set up **SLA definitions** with business-hour-aware calculations: Priority 1 cases had a 1-hour first response SLA; standard cases had a 4-hour response and 2-business-day resolution SLA. Configured automatic escalation notifications at 75% and 100% of SLA elapsed
- Built **playbooks** for common resolution paths — agents following the playbook for billing disputes, for example, had step-by-step guidance that reduced average handle time and improved consistency

### 2.2 Agent Workspace and AWA

- Configured **Agent Workspace** as the primary interface for the CSM team — set up contextual sidebars showing the customer's account details, open cases, entitlements, installed products, and recent interactions in a single view so agents didn't have to navigate multiple screens
- Implemented **Advanced Work Assignment (AWA)** to route cases to the right agents based on skills, availability, language, and current workload — configured capacity limits per agent so high-performers were not overwhelmed while newer agents were not underutilized
- Set up **presence management** so AWA only routed to agents who were actively available — reduced cases stuck in "assigned but not actioned" state that had been a chronic issue with the previous static assignment rules

### 2.3 B2B and B2C Portal Configuration

- Configured a **B2B portal** for business accounts — account-level login showed only the account's own cases, allowed account administrators to view cases created by any user within their organization, and exposed contract/entitlement data relevant to that account
- Set up **B2C portal** for individual consumers — simpler self-service view with case creation, status tracking, knowledge base search, and virtual agent for common inquiries
- Configured **consumer registration and identity verification** — new consumers could register via the portal and were sent an email verification step before gaining access to case creation; B2B account users were provisioned via SSO/SAML from the customer's identity provider

### 2.4 Inbound Email-to-Case

- Configured **inbound email actions** to convert customer emails to CSM cases automatically — set up routing rules based on the destination email address (e.g., support@company.com → P2 case, enterprise-support@company.com → P1 case)
- Built **deduplication logic** using Business Rules to check if an inbound email was a reply to an existing case (matching on the ticket number in the email subject or thread ID in headers) and append it as a work note rather than creating a new case
- Set up **auto-acknowledge emails** that sent the customer a case number and expected response time immediately on case creation

### 2.5 Predictive Intelligence for Routing

- Implemented **Predictive Intelligence** for case categorization — trained the model on 18 months of historical case data. After training, it correctly predicted the category for 73% of new cases, which fed the assignment rule engine and reduced manual triage time
- Set up a **feedback loop**: agents could correct the predicted category, which automatically fed corrected training data back to improve model accuracy over time
- Initially the model accuracy was low (around 45%) because the historical data had inconsistent category usage. Worked with the team to standardize categories and retrain before going live

### 2.6 CRM Integration

- Built a **REST integration with Salesforce CRM** via IntegrationHub — customer account data, contact records, and contract entitlements flowed from Salesforce into CSM so agents could see the full customer context without switching systems
- Set up bidirectional sync for case status: high-value B2B cases with an active Salesforce opportunity were mirrored back to Salesforce so account managers had visibility into open support issues affecting their deals
- All integration traffic went through Dell Boomi (Layer-7 API gateway) — I owned the ServiceNow side (REST message definition, transform maps, error handling) and coordinated with the Boomi team for the middleware configuration

---

## 3. CHALLENGES FACED

### Challenge 1: Inbound Email Creating Duplicate Cases for Replies

**Problem:** When customers replied to case notification emails, the system was creating new cases instead of appending the reply to the existing case. This resulted in multiple open cases for the same issue and confused both agents and customers.

**Solution:** Enhanced the inbound email action to parse the email subject line for the case number pattern (e.g., [CSM0012345]) and added a pre-action script that checked if a matching open case existed before creating a new one. If a match was found, the email body was added as a work note on the existing case and the case state was updated from "Awaiting Customer Response" back to "In Progress."

---

### Challenge 2: Predictive Intelligence — Low Initial Accuracy

**Problem:** After deploying Predictive Intelligence for case categorization, the model accuracy was 45% — barely better than random for a 12-category model. Agents were not trusting the predictions and ignoring them entirely, which defeated the purpose of the feature.

**Solution:** Investigated the training data and found that 6 of the 12 categories had fewer than 200 historical examples (the minimum recommended is 400+). Also found that category names had been used inconsistently over time — what was "Billing" in 2021 was "Payment" in 2022. Did a data cleanup pass to normalize categories on historical records, added 3 months of newly categorized cases to the training set, and retrained. Accuracy improved to 73% in the second run, which was high enough for agents to start trusting and using the predictions.

---

### Challenge 3: AWA Not Distributing Work Evenly

**Problem:** After deploying AWA, several agents were receiving significantly more cases than others at the same skill level. Some agents complained of being overwhelmed; others had almost nothing to do.

**Solution:** Reviewed the AWA configuration and found the issue: capacity was set the same for all agents, but availability was being managed manually and several agents had not updated their presence status. Added a mandatory presence check: agents had to log in through the workspace (which auto-set presence to Available) and AWA only routed to agents with an active presence record. Also reviewed capacity settings and adjusted them based on case complexity per queue — support agents handling billing disputes had lower capacity limits than agents handling simple how-to questions.

---

### Challenge 4: B2B Portal SSO Sync Not Working for Some Accounts

**Problem:** Several B2B accounts reported that their users could not log in to the portal via SSO. The error was generic and not informative enough to troubleshoot quickly.

**Solution:** Enabled detailed SAML assertion logging on the ServiceNow side and reviewed the failed assertions. Found two issues: some customer IdP configurations were sending the email attribute in a non-standard field that ServiceNow was not reading, and a few accounts had their contract records expired in ServiceNow (which restricted portal access). Fixed the SAML attribute mapping to check multiple field names for the email value, and added a scheduled job to notify the account management team 30 days before a contract expired to prevent access disruption.

---

### Challenge 5: SLA Pausing Not Working Correctly for "Awaiting Customer" State

**Problem:** Cases in "Awaiting Customer Response" state should have paused the resolution SLA clock. The SLA was continuing to tick even while waiting for the customer to respond, causing SLA breaches that were not the team's fault.

**Solution:** Reviewed the SLA definition and found that the "Awaiting Customer Response" state had been added to the case state model after the SLA definitions were created, and the SLA pause conditions were never updated to include it. Updated the SLA condition to include all "waiting" states in the pause filter. Also set up retroactive SLA recalculation for the affected historic cases so the reporting was accurate.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **Case (CSM)** | The primary record in CSM — tracks a customer interaction from creation to resolution |
| **Account / Consumer** | B2B: an Account is a business customer with multiple contacts. B2C: a Consumer is an individual customer |
| **Entitlement** | Defines what level of support a customer is entitled to based on their contract — drives SLA and routing |
| **AWA (Advanced Work Assignment)** | Routes work to agents based on skills, availability, and capacity — replaces static assignment group routing |
| **Agent Workspace** | The primary agent UI in CSM — shows case, customer context, KB, and related records in one view |
| **Inbound Email Action** | Converts incoming emails to cases automatically based on routing rules |
| **Predictive Intelligence** | ML-based auto-categorization — predicts case category and routing based on historical patterns |
| **B2B Portal** | Account-based self-service portal for business customers |
| **B2C Portal** | Consumer-based self-service portal for individual customers |
| **Playbook (CSM)** | Guided resolution workflow that walks agents through the steps for a specific case type |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Email-to-case duplicate rate reduced from **~25% to under 2%** after inbound email deduplication logic
- Predictive Intelligence case categorization accuracy improved from **45% to 73%** after data cleanup and retraining
- AWA implementation reduced average case wait time (time from creation to first agent touch) by **40%**
- SLA first-response compliance improved from **78% to 94%** after AWA and SLA pause corrections
- Agent handle time decreased by **~20%** after Agent Workspace rollout — context visible in one view vs. navigating multiple modules
- B2B portal self-service deflection: **~30%** of B2B cases submitted via portal without contacting support directly
