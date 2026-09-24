# Phase 3: Slack Integration

**Phase:** 3 of 4  
**Objective:** Build Slack notification system with rich formatting  
**Prerequisites:** Phase 2 complete  
**Estimated Tasks:** 4

---

## Tasks

### Task 3.1: Create Slack Webhook Client

Create `src/notifications/slack/client.ts` - HTTP client with retry logic.

### Task 3.2: Create Message Formatters

Create `src/notifications/slack/messages.ts`:

**Critical Alert Format:**
```
🔴 CRITICAL: CAMPAIGN_CONVERSION_GOAL

Account:     Blade
User:        your-email@example.com
Operation:   UPDATE
Time:        Dec 30, 2025, 2:51 PM

Changed Fields: `category, biddable`

[📊 View in Google Ads]
```

**Summary Format:**
```
📋 Google Ads Change Monitor Summary

Total Changes:  3
🔴 Critical:    1
🟡 Warning:     2

By Account:
• Blade: 2 change(s)
• BiOptimizers: 1 change(s)
```

### Task 3.3: Create Notification Orchestrator

Create `src/notifications/slack/notifier.ts` - sends individual alerts + summary.

### Task 3.4: Create Budget Change Formatter

Shows "$X → $Y (-Z%)" format for budget changes.

---

## Success Criteria

- [ ] Slack messages send successfully
- [ ] Rich formatting displays correctly
- [ ] "View in Google Ads" button works
- [ ] Summary includes all accounts
