# Case Study #2: Add Multi-Select Functionality to Retool

**Date:** 2025-10-06

**Category:** Decision Log

**Tags:** #Retool, #GraphQL, #Backend-Mutation, #Schema-Design, #Risk-Analysis  

**Status:** Deferred  

**Severity:** S3 – Medium

## 1. Summary
The Retool admin dashboard required a multi-select feature to mark multiple agent corrections as “resolved internally.” A frontend-only implementation was possible but introduced scalability and performance concerns due to multiple sequential GraphQL calls. A backend bulk mutation was designed but deferred due to schema complexity and deployment risk.

## 2. Context
This work centered on the `Agent Correction` table in the internal Retool dashboard.  

The table uses a GraphQL API backed by Django/Graphene mutations to update records.

Each correction item can be marked as `resolvedInternally` through an existing single-item mutation.

The need arose to enable multi-row selection for bulk resolution of corrections, which are frequent and often large in number.

## 3. Problem Statement
Manually resolving corrections one by one was inefficient for operations teams managing large volumes of flagged records.  
The existing mutation required N sequential requests, making it slow and error-prone.  
An initial attempt to implement client-side looping was tested but found too risky for production given load and partial failure possibilities.

## 4. Investigation

### Hypotheses
- **H1:** Retool can handle client-side looping efficiently enough for small batches.  
- **H2:** Backend bulk mutation needed for atomic updates and speed.  
- **H3:** Schema change may introduce risk due to permissions and filtering logic.

### Steps Taken
- Tested client-side batch update in Retool via JavaScript looping and GraphQL call.  
- Profiled network request duration on batches of 10–200 items.  
- Reviewed backend schema and existing mutation patterns.  
- Scoped the required changes for a bulk mutation to avoid unsafe client access.

### Key Evidence
- 200-item batch → ~40s total execution time (sequential calls).  
- Partial failures resulted in inconsistent UI state.  
- Backend mutation refactor would require schema update, new tests, and rollout coordination.

## 5. Decision
The team decided to defer backend schema changes and skip implementing client-side batch updates in production.

The feature was deemed too risky given performance concerns without backend refactor support.

## 6. Risk & Tradeoff Analysis

**Option 1 – Retool-Only Loop**  
*Benefit:* Immediate solution, no backend deployment.  
*Risk:* N separate API calls, slow for large batches, non-atomic updates.  
*Decision:* ❌ Deferred — acceptable only for local prototypes.

**Option 2 – Backend Bulk Mutation**  
*Benefit:* Single API call, atomic, efficient for large sets, cleaner UX.  
*Risk:* Requires schema addition, permission refactor, and testing.  
*Decision:* ✅ Accepted conceptually — deferred for a future backend release.

**Option 3 – Hybrid with Feature Flag**  
*Benefit:* Gradual rollout; can enable only for low-traffic clients.  
*Risk:* Added complexity, coordination across frontend/backends.  
*Decision:* ❌ Rejected — unnecessary overhead for limited benefit.

## 7. Mitigation & Handoff
- Documented test results and design proposal.  
- Added internal notes in GitHub issue.  
- Shared Loom demo with engineering leadership for future planning.  
- No production changes deployed.

## 8. Outcome & Learnings

**Outcome:**  
The multi-select feature was not released. The analysis clarified the performance bottleneck and shaped future design for bulk mutation APIs.

**Key Learnings:**  
- Client-side looping in Retool doesn’t scale for operational bulk actions.  
- Backend APIs should anticipate bulk mutation use cases early.  
- Atomic updates are essential for data integrity under concurrency.

## 9. Attachments
N/A

## 10. Reflection
This decision reinforced the value of limiting frontend workarounds in data-heavy admin tooling. It demonstrated that early backend extensibility—particularly around bulk operations—prevents reactive, high-risk patches later in the lifecycle.
