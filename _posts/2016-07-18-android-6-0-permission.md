---
title: Android 6.0 运行时权限处理
date: 2016-07-18 09:58:51
categories: Android
tags: permission
---

> 本文转载自: [阳春面的博客](https://www.aswifter.com/)
> 原文地址: https://www.aswifter.com/2015/11/04/android-6-permission/

### 运行时权限介绍

Android 6.0在我们原有的AndroidManifest.xml声明权限的基础上，
又新增了运行时权限动态检测，以下权限都需要在运行时判断：

```
身体传感器
日历
摄像头
通讯录
地理位置
麦克风
电话
短信
存储空间

```

<!-- more -->

### 运行时权限处理

Android6.0系统默认为targetSdkVersion小于23的应用默认授予了所申请的所有权限，
所以如果你以前的APP设置的targetSdkVersion低于23，在运行时也不会崩溃，
但这也只是一个临时的救急策略，用户还是可以在设置中取消授予的权限。

- 声明目标SDK版本

我们需要在build.gradle中声明targetSdkVersion为23

```
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"

    defaultConfig {
        applicationId "com.yourcomany.app
        minSdkVersion 18
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

```

- 检查并申请权限

我们需要在用到权限的地方，每次都检查是否APP已经拥有权限，
比如我们有一个下载功能，需要写SD卡的权限，
我们在写入之前检查是否有WRITE_EXTERNAL_STORAGE权限，没有则申请权限

```
if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
        != PackageManager.PERMISSION_GRANTED) {
    //申请WRITE_EXTERNAL_STORAGE权限
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
            WRITE_EXTERNAL_STORAGE_REQUEST_CODE);
}

```


- 请求权限后，系统会弹出请求权限的Dialog

- 用户选择允许或拒绝后，会回调onRequestPermissionsResult方法, 该方法类似于onActivityResult

```
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    doNext(requestCode,grantResults);
}

```

- 我们接着需要根据requestCode和grantResults(授权结果)做相应的后续处理

```

private void doNext(int requestCode, int[] grantResults) {
       if (requestCode == WRITE_EXTERNAL_STORAGE_REQUEST_CODE) {
           if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
               // Permission Granted
           } else {
               // Permission Denied
           }
       }
   }

```

### Fragment中运行时权限的特殊处理

- 在Fragment中申请权限，不要使用ActivityCompat.requestPermissions, 直接使用Fragment的requestPermissions方法，否则会回调到Activity的onRequestPermissionsResult
- 如果在Fragment中嵌套Fragment，在子Fragment中使用requestPermissions方法，onRequestPermissionsResult不会回调回来，建议使用getParentFragment().requestPermissions方法，
这个方法会回调到父Fragment中的onRequestPermissionsResult，加入以下代码可以把回调透传到子Fragment

```
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    List<Fragment> fragments = getChildFragmentManager().getFragments();
    if (fragments != null) {
        for (Fragment fragment : fragments) {
            if (fragment != null) {
                fragment.onRequestPermissionsResult(requestCode,permissions,grantResults);
            }
        }
    }
}

```

### 相关开源项目

- [PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher)

使用标注的方式，动态生成类处理运行时权限，目前还不支持嵌套Fragment。

- [RxPermissions](https://github.com/tbruyelle/RxPermissions)

基于RxJava的运行时权限检测框架

- [Grant](https://github.com/anthonycr/Grant)

简化运行时权限的处理，比较灵活

- [android-RuntimePermissions](https://github.com/googlesamples/android-RuntimePermissions)

Google官方的例子

### 附录

以下权限只需要在AndroidManifest.xml中声明即可使用


```
android.permission.ACCESS_LOCATION_EXTRA_COMMANDS
android.permission.ACCESS_NETWORK_STATE
android.permission.ACCESS_NOTIFICATION_POLICY
android.permission.ACCESS_WIFI_STATE
android.permission.ACCESS_WIMAX_STATE
android.permission.BLUETOOTH
android.permission.BLUETOOTH_ADMIN
android.permission.BROADCAST_STICKY
android.permission.CHANGE_NETWORK_STATE
android.permission.CHANGE_WIFI_MULTICAST_STATE
android.permission.CHANGE_WIFI_STATE
android.permission.CHANGE_WIMAX_STATE
android.permission.DISABLE_KEYGUARD
android.permission.EXPAND_STATUS_BAR
android.permission.FLASHLIGHT
android.permission.GET_ACCOUNTS
android.permission.GET_PACKAGE_SIZE
android.permission.INTERNET
android.permission.KILL_BACKGROUND_PROCESSES
android.permission.MODIFY_AUDIO_SETTINGS
android.permission.NFC
android.permission.READ_SYNC_SETTINGS
android.permission.READ_SYNC_STATS
android.permission.RECEIVE_BOOT_COMPLETED
android.permission.REORDER_TASKS
android.permission.REQUEST_INSTALL_PACKAGES
android.permission.SET_TIME_ZONE
android.permission.SET_WALLPAPER
android.permission.SET_WALLPAPER_HINTS
android.permission.SUBSCRIBED_FEEDS_READ
android.permission.TRANSMIT_IR
android.permission.USE_FINGERPRINT
android.permission.VIBRATE
android.permission.WAKE_LOCK
android.permission.WRITE_SYNC_SETTINGS
com.android.alarm.permission.SET_ALARM
com.android.launcher.permission.INSTALL_SHORTCUT
com.android.launcher.permission.UNINSTALL_SHORTCUT


```