# Ad Platform Change Monitor - Claude Code Quick Start

## Project Overview

Build a Cloudflare Worker that monitors advertising accounts for changes made by a specific user and sends Slack notifications. v1 focuses on Google Ads, with v2 expanding to Meta, LinkedIn, and TikTok.

## Quick Start Command

```bash
cd '/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf/ad-platform-change-monitor'
claude --dangerously-skip-permissions
```

Then in Claude Code:
```
Read PLANNING/IMPLEMENTATION-MASTER-PLAN.md and PLANNING/implementation-phases/PHASE-0-PROMPT.md, then execute Phase 0.
```

## Phase Execution

Execute phases in order:

| Phase | Command |
|-------|---------|
| 0 | `Read PLANNING/implementation-phases/PHASE-0-PROMPT.md and execute all tasks` |
| 1 | `Read PLANNING/implementation-phases/PHASE-1-PROMPT.md and execute all tasks` |
| 2 | `Read PLANNING/implementation-phases/PHASE-2-PROMPT.md and execute all tasks` |
| 3 | `Read PLANNING/implementation-phases/PHASE-3-PROMPT.md and execute all tasks` |
| 4 | `Read PLANNING/implementation-phases/PHASE-4-PROMPT.md and execute all tasks` |

## Prerequisites ✅ COMPLETE

1. **Slack Webhook URL** ✅ Configured in `.dev.vars`
2. **Google Ads API Credentials** ✅ Configured in `.dev.vars`
   - OAuth Client ID & Secret
   - Refresh Token
   - Developer Token
3. **Cloudflare Account** ✅ Already connected via MCP

## Key Configuration

```
Monitored Email: your-email@example.com
Accounts: Blade, BiOptimizers, RTT, Teleios
Polling: Every 30 minutes
```

## Expected Outcome

After all phases complete:
- Worker deployed to Cloudflare
- Checks Google Ads every 30 minutes
- Sends Slack alerts for critical/warning changes
- No duplicate notifications
