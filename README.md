# Ad Platform Change Monitor

Watches ad accounts for changes and posts alerts to Slack. Version 1 covers Google Ads through its `change_event` API, polling every 30 minutes on Cloudflare Workers with KV for deduplication. Meta, LinkedIn and TikTok are planned for v2.

Start with [CLAUDE.md](CLAUDE.md), then [SPECIFICATIONS/V1-MVP.md](SPECIFICATIONS/V1-MVP.md) and [PLANNING/IMPLEMENTATION-MASTER-PLAN.md](PLANNING/IMPLEMENTATION-MASTER-PLAN.md).

---

Maintained by Jordaaan Hill ([LinkedIn](https://www.linkedin.com/in/jordaaanhill)).
