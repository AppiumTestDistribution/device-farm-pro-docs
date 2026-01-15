---
title: Building WebDriverAgent from Source
---

The plugin has an in-build script that can help you build the WDA IPA to test on real devices. 

Run the command below to build the WebDriverAgent from source.
```javascript
appium plugin run device-farm prepare-wda
```
The above script will pick WebDriverAgent project installed globally from xcuitest-driver. 

If you want to provide the custom path of WebDriverAgent project then set _WDA_PROJECT_PATH_ or Provide a provision profile then run the command.
```javascript
appium plugin run device-farm prepare-wda --wda-project-path=<path-to-WDA-project> --mobile-provisioning-file=<path-to-provision-profile>
```

You should have all the provision certificates installed on your machine before building the WebDriverAgent from source. For Xcode versions 15 or below, ensure they are located in:
```
~/Library/MobileDevice/Provisioning\ Profiles
```

For Xcode versions 16 and above, place them in:
```
~/Library/Developer/Xcode/UserData/Provisioning\ Profiles
```
