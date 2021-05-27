# Example of wrapping Create React App with Cordova

### Requirements

 - Deploy React app as a hosted web app separate from the native Cordova app
 - Build native Cordova app shell for Android and iOS that loads the hosted web app
 - Push notifications for iOS and Android
 - Inject Cordova javascript into web app so it can interact with native Cordova app

### Cordova Plugins Used

 - cordova-plugin-whitelist (so our native Cordova app can only load URLs specified)
 - @havesource/cordova-plugin-push (Cordova push notification support for iOS and Android)
 - cordova-plugin-remote-injection (inject Cordova javascript into Hosted remote app to access native functionality)

# 1st Commit - Initial CRA and Cordova Setup

Create React App

```
npx create-react-app cordova-create-react-app-example --template typescript

cd cordova-create-react-app-example
```

Cordova Create

```
npx cordova create cordova io.ryseapp.ryse Ryse
```

notes: "Ryse" is YOUR app name "io.ryseapp.ryse" is typically YOUR domain reversed with the app name appended

# 2nd Commit - Setup Cordova to be able to run on iOS and Android Emulators

```
cd cordova
npm install cordova -D // so that your teammates won't need to have it installed globally
```

Add to `scripts` in cordova `package.json`

```
  "scripts": {
    "cordova": "cordova"
  },
```

Add iOS and Android platforms - this generates code in "cordova/platforms" that you should not modify

```
npm run cordova platform add ios
npm run cordova platform add android
```

## Prerequisites for building Cordova app

### iOS

Following steps from https://cordova.apache.org/docs/en/10.x/guide/platforms/ios/index.html

Download `XCode` from App Store

```
xcode-select --install
brew install ios-deploy
sudo gem install cocoapods
```

#### On new macs with M1 chip run:

```
sudo arch -x86_64 gem install ffi
```

### Android

Following steps from: https://cordova.apache.org/docs/en/10.x/guide/platforms/android/index.html

Download `Android Studio` and run through setup wizard

Add `JAVA_HOME` and `ANDROID_SDK_ROOT` environment variables to your PATH

example for ~/.zshrc

```
export ANDROID_SDK_ROOT=/Users/bryanhunter/Library/Android/sdk
export PATH=\${PATH}:/Users/bryanhunter/Library/Android/sdk/tools:/Users/bryanhunter/Library/Android/sdk/tools/bin:/Users/bryanhunter/Library/Android/sdk/platform-tools

// find Java SDK with

/usr/libexec/java_home -V | grep jdk

// should look like /Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH

source ~/.zshrc
```

#### Android Emulator for new macs with M1 chip

Have at least Android SDK version 4.1.3.

To install from Android Studio:

Tools -> AVD Manager -> Create Virtual Device -> Pixel4 (with Google Play symbol) -> Next -> Select System Image / Other Images / Release S with ABI arm64-v8a

note: may have to start Emulator twice for it to work.


## Verify "cordova run" works for iOS and Android

### iOS

```
npm run cordova run ios
```

![Cordova Run iOS - initial](./CordovaRunIos-initial.png)

### Android

Start your Android Emulator first

```
npm run cordova run android
```

![Cordova Run Android - initial](./CordovaRunAndroid-initial.png)

# 3rd Commit - Run React app and load inside of Cordova

## Modify config.xml with the following

```
<widget id="io.ryseapp.ryse" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0" xmlns:android="http://schemas.android.com/apk/res/android">

    <content src="http://localhost:3000" />
    <allow-navigation href="http://localhost:3000/*" />
    <platform name="android">
        <edit-config file="app/src/main/AndroidManifest.xml" mode="merge" target="/manifest/application">
            <application android:usesCleartextTraffic="true" />
        </edit-config>
    </platform>

```

Note: we can remove everything under the `www` folder, because we're loading a hosted app and not a bundled one.  However, Cordova still requires a `www` folder to exist to run, so I just create a file named `_cordova-requires-www-folder-to-start` to avoid confusion.

## Start the React app and run Cordova on iOS and Android

Start React app in terminal

navigate to app root directory

```
yarn start
```

Run Cordova on iOS and Android in another terminal

```
cd cordova
npm run cordova run ios
npm run cordova run android
```

![Cordova Run with React App - initial](./CordovaRunWithReactApp-initial.png)

## Modify React app public/index.html so that it looks and feels native instead of in a nested browser

```
  ...

    <meta name="viewport" content="initial-scale=1, maximum-scale=1.0, height=device-height, width=device-width, viewport-fit=cover, user-scalable=no">

  ...

    <style>
      html {
          border: 0;
          margin: 0;
          padding:0px;
          width: 100%;
          /* has to be vh and not % for IOS to fill entire phone screen */
          height: 100vh;
          overflow: hidden;
      }
      body {
          border: 0;
          margin: 0;
          padding:0px;
          /* Padding to avoid the "unsafe" areas behind notches in the screen */
          padding: env(safe-area-inset-top, 0px) env(safe-area-inset-right, 0px) env(safe-area-inset-bottom, 0px) env(safe-area-inset-left, 0px);
          position: fixed;
          width: 100%;
          height: 100%;
          overflow: hidden;
          display: flex;
          flex-direction: column;
      }
      * {
          box-sizing: border-box;
      }
    </style>

  ...

    <div id="root" style="height: 100%; width: 100%;"></div>
```

