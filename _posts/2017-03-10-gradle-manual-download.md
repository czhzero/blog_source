---
title: 为Android Studio 项目手动下载gradle
date: 2017-03-10 11:03:21
categories: Android
tags: gradle
---

原文:  http://www.cnblogs.com/duanguyuan/p/3997550.html

在 http://developer.android.com/samples/index.html 上下载的例子，导入Android Studio的时候，第一件事就是下载项目对应版本的gradle。gradle是一个主要面向Java应用的build工具，类似于大家熟知的eclipse下的Ant，但比Ant高级。

<!-- more -->

Andriod Studio下载gradle的过程很慢，在运行./studio.sh的终端中可以看见进度：


把终端中的网址复制下来，在浏览器中打开，可见有38MB多。Windows下可用迅雷下载，Linux下推荐Firefox + DownThemAll插件。

Android Studio下载的各个版本gradle放在~/.gradle/wrapper/dists/目录下（Windows下目录为C:\Users\用户名\.gradle\wrapper\dists）。如下显示了Andriod Studio已经下载了gradle部分文件的大小，可见已经下载了20MB左右：

```

wy@wy-Inspiron-7420:~/.gradle/wrapper/dists/gradle-1.10-bin/6oa4rff9viiqskhgd6uns5v1f8$ ll
total 21432
drwxrwxr-x 2 wy wy     4096 Sep 28 00:27 ./
drwxrwxr-x 3 wy wy     4096 Sep 28 00:20 ../
-rw-rw-r-- 1 wy wy        0 Sep 28 00:20 gradle-1.10-bin.zip.lck
-rw-rw-r-- 1 wy wy 21931207 Sep 28 00:33 gradle-1.10-bin.zip.part

```


在Android Studio中取消下载（不过貌似有个bug，取消不了，那就直接在运行studio.sh的终端中按Ctrl + C 退出Android Studio）。

将gradle-1.10-bin.zip.part移除，把自己下载的gradle-1.10-bin.zip复制到这个目录。然后再次启动Andriod Studio，会自动读取gradle并解压，然后用解压得到的gradle工具build你import进来的sample project。

再次查看gradle的下载目录，如下：

```
wy@wy-Inspiron-7420:~/.gradle/wrapper/dists/gradle-1.10-bin/6oa4rff9viiqskhgd6uns5v1f8$ ll
total 39472
drwxrwxr-x 3 wy wy     4096 Sep 28 00:38 ./
drwxrwxr-x 3 wy wy     4096 Sep 28 00:20 ../
drwxrwxr-x 6 wy wy     4096 Sep 28 00:38 gradle-1.10/
-rw-r----- 1 wy wy 40404574 Sep 28 00:37 gradle-1.10-bin.zip
-rw-rw-r-- 1 wy wy        0 Sep 28 00:20 gradle-1.10-bin.zip.lck
-rw-rw-r-- 1 wy wy        0 Sep 28 00:38 gradle-1.10-bin.zip.ok

```

lck和ok文件大小为0，没有实际内容，起一个标志的作用。ok表示此版本的gradle已经收拾妥当（在下载完毕之前是没有这个ok文件的）。lck文件不知什么作用。（猜测是lock的意思，标记这个版本的gradle是否有project在使用。如果没有被使用，当总的gradle文件达到缓存上限后，此版本的gradle会被删除）