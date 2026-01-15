---
title: Preparing WDA for tvOS Devices
---

For tvOS devices (Apple TV), you need to create a separate resigned IPA file. The process is similar to iOS, but with some important differences:

## Key Requirements for tvOS WDA:

1. **Bundle ID**: tvOS apps require a different bundle identifier format. Make sure your provisioning profile includes tvOS support.

2. **Provisioning Profile**: Your provisioning profile must explicitly support tvOS devices. When creating the provisioning profile in Xcode, ensure you select tvOS as a supported platform.

3. **File Naming**: The resigned IPA must be named exactly `wda-resign_tvos.ipa` for the device farm to recognize it as a tvOS WebDriverAgent.

## Steps for tvOS WDA Preparation:

1. Follow the same provisioning profile creation steps as outlined in the Creating Provisioning Profile section, but ensure tvOS is selected as a supported platform.

2. Use the same WDA.ipa file from the [Appium Device Farm GitHub repository](https://github.com/AppiumTestDistribution/appium-device-farm/raw/main/WDA.ipa).

3. In the iOS Resigner app:
   - Select the WDA.ipa file as input
   - Choose your Apple account from the Signing Certificate dropdown
   - Select the tvOS-compatible provisioning profile
   - **Important**: Save the output file as `wda-resign_tvos.ipa`

4. Upload the `wda-resign_tvos.ipa` file to the device farm using the Apps section in the dashboard.

The device farm will automatically detect and use the appropriate WDA file based on the connected device type:
- iOS devices (iPhone, iPad) → `wda-resign.ipa`
- tvOS devices (Apple TV) → `wda-resign_tvos.ipa`
