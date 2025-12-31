# System Architecture

## Overview

The Ad Platform Change Monitor is designed with a plugin architecture to support multiple advertising platforms while maintaining a unified notification system.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Cloudflare Worker                                  │
│                        (Cron: Every 30 minutes)                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Platform Adapters                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐                    │
│  │  Google  │  │   Meta   │  │ LinkedIn │  │  TikTok  │                    │
│  │   Ads    │  │   Ads    │  │   Ads    │  │   Ads    │                    │
│  │ Adapter  │  │ Adapter  │  │ Adapter  │  │ Adapter  │                    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘                    │
│       │v1           │v2           │v2           │v2                         │
└───────┼─────────────┼─────────────┼─────────────┼───────────────────────────┘
        │             │             │             │
        └─────────────┴──────┬──────┴─────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     Unified Change Processor                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│  │  Normalizer  │──│  Classifier  │──│ Deduplicator │                      │
│  └──────────────┘  └──────────────┘  └──────────────┘                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Notification System                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│  │    Slack     │  │    Email     │  │   Webhook    │                      │
│  │   (v1+v2)    │  │    (v2)      │  │    (v2)      │                      │
│  └──────────────┘  └──────────────┘  └──────────────┘                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Cloudflare KV                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│  │ OAuth Tokens │  │ Change State │  │   Configs    │                      │
│  └──────────────┘  └──────────────┘  └──────────────┘                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Platform Adapters

Each advertising platform has its own adapter that implements a common interface:

```typescript
interface PlatformAdapter {
  platform: Platform;
  getRecentChanges(config: AdapterConfig): Promise<RawChange[]>;
  normalizeChange(raw: RawChange): UnifiedChange;
  getAccountName(accountId: string): string;
  getChangeHistoryUrl(accountId: string): string;
}

type Platform = 'google_ads' | 'meta_ads' | 'linkedin_ads' | 'tiktok_ads';
```

### 2. Unified Change Model

All platform-specific changes are normalized to a common schema:

```typescript
interface UnifiedChange {
  id: string;
  platform: Platform;
  accountId: string;
  accountName: string;
  timestamp: Date;
  userEmail: string;
  resourceType: string;
  operation: ChangeOperation;
  changedFields: string[];
  oldValues: Record<string, unknown>;
  newValues: Record<string, unknown>;
  severity: Severity;
  category: ChangeCategory;
  platformUrl: string;
  rawData?: unknown;
}
```

### 3. Classification Rules

| Category | Severity | Examples |
|----------|----------|----------|
| Conversion Goal/Pixel Changes | 🔴 Critical | Goal removed, pixel disabled |
| Campaign/Ad Set Deletion | 🔴 Critical | Campaign removed |
| Budget Decrease >50% | 🔴 Critical | $1000 → $400 |
| Status Changes | 🔴 Critical | Active → Paused |
| Budget Changes <50% | 🟡 Warning | $1000 → $800 |
| Bid Strategy Changes | 🟡 Warning | tCPA → Max Conversions |
| Audience Modifications | 🟡 Warning | Targeting changes |
| Creative Changes | 🟢 Info | Ad copy updates |
| Keyword Changes | 🟢 Info | Add/remove keywords |

## Platform-Specific Details

### Google Ads (v1)

| Aspect | Details |
|--------|---------|
| API | Google Ads API v15 |
| Auth | OAuth 2.0 (refresh token) |
| Endpoint | change_event resource |
| Rate Limit | 15,000 requests/day |
| Lookback | Max 30 days |

### Meta Ads (v2)

| Aspect | Details |
|--------|---------|
| API | Marketing API v18.0 |
| Auth | System User Token |
| Endpoint | Activity History |
| Rate Limit | Varies by access level |
| Lookback | 90 days |

### LinkedIn Ads (v2)

| Aspect | Details |
|--------|---------|
| API | Marketing API |
| Auth | OAuth 2.0 |
| Endpoint | Change Audit Logs |
| Rate Limit | 100 requests/day |
| Lookback | 30 days |

### TikTok Ads (v2)

| Aspect | Details |
|--------|---------|
| API | Business API |
| Auth | Access Token |
| Endpoint | Audit Log |
| Rate Limit | 600 requests/minute |
| Lookback | 30 days |

## Failure Modes

| Failure | Handling |
|---------|----------|
| OAuth token expired | Auto-refresh from refresh token |
| API rate limit | Exponential backoff, notify admin |
| Platform API down | Skip platform, continue others |
| KV write failure | Retry with backoff |
| Slack webhook failure | Log error, continue monitoring |
