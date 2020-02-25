---
title: "Set Up Deep Links in Native Apps"
parent: "native-mobile"
menu_order: 72
description: "Connect URLs to your native app by adding a deep link".
tags: ["deeplink", "deep link", "url","native", "mobile", "developer", "native-builder", "developer app", "make it native"]
---

## 1. Introduction

While URLs typically open websites, they can also be used to open an installed app on your mobile divice. With this tutorial you will learn how to connect the URL `app://myapp` to your Mendix Native App installed on your Android or iOS device. It is also possible to pass additional data using path, query parameters, and hashes. Passing additional data could look like this: `app://myapp/task/123?action=close#info`.

A URL is constructed of these parts:

```txt
        username       host      port
        ┌──┴───┐ ┌──────┴──────┐ ┌┴┐
https://john.doe@www.example.com:123/forum/questions/?tag=networking&order=newest#top
└─┬─┘   └───────────┬──────────────┘└───────┬───────┘ └───────────┬─────────────┘ └┬┘
scheme          authority                  path                 query           hash
```

You can also register the handling of a normal weblink beginning with `http(s)://`. However this requires some more work for iOS, and is not covered in this tutorial [todo: is just iOS https not covered here, or both platforms not covered?]. In that case you could check [Universal Links: Make the Connection](https://www.raywenderlich.com/6080-universal-links-make-the-connection) by Owen L. Brown.

When an app is installed it registers the `schema` and optionally the `host` so the operating system will know what application should be opened when the URL is clicked. The application could either be closed or running in the background [todo: what?].

### 1.1 Testing

Please note that the Make It Native app has already the registered schema `makeitnative://` and can be used out of the box. If want to use the Make It Native app with that schema, see the [Using Deep Linking in Your App](#using-deep-linking) section below. If you want to change this schema, see [How to Create a Custom Developer App](/howto/mobile/how-to-devapps) to build your own custom developer app and then use this tutorial to change its schema [todo: which sections apply in this case?].

For development and this tutorial we recommend running the app from source against the local running Mendix Studio Pro. This will save you time when rebuilding and redeploying your app. To do this, follow the steps in the [Connecting to a Local Running Instance of Studio Pro](/refguide/native-builder#connect-local) section of the *Native Builder Reference Guide*.

## 2. Prerequisites

Before starting this how-to, make sure you have completed the following prerequisites:

* Complete the [Prerequisites](/howto/mobile/deploying-native-app#prerequisites) section of *How to Deploy Your First Mendix Native App*
* Install git [command line](https://git-scm.com/downloads) tool

## 3. Setting up App Deep Linking

If you do not already have a native template for your app, you can create one.

1. Create a shell app with Native Builder, with the `prepare` command. Using the how to [Deploy Your First Mendix Native App](/howto/mobile/deploying-native-app).

        Example command, replace the parameters with your project parametrs, local paths and tokens:

    ``` shell
    native-builder.exe prepare --project-name "Native Deep Link" --app-name "Native Deep Link" --java-home "C:\Program Files\AdoptOpenJDK\jdk-11.0.3.7-hotspot" --mxbuild-path "C:\Program Files\Mendix\8.6.0.715\modeler\mxbuild.exe" --project-path "C:\mendix-projects\NativeDeepLink\NativeDeepLink.mpr" --github-access-token "c3f322c471623" --appcenter-api-token "2d5b570693d34"  --app-identifier "com.mendix.native.deeplink" --runtime-url "https://nativedeeplink-sandbox.mxapps.io/" --mendix-version "8.6.0"
    ```
    
1. Open your command line interface (CLI) of choice or create a folder on your file system where you want to edit the build template:

    ```shell
    cd c:/github
    ```
    
1. Use git to clone your Native Builder template from GitHub: [todo: deepling -> deeplink?]

    ```shell
    git clone https://github.com/your-account/native-deepling-app
    ```

### 3.1 For Android Apps

The manifest file registers the schema and host on your Android device that will be associated with your Mendix app. [todo: give previous sentence context, and make a sentence here which intros steps]

1. Open the folder that you cloned your template into: `c:/github/native-deepling-app`.
1. Open *android/app/src/main/AndroidManifest.xml*.
1. In `activity`, add the attribute `android:launchMode="singleTask"`. For more information on Launch Mode, see this [Android documentation](https://developer.android.com/guide/topics/manifest/activity-element#lmode).
1. Add an `intent-filter` in the `activity`:

    ```xml
    <intent-filter android:label="@string/app_name">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="app" android:host="myapp" />
    </intent-filter>
    ```
    
    Fr more information on linking in Android, see this [Android documentation](https://developer.android.com/training/app-links/deep-linking#adding-filters).

### 3.2 For iOS Apps

The **plist** registers the schema and host, so that they will be associated with your app in iOS. [todo: plist what? file? Also as above so here.]

1. Open the folder that you cloned your template into: `c:/github/native-deepling-app`.
1. In Xcode (available on Apple Mac only) open *ios/NativeTemplate.xcworkspace*.
1. Open *ios/NativeTemplate/Info.plist*
1. Add `URL types`, then add `URL Schemes` and `URL identifier`:

   ![ios info plist](attachments/native-deep-link/ios-info-plist.png)
   
   When viewing *Info.plist* as a text file, you would see that a section is added:
   
    ```xml
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>app</string>
            </array>
            <key>CFBundleURLName</key>
            <string>myapp</string>
        </dict>
    </array>
    ```

1. Open *ios/AppDelegate.m* 
1. Before `@end`, add a new method:

    ```objc
    - (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
        return [RCTLinkingManager application:application openURL:url sourceApplication:sourceApplication annotation:annotation];
    }
    ```
    
    This method will register the opened URL so it can be used in the Deep Link Nanoflow actions. [todo: I think this should not be capitalized. Check with Andries.]

### 3.3 Rebuilding Your Native App

When running locally from source you have launch your app again, or use the Native Builder to build a new app. [todo: change to "If you are not running locally from source, use the Native Builder to build a new app with the `` command. Add a colon after last sentence here after the 1 method we recommend is made clear.]

1. With your CLI, open the folder that you cloned your template into: `cd c:/github/native-deepling-app`.
1. Add, commit, and push all changes from steps above:

    ```shell
    git add .
    git commit -m "Add deeplink handling"
    git push
    ```
    
1. Now rebuild and install your native app, to add our new capabilities. Use the how to for [template](/howto/mobile/deploying-native-app) or [dev app](/howto/mobile/how-to-devapps) to rebuild the app.

    ``` shell
    native-builder.exe build --project-name "Native Deep Link" --app-version "1.0.0" --build-number 1
    ```

## 4 Using Deep Linking in Your App {#using-deep-linking}

Now your app is ready to use links, so you will set up the additional path and query data handling (todo: check sentence). If you skip this section, the links to your app will just open the app. Nothing will be done with the additional data available in your URL.

### 4.1 Deep Linking Nanoflow Actions (todo: deep linking and nanoflow actions?)

Now you have to handle the incoming URL in your Mendix application. To do this, you will use the Nanoflow Actions **Register Deep Link** and **Parse Url To Object** found in the [Native Mobile Resources](https://appstore.home.mendix.com/link/app/109513/) module. This module is automatically included in your app if it began as an up-to-date Starter App. If you do not see these actions available in your app, please update the module through the App Store.

#### 4.1.1 Registering Deep Link

The Register Deep Link nanoflow action registers a callback nanoflow, which is called each time the app is opened via an URL. This "Callback URL Handler" nanoflow will receive the URL, of type string, as input parameter. Please note that the name of the input parameter is case sensitive and can not be changed.

#### 4.1.2 Parsing a URL To Object

This nanoflow action will create a new Mendix object, split a URL, and set all the oject attributes with their values. For example, the URL https://john.doe:secret@www.example.com:123/forum/questions/?tag=networking&order=newest#top has the following attributes and values:

| Attribute                                                   | Value                                                                                        |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| href                                                        | https://john.doe:secret@www.example.com:123/forum/questions/?tag=networking&order=newest#top |
| protocol                                                    | https:                                                                                       |
| hash                                                        | top                                                                                          |
| query                                                       | ?tag=networking&order=newest                                                                 |
| pathname                                                    | /forum/questions/                                                                            |
| auth                                                        | john.doe                                                                                     |
| host                                                        | www.example.com:123                                                                          |
| port                                                        | 123                                                                                          |
| hostname                                                    | www.example.com                                                                              |
| password                                                    | secret                                                                                       |
| username                                                    | john.doe                                                                                     |
| origin                                                      | https://www.example.com:123                                                                  |
| **Dynamically based on the number of slashes in the paths** |
| path0                                                       | forum                                                                                        |
| path1                                                       | questions                                                                                    |
| **Dynamically based on the number of query keys**           |
| tag                                                         | networking                                                                                   |
| order                                                       | newest                                                                                       |

### 4.2 Using Utilities in Your App [todo: check utilities]

Now that you have the utilities to register and process an URL, you can use them in your application:

1. In your app add the **App events** widget, which is also par of the Native Mobile Resource module, on your home page.
1. Select open the widget and in the tab `App events` section `Page load` select a `On load` action `Call nanoflow`, and create a new nanoflow named **OL_RegisterDeepLink**:

   ![app event register deeplink](attachments/native-deep-link/app-events-register-deep-link.png)
   This nanoflow will be called only once when the app is started.

1. Implement the **OL_RegisterDeepLink** nanoflow, add the action **Register DeepLink**, in the **Url handler**, create an nanoflow name **DL_ShowUrlDetails**:

   ![nanoflow register deeplink](attachments/native-deep-link/nanoflow-register-deep-link.png)
   This nanoflow will be called everytime the app is opened using a URL.

1. To parse the URL into we can use a non persistent entity named **DeepLinkParameter** from the Native Mobile Resource module. If you use query strings or more with you can copy this entity to your own module. The attributes are all optional and you should only add the attributes that are need for you implementation. Beside the standard list of possible URL parts, you can also add the keys of the query string. (For example `?name=Jhon&title=sir`) The attributes are not case sensitive. You can add attributes for path segments of the URL which will be split into `Path0` , `Path1`, and more:

    ![parameter entity](attachments/native-deep-link/entity-parameter.png)

1. Implement the Deep link handler nanoflow **DL_ShowUrlDetails**, like the image below. The nanoflow has one input parameter named **URL** and is of type `string` which is case sensitive. Use the `ParseI Url to Object` nanoflow action, and provide the URL and the entity of the parameter object. The Show message action will display a message with the details of the URL:

   ![nanoflow handle deep link](attachments/native-deep-link/nanoflow-handle-deep-link.png)

### 4.3 Testing Deep Linking

Go add some test links, for example `mayapp://app/task/123` and or `makeitnative://task/123` on your Mendix responsive or mobile page, restart the modeler, and open the page in your browser of your device. Tap the links to test:

![studio pro test page](attachments/native-deep-link/page-test-deep-link.png)

Please note that if you running the app not from a local source, you have to rebuild your app with the Native builder before testing.

## 5. Read more

*   [Native Builder](/refguide/native-builder)
*   [Deploying Native App](/howto/mobile/deploying-native-app)
*   [React Native Linking](https://facebook.github.io/react-native/docs/linking)
*   [Deep Linking Android](https://developer.android.com/training/app-links/deep-linking)
*   [Deep Linking iOS](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)
*   [Universal Linking iOS](https://developer.apple.com/ios/universal-links/)
*   [URL Schema vs Universal Link](https://medium.com/wolox-driving-innovation/ios-deep-linking-url-scheme-vs-universal-links-50abd3802f97)