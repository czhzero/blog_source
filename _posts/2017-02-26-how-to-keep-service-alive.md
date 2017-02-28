---
title: Android后台进程保活策略汇总
date: 2017-02-26 11:07:32
categories: Android
tags: 后台保活
---

后台常驻一直是Android开发者研究的课题，一方面是为了实现即时通信的长连接，一方面也是为了背地里做一些黑暗的事情。网上后台保活的方案也很多，但是有可执行效果的也不多，尤其是面对国内小米，魅族等深度定制的Android系统。本文也会持续更新，动态收集一些新的方案。

<!-- more -->

## 基本概念

一般来说，后台进程回收分为三种情况，第三种是最霸道的，他们权限最高，后台进程被杀死后，连系统广播都无法让你监听。

- 系统内存不够时的回收
- 360等安全软件的回收
- 厂商后台管理程序的回收

Android系统进程状态分为五个等级，具体每种进程含义这边不细说了。

- 前台进程
- 可见进程
- 服务进程
- 后台进程
- 空进程

这五种状态的进程相对于系统来说的重要性从上至下排列，空进程容易被杀死，其次是后台进程，然后是服务进程甚至是可见进程，而前台进程一般则不会被轻易干掉。

Android进程有两个比较重要的概念，一个是Importance等级，一个是adj值。Importance等级与adj值在ActivityManagerService中被关联起来，相较于Importance等级而言adj值可以赋予我们更多的参考价值。adj值越小，则越不容易回收，这个值会随着进程的状态转换而变化。

我们可以通过adb shell命令实时查看这个adj值。

> adb shell
> 
> ps | grep <关键字>

```
USER     PID   PPID  VSIZE  RSS     WCHAN    PC         NAME
root      1     0     812    668   ffffffff 00000000 S /init
root      2     0     0      0     ffffffff 00000000 S kthreadd

```

> adb shell
> 
> cat /proc/PID/oom_adj
> 

cat命令执行后，会得到一个adj整数数值。

## 策略选择

后台保活两个目标，一是轻易不让系统回收，二是杀死后可以重启。最好的方式当然系统白名单，让系统为你这个进程开个后门，这种方式不再讨论范围。

### 1.Service的onStartCommand函数返回START_STICKY

`START_STICKY`是官方提供的参数，意思是当service被内存回收了，系统会对service进行重启。面对360等内存回收，并没什么作用。

### 2.在service 的onDestory里面重启服务

onDestroy()方法只有在service正常停止的时候才会被调用，面对上述回收的第二与第三种方法没有效果。


### 3.守护线程相互监听

AB两个进程，A进程里面轮询检查B进程是否存活，没存活的话将其拉起，同样B进程里面轮询检查A进程是否存活，没存活的话也将其拉起，而我们的后台逻辑则随便放在某个进程里执行即可。

这种方法面对回收的时候，其实作用也不大，而且很消耗性能。另外，也有人提到过使用两个native进程监控，那种方法没试过。

### 4.AlarmManager or JobScheduler循环触发

```
public class AlarmService extends Service {


    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }


    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        initAlarm(this);
        Log.d("czh", "AlarmService onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    private void initAlarm(Context context) {
        Intent intent = new Intent(context, AlarmReceiver.class);
        intent.setAction("repeating");
        PendingIntent sender = PendingIntent.getBroadcast(context, 0, intent, 0);

        //开始时间
        long firsTime = SystemClock.elapsedRealtime();
        AlarmManager am = (AlarmManager) getSystemService(ALARM_SERVICE);
        am.setRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP, firsTime, 5 * 1000, sender);
    }
}

```

### 5.与系统service绑定

[论Android应用进程长存的可行性](http://blog.csdn.net/aigestudio/article/details/51348408) 一文中，提到用NotificationListenerService代替普通service，从而达到保活的作用。 原理是没有问题，在小米4上亲测过后，发现并没什么用。

```
@TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR2)
public class SimulateNotificationService extends NotificationListenerService {

    @Override
    public void onNotificationPosted(StatusBarNotification sbn) {
        super.onNotificationPosted(sbn);
    }


    @Override
    public void onNotificationRemoved(StatusBarNotification sbn) {
        super.onNotificationRemoved(sbn);
    }
}

```

```
    <service android:name=".service.SimulateNotificationService"
            android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE"
            android:process=":test5">

            <intent-filter>
                <action android:name="android.service.notification.NotificationListenerService" />
            </intent-filter>


        </service>

```



### 6.监听系统Receiver保活

使用Receiver来检测目标进程是否存活不失为一个好方法，静态注册一系列广播，什么开机启动、网络状态变化、时区地区变化、充电状态变化等等等等，这听起来好像很6，而且在大部分手机中都是可行的方案，但是对于深度定制的ROM，是的，又是深度定制，你没有看错，而且代表性人物还是魅族、小米，这两个业界出了名的喜欢“深度定制”系统。 
自从Android 3.1开始系统对我们的应用增加了一种叫做STOPPED的状态，什么叫STOPPED？就是安装了之后从未启动过的，大家可能经常在网上看到对开机广播的解释，说要想应用正确接收到开机广播那么就得先启动一下应用，这个说法的技术支持就来源于此，因为自Android 3.1后所有的系统广播都会在Intent添加一个叫做FLAG_EXCLUDE_STOPPED_PACKAGES的标识，说白了就是所有处于STOPPED状态的应用都不可以接收到系统广播。

### 7.提高进程优先级, Notification提权

这种保活手段是应用范围最广泛。它是利用系统的漏洞来启动一个前台的Service进程，与普通的启动方式区别在于，它不会在系统通知栏处出现一个Notification，看起来就如同运行着一个后台Service进程一样。这样做带来的好处就是，用户无法察觉到你运行着一个前台进程（因为看不到Notification）,但你的进程优先级又是高于普通后台进程的。

这种方法面对第二种回收方式有效，但是面对小米之类的后台回收，还是无能为力。

```
public class NotificationService extends Service {


    private final static int SERVICE_ID = 1001;

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }


    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        if (Build.VERSION.SDK_INT < 18) {
            startForeground(SERVICE_ID, new Notification());
        } else {
            Intent innerIntent = new Intent(this, InnerService.class);
            startService(innerIntent);
            startForeground(SERVICE_ID, new Notification());
        }


        return super.onStartCommand(intent, flags, startId);
    }


    /**
     * 给 API >= 18
     */
    public static class InnerService extends Service {

        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            startForeground(SERVICE_ID, new Notification());
            stopForeground(true);
            stopSelf();
            return super.onStartCommand(intent, flags, startId);
        }

        @Nullable
        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }

    }

}

```


### 8.不同的app进程，用广播相互唤醒

如果你手机安装了各种app,或者应用了各种第三方代sdk,即可互相唤醒。
假如你手机里装了支付宝、淘宝、天猫、UC等阿里系的app，那么你打开任意一个阿里系的app后，有可能就顺便把其他阿里系的app给唤醒了。

这个方法针对内存回收的三种方式均有效，只要有一个活着，其他的就会活下来。






## 总结

目前相对靠谱的方法就是6、7、8。第6种方法，要看手机，很多手机会屏蔽系统广播，第7种方法还是那句话，有条件就干。第8种方法在一定程度上保活，面对小米手机rom到系统回收还是没有办法的。


## 参考文献

- [论Android应用进程长存的可行性](http://blog.csdn.net/aigestudio/article/details/51348408)
- [关于 Android 进程保活，你所需要知道的一切](http://www.jianshu.com/p/63aafe3c12af)




