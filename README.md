# setup react native env ([origin](https://reactnative.dev/docs/environment-setup))
1. install node
2. install jdk
3. set env JAVA_HOME
4. install android sdk (or android studio)
5. set env:
```
	export ANDROID_HOME=$HOME/Android/Sdk
	export PATH=$PATH:$ANDROID_HOME/emulator
	export PATH=$PATH:$ANDROID_HOME/tools
	export PATH=$PATH:$ANDROID_HOME/tools/bin
	export PATH=$PATH:$ANDROID_HOME/platform-tools
```
6. install watchman?
7. install android sdks & tools via sdk manager (including NDK)
8. npm install -g expo-cli

# create project
1. expo init [proj_name]
2. choose bareminimum

# expo build ([origin](https://docs.expo.io/distribution/building-standalone-apps/#__next)
1. add bundleIds to app.json:
```	 {
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

2. for android:

	expo build:android -t apk
	
or config app [signing](https://docs.expo.io/distribution/app-signing/) then:

	expo build:android -t app-bundle 

# android native build @https://reactnative.dev/docs/signed-apk-android
1. npm install -g jetifier
2. npx jetify
3. (run on background terminal) npx react-native start
4. cd android
5. add to build.gradle below allprojects:
```
	tasks.withType(Javadoc).all { enabled = false }
```

## setup keystore
1. keytool -genkeypair -v -keystore [my-upload-key].keystore -alias [my-key-alias] -keyalg RSA -keysize 2048 -validity 10000
2. place keystore on android dir
3. add to gradle.properties:
```
	MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
	MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
	MYAPP_UPLOAD_STORE_PASSWORD=*****
	MYAPP_UPLOAD_KEY_PASSWORD=*****
```
*Note:*
* do NOT push gradle.properties to git!!
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
5. ./gradlew bundleRelease
