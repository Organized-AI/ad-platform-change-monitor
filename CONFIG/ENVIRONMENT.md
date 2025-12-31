# Environment Configuration

## Secrets (via wrangler secret put)

| Secret | Description |
|--------|-------------|
| GOOGLE_CLIENT_ID | OAuth 2.0 Client ID from Google Cloud Console |
| GOOGLE_CLIENT_SECRET | OAuth 2.0 Client Secret |
| GOOGLE_REFRESH_TOKEN | Long-lived refresh token for API access |
| GOOGLE_DEVELOPER_TOKEN | Google Ads API developer token |
| SLACK_WEBHOOK_URL | Incoming webhook URL for notifications |

## Setup Commands

```bash
# Add each secret interactively
wrangler secret put GOOGLE_CLIENT_ID
wrangler secret put GOOGLE_CLIENT_SECRET
wrangler secret put GOOGLE_REFRESH_TOKEN
wrangler secret put GOOGLE_DEVELOPER_TOKEN
wrangler secret put SLACK_WEBHOOK_URL
```

## KV Namespace

```bash
wrangler kv:namespace create "MONITOR_STATE"
# Copy the ID to wrangler.toml
```

## Local Development

For local development, create `.dev.vars` in the project root:

```bash
# .dev.vars (gitignored - never commit!)
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REFRESH_TOKEN=your-refresh-token
GOOGLE_DEVELOPER_TOKEN=your-developer-token
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/xxx/xxx/xxx
```

## Getting Credentials

### Google Ads API

1. **Google Cloud Console**: Create OAuth 2.0 credentials at https://console.cloud.google.com/apis/credentials
2. **Developer Token**: Apply at https://ads.google.com/aw/apicenter
3. **Refresh Token**: Use OAuth playground or google-auth-library to generate

### Slack Webhook

1. Go to https://api.slack.com/apps
2. Create new app → From scratch
3. Add "Incoming Webhooks" feature
4. Activate and add to channel
5. Copy webhook URL
