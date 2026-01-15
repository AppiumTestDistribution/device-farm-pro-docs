---
title: Manual WDA Setup Overview
---

If you prefer to manually build, re-sign, and upload the WebDriverAgent, follow the steps below. This approach gives you full control over the signing process.

## Prerequisites

1. A Mac machine with Xcode and Xcode command line tools installed
2. A valid Apple account (with or without a developer program subscription)
3. The iOS Resigner app for re-signing the WDA IPA, which you can download from [iOS Resigner GitHub](https://github.com/DanTheMan827/ios-app-signer/releases)

## Overview

The manual WDA setup process involves the following steps:

1. **Download WebDriverAgent** - Download the pre-built WDA IPA file
2. **Create Provisioning Profile** - Set up your Apple account and create a provisioning profile in Xcode
3. **Resign WDA IPA** - Use the iOS Resigner app to re-sign the WDA with your provisioning profile
4. **Prepare for tvOS** (optional) - Create a separate resigned IPA for tvOS devices if needed
5. **Upload WDA** - Upload the resigned IPA to the device farm
6. **Build from Source** (optional) - Alternative method to build WDA directly from source

Follow the individual guides for each step in the navigation menu.
