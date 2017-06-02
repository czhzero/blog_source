---
title: Activity的setContentView方法报空指针错误
date: 2017-06-02 17:25:40
categories: Bug分析
tags: setContentView createViewFromTag
---

Android写xml的时候，有时候写错某些标签，编译不会报错，例如码字的时候，可能会不小心把一些基本单词的大小写混淆。

<!-- more -->

```
 <view
            android:layout_width="match_parent"
            android:layout_height="0.5dp"
            android:layout_marginBottom="@dimen/margin_normal"
            android:layout_marginTop="@dimen/margin_normal"
            android:background="@color/grey_E0E3E1" />

```

> 这里的`view`必须首字母大写，否则Activity的setContentView方法会报错。
> 

错误日志如下，

```

FATAL EXCEPTION: main
Process: com.gezbox.winder.parttime, PID: 12924
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.gezbox.winder.parttime/com.gezbox.task.activity.OrderIncomeReportActivity}: java.lang.NullPointerException: Attempt to invoke virtual method 'boolean java.lang.String.equals(java.lang.Object)' on a null object reference
at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2660)
at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2730)
at android.app.ActivityThread.access$800(ActivityThread.java:185)
at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1567)
at android.os.Handler.dispatchMessage(Handler.java:111)
at android.os.Looper.loop(Looper.java:194)
at android.app.ActivityThread.main(ActivityThread.java:5847)
at java.lang.reflect.Method.invoke(Native Method)
at java.lang.reflect.Method.invoke(Method.java:372)
at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1010)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:805)
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'boolean java.lang.String.equals(java.lang.Object)' on a null object reference

//createViewFromTag方法报错
at android.view.LayoutInflater.createViewFromTag(LayoutInflater.java:721)

at android.view.LayoutInflater.rInflate(LayoutInflater.java:812)
at android.view.LayoutInflater.rInflate(LayoutInflater.java:815)
at android.view.LayoutInflater.inflate(LayoutInflater.java:510)
at android.view.LayoutInflater.inflate(LayoutInflater.java:420)
at android.view.LayoutInflater.inflate(LayoutInflater.java:371)
at com.android.internal.policy.impl.PhoneWindow.setContentView(PhoneWindow.java:437)

//SetContentView方法报错

at android.app.Activity.setContentView(Activity.java:2205)
at com.gezbox.task.activity.OrderIncomeReportActivity.onCreate(OrderIncomeReportActivity.java:24)
at android.app.Activity.performCreate(Activity.java:6117)
at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1123)
at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2613)
at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2730) 
at android.app.ActivityThread.access$800(ActivityThread.java:185) 
at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1567) 
at android.os.Handler.dispatchMessage(Handler.java:111) 
at android.os.Looper.loop(Looper.java:194) 
at android.app.ActivityThread.main(ActivityThread.java:5847) 
at java.lang.reflect.Method.invoke(Native Method) 
at java.lang.reflect.Method.invoke(Method.java:372) 
at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1010) 
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:805)

```

遇到这种错误大小写错误，一个不注意，还真有可能浪费很多时间。