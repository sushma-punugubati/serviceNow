# ServiceNow AI — Interview Notes
## "What AI have you set up in ServiceNow?" + Challenges Faced

---

## 1. OVERVIEW — ServiceNow AI Work

My AI work on the ServiceNow platform spans both the mature ML-based capabilities (Predictive Intelligence, Virtual Agent NLU) and the newer generative AI features (Now Assist, Now Assist for Creator, AI Agents). I have worked with these capabilities at Dell and through earlier client engagements.

Key areas:
- Now Assist for ITSM: incident summarization and resolution notes at Dell
- Now Assist for HRSD: case summarization and Employee Center chat responses (Apex Systems)
- Virtual Agent NLU configuration for ITSM and HR self-service deflection
- Predictive Intelligence for CSM case categorization and routing
- Now Assist for Creator: AI-assisted script and flow development in my daily workflow
- Early exploration of AI Agents via Agent Studio
- ServiceNow MCP: learning and experimenting with exposing ServiceNow to external AI tools

---

## 2. WHAT I DID — Project Work (Scenario-Based)

### 2.1 Now Assist for ITSM — Dell

- Configured **Now Assist incident summarization** for the Dell ITSM team — when an agent opens an incident, Now Assist automatically generates a concise summary showing: what the issue is, what has been tried, current status, and SLA position
- Customized the **summary prompt** to include CSDM business service context — summaries include which Application Service is affected and who the business owner is, not just the technical details of the incident
- Deployed **Now Assist resolution notes** — when resolving an incident, the AI drafts the resolution note from the work history. Agents edit and approve before closing. This improved resolution note completion rates significantly — agents are much more likely to complete a closure note when they only need to review a draft vs. write from scratch
- Configured the **AI Provider** (ServiceNow NowLLM) and set up AI Control Tower monitoring: usage rate, thumbs-up/down feedback, and correction frequency tracked weekly

### 2.2 Now Assist for HRSD — Apex Systems

- Deployed **Now Assist case summarization** for HR agents — HR cases can have long threads involving sensitive information. Now Assist generated summaries that captured the key facts without exposing the full sensitive thread to agents who did not need all the detail
- Configured **Now Assist for Employee Center** generative responses — employees could ask HR policy questions in natural language ("What is the paternity leave policy?") and receive AI-generated answers grounded in the Knowledge Base. Significantly reduced the number of simple HR policy questions reaching the HR case queue
- Worked closely on the **KB quality preparation** before enabling Now Assist — spent 2 weeks reviewing and updating HR KB articles to ensure AI responses would be accurate. Outdated articles were retired before the AI went live

### 2.3 Virtual Agent NLU Configuration

- Configured **Virtual Agent NLU topics** for the top 15 ITSM and HR intents — wrote 25-40 training utterances per topic covering varied phrasing, formal and informal language, and common misspellings
- Built **branching topic flows** so the VA could handle intent variations within a single topic: "I cannot log in" / "my account is locked" / "password expired" all routed to the same resolution flow with entity-driven branching for the specific sub-issue
- Configured **human handoff** with full context transfer: when VA could not resolve, the created ITSM incident included the full conversation transcript, the entities collected, and the VA topic that was active — agents did not need to ask users to repeat their issue
- Tracked VA resolution rates monthly and added training utterances for any phrases consistently triggering "I did not understand" — improved resolution rate from initial 42% to 68% over 4 months

### 2.4 Predictive Intelligence — CSM (Apex Systems)

- Configured **Predictive Intelligence for CSM case categorization** — trained the model on 18 months of historical case data to predict the category and assignment group from the case subject and description
- The initial model accuracy was only 45% — investigated and found the issue: historical category assignments were inconsistent, with the same case type labeled under 3 different categories depending on which agent created it. Did a data cleanup pass normalizing historical categories, then retrained. Accuracy improved to 73% in the second run
- Configured **feedback loop**: when agents corrected the predicted category, the correction was used as additional training data improving the model over subsequent retraining cycles
- Set confidence thresholds: 80%+ auto-apply, 60-79% show as suggestion, below 60% no prediction shown

### 2.5 Now Assist for Creator — Daily Development Use

- Use **Now Assist for Creator** regularly in my ServiceNow development work — primarily for generating boilerplate Business Rules, Script Includes, and REST API scaffolding
- Most effective for: generating the structure and scaffolding of a script (GlideRecord query pattern, error handling framework, API response parsing) that I then customize with the specific business logic
- Always review generated code carefully — check for unlimited queries (missing `setLimit()`), hardcoded sys_ids, missing null checks, and scope correctness before committing
- Use Now Assist code explanation for code review: paste inherited or undocumented legacy code and ask it to explain what it does — significantly speeds up understanding of code written by previous developers

### 2.6 AI Agents — Early Adoption

- Have been exploring **AI Agents via Agent Studio** — built a proof-of-concept incident triage agent that uses CMDB lookup and KB search to suggest priority and assignment group for new incidents
- The PoC showed the concept works but highlighted the importance of CSDM data quality — the agent's CMDB lookups were only useful when CIs had accurate business service relationships. Without CSDM, the agent had no business context to work with
- Exploring **AI Agent Fabric** for multi-agent coordination — the idea of an orchestrated major incident response (alert detection → impact analysis → communication → resolution suggestion) running autonomously is compelling, and I am building toward a production pilot

---

