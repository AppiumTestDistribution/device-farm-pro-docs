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

  "sessionIdleDetection": {
    "enabled": true,
    "idleTimeoutMs": 300000,
    "checkIntervalMs": 60000
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

## Session Idle Detection

Session idle detection automatically terminates sessions that have been inactive for a configured period. This prevents resource waste from abandoned or forgotten test sessions.

**How It Works:**

1. **Activity Tracking**: The node tracks the last command timestamp for each active session
2. **Periodic Checks**: Every 60 seconds (configurable), the node scans all sessions for inactivity
3. **Idle Detection**: If a session hasn't received any commands for longer than the timeout, it's marked as idle
4. **Automatic Cleanup**: Idle sessions are automatically terminated with full cleanup (app uninstall, video upload, device release)

**Configuration Options:**

| Field             | Type    | Default  | Description                                       |
| :---------------- | :------ | :------- | :------------------------------------------------ |
| `enabled`         | boolean | `true`   | Enable/disable idle session detection             |
| `idleTimeoutMs`   | number  | `300000` | Maximum idle time before termination (5 minutes)  |
| `checkIntervalMs` | number  | `60000`  | How often to check for idle sessions (60 seconds) |

**When It's Needed:**

- **CI/CD Pipelines**: Terminates sessions if a test suite crashes or hangs without properly closing the session
- **Manual Testing**: Automatically cleans up when testers forget to end their session or close the browser unexpectedly
- **Network Issues**: Handles cases where client loses connection but session remains active on the node
- **Resource Management**: Prevents devices from being locked indefinitely by stale sessions in shared environments
- **Cost Optimization**: In cloud/paid device farms, reduces unnecessary device usage time

**When to Adjust:**

- **Shorter timeout (2-3 minutes)**: High-demand environments where devices need quick turnover
- **Longer timeout (10-15 minutes)**: Manual testing sessions where testers need time to analyze results between commands
- **Disable (`enabled: false`)**: Only in development environments where you want full control over session lifecycle

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
