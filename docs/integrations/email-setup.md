---
title: Email Setup
---

Send notifications to your team via SMTP email.

## Step 1: Gather SMTP Credentials

You'll need:

- SMTP host (e.g., `smtp.gmail.com`)
- SMTP port (e.g., `587` for TLS)
- Username and password (if authentication required)
- From address

!!! note "Gmail Users"
    For Gmail, you'll need to create an [App Password](https://support.google.com/accounts/answer/185833) instead of using your regular password.

## Step 2: Add Integration in Device Farm

1. Navigate to **Settings > Integrations**
2. Click **Add Integration**
3. Select **Email**
4. Fill in the SMTP configuration:
    - **SMTP Host**: Your mail server (e.g., `smtp.gmail.com`)
    - **SMTP Port**: Usually `587` for TLS or `465` for SSL
    - **SMTP User**: Your email username (optional)
    - **SMTP Password**: Your email password or app password (optional)
    - **From Address**: The sender email address
    - **From Name**: Display name for the sender (optional)
    - **Use TLS**: Enable for secure connections
5. Click **Save**

## Step 3: Create Notification Rules

1. Expand your integration
2. Click **Add Rule**
3. Select the event type
4. Enter recipient email addresses (comma-separated)
5. For scheduled reports, configure the schedule
6. Click **Save**

## Testing Your Integration

### Test Connection

Click the **Send** icon (paper airplane) on any integration to send a test email. This verifies your SMTP configuration is correct.

### Send Report Now

Click the **Chart** icon on any integration to immediately send a report with the last 30 days of data.

## Common SMTP Configurations

### Gmail

| Setting | Value |
|---------|-------|
| SMTP Host | `smtp.gmail.com` |
| SMTP Port | `587` |
| Use TLS | Yes |
| Username | Your Gmail address |
| Password | App Password (not regular password) |

### Outlook/Office 365

| Setting | Value |
|---------|-------|
| SMTP Host | `smtp.office365.com` |
| SMTP Port | `587` |
| Use TLS | Yes |
| Username | Your Outlook address |
| Password | Your password |

### Amazon SES

| Setting | Value |
|---------|-------|
| SMTP Host | `email-smtp.<region>.amazonaws.com` |
| SMTP Port | `587` |
| Use TLS | Yes |
| Username | SES SMTP credentials |
| Password | SES SMTP credentials |

## Troubleshooting

### Notifications not arriving

1. Test the connection using the test button
2. Verify SMTP credentials are correct
3. Check if your email provider requires app passwords
4. Ensure the from address is allowed by your SMTP server
5. Check spam/junk folders
6. View the notification logs for error messages
