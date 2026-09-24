# Phase 1: Google Ads API Client

**Phase:** 1 of 4  
**Objective:** Build the Google Ads API client with OAuth token management  
**Prerequisites:** Phase 0 complete  
**Estimated Tasks:** 6

---

## Tasks

### Task 1.1: Create OAuth Token Manager

Create `src/adapters/google-ads/oauth.ts` - handles token refresh and caching in KV.

### Task 1.2: Create GAQL Query Builder

Create `src/adapters/google-ads/queries.ts` - builds change_event queries with date filters.

### Task 1.3: Create Google Ads API Client

Create `src/adapters/google-ads/client.ts` - main client for API requests.

### Task 1.4: Create Account Utilities

Create `src/adapters/google-ads/accounts.ts` - account name resolution.

### Task 1.5: Create Time Range Calculator

Utility for calculating query time ranges (formats as YYYY-MM-DD HH:MM:SS).

### Task 1.6: Write Unit Tests

Test query building and token refresh logic.

---

## Key Query

```sql
SELECT 
  change_event.change_date_time,
  change_event.change_resource_type,
  change_event.changed_fields,
  change_event.client_type,
  change_event.new_resource,
  change_event.old_resource,
  change_event.resource_change_operation,
  change_event.resource_name,
  change_event.user_email
FROM change_event
WHERE change_event.change_date_time >= 'START'
  AND change_event.change_date_time <= 'END'
  AND change_event.user_email = 'your-email@example.com'
ORDER BY change_event.change_date_time DESC
```

---

## Success Criteria

- [ ] OAuth token refresh works
- [ ] GAQL queries build correctly
- [ ] API client connects to Google Ads
- [ ] Account names resolve properly
