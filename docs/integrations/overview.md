---
title: Overview
---

Device Farm supports Slack and Email integrations to keep your team informed about test results, build status, and infrastructure health.

## Supported Integrations

| Type | Description |
|------|-------------|
| **Slack** | Send notifications to Slack channels via webhooks |
| **Email** | Send notifications via SMTP to team members |

## Notification Events

You can configure notifications for the following events:

### Real-time Alerts

| Event | Description |
|-------|-------------|
| **Session Failed** | Triggered when a test session fails |
| **Session Timeout** | Triggered when a test session times out |
| **Node Offline** | Triggered when a device node goes offline |
| **Node Online** | Triggered when a device node comes back online |
| **Build Started** | Triggered when a new build is created |
| **Build Complete** | Triggered when a build finishes with pass/fail summary |
| **Device Flagged** | Triggered when a device is flagged for issues |

### Scheduled Reports

| Report | Description |
|--------|-------------|
| **Daily Report** | Summary of test results for the previous day |
| **Weekly Report** | Summary of test results for the previous week |

Reports include:

- Total tests, passed, and failed counts
- Pass rate with color-coded status
- Average test duration
- Top 5 most-used devices

## Color Coding

Notifications use color coding based on pass rate:

| Pass Rate | Color | Status |
|-----------|-------|--------|
| >= 90% | Green | Passing |
| 70-89% | Yellow | Warning |
| < 70% | Red | Failing |
| No tests | Gray | No data |

## Getting Started

1. **[Set up Slack Integration](slack-setup.md)** - Send notifications to Slack channels
2. **[Set up Email Integration](email-setup.md)** - Send notifications via SMTP
3. **[Configure Scheduled Reports](scheduled-reports.md)** - Set up daily and weekly reports
