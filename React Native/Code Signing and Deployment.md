# Android

## Signing
To distribute your app on Google Play, App signing is required. Play App Signing uses two keys which are the *app signing key* and the *upload key*. The difference between the two is explained [here](https://developer.android.com/studio/publish/app-signing#keys_keystores). The *upload key* is needed for developers to upload the app to Google Play while the *app signing key* is used to sign the final app that users get from Play Store.  
The *app signing key* is created and managed by Google unless you choose not to allow it, but that's not recommended. If you choose to manage the app signing by yourself look [here](https://developer.android.com/studio/publish/app-signing#opt-out). Remember, if you lose the *app signing key*, you will not be able to update your app on Play Store. On the other hand, the *upload key* can be reset by [contacting support](https://support.google.com/googleplay/android-developer/answer/7384423#reset).  
This chapter will assume opting in to Play App Signing.

## Generating an upload key
To create your *upload key*, you can run this command in the terminal. *(do it in* `android/app`*)*  
  
On Windows:
```bash
keytool -genkeypair -v -keystore release.keystore -alias my-key-alias -storetype PKCS12 -keyalg RSA -keysize 2048 -validity 10000
```
  
On Mac:
```bash
keytool -genkey -v -keystore release.keystore -alias my-key-alias -storetype PKCS12 -keyalg RSA -keysize 2048 -validity 10000
```
  
You will be prompted for a password and some more info like name, organization, city...  
The result will be a file `release.keystore`.  

## Setting up Gradle variables
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

Then to set values to those variables (`MYAPP_KEY_ALIAS` and `MYAPP_KEY_PASSWORD`), create *(or edit)* the `android/gradle.properties` file and add values you used in creating the keystore.

```
MYAPP_KEY_ALIAS=my-key-alias
MYAPP_KEY_PASSWORD=*****
```

## Building

### Generating the APK

If you want to generate the apk file for testing purposes or uploading to Labs, you can do it by running this command in `android` directory:  
```bash
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

### Generating the bundle

If you want to upload the app to Google Play, you'll have to generate an AAB file.  
Like with generating the APK, you use the same command but replace `assemble` with `bundle`, like this:  
```bash
./gradlew bundleRelease
```
The AAB will be located in `android/app/build/outputs/bundle/release/app-release.aab` and you can upload it to Google Play.  

## Releasing

You can follow the steps for releasing the app [here](https://infinum.com/handbook/books/android/deployment/release-practices).