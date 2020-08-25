# Setup for Fastlane on Windows OS

***[Fastlane](https://docs.fastlane.tools/) is the easiest way to automate beta deployments and releases for your iOS and Android apps. It handles all tedious tasks, like generating screenshots, dealing with code signing, and releasing your application.***


## Contents
* [**1. Ruby DevKit Setup:**](#1-ruby-devkit-setup)
   * [1.1. Installation](#11-installation)
   * [1.2: Setup](#12-setup)
* [**2. Fastlane:**](#2-fastlane-setup)
   * [2.1. Fastlane Installation](#21-fastlane-installation)
   * [2.2: Fastlane Setup](#22-fastlane-setup)
   * [2.3: Fastfile and AppFile](#23-fastfile-and-appfile)
* [**3. Screengrab:**](#3-screengrab-setup)
   * [3.1: Screengrab Installation](#31-screengrab-installation)
   

  
  ## 1. Ruby DevKit Setup:
  
  Currently, fastlane is officially supported to run on macOS, but the Fastlane team is also working upon supporting bits of functionalities on Windows OS. To use Fastlane on Windows, we would need to install Ruby Devkit.
  
 #### 1.1. Installation: 
 Install the  [Ruby devkit](https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-2.6.6-1/rubyinstaller-2.6.6-1-x64.exe) package or download the latest version  from the [release notes](https://github.com/larskanis/rubyinstaller2/releases).
 
 #### 1.2. Setup: 
 
 Ruby 2.4 and newer integrates well into MSYS2 after installation on the target system to provide a build-and-runtime environment for installation of gems with C-extensions hence we would now want to install the MSYS2 toolchain.
 
 Once installed, Ruby devkit would run automatically asking what all components are required to be installed.  Just press **Enter** without any input to let the installer install the required components based on your system configuration.
 
 [<img src="https://github.com/ua108/android_docs/blob/master/Screenshots/ruby_install.png" width = "500" height="300"/>](https://github.com/ua108/android_docs/blob/master/Screenshots/ruby_install.png)
 
 
 ## 2. Fastlane Setup: 
 
[Reference tutorial](https://www.raywenderlich.com/10187451-fastlane-tutorial-for-android-getting-started#toc-anchor-010)
  
 #### 2.1. Fastlane Installation: 
 
 Once the Ruby devkit is installed, open command prompt and enter the following command to install fastlane.
 
 ``` gem install fastlane -NV```

 
 #### 2.2. Fastlane Setup: 

##### Fastlane is already initialized on the Carpeesh project(https://github.com/ua108/android_client_carpeesh/tree/master/fastlane). So we do not need to initialize this again. In case it is not initialized, the following steps can be followed to initialize fastlane.#####

* After fastlane is installed on your system, we need to initialize fastlane in the project if not done already. To do this, open the command prompt in the project directory and enter the following command.
 
 ```fastlane init```
 
* When prompted with ```Package Name (com.krausefx.app)```, enter the app’s unique package name.

* Type ```n``` when ```Path to .json secret``` message is prompted.

* Type ```n``` when ```Do you plan on uploading metadata, screenshots & builds to Google play using fastlane?```
 
* Press **Enter** for the next few questions.

 #### 2.3. Fastfile and AppFile: 
 
 Once the Fastlane setup for the project is done. We need two file for futher configuration.
 
 * Type ```fastlane test``` in the command prompt.
 
 This will create a fastlane folder with two files called Fastfile and AppFile.
 
  ## 3. Screengrab Setup: 
 
***Fastlane’s [screengrab](https://docs.fastlane.tools/actions/screengrab/) is an action that generates localized screenshots of your Android app for different device types and languages.***

 #### 3.1. Screengrab Installation: 
 
 To use the Screengrab tool we need the command line tool first. Type the following command to install this :
 
 ```gem install screengrab```
 
 * Add required permissions so that the UI tests can wake up the device to take screenshots. We have configured that in the [library project manifest file](https://github.com/ua108/android_lib/blob/master/shared_lib/src/main/AndroidManifest.xml)
 
 ```
  <!-- Allows unlocking your device and activating its screen so UI tests can succeed -->
  <uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
  <uses-permission android:name="android.permission.WAKE_LOCK" />

  <!-- Allows for storing and retrieving screenshots -->
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

  <!-- Allows changing locales -->
  <uses-permission xmlns:tools="http://schemas.android.com/tools"
    android:name="android.permission.CHANGE_CONFIGURATION"
    tools:ignore="ProtectedPermissions" />
```

* Add the dependencies in the app's [build.gradle](https://github.com/ua108/android_client_carpeesh/blob/master/carpeesh/build.gradle) for writing instrumentation tests.
```
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test:runner:1.1.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.0'
    androidTestImplementation 'com.android.support.test:rules:1.0.2'
    androidTestImplementation 'androidx.test:monitor:1.1.0'
    androidTestImplementation 'tools.fastlane:screengrab:1.2.0'
```

* Inside the defaultConfig block, add testInstrumentationRunner:

```
testInstrumentationRunner 'androidx.test.runner.AndroidJUnitRunner'
```
