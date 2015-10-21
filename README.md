react-native-code-push
===

React Native module for deploying script updates using the CodePush service.

Installation (iOS)
---

```
npm install --save react-native-code-push
```

After installing the React Native CodePush plugin, open your project in Xcode. Open the `react-native-code-push` in Finder, and drag the `CodePush.xcodeproj` into the Libraries folder of Xcode.

![Add CodePush to project](https://cloud.githubusercontent.com/assets/516559/10322414/7688748e-6c32-11e5-83c1-00d3e6758df4.png)


In Xcode, click on your project, and select the "Build Phases" tab of your project configuration. Drag libCodePush.a from `Libraries/CodePush.xcodeproj/Products` into the "Link Binary With Libraries" secton of your project's "Build Phases" configuration.

![Link CodePush during build](https://cloud.githubusercontent.com/assets/516559/10322221/a75ea066-6c31-11e5-9d88-ff6f6a4d6968.png)

Under the "Build Settings" tab of your project configuration, find the "Header Search Paths" section and edit the value.
Add a new value, `$(SRCROOT)/../node_modules/react-native-code-push` and select "recursive" in the dropdown.

![Add CodePush library reference](https://cloud.githubusercontent.com/assets/516559/10322038/b8157962-6c30-11e5-9264-494d65fd2626.png)

Finally, edit your project's `AppDelegate.m`.

At the top of the file, add the following line to import the CodePush headers.

```
#import "CodePush.h"
```

Then, find the following code:

```
jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle?platform=ios&dev=true"];
```

Replace it with this:

```
jsCodeLocation = [CodePush getBundleUrl];
```

This change allows CodePush to load the updated app location after an update has been applied.
Before any updates are installed, CodePush will load your app from the bundled "main.jsbundle" file.
After updates are installed, CodePush will load your app from the writable user directory, where the update has been downloaded.

Installation (Android)
---

```
npm install --save react-native-code-push
```

Then, download [ReactNativeCodePush.aar](https://github.com/Microsoft/react-native-code-push/blob/first-check-in-for-android/Examples/CodePushDemoApp/android/CodePushSDK/ReactNativeCodePush.aar?raw=true)

After installing the React Native CodePush plugin, open your React Native project in Android Studio. Go to ```File > New Module > Import .JAR/.AAR Package``` and click Next. Browse to your downloaded .AAR file and click Next. This will import the library package into your Project bundle.

Next, go to ```File > Project Structure```. Click on ```app``` (or whatever your main project module is named), go to ```Dependencies```, click on the "+" sign at the bottom and add the new ```ReactNativeCodePush``` module as a Module Dependency. Click Ok. 

![Add Module Dependency](https://cloud.githubusercontent.com/assets/8598682/10624328/414e2cc6-774c-11e5-9a52-d4551f01e7a1.png)

Finally, copy and paste this [code](/Examples/CodePushDemoApp/android/app/src/main/java/com/codepushdemoapp/MainActivity.java) into your ```MainActivity.java```.

Methods
---

* [checkForUpdate](#codepushcheckforupdate): Checks the service for updates
* [notifyApplicationReady](#codepushnotifyapplicationready): Notifies the plugin that the update operation succeeded. **Note: currently only supported in iOS**.
* [getCurrentPackage](#codepushgetcurrentpackage): Gets information about the currently applied package.

Objects
---

* [LocalPackage](#localpackage): Contains information about a locally installed package.
* [RemotePackage](#remotepackage): Contains information about an updated package available for download.

Getting Started:
---

* Add the plugin to your app
* Open your app's `Info.plist` and add a "CodePushDeploymentKey" entry with your app's deployment key
* In your app's `Info.plist` make sure your "CFBundleShortVersionString" value is a valid [semver](http://semver.org/) version.

* To publish an update for your app, run `react-native bundle`, and then publish `iOS/main.jsbundle` using the CodePush CLI.

Running the Example
---

* Clone this repository
* From the root of this project, run `npm install`
* `cd` into `Examples/CodePushDemoApp`
* From this demo app folder, run `npm install`
* Open `Info.plist` and fill in the value for CodePushDeploymentKey
* Run `npm start` to launch the packager
* Open `CodePushDemoApp.xcodeproj` in Xcode
* Launch the project

Running Tests
---

* Open `CodePushDemoApp.xcodeproj` in Xcode
* Navigate to the test explorer (small grey diamond near top left)
* Click on the 'play' button next to CodePushDemoAppTests
* After the tests are completed, green ticks should appear next to the test cases to indicate success


## LocalPackage
Contains details about an update package that has been downloaded locally or already applied (currently installed package).
### Properties
- __deploymentKey__: Deployment key of the package. (String)
- __description__: Package description. (String)
- __label__: Package label. (String)
- __appVersion__: The native version of the application this package update is intended for. (String)
- __isMandatory__: Flag indicating if the update is mandatory. (Boolean)
- __packageHash__: The hash value of the package. (String)
- __packageSize__: The size of the package, in bytes. (Number)

### Methods
- __apply(rollbackTimeout): Promise__: Applies this package to the application. The application will be reloaded with this package and on every application launch this package will be loaded.
If the rollbackTimeout parameter is provided, the application will wait for a codePush.notifyApplicationReady() for the given number of milliseconds.
If codePush.notifyApplicationReady() is called before the time period specified by rollbackTimeout, the apply operation is considered a success.
Otherwise, the apply operation will be marked as failed, and the application is reverted to its previous version. **Note: Rollbacks are currently only supported in iOS.**

## RemotePackage
Contains details about an update package that is available for download.
### Properties
- __deploymentKey__: Deployment key of the package. (String)
- __description__: Package description. (String)
- __label__: Package label. (String)
- __appVersion__: The native version of the application this package update is intended for. (String)
- __isMandatory__: Flag indicating if the update is mandatory. (Boolean)
- __packageHash__: The hash value of the package. (String)
- __packageSize__: The size of the package, in bytes. (Number)
- __downloadUrl__: The URL at which the package is available for download. (String)

### Methods
- __download(): Promise<LocalPackage>__: Downloads the package update from the CodePush service. Returns a Promise that resolves with the LocalPackage.

## codePush.checkForUpdate
Queries the CodePush server for updates.
```javascript
codePush.checkForUpdate(): Promise<RemotePackage>;
```

`checkForUpdate` returns a Promise that resolves when the server responds with an update.


Usage: 
```javascript
codePush.checkForUpdate().then((update) => {
    console.log(update);
});
```

## codePush.getCurrentPackage
```javascript
codePush.getCurrentPackage(): Promise<LocalPackage>;
```
Get the currently installed package information. Returns a Promise that resolves with the local package.

## codePush.notifyApplicationReady
```javascript
codePush.notifyApplicationReady(): Promise<void>;
```

Notifies the plugin that the update operation succeeded.
Calling this function is required if a rollbackTimeout parameter is passed to your ```LocalPackage.apply``` call.
If automatic rollback was not used, calling this function is not required and will result in a noop.
