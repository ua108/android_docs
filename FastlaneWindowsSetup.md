# Setup for Fastlane on Windows OS

***[Fastlane](https://docs.fastlane.tools/) is the easiest way to automate beta deployments and releases for your iOS and Android apps. It handles all tedious tasks, like generating screenshots, dealing with code signing, and releasing your application.***


## Contents
* [**1. Ruby DevKit Setup:**](#1-ruby-devkit-setup)
   * [1.1. Installation](#11-installation)
   * [1.2: Setup](#12-setup)
* [**2. Fastlane:**](#2-fastlane-setup)
   * [2.1. Fastlane Installation](#21-fastlane-installation)
   * [2.2: Fastlane Setup](#22-fastlane-setup)
* [**3: Fastfile and AppFile](#3-fastfile-and-appfile)
    * [3.1: AppFile](#31-appfile)
    * [3.2: FastFile](#32-fastfile)
* [**4. Instrumentation tests:**](#4-instrumentation-tests)
   * [4.1: Onboarding instrumentation tests](#41-onboarding-instrumentation-tests)

   

  
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

 ## 3. Fastfile and AppFile: 
 
 Once the Fastlane setup for the project is done. We need two file for futher configuration.
 
 * Type ```fastlane test``` in the command prompt.
 
 This will create a [fastlane](https://github.com/ua108/android_client_carpeesh/tree/master/fastlane) folder with two files called [Fastfile](https://github.com/ua108/android_client_carpeesh/blob/master/fastlane/Fastfile) and [AppFile](https://github.com/ua108/android_client_carpeesh/blob/master/fastlane/Appfile).
 
 #### 3.1 [AppFile](https://github.com/ua108/android_client_carpeesh/blob/master/fastlane/Appfile):
 
 The Appfile stores useful information that are used across all fastlane tools like the application Bundle Identifier, to deploy your lanes faster and tailored on your project needs. The AppFile takes a json key file which connects the Play store account internally. This file can be downloaded from [1Password vault](https://start.1password.com/open/i?a=OABLEGTLPRB3XGWOI2ORCRUNUE&h=my.1password.eu&i=dti6cyqfwssaq2pxtd6n32ehce&v=2zu3qtf2vpsrn5xi7xxr6iyuli) with access permissions. Once downloaded, it can be saved to the project parent folder.
 
 ```
 // API key file. This is a JSON file that contains the credential data that fastlane uses internally to connect to your Play Store account.
 json_key_file("../_secret/api-899748*******-******8-**********8d.json")
 
 // App bundle id
 package_name("com.urbananalytica.carpeesh") # e.g. com.krausefx.app
 ```
 
  #### 3.2 [FastFile](https://github.com/ua108/android_client_carpeesh/blob/master/fastlane/Fastfile):
  
  This stores the automation configuration that can be run with fastlane.

  ```
default_platform(:android)

# Webhook url which can be found on slack apps https://api.slack.com/apps/AH****2W/incoming-webhooks?
ENV["SLACK_URL"] = "https://hooks.slack.com/services/*******Q/*******8/******************Q1"
# Slack channel name which should be updated
ENV["SLACK_CHANNEL"] = "#fastlane"

platform :android do
  desc "Runs all the tests"
  lane :test do
    gradle(task: "test")
  end

  
  lane :internal do
    gradle(
        task: "clean"
        )

    
  increment_version_name(app_project_dir: '**/carpeesh', bump_type: 'minor')
  increment_version_code(app_project_dir: '**/carpeesh')
    gradle(
	task: 'assemble',
    	build_type: 'Release',
  	print_command: false,
  	properties: {
    "android.injected.signing.store.file" => "file_path",
    "android.injected.signing.store.password" => "keystore_password",
    "android.injected.signing.key.alias" => "alias",
    "android.injected.signing.key.password" => "key_password"
  	}

    )

  upload_to_play_store(
	track: 'internal',
	skip_upload_screenshots: true
	)

  version = "Internal release version: " + get_version_name(app_project_dir: '**/carpeesh')

    slack(message: version)
    # sh "your_script.sh"
    # You can also use other beta testing services here
  end

  desc "Deploy a new version to the Google Play"
  lane :deploy do
    gradle(task: "clean assembleRelease")
    upload_to_play_store
  end

lane :icon do
  android_appicon(
    appicon_image_file: 'fastlane/metadata/app_icon.png',
    appicon_icon_types: [:launcher],
    appicon_path: 'carpeesh/src/main/res/mipmap'
  )

end
end
  ```



 ## 4. Instrumentation Tests: 
 
***To capture screenshots and create metadata we need to have instrumentation tests to check the sanity of the app and automatic creation on the metadata.***

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

 #### 4.1. [Onboarding instrumentation tests](https://github.com/ua108/android_client_carpeesh/blob/master/carpeesh/src/androidTest/java/com/urbananalytica/carpeesh/OnboardingScreenshotTest.java): 
 
 ```
   @Test
    public void testTakeScreenshot() {

        Context appContext = InstrumentationRegistry.getTargetContext();
        Intent i0 = new Intent(appContext, Onboarding0Activity.class);
        //Pass dummy url data
        i0.putExtra("urldata", "eyJUT***************QyMjM5NzE0NCJ9");
        ob0.launchActivity(i0);
        //Take a screen shot of Onboarding 0 screen
        Screengrab.screenshot("Onboarding1");
        ob0.finishActivity();

        // Launch onboarding 1 screen
        Intent i1 = new Intent(appContext, Onboarding1Activity.class);
        ob1.launchActivity(i1);
        
        //Take a screenshot of onboarding 2 screen
        Screengrab.screenshot("Onboarding2");
 ```

* With instrumentation testing on Android, when you install a separate APK package, it installs the test APK to drive the UI automation. To do so enter the following command in the project directory with the gradlew file.

```gradlew assembleDebug assembleAndroidTest```

* Once completed there will be two apk files i.e a normal APK saved under ```android_client_carpeesh\carpeesh\build\outputs\apk\debug\carpeesh-debug.apk``` and the test APK under ```android_client_carpeesh\carpeesh\build\outputs\apk\androidTest\debug\carpeesh-debug-androidTest.apk```
