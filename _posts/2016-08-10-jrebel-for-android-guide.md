---
title: Android Studio提高开发效率必备神器 - JRebel for Android
date: 2016-08-10 14:28:26
categories: 开发工具
tags: JRebel
---


Android开发的同学都知道，随着Android项目越来越大，编译时间也会逐步增加，项目里随便改几行代码，重新编译一下，少则几十秒，多则三四分钟，实在让人崩溃。网上很多技术博客都提出了各种提高编译速度的版本，本人也试验过[**很多方法**](http://www.czhzero.com/2016/07/21/android-studio-compile-speed-up/)，收效甚微。

这篇文章想给大家介绍的内容就是一个让你不用再苦等的AndroidStudio插件工具。

> JRebel for Android
> 
> 官网地址: http://zeroturnaround.com/software/jrebel-for-android/

<!-- more -->

---

## JRebel功能介绍

- 跳过编译，直接安装运行

JRebel的Run功能和Android Studio的Run功能不同，它不会每次Run都检查编译一下所有代码，相反它每次只会处理你修改过的代码，并且直接运行到你的机器上去。

![JRebel](http://o7y1sf21i.bkt.clouddn.com/blog/010/Card1_18_@2x.gif)


- 实时调整UI布局

很多时候Android开发人员会接到产品经理的 "把这个字体改大一点，我看看效果" "把这个按钮先临时隐藏掉" 等等诸如此类需求。虽然只是修改一行代码，但是我们编译运行要三四分钟。是不是很痛苦？

这个时候只要点击一下JRebel的 **[Apply Changes ]** 按钮。你的修改便会直接显现到你的app上，而且你的app都不会回到登录页面，还是停留在原来的页面。是不是很牛？

![JRebel](http://o7y1sf21i.bkt.clouddn.com/blog/010/Card2_18_@2x.gif)

- 方便快捷的修改bug

修改bug时候的时候，与调整ui一样快捷，只需十秒钟就可以验证。

![JRebel](http://o7y1sf21i.bkt.clouddn.com/blog/010/Card3_18_@2x.gif)


## JRebel安装与使用

- 通过AndroidStudiox下载插件

打开AndroidStudio的Prefrences菜单，按下图操作，即可下载安装

![JRebel](http://o7y1sf21i.bkt.clouddn.com/blog/010/lALOZk-yXc0Cxs0Ejw_1167_710.png)


![JRebel](http://o7y1sf21i.bkt.clouddn.com/blog/010/lALOZk-0Ac0Ci80DJw_807_651.png)

如果下载速度过慢，可以先用迅雷下载安装包，再选择本地安装。[Jrebel for Android 1.3.2 下载地址](http://7xvouf.com1.z0.glb.clouddn.com/jrebel-for-android.zip)


![](http://o7y1sf21i.bkt.clouddn.com/blog/010/lALOZmOPi80C0c0Feg_1402_721.png)



- JRebel使用

安装完成后，系统提示你重启AndroidStudio,重启后，会再提示你输入Lisence或者进行试用。

这里我选择的是试用21天。操作完成后，工具栏多出三个按钮。

![](http://o7y1sf21i.bkt.clouddn.com/blog/010/lALOZlAoTE7NAuA_736_78.png)

同时Run菜单栏也会多次三个选项菜单。

![](http://o7y1sf21i.bkt.clouddn.com/blog/010/lALOZlAqrs0Bqc0BUg_338_425.png)

其中，

1. 选项一是JRebel Run, 相当于官方的Run
2. 选项二是JRebel Debug, 相当于官方的Debug
3. 选项三是Apply Changes, 可以不重启app,直接将修改应用到app中。


> 注意，只有先运行选项一或者选择二之后，才能运行三。另外，[Apply Changes]在某些Android 6.0以上机器上无法正常使用，不过选项一和选项二是是可以使用的。期待官方修复这个问题。


## 总结

Jrebel for Android 与 Android Studio 2.0官方的Instant Run对Sdk版本没什么限制。同时官方Instant Run在使用时仍然有许多限制，具体可参看[Android Developers](https://developer.android.com/studio/run/index.html)。不过可惜的是，Jrebel for Android并不是免费的，如何选择，各位看官就仁者见仁智者见智了。
