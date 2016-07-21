---
title: Android Studio 需要配置的版本号整理
date: 2016-07-21 14:16:56
categories: Android
tags: Android Studio
---

Android Studio 使用Gradle进行项目构建，相比Eclipse方便了许多，但是也同时引入各种build.gradle，配置各种工具的版本号。
下面我们就来理一理那些需要配置的版本参数。

<!-- more -->

### JDK 

JDK配置比较简单，安装完JDK，配置环境变量后，
进入到[Project Structure]配置界面JDK路径即可。

![JDK](http://o7y1sf21i.bkt.clouddn.com/blog/4/lALOXzh4gs0Bfc0D1g_982_381.png)




### SDK Platform

配置JDK的时候，[Project Structure]配置界面配置一下Android SDK路径。

另外,在项目模块目录下的build.gradle文件内还要做如下配置:

```
android {
    ...
    compileSdkVersion 23
    ...
}
```


### SDK Build Tools

Android SDK的bulid tools的配置方法与SDK platform的配置方法一样，进入到项目模块目录下的build.gradle文件内做如下配置：


```
android {
    ...
    buildToolsVersion "23.0.3"
    ...
}
```

### SDK Tools

SDK Tools的版本，在gradle文件中不需要配置，是Android SDK的工具组件，与编译关系不大。需要更新的话，可以打开Android Studio工具栏里的[sdk manager]

![SDK Tools](http://o7y1sf21i.bkt.clouddn.com/blog/4/lALOXzh6Is0CzM0Few_1403_716.png)

### Gradle Version

Gradle是整个项目构建版本，一直在更新，具体的更新信息可以参考[gradle官网](https://docs.gradle.org)

修改Gradle版本，同样是进入到[Project Structure]配置界面.

![Gradle Version](http://o7y1sf21i.bkt.clouddn.com/blog/4/lALOXyEtXczizQUC_1282_226.png)


### Android Plugin for Gradle

除了配置gradle版本外，我们还需要配置与之配套的 gradle plugin版本。

进入到项目根目录build.gradle，进行如下配置。


```
buildscript {
  ...
  dependencies {
    classpath 'com.android.tools.build:gradle:2.1.0'
  }
}
```

### 参考文献

- [gradle](https://docs.gradle.org)
- [android developer](https://developer.android.com/studio/releases)






