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