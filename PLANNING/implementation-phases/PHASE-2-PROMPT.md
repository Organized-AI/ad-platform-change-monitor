# Phase 2: Change Detection Logic

**Phase:** 2 of 4  
**Objective:** Implement change classification and deduplication  
**Prerequisites:** Phase 1 complete  
**Estimated Tasks:** 5

---

## Tasks

### Task 2.1: Create Change Classifier

Create `src/core/classifier.ts` with 15+ classification rules:

| Resource Type | Severity |
|---------------|----------|
| CAMPAIGN_CONVERSION_GOAL | 🔴 Critical |
| CUSTOMER_CONVERSION_GOAL | 🔴 Critical |
| CONVERSION_ACTION (status change) | 🔴 Critical |
| CAMPAIGN (status=PAUSED/REMOVED) | 🔴 Critical |
| CAMPAIGN_BUDGET (>50% decrease) | 🔴 Critical |
| CAMPAIGN_BUDGET (<50% change) | 🟡 Warning |
| Bid strategy changes | 🟡 Warning |
| AD_GROUP_AD | 🟢 Info |
| AD_GROUP_CRITERION | 🟢 Info |

### Task 2.2: Create Budget Analyzer

Create `src/core/budget-analyzer.ts` - calculates percentage changes, flags >50% as critical.

### Task 2.3: Create Deduplicator

Create `src/core/deduplicator.ts` - stores processed change IDs in KV (max 1000, FIFO).

### Task 2.4: Create Change Detector

Create `src/core/detector.ts` - orchestrates detection across multiple accounts.

### Task 2.5: Create State Manager

Manages lastCheckTimestamp and processedChangeIds in KV.

---

## Success Criteria

- [ ] Changes classified correctly by severity
- [ ] Budget decreases >50% flagged as critical
- [ ] No duplicate notifications
- [ ] Multi-account detection works
