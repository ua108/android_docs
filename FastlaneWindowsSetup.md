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
* [**4. Deploying to Play Store:**](#4-deploying-to-play_store)
  

   

  
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
 
 * Validate the json key file by executing the following command. Replace the <local_json_key_path> with the local path of the json file downloaded and saved from 1Password.
 
 ```bundle exec fastlane run validate_play_store_json_key json_key:<local_json_key_path>```
 
 * On successful validation, the following messages will be displayed.
 
  [<img src="https://github.com/ua108/android_docs/blob/master/Screenshots/google_play_connect.PNG" width = "500" height="300"/>](https://github.com/ua108/android_docs/blob/master/Screenshots/google_play_connect.PNG)
 
 
  #### 3.2 [FastFile](https://github.com/ua108/android_client_carpeesh/blob/master/fastlane/Fastfile):
  
  This stores the automation configuration that can be run with fastlane. Fastfile consists of different lanes that when executed along with ```fastlane <lanename>``` performs certain action specified within the lane.

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

 desc "Build"
 lane :build do
   gradle(task: "clean assembleRelease")
 end
  
  lane :internal do
    gradle(
        task: "clean"
        )

  #Add build signing details for apk generation
  increment_version_name(app_project_dir: '**/carpeesh', bump_type: 'minor')
  increment_version_code(app_project_dir: '**/carpeesh')
    gradle(
	task: 'assemble',
    	build_type: 'Release',
  	print_command: false,
  	properties: {
    "android.injected.signing.store.file" => "keystore_file_path",
    "android.injected.signing.store.password" => "keystore_password",
    "android.injected.signing.key.alias" => "alias",
    "android.injected.signing.key.password" => "key_password"
  	}

    )

# Action to upload to play store with parameters .eg Do not upload automatic screenshots. 
# Uploads the APK built in the gradle step above and releases it to all production users
  upload_to_play_store(
	track: 'internal',
	skip_upload_screenshots: true
	)

  version = "Internal release version: " + get_version_name(app_project_dir: '**/carpeesh')

    slack(message: version)
    # sh "your_script.sh"
    # You can also use other beta testing services here
  end

# Deploy the build to the associated Play store account.
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
  
  * The [keystore](https://start.1password.com/open/i?a=OABLEGTLPRB3XGWOI2ORCRUNUE&h=my.1password.eu&i=oraj2a2krxahvr2spo3xaa2ode&v=2zu3qtf2vpsrn5xi7xxr6iyuli) file is downloaded from 1Password and stored  locally. Replace the values for ```keystore_file_path, keystore_password, alias, key_password``` with the local keystore file path and the credentials mentioned on 1Password.
  
  * Execute the following command to test if the apk is generated successfully.
  
  ```bundle exec fastlane build```
  
  On successful execution, there should be a release build created in the folder ```...\carpeesh\build\outputs\apk\release```



 ## 4. Deploying to Play Store: 

Now, we have configured our Fastfile and Appfile. Since the app is already listed on the Google play store, we need not create another listing for it. To download the metadata uploaded on Google play listing to your local directory use the following 

```fastlane supply init```

This will download the metadata like images, screenshots, icons, description etc. from the Google play store to [```fastlane/metadata/android```](https://github.com/ua108/android_client_carpeesh/tree/master/fastlane/metadata/android/en-US) folder.


Next, we can upload the new build.

Increment the build number and version name in the [```build.gradle```](https://github.com/ua108/android_client_carpeesh/blob/master/carpeesh/build.gradle) file. 

```
  versionCode 93
  versionName "0.93.0"
```

* Once done ,  execute the following command 

```bundle exec fastlane internal```

When that completes you should have the appropriate APK ready to go in the standard output directory. To upload your apk and metadata to Google Play Store. This will upload the apk file to the "Internal test" track on the Google play store.

After it runs to success, you can go to [Google Play Console](https://play.google.com/apps/publish/?account=8997481909084941506&noredirect=#ManageReleasesPlace:p=com.urbananalytica.carpeesh&appid=4975293088165532182) > Click on Carpeesh > App Releases > Internal test track > EDIT RELEASE

Back in the Fastfile we can see a code block as follows :

```
# Deploy the build to the associated Play store account.
  desc "Deploy a new version to the Google Play"
  lane :deploy do
    gradle(task: "clean assembleRelease")
    upload_to_play_store
  end
```
When we run that lane it will create sign apk and deploy to Google Play.

```upload_to_play_store``` is Fastlane action. ```track``` is its parameter which indicate that whether you want to release your apk to Beta or Production. you can see its available parameter via ```bundle exec fastlane action upload_to_play_store```

* Now we can run lane deploy to deploy the app to the **Production** track via: 

``` bundle exec fastlane deploy```

* In Android App Bundles and APKs to add you will see your apk that you have built or if you don’t see your apk, you can click on ADD FROM LIBRARY.

* The app is now ready to be published.
