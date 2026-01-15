---
title: Node Setup
---

## Installation

```
npm install -g farm-runner
```

<div style="text-align: center">
  <img src="https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/remote.jpg" class="center"/>
</div>

## Start a Node

To connect a machine with devices to the hub, you need the `accessKey` and `token` from the hub. These credentials are required for the runner to connect to the orchestrator.

## Getting Access Key and Token

1. Log in to the hub dashboard at `http://localhost:3000` using the admin credentials
2. Click on your profile name in the header
3. Navigate to **Profile → API Token**
4. The access key will be displayed above the API tokens table
5. Click "Generate Token" to create a new API token
6. Copy both the access key and the generated token

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="assets/images/authentication/api-token.png" alt="API Token Management" style="width: 70%; max-width: 1400px;">
</div>

## Configuring the Node

1.  Create `node.config.json`:
    ```json
    {
      "hubUrl": "http://localhost:3000",
      "accessKey": "your-access-key-from-profile",
      "token": "your-generated-token",
      "name": "My-Mac-Mini",
      "platform": "both"
    }
    ```
2.  Run the node:
    ```bash
    farm-runner -c node.config.json
    ```
