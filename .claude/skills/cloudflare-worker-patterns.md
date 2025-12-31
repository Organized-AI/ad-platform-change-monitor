# Cloudflare Worker Patterns

Best practices for this Cloudflare Worker project.

## Project Structure

```
src/
├── index.ts              # Worker entry with scheduled() and fetch()
├── types/                # TypeScript types
├── adapters/             # Platform API clients (plugin architecture)
│   └── google-ads/       # v1: Google Ads adapter
├── core/                 # Business logic
│   ├── classifier.ts     # Severity classification
│   ├── deduplicator.ts   # Prevent duplicate alerts
│   └── detector.ts       # Change detection
└── notifications/        # Output channels
    └── slack/            # Slack webhook integration
```

## Handler Patterns

### Scheduled Handler (Cron)

```typescript
export default {
  async scheduled(
    controller: ScheduledController,
    env: Env,
    ctx: ExecutionContext
  ): Promise<void> {
    const startTime = Date.now();
    
    try {
      // 1. Get state from KV
      const state = await getState(env.MONITOR_STATE);
      
      // 2. Fetch changes from all platforms
      const changes = await detectChanges(env, state);
      
      // 3. Classify and deduplicate
      const newChanges = deduplicateChanges(changes, state.processedIds);
      
      // 4. Send notifications
      await sendNotifications(env, newChanges);
      
      // 5. Update state
      await updateState(env.MONITOR_STATE, newChanges);
      
      console.log(`✅ Complete in ${Date.now() - startTime}ms`);
    } catch (error) {
      console.error('❌ Monitor failed:', error);
      // Don't throw - let cron continue next interval
    }
  }
};
```

### HTTP Handler (Manual Triggers)

```typescript
async fetch(request: Request, env: Env): Promise<Response> {
  const url = new URL(request.url);
  
  switch (url.pathname) {
    case '/health':
      return json({ status: 'healthy', ... });
    
    case '/trigger':
      if (request.method !== 'POST') {
        return json({ error: 'Method not allowed' }, 405);
      }
      // Run detection manually
      return json({ message: 'Triggered', ... });
    
    default:
      return new Response('Not Found', { status: 404 });
  }
}
```

## KV State Management

```typescript
interface MonitorState {
  lastCheckTimestamp: string;
  processedChangeIds: string[];  // Max 1000, FIFO
}

async function getState(kv: KVNamespace): Promise<MonitorState> {
  const data = await kv.get('monitor-state', 'json');
  return data || { lastCheckTimestamp: '', processedChangeIds: [] };
}

async function updateState(
  kv: KVNamespace,
  newIds: string[]
): Promise<void> {
  const state = await getState(kv);
  
  // Add new IDs, keep max 1000 (FIFO)
  state.processedChangeIds = [
    ...newIds,
    ...state.processedChangeIds
  ].slice(0, 1000);
  
  state.lastCheckTimestamp = new Date().toISOString();
  
  await kv.put('monitor-state', JSON.stringify(state));
}
```

## OAuth Token Refresh

```typescript
async function getAccessToken(env: Env, kv: KVNamespace): Promise<string> {
  // Check cache first
  const cached = await kv.get('oauth-token', 'json');
  if (cached && cached.expiresAt > Date.now() + 300000) {
    return cached.accessToken;
  }
  
  // Refresh token
  const response = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      client_id: env.GOOGLE_CLIENT_ID,
      client_secret: env.GOOGLE_CLIENT_SECRET,
      refresh_token: env.GOOGLE_REFRESH_TOKEN,
      grant_type: 'refresh_token'
    })
  });
  
  const data = await response.json();
  
  // Cache with expiry
  await kv.put('oauth-token', JSON.stringify({
    accessToken: data.access_token,
    expiresAt: Date.now() + (data.expires_in * 1000)
  }));
  
  return data.access_token;
}
```

## Error Handling

```typescript
// Wrap external calls with retry
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * Math.pow(2, i)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Use in API calls
const changes = await withRetry(() => 
  googleAdsClient.getChanges(env, since)
);
```

## Response Helpers

```typescript
function json(data: unknown, status = 200): Response {
  return new Response(JSON.stringify(data, null, 2), {
    status,
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 'no-store'
    }
  });
}
```

## Environment Type Safety

```typescript
// src/types/index.ts
export interface Env {
  // KV Bindings
  MONITOR_STATE: KVNamespace;
  
  // Secrets (via wrangler secret put)
  GOOGLE_CLIENT_ID: string;
  GOOGLE_CLIENT_SECRET: string;
  GOOGLE_REFRESH_TOKEN: string;
  GOOGLE_DEVELOPER_TOKEN: string;
  SLACK_WEBHOOK_URL: string;
  
  // Variables (via wrangler.toml [vars])
  MONITORED_USER_EMAIL: string;
  MONITORED_ACCOUNTS: string;
  ACCOUNT_NAMES: string;
  LOGIN_CUSTOMER_ID: string;
}
```

## Testing Locally

```bash
# Start dev server (uses .dev.vars)
wrangler dev

# Test cron handler
curl -X POST http://localhost:8787/__scheduled

# Test endpoints
curl http://localhost:8787/health
curl -X POST http://localhost:8787/trigger
```
