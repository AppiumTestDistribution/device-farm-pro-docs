---
title: Slack Setup
---

Send notifications to your team's Slack channels using incoming webhooks.

## Step 1: Create a Slack Webhook

1. Go to [Slack API](https://api.slack.com/apps) and create a new app
2. Navigate to **Incoming Webhooks** and enable it
3. Click **Add New Webhook to Workspace**
4. Select the channel for notifications
5. Copy the webhook URL

## Step 2: Add Integration in Device Farm

1. Navigate to **Settings > Integrations**
2. Click **Add Integration**
3. Select **Slack**
4. Enter a name (e.g., "QA Team Notifications")
5. Paste the webhook URL
6. Optionally specify a channel override
7. Click **Save**

## Step 3: Create Notification Rules

1. Expand your integration by clicking the chevron
2. Click **Add Rule** or use **Quick Daily Report** for a pre-configured daily report
3. Select the event type
4. For scheduled reports, select a schedule preset or enter a custom cron expression
5. Click **Save**

## Slack Message Examples

### Build Complete Notification

```
✅ Build Passed
Build: my-regression-suite

Results Summary
• Total Sessions: 100
• Passed: 95 ✅
• Failed: 5 ❌
• Pass Rate: 95% ⭐
• Duration: 1h 30m ⏱️

📱 Devices Used
🥇 Pixel 6 - 30 sessions
🥈 iPhone 14 - 25 sessions
🥉 Galaxy S22 - 20 sessions

[View Build Details]
```

### Daily Report Notification

```
📊 Daily Report - 2026-01-30

Summary
• Total Tests: 250
• Passed: 230 ✅
• Failed: 20 ❌
• Pass Rate: 92% ⭐
• Avg Duration: 5m 30s

📱 Top Devices
🥇 Pixel 7 - 45 sessions
🥈 iPhone 15 Pro - 38 sessions
🥉 Galaxy S23 - 32 sessions

[View Analytics]
```

## Testing Your Integration

### Test Connection

Click the **Send** icon (paper airplane) on any integration to send a test message. This verifies your webhook URL is correct.

### Send Report Now

Click the **Chart** icon on any integration to immediately send a report with the last 30 days of data. Useful for testing report formatting without waiting for the scheduled time.

## Troubleshooting

### Notifications not arriving

1. Verify the webhook URL is correct and starts with `https://hooks.slack.com/`
2. Check that the integration is enabled
3. Check that the notification rule is enabled
4. View the notification logs for error messages (click **View Logs** in the Integrations header)
