---
title: Android性能优化(五)－ 安装包大小优化 
date: 2017-03-08 22:44:07
categories: Android
tags: 性能优化
---

应用安装包大小对应用的使用没有影响，但是如果安装包过大，用户每次更新下载的门槛就会越高，影响用户体验和使用意愿。

<!-- more -->

性能优化系列:

- [Android性能优化(一)－UI优化](http://www.czhzero.com/2017/02/07/performance-optimization-1/)
- [Android性能优化(二)－数据库优化](http://www.czhzero.com/2017/02/07/performance-optimization-2/)
- [Android性能优化(三)－ 移动网络优化](http://www.czhzero.com/2017/02/07/performance-optimization-3/)
- [Android性能优化(四)－ 代码优化](http://www.czhzero.com/2017/02/07/performance-optimization-4/)
- [Android性能优化(五)－ 安装包大小优化]()
- [Android性能优化(六)－ 省电优化]()
- [Android性能优化(七)－ 内存优化]()
- [Android性能优化(八)－ 稳定性优化]()


## 应用安装包构成

### 1.assets

assets可以存放一些不需要编译的资源，如果配置文件，Webview本地资源，图片资源等等。代码中通过AssetManager进行资源的饮用。

### 2.lib

存放native文件，该目录会根据Cpu类型不同而分为不同的目录，从而加载不同的so库。通常分为这四种目录,ARM,ARM-v7a,ARM64-v8a,MIPS,X86等等。

### 3.res

存放资源，这个目录的资源均会被映射到R.java中。

### 4.META-INF

保存应用的签名信息

### 5.AndroidManifest.xml

Android应用程序的配置文件。

### 6.classes.dex

Java字节码class文件合并组合的文件。

### 7.proguard.cfg

代码混淆配置文件

### 8.resources.arsc

记录资源文件和资源ID的映射关系。

> 一般Apk大小中的大头是，classes.dex,lib和资源文件。
> 

## 常用方案

### 1.代码混淆

按照代码混淆规则，对代码进行混淆，压缩和优化。

- proguard-rules.pro编辑
- 配置gradle文件，增加`minifyEnabled true`属性

### 2.资源优化

#### Android Lint去冗余
#### 资源文件最小化

- 尽量使用一套图片
- 使用轻量级的第三方库
- 减少apk中预置图片，改为从服务端下载

#### 图片优化

- 降低图片色彩位数
- 图片压缩


### 3.其他优化

- 避免使用重复的第三方库
- 插件化
- 使用WebP图片格式 

 

