# Case Study: <Short, Descriptive Title>

**Date:** YYYY-MM-DD  
**Category:** <Bug / Risk Assessment / Incident / Investigation / Decision Log>  
**Tags:** <e.g. API, Celery, Database, Caching, Deployment>  
**Status:** <Completed / Deferred / Ongoing>  
**Severity:** <S1–S4 or Low–Critical>

## 1. Summary
A brief, high-level overview of the issue or scenario (2–3 sentences).  
Focus on **what happened**, **why it mattered**, and **what was at stake**.

_Example:_  
“Observed duplicate webhook deliveries in the payment pipeline causing inconsistent downstream states. Investigated the source and deferred the hotfix due to migration risk during peak hours.”

## 2. Context
Describe the system, service, or environment involved.  
Include relevant background such as architecture components, dependencies, and recent changes.

_Example:_  
“This affected the `payments-webhook` microservice responsible for emitting transaction updates to downstream billing systems. The service runs on Celery workers with Redis as a broker and uses a PostgreSQL event log for idempotency.”

## 3. Problem Statement
Outline the symptoms, measurable impact, and detection context.  
Include metrics, alerts, or reports that indicated the issue.

_Example:_  
“Error rate increased from 0.3% to 12% on webhook deliveries between 09:00–11:00 UTC. Duplicate events led to double billing for 18 customers. Detected via SLO breach on ‘successful webhook deliveries <99%’.”

## 4. Investigation
Summarize your thought process and evidence collected.

### Hypotheses
List initial and refined hypotheses.

- **H1:** Network retries causing duplicates  
- **H2:** Consumer missing idempotency header  
- **H3:** Race condition during concurrent ACKs  

### Steps Taken
- Collected logs and traces from service instances  
- Reproduced scenario in staging using replayed payloads  
- Compared database entries pre- and post-incident  
- Eliminated network layer as root cause after latency profiling  

### Key Evidence
Include brief log excerpts, metrics snapshots, or query results.

## 5. Decision
State the conclusion reached and reasoning.

_Example:_  
“Identified missing idempotency key on consumer side. Decided against immediate hotfix due to schema migration risk during peak traffic. Opted for temporary flag-based mitigation.”


## 6. Risk & Tradeoff Analysis
Analyze the potential impact of each possible action. Outline the benefit, risk, and final decision for each.

**Option 1 – <Describe First Option>**  
*Benefit:* <Summarize the main advantage or desired outcome.>  
*Risk:* <Summarize potential negative consequences or failure modes.>  
*Decision:* <Accepted / Deferred / Rejected> — <Short explanation for why.>

**Option 2 – <Describe Second Option>**  
*Benefit:* <Summarize the main advantage or desired outcome.>  
*Risk:* <Summarize potential negative consequences or failure modes.>  
*Decision:* <Accepted / Deferred / Rejected> — <Short explanation for why.>

**Option 3 – <Describe Third Option>**  
*Benefit:* <Summarize the main advantage or desired outcome.>  
*Risk:* <Summarize potential negative consequences or failure modes.>  
*Decision:* <Accepted / Deferred / Rejected> — <Short explanation for why.>

_Add additional options as needed. Keep analysis concise but explicit about reasoning and tradeoffs._


## 7. Mitigation & Handoff
Describe the temporary or permanent mitigations applied and how the issue was documented or handed off.

- Added feature flag `ENABLE_WEBHOOK_DEDUP` (default ON)  
- Updated on-call runbook with new alert condition  
- Opened follow-up ticket for schema change and test coverage  
- Communicated risk summary to team leads  

## 8. Outcome & Learnings
Summarize impact and lessons learned.

**Outcome:**  
Error rate reduced to <1% within 2 hours. No new duplicates observed.

**Key Learnings:**  
- Add idempotency validation to consumer endpoints  
- Avoid peak-hour DB schema migrations  
- Automate alert correlation to reduce detection time  

## 9. Attachments (Optional)
- `repro_steps.md` – Sanitized reproduction guide  
- `diagram.png` – Architecture or flow visualization  
- `logs.txt` – Redacted logs or metrics screenshots  

## 10. Reflection
Short, candid commentary on how the process improved your judgment or system understanding.

_Example:_  
“This case reinforced the value of staging load tests and rollback safety reviews. Clearer decision criteria prevented a risky migration and helped formalize our incident response flow.”
