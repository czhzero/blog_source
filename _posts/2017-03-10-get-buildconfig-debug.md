---
title: 如何在子模块中获取BuildConfig.DEBUG值
date: 2017-03-10 10:57:05
categories: Android
tags:	BuildConfig
---

   在Android开发中，我们使用android.util.Log来打印日志，方便我们的开发调试。但是这些代码不想在发布后执行，我们并不想在软件发布后调试日志被其他开发者看到，我们一般可以通过设定一个布尔类型达常量，来标记软件为Debug模式还是Release模式。
ADT 17版本以后，我们可以通过读取build/BuildConfig.java文件里DEBUG常量来判断。

<!-- more -->

```
if (BuildConfig.DEBUG) {
    android.util.Log.d(TAG,"something");
}
```

但是，如果子项目里也需要用到BuildConfig.DEBUG的值，并像如上的方式直接使用的话，会发现BuildConfig.DEBUG获取到的值一直为false。

出现这种现象的原因是，在子项目里虽然也有bulid文件夹，但是编译成最终apk的时候并未正确去正确编译子项目里的BuildConfig.java文件。

因此若要获取到正确的BuildConfig.DEBUG值，还必须通过反射的方式，获取到主项目里的BuildConfig.DEBUG值。

```
public static Object getBuildConfigValue(Context context, String fieldName) {
    try {
        Class<?> clazz = Class.forName(context.getPackageName() + ".BuildConfig");
        Field field = clazz.getField(fieldName);
        return field.get(null);
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (NoSuchFieldException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
    return null;
}

boolean debug = (Boolean) getBuildConfigValue(this, "DEBUG");

```

这样就可以获得到真正的BuildConfig.DEBUG。