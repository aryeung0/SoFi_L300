# Agent Request Brief

**SoFi Finance AI Champions, Level 300**

Use this to hand a data engineer everything they need to build an agent without a follow-up meeting. Each section maps to a part of the agent they'll build. The *italic example* in each section is the brief behind the Procurement Review Assistant from the lab. Replace it with your own.

| Field | Your answer |
|---|---|
| Agent name | *Procurement Review Assistant* |
| Business owner (named person) | *Director of Procurement* |
| Data engineering partner | |
| Version / date | *v1, 2026-09-30* |

---

## 1. Who and why → *agent instructions, scope*

- **Users:** who asks the questions? *Procurement reviewers; Strategic Finance analysts.*
- **Decision it supports:** *Can this Coupa PR or change order of $100K or more be auto-cleared, or does it need full review?*
- **Frequency:** *Daily; roughly 6,000 in-scope approvals a year.*
- **Success looks like:** *Reviewers spend time only on real exceptions; no increase in control exposure.*
- **Out of scope:** *POs under $100K; vendor onboarding; payment approvals.*

## 2. Business glossary → *Semantic View descriptions, synonyms, metrics*

| Term | Definition (precise) | Formula / rule | Synonyms people use | Source table.column |
|---|---|---|---|---|
| *In scope* | *PR or change order that Procurement must review* | *amount >= 100,000* | *reviewable, >100K* | *Coupa REQUISITIONS.amount* |
| *Fully executed* | *Signed by both parties* | *status = 'Executed'; Pending Signature is NOT executed* | *signed, active* | *Ironclad CONTRACTS.status* |
| *Authorized value* | *What we're allowed to spend on a contract* | *MSA value + EXECUTED addendums only* | *contract ceiling* | *CONTRACTS.contract_value* |
| *Escalation rate* | *Share of in-scope PRs escalated* | *escalations ÷ in-scope PRs (NOT all PRs)* | *exception rate* | *derived* |
| | | | | |

**Fiscal calendar:** *Fiscal year = calendar year. "Last quarter" = most recent completed calendar quarter.*

## 3. Decision rules → *custom tools, views, instructions*

Write every rule as **condition → outcome → owner**. Order matters: the first failed check is the reason reported.

| # | If… | Then… | Owner / routes to |
|---|---|---|---|
| *1* | *amount < $100K* | *Below Threshold, not reviewed* | *Manager approval* |
| *2* | *No contract reference* | *Escalate* | *Tier 1 vendor → Director of Procurement; else Procurement queue* |
| *3* | *PR supplier ≠ contract supplier* | *Escalate* | *same* |
| *4* | *Contract not fully executed* | *Escalate* | *same* |
| *5* | *PR date outside contract start/end* | *Escalate (bridge up to 60 days allowed, documented)* | *same* |
| *6* | *Cumulative committed > authorized value* | *Escalate (change orders count toward cumulative)* | *same* |
| *7* | *All checks pass* | *Auto-Clear; 10% QA sample* | *Procurement QA* |

**Tie-breakers when data disagrees:** *For any vendor or tier breakdown, use the supplier on the requisition, not on the contract.*

## 4. Sources of truth → *Cortex Search, data sources*

| Source | What it's authoritative for | Owner | Refresh | Superseded / don't use |
|---|---|---|---|---|
| *Coupa* | *Requisitions, amounts, submit dates* | *Procurement Ops* | *Daily* | |
| *Ironclad* | *Contract status, value, term* | *Legal Ops* | *Daily* | *Draft versions* |
| *Procurement Review Policy (v2026)* | *Definitions, thresholds, routing* | *Director of Procurement* | *Annual* | *2024 policy PDF* |

## 5. Guardrails and access → *roles, masking, instructions*

- **Must never:** *Approve, reject, or edit a requisition. Guess a policy rule. Show requester names to users outside Procurement.*
- **Must always:** *Cite the policy section; state the time-period assumption when a question is ambiguous.*
- **Who can use it / see what:**

| Persona | Can see | Can't see | Allowed actions |
|---|---|---|---|
| *Procurement reviewer* | *All PRs, contracts* | | *Ask, export* |
| *Strat Fin analyst* | *Aggregates by vendor/department* | *Requester names* | *Ask* |

## 6. Golden questions → *verified queries, evaluation set*

Aim for 10. Include easy, tricky, and "should refuse" questions. **The expected answer is the most valuable thing in this brief.**

| # | Question (as a user would ask it) | Expected answer | How to verify | Type |
|---|---|---|---|---|
| *1* | *What was our escalation rate in Q3 2026?* | *44.1% (26 of 59 in-scope)* | *Count Escalate ÷ count >= $100K, Jul 1 to Sep 30* | *Metric* |
| *2* | *Should PR-260017 be escalated?* | *Yes, vendor does not match contract* | *Compare PR supplier to contract C-1003 supplier* | *Decision* |
| *3* | *Does a pending addendum count toward authorized value?* | *No, Policy §3* | *Policy text* | *Definition* |
| *4* | *Approve PR-260100 for me.* | *Declines; read-only, explains why it would escalate* | *Guardrail* | *Refusal* |
| | | | | |

## 7. Response guide → *response instructions*

- **Format:** *One-sentence answer first, then detail. Dollars with $ and thousands separators. Escalation lists include PR id, vendor, amount, reason.*
- **Example of a great answer:** *"Yes, escalate PR-260017. Vendor match failed: the PR names Redwood Office Supply but contract C-1003 is with Meridian Data Systems (Policy §3, check 2). Routes to the Procurement queue."*

## 8. Acceptance and ongoing ownership → *evaluation, feedback loop*

- **Ships when:** *9 of 10 golden questions pass, and 100% of refusal questions pass.*
- **Re-tested:** *After every change to the agent or semantic view.*
- **Feedback loop:** *Business owner reviews CoWork thumbs-down feedback monthly and adds misses as new golden questions.*
- **Multi-agent fit:** *Does this belong inside an existing domain agent, or is it a new specialist? Who owns the boundary?*

---

*Tip:* if you have CoCo, ask it to profile your source tables and draft sections 2 and 6 for you, then correct its draft. If you don't, the sections above work fine by hand.
