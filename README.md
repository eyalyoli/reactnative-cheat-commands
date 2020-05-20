#setup react native env @https://reactnative.dev/docs/environment-setup
install node
install jdk
set env JAVA_HOME
install android sdk (or android studio)
set env:
	export ANDROID_HOME=$HOME/Android/Sdk
	export PATH=$PATH:$ANDROID_HOME/emulator
	export PATH=$PATH:$ANDROID_HOME/tools
	export PATH=$PATH:$ANDROID_HOME/tools/bin
	export PATH=$PATH:$ANDROID_HOME/platform-tools
install watchman?

install android sdks & tools via sdk manager (including NDK)

npm install -g expo-cli

#create project
expo init [proj_name]
choose bareminimum

#expo build @https://docs.expo.io/distribution/building-standalone-apps/#__next
add bundleIds to app.json:
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

for android:
	expo build:android -t apk
or config app signing @https://docs.expo.io/distribution/app-signing/ then:
	expo build:android -t app-bundle 

#android native build @https://reactnative.dev/docs/signed-apk-android
npm install -g jetifier
npx jetify

(on background terminal)
npx react-native start

cd android
add to build.gradle below allprojects:
	tasks.withType(Javadoc).all { enabled = false }

##setup keystore
keytool -genkeypair -v -keystore [my-upload-key].keystore -alias [my-key-alias] -keyalg RSA -keysize 2048 -validity 10000
place keystore on android dir
add to gradle.properties:
	MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
	MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
	MYAPP_UPLOAD_STORE_PASSWORD=*****
	MYAPP_UPLOAD_KEY_PASSWORD=*****
* do NOT push gradle.properties to git!!
** a more secure way can be provided using OS tools
setup signing options in app/build.gradle:
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


./gradlew bundleRelease
