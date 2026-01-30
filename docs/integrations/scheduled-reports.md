---
title: Scheduled Reports
---

Configure automated daily and weekly reports to keep your team informed about test results.

## Report Types

### Daily Report

Provides a summary of test results for the previous 24 hours, including:

- Total tests run
- Passed and failed counts
- Pass rate percentage
- Average test duration
- Top 5 most-used devices

### Weekly Report

Provides a summary of test results for the previous 7 days with the same metrics as the daily report.

## Schedule Configuration

For Daily and Weekly reports, you can configure when they run using cron expressions.

### Preset Schedules

The UI provides common presets for quick setup:

| Schedule | Cron Expression |
|----------|-----------------|
| Daily at 9 AM | `0 9 * * *` |
| Daily at 6 PM | `0 18 * * *` |
| Weekdays at 9 AM | `0 9 * * 1-5` |
| Weekly (Monday 9 AM) | `0 9 * * 1` |
| Weekly (Friday 5 PM) | `0 17 * * 5` |
| First of Month (9 AM) | `0 9 1 * *` |

### Custom Cron Expressions

Cron format: `minute hour day-of-month month day-of-week`

Examples:

- `0 8 * * *` - Every day at 8:00 AM
- `30 17 * * 1-5` - Weekdays at 5:30 PM
- `0 9 1,15 * *` - 1st and 15th of each month at 9:00 AM

The UI shows a human-readable description of your cron expression and the next scheduled run time.

### Cron Field Reference

| Field | Values | Special Characters |
|-------|--------|-------------------|
| Minute | 0-59 | * , - |
| Hour | 0-23 | * , - |
| Day of Month | 1-31 | * , - |
| Month | 1-12 | * , - |
| Day of Week | 0-6 (0=Sunday) | * , - |

## Quick Daily Report

For convenience, each integration has a **Quick Daily Report** button that creates a pre-configured daily report rule running at 9 AM.

## Viewing Notification History

Click **View Logs** in the Integrations header to see a history of all sent notifications, including:

- Timestamp
- Integration name
- Event type
- Status (sent/failed/pending)
- Error message (if failed)

## Timezone

!!! note "Server Time"
    Reports run based on server time (UTC). Plan your cron expressions accordingly.
