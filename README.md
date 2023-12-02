## Android Auto as user app with media apps support on GrapheneOS / Android 14

This repository describes how to use my fork of GrapheneOS to add Android Auto support (with media apps support) to Android 14. However, [the changes](#patches-used) themselves don't rely on GrapheneOS, so it should be possible to make this work on a different OS with slight modifications. (If you are looking for an Android 13 patch, see [here](https://github.com/sn-00-x/android-auto-android13))

To make this run, you need to build your own rom. If you are instead looking for a simpler method to run Android Auto (that involves rooting your device), have a look at [aa4mg](https://github.com/sn-00-x/aa4mg)

## Build

I've forked some repositories of GrapheneOS and am going to apply my patches onto their stable releases. However, I will do this irregularly and don't commit to any schedules. It may sometimes take ages for me to release a new version. If you can't wait, you can always apply [the patches](#patches-used) yourself.

When there is a new version, you will find a new tag under [platform_manifest tags](https://github.com/sn-00-x/grapheneos-platform_manifest/tags). The tag name is always "-sn" appended to a valid GrapheneOS stable release. At the time of writing the current tag is _2023112900-sn_

Simply follow the official [GrapheneOS build instructions](https://grapheneos.org/build), but use my platform manifest instead of GrapheneOS's (replace TAG_NAME with a valid tag from [platform_manifest tags](https://github.com/sn-00-x/grapheneos-platform_manifest/tags)):
```
repo init -u https://github.com/sn-00-x/grapheneos-platform_manifest.git -b refs/tags/TAG_NAME
```

That's it. Now you already would be able to install Android Auto as a user app (see [below](#get-android-auto-and-google-maps)).

Optional:
- If you additionally want to use Screen2Auto, you need to change it's package name and set it together with the sha256 hash of your signing cert in `frameworks/base/core/java/android/app/compat/sn00x/AndroidAutoHelper.java` ( PACKAGE_SCREEN2AUTO and SIGNATURE_SCREEN2AUTO). There is a helper script to do that. See [Install Screen2Auto](#install-screen2auto) section below.
- To be able to play content protected by DRM, such as Netflix, the device must be missing liboemcrypto.so. There's a helper script to disable it. Just run ```script/liboemcrypto-disable.sh``` *after downloading proprietary files* (That means: Run it _after_ ```adevtool```).
  Note that without liboemcrypto.so Widevine DRM will fall back to using L3 and Netflix will only stream in SD, but at least can be mirrored to the head unit.

## Get Android Auto and Google Maps

- Be sure to get Android Auto and Maps with correct signatures. When downloading directly from PlayStore (without having preinstalled stubs on your system), the apps may be provided with other signatures. However, those may checked and rejected when connecting to a car.
  Android Auto must be signed with eeb557fc154afc0d8eec621bdc7ea950 / 9ca91f9e704d630ef67a23f52bf1577a92b9ca5d. Get it e.g. [here](https://www.apkmirror.com/apk/google-inc/android-auto/android-auto-10-2-6332-release/android-auto-10-2-633224-release-android-apk-download/) and then update to the latest version.
  Google Maps must be signed with 38918a453d07199354f8b19af05ec6562ced5788 / f0fd6c5b410f25cb25c3b53346c8972fae30f8ee7411df910480ad6b2d60db83. Get it e.g. [here](https://www.apkmirror.com/apk/google-inc/maps/maps-11-93-0307-release/google-maps-11-93-0307-android-apk-download/) and then update to the latest version.
- Go to phone settings and grant "nearby devices" permission to Android Auto
- Connect to car, follow instructions
- In case your device says there is no app to handle the connection, go to phone settings -> Apps -> Android Auto -> Additional settings in the app, then replug the device.
- When an error regarding restricted settings is shown, go to phone settings -> Apps -> Android Auto -> click the three dots in the upper right corner -> Allow restricted settings
- Continue setting up android auto
- In case you run in any problems, replug device and/or try to go to phone settings -> Apps -> Android Auto -> Additional settings in the app -> click "+ CONNECT A CAR"

## Get Google App and Text to Speech

Either get "Google" and "Speech Recognition & Synthesis" from PlayStore or use @SolidHal's [G Apps stub](https://github.com/SolidHal/android-auto-stub/raw/main/gappstub/gappsstub.apk) and [Speech Services stub](https://github.com/SolidHal/android-auto-stub/raw/main/speechservicestub/speech-services-stub.apk) from [SolidHal/android-auto-stub](https://github.com/SolidHal/android-auto-stub).

## Install Screen2Auto

Screen2Auto 3.7 can be obtained from https://inceptive.ru/projects/s2a/download

However Google has blacklisted Screen2Auto (after all it circumvents security restrictions of Android Auto).
So you need to find a way to rename the package and sign it with your own cert. I will neither provide a renamed package nor provide any information on how to do this. I don't want Google to blacklist the version I use or change the way they blacklist apps.

After renaming and signing Screen2Auto, you must set it's package name and cert hash in ```AndroidAutoHelper.java```. There's a helper script to do that. Just run e.g.:
```
script/screen2auto.sh your.package.name 0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF
```

Assuming you have a renamed version of Screen2Auto, install it and only grant "System settings recording" when asked for permissions. Screen2Auto needs this permission to modify system settings to be able to e.g. rotate the screen. Leave the other permissions untouched. The patch in this repo handles granting "display over other apps" permission to Screen2Auto only while connected to a head unit.

Make the following changes in Screen2Auto:
- Touch button settings -> Set Double tab to "Launcher"
- Other settings -> Enable "Alternative touch" (but click cancel when asked for enabling Accessibility Service)

If touch does not work for you correctly, try to select another profile under Other settings -> Alternative touch

## Patches used

frameworks/base:
- [Support Android Auto as user app (part one)](https://github.com/sn-00-x/grapheneos-platform_frameworks_base/commit/5a6e0f90c018a53b52c86609a98abe4d07a6810b)
- [Fake InstallSource to make AA show non-PlayStore apps](https://github.com/sn-00-x/grapheneos-platform_frameworks_base/commit/7aa058964c554e7603af04b687cdeef8e46b8ccf)
- [Screen2Auto as user app](https://github.com/sn-00-x/grapheneos-platform_frameworks_base/commit/b51a4079c4dd7ffad92918a67f1a8f19cc1d805f)
- [Prevent Media Apps from using FLAG_SECURE](https://github.com/sn-00-x/grapheneos-platform_frameworks_base/commit/5f523c886b8fdce56bc39e0925e3c723b03b4acd)

packages/modules/Permission:
- [Support Android Auto as user app (part two)](https://github.com/sn-00-x/grapheneos-platform_packages_modules_Permission/commit/7eb8ea6266fbc7b45639c0529336082f30162ca7)

script:
- [Add helper script to set modified Screen2Auto's package name and certificate hash](https://github.com/sn-00-x/grapheneos-script/commit/4c52d82bc442ce9cf2550322dcee64b608820021)
- [Helper script to disable liboemcrypto.sh](https://github.com/sn-00-x/grapheneos-script/commit/3241f91cf5730bef15e209511e26272e895cac78)

## Thanks
big thanks to everyone involved in the thread here https://github.com/microg/GmsCore/issues/897

https://github.com/VarunS2002/Xposed-Disable-FLAG_SECURE

https://github.com/Magisk-Modules-Repo/liboemcryptodisabler
