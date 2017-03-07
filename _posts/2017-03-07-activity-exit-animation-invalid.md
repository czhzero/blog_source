---
title: 设置Activity退出动画无效问题的解决
date: 2017-03-07 15:52:29
categories: Bug分析
tags:	动画
---

Activity系统默认的进入动画是从右侧进入到左侧停止，退出动画是从左到右移动直到完全退出界面。如果要修改Activity进入和退出动画有两种方式。

<!-- more -->


## 第一种方式:overridePendingTransition方法

- startActivity()方法之前调用overridePendingTransition方法。

```

startActivity(intent);
overridePendingTransition(R.anim.fade_in, R.anim.fade_out);

```


- 重写Activity的finish方法, 并调用overridePendingTransition方法。

```
@Override
public void finish() {
    super.finish();
    overridePendingTransition(R.anim.fade_in, R.anim.fade_out);
}

```

## 第二种方式:设置Activity的theme属性


在`values`文件夹的`styles.xml`中增加样式

```
<style name="Anim_fade parent="android:Theme.Light.NoTitleBar.Fullscreen">
    <item name="android:windowAnimationStyle">@style/fade</item>
</style>

<style name="fade" parent="@android:style/Animation.Activity">
    <item name="android:activityOpenEnterAnimation">@anim/fade_in</item>
    <item name="android:activityOpenExitAnimation">@anim/fade_out</item>
    <item name="android:activityCloseEnterAnimation">@anim/fade_in</item>
    <item name="android:activityCloseExitAnimation">@anim/fade_out</item>
</style>

```

在AndroidManifest.xml文件中设置Activity的样式,

```
<activity
    Android:name=".TestActivity"
    android:theme="@style/Anim_fade" >
</activity>

```

## 解决方案

为了简洁通用，推荐使用第二种方式进行设置动画。使用第二种方式，有的机器虽然进入的动画是可用的，但是退出的动画无效，你必须使用第一种方式的重写finish方法实现退出动画。






