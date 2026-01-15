---
title: Troubleshooting
hide:
  - navigation
---

## Common Issues

### Node Connection & Offline Issues

**Problem**: Node shows as offline in dashboard or logs show "Node did not reconnect in time"

**Causes**:
- Node process crashed or stopped
- Network connectivity issues between Hub and Node
- Heartbeat timeout exceeded (default: 120 seconds)

**Resolution**:
1. Check if node process is running: `ps aux | grep farm-runner`
2. Verify network connectivity: `ping <hub-host>`
3. Check node logs for errors or crashes
4. Restart the node: `npm start` (in packages/node directory)
5. Adjust heartbeat timeout if needed in `hub.config.json`:
   ```json
   {
     "nodeMonitor": {
       "timeoutMs": 180000  // Increase to 180 seconds if needed
     }
   }
   ```

---

### Devices Not Detected

**Android Devices Not Showing Up**

**Symptoms**: Devices connected but not appearing in dashboard

**Causes**:
- ADB server not running or device not authorized
- `ANDROID_HOME` or `ANDROID_SDK_ROOT` not set
- Device in "unauthorized" or "offline" state

**Resolution**:
1. Check ADB connection: `adb devices` (should show "device", not "unauthorized" or "offline")
2. If unauthorized, accept the authorization prompt on the device
3. Set environment variables:
   ```bash
   export ANDROID_HOME=/path/to/android/sdk
   export ANDROID_SDK_ROOT=/path/to/android/sdk
   ```
4. Restart ADB server: `adb kill-server && adb start-server`
5. Wait for device reconciliation (runs every 60 seconds) or restart the node

**iOS Devices Not Showing Up**

**Symptoms**: iOS devices/simulators not appearing in dashboard

**Causes**:
- Xcode Command Line Tools not installed
- Device not trusted
- Simulator not booted (if only using booted simulators)

**Resolution**:
1. Install Xcode Command Line Tools: `xcode-select --install`
2. For real devices:
   - Accept "Trust This Computer" dialog on the device
   - Check device list: `xcrun devicectl device list`
3. For simulators:
   - Check simulators: `xcrun simctl list devices`
   - Boot simulator if needed: `xcrun simctl boot <udid>`
   - Configure `bootedSimulators: false` in `node.config.json` to show all simulators
4. Wait for device reconciliation or restart the node

---

### Session Creation Failures

**Problem**: "Session creation timeout" or "No free device matching capabilities found"

**Common Scenarios**:

1. **No devices available**
   - All devices are busy with other sessions
   - Check available devices: GET `/api/devices`
   - Wait for a device to become free or add more devices

2. **Capability mismatch**
   - Requested platform/version doesn't match any device
   - Verify capabilities match available devices:
     ```json
     {
       "platformName": "android",  // Must be lowercase: "android" or "ios"
       "platformVersion": "13",    // Must match device version
       "deviceName": ".*"          // Use regex for flexible matching
     }
     ```

3. **Session creation timeout (default: 120 seconds)**
   - Device slow to respond or Appium startup slow
   - Increase timeout in `hub.config.json`:
     ```json
     {
       "websocket": {
         "sessionCreationTimeout": 180000  // 180 seconds
       }
     }
     ```

4. **Appium server failed to start**
   - Check node logs for Appium errors
   - Verify Appium installation: `npm list appium` (in packages/node)
   - Check port availability: `lsof -i :<port>`

---

### WebRTC Streaming Issues

**Problem**: Live device screen not loading or black screen

**Causes**:
- STUN/TURN servers not configured or unreachable
- Network firewall blocking WebRTC ports
- Browser WebRTC compatibility issues

**Resolution**:
1. Configure STUN/TURN servers in `hub.config.json`:
   ```json
   {
     "webrtc": {
       "iceServers": [
         { "urls": "stun:stun.l.google.com:19302" },
         {
           "urls": "turn:your-turn-server.com:3478",
           "username": "your-username",
           "credential": "your-password"
         }
       ]
     }
   }
   ```
   Or via environment variable:
   ```bash
   export WEBRTC_ICE_SERVERS='[{"urls":"stun:stun.l.google.com:19302"}]'
   ```

2. Check firewall allows WebRTC traffic (UDP ports, typically 10000-11000)

3. Test in different browsers (Chrome, Firefox, Safari)

4. Check browser console for WebRTC errors

5. For hosted environments, TURN servers are required for NAT traversal

---

### Storage & Log Upload Issues

**Problem**: Session logs/videos not saved or "Failed to upload session logs"

**Causes**:
- Storage configuration invalid or missing
- Storage bucket/path not accessible
- Insufficient permissions