# 4th Commit - Add cordova-plugin-remote-injection and verify access on React app

Temporarily add this to your React public/index.html

```
    <script>
      document.addEventListener('deviceready', () => alert('it works!'), false);
    </script>
```

You can verify that this "alert" does not display before you add the plugin.

Add the cordova-plugin-remote-injection plugin to your Cordova app

```
npm run cordova plugin add cordova-plugin-remote-injection
```

Start React app and run Cordova to verify access to injected Cordova javascript on React app

```
yarn start

...

npm run cordova run ios
```

![Cordova with Remote Injection Plugin - initial](./CordovaRemoteInjection.png)

Remove the `<script></script>` from public/index.html because we were just using that to test.

# 5th Commit - Add Icons for iOS and Android

Use https://appicon.co/ to Upload your icon (public/logo512.png for default Create React App)

Click Generate and Unzip the AppIcons folder and move it under /cordova

![App Icon Generator](./AppIconGenerator.png)

Add to your `config.xml`

```
    <!-- ICONS -->
    <platform name="ios">
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/16.png" width="16" height="16" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/20.png" width="20" height="20" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/29.png" width="29" height="29" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/32.png" width="32" height="32" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/40.png" width="40" height="40" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/48.png" width="48" height="48" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/50.png" width="50" height="50" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/55.png" width="55" height="55" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/57.png" width="57" height="57" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/58.png" width="58" height="58" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/60.png" width="60" height="60" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/64.png" width="64" height="64" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/72.png" width="72" height="72" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/76.png" width="76" height="76" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/80.png" width="80" height="80" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/87.png" width="87" height="87" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/88.png" width="88" height="88" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/100.png" width="100" height="100" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/114.png" width="114" height="114" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/120.png" width="120" height="120" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/128.png" width="128" height="128" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/144.png" width="144" height="144" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/152.png" width="152" height="152" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/167.png" width="167" height="167" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/172.png" width="172" height="172" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/180.png" width="180" height="180" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/196.png" width="196" height="196" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/216.png" width="216" height="216" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/256.png" width="256" height="256" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/512.png" width="512" height="512" />
        <icon src="AppIcons/Assets.xcassets/AppIcon.appiconset/1024.png" width="1024" height="1024" />
    </platform>
    <platform name="android">
        <icon src="AppIcons/android/mipmap-mdpi/ic_launcher.png" density="mdpi" />
        <icon src="AppIcons/android/mipmap-hdpi/ic_launcher.png" density="hdpi" />
        <icon src="AppIcons/android/mipmap-xhdpi/ic_launcher.png" density="xhdpi" />
        <icon src="AppIcons/android/mipmap-xxhdpi/ic_launcher.png" density="xxhdpi" />
        <icon src="AppIcons/android/mipmap-xxxhdpi/ic_launcher.png" density="xxxhdpi" />
    </platform>
```

Verify the Icons on Android and iOS

![Cordova Icons](./CordovaIcons.png)

# 6th Commit - Add Splash Screens

```
npm run cordova plugin add cordova-plugin-splashscreen
```

For both Android and iOS, you typically want your Splash Screen image to be centered and the main contents of the image to not be anywhere near the edges so that it doesn't accidentally get clipped.

For Android, you need PNGs of size:

- 300x300 - mdpi
- 450x450 - hdpi
- 600x600 - xhdpi
- 900x900 - xxhdpi
- 1200x1200 - xxxhdpi

For iOS, you can use a single universal image of size:

- 2732x2732 - universal

You can set the SplashScreen delay, we chose 3 seconds to give the React app a little bit of time to load before displaying it.

Add to your `config.xml`

```
    <!-- SPLASH SCREENS -->
    <platform name="android">
        <splash src="SplashScreens/android/port-hdpi.png" density="hdpi"/>
        <splash src="SplashScreens/android/port-mdpi.png" density="mdpi"/>
        <splash src="SplashScreens/android/port-xhdpi.png" density="xhdpi"/>
        <splash src="SplashScreens/android/port-xxhdpi.png" density="xxhdpi"/>
        <splash src="SplashScreens/android/port-xxxhdpi.png" density="xxxhdpi"/>
    </platform>
    <platform name="ios">
        <splash src="SplashScreens/ios/Default@2x~universal~anyany.png" />
    </platform>
    <preference name="SplashScreenDelay" value="3000" />
```

Verify the SplashScreens on Android and iOS

![Cordova Splash Screen](./CordovaSplashScreens.png)
