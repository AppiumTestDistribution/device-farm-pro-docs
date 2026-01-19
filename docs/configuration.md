---
title: Configuration
---

## Hub Configuration

The Hub orchestrates the grid. It requires a connection to PostgreSQL.

### hub.config.json

```json
{
  "webdriverSessionPath": "/wd/hub",
  "storage": {
    "type": "local",
    "local": {
      "basePath": ".cache/device-farm/logs"
    }
    // Options: "aws-s3", "cloudflare-r2"
  },
  "ssl": {
    "enabled": false,
    "cert": "./cert.pem",
    "key": "./key.pem"
  },
  "nodeMonitor": {
    "checkInterval": 10000,
    "timeoutMs": 40000
  },
  "websocket": {
    "heartbeatInterval": 30000,
    "nodeAvailabilityCheckTimeout": 60000,
    "sessionCreationTimeout": 120000,
    "sessionRequestTimeout": 30000,
    "nodeCleanupDelaySeconds": 30
  },
  "webrtc": {
    "browserDisconnectTimeout": 30000,
    "allowSessionReconnection": true
  }
}
```

### SSL Configuration

The Device Farm hub can run HTTPS directly without nginx. The hub server serves everything: API, WebSocket, and frontend.

**Configuration Options:**

| Field      | Type    | Default | Description                                                      |
| :--------- | :------ | :------ | :--------------------------------------------------------------- |
| `enabled`  | boolean | `false` | Enable HTTPS on the hub server                                   |
| `cert`     | string  | -       | Path to SSL certificate file (absolute path recommended)         |
| `key`      | string  | -       | Path to SSL private key file (absolute path recommended)         |

**What Gets Served:**

When you enable HTTPS on the hub server, everything runs on HTTPS:

- ✅ **Frontend UI** - Served directly by Express at `https://localhost:3000/`
- ✅ **API Endpoints** - All `/api/*` routes
- ✅ **WebSocket** - Socket.IO connections at `https://localhost:3000/socket.io`
- ✅ **WebDriver Endpoints** - Appium endpoints at `/wd/hub/*`

#### Generating SSL Certificates

For development, you can generate self-signed certificates:

```bash
# Create certs directory
mkdir -p certs
cd certs

# Generate self-signed certificate (for development)
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes \
  -subj "/C=US/ST=State/L=City/O=Device Farm/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"

# Set secure permissions
chmod 600 key.pem
chmod 644 cert.pem

cd ..
```

#### Configuration File Method

Configure SSL in `hub.config.json`:

```json
{
  "ssl": {
    "enabled": true,
    "cert": "/absolute/path/to/certs/cert.pem",
    "key": "/absolute/path/to/certs/key.pem"
  }
}
```

#### Environment Variables Method

Alternatively, you can use environment variables:

```bash
# Enable HTTPS
export SSL_ENABLED=true
export SSL_CERT=$(pwd)/certs/cert.pem
export SSL_KEY=$(pwd)/certs/key.pem

# Optional: Certificate Authority bundle
# export SSL_CA=$(pwd)/certs/ca.pem

# Optional: Private key passphrase (if your key is encrypted)
# export SSL_PASSPHRASE=your_passphrase
```

#### Production Certificates

