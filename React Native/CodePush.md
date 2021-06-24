CodePush is a powerfull tool for React Native developers created by Microsoft to deploy and update apps directly on user's devices.
Anyway, you should still be careful when using it. There are some limitations.

**NOTE: This guidline is only applicable on React Native projects created with `react-native init` command.**

### Limitations

- You should only deploy or update JS, HTML, CSS files and images, otherwise it will not work.
- Again as the first bulletpoint mentions, you can't update the binary of the app (read native parts of the app).
- If you missuse the CodePush tool it's easy possbile that Apple or Google will warn you and maybe ban your app (twice a month should be fine).
- For Android devices, CodePush will only work on TLS 1.2 compatible devices.
- CodePush API usage has a rate limitation of 70 requests per second applied.


### Advantages

Now if you are aware of the limitations, let's mention some of the advatages of using CodePush inside your app.

- Quick deployment of bug fixes and smaller features
- Static text and images update
- Instantly updates (the app updates in under a minute)
- Great CLI to work with
- Web app dashboard to work with
- Easy app versions overview
- Multiple enviremonts (Alpha, Beta, Production, Staging and more)
- Separate Android and iOS deployment, monitoring etc.
- Statistics report dashboard *
- Crahslitics report dashboard *
- Deployment of the app in one place (no need to switch between Google Play Console and App Store Connect)

***Needs additional implementation inside the app in addition to the CodePush sdk.**

### App Center CLI

- First step is to install the App Center CLI which is the official name for all the SDK's that are provided including CodePush. 
   Install it global.

```npm
npm install -g appcenter-cli
```

### Implementation of AppCenter and CodePush SDK's

- The next step is to, as they say in the official [https://docs.microsoft.com/en-us/appcenter/distribution/codepush/](documentation) *CodePush-ify* the app. To do so, follow the next steps carefully and implement the SDK's (both AppCenter and CodePush) for Android and iOS. Also, assuming that all the projects you will upgrade with CodePush have a newer version of React Native, the next steps are for React Native version 0.60 and above, however if you'r implementin the SDK on a older project with React Native version lower than 0.60 please refer to the official [https://docs.microsoft.com/en-us/appcenter/distribution/codepush/rn-get-started](documentation).

1. First you need to create your app in the App Center Portal. Go to [https://appcenter.ms/](App Center) create an account if you don't have one.
After successfull login click on the button `Add new app` and fill out all reqired fields. Select the OS for which you are creating the app (Android/iOS) and select React Native as the developmetn platform. After creating the app navigate to the Settings section and click on the 3 dots in the upper right corner, then click `Copy app secret`.

2. Then install npm packages inside your project.

```npm
npm install --save appcenter react-native-code-push
```

### iOS

1. Install necessary CocoaPods.

```pod
cd ios && pod install && cd ..
```

2. Create a new file with the name AppCenter-Config.plist with the following content and replace {APP_SECRET_VALUE} with your app secret value. Don't forget to add this file to the Xcode project (right-click the app in Xcode and click Add files to ...).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
    <key>AppSecret</key>
    <string>{APP_SECRET_VALUE}</string>
    </dict>
</plist>
```

3. Add import statements to `AppDelegate.m` (above `#if DEBUG` or `#ifdef FB_SONARKIT_ENABLED`)

```objectice-c
#import <AppCenterReactNative.h>
#import <CodePush/CodePush.h>
```
4. Also in `AppDelegate.m` find the following line of code (which sets the source URL for bridge for production releases), it should be somewhere at the end of the file (~ line 60 on a fresh project)

```objectice-c
return [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
```

and replace it with

```objectice-c
return [CodePush bundleURL];
```

This change configures your app to always load the most recent version of your app's JS bundle.

***The bundleURL method assumes your app's JS bundle is named main.jsbundle.***

To still debug your app with Chrome Dev Tools, live reload etc. make sure your sourceURLForBridge looks like this:

```objective-c
- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
{
  #if DEBUG
    return [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
  #else
    return [CodePush bundleURL];
  #endif
}
```







