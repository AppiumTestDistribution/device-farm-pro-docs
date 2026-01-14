# Authentication and Device Management

This section covers the authentication and device management features of Appium Device Farm. These features help secure your device farm and provide better control over device access and management. 

- All endpoints will be secured
- Access requires valid credentials
- Device access is controlled through user roles and team assignments

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/login-screen.png" alt="Login Screen" style="width: 80%; max-width: 1200px;">
</div>

## Default Credentials

Before installing the device farm plugin, you can set default admin credentials using environment variables:

```bash
export DEFAULT_ADMIN_USERNAME=your_admin_username
export DEFAULT_ADMIN_PASSWORD=your_admin_password
```

If not set, the default credentials are:
- Username: `admin@device-farm.org`
- Password: `Admin@123`

!!! warning "Security Note"
    It's highly recommended to change the default credentials after your first login.

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/reset-password.png" alt="Reset Password" style="width: 70%; max-width: 1400px;">
</div>

## User Roles

The device farm supports two user roles:

### Admin
- Create and manage users
- Create and manage teams
- Manage all devices
- Access all features
- Generate and manage API tokens

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/users.png" alt="User Management" style="width: 0%; max-width: 1400px;">
</div>
<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/teams.png" alt="Team Management" style="width: 70%; max-width: 1400px;">
</div>

### User
- Login to the system
- Access devices based on team permissions
- Generate personal API tokens
- Change their password

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/menu-items.png" alt="Menu Items" style="width: 70%; max-width: 900px;">
</div>

## Node Authentication

When connecting a node to an authenticated hub, you need to provide authentication credentials:
```

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/servers.png" alt="Servers Management" style="width: 90%; max-width: 1400px;">
</div>

### Getting Access Key and Token

1. Log in to the device farm dashboard
2. Click on your profile name in the header
3. Navigate to the "API Tokens" tab
4. Click "Generate Token"
5. Enter a token name and optional expiration date
6. Click "Create"

The access key will be displayed above the API tokens table. Use both the access key and token when registering nodes.

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/api-token.png" alt="API Token Management" style="width: 70%; max-width: 1400px;">
</div>
<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/api-token-generate.png" alt="Generate API Token" style="width: 70%; max-width: 1400px;">
</div>

## Automation Authentication

For automated testing, include authentication credentials in your WebDriver capabilities:

```javascript
const capabilities = {
  platformName: 'Android',
  'appium:uiautomator2ServerInstallTimeout': '50000',
  'appium:automationName': 'UIAutomator2',
  'appium:newCommandTimeout': 30000,
  'appium:app': 'wdio-1765871007357-fd191b8b.apk',
  'df:options': {
    accessKey: 'access_key',
    token: 'token',
    screenshotOnFailure: false,
    screenshotOnAll: false,
    recordVideo: true,
    buildName,
  },
};
```

## Device Management

### Device Administration

Admin users can:
- View all devices in the hub network
- Edit device names and tags
- Flag devices for maintenance
- Exclude devices from the pool
- Assign devices to teams

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/device-management.png" alt="Device Management" style="width: 70%; max-width: 1400px;">
</div>
<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/device-management-2.png" alt="Device Management Details" style="width: 70%; max-width: 1400px;">
</div>

### Device Persistence

Device information is persisted in the database, including:
- Device names
- Tags
- Status
- Team assignments
- Maintenance flags

### Device Access Control

- Devices can be assigned to specific teams
- Users can only access devices assigned to their team
- Admins can override team assignments
- Maintenance flags prevent device access during maintenance

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/manage-team-device.png" alt="Manage Team Devices" style="width: 70%; max-width: 1400px;">
</div>
<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/manage-team-user-1.png" alt="Manage Team Users 1" style="width: 70%; max-width: 1400px;">
</div>

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/appium-device-farm/main/documentation/docs/assets/images/authentication/manage-team-user-2.png" alt="Manage Team Users 2" style="width: 80%; max-width: 1000px;">
</div>

## Best Practices

1. **Security**
2. Change default credentials immediately
3. Use strong passwords
4. Regularly rotate API tokens
5. Set appropriate token expiration dates
6. **Device Management**
7. Use meaningful device names
8. Tag devices appropriately
9. Flag devices for maintenance when needed
10. Regularly review team assignments
11. **Node Configuration**
12. Store node credentials securely
13. Use environment variables for sensitive data
14. Monitor node connection status
15. Implement proper error handling for authentication failures