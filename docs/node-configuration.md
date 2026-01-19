---
title: Node Configuration
---

The Node manages the devices. It auto-detects connected USB devices.

## node.config.json

```json
{
  "hubUrl": "http://localhost:3000",
  "accessKey": "YOUR_ACCESS_KEY",
  "token": "YOUR_TOKEN",
  "name": "Node-01",
  "platform": "both",
  "iosDeviceType": "real",
  "androidDeviceType": "both",

  "maxSessions": 5,
  "heartbeatInterval": 10000,

  "appiumServer": {
    "enabled": true,
    "serverArgs": {
      "port": 4723,
      "use-drivers": ["uiautomator2", "xcuitest"],
      "allow-cors": true,
      "relaxed-security": true
    }
  },

  "deviceReconciliation": {
    "enabled": true,
    "intervalMs": 60000
  },

  "androidCleanUpApps": ["com.example.test"],
  "iosCleanUpApps": ["com.example.test"],
  "wdaResignedIpaPath": "/path/to/signed-wda.ipa"
}
```

## Device Reconciliation

Device Reconciliation is a periodic background process that ensures device state stays synchronized between the node and hub. It acts as a failsafe to catch missed device connection/disconnection events.

**How It Works:**

1. **Periodic Polling**: Every 60 seconds (by default), the reconciliation service queries the actual device state from ADB (Android) or simctl/usbmux (iOS)
2. **State Comparison**: Compares actual device state with what's tracked in the system
3. **Automatic Correction**:
   - Devices connected but not tracked → automatically registered
   - Devices tracked but disconnected → marked as inactive
4. **Non-Intrusive**: Works alongside real-time event tracking without interfering

**Configuration Options:**

| Field        | Type    | Default | Description                                           |
| :----------- | :------ | :------ | :---------------------------------------------------- |
| `enabled`    | boolean | `true`  | Enable/disable device reconciliation                  |
| `intervalMs` | number  | `60000` | How often to reconcile device state (in milliseconds) |

**When to Adjust:**

- **Default (60 seconds)**: Good balance between responsiveness and system load
- **High-frequency changes (30 seconds)**: If devices are frequently connected/disconnected
- **Stable environments (120 seconds)**: If devices rarely change and you want to reduce overhead
- **Disable**: Set `enabled: false` only if you're experiencing issues with reconciliation

**Benefits:**

- Catches missed ADB/simctl tracker events (network issues, daemon restarts)
- Ensures eventual consistency within the configured interval
- Handles edge cases like unauthorized devices or physical disconnections
- Automatically recovers from node crashes or restarts

## Session Idle Detection (Manual Sessions)

For **manual/interactive sessions** accessed through the dashboard UI, Device Farm uses a frontend-driven idle detection system. This ensures accurate detection of user activity while preventing false timeouts.

**How It Works:**

1. **Frontend Activity Tracking**: The browser tracks actual user activity (mouse movements, keyboard input, touch events, scrolling)
2. **Heartbeat Mechanism**: The frontend sends periodic heartbeats (every 30 seconds) to the hub to indicate the session is still active
3. **Idle Warning**: When no activity is detected for a configurable period, a warning modal appears with a countdown
4. **User Can Continue**: Users can click "Continue Session" to reset the timer and keep working
5. **Automatic Cleanup**: If the user doesn't respond, the session is terminated with full cleanup

**Browser Disconnect Handling:**

- When the browser tab/window is closed, a disconnect signal is immediately sent to the hub
- The hub terminates the session and releases the device without waiting for timeouts
- This uses the `sendBeacon` API for reliable delivery during page unload

**Hub Safety Net:**

The hub has an orphaned session cleanup cron job as a safety net. If heartbeats stop (e.g., browser crashes, network failure), the hub will automatically clean up orphaned sessions after 5 minutes.

**Default Timeouts:**

| Component | Default | Description |
| :-------- | :------ | :---------- |
| Frontend idle timeout | 5 minutes | Time before idle warning appears |
| Warning countdown | 30 seconds | Time user has to click "Continue Session" |
| Heartbeat interval | 30 seconds | How often frontend sends heartbeats |
| Hub orphan threshold | 5 minutes | Safety net cleanup for sessions without heartbeats |

**What Sessions Are Affected?**

- ✅ **Manual Sessions**: Dashboard UI live device control sessions
- ❌ **Automation Sessions**: Regular WebDriver/Appium test scripts are NOT affected. They manage their own lifecycle through Appium's `newCommandTimeout` capability

:::note
The node-side `sessionIdleDetection` configuration has been deprecated. Idle detection is now handled by the frontend and hub for more accurate activity tracking.
:::

## App Cleanup Configuration

You can specify apps to automatically uninstall after each session:

**Configuration Options:**

| Field                | Type     | Default | Description                                      |
| :------------------- | :------- | :------ | :----------------------------------------------- |
| `androidCleanUpApps` | string[] | `[]`    | Android package names to uninstall after session |
| `iosCleanUpApps`     | string[] | `[]`    | iOS bundle IDs to uninstall after session        |

**Default Cleanup Apps:**

- **Android**: Automatically uninstalls `io.appium.uiautomator2.server` and `io.appium.uiautomator2.server.test` (UiAutomator2 driver apps)
- **iOS/tvOS**: Automatically uninstalls the WebDriverAgent (WDA) bundle ID from the database

**Note**: User-specified cleanup apps are added to the default driver apps, not replaced.
