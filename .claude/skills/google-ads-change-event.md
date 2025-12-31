# Google Ads Change Event API

Reference for querying Google Ads change_event resource.

## GAQL Query Structure

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
WHERE change_event.change_date_time >= '{start_time}'
  AND change_event.change_date_time <= '{end_time}'
  AND change_event.user_email = '{user_email}'
ORDER BY change_event.change_date_time DESC
LIMIT 1000
```

## Time Format

Google Ads API requires: `YYYY-MM-DD HH:MM:SS` (no timezone, assumes account timezone)

```typescript
function formatGoogleAdsTime(date: Date): string {
  return date.toISOString()
    .replace('T', ' ')
    .replace('Z', '')
    .slice(0, 19);
}
```

## Resource Types to Monitor

| Resource Type | Severity | Description |
|---------------|----------|-------------|
| CAMPAIGN_CONVERSION_GOAL | 🔴 Critical | Goal assignment changes |
| CUSTOMER_CONVERSION_GOAL | 🔴 Critical | Account-level goal changes |
| CONVERSION_ACTION | 🔴 Critical | Conversion action config |
| CAMPAIGN | 🔴 Critical | Campaign status changes |
| CAMPAIGN_BUDGET | 🔴/🟡 | Budget changes (>50% = critical) |
| AD_GROUP | 🟡 Warning | Ad group changes |
| AD_GROUP_AD | 🟢 Info | Ad creative changes |
| AD_GROUP_CRITERION | 🟢 Info | Keyword/audience changes |

## API Request

```typescript
async function queryChanges(
  accessToken: string,
  customerId: string,
  loginCustomerId: string,
  query: string
): Promise<ChangeEvent[]> {
  const response = await fetch(
    `https://googleads.googleapis.com/v15/customers/${customerId}/googleAds:searchStream`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'developer-token': DEVELOPER_TOKEN,
        'login-customer-id': loginCustomerId,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ query })
    }
  );
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(`Google Ads API error: ${JSON.stringify(error)}`);
  }
  
  const data = await response.json();
  return parseChangeEvents(data);
}
```

## Response Parsing

```typescript
interface RawChangeEvent {
  changeEvent: {
    changeDateTime: string;
    changeResourceType: string;
    changedFields: string;
    clientType: string;
    newResource: string;
    oldResource: string;
    resourceChangeOperation: string;
    resourceName: string;
    userEmail: string;
  };
}

function parseChangeEvents(response: any[]): ChangeEvent[] {
  const events: ChangeEvent[] = [];
  
  for (const batch of response) {
    if (batch.results) {
      for (const result of batch.results) {
        events.push({
          changeDateTime: result.changeEvent.changeDateTime,
          changeResourceType: result.changeEvent.changeResourceType,
          changedFields: result.changeEvent.changedFields,
          clientType: result.changeEvent.clientType,
          newResource: result.changeEvent.newResource || '',
          oldResource: result.changeEvent.oldResource || '',
          resourceChangeOperation: result.changeEvent.resourceChangeOperation,
          resourceName: result.changeEvent.resourceName,
          userEmail: result.changeEvent.userEmail
        });
      }
    }
  }
  
  return events;
}
```

## Client Types

| Value | Description |
|-------|-------------|
| GOOGLE_ADS_WEB_CLIENT | Google Ads UI |
| GOOGLE_ADS_AUTOMATED_RULE | Automated rules |
| GOOGLE_ADS_SCRIPTS | Google Ads Scripts |
| GOOGLE_ADS_API | API client |

## Operation Types

| Value | Description |
|-------|-------------|
| CREATE | New resource created |
| UPDATE | Resource modified |
| REMOVE | Resource deleted |

## Parsing Changed Fields

The `changedFields` field contains a FieldMask string:

```typescript
function parseChangedFields(fieldMask: string): string[] {
  if (!fieldMask) return [];
  return fieldMask.split(',').map(f => f.trim());
}

// Example: "status,primaryConversionGoal.category"
// Returns: ["status", "primaryConversionGoal.category"]
```

## Budget Change Detection

```typescript
function analyzeBudgetChange(
  oldResource: string,
  newResource: string
): { oldAmount: number; newAmount: number; percentChange: number } | null {
  try {
    const oldData = JSON.parse(oldResource);
    const newData = JSON.parse(newResource);
    
    const oldAmount = parseInt(oldData.amountMicros) / 1_000_000;
    const newAmount = parseInt(newData.amountMicros) / 1_000_000;
    
    const percentChange = ((newAmount - oldAmount) / oldAmount) * 100;
    
    return { oldAmount, newAmount, percentChange };
  } catch {
    return null;
  }
}
```

## Limitations

- **Lookback**: Max 30 days of change history
- **Rate Limits**: 15,000 requests/day per developer token
- **Streaming**: Use searchStream for large result sets
- **Field Mask**: Some changes show minimal field info

## Change History URL

```typescript
function getChangeHistoryUrl(customerId: string): string {
  return `https://ads.google.com/aw/history?ocid=${customerId}`;
}
```
