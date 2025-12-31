# Phase 4: Cloudflare Worker Integration

**Phase:** 4 of 4  
**Objective:** Wire everything together and deploy  
**Prerequisites:** Phases 1-3 complete  
**Estimated Tasks:** 6

---

## Tasks

### Task 4.1: Create Main Worker

Update `src/index.ts` with full scheduled() and fetch() handlers.

### Task 4.2: Create KV Namespace

```bash
wrangler kv:namespace create "MONITOR_STATE"
# Update wrangler.toml with returned ID
```

### Task 4.3: Configure Secrets

```bash
wrangler secret put GOOGLE_CLIENT_ID
wrangler secret put GOOGLE_CLIENT_SECRET
wrangler secret put GOOGLE_REFRESH_TOKEN
wrangler secret put GOOGLE_DEVELOPER_TOKEN
wrangler secret put SLACK_WEBHOOK_URL
```

### Task 4.4: Create Deployment Script

Create `scripts/deploy.sh` with pre-flight checks.

### Task 4.5: Implement All Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /health | GET | Health check |
| /trigger | POST | Manual run |
| /changes | GET | Recent changes |
| /test-slack | POST | Test notification |

### Task 4.6: Deploy and Verify

```bash
npm run deploy
wrangler tail  # Monitor logs
```

---

## Success Criteria

- [ ] Worker deploys successfully
- [ ] Cron trigger fires every 30 minutes
- [ ] Manual trigger works
- [ ] Slack notifications arrive
- [ ] No duplicate alerts

---

## Post-Deployment

1. Monitor first 24 hours via `wrangler tail`
2. Make a test change in Google Ads UI
3. Verify notification received within 35 minutes
4. Check Cloudflare Dashboard → Workers → Triggers
