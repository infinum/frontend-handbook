## Android

### Signing
To distribute your app on Google Play, App signing is required. Play App Signing uses two keys which are the *app signing key* and the *upload key*. The difference between the two is explained [here](https://developer.android.com/studio/publish/app-signing#keys_keystores). The *upload key* is needed for developers to upload the app to Google Play while the *app signing key* is used to sign the final app that users get from Play Store.  
The *app signing key* is created and managed by Google unless you choose not to allow it, but that's not recommended. If you choose to manage the app signing by yourself look [here](https://developer.android.com/studio/publish/app-signing#opt-out). Remember, if you lose the *app signing key*, you will not be able to update your app on Play Store. On the other hand, the *upload key* can be reset by [contacting support](https://support.google.com/googleplay/android-developer/answer/7384423#reset).  
This chapter will assume opting in to Play App Signing.

### Generating an upload key
To create your *upload key*, you can run this command in the terminal. *(do it in* `android/app`*)*  
  
```
keytool -genkey -v -keystore release.keystore -alias my-key-alias -storetype PKCS12 -keyalg RSA -keysize 2048 -validity 10000
```
  
You will be prompted for a password and some more info like name, organization, city...  
The result will be a file `release.keystore`.  

### Setting up Gradle variables
Edit the file `android/app/build.gradle` in your project folder to add the release signing config and also change the release build type to use the newly added config.

```
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        debug { ... }
        release {
            storeFile file('release.keystore')
            keyAlias MYAPP_KEY_ALIAS
            storePassword MYAPP_KEY_PASSWORD
            keyPassword MYAPP_KEY_PASSWORD
        }
    }
    buildTypes {
        debug { ... }
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
```

Then to set values to those variables (`MYAPP_KEY_ALIAS` and `MYAPP_KEY_PASSWORD`), create *(or edit)* the `~/.gradle/gradle.properties` file and add values you used in creating the keystore. You can do the same thing on a project level by creating or editing `android/gradle.properties` but make sure to put this file into the `.gitignore`. The first approach is more secure and recommended.

```
MYAPP_KEY_ALIAS=my-key-alias
MYAPP_KEY_PASSWORD=*****
```


### Building

#### Generating the APK

If you want to generate the apk file for testing purposes or uploading to Labs, you can do it by running this command in `android` directory: 

```
./gradlew assembleRelease
```

The APK will be located in `android/app/build/outputs/apk/release/app-release.apk` and will be signed with your `upload key`.  
  
If your app has multiple flavours like in this example:

```
productFlavors {
    staging { ... }
    production { ... }
}
```

you would use `assembleStagingRelease` or `assembleProductionRelease` commands.  

#### Generating the bundle

If you want to upload the app to Google Play, it's recommended you generate an AAB file.  
Like with generating the APK, you use the same command but replace `assemble` with `bundle`, like this:  

```
./gradlew bundleRelease
```

The AAB will be located in `android/app/build/outputs/bundle/release/app-release.aab` and you can upload it to Google Play.  
  
If your project for some reason requires uploading the APK file to Google Play, you can upload the one signed with the *upload key* but you lose the benefits of app spliting. Read more about this [here](https://developer.android.com/guide/app-bundle).

### Releasing

You can follow the steps for releasing the app [here](https://infinum.com/handbook/books/android/deployment/release-practices).  
  
  

## iOS

### Signing
To distribute your app on the App Store, a few steps are required. You will need Xcode and an Apple developer account.  
Before you start, make sure to read [this chapter](https://infinum.com/handbook/books/ios/project-flow/certificates-provisioning-profiles-and-apns) about certificates and provisioning profiles.  
After you get your certificate and profile, you're set. App signing is done automatically.  

### Building

To build the app, in Xcode first go to Product > Scheme > Edit scheme, and change Build configuration to Release. Now you can build and run the app without Metro server.  

### Releasing

#### Archive

To release the app, you first need to create an archive. You do that in Xcode by going to Product > Archive. After the archive builds successfully, a new window will be opened with a list of all archives. The top one will be selected and it's the one you've just built.  

#### Distribution

With the archive selected, click on the Distribute App button on the right, and after verifying the info that appears, click the Upload button. This will launch a wizard for exporting the app. The default settings should be fine for React Native apps.

#### Distribution methods

In the wizard for exporting the app, there is one screen where you can choose the distribution method. There are 2 methods that you're going to use the most.  
The first one is App Store which will export the app to App Store Connect where you can deploy it on TestFlight or App Store.  
The second one is Enterprise which will export the app to IPA file which you can then upload to labs.  
  
For more info about app distribution, you can look [here](https://help.apple.com/xcode/mac/current/#/devac02c5ab8).