---
title: Automation Authentication
---

## WebDriver Capabilities

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
