  <h3 align="center"><span style="color:#3ddc84;">Android</span> Apps</h3>
  <p align="center">
    Basics of Reverse Engineering/Analysing Android Apps!
    <br/>
</p>

## Android
Android is an open-source, Linux-based software stack

## Major Components of Android Platform
+ Linux Kernel
Android RunTime (ART) Relies on Linux Kernel for underlying functionalities such as threading, low level memory managemant
> Drivers (Audio, Binder, Display, Shared Memory, Bluetooth, Wifi)
> Power Management

+ Hardware Abstraction Layer (HAL)
> (Audio, Bluetooth, Camera, Sensors)

+ Android RunTime
> (ART, Core Libraries)

+ Native C/C++ Libraries
> (WebKit, OpenMax AL, Libc, Media Framework, OpenGL ES)

+ Java API Framework
> Content Providers
> View System
> Manager (Activity, Location, Package, Notification, Resources. Window)

+ System Apps
> (Dialer, Email, Calender, Camera)

## Android Package Kit (APK )
Primary way in which Android apps are distributed and installed.
Upon extraction, contains usually following files :
>+ META-INF (manifest file, app's certificate, list of resources)
>+ RES (resources not compiled into resources.arsc)
>+ AndroidManifest.xml (additional Android Manifest describing name, version etc)
>+ lib (platform-dependent compiled code)
>+ assets (application assets)

## App Sandboxing
The Android platform takes advantage of the Linux user-based protection to identify and isolate app resources. This isolates apps from each other and protects apps and the system from malicious apps. To do this, Android assigns a unique user ID (UID) to each Android application and runs it in its own process. Android uses the UID to set up a kernel-level Application Sandbox.
Reference : https://source.android.com/security/app-sandbox

###### Permissions
> Types
> + Intall-time Permission (less sensitive like access to Internet)
> + Runtime Permission (access to private data like microphone, camera, location)

Reference : https://developer.android.com/guide/topics/permissions/overview.

![workflow_overview.svg](https://developer.android.com/static/images/training/permissions/workflow-overview.svg)

## Tools
+ [Android Studio](https://developer.android.com/studio)
+ Android Debug Bridge (ADB) (installed with Studio, under /Android/sdk/platform-tools) or download separately from [here](https://developer.android.com/studio/releases/platform-tools)
+ [ApkTool](https://ibotpeaches.github.io/Apktool/install)
+ [Enjarify](https://github.com/google/enjarify)
+ [Dex2Jar](https://github.com/pxb1988/dex2jar)
+ [JD-GUI](https://java-decompiler.github.io/)
+ [Bytecode Viewer](https://www.bytecodeviewer.com/)
+ [AndroGuard](https://androguard.readthedocs.io/en/latest/intro/installation.html)
+ [Objection](https://github.com/sensepost/objection)

> Virtual Platforms for Android Security professionals
>+ [AndroidTamer](https://androidtamer.com/)
>+ [Santoku](https://santoku-linux.com/)

## Development
Development is out of context of these notes :)
> Check out : https://developers.google.com/learn

## Analysing Android Apps
![analysing.gif](https://c.tenor.com/q0Cj0U0_4-0AAAAM/genius-smart.gif)
> Types
> + Static Analysis (without executing code)
> + Dynamic Analysis (based on execution/emulation)

##### ADB Basics

> ADB
> + list devices connected
> ```sh
> adb devices 
> ```
> + selects specific devices if more than one connected
> ```sh
> adb -s DEVICE_NAME 
> ```
> + sends file to android device, "/sdcard/" is root internal storage path
> ```sh
> adb push LOCAL_FILE_LOCATION HOST_FILE_LOCATION
> ```
> + retrieves file from android device
> ```sh
> adb pull HOST_FILE_LOCATION LOCAL_FILE_LOCATION
> ```
> + installs apk on android
> ```sh
> adb install myapp.apk
> ```
> + used to enter shell of your device
> ```sh
> adb shell
> ```
> > + gets installed packages and greps the one matching APP_NAME (inside adb shell >>>)
> > ```sh
> > pm list package | grep APP_NAME
> > ```
> + uninstalls app
> ```sh
> adb uninstall MY_APP.PACKAGE.NAME
> ```

##### Extracting .apk from installed app

+ Download [APK Extractor](https://m.apkpure.com/apk-extractor/com.ext.ui) to get .apk file from app already installed.
+ Usually .apk is Extracted at "/storage/emulated/0/ExtractedApks"
+ Use 'adb pull' command to get the apk on ur Local machine.

##### Static Analysis
###### APKtool
+ Decompiling Source from apk
```sh
apktool d MY_APP.apk
```
+ Get into extracted Directory and check *AndroidManifest.xml* to see for all the permissions the app is requesting and see if something's malicious. Permissions in manifest are written in like : 
> + android.permission.RECEIVE_SMS
> + android.permission.READ_CONTACTS
> + android.permission.INTERNET

Inside 'smali' folder, check for .smali (human-readable bytecode) to gather revealed/hard-coded end-ponts like finding just url 'pastebin.com' reveals that it is sending some sort of data over internet most prolly stealing :)

###### ByteCodeViewer
It decompiles the .apk both in smali and java, Java based code decompilation is not guaranteed 100% accurate and is not compilabe whereas smali code files are compilable.
Use different versions of decompilers inside "View > Pane" to see which one works perfect.

Decompilers Available :
+ Procyon
+ CFR
+ FernFlower (default)
+ JD-GUI
+ JADX ....etc

Same Methodolgy, check in for malicious activities by doing source-code review.

###### Enjarify/JD-GUI
+ Use *enjarify* to get the .jar version of .apk file to be further analysed in JD-GUI.
```sh
./enjarify.sh MY_APP.apk
```
+ Import the newly generated MD_HASH_OF_APP-enjarify.jar in JD-GUI for further analysing.

##### Dynamic Analysis 
###### Objection
*Unsigned* apks may encounter few problems especially in Dynamic Analysis. Manual Signing of apk is done from *Android Studio*, Go to "Build > Generate Signed Bundle/APK", create a key and let the rest settings default, Make sure both V1 and V2 are selected under 'Signature Versions'

Now for Dynamic Analysis part with *Objection*, think of it like Android Studio BreakPoint Debugging feature but in runtime.

> + Patch the APK first to work with Objection
> ```sh
> objection patchapk --source MY_APP.apk --gadget-version 12.7.24
> ```
> "--gadget-version" choose which works best.
> + 'adb install' the app, open the app, and it will wait for Objection Explorer
> + Back in terminal type
> ```sh
> objection explore
> ```
> to open Runtime Mobile Exploration, and will prompt with shell
> + Objection RME console
> ```sh
> android hooking list activities
> android hooking list services
> android hooking list receivers
> ```
> To set breakpoint/dump from specific class
> ```sh
> android hooking watch class_method asvid.github.io.fridaapp.MainActivity.sum --dump-args --dump-backtrace --dump-return
> ```

More about Objection : https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/frida-tutorial/objection-tutorial

###### Automated Analysis Sandbox
*Automated Analysis Sandbox/Automated Malware Analysis*
Online Sandbox/Analysing Platforms :
+ [Koodous](https://koodous.com/)
+ [APKlab](https://www.apklab.io/) (Invite-only)
+ [SandDroid](http://sanddroid.xjtu.edu.cn/)

## Methodology/Case Study
![methods.gif](https://c.tenor.com/p2bID6-7Z0oAAAAC/summerheightshigh-dramateacher.gif)

###### Analysing real ransomware (simplocker)
> Find Simplocker.dex source here https://github.com/ashishb/android-malware/tree/master/simplocker

It's a standard ransomware sample that prompts the victim with a warning screen telling them to pay a ransom to restore the device.

**Automated Ananlysis**
Simply checking for *simplocker* with query *tag:simplocker* in [Koodous](https://koodous.com/) gave the Analysis Information as it was already submitted by someone.
Check analysis info here : https://koodous.com/apks/8a918c3aa53ccd89aaa102a235def5dcffa047e75097c1ded2dd2363bae7cf97/general-information

**Static Analysis**

+ To Convert .dex in .jar format
```sh
py -O -m enjarify.main 8E9561215E1ACE91F93B4FAD30DA6F368A9E743D3BE59EA34061ECA8EBAB1F33.dex
```
+ Import .jar file to JD-GUI
+ Analysing org.simplocker.MainService.class, reveals methods of connecting to Tor Service.
$$NOTES.ADD.SCREENSHOT$$
+ Further analysing org.simplocker.Constants.class, reveals *ADMIN_URL*, *CIPHER_KEY (decryption key)*, etc.
$$NOTES.ADD.SCREENSHOT$$
+ org.simplocker.AesCrypt.class shows the methods used to Encrypt and Decrypt the data

**Finding more Android-Malwares samples**
+ https://www.vx-underground.org/
+ https://github.com/ashishb/android-malware
+ More to be added....

**ToDo**
+ Add more Analysis samples maybe :)

