# Phase 0: Project Setup

**Phase:** 0 of 4  
**Objective:** Initialize the Cloudflare Worker project with all dependencies and configuration  
**Prerequisites:** None  
**Estimated Tasks:** 4

---

## Tasks

### Task 0.1: Initialize Node.js Project

Create `package.json`:

```json
{
  "name": "ad-platform-change-monitor",
  "version": "1.0.0",
  "description": "Multi-platform ad change monitoring with Slack notifications",
  "main": "src/index.ts",
  "scripts": {
    "dev": "wrangler dev",
    "deploy": "wrangler deploy",
    "tail": "wrangler tail",
    "test": "vitest"
  },
  "dependencies": {
    "google-auth-library": "^9.0.0"
  },
  "devDependencies": {
    "@cloudflare/workers-types": "^4.20231218.0",
    "typescript": "^5.3.3",
    "wrangler": "^3.22.1",
    "vitest": "^1.1.0"
  }
}
```

Run: `npm install`

---

### Task 0.2: Create TypeScript Configuration

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "ESNext",
    "moduleResolution": "node",
    "lib": ["ES2021"],
    "types": ["@cloudflare/workers-types"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

### Task 0.3: Create Wrangler Configuration

Create `wrangler.toml`:

```toml
name = "ad-platform-change-monitor"
main = "src/index.ts"
compatibility_date = "2024-01-01"

# Cron trigger - runs every 30 minutes
[triggers]
crons = ["*/30 * * * *"]

# KV namespace for state storage
[[kv_namespaces]]
binding = "MONITOR_STATE"
id = "YOUR_KV_NAMESPACE_ID"

# Environment variables
[vars]
MONITORED_USER_EMAIL = "jordan@bluehighlightedtext.com"
POLLING_INTERVAL_MINUTES = "30"
LOGIN_CUSTOMER_ID = "4761832056"
MONITORED_ACCOUNTS = "1741833734,7994854565,2290369257,6890103064"
ACCOUNT_NAMES = "Blade,BiOptimizers,RTT,Teleios"
```

---

### Task 0.4: Create Directory Structure and Entry Point

Create directory structure and type definitions in `src/types/index.ts` and entry point in `src/index.ts`.

---

## Success Criteria

- [ ] `npm install` completes without errors
- [ ] `npm run dev` starts the worker locally
- [ ] TypeScript compiles without errors
- [ ] Health check endpoint returns valid JSON

---

## Git Commit

```bash
git add .
git commit -m "Phase 0: Project setup complete"
```
