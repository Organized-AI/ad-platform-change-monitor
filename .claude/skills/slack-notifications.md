# Slack Notification Patterns

Reference for sending rich Slack messages via webhooks.

## Webhook Client

```typescript
async function sendSlackMessage(
  webhookUrl: string,
  message: SlackMessage
): Promise<boolean> {
  try {
    const response = await fetch(webhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(message)
    });
    
    if (!response.ok) {
      console.error('Slack error:', await response.text());
      return false;
    }
    
    return true;
  } catch (error) {
    console.error('Slack send failed:', error);
    return false;
  }
}
```

## Message Types

### Critical Alert

```typescript
function createCriticalAlert(change: ProcessedChange): SlackMessage {
  return {
    blocks: [
      {
        type: 'header',
        text: {
          type: 'plain_text',
          text: `🔴 CRITICAL: ${change.resourceType}`,
          emoji: true
        }
      },
      {
        type: 'section',
        fields: [
          { type: 'mrkdwn', text: `*Account:*\n${change.accountName}` },
          { type: 'mrkdwn', text: `*User:*\n${change.userEmail}` },
          { type: 'mrkdwn', text: `*Operation:*\n${change.operation}` },
          { type: 'mrkdwn', text: `*Time:*\n${formatTime(change.timestamp)}` }
        ]
      },
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `*Changed Fields:* \`${change.changedFields.join(', ')}\``
        }
      },
      {
        type: 'actions',
        elements: [
          {
            type: 'button',
            text: { type: 'plain_text', text: '📊 View in Google Ads' },
            url: getChangeHistoryUrl(change.accountId),
            style: 'primary'
          }
        ]
      }
    ]
  };
}
```

### Warning Alert

```typescript
function createWarningAlert(change: ProcessedChange): SlackMessage {
  return {
    blocks: [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `🟡 *WARNING:* ${change.resourceType} in *${change.accountName}*`
        }
      },
      {
        type: 'context',
        elements: [
          {
            type: 'mrkdwn',
            text: `${change.userEmail} • ${formatTime(change.timestamp)} • ${change.operation}`
          }
        ]
      }
    ]
  };
}
```

### Summary Message

```typescript
function createSummaryMessage(
  changes: ProcessedChange[],
  duration: number
): SlackMessage {
  const critical = changes.filter(c => c.severity === 'critical').length;
  const warning = changes.filter(c => c.severity === 'warning').length;
  
  const byAccount = changes.reduce((acc, c) => {
    acc[c.accountName] = (acc[c.accountName] || 0) + 1;
    return acc;
  }, {} as Record<string, number>);
  
  return {
    blocks: [
      {
        type: 'header',
        text: {
          type: 'plain_text',
          text: '📋 Google Ads Change Monitor Summary',
          emoji: true
        }
      },
      {
        type: 'section',
        fields: [
          { type: 'mrkdwn', text: `*Total Changes:*\n${changes.length}` },
          { type: 'mrkdwn', text: `*Check Duration:*\n${duration}ms` },
          { type: 'mrkdwn', text: `*🔴 Critical:*\n${critical}` },
          { type: 'mrkdwn', text: `*🟡 Warning:*\n${warning}` }
        ]
      },
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: '*By Account:*\n' + 
            Object.entries(byAccount)
              .map(([name, count]) => `• ${name}: ${count} change(s)`)
              .join('\n')
        }
      },
      {
        type: 'context',
        elements: [
          {
            type: 'mrkdwn',
            text: `Monitored user: jordan@bluehighlightedtext.com | ${new Date().toLocaleString()}`
          }
        ]
      }
    ]
  };
}
```

### Budget Change Alert

```typescript
function createBudgetAlert(
  change: ProcessedChange,
  oldAmount: number,
  newAmount: number,
  percentChange: number
): SlackMessage {
  const emoji = percentChange < -50 ? '🔴' : percentChange < 0 ? '🟡' : '🟢';
  const direction = percentChange < 0 ? 'decreased' : 'increased';
  
  return {
    blocks: [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `${emoji} *Budget ${direction}* in *${change.accountName}*`
        }
      },
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `$${oldAmount.toLocaleString()} → $${newAmount.toLocaleString()} (${percentChange.toFixed(1)}%)`
        }
      }
    ]
  };
}
```

### No Changes Message

```typescript
function createNoChangesMessage(): SlackMessage {
  return {
    blocks: [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: '✅ No changes detected in monitored accounts.'
        }
      },
      {
        type: 'context',
        elements: [
          { type: 'mrkdwn', text: `Last check: ${new Date().toLocaleString()}` }
        ]
      }
    ]
  };
}
```

## Rate Limiting

Slack recommends max 1 message per second:

```typescript
async function sendWithRateLimit(
  webhookUrl: string,
  messages: SlackMessage[]
): Promise<void> {
  for (const message of messages) {
    await sendSlackMessage(webhookUrl, message);
    await new Promise(r => setTimeout(r, 500)); // 500ms delay
  }
}
```

## Notification Strategy

```typescript
async function notifyChanges(
  env: Env,
  changes: ProcessedChange[],
  duration: number
): Promise<void> {
  const messages: SlackMessage[] = [];
  
  // Individual alerts for critical changes
  const criticalChanges = changes.filter(c => c.severity === 'critical');
  for (const change of criticalChanges) {
    messages.push(createCriticalAlert(change));
  }
  
  // Batched alerts for warnings
  const warnings = changes.filter(c => c.severity === 'warning');
  if (warnings.length > 0) {
    // Group warnings into single message
    messages.push(createBatchedWarnings(warnings));
  }
  
  // Summary at end
  if (changes.length > 0) {
    messages.push(createSummaryMessage(changes, duration));
  }
  
  await sendWithRateLimit(env.SLACK_WEBHOOK_URL, messages);
}
```

## Testing

```typescript
// Test Slack connection
async function testSlack(env: Env): Promise<Response> {
  const success = await sendSlackMessage(env.SLACK_WEBHOOK_URL, {
    blocks: [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: '🧪 *Test message from Ad Platform Change Monitor*\n\nSlack integration is working!'
        }
      }
    ]
  });
  
  return new Response(JSON.stringify({ success }), {
    headers: { 'Content-Type': 'application/json' }
  });
}
```
