# Setup react native env ([origin](https://reactnative.dev/docs/environment-setup))
## React native with expo-cli
1. install node
2. install jdk - better to go with openjdk, gradle 6.3 (check your gradle compatibility) supports jdk14 (you can check the version with ```java -version```), for macOS ```brew cask install adoptopenjdk14```
3. set env JAVA_HOME
4. install watchman (for macOS it is a must - `brew install watchman`)
5. ```npm install -g expo-cli```
6. ```npm install -g react-native react-native-cli``` - recommended as the project is moving to pure RN


## Android setup
1. install android cmdtools (or android studio)
	
	1.1. for cmdtools - extract zip to ```$ANDROID_HOME/cmdline-tools/```
	
	1.2. ```tools``` and ```tools/bin``` now should be under ```cmdline-tools``` (when setting env)
2. set env:
```
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```
3. install android sdks & tools via sdk manager (including NDK), example:
```
add-ons;addon-google_apis-google-24
platform-tools
platforms;android-28
emulator
extras;google;google_play_services
system-images;android-28;google_apis_playstore;x86_64
ndk-bundle
```

## iOS setup
1. all installation are done with brew - ```/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)```
2. install [cocaopods](https://guides.cocoapods.org/using/getting-started.html) - deps managment
3. `npx pod install`

# Project management
## Create project
1. expo init [proj_name]
2. choose bareminimum

## Run project
### Using expo
1. ```expo start``` - this runs the metro bundler
2. choose from the opened web UI where to run the app

### Using react-native
1. to run the metro bundler, run on a background terminal: ```npx react-native start```
2. ```npx react-native run-android/run-ios```

### File watches limit fix
If after running the metro bundler you get warnings/errors on number of file watched was reached then run:
```
echo 524288 | sudo tee -a /proc/sys/fs/inotify/max_user_watches
```

# Android build

## Using expo ([origin](https://docs.expo.io/distribution/building-standalone-apps/#__next))
1. add bundleIds to app.json:
```
{
   "expo": {
    ...
    "ios": {
      "bundleIdentifier": "com.yourcompany.yourappname",
      "buildNumber": "1.0.0"
    },
    "android": {
      "package": "com.yourcompany.yourappname",
      "versionCode": 1
    }
    ...
   }
 }
```

2. run:

	expo build:android -t apk
	
or config app [signing](https://docs.expo.io/distribution/app-signing/) then run:

```expo build:android -t app-bundle```

## Android native build ([origin](https://reactnative.dev/docs/signed-apk-android))
1. ``npm install -g jetifier``
2. ``npx jetify``
3. run on background terminal: ``npx react-native start``
4. ```cd android```
5. setup keystore (see below) 
6. For direct install on device use APK format:
	```./gradlew assembleRelease``` (add ```--stacktrace``` to catch error)
   For publishing use Android App Bundle format:
	```./gradlew bundleRelease``` (add ```--stacktrace``` to catch error)
The build will wait for you in ```/android/app/build/outputs/```

## Setup keystore
1. ```keytool -genkeypair -v -keystore [my-upload-key].keystore -alias [my-key-alias] -keyalg RSA -keysize 2048 -validity 10000```
2. place keystore on android dir
3. add to gradle.properties:
```
MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
MYAPP_UPLOAD_STORE_PASSWORD=*****
MYAPP_UPLOAD_KEY_PASSWORD=*****
```
*Note:*
* **do NOT push gradle.properties to git!!**
* a more secure way can be provided using OS tools
4. setup signing options in app/build.gradle:
```
...
android {
    ...
    defaultConfig { ... }
    signingConfigs {
	release {
	    if (project.hasProperty('MYAPP_UPLOAD_STORE_FILE')) {
		storeFile file(MYAPP_UPLOAD_STORE_FILE)
		storePassword MYAPP_UPLOAD_STORE_PASSWORD
		keyAlias MYAPP_UPLOAD_KEY_ALIAS
		keyPassword MYAPP_UPLOAD_KEY_PASSWORD
	    }
	}
    }
    buildTypes {
	release {
	    ...
	    signingConfig signingConfigs.release
	}
    }
}
...
```

## Build troubleshooting
* "Task :app:createReleaseExpoManifest FAILED" and "Error: Failed to connect to the packager server" => check step 3!
* "Task :react-native-reanimated:androidJavadoc FAILED" => add to build.gradle below allprojects:
```
tasks.withType(Javadoc).all { enabled = false }
```
* "Lint found error on the project. Aborting build..." => add to the module's build.gradle under android:
```
lintOptions {
	abortOnError false
}
```
* OutOfMemory errors on build => add to gradle.properties on android:
```
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m
```
* "Task :app:createReleaseExpoManifest FAILED'" and "Error: Cannot find module '@react-native-community/cli/build" => run
```
npm i --save-dev @react-native-community/cli
```
* The app gets build but can't run on a device becuase of hard to understand (not your code) errors => 
  1. maybe the device is not connected to ADB or your PC's network?
  2. maybe react-native is not installed correct globally?! You can check this by `npm list -g` this should show you what you installed in env setup above

# iOS build

## Background
Building uses Cocaopods (for dependency management) and Xcode. You need to have a Mac.

To simulate iOS products, we can use:
* a pyshical iOS device
* use [Appetize](https://appetize.io/) web service

## Install Mac on PC
1. install [Mac virtual machine](https://www.geekrar.com/install-macos-mojave-on-virtualbox-on-windows-pc-new-method/)
2. make a bootable Mac installation usb (using the virtual machine)
3. boot & install Mac from the usb

## Build troubleshooting
* "gyp: No Xcode or CLT version detected" => run
```
xcode-select --print-path
sudo rm -r -f /Library/Developer/CommandLineTools
xcode-select --install
```
  run xcode gui after that
* "SDK “iphoneos” cannot be located" => ```sudo xcode-select --switch /Applications/Xcode.app``` after installing xcode from app store
* "Cannot find module in /node_modules/..." and they exist => clean-up the project and re-run, else refresh `package-lock.json`, else use `npx` with `react-native` commands

# Libraries

## Kitten UI
* to run kittenUI for web you need to:
	1. ```npm i -D @expo/webpack-config```
	2. create ```webpack.config.js``` in root with the contents:
	```
	const createExpoWebpackConfigAsync = require('@expo/webpack-config');

	module.exports = async function(env, argv) {
	    const config = await createExpoWebpackConfigAsync({
		...env,
		babel: {
		    dangerouslyAddModulePathsToTranspile: ['@ui-kitten/components']
		}
	    }, argv);
	    return config;
	};
	```