For production, use certificates from a trusted CA (e.g., Let's Encrypt):

```bash
# Install certbot
sudo apt-get install certbot  # or brew install certbot

# Get certificate for your domain
sudo certbot certonly --standalone -d your-domain.com

# Use the certificates
export SSL_CERT=/etc/letsencrypt/live/your-domain.com/fullchain.pem
export SSL_KEY=/etc/letsencrypt/live/your-domain.com/privkey.pem
```

Or configure in `hub.config.json`:

```json
{
  "ssl": {
    "enabled": true,
    "cert": "/etc/letsencrypt/live/your-domain.com/fullchain.pem",
    "key": "/etc/letsencrypt/live/your-domain.com/privkey.pem"
  }
}
```

#### Node Configuration for HTTPS

When the hub uses HTTPS, nodes must connect using HTTPS as well. Update your `node.config.json`:

```json
{
  "hubUrl": "https://localhost:3000",
  "accessKey": "your-access-key",
  "token": "your-token"
}
```

Or via environment:

```bash
export HUB_URL=https://localhost:3000
```

#### Troubleshooting

**Browser Shows "Not Secure" Warning:**

This is normal for self-signed certificates. Click "Advanced" → "Proceed to localhost" (Chrome) or add an exception.

**Certificate File Not Found:**

```bash
# Check files exist
ls -la certs/cert.pem
ls -la certs/key.pem

# Use absolute paths in config files
export SSL_CERT=$(pwd)/certs/cert.pem
export SSL_KEY=$(pwd)/certs/key.pem
```

**Node Can't Connect to Hub:**

- Ensure `hubUrl` in node config uses `https://` not `http://`
- If using self-signed certs, you may need to disable SSL verification in the HTTP client (not recommended for production)

### Node Monitor Configuration

Controls health monitoring of connected nodes to detect disconnections and failures. This is critical for maintaining system reliability in a distributed environment.

**Why It's Needed:**

In a distributed system with hub-node architecture, nodes can fail or become unreachable for many reasons:

- **Hardware Failures**: Node machine crashes, power loss, or hardware malfunction
- **Network Issues**: Network disconnections, firewall changes, or connectivity problems
- **Process Crashes**: Node process terminates unexpectedly due to bugs or resource exhaustion
- **System Overload**: Node becomes unresponsive due to high CPU/memory usage
- **Silent Failures**: WebSocket connection appears active but node is actually frozen or hung

Without active monitoring, the hub would:

- Continue sending requests to dead nodes (wasting time waiting for timeouts)
- Show incorrect device availability in the dashboard (devices on dead nodes appear available)
- Leave sessions in limbo (active sessions on failed nodes never get cleaned up)
- Cause test failures and poor user experience

**How It Works:**

1. **Periodic Health Checks**: Every 10 seconds, the hub sends a ping to all connected nodes
2. **Response Monitoring**: Each node must respond within 40 seconds (4x check interval for network tolerance)
3. **Failure Detection**: If a node doesn't respond within timeout, it's marked offline
4. **Automatic Cleanup**: Hub triggers cleanup of all devices and sessions on the failed node
5. **Recovery**: When node comes back online, it's automatically detected and re-registered

**Configuration Options:**

| Field           | Type   | Default | Description                                                                                                                                                                        |
| :-------------- | :----- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `checkInterval` | number | `10000` | How often to check node health in milliseconds (default: 10 seconds). The hub periodically pings all connected nodes to ensure they're responsive.                                 |
| `timeoutMs`     | number | `40000` | Maximum time to wait for node response before marking as offline (default: 40 seconds). If a node doesn't respond within this timeout, it's marked offline and cleanup is started. |

**Recommended Settings:**

- **`timeoutMs`** should be 3-4x `checkInterval` to tolerate temporary network delays
- **Default (10s/40s)**: Good balance for most environments - detects failures quickly without false positives
- **Unstable networks (15s/60s)**: Longer intervals to avoid marking nodes offline due to temporary network hiccups
- **High-reliability (5s/20s)**: Faster detection in stable networks with redundant nodes

### WebSocket Configuration

Controls WebSocket communication between hub and nodes, including heartbeats and request timeouts.

**Why Heartbeat Is Needed:**

WebSocket connections can appear "open" even when the connection is actually dead due to:

- **Network middleboxes** (firewalls, NAT, proxies) closing idle connections without notification
- **TCP timeouts** where the connection dies silently without proper close handshake
- **One-way communication loss** where hub can send but node can't respond (or vice versa)

Without heartbeats:

- Dead connections wouldn't be detected until a command is sent (wasting time)
- Idle connections would be closed by firewalls/proxies (breaking communication)
- Hub would only learn about node failures when trying to create a session (bad user experience)

**Heartbeat vs Node Monitor:**

These work together but serve different purposes:

| Feature       | Heartbeat (`heartbeatInterval`)                  | Node Monitor (`nodeMonitor.checkInterval`)            |
| :------------ | :----------------------------------------------- | :---------------------------------------------------- |
| **Direction** | Node → Hub (node sends, hub listens)             | Hub → Node (hub sends ping, node responds)          |
| **Purpose**   | Keep WebSocket alive, passive connection check   | Active health check with timeout enforcement          |
| **Frequency** | Every 30 seconds (more frequent)                 | Every 10 seconds (more frequent for faster detection) |
| **Failure**   | Hub marks node offline if no heartbeat received  | Hub marks node offline if ping times out              |
| **Use Case**  | Prevents idle connection timeouts, detects drops | Actively verifies node is responsive and processing   |

**Configuration Options:**

| Field                          | Type   | Default  | Description                                                                                                                                                                                                                                                                                                                           |
| :----------------------------- | :----- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `heartbeatInterval`            | number | `30000`  | How often nodes send heartbeat messages to hub in milliseconds (default: 30 seconds). Heartbeats keep the WebSocket connection alive and help detect network issues. Should be less than firewall idle timeout (typically 60-120 seconds).                                                                                            |
| `nodeAvailabilityCheckTimeout` | number | `60000`  | Maximum time to wait for node availability before sending request (default: 60 seconds). Used when querying if a node is ready to accept commands.                                                                                                                                                                                    |
| `sessionCreationTimeout`       | number | `120000` | Timeout for session creation requests in milliseconds (default: 120 seconds). Session creation can take time due to app installation, device preparation, and Appium startup.                                                                                                                                                         |
| `sessionRequestTimeout`        | number | `30000`  | Timeout for regular session requests like click, getText, etc. in milliseconds (default: 30 seconds).                                                                                                                                                                                                                                 |
| `nodeCleanupDelaySeconds`      | number | `30`     | Delay in seconds before cleaning up sessions and devices for permanently offline nodes (default: 30 seconds). When a node disconnects, the hub waits this duration before terminating sessions. If the node reconnects within this time, cleanup is cancelled. Can also be set via `NODE_CLEANUP_DELAY_SECONDS` environment variable. |

**Recommended Heartbeat Settings:**

- **Default (30s)**: Works well with most firewalls/proxies (typically timeout after 60-120s of inactivity)
- **Aggressive (15s)**: Very strict networks with short idle timeouts
- **Relaxed (60s)**: Internal networks with no firewall restrictions (not recommended for production)
- **Never disable**: Heartbeat is critical for connection health - always keep enabled

### WebRTC Configuration

Controls WebRTC connections for manual device sessions, including browser disconnect handling and reconnection support.

| Field                      | Type    | Default | Description                                                                                                                                                                                  |
| :------------------------- | :------ | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `browserDisconnectTimeout` | number  | `30000` | Grace period in milliseconds before terminating session after browser disconnect (default: 30 seconds). During this period, users can reconnect to their session without losing their work.  |
| `allowSessionReconnection` | boolean | `true`  | Whether to allow session reconnection within grace period (default: `true`). Set to `false` to immediately terminate sessions on browser disconnect instead of waiting for the grace period. |

## Node Configuration

The Node manages the devices. It auto-detects connected USB devices.

### node.config.json

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

### Device Reconciliation

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

### Session Idle Detection (Manual Sessions)

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

### App Cleanup Configuration

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