**Resolution**:

1. **For Local Storage** (default):
   ```json
   {
     "storage": {
       "type": "local",
       "local": {
         "basePath": "/absolute/path/to/logs"  // Must be absolute path
       }
     }
   }
   ```
   - Ensure directory exists and has write permissions
   - Check disk space availability

2. **For Cloud Storage** (S3/Azure/Cloudflare R2):
   - Verify credentials are correct
   - Test bucket access with provider's CLI
   - Ensure bucket exists and has appropriate permissions
   - Check `hub.config.json` has complete storage configuration

3. **Environment variable alternative**:
   ```bash
   export SESSION_LOGS_PATH=/path/to/logs
   ```

---

### Authentication Failures

**Problem**: "Authentication failed - no authorization token provided" or "Invalid token"

**Common Issues**:

1. **Missing Authorization Header**
   - Include header: `Authorization: Bearer <token>` or `Authorization: Basic <base64-credentials>`
   - Or use cookie-based auth after Google OAuth login

2. **Token Expired**
   - JWT tokens expire after 24 hours by default
   - Generate new token via login endpoint
   - Check token expiry: Decode JWT and check `exp` field

3. **Invalid Access Key**
   - Verify access key matches registered key in system
   - Create new access key if lost: POST `/api/users/{userId}/access-key`

4. **User Not Active**
   - Contact admin to reactivate user account
   - Check user status: GET `/api/users/{userId}`

---

### Database Connection Issues

**Problem**: "Unable to connect to the database" on Hub startup

**Causes**:
- PostgreSQL not running
- Wrong database credentials
- Database doesn't exist
- Network connectivity issues

**Resolution**:
1. Verify PostgreSQL is running:
   ```bash
   # Check status
   pg_isready -h localhost -p 5432

   # Or check process
   ps aux | grep postgres
   ```

2. Test connection manually:
   ```bash
   psql -h <host> -U <user> -d <database>
   ```

3. Check environment variables or config:
   ```bash
   # Option 1: Individual variables
   export DB_HOST=localhost
   export DB_USER=postgres
   export DB_PASSWORD=password
   export DB_NAME=device_farm

   # Option 2: Connection URL
   export DATABASE_URL=postgresql://user:password@host:5432/database
   ```

4. Create database if missing:
   ```bash
   createdb device_farm
   ```

5. Run migrations:
   ```bash
   cd packages/hub
   npm run migrate
   ```

---

### Port Conflicts & Exhaustion

**Problem**: "Failed to start Appium server" or "Port already in use"

**Causes**:
- Another Appium instance using the same port
- Previous session didn't cleanup properly
- Port range exhausted (too many concurrent sessions)

**Resolution**:
1. Check what's using the port:
   ```bash
   lsof -i :<port>
   # Kill process if needed
   kill -9 <pid>
   ```

2. For MJPEG/WebRTC port exhaustion:
   - Default range: 10000-11000 (only 1000 ports)
   - Increase range in `node.config.json`:
     ```json
     {
       "capabilityPortRange": {
         "min": 10000,
         "max": 20000
       }
     }
     ```

3. Restart node to force port cleanup

4. Check for zombie Appium processes:
   ```bash
   ps aux | grep appium
   killall -9 node  # If needed (use with caution)
   ```

---

### SSL/HTTPS Configuration Issues

**Problem**: "SSL configuration error: Could not read certificate files"

**Causes**:
- Certificate files not found at specified paths
- Incorrect file permissions
- Invalid or expired certificates

**Resolution**:
1. Verify SSL configuration in `hub.config.json`:
   ```json
   {
     "ssl": {
       "enabled": true,
       "cert": "/absolute/path/to/cert.pem",
       "key": "/absolute/path/to/key.pem",
       "ca": "/absolute/path/to/ca.pem"  // Optional
     }
   }
   ```

2. Or via environment variables:
   ```bash
   export SSL_ENABLED=true
   export SSL_CERT=/path/to/cert.pem
   export SSL_KEY=/path/to/key.pem
   ```

3. Verify certificate files exist and are readable:
   ```bash
   ls -la /path/to/cert.pem /path/to/key.pem
   ```

4. Test certificate validity:
   ```bash
   openssl x509 -in cert.pem -text -noout
   openssl x509 -in cert.pem -noout -dates  # Check expiry
   ```

5. Ensure paths are absolute, not relative

---

### High Resource Usage

**Problem**: High CPU/memory usage on Hub or Node

**Common Causes**:
- Too many concurrent sessions
- Video recording overhead
- Device logs capturing overhead (Android ADB logcat)
- Memory leaks in long-running sessions
- Insufficient hardware resources

