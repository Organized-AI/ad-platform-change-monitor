# Cloudflare Worker CLI Commands

Quick reference for Cloudflare Workers CLI operations.

## Development

```bash
# Start local dev server (uses .dev.vars for secrets)
wrangler dev

# Start with specific port
wrangler dev --port 8787

# Remote mode (uses production KV/secrets)
wrangler dev --remote
```

## Deployment

```bash
# Deploy to production
wrangler deploy

# Deploy to specific environment
wrangler deploy --env staging

# View deployment details
wrangler deployments list
```

## KV Storage

```bash
# Create KV namespace
wrangler kv:namespace create "MONITOR_STATE"

# Create preview namespace (for local dev)
wrangler kv:namespace create "MONITOR_STATE" --preview

# List namespaces
wrangler kv:namespace list

# Read from KV
wrangler kv:key get --binding=MONITOR_STATE "state"

# Write to KV
wrangler kv:key put --binding=MONITOR_STATE "state" '{"lastCheck":"..."}'

# Delete from KV
wrangler kv:key delete --binding=MONITOR_STATE "state"
```

## Secrets Management

```bash
# Add secret (interactive)
wrangler secret put GOOGLE_CLIENT_ID

# Add secret (non-interactive)
echo "value" | wrangler secret put SECRET_NAME

# List secrets
wrangler secret list

# Delete secret
wrangler secret delete SECRET_NAME
```

## Logs & Monitoring

```bash
# Tail live logs
wrangler tail

# Tail with filters
wrangler tail --format pretty
wrangler tail --status error
wrangler tail --search "change detected"

# View recent invocations
wrangler deployments list
```

## Cron Triggers

```bash
# Cron triggers are defined in wrangler.toml
# To test scheduled handler locally:
curl -X POST http://localhost:8787/__scheduled

# View cron configuration
cat wrangler.toml | grep -A2 "\[triggers\]"
```

## Testing Endpoints

```bash
# Health check
curl http://localhost:8787/health

# Manual trigger
curl -X POST http://localhost:8787/trigger

# Test Slack
curl -X POST http://localhost:8787/test-slack

# Get recent changes
curl http://localhost:8787/changes
```

## Troubleshooting

```bash
# Check wrangler version
wrangler --version

# Update wrangler
npm install -g wrangler@latest

# Whoami (check auth)
wrangler whoami

# Login
wrangler login
```
