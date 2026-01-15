---
title: Capabilities
---

| Capability Name              | Description                                                                                                                                                                                                                                                        |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| df:build                     | Allocate a build name in dashboard. Default value is `null`.                                                                                                                                                                                                       |
| df:recordVideo               | Enable video recording for the session. Default value is `false`.                                                                                                                                                                                                  |
| df:videoTimeLimit            | Maximum duration of the video recording in seconds. Default value is 1800 seconds (3 minutes).                                                                                                                                                                     |
| df:iPhoneOnly                | Allocate only iPhone simulators for execution when to true. Default value is `false`.                                                                                                                                                                              |
| df:iPadOnly                  | Allocate only iPad simulators for execution when to true. Default value is `false`.                                                                                                                                                                                |
| df:deviceAvailabilityTimeout | When create session requests are more than available connected devices, plugin waits for a certain interval for device availability before it timeout. Default value is `180000` milliseconds.                                                                     |
| df:deviceRetryInterval       | When create session requests are more than available connected devices, plugin polls for device availability in certain intervals. Default value is `10000` milliseconds.                                                                                          |
| df:udids                     | Comma separated list of device udid's to execute tests only on specific devices `df:udids: device1UDID,device2UDID`                                                                                                                                                |
| appium:platformName          | Requests asession for the provided platform name. Valid options are `iOS`, `tvOS`, or `Android`, ex: `'appium:platformName': tvOS`                                                                                                                                 |
| appium:platformVersion       | This capability is used to filter devices/simulators based on SDK. Only devices/simulators that are an exact match with the platformVerson would be considered for test run. `appium:platformVersion` is optional argument. ex: `'appium:platformVersion': 16.1.1` |
| df:minSDK                    | This capability is used to filter devices/simulators based on SDK. Devices/Simulators with SDK greater then or equal to minSDK would only be considered for test run. `df:minSDK` is optional argument. ex: `'appium:minSDK': 15`                                  |
| df:maxSDK                    | This capability is used to filter devices/simulators based on SDK. Devices/Simulators with SDK less then or equal to maxSDK would only be considered for test run. `df:maxSDK` is optional argument. ex: `'appium:maxSDK': 15`                                     |
| df:filterByHost              | This capability is used to filter devices/simulators based on node IP. This will only consider devices from specific node. `df:options` is optional argument. ex: `'filterByHost': '192.168.0.226',`                                                               |
| df:options                   | Set all device farm related capabilities as object. `df:options` is optional argument. ex: `'df:options': { filterByHost: '192.168.0.226', recordVideo: true },`                                                                                                   |
| df:deviceLogs                | Enable device logs capture (Android ADB logcat). For automation sessions only. Manual sessions do not capture device logs. Default value is `false`. ex: `'df:options': { deviceLogs: true },`                                                                     |
| df:tags                      | Will consider the devices only with the tagged specified. `df:tags` is optional argument. ex: `'df:tags': ['team1','AndroidGroup'],`. Default value is empty array.                                                                                                |
| df:androidCleanUpApps        | List of Android packages to uninstall before session starts. Can be used in `df:options`. Overrides server configuration if provided. ex: `'df:options': { androidCleanUpApps: ['com.example.app'] }`                                                              |
| df:iosCleanUpApps            | List of iOS bundle IDs to uninstall before session starts. Can be used in `df:options`. Overrides server configuration if provided. ex: `'df:options': { iosCleanUpApps: ['com.example.app'] }`                                                                    |

### Cleanup Apps Precedence
If `androidCleanUpApps` or `iosCleanUpApps` are provided in `df:options` (session capabilities), they will take precedence over the server configuration.
- To use server configuration: Do not provide these keys in `df:options`.
- To override server configuration: Provide the list of apps in `df:options`.
- To disable cleanup for a specific session (when server config has cleanup apps): Provide an empty array `[]` in `df:options`.

---

## Client Usage (Capabilities)