**Mitigation**:
1. Limit concurrent sessions per node
2. Disable video recording for automation sessions if not needed:
   ```json
   {
     "df:recordVideo": false
   }
   ```
3. Disable device logs capture for automation sessions if not needed:
   ```json
   {
     "df:deviceLogs": false
   }
   ```
   **Note**: Device logs (Android ADB logcat) continuously capture device output during sessions, which consumes CPU and I/O resources. For automation sessions, device logs are only captured when explicitly enabled with `df:deviceLogs: true`. Manual sessions do not capture device logs by default.

4. Monitor and restart nodes periodically
5. Increase hardware resources (CPU, RAM)
6. Scale horizontally by adding more nodes
7. Reduce video quality settings for manual sessions

---

### Getting Help

If you encounter issues not covered here:

1. **Check logs**:
   - Hub logs: Check application output or configured log directory
   - Node logs: Check application output
   - Appium logs: Available via session API: GET `/api/sessions/{sessionId}/logs/appium`
   - Device logs (Android): GET `/api/sessions/{sessionId}/logs/device`

2. **Enable debug logging**:
   ```bash
   # Hub
   export LOG_LEVEL=debug

   # Node
   export LOG_LEVEL=debug
   ```

3. **Check system status**:
   - Nodes: GET `/api/nodes`
   - Devices: GET `/api/devices`
   - Sessions: GET `/api/sessions`

4. **Report issues**:
   - Include logs with error messages
   - Provide configuration (sanitize sensitive data)
   - Describe steps to reproduce
   - Note: Device Farm version, OS, Node/Java versions

## 📺 Streaming Settings

The **Streaming Settings** section allows you to customize the streaming performance and quality of the iOS simulator, tvOS devices, and real device screens within the Appium Device-farm dashboard. This is particularly useful when accessing devices from remote machines for real-time control, ensuring the best balance between performance and visual clarity.

### 🔧 Settings Descriptions:

- **Framerate (1 - 60):**  
  Sets the number of frames per second (FPS) for the streamed display.

  - Higher values (e.g., 60) provide smoother motion but increase resource usage.
  - Lower values (e.g., 10) can improve stability under low bandwidth conditions.

- **Video Quality (1 - 100):**  
  Controls the visual quality of the stream.

  - `0` offers the lowest quality to save bandwidth.
  - `100` delivers the highest clarity, useful for detailed UI inspections.
  - Adjust this setting to balance visual clarity with network constraints.

- **Scaling Factor (1 - 100):**  
  Adjusts the size of the streamed display.

  - `100` represents the original size.
  - Lower values shrink the display, helpful when viewing multiple devices simultaneously.

- **Save Settings:**  
  Click this button to apply the changes. All adjustments will take effect immediately for the ongoing session.

These settings are designed to offer flexibility, allowing testers to tailor the streaming experience based on their hardware, network conditions, and testing needs.

## Notes

### Session Idle Timeout (Manual Sessions Only)

**This feature applies only to manual/interactive sessions accessed through the dashboard UI, not automation sessions.**

By default, if there is no activity on a **manual session** for more than **300 seconds (5 minutes)**, the device allocated to the respective session will be unblocked and made available for new session requests. This prevents devices from being locked indefinitely when users walk away from interactive sessions.

You can customize this timeout by configuring the `sessionIdleDetection` settings in your `node.config.json`:

```json
{
  "sessionIdleDetection": {
    "enabled": true,
    "idleTimeoutMs": 100000,      // 100 seconds
    "checkIntervalMs": 30000      // Check every 30 seconds
  }
}
```

Or via environment variables:

```bash
SESSION_IDLE_DETECTION_ENABLED=true
SESSION_IDLE_TIMEOUT_MS=100000              # 100 seconds
SESSION_IDLE_CHECK_INTERVAL_MS=30000        # Check every 30 seconds
```

**Configuration Options:**

- `enabled` - Enable or disable idle session detection (default: `true`)
- `idleTimeoutMs` - Maximum idle time before session termination in milliseconds (default: `300000` = 5 minutes)
- `checkIntervalMs` - How often to check for idle sessions in milliseconds (default: `60000` = 60 seconds)

**What Sessions Are Affected?**

- ✅ **Manual Sessions**: Dashboard UI live device control sessions (identified by `df:sessionType: 'manual'` or `appium:newCommandTimeout: '0'`)
- ❌ **Automation Sessions**: Regular WebDriver/Appium test scripts are NOT affected by this timeout. They manage their own lifecycle through Appium's `newCommandTimeout` capability.
