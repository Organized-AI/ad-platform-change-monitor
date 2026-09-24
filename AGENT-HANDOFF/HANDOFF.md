# Agent Handoff Document

## Project: Ad Platform Change Monitor

### Quick Context

A Cloudflare Worker that monitors Google Ads accounts (v1) for changes made by your-email@example.com and sends Slack notifications.

**Origin:** Built after BLADE Google Ads incident (Dec 23, 2025) where conversion goals were modified and went unnoticed for 7 days.

### Key Files to Read

| Priority | File | Purpose |
|----------|------|---------|
| 1 | CLAUDE.md | Project overview |
| 2 | ARCHITECTURE/SYSTEM-DESIGN.md | Multi-platform architecture |
| 3 | SPECIFICATIONS/V1-MVP.md | Current scope |
| 4 | PLANNING/implementation-phases/ | Phase prompts |

### Current State

| Component | Status |
|-----------|--------|
| Planning | ✅ Complete |
| Credentials | ✅ Configured in .dev.vars |
| Phase 0-4 | ⏳ Not started |

### Next Steps

```bash
cd '/Users/supabowl/Library/Mobile Documents/com~apple~CloudDocs/BHT Promo iCloud/Organized AI/Windsurf/ad-platform-change-monitor'
claude --dangerously-skip-permissions
```

Then: `Read PLANNING/implementation-phases/PHASE-0-PROMPT.md and execute all tasks`

### Monitored Accounts

- Blade (1741833734)
- BiOptimizers (7994854565)
- RTT (2290369257)
- Teleios (6890103064)