## 3. CHALLENGES FACED

### Challenge 1: Now Assist KB Quality — Outdated Articles Producing Wrong AI Responses

**Problem:** At Apex Systems, we deployed Now Assist for Employee Center before fully auditing the HR Knowledge Base. Within the first week, employees were getting AI-generated responses about a parental leave benefit that had been updated 6 months prior. The AI was accurately reflecting the KB — but the KB was wrong. Several employees made decisions based on the outdated AI responses, causing HR to have to intervene manually and creating distrust in the AI feature.

**Solution:** Paused the generative response feature for HR policy questions immediately. Ran a full KB audit: identified 23 articles that contained outdated policy information and updated or retired them. Implemented mandatory KB review schedules going forward — articles older than 12 months without review auto-moved to "Review Required" status and removed from AI context. Re-enabled the feature 3 weeks later with a clear user disclaimer: "AI responses are based on published HR policies — for recent changes, contact HR directly." Reinforced the lesson: AI is only as good as the data it is grounded in.

---

### Challenge 2: Virtual Agent Resolution Rate Plateau

**Problem:** After the initial 4 months of tuning, the VA resolution rate improved from 42% to 68% and then plateaued. Additional training utterances were not improving the rate further. Analysis showed we were getting the same missed intents repeatedly.

**Solution:** The issue was not NLU accuracy — for the common intents, NLU was correctly classifying. The problem was that about 15% of users were asking about intents for which no topic existed — niche IT requests and specific policy questions that were never covered in the initial topic scope. Rather than trying to improve NLU for existing topics, expanded the topic coverage: added 8 new topics for the most frequently asked uncovered intents (application access requests, VPN configuration help, new employee setup status). Resolution rate improved to 74% after the new topics were deployed.

---

### Challenge 3: Predictive Intelligence Model Drift After Organizational Change

**Problem:** At Dell, we had Predictive Intelligence routing incidents to assignment groups with 78% accuracy. After a reorganization that restructured 6 IT teams and renamed several assignment groups, accuracy dropped to 51% within 2 weeks. The model was confidently routing to groups that had been renamed or no longer existed.

**Solution:** Immediately retrained the model using only the last 3 months of historical data (post-reorganization) so the new team structure was reflected in training data. The retraining was urgent — could not wait for 18 months of clean data to accumulate. The initial post-retraining accuracy was lower (64%) because of less training data volume, but improved to 76% over the next 3 months as more post-reorganization incidents accumulated. Added organizational change to the Predictive Intelligence maintenance trigger: any time assignment groups are renamed, added, or merged, a PI retraining is scheduled within 2 weeks.

---

### Challenge 4: Now Assist Code Generation Missing ServiceNow Best Practices

**Problem:** A junior developer on my team started using Now Assist for Creator heavily for Business Rule generation. Several of the generated Business Rules had unlimited GlideRecord queries (no `.setLimit()`) and hardcoded sys_ids for assignment groups. Two of these made it through code review to DEV, where one caused a performance issue during a heavy load test.

**Solution:** Created a "Now Assist Code Review Checklist" specifically for AI-generated code — it goes through the most common AI code quality gaps: query limits, no hardcoded sys_ids or field values, proper null checks, scoped API usage, security implications. Added this checklist to our code review process for any commit tagged as "AI-generated." Also updated my code review guidance for the team: AI code saves time on structure and boilerplate, but developers must own the correctness — reviewing AI-generated code is not optional.

---

## 4. KEY CONCEPTS TO KNOW (Quick Reference)

| Concept | What to Say |
|---|---|
| **Now Assist** | GenAI layer — LLM-powered summaries, drafts, and suggestions for agents, employees, and developers |
| **Virtual Agent** | Conversational chatbot using NLU — handles intents via structured topic flows with optional Now Assist for open Q&A |
| **NLU** | Natural Language Understanding — converts free text to intent + entity — better than keyword matching |
| **Predictive Intelligence** | ML-based prediction — auto-categorize, route, and prioritize based on historical patterns |
| **AI Agents** | Autonomous multi-step AI that interprets goals, decides actions, and executes workflows without scripted paths |
| **AI Control Tower** | Governance layer — enables/disables AI skills, monitors usage and quality, provides audit trail |
| **Now Assist for Creator** | AI-assisted development — code generation, flow design, test creation from natural language descriptions |
| **AI Agent Fabric** | Orchestration layer for multi-agent collaboration on complex tasks |
| **ServiceNow MCP** | Model Context Protocol — exposes ServiceNow actions to external AI tools via standardized protocol |
| **ServiceNow NowLLM** | ServiceNow-hosted LLM — AI processing stays in-tenant, no external data transmission |

---

## 5. METRICS / OUTCOMES TO QUOTE

- Now Assist incident summarization at Dell: agents report **8-10 minutes saved per complex incident** (shift handoffs, investigations)
- Now Assist resolution note completion: improved from **72% to 97%** after AI drafts were introduced
- Virtual Agent resolution rate: improved from **42% to 74%** over 6 months of NLU tuning and topic expansion
- Predictive Intelligence CSM case accuracy: improved from **45% to 73%** after data quality cleanup and retraining
- Now Assist for Employee Center: **~30% of HR policy questions** answered by AI without creating HR cases
- KB quality prerequisite: identified and retired **23 outdated HR KB articles** before AI deployment — preventing AI from propagating stale information
