# Ad Platform Change Monitor - Implementation Master Plan

**Created:** December 30, 2025  
**Project Path:** `/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf/ad-platform-change-monitor`  
**Runtime:** Node.js/TypeScript + Cloudflare Workers  
**Repository:** TBD (organized-ai or jhillbht)
**Version:** v1 (Google Ads only) → v2 (Multi-platform)

---

## Related Documents

| Document | Location |
|----------|----------|
| Project Overview | `CLAUDE.md` |
| System Architecture | `ARCHITECTURE/SYSTEM-DESIGN.md` |
| v1 Specification | `SPECIFICATIONS/V1-MVP.md` |
| v2 Roadmap | `SPECIFICATIONS/V2-ROADMAP.md` |
| Environment Config | `CONFIG/ENVIRONMENT.md` |
| Agent Handoff | `AGENT-HANDOFF/HANDOFF.md` |

---

## Problem Statement

Critical Google Ads changes (conversion goal modifications, budget changes, campaign status changes) can go unnoticed until performance degrades. The BLADE incident on Dec 23, 2025 demonstrated how a goal configuration change could impact conversion tracking without immediate visibility.

## Solution Overview

A 24/7 monitoring system that:
1. Polls Google Ads change_event API every 30 minutes
2. Filters for changes made by specific users (your-email@example.com)
3. Sends Slack notifications for critical changes
4. Provides endpoints for manual triggers and health checks

---

## Monitored Accounts

| Account Name | Customer ID | Login Customer ID |
|--------------|-------------|-------------------|
| Blade | 1741833734 | 4761832056 |
| BiOptimizers | 7994854565 | 4761832056 |
| RTT (Marisa Peer) | 2290369257 | 4761832056 |
| Teleios Health | 6890103064 | 4761832056 |

---

## Implementation Phases Overview

| Phase | Name | Components | Dependencies |
|-------|------|------------|--------------|
| 0 | Project Setup | package.json, wrangler.toml, tsconfig | None |
| 1 | Google Ads API Client | OAuth handler, GAQL query builder | Phase 0 |
| 2 | Change Detection Logic | Polling, filtering, deduplication | Phase 1 |
| 3 | Slack Integration | Webhook client, message formatting | Phase 0 |
| 4 | Cloudflare Worker | Cron handler, KV storage | Phases 1-3 |

---

## File Structure (v2-Ready)

```
ad-platform-change-monitor/
├── CLAUDE.md                      # Project overview
├── .claude/                       # Claude Code configuration
│   ├── agents/                    
│   ├── commands/                  
│   ├── hooks/                     
│   └── settings.json              
├── PLANNING/
│   ├── IMPLEMENTATION-MASTER-PLAN.md
│   └── implementation-phases/
│       ├── PHASE-0-PROMPT.md
│       ├── PHASE-1-PROMPT.md
│       ├── PHASE-2-PROMPT.md
│       ├── PHASE-3-PROMPT.md
│       └── PHASE-4-PROMPT.md
├── ARCHITECTURE/
│   └── SYSTEM-DESIGN.md
├── SPECIFICATIONS/
│   ├── V1-MVP.md
│   └── V2-ROADMAP.md
├── CONFIG/
│   └── ENVIRONMENT.md
├── AGENT-HANDOFF/
│   └── HANDOFF.md
├── src/
│   ├── index.ts
│   ├── types/
│   ├── adapters/google-ads/
│   ├── core/
│   └── notifications/slack/
├── wrangler.toml
├── package.json
└── tsconfig.json
```

---

## Environment Variables

```toml
# wrangler.toml [vars]
MONITORED_USER_EMAIL = "your-email@example.com"
POLLING_INTERVAL_MINUTES = "30"
LOGIN_CUSTOMER_ID = "4761832056"
MONITORED_ACCOUNTS = "1741833734,7994854565,2290369257,6890103064"
ACCOUNT_NAMES = "Blade,BiOptimizers,RTT,Teleios"
```

## Secrets (via wrangler secret put)
- GOOGLE_CLIENT_ID
- GOOGLE_CLIENT_SECRET
- GOOGLE_REFRESH_TOKEN
- GOOGLE_DEVELOPER_TOKEN
- SLACK_WEBHOOK_URL
