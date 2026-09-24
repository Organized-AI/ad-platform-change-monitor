# Ad Platform Change Monitor

A multi-platform advertising change monitoring system that tracks account modifications and sends real-time Slack notifications.

## Project Vision

**v1 (Current):** Google Ads change monitoring  
**v2 (Planned):** Multi-platform support (Meta, LinkedIn, TikTok)

## Quick Context

This tool was born from a real incident: a conversion goal change in a Google Ads account went unnoticed for 7 days, impacting campaign performance. The goal is to catch these changes within 30 minutes.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Runtime | Cloudflare Workers |
| Language | TypeScript |
| Storage | Cloudflare KV |
| Scheduling | Cron Triggers |
| Notifications | Slack Webhooks |
| APIs | Google Ads API (v1), Meta Marketing API (v2), LinkedIn Marketing API (v2), TikTok Business API (v2) |

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | This file - project overview |
| `ARCHITECTURE/SYSTEM-DESIGN.md` | Multi-platform architecture |
| `SPECIFICATIONS/V1-MVP.md` | v1 Google Ads scope |
| `SPECIFICATIONS/V2-ROADMAP.md` | v2 multi-platform roadmap |
| `PLANNING/IMPLEMENTATION-MASTER-PLAN.md` | Detailed implementation plan |
| `PLANNING/implementation-phases/` | Phase-by-phase prompts |

## Commands

```bash
# Development
npm run dev          # Local dev server
npm test             # Run tests

# Deployment
npm run deploy       # Deploy to Cloudflare

# Monitoring
wrangler tail        # View live logs
```

## Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check + config |
| `/trigger` | POST | Manual detection run |
| `/changes` | GET | Get recent changes |
| `/test-slack` | POST | Test Slack connection |

## Alert Severity

| Level | Emoji | Triggers |
|-------|-------|----------|
| Critical | 🔴 | Conversion goals, campaign removal, budget -50%+ |
| Warning | 🟡 | Budget changes, bid strategy |
| Info | 🟢 | Ad/keyword changes |

## Environment Variables

```toml
# wrangler.toml [vars]
MONITORED_USER_EMAIL = "your-email@example.com"
MONITORED_ACCOUNTS = "1741833734,7994854565,..."
ACCOUNT_NAMES = "Blade,BiOptimizers,..."
LOGIN_CUSTOMER_ID = "4761832056"

# Secrets (via wrangler secret put)
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
GOOGLE_REFRESH_TOKEN
GOOGLE_DEVELOPER_TOKEN
SLACK_WEBHOOK_URL
```

## Project Status

- [x] Planning complete
- [ ] Phase 0: Project setup
- [ ] Phase 1: Google Ads API client
- [ ] Phase 2: Change detection logic
- [ ] Phase 3: Slack integration
- [ ] Phase 4: Cloudflare Worker integration
- [ ] Phase 5: Testing & deployment

## Architecture Principles

1. **Platform Agnostic Core** - Detection and notification logic is platform-independent
2. **Plugin Architecture** - Each ad platform is a pluggable adapter
3. **Unified Change Model** - All platforms normalize to a common change schema
4. **Graceful Degradation** - One platform failure doesn't affect others
