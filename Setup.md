# Setup Steps from Tutorial 4 to 5
This guide will show you the steps to perform if you are working from Tutorial 4 source code.

# Components to install

## Camera Module

```
yarn add react-native-camera
cd ios && pod install && cd ..
```

Add the following lines in the info.plist for iOS platform.

```
<!-- Required with iOS 10 and higher -->
<key>NSCameraUsageDescription</key>
<string>Allow the user to use the camera to take profile picture.</string>

<!-- Required with iOS 11 and higher: include this only if you are planning to use the camera roll -->
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Allow the user to pick an existing picture as the profile photo.</string>

<!-- Include this only if you are planning to use the camera roll -->
<key>NSPhotoLibraryUsageDescription</key>
<string>Allow the user to pick an existing picture as the profile photo.</string>

<!-- Include this only if you are planning to use the microphone for video recording -->
<key>NSMicrophoneUsageDescription</key>
<string>Allow the user to record a video with sound for profile photo.</string>
```

Add in the following permissions for Android platform in the android/app/src/main/AndroidManifest.xml.

```
<!-- Required -->
<uses-permission android:name="android.permission.CAMERA" />

<!-- Include this only if you are planning to use the camera roll -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

<!-- Include this only if you are planning to use the microphone for video recording -->
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
```

Insert the following lines in android/app/build.gradle:

```
android {
  ...
  defaultConfig {
    ...
    missingDimensionStrategy 'react-native-camera', 'general' // <--- insert this line
  }
}
```

Setup the TextDector for the camera module in iOS. Edit the Podfile in ios folder.

```
pod 'react-native-camera', path: '../node_modules/react-native-camera', subspecs: [
    'TextDetector'
  ]
```
Then run the `cd ios && pod install && cd ..` command.

Setup the MLKit for iOS

* Register your app in Firebase console.
* Download GoogleService-Info.plist and add it to your project
* Add pod 'Firebase/Core' to your podfile
* In your AppDelegate.m file add the following lines:
  ```
  #import <Firebase.h> // <--- add this
  ...

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
  {
    [FIRApp configure]; // <--- add this
    ...
  }
  ```

Setup the MLKit for Android

Modify the following lines in android/app/build.gradle:
```
android {
  ...
  defaultConfig {
    ...
    missingDimensionStrategy 'react-native-camera', 'mlkit' // <--- replace general with mlkit
  }
}
```

* Register your app in Firebase console.
* Download google-services.json and place it in android/app/
* Add the folowing to project level build.gradle:
```
buildscript {
  dependencies {
  // Add this line
  classpath 'com.google.gms:google-services:4.3.3' // <--- you might want to use different version
  }
}
```
* add to the android/app/build.gradle file
```
dependencies {
  ...
  implementation 'com.google.firebase:firebase-analytics:17.2.2'
  ...
}

...
apply plugin: 'com.android.application'
apply plugin: 'com.google.gms.google-services'
```
* Configure your app to automatically download the ML model to the device after your app is installed from the Play Store. If you do not enable install-time model downloads, the model will be downloaded the first time you run the on-device detector. Requests you make before the download has completed will produce no results.
```
<application ...>
...
  <meta-data
      android:name="com.google.firebase.ml.vision.DEPENDENCIES"
      android:value="ocr, face" /> <!-- choose models that you will use -->
</application>
```

Enable Multiple Dex for Android.

https://developer.android.com/studio/build/multidex

In the android/app/build.gradle add
```
android {
    defaultConfig {
        ...
        multiDexEnabled true  <== Add in this line
    }
    ...
}
```
In the same file dependencies section, add
```
dependencies {
  ...
  implementation 'com.android.support:multidex:1.0.3'
  ...
}
```

In the android/app/src/main/java/MainApplication.java, replace the extends Application to MultiDexApplication

```
import androidx.multidex.MultiDexApplication;

public class MainApplication extends MultiDexApplication implements ReactApplication {
  ...
}
```

Run the following command to clear the android build files and rebuild the application.

```
cd android && ./gradlew clean && cd ..
npx react-native run-android
```

## React Native FileSystem
We need to install the react-native-fs npm module to allow the app to save the photo capture by the camera so that it can be display on the user profile photo.

```
yarn add react-native-fs
```

Add the following in the ios/Podfile

```
pod 'RNFS', :path => '../node_modules/react-native-fs'
```
Change the Flipper in ios/Podfile from 0.33.1 to 0.37.0 to solve Image component bug when displaying base64 image in iOS.

```
def add_flipper_pods!(versions = {})
  #versions['Flipper'] ||= '~> 0.33.1'
  versions['Flipper'] ||= '~> 0.37.0'
  ...
```

then run the command `cd ios && pod install && cd ..`