=== "Overview"

    To use Device Farm, add the `df:options` block to your Appium capabilities.

    ### Essential Capabilities

    | Capability    | Type      | Description                              |
    | :------------ | :-------- | :--------------------------------------- |
    | `accessKey`   | `string`  | **Required.** Your authentication key.   |
    | `token`       | `string`  | **Required.** Your authentication token. |
    | `runsOn`      | `string`  | **Required.** `real` or `simulated`.     |
    | `buildName`   | `string`  | Group tests under a specific build name. |
    | `recordVideo` | `boolean` | Enable video recording of the session.   |

    ### Example: WebdriverIO

    ```javascript
    export const config = {
      hostname: 'localhost',
      port: 3000,
      path: '/wd/hub',
      capabilities: [
        {
          platformName: 'iOS',
          'appium:automationName': 'XCUITest',
          'appium:deviceName': 'iPhone 15',
          'appium:app': '/path/to/app.ipa',

          // Device Farm Specific Options
          'df:options': {
            accessKey: 'admin',
            token: 'password',
            runsOn: 'real',
            buildName: 'Nightly Regression',
            recordVideo: true,
            iosCleanUpApps: ['com.mycompany.app'],
          },
        },
      ],
    };
    ```

=== "All Available Options"

    All Device Farm specific capabilities must be nested under the `df:options` object in your test capabilities.

    #### 🔐 Authentication & Selection (Required)

    | Field       | Type   | Required | Description                             |
    | :---------- | :----- | :------- | :-------------------------------------- |
    | `accessKey` | string | Yes      | Your Device Farm access key.            |
    | `token`     | string | Yes      | Your Device Farm authentication token.  |
    | `runsOn`    | string | Yes      | Device type: `'real'` or `'simulated'`. |

    #### 🛠 Build & Session Management

    | Field       | Type   | Default | Description                                       |
    | :---------- | :----- | :------ | :------------------------------------------------ |
    | `buildName` | string | UUID    | Name of the build (groups related test sessions). |
    | `name`      | string | -       | Custom session name for easier identification.    |

    #### 📦 App Management

    | Field                | Type     | Default | Description                                           |
    | :------------------- | :------- | :------ | :---------------------------------------------------- |
    | `androidCleanUpApps` | string[] | `[]`    | Android app package names to uninstall after session. |
    | `iosCleanUpApps`     | string[] | `[]`    | iOS bundle IDs to uninstall after session.            |
    | `forceDownloadApp`   | boolean  | `false` | Force re-download of app from hub even if cached.     |

    #### 🎥 Video & Screenshots

    | Field                 | Type    | Default | Description                            |
    | :-------------------- | :------ | :------ | :------------------------------------- |
    | `recordVideo`         | boolean | `false` | Enable video recording of the session. |
    | `videoResolution`     | string  | -       | Video resolution (e.g., `'1280x720'`). |
    | `videoTimeLimit`      | number  | -       | Maximum video duration in seconds.     |
    | `screenshotOnFailure` | boolean | `false` | Capture screenshot on command failure. |
    | `screenshotOnAll`     | boolean | `false` | Capture screenshot on every command.   |

    #### 🔍 Device Filtering (Advanced)

    | Field          | Type     | Default | Description                      |
    | :------------- | :------- | :------ | :------------------------------- |
    | `udids`        | string[] | -       | Specific device UDIDs to use.    |
    | `tags`         | string[] | -       | Filter devices by custom tags.   |
    | `filterByHost` | string   | -       | Filter devices by node hostname. |
    | `iPhoneOnly`   | boolean  | `false` | Use only iPhone devices (iOS).   |
    | `iPadOnly`     | boolean  | `false` | Use only iPad devices (iOS).     |
    | `minSDK`       | number   | -       | Minimum SDK/OS version.          |
    | `maxSDK`       | number   | -       | Maximum SDK/OS version.          |

    #### 🍎 iOS Specific & Logs

    | Field         | Type    | Default | Description                                                                       |
    | :------------ | :------ | :------ | :-------------------------------------------------------------------------------- |
    | `wdaId`       | string  | -       | Specific WebDriverAgent IPA ID to use (iOS real devices).                        |
    | `deviceLogs`  | boolean | `false` | Enable device logs capture (Android ADB logcat). For automation sessions only.   |
