---
title: User Authentication
---

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
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/authentication/reset-password.png" alt="Reset Password" style="width: 70%; max-width: 1400px;">
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
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/authentication/users.png" alt="User Management" style="width: 0%; max-width: 1400px;">
</div>
<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/authentication/teams.png" alt="Team Management" style="width: 70%; max-width: 1400px;">
</div>

### User
- Login to the system
- Access devices based on team permissions
- Generate personal API tokens
- Change their password

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img src="https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/authentication/menu-items.png" alt="Menu Items" style="width: 70%; max-width: 900px;">
</div>
