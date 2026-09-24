# V1 MVP Specification: Google Ads Change Monitor

## Overview

Version 1 focuses exclusively on Google Ads change monitoring with Slack notifications.

## Scope

### In Scope ✅

- Google Ads change detection via `change_event` API
- Filter by user email (your-email@example.com)
- Multi-account monitoring (4 accounts)
- Severity classification (critical, warning, info)
- Slack notifications for critical/warning changes
- Deduplication to prevent repeat alerts
- 30-minute polling interval
- Manual trigger endpoint
- Health check endpoint

### Out of Scope ❌ (v2)

- Meta Ads monitoring
- LinkedIn Ads monitoring
- TikTok Ads monitoring
- Email notifications
- Custom webhook notifications
- Web dashboard
- Historical change browser
- Custom classification rules
- Multi-user monitoring

## Monitored Accounts

| Account | Customer ID | Type |
|---------|-------------|------|
| Blade | 1741833734 | Client |
| BiOptimizers | 7994854565 | Client |
| RTT (Marisa Peer) | 2290369257 | Client |
| Teleios Health | 6890103064 | Client |

## Alert Classification

### 🔴 Critical Alerts (Immediate)

| Change Type | Detection Method |
|-------------|------------------|
| Conversion goal modified | `CAMPAIGN_CONVERSION_GOAL` resource |
| Conversion action removed | `CONVERSION_ACTION` + DELETE operation |
| Conversion action disabled | `status` field change |
| Campaign removed | `CAMPAIGN` + DELETE operation |
| Campaign paused | `status` field = PAUSED |
| Budget decreased >50% | `amountMicros` comparison |

### 🟡 Warning Alerts (Batched)

| Change Type | Detection Method |
|-------------|------------------|
| Budget changed <50% | `CAMPAIGN_BUDGET` resource |
| Bid strategy changed | `biddingStrategy` field |
| Audience modified | `AD_GROUP_CRITERION` resource |

## Technical Requirements

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1 | Poll Google Ads API every 30 minutes | P0 |
| FR-2 | Filter changes by monitored user email | P0 |
| FR-3 | Classify changes by severity | P0 |
| FR-4 | Send Slack notifications for critical changes | P0 |
| FR-5 | Deduplicate to prevent repeat notifications | P0 |
| FR-6 | Support manual trigger via HTTP endpoint | P1 |
| FR-7 | Provide health check endpoint | P1 |
| FR-8 | Handle OAuth token refresh automatically | P0 |

## Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| Detection rate | 100% of monitored changes | Audit against UI change history |
| Notification delivery | >99% | Slack webhook success rate |
| False positive rate | <5% | Manual review of alerts |
| Latency | <5 min | Timestamp comparison |
