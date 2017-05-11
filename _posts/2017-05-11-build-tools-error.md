---
title: Android Studio Error:failed to find Build Tools revision 25.0.2
date: 2017-05-11 14:50:40
categories: Android
tags:	build-tools
---

Android Studio 升级时，会有各种插件版本需要进行更新。[Android Studio 需要配置的版本号整理](http://czhzero.com/2016/07/21/android-version-configure/) 这里有详细介绍。

本文主要介绍` failed to find Build Tools revision`这个问题。

<!-- more -->

目前来说Android Studio的下载源已经可以不用翻墙，直接下载build tools了，所以下载一般是会成功的。

![](http://o7y1sf21i.bkt.clouddn.com/blog/024/2017051101.png)

但是有时候会遇到，明明下载成功了，Android Stuido却还是提示build fail问题。

打开Android Studio 的Sdk Manager, 发现状态也还是`[not installed]`状态。

![](http://o7y1sf21i.bkt.clouddn.com/blog/024/2017051102.png)


尝试过用Android Studio的tools工具命令，`android list sdk -a`查看版本，版本信息全部正常。

猜想一定是哪个配置文件一直没更新或者错误，从而导致Android Studio一直找不到Build Tools。

![](http://o7y1sf21i.bkt.clouddn.com/blog/024/2017051103.png)


一个个排查，发现这个`source.properties`文件坑了Android Studio,让他一直找不到Build Tools Version。

```
### Android Tool: Source of this archive.
#Tue May 19 11:31:42 CST 2015
Archive.HostOs=macosx
Pkg.License=To get .....\n\nJune 2014.\n    
Pkg.LicenseRef=android-sdk-license
Pkg.Obsolete=
Pkg.Revision=21.1.1
Pkg.SourceUrl=https\://dl-ssl.google.com/android/repository/repository-10.xml

```

删掉这个文件重新，Build成功。貌似没有这个文件sdk照样可以使用。