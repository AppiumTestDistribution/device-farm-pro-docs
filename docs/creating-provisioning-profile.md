---
title: Creating Provisioning Profile
---

Open Xcode and sign in with your Apple ID.

![xcode](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step1-xcode-settings.png)

click the `+` icon to add a new account

![Xcode create account](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step2-add-account.png)

Select `Apple ID` from the options and click `Continue`

![Xcode sign in](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step3-add-account-apple-id.png)

Log in with your existing Apple ID or click `Create Apple ID` to create a new Apple account.

![Xcode account](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step4-login-email.png)

Once logged in successfully, you should see your account added to the account list.

![Xcode account list](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step5-after-login.png)

Now, create the provisioning profile for re-signing the WDA. Open `Xcode > File > New > Project`.

![Xcode create project](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step6-create-project.png)

Select any app type from the list. Choosing `App` is recommended for simplicity.

![Xcode select app](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step7-selcect-app-type.png)

Enter a Project Name and a valid `Organization Identifier`. It can be any value, but ensure it is unique. Select the Apple account you logged in with from the Team dropdown. After entering the details, click `Next` and choose a folder to create the project.

![Xcode org id](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step8-enter-project-details.png)

Once the project is created, click on the `Project Name` in the left pane and select the `Signing & Capabilities` tab. You should see `Xcode Managed Profile` listed under Provisioning Profile.

![xcode signing](https://raw.githubusercontent.com/AppiumTestDistribution/device-farm-pro-docs/refs/heads/main/docs/assets/images/wda-signing/step9-post-project-creation.png)

You can also check if the provisioning profile is created in the `~/Library/MobileDevice/Provisioning Profiles directory`. Once the provisioning profile is present, you have successfully created it and can now use it to sign the WDA file.
