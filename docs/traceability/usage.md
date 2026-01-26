---
title: Usage Guide
---

# Usage Guide

## Filter Logs by Session ID

1. Run a test that creates a session
2. Copy the session ID from the test output or Hub logs
3. Open the Grafana dashboard
4. Paste the session ID in the **Session ID** filter
5. View all Hub and Node logs for that session

## Trace a Request Flow

When debugging an issue:

1. Find a log entry related to your issue
2. Click on the log line to expand details
3. Copy the `traceId` value
4. Paste it in the **Trace ID** filter
5. See all related logs across Hub and Node

## Search for Errors

Use the **Filter** field to search for specific text:

- `error` - Find all error messages
- `timeout` - Find timeout-related logs
- `device allocated` - Find device allocation events

## Log Fields

Each log entry includes these fields (visible when clicking on a log line):

| Field | Description |
|-------|-------------|
| `service` | `hub` or `node` |
| `sessionId` | The Appium session ID |
| `traceId` | Distributed trace ID |
| `spanId` | Span ID within the trace |
| `deviceId` | Device identifier |
| `udid` | Device UDID |
| `log_level` | `info`, `warn`, `error`, `debug` |
| `log_message` | The actual log message |
